## Core wrapper (advanced integration)

The HiPay Enterprise SDK for iOS contains a layer referred to as the *core wrapper*, which is basically a helpful wrapper of the HiPay Enterprise platform's REST API. By using it, you won't have to send HTTP requests or deal with XML or JSON deserialization. The core wrapper will take care of this for you.

It exposes models and methods that allow you to easily send payment requests. In fact, the built-in payment screen itself relies on the core wrapper for performing order requests, retrieving transaction details, etc.

The sections below describe the main interfaces you can rely on to perform order requests, tokenize credit cards, retrieve transaction details, etc. The payment workflow implementation details and the user interface are up to you.

### Tokenizing a credit or debit card

To tokenize a credit card, you need to leverage the *Secure Vault* client, as below.

This step is mandatory in order to make payments with credit or debit cards, either for one-shot or recurring payments.

#### Objective-C
```objectivec
[[HPFSecureVaultClient sharedClient]
 generateTokenWithCardNumber:@"4111111111111111"
             cardExpiryMonth:@"12"
              cardExpiryYear:@"2020"
                  cardHolder:@"John Doe"
                securityCode:@"496"
                    multiUse:YES
        andCompletionHandler:^(HPFPaymentCardToken * _Nullable cardToken,
                               NSError * _Nullable error) {
     
     /* The cardToken object should be defined with your token
      * if the tokenization was completed successfully.
      * Otherwise, the error object will be defined */
     
 }];
```

### Tokenizing an encrypted Apple Pay token

You can tokenize the Apple Pay token using the Secure Vault client as well.

#### Objective-C
```objectivec

/* We assume you received a PKPayment object
 * from the Apple PassKit delegate method */
NSString *decodedString = [[NSString alloc] initWithData:payment.token.paymentData
												encoding:NSUTF8StringEncoding];

[[HPFSecureVaultClient sharedClient]
 generateTokenWithApplePayToken:decodedString
             privateKeyPassword:@"YOUR P12 CERTIFICATE PASSWORD"
        andCompletionHandler:^(HPFPaymentCardToken * _Nullable cardToken,
                               NSError * _Nullable error) {
     
     /* The cardToken object should be defined with your token
      * if the tokenization was completed successfully.
      * Otherwise, the error object will be defined. */
     
 }];
```

### Updating a token or retrieving information about a token

You can also update a token by using the *Secure Vault* method named `updatePaymentCardWithToken:requestID:setCardExpiryMonth:cardExpiryYear:cardHolder:completionHandler:`.

To get information about a token previously generated, use the Secure Vault method named `lookupPaymentCardWithToken:requestID:completionHandler:`.

### Requesting a new order (making payments)

You can make payments using the *Gateway Client* and its *requestNewOrder* method. This request will both create a new order based on the information you provided and process the payment simultaneously.

