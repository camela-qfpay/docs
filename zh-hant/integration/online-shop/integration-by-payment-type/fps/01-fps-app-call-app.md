---
id: fps-app-call-app
title: FPS 應用程式互調
sidebar_label: FPS 應用程式互調
description: FPS 應用程式互調用支付集成指南 (App-to-App, H5-to-App 支援 Android 和 iOS)
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import Link from '@docusaurus/Link';

# FPS 應用程式互調用支付集成指南

本頁說明商戶 App 或 H5 如何透過 QFPay API 和 Intent/Universal Link 等方式喚起銀行支付應用，並完成 FPS App-to-App 的支付流程。支援 Android 與 iOS 系統，並提供完整代碼範例與 Demo 下載連結。

---

## 1. 取得支付參數

**API 端口**：`/trade/v1/payment`  
**請求方法**：`POST`  
**支付編碼**：`800210`

### 請求參數

| 參數名稱 | 是否必填 | 類型 | 描述 |
|----------|----------|------|------|
| 通用請求參數 | 是 | 依照平台設定 | 詳見 [通用請求參數](/docs/api-reference/request-format#public-payment-request-parameters) |

:::warning 僅適用於 HSBC FPS 商戶

如你的 FPS 為 **HSBC 直連模式**，商戶必須申請一張**獨立的 SSL 憑證（證書）**，且該憑證之**網域名稱必須與商戶主體一致**。

此設定為以下用途所必須：
- iOS Universal Link 回調驗證
- Android HTTPS 回調／重導驗證
- HSBC 生產環境上線及安全審核

📄 詳細申請流程、CSR 產生方式及所需文件，請參閱：
[FPS e-Cert 申請說明文件](/docs/online-shop/integration-by-payment-type/fps/fps-ecert-setup)

:::

### 回應參數

| 參數名稱 | 類型 | 描述 |
|----------|------|------|
| 通用回應參數 | 依照平台設定 | 詳見 [通用回應參數](/docs/api-reference/response-format#field-description) |
| `pay_params` | String(128) | 用於拉起銀行 FPS 支付應用的 URL，例如：`https://fps.xxx/xxx` |

---

## 2. Android FPS 支付流程

### 2.1 原生 App-to-App 流程

1. 商戶首先通過網站 API 獲取 pay\_params URL
2. 通過 Android `Intent` 方式啟動 FPS 支付 App
3. 加設 Action: `hk.com.hkicl` ，Key: `url` ，Value: 支付 URL
4. 使用 `startActivityForResult` 連動支付 App
5. 在 `onActivityResult` 中根據 requestCode 接收支付結果
6. 商戶 App 需根據自己的訂單狀態查詢系統確認結果

<Link href="/img/fps_app-call-app_chin.jpeg" target="_blank">![FPS App-call-app process-flow](@site/static/img/fps_app-call-app_chin.jpeg)</Link>

> 參考代碼: Android Java 啟動範例
```java
///支付發起請求代碼
int payRequestCode = 100;

//支付鏈接參數
String payUrl = "https://fps.qfpay.global/trade/v1/urltranslate/PAYCORE_SHORT_URL_202511075370911194";

//封裝支付選擇應用（銀行 app） Intent
Intent intent = new Intent("hk.com.hkicl");

//Intent添加參數“url”，值為支付鏈接參數
intent.putExtra("url", payUrl);

//啟動app 選擇器，選擇支付銀行
startActivityForResult(intent, payRequestCode);


//獲取支付返回结果
    @Override
    public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data, @NonNull ComponentCaller caller) {
        super.onActivityResult(requestCode, resultCode, data, caller);
        if (requestCode == payRequestCode) {
            //支付结果
            if (resultCode == RESULT_OK) { // 支付成功

            } else if (resultCode == RESULT_CANCELED) { // 支付失敗

            }
        }
    }
```

### 2.2 Android H5-to-App 支付流程

H5 頁面可透過 WebView 調用 FPS App 完成支付，需實作以下幾步：

1. **WebView 設定**
   - 啟用 JavaScript：`webView.getSettings().setJavaScriptEnabled(true)`
   - 綁定 JS 與 Android 溝通橋接：`addJavascriptInterface(new JsBridge(), "AndroidBridge")`

2. **H5 觸發支付流程**
   - 當用戶點擊付款，H5 使用 JS 方法調用 Android：  
     `AndroidBridge.handleMessage(JSON.stringify({ url: 'https://fps.qfapi.com/xxx' }))`

3. **原生 App 處理 JS 傳入的支付參數**
   - 接收後組裝 `Intent` 發送支付請求：
     ```java
     Intent intent = new Intent("hk.com.hkicl");
     intent.putExtra("url", payUrl);
     startActivityForResult(intent, REQUEST_CODE);
     ```

4. **接收支付結果並回傳給 H5**
   - 在 `onActivityResult()` 中將結果用 JS 傳回 WebView：
     ```java
     String resultJson = "{code: '0000', msg: 'success'}";
     webView.evaluateJavascript("javascript:window.handleNativeCallback(" + resultJson + ")", null);
     ```

5. **商戶應以自身訂單查詢為準**確認最終支付狀態。

> 參考代碼: Android WebView H5 啟動範例

```java
public class Web2AppCallPayActivity extends Activity {

    /**
     * 商户 App 加載的 H5連結
     */
    private static final String WEB_PAY_LINK = "https://img-int.qfapi.com/upstatic/20251119/fpsH5CallApp/index.html";


    /**
     * 商户 App 内部 Web 組建
     */
    private WebView webView;

    /**
     * Web 端發起支付時出入的 callBackId
     */
    private String callBackId;

    /**
     * 支付發起請求代碼
     */
    int payRequestCode = 100;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState, @Nullable PersistableBundle persistentState) {
        super.onCreate(savedInstanceState, persistentState);
        setContentView(R.layout.more_view);

        webView = findViewById(R.id.web_pay);
        //使 Web 组件支持 Js 交互
        webView.getSettings().setJavaScriptEnabled(true);
        //Web 端需要指定交互的接收方法，已經交互操作 Name，
        webView.addJavascriptInterface(this, "AndroidInterface");
        //Web 加載 H5連結
        webView.loadUrl(WEB_PAY_LINK);
    }


    @Override
    public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data, @NonNull ComponentCaller caller) {
        super.onActivityResult(requestCode, resultCode, data, caller);
        if (requestCode == payRequestCode) { // 此次返回结果为拉起支付的操作

            EvaluateBean evaluateBean = new EvaluateBean(); //支付结果需要封装為對象，转换为 json 字符串回傳至H5端
            evaluateBean.setCode(resultCode); //设置支付结果代码
            if (resultCode == RESULT_OK) { //支付成功
                evaluateBean.setRespmsg("Pay Success");
            } else if (resultCode == RESULT_CANCELED) {  //支付取消
                evaluateBean.setRespmsg("Pay Cancel");
            }
            String evaluateJson = new Gson().toJson(evaluateBean);//支付结果轉換为 json 字符串
            //將支付结果回傳给 H5端， 方法名固定为javascript:window.handleNativeCallback();
            //需要傳入兩個參數
            //一、H5端發起支付時傳入的 callbackId 參數
            //二、支付結果包括 code 和 文字描述，將兩個參數封装成 Bean，然後轉換為 Json字符串傳遞回 H5端
            webView.evaluateJavascript("javascript:window.handleNativeCallback('" + callBackId + ")" + ",(" + evaluateJson + "')", null);
        }
    }


    /**
     * H5端通过此方法来通知商户 App 操作， 方法名需要添加注解@JavascriptInterface
     * 方法名必須為handleMessage
     *
     * @param paramFromWebPay H5端將支付参数傳遞過來
     */
    @JavascriptInterface
    public void handleMessage(String paramFromWebPay) {
        if (TextUtils.isEmpty(paramFromWebPay)) return;
        WebParamsBean webParamsBean = new Gson().fromJson(paramFromWebPay, WebParamsBean.class);

        //Web 端發起支付時出入的 callBackId
        callBackId = webParamsBean.getCallbackId();

        //獲取H5端返回的支付所需要的参数 Url
        String paymentRequestURL = webParamsBean.getParams().getPaymentRequestURL();

        //商户 App 發起調用本地银行 App 操作
        launchBankPay(paymentRequestURL);
    }

    /**
     * 商户 App 發起調用本地银行 App 操作
     *
     * @param paymentRequestURL H5端將支付參數傳遞過來
     */
    private void launchBankPay(String paymentRequestURL) {
        Intent intent = new Intent("hk.com.hkicl");
        intent.putExtra("url", paymentRequestURL);
        startActivityForResult(intent, payRequestCode);
    }
}
```

---

## 3. iOS App-to-App 支付流程

### 3.1 原生 App 啟動 FPS 支付 App

1. 商戶首先調用支付參數接口，獲取支付參數 URL（例如：`https://fps.qfpay.global/trade/v1/urltranslate/PAYCORE_SHORT_URL_202511075370911194`）
2. 商戶 App 使用 iOS App Extension 框架 + `UIActivityViewController` 調起支持的支付 App（如銀行 App）
3. 消費者在支付 App 完成付款後，透過回調參數 `callback`（即商戶 App 的 Universal Link）跳轉回商戶 App
4. 商戶 App 必須根據自身的訂單查詢系統確認最終支付結果

#### 集成步驟：

* 創建一個 `NSExtensionItem`
* 將 `pay_params` URL 和 `callback` 包裝進 `NSItemProvider`
* 設定 `NSItemProvider` 的 UTI 類型為 `hk.com.hkicl`
* 使用 `UIActivityViewController` 顯示選單，讓用戶選擇支付 App

🔗 Apple 官方文件：
[Universal Links 官方說明](https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app)
[Associated Domains 配置方式](https://developer.apple.com/documentation/xcode/supporting-associated-domains)

<Tabs>
<TabItem value='objectivec' label='Objective-C'>

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
- (void)fpsPaymentResult:(NSDictionary *) result {
    NSLog(@"%@", result);
    [[NSNotificationCenter defaultCenter] postNotificationName:kNotificationNameFPSPaymentH5CallAppResult object:result];
}
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
    // 1. 獲取支付参数（從商户服務器獲取如https://fps.qfpay.global/trade/v1/urltranslate/PAYCORE_SHORT_URL_202511075370911194）
    // Demo測試可在輸入框中输入響應的支付參數
    @try {
        if (!paymentRequestURL || paymentRequestURL.length <= 0) {
            [self showAlert];
            return;
        }
    } @catch (NSException *exception) {
        return;
    }
    // 商户的Universal Link（用於支付完成後回調商户App）可參閱Apple开发文檔 https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app
    NSString *callbackURL = @"https://img-int.qfapi.com/trade/123"; //
    
    // 2. 封装數據為字典（包含URL和callback）
    NSDictionary *paymentPayload = @{
        @"URL": paymentRequestURL,
        @"callback": callbackURL
    };
        
    NSItemProvider *itemProvider = [[NSItemProvider alloc]
            initWithItem:paymentPayload
            typeIdentifier:@"hk.com.hkicl"];
    
    // 4. 創建NSExtensionItem并添加附件
    NSExtensionItem *extensionItem = [[NSExtensionItem alloc] init];
    extensionItem.attachments = @[itemProvider];
    
    // 5. 初始化UIActivityViewController（系统應用選擇器）
    UIActivityViewController *activityVC = [[UIActivityViewController alloc]
                                            initWithActivityItems:@[extensionItem]
                                            applicationActivities:nil];
    
    // 6. 適配iPad（必須指定彈出位置，否則崩潰）
    if ([[UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPad) {
        activityVC.popoverPresentationController.sourceView = [self getCurrentWindowRootVC].view;
        activityVC.popoverPresentationController.sourceRect = [self getCurrentWindowRootVC].view.frame; // 從按鈕位置彈出
        activityVC.popoverPresentationController.permittedArrowDirections = UIPopoverArrowDirectionUp;
    }
    
    // 7. 處理擴展返回的臨時结果（非最終支付結果，僅作參考）
    activityVC.completionWithItemsHandler = ^(UIActivityType  _Nullable activityType,
                                              BOOL completed,
                                              NSArray * _Nullable returnedItems,
                                              NSError * _Nullable error) {
        if (completed) {
            NSLog(@"用户选择了擴展：%@，處理完成", activityType);
            // 解析擴展返回的臨時數據（如支付已發起）
          
        } else if (error) {
            NSLog(@"擴展調用失敗：%@", error.localizedDescription);
        } else {
            NSLog(@"用户取消了操作");
        }
    };
    // 8. 展示應用選擇器
    [[self getCurrentWindowRootVC] presentViewController:activityVC animated:YES completion:nil];
}
- (void)showAlert{
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"没有輸入參數"
                                                                   message:@"請先輸入FPS支付參數"
                                                            preferredStyle:UIAlertControllerStyleAlert];
    
    [alert addAction:[UIAlertAction actionWithTitle:@"确定"
                                               style:UIAlertActionStyleCancel
                                             handler:nil]];
    [[self getCurrentWindowRootVC] presentViewController:alert animated:YES completion:nil];
}
#pragma mark - 解析連結中的查詢參數
- (void)parseQueryParamsFromCallbackURL:(NSURL *)url {
    NSURLComponents *components = [NSURLComponents componentsWithURL:url resolvingAgainstBaseURL:NO];
    NSArray<NSURLQueryItem *> *queryItems = components.queryItems;
    
    NSMutableDictionary *params = [NSMutableDictionary dictionary];
    for (NSURLQueryItem *item in queryItems) {
        if (item.value) {
            params[item.name] = item.value;
        }
    }
    [self fpsPaymentResult: [params copy]];
}

@end
```
</TabItem>

<TabItem value='swift' label='Swift'>

```swift
import Foundation
import UIKit
class FPSAppCallAppTool:NSObject{
    static let shared = FPSAppCallAppTool()
    
    func fpsPaymentResult(_ result: [String: Any]) {
        print(result)
        NotificationCenter.default.post(name: NSNotification.Name.init("kNotificationNameFPSPaymentH5CallAppResult"), object: result)
    }
    
    func invokePaymentExtension(paymentRequestURL: String?) {
        // 1. 獲取支付參數（从輸入框中讀取）
        guard let paymentRequestURL = paymentRequestURL, !paymentRequestURL.isEmpty else {
                showAlert()
                return
        }

        // 商户的Universal Link（用於支付完成後回調商户App）可參閱Apple開發文檔 https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app
        let callbackURL = "https://img-int.qfapi.com/trade/123"; //

        // 2. 封裝數據为字典
        let paymentPayload: [String: String] = [
            "URL": paymentRequestURL,
            "callback": callbackURL
        ]

        // 3. 創建 NSItemProvider（注意：需使用 NSCoding 兼容類型，Dictionary 需轉為 NSDictionary）
        let itemProvider = NSItemProvider(
            item: paymentPayload as NSDictionary,
            typeIdentifier: "hk.com.hkicl"
        )

        // 4. 創建 NSExtensionItem 并添加附件
        let extensionItem = NSExtensionItem()
        extensionItem.attachments = [itemProvider]

        // 5. 初始化 UIActivityViewController
        let activityVC = UIActivityViewController(
            activityItems: [extensionItem],
            applicationActivities: nil
        )

        // 6. 適配 iPad（必須設置 popover）
        if UIDevice.current.userInterfaceIdiom == .pad,
            let rootVC = getCurrentWindowRootVC() {
            activityVC.popoverPresentationController?.sourceView = rootVC.view
            activityVC.popoverPresentationController?.sourceRect = rootVC.view.bounds
            activityVC.popoverPresentationController?.permittedArrowDirections = .up
        }

        // 7. 設置完成回調
        activityVC.completionWithItemsHandler = { [weak self] (activityType, completed, returnedItems, error) in
            if completed {
                print("用户选择了擴展：\(activityType?.rawValue ?? "unknown")，處理完成")
                // 可在此解析 returnedItems（如果擴展返回了數據）
            } else if let error = error {
                print("擴展调用失败：\(error.localizedDescription)")
            } else {
                print("用户取消了操作")
            }
        }

        // 8. 展示應用選擇器
        if let rootVC = getCurrentWindowRootVC() {
            rootVC.present(activityVC, animated: true, completion: nil)
        } else {
            showAlert(message: "無法獲取當前界面，無法啟動支付")
        }
    }
    
    private func getCurrentWindowRootVC() -> UIViewController? {
        for scene in UIApplication.shared.connectedScenes {
            guard let windowScene = scene as? UIWindowScene else { continue }
            for window in windowScene.windows where window.isKeyWindow {
                return window.rootViewController
            }
        }
        return nil
    }
    
    private func showAlert(title: String = "没有輸入參數",
                               message: String = "請先輸入FPS支付參數") {
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "確定", style: .cancel))

        if let rootVC = getCurrentWindowRootVC() {
            rootVC.present(alert, animated: true)
        }
    }
    
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
</TabItem>
</Tabs>

### 3.2 iOS H5-to-App 雙向通信支付流程

H5 調用原生 App 的核心在於建立雙向通信機制，完成支付參數的傳遞與結果回傳。

1. H5 調用後端接口獲得支付參數
2. 使用 JsBridge 方法將支付參數傳遞給原生 App
3. App 接收到後呼叫 FPS App 進行支付（同 3.1 流程）
4. 支付完成後 App 將結果透過 `evaluateJavaScript` 回傳給 H5 顯示
5. 使用 `WKWebView` + JSBridge 實現，推薦參考範例程式碼整合

<Tabs>

<TabItem value='objectivec' label='Objective-C WKWebview'>

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
    [userContentController addScriptMessageHandler:self name:@"NativeBridge"]; // 注册名為 "NativeBridge" 的通道
    
    WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
    // web内容處理池
    config.processPool = [[WKProcessPool alloc] init];
    config.userContentController = userContentController;
    // 在iOS上默認為NO，表示不能自動通過窗口打開
    config.preferences.javaScriptCanOpenWindowsAutomatically = YES;
    //    config.preferences.javaScriptEnabled = YES;
    [config.preferences setValue:@YES forKey:@"allowFileAccessFromFileURLs"];
    
    self = [super initWithFrame:frame configuration:config];
    if (self) {
        self.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
        self.navigationDelegate = self;
        self.UIDelegate = self;
        if (@available(iOS 16.4, *)) {
            self.inspectable = YES;
        }
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(fpsPaymentResult:) name:kNotificationNameFPSPaymentH5CallAppResult object:nil];
    }
    return self;
}
- (void)fpsPaymentResult:(NSNotification *)notification {
    NSLog(@"%@", notification.object);
    NSDictionary *params = nil;
    NSString *ret = notification.object[@"is_successful"];
    if ([ret isEqualToString: @"0"]) {
        params = @{
            @"code": @"3000",
            @"respmsg": @"Failed",
        };
    }else{
        params = @{
            @"code": @"0000",
            @"respmsg": @"Success",
        };
    }
    
    // 將 userInfo 轉為 JSON 字符串，再拼成 JS 函数調用
    NSError *error;
    NSData *jsonData = [NSJSONSerialization dataWithJSONObject:params options:0 error:&error];
    if (error) {
        NSLog(@"JSON 序列化失敗: %@", error);
        return;
    }
    NSString *jsonString = [[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];
    
    // 構造 JS 代碼
    NSString *jsCode = [NSString stringWithFormat:@"window.handleNativeCallback('%@', '%@');", self.callbackId, jsonString];
    
    // 主動調用 JS 回調
    [self evaluateJavaScript:jsCode completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        if (error) {
            NSLog(@"❌❌❌❌❌JS 注入失敗: %@", error);
        } else {
            NSLog(@"✅✅✅✅✅✅ JS 注入成功");
        }
    }];
}
#pragma mark - WKScriptMessageHandler
// 處理 JS 發來的消息
- (void)userContentController:(WKUserContentController *)userContentController
      didReceiveScriptMessage:(WKScriptMessage *)message {
    
    if ([message.name isEqualToString:@"NativeBridge"]) {
        NSDictionary *body = message.body;
        
        // 假設 JS 傳了 { action: "getUserInfo", callback: "callback_123" }
        NSString *action = body[@"action"];
        NSString *callbackId = body[@"callbackId"]; // 回調函數名
        if ([action isEqualToString:@"FPSH5CallApp"]) {
            // 模擬原生數據
            NSString *paymentRequestURL = body[@"params"][@"paymentRequestURL"];
            self.callbackId = callbackId;
            [[FPSAppCallAppTool shareInstance] invokePaymentExtension:paymentRequestURL];
        }
    }
}
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
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(nonnull NSString *)message initiatedByFrame:(nonnull WKFrameInfo *)frame completionHandler:(nonnull WK_SWIFT_UI_ACTOR void (^)(void))completionHandler{
    NSLog(@"%@", message);
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"提示"
                                                                       message:message
                                                                preferredStyle:UIAlertControllerStyleAlert];
    [alert addAction:[UIAlertAction actionWithTitle:@"確定"
                                               style:UIAlertActionStyleDefault
                                             handler:^(UIAlertAction * _Nonnull action) {
        completionHandler(); // 必须調用，否則 JS 會卡住
    }]];
    
    [[self getCurrentWindowRootVC] presentViewController:alert animated:YES completion:nil];
}
#pragma mark - WKNavigationDelegate
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(WKNavigation *)navigation {
    NSLog(@"監聽頁面開始加載...");
}

- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
    NSLog(@"✅ 頁面加載成功: %@", webView.URL);
    // 注入回調處理函數到 JS 全局作用域
//    NSString *setupJS = @"if (typeof window.handleNativeCallback !== 'function') {"
//        @"window.nativeCallbacks = {};"
//        @"window.handleNativeCallback = function(callbackId, result) {"
//            @"var callback = window.nativeCallbacks[callbackId];"
//            @"if (callback && typeof callback === 'function') {"
//                @"callback(result);"
//                @"delete window.nativeCallbacks[callbackId];"
//            @"}"
//        @"};"
//        @"void(0);"   // ← 關鍵！强制返回 undefined
//        @"}";
//    [self evaluateJavaScript:setupJS completionHandler:^(id _Nullable result, NSError * _Nullable error) {
//        if (error) {
//            NSLog(@"❌ JS 注入失敗: %@", error);
//        } else {
//            NSLog(@"✅ JS 注入成功");
//        }
//    }];
}

- (void)webView:(WKWebView *)webView didFailNavigation:(WKNavigation *)navigation withError:(NSError *)error {
    NSLog(@"❌ 導航失敗: %@", error.localizedDescription);
}

- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(WKNavigation *)navigation withError:(NSError *)error {
    NSLog(@"❌ 頁面加載失敗（如 DNS、SSL、網絡問題）: %@", error.localizedDescription);
    NSLog(@"錯誤詳情: %@", error);
}

