# LiftForward Merchant Integration
A developers guide to integrating your ecommerce site with LiftForward's checkout library and APIs.

## Integration Steps

1. Contact our merchant integration specialists to be setup as a merchant. 
2. Schedule demo/kickoff meeting with LiftForward engineering and receive API keys.
3. Embed LiftForward checkout.js library into your e-commerse site checkout.
4. Create a authorization page to capture the LiftForward **authorization_token** and create a **charge** via the LiftForward **charges** api.

## User Flow

1. Customer creates a cart on the merchant's e-commerce site and begins to checkout.
2. Customer supplies shipping address and merchant's e-commerce site calculates sales tax rate, taxes and any additional fees (such as the California environmental fee).
3. Customer clicks the **Checkout with LiftForward** button to select LiftForward as a payment method.
4. Customer is redirected to the LiftForward site and fills out a credit application application. 
5. LiftForward reviews the customers application and approves it.
6. Customer e-signs an [TBD] agreement with LiftForward.
7. Customer is redirected from LiftForwards site to the **charge_authorized_url** passed in step 4 with an added **authorization_token** parameter.
8. The page at the **charge_authorized_url** reads the request parameter and temporarily stores this **authorization_token**.
9. The page at the **charge_authorized_url** sends a POST request to the LiftForward Charges API endpoint (/v2/charges) with the saved **authorization_token**.
10. The page at the **charge_authorized_url** expects a response from the LiftForward API with the charge information (charge_id, merchant_checkout_id, amount and status)
11. The page at the **charge_authorized_url** validates the charge details
12. If valid, the page at the **charge_authorization_url** stores the charge_id from the returned charge object. This **Charge ID** is attached to the order; and is used to uniquely identify the order. **All future charge actions require this identifier.**
13. Order is created.
14. Customer is presented with order confirmation page/message.



## Integrating LiftForward checkout.js

### Embed LiftForward
Embed LiftForward's JS runtime code

```
<script src="https://checkout.liftforward-staging.com/checkout.js"></script>
```

Initialize with your credentials and set the environment

```
<script>
  liftforward.init({ merchant_id: 'XXXXXXXXXXXXXXX',
                     public_key: 'XXXXXXXXXXXXXXX',
                     environment: 'XXXXXXXXXXXXXXX'});
</script>
```
Note: replace the `merchant_id` and `public_key` values with your own values. The API key must match the Liftforward-environment you're referencing ('test' or 'production').

Once the LiftForward JS is initialized, the liftforward.checkout method is made available:

`liftforard.checkout(salesQuote, options)` - Sends the sales quote and options object via POST request.

### Sales Quote Object
Syntax notes
* The sales quote object is sent as a JSON object.
* All numerical values must be sent as integer USD cents ($25.00 -> 2500).
* The items are stored in an object array. You should build this array dynamically by adding item objects to it based on sales quote contents.

Sales Quote object validation
The following data in the sales quote object is required:

* Term: Number of payments customer is going to make on the membership.

There must be at least one item. The following data in each item object is required:

* Quantity: Count of item included in sales quote.
* SKU: ID provided by merchant to identifier the item.
* Title: Short title for the item.
* Unit Sale Price: Unit price item was sold at. Used to calculate total_sale_price and pre_tax_term_payment.

Example Object:
```
var salesQuote = {
  term: 24,
  total_sales_tax: 77.99,
  total_shipping_fee: 42.00,
  customer_organization_name: "Los Polos Hermanos",
  customer_first_name: "Gus",
  customer_last_name: "Fring",
  line_items: [{
    quantity: 4,
    title: "Point of Sale System",
    sku: "H-BM1-WW",
    unit_sale_price: 199.99,
    image_url: "https://jumbotron-production-f.squarecdn.com/assets/129869ad5838545704cd.jpg"
  }]
};
```

### Options Object
There are two options - both are optional:
* Merchant Checkout ID: The ID of the checkout in your ecommerce system.
* Charge Authorized Url: The URL you want the user to be redirected to once they sign an agreement with LiftForward.

Example Object:
```
let options = {
  merchantCheckoutId: 'ch-03u849vs2f',
  chargeAuthorizedUrl: 'https://tech.liftforward.com/thank-you.html'
}
```

### Initiate checkout
Initiating checkout

With the salesQuote and options objects built, you can now call:

```
liftforard.checkout(salesQuote, options);
```

This method will send a POST request to the /sales-quotes/ API endpoint with the sales quote object as the data payload. Then, it will redirect the user to the LiftForward checkout flow on the liftforward.com domain.

### Receive Authorization Token
After the user has signed an agreement with LiftForward, they will be redirected back to your site. The exact URL they are redirected to is specificed in the `options` object in the initial `liftforward.checkout(salesQuote, options)` call.

LiftForward will append `merchant_checkout_id` and `authorization_token` query params to this URL.

When the page is loading on the redirect we recommend that you do two things:
1. You can use the `merchant_checkout_id` query param to look up the checkout in your ecommerce system in order to show the user what they just ordered.
2. Take the `authorization_token` from the query param and create a charge on LiftForward.

POST to the `/charges` endpoint

Test
`https://api.liftforward.com/test/v2/charges`

Production
`https://api.liftforward.com/v2/charges`

Headers
```
"Content-Type":"application/json"
"apikey":"XXXXXXXXXXXXXXX"
```
Note: replace the `apikey` value with your actual API key. The API key must match the Liftforward-environment you're referencing ('test' or 'production').

Body
```
{"authorization_token": "XXXXXXXXXXXXXXX"}
```
Note: replace the `authorization_token` value with the value from the `authorization_token` query param in the redirect url

Response
```
{
    "charge": {
        "id": "CHF5T3M22NS",
        "amount": 319.98,
        "merchant_checkout_id": "ch-03u849vs2f",
        "status": "authorized"
    }
}
```

This is a charge between your company and LiftForward. When it is created it is in the `authorized` state - similar to how a Credit Card transaction will first be authorized before it is actually submitted for settlement.

## Capture the Charge
Log in to your partner site and go to the charges page
https://XXXXXXXXXXXXXXXX.liftforward-staging.com/sales/charges

Note: replace `XXXXXXXXXXXXXXXX` with your `merchant_id`.

![screenshot 2018-05-10 09 37 18](https://user-images.githubusercontent.com/529744/39872382-cead5b46-5435-11e8-9e20-52d56a567157.png)

Find the charge you want to capture, and press capture.