Therefore, you need to provide your order ID, the amount of the transaction, the payment product (Visa, MasterCard, PayPal, etc.) and the related payment method information (i.e., the token if it's a credit or debit card). You can also provide a lot of other optional information.

#### Objective-C
```objectivec
/* Create an order request which
 * contains information about your order */
HPFOrderRequest *request = [[HPFOrderRequest alloc] init];
request.amount = @(155.50);
request.currency = @"EUR";
request.orderId = @"TEST_9641952";
request.shortDescription = @"Outstanding shirt";
request.operation = HPFOrderRequestOperationSale;
    
/* Below, optional properties are defined as well.
 * Check the HPFPaymentPageRequest documentation
 * for the full list of parameters */
request.customer.country = @"FR";
request.customer.firstname = @"John";
request.customer.lastname = @"Doe";
request.customer.email = @"yourclient@domain.com";
    
/* Payment method info, in this case,
 * we re-use the token which has been
 * generated in the previous section */
request.paymentProductCode = HPFPaymentProductCodeVisa;
request.paymentMethod = [HPFCardTokenPaymentMethodRequest
    cardTokenPaymentMethodRequestWithToken:@"f39bfab2b6c96fa30dcc0e55aa3da4125a49ab03"
                                       eci:HPFECISecureECommerce
                   authenticationIndicator:HPFAuthenticationIndicatorBypass];
    
[[HPFGatewayClient sharedClient] requestNewOrder:request signature:signature
                           withCompletionHandler:^(HPFTransaction * _Nullable transaction,
                                                   NSError * _Nullable error) {
    /* Check the transaction object, particularly transaction.state.
     * Or check the error object if the request failed. */
}];
    
```

#### Swift
```Swift
/* Create an order request which
 * contains information about your order */
let request = HPFOrderRequest()
request.amount = 155.50;
request.currency = "EUR"
request.orderId = "TEST_859674"
request.shortDescription = "Outstanding shirt"
request.operation = HPFOrderRequestOperation.Sale
    
/* Below, optional properties are defined as well.
 * Check the HPFPaymentPageRequest documentation
 * for the full list of parameters */
request.customer.country = "FR"
request.customer.firstname = "John"
request.customer.lastname = "Doe"
request.customer.email = "yourclient@domain.com"
    
/* Payment method info, in this case,
 * we re-use the token which has been
 * generated in the previous section */
request.paymentProductCode = HPFPaymentProductCodeVisa
request.paymentMethod = HPFCardTokenPaymentMethodRequest(
    token: "f39bfab2b6c96fa30dcc0e55aa3da4125a49ab03",
    eci: HPFECI.HPFECISecureECommerce,
    authenticationIndicator: HPFAuthenticationIndicator.Bypass)
    
HPFGatewayClient.sharedClient()
    .requestNewOrder(request, signature: signature) { (
        transaction: HPFTransaction?,
        error: NSError?) -> Void in
    /* Check the transaction object, particularly transaction.state. 
     * Or check the error object if the request failed. */
}
```


### Implementation note 
The signature parameter is needed for security purposes.  
Please refer to the [Generating a server-side signature](#generating-a-server-side-signature)  section for details.

When requesting a new order, do not forget to check the state of the newly created transaction. Moreover, you may need to redirect your users to a web page if the *forwardUrl* property is defined. In this case, you need to define the redirect URL properties on your order request object: *acceptURL*, *declineURL*, etc. In order for the HiPay platform to properly redirect users to your app, we advise you to use redirect URLs based on [iOS app URL schemes][apple-scheme] (e.g.: `myapp://order-result`).

### Getting the payment products enabled on your account

In order to get the exact list of payment methods enabled on your account, you can leverage the `getPaymentProductsForRequest:withCompletionHandler:` method of the *Gateway Client*. You need to provide this endpoint with information about your order (by using  `HPFPaymentPageRequest` because the list of payment products may change depending on order-related information (i.e. the currency)). 

#### Objective-C
```objectivec
HPFPaymentPageRequest *request = [[HPFPaymentPageRequest alloc] init];
request.amount = @(155.50);
request.currency = @"EUR";
    
// We just want the payment card products
request.paymentProductCategoryList = @[HPFPaymentProductCategoryCodeCreditCard,
                                       HPFPaymentProductCategoryCodeDebitCard];
    
[[HPFGatewayClient sharedClient] getPaymentProductsForRequest:request
                                        withCompletionHandler:^(NSArray<HPFPaymentProduct *> * _Nonnull paymentProducts,
                                                                NSError * _Nullable error) {
    // Check the paymentProducts array
}];
```

#### Swift
```Swift
let request = HPFPaymentPageRequest();
request.amount = 155.50;
request.currency = "EUR";
    
// We just want the payment card products
request.paymentProductCategoryList = [
    HPFPaymentProductCategoryCodeCreditCard,
    HPFPaymentProductCategoryCodeDebitCard
]
    
HPFGatewayClient.sharedClient()
    .getPaymentProductsForRequest(request) { (
        paymentProducts: [HPFPaymentProduct],
        error: NSError?) -> Void in
    // Check the paymentProducts array
}
```

### Requesting information about a transaction (checking transaction state)

You can get the transactions related to an order or get information about a specific transaction by using the following methods: 

- `getTransactionWithReference:withCompletionHandler:`
- `getTransactionsWithOrderId:withCompletionHandler:`

Please find below an example with the transactions linked to the merchant order ID "TEST_89897":

#### Objective-C
```objectivec
[[HPFGatewayClient sharedClient] getTransactionsWithOrderId:@"TEST_89897" signature:signature
                                      withCompletionHandler:^(NSArray<HPFTransaction *> * _Nullable transactions,
                                                              NSError * _Nullable error) {
    // Check the transactions array
}];
```

#### Swift
```Swift
HPFGatewayClient.sharedClient()
    .getTransactionsWithOrderId("TEST_89897", signature: signature) { (
        transactions: [HPFTransaction]?,
        error: NSError?) -> Void in
    // Check the transactions array
}
```


### Implementation note 
The *signature* parameter is required for security purposes.  
Please refer to the [Generating a server-side signature](#generating-a-server-side-signature) section for details.


### Card storage feature

The card storage feature allows to register a `HPFPaymentCardToken` object in the iOS device *Keychain*, necessary to use the 1-click payment for your customers.

Since the card storage option is turned ON, you have access to these three `HPFPaymentCardTokenDatabase` class methods:

To get every tokens associated to a currency :  
`paymentCardTokensForCurrency:` 

To remove a specific token associated to a currency :  
`delete:forCurrency` 

To remove every tokens located in the iOS device keychain :  
`clearPaymentCardTokens`


## In-store physical payments with a mPOS

Before going through this part, make sure you have followed the specific installation section for using a mobile point of sale (mPOS) payment terminal.

In order to use the payment terminal, you need to leverage the `HPFPOSManager` which exposes several public methods mentioned below.

### Terminal connection

When your application is launched, use the `connect` method to allow the terminal to receive transactions. You may call the `disconnect` method if using the payment terminal is no longer relevant in the user flow. 

You can check the connection state by using the `connectionState` property. This can be useful for displaying a green light or red light to indicate to the user (basically the shopkeeper) that the connection with the terminal is working.

### Barcode reader

If the payment terminal has a built-in barcode reader, you can get the scanned codes by observing `HPFPOSBarCodeNotification` notifications. Please check the example below.

### Implementation example

#### Objective-C
```objectivec
#import "HPFPOSManager.h"

[...]

/* To start the connection, just call
 * this code at the appropriate time */
[[HPFPOSManager sharedManager] connect];

/* Once connect is called, do not assume 
 * the library has fully connected to 
 * the device after this call, but 
 * wait for the notifications. */
[[NSNotificationCenter defaultCenter] addObserver:self 
                                         selector:@selector(stateChangeNotification:)
                                             name:HPFPOSStateChangeNotification object:nil];
    
[[NSNotificationCenter defaultCenter] addObserver:self 
                                         selector:@selector(barCodeNotification:)
                                             name:HPFPOSBarCodeNotification object:nil];
                                             
[...]

// Notifies about the current connection state
- (void)stateChangeNotification:(NSNotification *)notification
{
    NSDictionary *userInfo = [notification userInfo];
    NSNumber *state = userInfo[HPFPOSConnectionStateKey];
    
    // Connection state
    HPFPOSConnectionState connectionState = [state unsignedIntegerValue];
}

// Notification sent when barcode is successfuly read
- (void)barCodeNotification:(NSNotification *)notification
{
    NSDictionary *userInfo = [notification userInfo];
    
    // Barcode
    NSString *barCode = userInfo[HPFPOSBarCodeKey];
    
    // Barcode type
    NSString *barCodeType = userInfo[HPFPOSBarCodeTypeKey];
}                                             
```


#### Swift
```swift
/* To start the connection, just call
 * this code at the appropriate time */
HPFPOSManager.shared().connect()

/* Once connect is called, do not assume 
 * the library has fully connected to 
 * the device after this call, but 
 * wait for the notifications. */

        
let nc = NotificationCenter.default
nc.addObserver(self,
               selector: #selector(ViewController.stateChangeNotification),
               name: NSNotification.Name.HPFPOSStateChange,
               object: nil)
        
nc.addObserver(self,
               selector: #selector(ViewController.barCodeNotification),
               name: NSNotification.Name.HPFPOSBarCode,
               object: nil)
                                             
[...]

// Notifies about the current connection state
@objc func stateChangeNotification(notification:Notification) -> Void {
        
    let userInfo = notification.userInfo        
    let state  = userInfo![HPFPOSConnectionStateKey] as? NSNumber        
        
    // Connection state
    let connectionState = state?.intValue;
}

// Notification sent when barcode is successfuly read
@objc func barCodeNotification(notification:Notification) -> Void {
        
    let userInfo = notification.userInfo        
    
    // Barcode
    let barCode  = userInfo![HPFPOSBarCodeKey] as? String    
    
    // Barcode type
    let barCodeType  = userInfo![HPFPOSBarCodeTypeKey] as? String 
}                                     
```

### Processing transactions

Once your terminal is connected (thanks to the `connect` method), you can initialize transactions on it. Transactions must be initialized through the [HiPay's omnichannel API](https://developer.hipay.com/getting-started/platform-hipay-enterprise/omnichannel/). 

The SDK only allows you to make the payment terminal connected. Transactions must be initialized server-side by using the API mentioned above.

For a better user experience, once the transaction has been initialized server-side, we recommend you to call the `wakeUp` method, which will force the payment terminal to get the newly initialized transaction. Otherwise, your shopkeeper will have to press a button on the payment terminal in order for it to get the pending transaction.

[apple-scheme]: https://developer.apple.com/library/ios/featuredarticles/iPhoneURLScheme_Reference/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007899