- (void)dealloc {
    // 移除 handler，防止内存泄漏
    [self.configuration.userContentController removeScriptMessageHandlerForName:@"NativeBridge"];
    [[NSNotificationCenter defaultCenter] removeObserver:self name:kNotificationNameFPSPaymentH5CallAppResult object:nil];
}
@end
```

</TabItem>

<TabItem value='swift' label='Swift WKWebview'>

```swift

@preconcurrency import WebKit
import UIKit
class FPSWKWebView: WKWebView{
    
    private var callbackId: String?
    
    init(frame: CGRect) {
        let userContentController = WKUserContentController()
        
        // 2. 配置 WebView
        let configuration = WKWebViewConfiguration()
        configuration.userContentController = userContentController
        configuration.processPool = WKProcessPool()
        configuration.preferences.javaScriptCanOpenWindowsAutomatically = true
        
        super.init(frame: frame, configuration: configuration)
        userContentController.add(self, name: "NativeBridge")
        
        navigationDelegate = self
        uiDelegate = self
        
        if #available(iOS 16.4, *){
            isInspectable = true
        }
        NotificationCenter.default.addObserver(self, selector: #selector(fpsPaymentResult(_:)), name: Notification.Name.init(rawValue: "kNotificationNameFPSPaymentH5CallAppResult"), object: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    // 支付结果處理（接收来自 FPSAppCallAppTool 的通知）
    @objc private func fpsPaymentResult(_ notification: Notification) {
        guard let paramsDict = notification.object as? [String: Any],
              let isSuccess = paramsDict["is_successful"] as? String else {
            return
        }
        
        // 構造返回给 JS 的結果
        let result: [String: String] = Int(isSuccess) == 0 ? [
            "code": "3000",
            "respmsg": "Failed"
        ] : [
            "code": "0000",
            "respmsg": "Success"
        ]
        
        // 序列化为 JSON 字符串
        guard let jsonData = try? JSONSerialization.data(withJSONObject: result, options: []),
              let jsonString = String(data: jsonData, encoding: .utf8) else {
            print("❌ JSON 序列化失败")
            return
        }
        
        // 構造 JS 調用代碼：window.handleNativeCallback('callbackId', '{"code":"0000",...}')
        guard let callbackId = callbackId else {
            print("⚠️ 無有效的 callbackId，無法回調 JS")
            return
        }
        
        let jsCode = "window.handleNativeCallback('\(callbackId)', '\(jsonString)');"
        
        // 執行 JS
        evaluateJavaScript(jsCode) { result, error in
            if let error = error {
                print("❌ JS 注入失敗: \(error)")
            } else {
                print("✅ JS 回調注入成功")
            }
        }
    }
    
    // 獲取rootVC
    func getCurrentWindowRootVC() -> UIViewController? {
        for scene in UIApplication.shared.connectedScenes {
            guard let windowScene = scene as? UIWindowScene else { continue }
            for window in windowScene.windows where window.isKeyWindow {
                return window.rootViewController
            }
        }
        return nil
    }
    deinit{
        NotificationCenter.default.removeObserver(self)
    }
}
extension FPSWKWebView: WKScriptMessageHandler{
    func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
        guard message.name == "NativeBridge",
            let body = message.body as? [String: Any] else {
            return
        }
               
