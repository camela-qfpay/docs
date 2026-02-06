---
id: "fps-app-call-app"
title: "FPS App-to-App Integration"
sidebar_label: "FPS App-to-App Integration"
description: "Integration guide for FPS App-to-App and H5-to-App payments (Android & iOS)"
---

This page explains how a merchant App or H5 page can use QFPay API, Android Intents, or iOS Universal Links to launch bank payment apps and complete FPS App-to-App payment flows. Android and iOS platforms are supported. Code samples and demo links are provided.

---

## Integration Overview

This guide provides integration methods for FPS App-to-App payments across Android and iOS platforms. Based on your app architecture, choose the relevant flow from below.

| Platform | Flow Type  | Usage Scenario                         | Launch Method              |
| -------- | ---------- | -------------------------------------- | -------------------------- |
| Android  | Native App | Your Android app directly triggers FPS | `Intent` to FPS App        |
| Android  | H5 to App  | WebView in Android app triggers FPS    | JSBridge + `Intent`        |
| iOS      | Native App | Your iOS app directly triggers FPS     | `UIActivityViewController` |
| iOS      | H5 to App  | WKWebView in iOS app triggers FPS      | WKWebView + JSBridge       |

For each integration method, you will:

1. Retrieve payment link via API.
2. Use platform-specific code to open the FPS bank app.
3. Receive callback and verify result.

<Link href="/img/fps_app-call-app_en.jpeg" target="_blank">
  ![FPS App-call-app process-flow](@site/static/img/fps_app-call-app_en.jpeg)
</Link>

## 1. Retrieve Payment Parameters

**API Endpoint**: `/trade/v1/payment` **Method**: `POST` **PayType**: `800210`

### Request Parameters

