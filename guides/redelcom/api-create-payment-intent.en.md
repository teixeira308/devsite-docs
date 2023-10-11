# Create a payment intent

A payment intent is a request containing the details of a transaction to be made. 

To create a payment intent, you need to have previously made the call to obtain the terminal code to which the transaction will be associated. Keep in mind that if you have already done this once and saved the terminal `id`, you won't need to repeat this query, as that code is unique and unchanging.

Next, make a POST call to the API `{https://api-dev.redelcom.cl:20010/v2}/pago`, adding the value obtained previously to the `terminalId` field and replacing `X-Authentication`, `clientId`, and `secret` with your credentials:

```curl
curl -X POST \
 'https://api-dev.redelcom.cl:20010/v2/pago' \
--header 'X-Authentication: clientId;secret' \
--header 'Content-Type: application/json' \
--data '{


   "amount": 6900,
   "description": "Compra de prueba",
   "paymentType": "EFECTIVO",
   "products": [{
       "description": "PIZZA MEDIANA",
       "id": "1845",
       "iva_exempt": false,
       "quantity": 10,
       "unit_price": 300
   }],
   "rdcDTE": true,
   "requestTip": true,
   "responseCallback": "https://eom2h2tqm3g6y7l.m.pipedream.net",
   "terminalId": "3127",
   "userTransactionId": "345565"


}'

```


Configure the fields of this request following these specifications:

| Field | Type | Description |
|---|---|---|
| `amount` | integer | **Mandatory**. <br>Transaction amount. It must be a number between 1 and 99999999. |
| `Description` | string | **Optional**. <br>Additional information to display on the payment request. |
| `dte` | string | **Optional**. <br>You should configure this if the Documento Tributario Electrónico (DTE) associated with the payment is not generated by Redelcom and you want the POS to print it. Only applies if `rdcDTE` is `false`. <br>You can refer to the "Print DTE" section for more information. |
| `paymentType` | enum | **Optional**. <br>Payment method to use. If the parameter is omitted, the payment method must be chosen at the POS. <br>Possible values: `EFECTIVO`, `TARJETA`, `WALLET`, `SIN_EFECTIVO`, `SIN_WALLET`. |
| Products - `description` | string | **Optional**. <br>Brief description of the product sold. |
| Products - `id` | string | **Optional**. <br>Product code. |
| Products - `iva_exempt` | boolean | **Optional**. <br>Whether the product is exempt from VAT or not. Possible values: `True` if exempt; `false` if not. |
| Products - `quantity` | integer | **Optional**. <br>Number of items. |
| Products - `unit_price` | integer | **Optional**. <br>Unit price of the sold item. |
| `rdcDTE` | boolean | **Optional**. <br>Whether Redelcom should generate the DTE (by default, it does not). |
| `requestTip` | boolean | **Optional**. <br>Whether the terminal should request a tip (by default, it does not). |
| `responseCallback` | string | **Optional**. <br>The integrator's own URL that Redelcom will invoke to report the status of a payment at the end of the transaction, whether it is successful or not. To configure it, go to the "Implementing the Response URL" subsection.<br>We recommend using this method to avoid having to make a query call for payment intentions. |
| `terminalId` | string | **Mandatory**. <br>Terminal code to which the transaction will be associated, obtained from the "Get terminal" call. |
| `UserTransactionId` | string | **Mandatory**. <br>Referential code for the payment request, which you can define. |


If the request is successful, the response will return the code associated with the payment request, and it will look similar to the following example:

```json
{
   "rdcTransactionId": "294-3727-16384"
}

```


> WARNING 
> 
> Important 
>
> We recommend storing the `rdcTransactionId` associated with each transaction made to facilitate later queries for a payment intention.

## Errors processing payment intents

In the following table, you can see the main reasons why this request **may not** be processed:

| Error | Reason |
|---|---|
| STATUS CODE - 400 | Invalid payment information. Review the sent information and try again. |
| STATUS CODE 401 | Invalid or nonexistent authentication. Review your credentials. |
| STATUS CODE - 409 | There is already a pending payment intention. Please try again later. |


## Implementing the Response URL

When creating a payment intent, we recommend configuring the `responseCallback` parameter, which allows you to receive updates on the final statuses of these intentions at your own URL.

To implement this `responseCallback`, please note:

* It should be a `POST` call with a single parameter in the query string, `rdcTransactionId`, which is the transaction identifier provided by RedelCom when requesting payment.
* The `Content-Type` should be `application/json`.

This implementation will be considered successful if the service returns an `HTTP 200 OK` status code. If, on the other hand, the response returns a `5xx` status code, the service will retry the URL call every 1 minute, up to 3 times. If the returned response is not one of these two options mentioned, the implementation will be considered failed and will not be retried.