        guard let action = body["action"] as? String else { return }

        if action == "FPSH5CallApp" {
           // 提取參數
           guard let params = body["params"] as? [String: Any],
                 let paymentRequestURL = params["paymentRequestURL"] as? String else {
               return
           }
           
           // 保存回調 ID（由 JS 傳入）
           callbackId = body["callbackId"] as? String
           
           // 調起支付擴展
           FPSAppCallAppTool.shared.invokePaymentExtension(paymentRequestURL: paymentRequestURL)
        }
    }
}
extension FPSWKWebView: WKUIDelegate{
    func webView(
            _ webView: WKWebView,
            runJavaScriptAlertPanelWithMessage message: String,
            initiatedByFrame frame: WKFrameInfo,
            completionHandler: @escaping () -> Void
        ) {
            let alert = UIAlertController(title: "提示", message: message, preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "确定", style: .default) { _ in
                completionHandler() // ⚠️ 必須調用！否則 JS 阻塞
            })

            // 獲取當前頂層 ViewController 並彈出
            if let topVC = getCurrentWindowRootVC() {
                topVC.present(alert, animated: true)
            } else {
                // 安全兜底：避免 JS 卡死
                completionHandler()
            }
        }
}
extension FPSWKWebView: WKNavigationDelegate{
    func webView(_ webView: WKWebView, didStartProvisionalNavigation navigation: WKNavigation!) {
        print("監聽頁面開始加載...");
    }
    func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
        print("✅ 頁面加載成功: %@", webView.url as Any);
    }
    func webView(_ webView: WKWebView, didFail navigation: WKNavigation!, withError error: any Error) {
        print("❌ 導航失敗: %@", error.localizedDescription);
    }
    func webView(_ webView: WKWebView, didFailProvisionalNavigation navigation: WKNavigation!, withError error: any Error) {
        print("❌ 頁面加載失敗（如 DNS、SSL、網絡問題）: %@", error.localizedDescription);
        print("錯誤詳情: %@", error);
    }
}
```
</TabItem>

<TabItem value='jsbridge' label='JsBridge'>

```javascript
function isAndroidPhone() {
    const ua = navigator.userAgent.toLowerCase();
    return /android/i.test(ua);
}

