# Adyen Checkout for iOS

[![CI Status](http://img.shields.io/travis/Adyen/adyen-checkout-ios.svg?style=flat)](https://travis-ci.org/Adyen/adyen-checkout-ios)
[![Version](https://img.shields.io/cocoapods/v/AdyenCheckout.svg?style=flat)](http://cocoapods.org/pods/AdyenCheckout)

## Installation
AdyenCheckout is available through [CocoaPods](http://cocoapods.org). To install, add the following line to your Podfile:

```ruby
pod "AdyenCheckout"
```

## Usage

To start using the library, provide your public key and token that you use to connect to Adyen services:

``` swift
Checkout.shared.publicKey = "YOUR_ADYEN_PUBLIC_KEY"
// OR
Checkout.shared.token = "YOUR_ADYEN_TOKEN"
```
Show the `CheckoutCardViewController` to checkout with card:

``` swift
// Create a payment request.
let request = CheckoutRequest()
request.amount = 1.99
request.currency = "EUR"
request.reference = "Merchant Reference 123"

// Create the Checkout view controller.
let vc = CheckoutCardViewController(checkoutRequest: request)
vc.delegate = self
vc.titleText = "My Company"

// Present the controller.
let nc = UINavigationController(rootViewController: vc)
self.presentViewController(nc, animated: true, completion: nil)
```

And implement the `CheckoutViewControllerDelegate` functions:

``` swift
func checkoutViewController(controller: CheckoutViewController, authorizedPayment payment: CheckoutPayment) {
    // Dismiss the view controller.
    controller.dismissViewControllerAnimated(true, completion: nil)

    // Send the payment to your server, which should validate it with Adyen.
    sendPayment(payment) { (psp, error) -> Void in
        if (error != nil) {
            let alert = UIAlertView(title: "Error", message: error!.localizedDescription, delegate: nil, cancelButtonTitle: "OK")
            alert.show()
        } else {
            let alert = UIAlertView(title: "Success!", message: "PSP: \(psp)", delegate: nil, cancelButtonTitle: "OK")
            alert.show()
        }
    }
}

func checkoutViewController(controller: CheckoutViewController, failedWithError error: NSError) {
    // Dismiss the view controller.
    controller.dismissViewControllerAnimated(true, completion: nil)

    // Show an error.
    let alert = UIAlertView(title: "Error", message: error.localizedDescription, delegate: nil, cancelButtonTitle: "OK")
    alert.show()
}
```

The `CheckoutPayment` contains `amount`, `currency`, `reference`, and `paymentData` (which is Base64-encrypted data).


## Advanced usage

Take power with the advance library usage. Display the card fields in your view by adding a separate `CardPaymentField`, `UIView` class.

``` swift
let paymentFieldView = CardPaymentField()
self.view.addSubview(paymentFieldView)
```

Ensure that you have a valid public key for encryption, or get it with a token:

``` swift
if (Checkout.shared.publicKey == nil) {
    Checkout.shared.fetchPublickKey({ (publicKey, error) -> Void in
        if (error != nil) {
            // Handle an error.    
        }
    })
}
```

Encrypt the obtained data:

``` swift
if (paymentFieldView.valid) {
    let card = paymentFieldView.paymentData()
    do {
        let paymentData = try card.serialize()
        let payment = CheckoutPayment(request: self.request, encryptedData: paymentData)
        // Send a payment to Adyen.
    }
    catch let error as NSError {
        // Handle an error.
    }
}

```

## Fetching a Public Key

Assign your token to `Checkout.shared.token` and call `Checkout.shared.fetchPublicKey` to retrieve `publicKey` from Adyen.

``` swift
Checkout.shared.token = "8714279..."
Checkout.shared.fetchPublicKey { (publicKey, error) in
    //  Check for `error`.
    //  ...
    
    //  Use `publicKey` (e.g. store in a keychain for later access).
    //  ...
}
```

## Questions?

If you have any questions or suggestions, please contact your account manager or send your inquiry to support@adyen.com.

## License

This repository is open-source and available under the [MIT license](https://en.wikipedia.org/wiki/MIT_License). See the LICENSE file for more information.
