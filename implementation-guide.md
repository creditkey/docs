# Credit Key Merchant Implementation Guide

## Table of Contents
- [Synopsis](#synopsis)
- [Overview](#overview)
- [Return to Merchant after Credit Key Checkout](#return-to-merchant-after-credit-key-checkout)
    - [Return URL](#return-url)
    - [Cancel URL](#cancel-url)
    - [Actions Upon Return](#actions-upon-return)
- [Merchant Order ID](#merchant-order-id)
- [Authorize and Capture](#authorize-and-capture)

---

## Synopsis
This implementation guide should be reviewed before performing a merchant integration with Credit Key - either via SDK, or direct [API](https://app.swaggerhub.com/apis/creditkey/creditkey-merchant-api/1.1.0). It should familiarize you with the general requirements of a Credit Key merchant implementation.

This guide will refer to various [Credit Key API](https://app.swaggerhub.com/apis/creditkey/creditkey-merchant-api/1.1.0) calls.  A similar name is used for each call in each official Credit Key SDK, and directly via the [Credit Key API](https://app.swaggerhub.com/apis/creditkey/creditkey-merchant-api/1.1.0).  While this document refers to such API calls conceptually, you should refer to the documentation of the SDK you are using, or the [general API documentation](https://app.swaggerhub.com/apis/creditkey/creditkey-merchant-api/1.1.0), for specific syntax.

**There are some very important details contained in this document and we strongly recommend that you read and understand it, so your implementation will not require a lot of rework before launching.**  Feel free to reach out to Credit Key Support with any questions.

---

## Overview
[Credit Key](https://www.creditkey.com) checkout works similarly as services like [PayPal](https://www.paypal.com) in the sense that the user will be redirected to special checkout pages hosted on [creditkey.com](https://www.creditkey.com) to complete the checkout process.

When rendering your checkout page, you should always call ```is_displayed_in_checkout``` to determine whether or not to display Credit Key as a payment option.

When the user selects Credit Key as a payment option on your checkout page, you will need to call ```begin_checkout```.  Using this method you will send information about the order to Credit Key, such as the items in the user's shopping cart, the total amount to be billed, the billing and shipping addresses specified by the user in checkout, and so forth.  This method will return a unique [creditkey.com](https://www.creditkey.com) URL which you should redirect the user's browser to, in order for them to complete the checkout process.

After successful checkout on Credit Key's site, Credit Key will redirect the user's browser back to a unique URL provided by you to ```begin_checkout```.  At this point you should call ```complete_checkout``` to validate that the payment was successful and complete the order.  Upon successful return from ```complete_checkout```, you should place the order in your system - then display your own order confirmation place to the user.

When the order ships, you should call ```confirm_order``` to notify Credit Key that the order has shipped.  If ```confirm_order``` isn't called for several days after completing checkout, Credit Key will automatically cancel the order in it's system and the payment will not be issued.

If the order is canceled before shipment, you can call ```cancel_order``` to cancel the payment.  To issue a full or partial refund, use ```refund```.

---

## Return to Merchant after Credit Key Checkout
You will need to implement at least one, possibly two, endpoints or controller actions on your system to receive users returning from Credit Key checkout.  These URL's are provided to Credit Key each time a user selects the option to check out with Credit Key, when calling ```begin_checkout```.  They can be unique user-specific URL's.

If the Cancel URL or Return URL you provide to Credit Key include the string ```%CKKEY%```, then upon redirect this string will be replaced with the Credit Key Order ID.

### Return URL
The Return URL will be a URL on your system that Credit Key redirects the user's browser to after successful checkout.  When the user returns to this URL, you should validate the successful payment with Credit Key, complete the order in your system, and then display your order confirmation page.  Credit Key will not redirect a user to this URL if they have not successfully completed Credit Key checkout.

We recommend creating a session-specific URL for each request that contains identifying information about the session, such as the primary key in your system used to refer to the user's checkout session.  This way you will easily be able to line up the Credit Key order with the user's shopping cart session.  However, if you track checkout sessions with cookies, a general URL might work in your scenario.

### Cancel URL
Credit Key will redirect users to the Cancel URL when checkout was not completed successfully - such as when the user canceled the Credit Key checkout session, or if the user was not able to be approved for a loan.  In many cases, you can simply provide the URL to your checkout page for the Cancel URL.  But if you want to take another action besides going back to the checkout page, or perform tracking, you can redirect elsewhere.

### Actions Upon Return

In the endpoint you setup to handle the [Return URL](#return-url), you should take the following actions:

1. Call ```complete_checkout```, passing the Credit Key Order ID provided in the URL by Credit Key.  This method should return ```true``` to indicate the payment is authorized and you can continue placing the order.  If ```false``` is returned, or an error/exception is thrown, you should return an error and you should not continue placing the order.
2. Place the order as a new order in your system as an order with an authorized payment.
3. Call ```update_order``` to provide Credit Key with your local merchant Order ID and Order Status.

---

## Merchant Order ID

At various times, you'll be required to send a ```merchant_order_id``` to Credit Key.  This is the Order ID used in the merchant system, and will be displayed to the customer.

Many e-commerce platforms represent the Order ID as both an internal primary key from the database, and a friendly Order ID displayed to the user. **You should always send the friendly Order ID displayed to the user.**  The ```merchant_order_id``` value is used to display important order information to an end-user within the Credit Key system, so it should be the same order ID that they are shown in email correspondence with the merchant, and on the Order Confirmation page.

---

## Authorize and Capture

Your e-commerce system should not be configured to auto-capture Credit Key orders.  Rather, orders should be authorized at first, and captured upon shipment.  Your capture code can then call ```confirm_order``` upon Capture.  Any changes to shopping cart contents and prices that occur before capture should be sent via ```update_order```.

Any order that is not captured/confirmed within 30 days will be automatically canceled.
Therefore, you must ship all orders within 30 days.  If this is not feasible for your
company, you should discuss it with your Credit Key support rep before launch to find a
solution.