const iOSCallNative = (action: string, params:any = {}) => {
    return new Promise((resolve, reject) => {
        const callbackId = 'cb_' + Date.now() + '_' + Date.now();
        if (window?.webkit?.messageHandlers?.NativeBridge) {
            // 註冊回調
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
const androidCallNative = (action: string, params:any = {}) => {
    // 檢查是否在Android WebView環境中
    return new Promise((resolve, reject) => {
        const callbackId = 'cb_' + Date.now() + '_' + Date.now();
        if (window?.AndroidInterface && typeof window?.AndroidInterface.handleMessage === 'function') {
            // 調用Android原生方法
            const callbacks = window?.nativeCallbacks;
            if (callbacks) {
                callbacks[callbackId] = resolve;
            }
            window?.AndroidInterface.handleMessage(JSON.stringify({
                action: action,
                params: params,
                callbackId: callbackId
            }));
        }else {
            reject(new Error('Not in WKWebView'));
        }
    })
}
export const callNative = (action: string, params:any = {}) => {
    if (typeof window.handleNativeCallback !== 'function') {
        window.nativeCallbacks = {};
        window.handleNativeCallback = function(callbackId: string, result: any) {
            const callbackMap = window?.nativeCallbacks;
            if (callbackMap) {
                let callback = callbackMap[callbackId];
                if (callback && typeof callback === 'function') {
                    callback(result);
                    delete callbackMap[callbackId];
                }
            }

        };
    }
    if (isAndroidPhone()) {
        return androidCallNative(action, params);
    }else {
        return iOSCallNative(action, params);
    }
}
```
</TabItem>
</Tabs>

---

## 下載範例 Demo

- Android 原生 App-to-App 示範：[FPS H5 Demo](@site/static/files/FPS%20h5%20app%20call%20app%20demo.zip)
- Android H5-to-App 示範：[FPS H5 Demo](@site/static/files/FPS%20h5%20app%20call%20app%20demo.zip)


- Objective-C 示範: [FPS Objective-C Demo](https://img-int.qfapi.com/upstatic/20251120c100/FPSDemo/FPSElement.zip)
- Swift 示範: [FPS Swift Demo](https://img-int.qfapi.com/upstatic/20251120c100/FPSDemo/FPSElementDemo.zip)
- JSBridge 示範: [fps-jsBridge Demo](https://img-int.qfapi.com/upstatic/20251120c100/FPSDemo/fps-bridge.zip)
