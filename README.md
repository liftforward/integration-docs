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

### 1. Obtain Merchant Identifiers and Access Keys
There are two environments: `test` and `production`.

Our merchant integration specialist will be supplying you with test credentials for each.

Example Credentials:
```
MERCHANT_ID: newpartnerfordemo
API_KEY: DALtvADu9HAQh0uZ8eI7caIn2emiCNNU
PUBLIC_KEY: 6b4edc235374a97001184a18f257b821
ENVIRONMENT: test
```

Everywhere in these doc you see `***VARIABLE***` you should replace this with the values we provide you

Once you have successfully integreated in the `test` environment, we will set you up with `production` credentials.

### 2. Import LiftForward Script
Embed LiftForward's JS runtime code on the page that your customers are checking out on.

```
<script src="https://checkout.liftforward.com/checkout.js"></script>
```

### 3. Initialize LiftForward Script
Once the script has been loaded on the page, you can initialize it with your credentials.

```
<script>
  liftforward.init({ merchant_id: '***MERCHANT_ID***',
                     public_key: '***PUBLIC_KEY***',
                     environment: '***ENVIRONMENT***'});
</script>
```

### 4. Create an onclick Function
There will be a button that when clicked will send LiftForward information and redirect the user. The first step is to create the function that will do this.

#### Complete Example
Here is an example of an `onclick` function that does everything needed:
```
<script>
  var onCheckoutButtonClick = function() {
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
    var options = {
      merchant_checkout_id: 'ch-03u849vs2f',
      charge_authorized_url: 'https://store.merchant.com/order-confirmation-page.html'
    }
    liftforward.checkout(salesQuote, options);
  }
</script>
```

There are three parts - the `salesQuote` object, the `options` object (optional), and the liftforward `checkout` function.

#### Sales Quote Object
Syntax notes
* You should build this object dynamically by adding item objects to it based on cart/checkout contents of your page.
* The sales quote object is sent as a JSON object.
* The items are stored in an object array.

Sales Quote object validation
The following data in the sales quote object is required:

* Term: Number of payments customer is going to make on the membership.
* Sales Tax Rate: The sales tax rate based on the customer's shipping address.

There must be at least one item. The following data in each item object is required:

* Quantity: Count of item included in sales quote.
* SKU: ID provided by merchant to identifier the item.
* Title: Short title for the item.
* Unit Sale Price: Unit price item was sold at. Used to calculate total_sale_price.

Minimum Example Object:
```
var salesQuote = {
  term: 24,
  sales_tax_rate: 8.89,
  line_items: [{
    quantity: 4,
    title: "Point of Sale System",
    sku: "H-BM1-WW",
    unit_sale_price: 199.99
  }]
};
```

Full Example Object:
```
{
  "term": 24,
  "customer_email": "gus.fring@lospoloshermanos.com",
  "customer_first_name": "Gus",
  "customer_last_name": "Fring",
  "customer_organization_name": "Los Polos Hermanos",
  "customer_street": "12000 – 12100 Coors Rd SW",
  "customer_city": "Albuquerque",
  "customer_state": "NM",
  "customer_postal": "87045",
  "customer_country_code": "US",
  "customer_phone": "123-456-7890",
  "customer_website": "https://lospoloshermanos.com",
  "customer_description": "Fried chicken franchise popular in the south west.",
  "customer_organization_structure": "corporation",
  "merchant_customer_id": "123123123",
  "merchant_sales_quote_id": "3422343242",
  "sales_rep_first_name": "Walter",
  "sales_rep_last_name": "White",
  "sales_rep_email": "walter.white@jpwynnehighschool.edu",
  "shipping_first_name": "Gus",
  "shipping_last_name": "Fring",
  "shipping_organization_name": "Los Polos Hermanos",
  "shipping_street": "12000 – 12100 Coors Rd SW",
  "shipping_city": "Albuquerque",
  "shipping_state": "NM",
  "shipping_postal": "87045",
  "shipping_country_code": "US",
  "shipping_phone": "US",
  "shipping_email": "US",
  "shipping_via": "expedited",
  "effective_tax_rate": 0.075,
  "total_sales_tax": 34.44,
  "total_shipping_fee": 35,
  "miscellaneous_fees": [
    {
      "name": "ca-env-fee",
      "display_name": "California Environmental Fee",
      "fee": 6
    }
  ],
  "meta_data": {},
  "line_items": [
    {
      "quantity": 2,
      "sku": "BM101",
      "title": "Blue Rock Candy",
      "description": "Blue Rock Candy is the best candy ever!",
      "category": "candy",
      "unit_retail_price": 283.34,
      "unit_sale_price": 232.44,
      "image_url": "//i.pinimg.com/474x/e1/27/4e/bm101.jpg",
      "meta_data": {}
    }
  ]
}
```

