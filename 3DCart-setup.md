# Credit Key 3DCart Setup

## Order Confirmation Page

1. In the 3dcart Admin, navigate to Content, Extra Pages, and then Add a new page with the following values:

```
Page Name: Credit Key Order Confirmation
Sub Page of: New
Sorting: 0
Hide Link: Checked
```

2. For the newly created page, click the gear icon and then select Content and Site Content.

3. Click on Preview, which will launch a preview of the page in a new window.  Save the URL of the page launched for later.

4. Uncheck ```WYSIWYG Mode Off/On``` and enter the following into the field at the bottom of the screen.  Replace ```<public key>``` with the public key provided to you.

```
<script type="text/javascript" src="https://www.creditkey.com/app/merchant-asset/order-confirmation-iframe-loader.js?public_key=<public key>"></script>
```

5. Click Save.

## Setup Payment Method

1. In the 3DCart admin, navigate to "Settings", "Payment", then click on "Select Payment Methods."  In the "Offline Payment Methods" section, click the "Add New" button and enter the following options:

```
Payment Method: Custom
Country: United States
```

2. After adding the payment method, you should see a new offline payment method named "Custom".  Click on the pencil icon next to the name that's labeled "Caption", and set the name to "Credit Key."

3. Now click on "Exclude List."  In the end of the URL, you should see a query string like ```paymentid=<number>_ofl```.  The number you find here is the payment method ID.  Save this ID number for a future step.

## Edit Checkout Template

1. If you don't have one already, create a copy of ```checkout-singlepage.html``` or ```checkout-singlepage-v2.html``` into your local theme directory.

2. In the copied file, find the area where the Shopping Cart is displayed, starting with the line ```<!--START: SHOPPING_CART_TEMP-->```.  Insert the following code immediately on the next line.  Replace ```<order confirmation page URL>``` with the URL you saved in an earlier step.

```
<script type="text/javascript">
    window.creditKey = window.creditKey || {};
    window.creditKey.orderConfirmationUrl = '<order confirmation page URL>?orderKey=%CKKEY%';
</script>
```

3. Now, find ```<!--START: SHOPPING_CART_ITEM-->```, and put the following code immediately on the next line:

```
<script type="text/javascript">
    window.creditKey.cartItems = window.creditKey.cartItems || [];
    window.creditKey.cartItems.push({
        merchant_id: "[ITEM_CATALOGID]",
        name: "[ITEM_NAME]",
        quantity: "[ITEM_QUANTITY]",
        price: "[ITEM_PRICE]",
        sku: "[ITEM_ID]"
    });
</script>
```

4. Find the line ```id="divPaymentMethods"``` and insert the following code immediately after this line:

```
<div id="creditkey-payment-option" class="customGateway creditkey-payment dpm-provider hidden" >
    <h4>
        <label for="offline-[id]-credit">
            <p class="creditkey-description"><input onclick="javascript:checkoutSwitch(false);controlDivPayment('[id]-custom-credit');javascript:window.creditKey.beginCheckout();" id="offline-[id]-credit" name="payment" type="radio" value="offline-[id]-credit" [FIRSTOPTIONCHECKED] />
            <span>Credit Key</span> 0% for up to 30 days. Get a decision in seconds.</p>
        </label>
    </h4>
    <div class="clear"></div>
    <div id="divPaymentOption[id]-custom-credit" name="divPaymentOption" style="display:none;">
        <div class="payment-desc">
            <div class="creditkey-checkout-section">
                <div class="creditkey-logo-section">
                    <a href="javascript:document.getElementById('offline-[id]-credit').click();"><img src="https://www.creditkey.com/app/merchant-checkout-assets/CK-btn@2x.png" alt="Credit Key" /></a>
                </div>
            </div>
            <p id="creditkey-error-message" class="creditkey-error">There was an error while trying to check out with Credit Key. Please try again later.</p>
            <div class="clear"></div>
        </div>
    </div>
    <div class="clear"></div>
</div>
<style>
    .creditkey-payment.hidden {
        display: none;
    }
    .creditkey-checkout-section {
        display: flex;
        flex-flow: row;
        align-items: center;
        justify-content: flex-start;
    }
    .creditkey-logo-section {
        flex-grow: 1;
    }
    .creditkey-logo-section img {
        width: 242px;
    }
    p.creditkey-description span{
        font-weight: 700;
        color: #4a4a4a;
        font-size: 16px;
    }
    p.creditkey-description {
        color: #19AD58;
        font-size: 14px;
        font-weight: bold;
        margin:0;
    }
    .creditkey-radio {
        margin-right: 5px;
    }
    .creditkey-error {
        display: none;
        color: #fd3740;
        font-size: 14px;
        font-weight: bold;
    }		
    .creditkey-error.show {
        display: block;
    }
    .creditkey-modal {
        overflow: auto;
    }
</style>

```