| Parameter      | Required | Type   | Description                                                                                                    |
| -------------- | -------- | ------ | -------------------------------------------------------------------------------------------------------------- |
| `pay_type`     | ✅        | String | Fixed value `800210` for FPS App-to-App                                                                        |
| `txamt`        | ✅        | Int    | Transaction amount in cents (e.g., 100 = \$1)                                                                  |
| `txdtm`        | ✅        | String | Transaction time (format: `YYYY-MM-DD hh:mm:ss`)                                                               |
| `out_trade_no` | ✅        | String | Merchant’s unique transaction reference                                                                        |
| ...            | ...      | ...    | See [Public Request Parameters](/docs/api-reference/request-format#public-payment-request-parameters) for more |

:::warning For HSBC FPS Merchants Only

If your FPS integration is via **HSBC direct channel**, you must apply for a **dedicated domain SSL certificate** that matches your merchant legal name.

This is required to:

- Enable secure callbacks via Universal Links (iOS) or HTTPS redirects (Android)
- Complete certification with HSBC and pass production onboarding

📄 See [FPS e-Cert Setup Guide](/docs/online-shop/integration-by-payment-type/fps/fps-ecert-setup) for application steps, CSR command, and document checklist.

:::

### Response Parameters

| Name          | Type        | Description                                                                             |
| ------------- | ----------- | --------------------------------------------------------------------------------------- |
| Public Params | -           | See [Public Response Parameters](/docs/api-reference/response-format#field-description) |
| `pay_params`  | String(128) | URL to launch the FPS payment app, e.g. `https://fps.xxx/xxx`                           |

---

## 2. Android FPS Payment Flow

### 2.1 Native App-to-App Flow (Android)

This flow allows a native Android app to directly launch a bank payment app via Intent. The result is returned through `onActivityResult`. You should still verify final payment status via your backend.

1. The merchant retrieves the `pay_params` URL from API
2. Use Android `Intent` to launch the FPS payment app
3. Set Intent Action to `hk.com.hkicl`, Key: `url`, Value: pay URL
4. Use `startActivityForResult` to launch the bank app
5. Handle result in `onActivityResult`
6. Merchant should verify final result using their own order query API

> Reference Code: Android Java Example

```java
// Request code to identify the payment result
int payRequestCode = 100;

// Payment URL retrieved from the backend
String payUrl = "https://fps.qfpay.global/trade/v1/urltranslate/PAYCORE_SHORT_URL_202511075370911194";

// Create an Intent to launch the target payment app (e.g., bank app)
Intent intent = new Intent("hk.com.hkicl");

// Add the payment URL as a parameter to the Intent
intent.putExtra("url", payUrl);

// Start the payment app using the Intent
startActivityForResult(intent, payRequestCode);


// Handle the result returned from the payment app
@Override
public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data, @NonNull ComponentCaller caller) {
    super.onActivityResult(requestCode, resultCode, data, caller);
    if (requestCode == payRequestCode) {
        // Check payment result
        if (resultCode == RESULT_OK) {
            // Payment success
        } else if (resultCode == RESULT_CANCELED) {
            // Payment failed or cancelled
        }
    }
}
```

### 2.2 H5-to-App Flow via Android WebView

This flow is for Android apps using WebView to trigger FPS bank payment app from an H5 page using JSBridge.”

1. WebView must enable JavaScript: `setJavaScriptEnabled(true)`
2. Bind JS bridge via `addJavascriptInterface`
3. H5 invokes Android with: `AndroidBridge.handleMessage(JSON.stringify({ url: 'https://fps.qfapi.com/xxx' }))`
4. Android receives and constructs `Intent` to launch FPS payment app
5. Use `startActivityForResult` to start payment
6. On result, use `evaluateJavascript` to send result back to H5
7. Merchant should verify with their backend API

> Reference Code: Android WebView Example — See full demo below.

```java
public class Web2AppCallPayActivity extends Activity {

    /**
     * The H5 payment page URL loaded by the merchant app
     */
    private static final String WEB_PAY_LINK = "https://img-int.qfapi.com/upstatic/20251119/fpsH5CallApp/index.html";

    /**
     * Internal WebView component of the merchant app
     */
    private WebView webView;

    /**
     * Callback ID provided when H5 initiates payment
     */
    private String callBackId;

    /**
     * Request code used to identify payment result
     */
    int payRequestCode = 100;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState, @Nullable PersistableBundle persistentState) {
        super.onCreate(savedInstanceState, persistentState);
        setContentView(R.layout.more_view);

        webView = findViewById(R.id.web_pay);
        // Enable JavaScript interaction in WebView
        webView.getSettings().setJavaScriptEnabled(true);
        // Bind JavaScript interface for communication from H5 to App
        webView.addJavascriptInterface(this, "AndroidInterface");
        // Load the H5 payment page
        webView.loadUrl(WEB_PAY_LINK);
    }

    @Override
    public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data, @NonNull ComponentCaller caller) {
        super.onActivityResult(requestCode, resultCode, data, caller);
        if (requestCode == payRequestCode) { // Return result from launching payment

            // Wrap the result into an object and convert to JSON string to return to H5
            EvaluateBean evaluateBean = new EvaluateBean();
            evaluateBean.setCode(resultCode); // Set result code
            if (resultCode == RESULT_OK) { // Payment successful
                evaluateBean.setRespmsg("Pay Success");
            } else if (resultCode == RESULT_CANCELED) {  // Payment cancelled
                evaluateBean.setRespmsg("Pay Cancel");
            }
            String evaluateJson = new Gson().toJson(evaluateBean); // Convert result to JSON

            // Return payment result to H5 by invoking `window.handleNativeCallback()`
            // Two parameters are passed:
            // 1. callbackId passed in by H5 when initiating payment
            // 2. Result object (code and message), converted to JSON string
            webView.evaluateJavascript("javascript:window.handleNativeCallback('" + callBackId + ")" + ",(" + evaluateJson + "')", null);
        }
    }

    /**
     * This method is called by the H5 page to notify the merchant app to start payment
     * It must be annotated with @JavascriptInterface and named `handleMessage`
     *
     * @param paramFromWebPay The payment parameters passed from H5
     */
    @JavascriptInterface
    public void handleMessage(String paramFromWebPay) {
        if (TextUtils.isEmpty(paramFromWebPay)) return;
        WebParamsBean webParamsBean = new Gson().fromJson(paramFromWebPay, WebParamsBean.class);

        // Store the callback ID for later result response
        callBackId = webParamsBean.getCallbackId();

        // Retrieve the payment URL returned from H5
        String paymentRequestURL = webParamsBean.getParams().getPaymentRequestURL();

        // Launch the local bank app to start payment
        launchBankPay(paymentRequestURL);
    }

    /**
     * Merchant app launches local bank app with the payment request URL
     *
     * @param paymentRequestURL The payment URL passed from H5
     */
    private void launchBankPay(String paymentRequestURL) {
        Intent intent = new Intent("hk.com.hkicl");
        intent.putExtra("url", paymentRequestURL);
        startActivityForResult(intent, payRequestCode);
    }
}
```

---

## 3. iOS FPS Payment Flow

### 3.1 Native App-to-App Flow

This iOS integration uses `UIActivityViewController` and App Extensions to trigger the FPS bank app from a native app.

1. Merchant calls API to retrieve pay URL (`pay_params`)
2. iOS App uses `UIActivityViewController` + App Extension to launch bank app
3. Consumer completes payment and redirects back using `callback` Universal Link
4. Merchant verifies result via internal order API

#### Integration Steps

- Create a `NSExtensionItem`
- Package `pay_params` and `callback` inside `NSItemProvider`
- Set UTI type to `hk.com.hkicl`
- Use `UIActivityViewController` to launch app selector

🔗 Official Apple Docs:

- [Supporting Universal Links](https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app)
- [Configuring Associated Domains](https://developer.apple.com/documentation/xcode/supporting-associated-domains)

<Tabs>
  <Tab title="Tab">
    ```objectivec
    #import "FPSAppCallAppTool.h"
    #import <UIKit/UIKit.h>
    #import "define.h"
    
    @implementation FPSAppCallAppTool
    
    + (FPSAppCallAppTool *)shareInstance {
        static FPSAppCallAppTool *model = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            if (!model) {
                model = [[FPSAppCallAppTool alloc] init];
            }
        });
        return model;
    }
    
    // Handle and broadcast the FPS payment result
    - (void)fpsPaymentResult:(NSDictionary *)result {
        NSLog(@"%@", result);
        [[NSNotificationCenter defaultCenter] postNotificationName:kNotificationNameFPSPaymentH5CallAppResult
                                                            object:result];
    }
    
    // Get the current active root ViewController
    - (UIViewController *)getCurrentWindowRootVC {
        for (UIScene *scene in [UIApplication sharedApplication].connectedScenes) {
            if ([scene isKindOfClass:[UIWindowScene class]]) {
                UIWindowScene *windowScene = (UIWindowScene *)scene;
                for (UIWindow *window in windowScene.windows) {
                    if (window.isKeyWindow) {
                        return window.rootViewController;
                    }
                }
            }
        }
        return nil;
    }
    
    - (void)invokePaymentExtension:(NSString *)paymentRequestURL {
        // 1. Retrieve payment parameter URL
        // (Example: https://fps.qfpay.global/trade/v1/urltranslate/PAYCORE_SHORT_URL_202511075370911194)
        // In demo mode, this value can be entered manually via input field
        @try {
            if (!paymentRequestURL || paymentRequestURL.length <= 0) {
                [self showAlert];
                return;
            }
        } @catch (NSException *exception) {
            return;
        }
    
        // Merchant Universal Link for callback after payment
        // Refer to Apple documentation:
        // https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app
        NSString *callbackURL = @"https://img-int.qfapi.com/trade/123";
    
        // 2. Package URL and callback into a dictionary
        NSDictionary *paymentPayload = @{
            @"URL": paymentRequestURL,
            @"callback": callbackURL
        };
    
        NSItemProvider *itemProvider = [[NSItemProvider alloc]
            initWithItem:paymentPayload
            typeIdentifier:@"hk.com.hkicl"];
    
        // 4. Create NSExtensionItem and attach payload
        NSExtensionItem *extensionItem = [[NSExtensionItem alloc] init];
        extensionItem.attachments = @[itemProvider];
    
        // 5. Initialise UIActivityViewController (system app chooser)
        UIActivityViewController *activityVC = [[UIActivityViewController alloc]
            initWithActivityItems:@[extensionItem]
            applicationActivities:nil];
    
        // 6. iPad adaptation (popover must be specified to avoid crash)
        if ([[UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPad) {
            activityVC.popoverPresentationController.sourceView =
                [self getCurrentWindowRootVC].view;
            activityVC.popoverPresentationController.sourceRect =
                [self getCurrentWindowRootVC].view.frame;
            activityVC.popoverPresentationController.permittedArrowDirections =
                UIPopoverArrowDirectionUp;
        }
    
        // 7. Handle temporary results returned by the extension
        // (This is NOT the final payment status)
        activityVC.completionWithItemsHandler = ^(UIActivityType _Nullable activityType,
                                                  BOOL completed,
                                                  NSArray * _Nullable returnedItems,
                                                  NSError * _Nullable error) {
            if (completed) {
                NSLog(@"User selected extension: %@, process completed", activityType);
                // Temporary extension return data can be parsed here if required
            } else if (error) {
                NSLog(@"Extension invocation failed: %@", error.localizedDescription);
            } else {
                NSLog(@"User cancelled the operation");
            }
        };
    
        // 8. Present the system app chooser
        [[self getCurrentWindowRootVC] presentViewController:activityVC
                                                    animated:YES
                                                  completion:nil];
    }
    
    // Alert when payment parameter is missing
    - (void)showAlert {
        UIAlertController *alert = [UIAlertController
            alertControllerWithTitle:@"Missing Parameters"
                             message:@"Please enter FPS payment parameters first"
                      preferredStyle:UIAlertControllerStyleAlert];
    
        [alert addAction:[UIAlertAction actionWithTitle:@"OK"
                                                  style:UIAlertActionStyleCancel
                                                handler:nil]];
    
        [[self getCurrentWindowRootVC] presentViewController:alert
                                                    animated:YES
                                                  completion:nil];
    }
    
    #pragma mark - Parse query parameters from callback URL
    - (void)parseQueryParamsFromCallbackURL:(NSURL *)url {
        NSURLComponents *components =
            [NSURLComponents componentsWithURL:url
                        resolvingAgainstBaseURL:NO];
        NSArray<NSURLQueryItem *> *queryItems = components.queryItems;
    
        NSMutableDictionary *params = [NSMutableDictionary dictionary];
        for (NSURLQueryItem *item in queryItems) {
            if (item.value) {
                params[item.name] = item.value;
            }
        }
    
        [self fpsPaymentResult:[params copy]];
    }
    
    @end
    
    ```
  </Tab>
  <Tab title="Tab">
    ```swift
    import Foundation
    import UIKit
    
    class FPSAppCallAppTool: NSObject {
        static let shared = FPSAppCallAppTool()
        
        /// Broadcast FPS payment result using NotificationCenter
        func fpsPaymentResult(_ result: [String: Any]) {
            print(result)
            NotificationCenter.default.post(
                name: NSNotification.Name("kNotificationNameFPSPaymentH5CallAppResult"),
                object: result
            )
        }
        
        /// Invoke FPS payment via iOS App Extension
        func invokePaymentExtension(paymentRequestURL: String?) {
            // 1. Retrieve payment parameters (e.g. from input field)
            guard let paymentRequestURL = paymentRequestURL, !paymentRequestURL.isEmpty else {
                showAlert()
                return
            }
    
            // Merchant's Universal Link for payment callback
            // See Apple docs: https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app
            let callbackURL = "https://img-int.qfapi.com/trade/123"
    
            // 2. Package data as dictionary
            let paymentPayload: [String: String] = [
                "URL": paymentRequestURL,
                "callback": callbackURL
            ]
    
            // 3. Create NSItemProvider
            // Note: Dictionary must conform to NSCoding, so cast as NSDictionary
            let itemProvider = NSItemProvider(
                item: paymentPayload as NSDictionary,
                typeIdentifier: "hk.com.hkicl"
            )
    
            // 4. Create NSExtensionItem and attach payload
            let extensionItem = NSExtensionItem()
            extensionItem.attachments = [itemProvider]
    
            // 5. Create UIActivityViewController to launch external apps
            let activityVC = UIActivityViewController(
                activityItems: [extensionItem],
                applicationActivities: nil
            )
    
            // 6. iPad-specific: Configure popover presentation to avoid crash
            if UIDevice.current.userInterfaceIdiom == .pad,
               let rootVC = getCurrentWindowRootVC() {
                activityVC.popoverPresentationController?.sourceView = rootVC.view
                activityVC.popoverPresentationController?.sourceRect = rootVC.view.bounds
                activityVC.popoverPresentationController?.permittedArrowDirections = .up
            }
    
            // 7. Handle activity result (Note: not actual payment status)
            activityVC.completionWithItemsHandler = { [weak self] activityType, completed, returnedItems, error in
                if completed {
                    print("User selected extension: \(activityType?.rawValue ?? "unknown"), completed")
                    // You may parse returnedItems here if needed
                } else if let error = error {
                    print("Extension failed: \(error.localizedDescription)")
                } else {
                    print("User cancelled the operation")
                }
            }
    
            // 8. Present the app chooser
            if let rootVC = getCurrentWindowRootVC() {
                rootVC.present(activityVC, animated: true)
            } else {
                showAlert(message: "Unable to get current view. Cannot initiate payment.")
            }
        }
        
        /// Get the current root UIViewController
        private func getCurrentWindowRootVC() -> UIViewController? {
            for scene in UIApplication.shared.connectedScenes {
                guard let windowScene = scene as? UIWindowScene else { continue }
                for window in windowScene.windows where window.isKeyWindow {
                    return window.rootViewController
                }
            }
            return nil
        }
        
        /// Display alert with default or custom message
        private func showAlert(
            title: String = "Missing Parameter",
            message: String = "Please enter the FPS payment parameter first"
        ) {
            let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "OK", style: .cancel))
    
            if let rootVC = getCurrentWindowRootVC() {
                rootVC.present(alert, animated: true)
            }
        }
        
        /// Parse query parameters from the callback URL after payment
        func parseQueryParamsFromCallbackURL(_ url: URL) {
            guard let components = URLComponents(url: url, resolvingAgainstBaseURL: false),
                  let queryItems = components.queryItems else {
                return
            }
            
            var params: [String: String] = [:]
            for item in queryItems {
                if let value = item.value {
                    params[item.name] = value
                }
            }
            
            fpsPaymentResult(params)
        }
    }
    ```
  </Tab>
</Tabs>

---

### 3.2 H5-to-App Flow via WKWebView

This is for H5 pages rendered in iOS WKWebView, which use JSBridge to trigger the bank app and return result.

1. H5 retrieves pay parameters from API
2. Use JsBridge to send `pay_params` to native app
3. Native app launches FPS App via `UIActivityViewController`
4. After payment, native app sends result back to H5 via `evaluateJavaScript`
5. Implemented using `WKWebView` + JSBridge

<Tabs>
  <Tab title="Tab">
    ```objectivec
    #import "FPSWKWebView.h"
    #import <WebKit/WebKit.h>
    #import "FPSAppCallAppTool.h"
    #import "define.h"
    
    @interface FPSWKWebView ()<WKScriptMessageHandler, WKNavigationDelegate, WKUIDelegate>
    @property(copy, nonatomic) NSString *callbackId;
    @end
    
    @implementation FPSWKWebView
    
    - (instancetype)initWithFrame:(CGRect)frame{
        WKUserContentController *userContentController = [[WKUserContentController alloc] init];
        
        // Register JavaScript message channel named "NativeBridge"
        [userContentController addScriptMessageHandler:self name:@"NativeBridge"];
        
        WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
        
        // Web process pool
        config.processPool = [[WKProcessPool alloc] init];
        config.userContentController = userContentController;
        
        // Allow JavaScript to open windows automatically
        config.preferences.javaScriptCanOpenWindowsAutomatically = YES;
        
        // Allow file access if needed
        [config.preferences setValue:@YES forKey:@"allowFileAccessFromFileURLs"];
        
        self = [super initWithFrame:frame configuration:config];
        if (self) {
            self.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
            self.navigationDelegate = self;
            self.UIDelegate = self;
            
            if (@available(iOS 16.4, *)) {
                self.inspectable = YES;
            }
            
            // Listen for FPS payment result notifications
            [[NSNotificationCenter defaultCenter] addObserver:self
                                                     selector:@selector(fpsPaymentResult:)
                                                         name:kNotificationNameFPSPaymentH5CallAppResult
                                                       object:nil];
        }
        return self;
    }
    
    #pragma mark - Receive Payment Result from Native App
    
    - (void)fpsPaymentResult:(NSNotification *)notification {
        NSLog(@"%@", notification.object);
        
        NSDictionary *params = nil;
        NSString *ret = notification.object[@"is_successful"];
        
        // Construct payment result response
        if ([ret isEqualToString:@"0"]) {
            params = @{
                @"code": @"3000",
                @"respmsg": @"Failed",
            };
        } else {
            params = @{
                @"code": @"0000",
                @"respmsg": @"Success",
            };
        }
        
        // Convert result to JSON string
        NSError *error;
        NSData *jsonData = [NSJSONSerialization dataWithJSONObject:params options:0 error:&error];
        if (error) {
            NSLog(@"JSON serialization failed: %@", error);
            return;
        }
        
        NSString *jsonString = [[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];
        
        // Construct JavaScript callback
        NSString *jsCode = [NSString stringWithFormat:
            @"window.handleNativeCallback('%@', '%@');",
            self.callbackId,
            jsonString
        ];
        
        // Execute JavaScript callback
        [self evaluateJavaScript:jsCode completionHandler:^(id _Nullable result, NSError * _Nullable error) {
            if (error) {
                NSLog(@"❌ JavaScript injection failed: %@", error);
            } else {
                NSLog(@"✅ JavaScript callback executed successfully");
            }
        }];
    }
    
    #pragma mark - WKScriptMessageHandler
    // Handle messages sent from JavaScript
    
    - (void)userContentController:(WKUserContentController *)userContentController
          didReceiveScriptMessage:(WKScriptMessage *)message {
        
        if ([message.name isEqualToString:@"NativeBridge"]) {
            NSDictionary *body = message.body;
            
            // Example JS payload:
            // { action: "FPSH5CallApp", callbackId: "callback_123", params: {...} }
            
            NSString *action = body[@"action"];
            NSString *callbackId = body[@"callbackId"];   // JS callback ID
            
            if ([action isEqualToString:@"FPSH5CallApp"]) {
                // Retrieve payment request URL from H5
                NSString *paymentRequestURL = body[@"params"][@"paymentRequestURL"];
                self.callbackId = callbackId;
                
                // Launch FPS payment app
                [[FPSAppCallAppTool shareInstance] invokePaymentExtension:paymentRequestURL];
            }
        }
    }
    
    #pragma mark - Get Root View Controller
    
    - (UIViewController *)getCurrentWindowRootVC {
        for (UIScene *scene in [UIApplication sharedApplication].connectedScenes) {
            if ([scene isKindOfClass:[UIWindowScene class]]) {
                UIWindowScene *windowScene = (UIWindowScene *)scene;
                for (UIWindow *window in windowScene.windows) {
                    if (window.isKeyWindow) {
                        return window.rootViewController;
                    }
                }
            }
        }
        return nil;
    }
    
    #pragma mark - WKUIDelegate
    // Handle JavaScript alert dialogs
    
    - (void)webView:(WKWebView *)webView
    runJavaScriptAlertPanelWithMessage:(nonnull NSString *)message
    initiatedByFrame:(nonnull WKFrameInfo *)frame
    completionHandler:(nonnull WK_SWIFT_UI_ACTOR void (^)(void))completionHandler {
        
        NSLog(@"%@", message);
        
        UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"Notice"
                                                                       message:message
                                                                preferredStyle:UIAlertControllerStyleAlert];
        
        [alert addAction:[UIAlertAction actionWithTitle:@"OK"
                                                  style:UIAlertActionStyleDefault
                                                handler:^(UIAlertAction * _Nonnull action) {
            completionHandler(); // Must be called! Otherwise, JavaScript will hang.
        }]];
        
        [[self getCurrentWindowRootVC] presentViewController:alert animated:YES completion:nil];
    }
    
    #pragma mark - WKNavigationDelegate
    
    - (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(WKNavigation *)navigation {
        NSLog(@"Page started loading...");
    }
    
    - (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
        NSLog(@"✅ Page loaded successfully: %@", webView.URL);
        
        // Optional: Inject global JS callback handler here if needed
    }
    
    - (void)webView:(WKWebView *)webView
    didFailNavigation:(WKNavigation *)navigation
           withError:(NSError *)error {
        NSLog(@"❌ Navigation failed: %@", error.localizedDescription);
    }
    
    - (void)webView:(WKWebView *)webView
    didFailProvisionalNavigation:(WKNavigation *)navigation
           withError:(NSError *)error {
        NSLog(@"❌ Page loading failed (DNS / SSL / Network): %@", error.localizedDescription);
        NSLog(@"Error details: %@", error);
    }
    
    - (void)dealloc {
        // Remove JS handler to prevent memory leaks
        [self.configuration.userContentController removeScriptMessageHandlerForName:@"NativeBridge"];
        [[NSNotificationCenter defaultCenter] removeObserver:self
                                                        name:kNotificationNameFPSPaymentH5CallAppResult
                                                      object:nil];
    }
    
    @end
    ```
  </Tab>
  <Tab title="Tab">
    ```swift
    @preconcurrency import WebKit
    import UIKit
    
    class FPSWKWebView: WKWebView {
        
        private var callbackId: String?
        
        init(frame: CGRect) {
            let userContentController = WKUserContentController()
            
            // 2. Configure WebView
            let configuration = WKWebViewConfiguration()
            configuration.userContentController = userContentController
            configuration.processPool = WKProcessPool()
            configuration.preferences.javaScriptCanOpenWindowsAutomatically = true
            
            super.init(frame: frame, configuration: configuration)
            userContentController.add(self, name: "NativeBridge") // Register JS bridge named "NativeBridge"
            
            navigationDelegate = self
            uiDelegate = self
            
            if #available(iOS 16.4, *) {
                isInspectable = true
            }
            
            NotificationCenter.default.addObserver(
                self,
                selector: #selector(fpsPaymentResult(_:)),
                name: Notification.Name("kNotificationNameFPSPaymentH5CallAppResult"),
                object: nil
            )
        }
        
        required init?(coder: NSCoder) {
            fatalError("init(coder:) has not been implemented")
        }
    
        // Handle payment result notification (from FPSAppCallAppTool)
        @objc private func fpsPaymentResult(_ notification: Notification) {
            guard let paramsDict = notification.object as? [String: Any],
                  let isSuccess = paramsDict["is_successful"] as? String else {
                return
            }
            
            // Construct result for JS
            let result: [String: String] = Int(isSuccess) == 0 ? [
                "code": "3000",
                "respmsg": "Failed"
            ] : [
                "code": "0000",
                "respmsg": "Success"
            ]
            
            // Serialize to JSON string
            guard let jsonData = try? JSONSerialization.data(withJSONObject: result, options: []),
                  let jsonString = String(data: jsonData, encoding: .utf8) else {
                print("❌ Failed to serialize JSON")
                return
            }
            
            // Construct JS call: window.handleNativeCallback('callbackId', '{"code":"0000",...}')
            guard let callbackId = callbackId else {
                print("⚠️ No valid callbackId, cannot call back to JS")
                return
            }
            
            let jsCode = "window.handleNativeCallback('\(callbackId)', '\(jsonString)');"
            
            // Execute JS
            evaluateJavaScript(jsCode) { result, error in
                if let error = error {
                    print("❌ Failed to inject JS: \(error)")
                } else {
                    print("✅ JS callback injection successful")
                }
            }
        }
    
        // Get top-most root ViewController
        func getCurrentWindowRootVC() -> UIViewController? {
            for scene in UIApplication.shared.connectedScenes {
                guard let windowScene = scene as? UIWindowScene else { continue }
                for window in windowScene.windows where window.isKeyWindow {
                    return window.rootViewController
                }
            }
            return nil
        }
    
        deinit {
            NotificationCenter.default.removeObserver(self)
        }
    }
    
    extension FPSWKWebView: WKScriptMessageHandler {
        // Handle message from JS
        func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
            guard message.name == "NativeBridge",
                  let body = message.body as? [String: Any] else {
                return
            }
    
            guard let action = body["action"] as? String else { return }
    
            if action == "FPSH5CallApp" {
                // Extract parameters
                guard let params = body["params"] as? [String: Any],
                      let paymentRequestURL = params["paymentRequestURL"] as? String else {
                    return
                }
                
                // Save callback ID (passed from JS)
                callbackId = body["callbackId"] as? String
                
                // Trigger payment extension
                FPSAppCallAppTool.shared.invokePaymentExtension(paymentRequestURL: paymentRequestURL)
            }
        }
    }
    
    extension FPSWKWebView: WKUIDelegate {
        func webView(
            _ webView: WKWebView,
            runJavaScriptAlertPanelWithMessage message: String,
            initiatedByFrame frame: WKFrameInfo,
            completionHandler: @escaping () -> Void
        ) {
            let alert = UIAlertController(title: "Notice", message: message, preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "OK", style: .default) { _ in
                completionHandler() // ⚠️ Must be called, or JS will be blocked
            })
    
            // Show alert on the topmost ViewController
            if let topVC = getCurrentWindowRootVC() {
                topVC.present(alert, animated: true)
            } else {
                // Fallback: avoid JS lock-up
                completionHandler()
            }
        }
    }
    
    extension FPSWKWebView: WKNavigationDelegate {
        func webView(_ webView: WKWebView, didStartProvisionalNavigation navigation: WKNavigation!) {
            print("Started loading page...")
        }
    
        func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
            print("✅ Page loaded: \(String(describing: webView.url))")
        }
    
        func webView(_ webView: WKWebView, didFail navigation: WKNavigation!, withError error: any Error) {
            print("❌ Navigation failed: \(error.localizedDescription)")
        }
    
        func webView(_ webView: WKWebView, didFailProvisionalNavigation navigation: WKNavigation!, withError error: any Error) {
            print("❌ Page load failed (DNS, SSL, or network issue): \(error.localizedDescription)")
            print("Error details: \(error)")
        }
    }
    ```
  </Tab>
  <Tab title="Tab">
    ```javascript
    function isAndroidPhone() {
        const ua = navigator.userAgent.toLowerCase();
        return /android/i.test(ua);
    }
    
    // Call native function in iOS using WKWebView bridge
    const iOSCallNative = (action: string, params: any = {}) => {
        return new Promise((resolve, reject) => {
            const callbackId = 'cb_' + Date.now() + '_' + Date.now();
            if (window?.webkit?.messageHandlers?.NativeBridge) {
                // Register callback
                const callbacks = window?.nativeCallbacks;
                if (callbacks) {
                    callbacks[callbackId] = resolve;
                }
                window?.webkit?.messageHandlers.NativeBridge.postMessage({
                    action: action,
                    params: params,
                    callbackId: callbackId
                });
            } else {
                reject(new Error('Not in WKWebView'));
            }
        });
    }
    
    // Call native function in Android using AndroidInterface bridge
    const androidCallNative = (action: string, params: any = {}) => {
        return new Promise((resolve, reject) => {
            const callbackId = 'cb_' + Date.now() + '_' + Date.now();
            // Check if in Android WebView
            if (window?.AndroidInterface && typeof window?.AndroidInterface.handleMessage === 'function') {
                // Call Android native method
                const callbacks = window?.nativeCallbacks;
                if (callbacks) {
                    callbacks[callbackId] = resolve;
                }
                window?.AndroidInterface.handleMessage(JSON.stringify({
                    action: action,
                    params: params,
                    callbackId: callbackId
                }));
            } else {
                reject(new Error('Not in WKWebView'));
            }
        });
    }
    
    // Unified call entry point for iOS and Android
    export const callNative = (action: string, params: any = {}) => {
        // Initialize global callback handler if not already set
        if (typeof window.handleNativeCallback !== 'function') {
            window.nativeCallbacks = {};
            window.handleNativeCallback = function(callbackId: string, result: any) {
                const callbackMap = window?.nativeCallbacks;
                if (callbackMap) {
                    const callback = callbackMap[callbackId];
                    if (callback && typeof callback === 'function') {
                        callback(result);
                        delete callbackMap[callbackId];
                    }
                }
            };
        }
    
        // Route to platform-specific call
        if (isAndroidPhone()) {
            return androidCallNative(action, params);
        } else {
            return iOSCallNative(action, params);
        }
    }
    ```
  </Tab>
</Tabs>

---

---

## Error Handling & Fallbacks

### General Result Codes

| Code              | Meaning                                                        |
| ----------------- | -------------------------------------------------------------- |
| `RESULT_OK`       | Payment app confirmed successful initiation                    |
| `RESULT_CANCELED` | Payment cancelled or failed (user back, timeout, or app error) |

You should always verify payment result with your backend by checking the status of `out_trade_no`.

### What if the FPS app is not installed?

If no compatible FPS payment app is found, `startActivityForResult` or `UIActivityViewController` may not launch anything.

✅ Recommendation:

- Always provide a fallback method (e.g., fallback QR code, error message, retry option).

### Network or Callback Fails?

If the callback via JS or Universal Link fails:

- Inform the user payment is being verified
- Query the backend again after 3–5 seconds
- Show a loading screen or retry button

---

## Demo Downloads

| Platform | Flow Type                       | Demo                                                                                         |
| -------- | ------------------------------- | -------------------------------------------------------------------------------------------- |
| Android  | Native App-to-App               | [Download](@site/static/files/FPS%20h5%20app%20call%20app%20demo.zip)                        |
| Android  | H5-to-App via WebView           | [Download](@site/static/files/FPS%20h5%20app%20call%20app%20demo.zip)                        |
| iOS      | Native App-to-App (Objective-C) | [FPS ObjC Demo](https://img-int.qfapi.com/upstatic/20251120c100/FPSDemo/FPSElement.zip)      |
| iOS      | Native App-to-App (Swift)       | [FPS Swift Demo](https://img-int.qfapi.com/upstatic/20251120c100/FPSDemo/FPSElementDemo.zip) |
| iOS      | H5-to-App via JSBridge          | [JSBridge Demo](https://img-int.qfapi.com/upstatic/20251120c100/FPSDemo/fps-bridge.zip)      |