#### Options Object
You can optionally pass in an options object. Within this object there are two options - both are optional:
* Merchant Checkout ID: The ID of the checkout in your ecommerce system.
* Charge Authorized URL: The URL you want the user to be redirected to once they sign an agreement with LiftForward.

Example Object:
```
var options = {
  merchant_checkout_id: 'ch-03u849vs2f',
  charge_authorized_url: 'https://store.merchant.com/order-confirmation-page.html'
}
```

#### The LiftForward Checkout Function
```
liftforward.checkout(salesQuote, options);
```

This method will send a POST request to the /sales-quotes/ API endpoint with the sales quote object as the data payload. Then, it will redirect the user to the LiftForward checkout flow on the liftforward.com domain.

### 5. Invoke the `onclick` function
Using a HTML button is a simply way of integrating with LiftForward, but is not the only way. As long as the `onclick` function event is called - you can imagine many other ways to make it work.

This button will be a payment method for the user's checkout - either the only one or one of many (such as credit cards, paypal, etc). Be sure to attach an `onclick` attribute with the same name as the `onclick` function you previously defined.
```
<button class="btn btn-lg btn-outline-primary btn-primary" onclick="onCheckoutButtonClick()">Checkout</button>
```

## Receive Authorization Token
After the user clicks the button, they will be redirected to LiftForward's website where they will apply for a membership. After they are approved and they sign the membership agreement, they will be redirected to the `charge_authorized_url` page. The exact URL they are redirected to is specificed by the `charge_authorized_url` property of the `options` object in the initial `liftforward.checkout(salesQuote, options)` call.

LiftForward will append `merchant_checkout_id` and `authorization_token` query params to this `charge_authorized_url` before redirecting.

When the `charge_authorized_url` page is loading on the redirect we recommend that it do two things:
1. You can use the `merchant_checkout_id` query param to look up the checkout in your ecommerce system in order to show the user what they just ordered.
2. Exchange the `authorization_token` for a charge.

#### Creating Charges
You will need to exchange your `authorization_token` for a charge. There are three reasons for this.

1. To ensure that the `authorization_token` is real.
2. To ensure that the `authorization_token` hasn't been used before.
3. To verify which `merchant_checkout_id` the `authorization_token` belongs to.

A charge should be created when you intend to fulill the customers order. When the charge is created it is in the `authorized` state - similar to how a Credit Card transaction will first be authorized before it is actually submitted for settlement.

POST to the `/charges` endpoint

Test
`https://api.liftforward.com/test/v2/charges`

Production
`https://api.liftforward.com/v2/charges`

Headers
```
"Content-Type":"application/json"
"apikey":"***API_KEY***"
```

Body
```
{"authorization_token": "XXXXXXXXXXXXXXXXX"}
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

#### Fulfillment
Once the charge has been created between your company and LiftForward, you can begin your own fulfillment process - including shipments and emails.

## Capture the Charge
Once you have shipped the device you can actually capture the charge. LiftFoward will then begin collecting montly payments from the customer.

There are two ways to do this - via our partner site, or via API call.

#### Partner Site
Log in to your partner site and go to the charges page

Test
https://`***MERCHANT_ID***`.liftforward-staging.com/sales/charges

Production
https://`***MERCHANT_ID***`.liftforward.com/sales/charges

![screenshot 2018-05-10 09 37 18](https://user-images.githubusercontent.com/529744/39872382-cead5b46-5435-11e8-9e20-52d56a567157.png)

Find the charge you want to capture, and press capture.

#### API call

POST to the `/charges/:id/capture` endpoint

Test
`https://api.liftforward.com/test/v2/charges/:id/capture`

Production
`https://api.liftforward.com/v2/charges/:id/capture`

Headers
```
"Content-Type":"application/json"
"apikey":"***API_KEY***"
```

Body
```

```
Note: the body is empty

Response
```
{
    "charge": {
        "id": "CHF5T3M22NS",
        "amount": 319.98,
        "merchant_checkout_id": "ch-03u849vs2f",
        "status": "captured"
    }
}
```