5. Replace [id] with payment id number from the custom gateway id. 


6. Find the line ```<!--START: CUSTOM-->```.  In the ```div``` containing the HTML for a payment method (possibly containing the CSS class ```customGateway```), add an ```id``` attribute of ```offline-payment-[id]``` like so:

```
<div id="offline-payment-[id]" class="customGateway">
```

7. Find the line by ```<!--START: DISPLAY_PROMOS-->``` and insert the following immediately afterward:
```
<script>
    window.creditKey = window.creditKey || {};
    window.creditKey.promos = window.creditKey.promos || {};
    window.creditKey.promos['promo_[id]'] = {
        PromotionName: '[promotion_name]',
        Coupon: '[coupon]',
        DiscountAmount: parseFloat('[discount_amount]'.replace(/^\$/, '').replace(/\,/g, ''))
    };
</script>
```

8. Insert the following code into the very bottom of the file - replacing ```<public key>``` with the public key given to you by Credit Key, and replacing the ```<payment method ID>``` with the payment method ID you saved when you created the payment method in steps 1 through 3.

```
<script type="text/javascript" src="https://www.creditkey.com/app/merchant-asset/3dcart.js?public_key=<public key>&payment_method_id=<payment method ID>"></script>
<style>
    #offline-payment-<payment method ID> {
        display: none;
    }
</style>
```

9. Find ```<!-- START: GIFTCERTS -->``` and place the following immediately BEFORE it:

```
<script>
    window.creditKey = window.creditKey || {};
    window.creditKey.charges = {
        total: '[SUBTOTAL]',
        shipping: '[SHIPPING]',
        taxes: '[TAX]',
        discount_amount: '[DISCOUNT]',
        grand_total: '[TOTAL]'
    };
</script>
```

## Set up Web Hooks

1. In the 3DCart admin, navigate to "Modules", and type "webhooks" into the search box.  Click the "Settings" button.

2. Click "Add Webhook" and enter the following values, replacing ```<public_key>``` and ```<shared_secret>``` with the values provided to you by Credit Key staff.  Then click "Save".

```
Webhook Name: Credit Key New
Webhook URL: https://www.creditkey.com/app/3dcart/order_update?public_key=<public_key>&shared_secret=<shared_secret>
Format: JSON
Event: Order New
```

3. Add a second Webhook by again clicking the "Add Webhook" button, and use the values below - then click "Save" again.

```
Webhook Name: Credit Key Update
Webhook URL: https://www.creditkey.com/app/3dcart/order_update?public_key=<public_key>&shared_secret=<shared_secret>
Format: JSON
Event: Order Status Change
```

## Set up REST API connection

1. In the 3DCart admin, navigate to "Modules", and type "REST API" into the search box.  Click the "Settings" button.

2. Click "Add" and enter the REST API Key provided by Credit Key. Then click "Save".



## Testing

Now, navigate to your website and add some items to the shopping cart - then navigate to the Checkout page.  If setup correctly, you should see the option to checkout with Credit Key.
