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

`liftforard.checkout.open(salesQuote, options)` - Sends the sales quote and options object via POST request.

TODO: show html etc for checkout.js

what methods to call etc..

### Sales Quote Object

TODO: provide details

### Receive Authorization Token

TODO: provide details

...

TODO add more detail of remaining steps
