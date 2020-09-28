
# Callbacks {#webhook}

###### Last update: 2020-07-28 | [Edit on GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/_webhook_en.html.md)

> Outgoing payments - notification of payment in process

~~~http
POST /some-hook.php HTTP/1.1
Accept: application/json
Content-type: application/json
Host: falcon.com

{"hash": "50779a03d90c4fa60ac44dfd158dbceec0e9c57fa4cf4f5298450fdde1868945",
 "hookId": "f57f95e2-149f-4278-b2cb-4114bc319727",
 "messageId": "f9a197a8-26b6-4d42-aac4-d86b789c373c",
 "payment": {"account": "thedandod",
             "comment": "",
             "commission": Null,
             "date": "2018-05-18T16:05:15+03:00",
             "errorCode": "0",
             "personId": 79254914194,
             "provider": 25549,
             "signFields": "sum.currency,sum.amount,type,account,txnId",
             "status": "WAITING",
             "sum": {"amount": 1.73, "currency": 643},
             "total": {"amount": 1.73, "currency": 643},
             "txnId": "13117338074",
             "type": "OUT"},
 "test": false,
 "version": "1.0.0"}
~~~

> Outgoing payment - notification of successful payment

~~~http
POST /some-hook.php HTTP/1.1
Accept: application/json
Content-type: application/json
Host: falcon.com

{"hash": "50779a03d90c4fa60ac44dfd158dbceec0e9c57fa4cf4f5298450fdde1868945",
 "hookId": "f57f95e2-149f-4278-b2cb-4114bc319727",
 "messageId": "6e2a0e32-4c8d-4fe2-9eed-fe3b6a726ff4",
 "payment": {"account": "thedandod",
             "comment": "",
             "commission": {"amount": 0.0, "currency": 643},
             "date": "2018-05-18T16:05:15+03:00",
             "errorCode": "0",
             "personId": 79254914194,
             "provider": 25549,
             "signFields": "sum.currency,sum.amount,type,account,txnId",
             "status": "SUCCESS",
             "sum": {"amount": 1.73, "currency": 643},
             "total": {"amount": 1.73, "currency": 643},
             "txnId": "13117338074",
             "type": "OUT"},
 "test": false,
 "version": "1.0.0"}
~~~

> Outgoing payments - notification of unsuccessful payment

~~~http
POST /some-hook.php HTTP/1.1
Accept: application/json
Content-type: application/json
Host: falcon.com

{"hash": "0637b07b1018d76585db26b0f8077016b12996006429e22a7dc5b6982710a1ef",
 "hookId": "f57f95e2-149f-4278-b2cb-4114bc319727",
 "messageId": "1133873b-9bb6-4adb-9bfe-7be3a9aa999f",
 "payment": {"account": "borya241203",
             "comment": "",
             "commission": None,
             "date": "2018-05-20T05:19:16+03:00",
             "errorCode": "5",
             "personId": 79254914194,
             "provider": 25549,
             "signFields": "sum.currency,sum.amount,type,account,txnId",
             "status": "ERROR",
             "sum": {"amount": 1.01, "currency": 643},
             "total": {"amount": 1.01, "currency": 643},
             "txnId": "13126423989",
             "type": "OUT"},
 "test": false,
 "version": "1.0.0"}
~~~

> Incoming payment - notification of successful payment

~~~http
POST /some-hook.php HTTP/1.1
Accept: application/json
Content-type: application/json
Host: falcon.com

{"hash": "a56ed0090fa3fd2fd0b002ed80f85a120037a6a85f840938888275e1631da96f",
 "hookId": "8c79f60d-0272-476b-b120-6e7629467328",
 "messageId": "bba24947-ab5f-4b33-881b-738fc3a4c9e1",
 "payment": {"account": "79042426915",
             "comment": "Order i_4769798 Счет №65361451. Пополнение аккаунта "
                        "P11689160 (garik3315@gmail.com) в платежной системе "
                        "Payeer. Внимание! Не меняйте сумму, валюту и "
                        "комментарий к переводу, не делайте повторный перевод, "
                        "в ином случае Ваш платеж зачислен НЕ будет!",
             "commission": {"amount": 0.0, "currency": 643},
             "date": "2018-03-25T13:16:48+03:00",
             "errorCode": "0",
             "personId": 79645265240,
             "provider": 7,
             "signFields": "sum.currency,sum.amount,type,account,txnId",
             "status": "SUCCESS",
             "sum": {"amount": 1.09, "currency": 643},
             "total": {"amount": 1.09, "currency": 643},
             "txnId": "12565018935",
             "type": "IN"},
 "test": false,
 "version": "1.0.0"}
~~~

Webhook allows you to receive real-time HTTP notifications of events (outgoing / incoming payments). You need to implement a web service to receive and processing of POST-requests according to the [requests format](#hook_format).

**You need to respond the notification with HTTP 200 OK within 1-2 sec. If QIWI service has no response, it sends next notification in 10 min, then in 1 hour.**

Pools of IP-addresses from which QIWI service sends notifications:

* 79.142.16.0/20
* 195.189.100.0/22
* 91.232.230.0/23
* 91.213.51.0/24

If your web service works behinds the firewall, you need to add these IP-addresses to the list of allowed addresses for incoming TCP packets.

## Quick start {#quick_hook}

0. Implement web service for [webhook requests](#hook_format). Make sure to implement correctly the digital signature verification.
1. [Register your service](#hook_reg). **Please note that its URL original length (before URL-encoding) cannot be longer than 100 symbols**.
2. Request for [signature key](#hook_key).
3. Test your service with [test request](#hook_test). Empty notification will be sent to your service registered at stage 1.

To change webhook service URL:

1. [Remove current webhook service](#hook_remove).
2. [Register new webhook service](#hook_reg). **Please note that its URL original length (before URL-encoding) cannot be longer than 100 symbols**.
3. Request for new [signature key](#hook_key).
4. Test your service with [test request](#hook_test). Empty notification will be sent to your service registered at stage 1.


## Processing notification {#hook_format}

Each notification is an incoming POST-request with single payment data in JSON body. JSON scheme is as followed:


~~~http
POST /some-hook.php HTTP/1.1
Accept: application/json
Content-type: application/json
Host: falcon.com

{"hash": "a56ed0090fa3fd2fd0b002ed80f85a120037a6a85f840938888275e1631da96f",
 "hookId": "8c79f60d-0272-476b-b120-6e7629467328",
 "messageId": "bba24947-ab5f-4b33-881b-738fc3a4c9e1",
 "payment": {"account": "79042426915",
             "comment": "Order i_4769798 Счет №65361451. Пополнение аккаунта "
                        "P11689160 (garik3315@gmail.com) в платежной системе "
                        "Payeer. Внимание! Не меняйте сумму, валюту и "
                        "комментарий к переводу, не делайте повторный перевод, "
                        "в ином случае Ваш платеж зачислен НЕ будет!",
             "commission": {"amount": 0.0, "currency": 643},
             "date": "2018-03-25T13:16:48+03:00",
             "errorCode": "0",
             "personId": 79645265240,
             "provider": 7,
             "signFields": "sum.currency,sum.amount,type,account,txnId",
             "status": "SUCCESS",
             "sum": {"amount": 1.09, "currency": 643},
             "total": {"amount": 1.09, "currency": 643},
             "txnId": "12565018935",
             "type": "IN"},
 "test": false,
 "version": "1.0.0"}
~~~

~~~php
<?php

//Procedure returns string of sorted values from notification parameters and signature hash for verification
function getReqParams(){

    //Make sure that it is a POST request.
    if(strcasecmp($_SERVER['REQUEST_METHOD'], 'POST') != 0){
        throw new Exception('Request method must be POST!');
    }

    //Receive the RAW post data.
    $content = trim(file_get_contents("php://input"));

    //Attempt to decode the incoming RAW post data from JSON.
    $decoded = json_decode($content, true);

    //If json_decode failed, the JSON is invalid.
    if(!is_array($decoded)){
        throw new Exception('Received content contained invalid JSON!');
    }

    //Check if test
    if ($decoded['test'] == 'true') {
      throw new Exception('Test!');
    }

    // String of parameters
    $reqparams = $decoded['payment']['sum']['currency'] . '|' . $decoded['payment']['sum']['amount'] . '|'. $decoded['payment']['type'] . '|' . $decoded['payment']['account'] . '|' . $decoded['payment']['txnId'];
    // Signature
    foreach ($decoded as $name=>$value) {
       if ($name == 'hash') {
            $SIGN_REQ = $value;
       }
    }

    return [$reqparams, $SIGN_REQ];
}

// Resulted data

$Request = getReqParams();

// Base64 encoded key for decryption (method /hook/{hookId}/key)

$NOTIFY_PWD = "JcyVhjHCvHQwufz+IHXolyqHgEc5MoayBfParl6Guoc=";

// Get SHA-256 hash of the string and encrypt with your webhook key

$reqres = hash_hmac("sha256", $Request[0], base64_decode($NOTIFY_PWD));

// Verify signature

if (hash_equals($reqres, $Request[1])) {
    $error = array('response' => 'OK');
}
else $error = array('response' => 'error');

//Response

header('Content-Type: application/json');
$jsonres = json_encode($error);
echo $jsonres;
error_log('error code' . $jsonres);
?>
~~~

~~~python
import base64
import hmac
import hashlib

# Base64 encoded key (get it with /hook/{hookId}/key request)
webhook_key_base64 = 'JcyVhjHCvHQwufz+IHXolyqHgEc5MoayBfParl6Guoc='

# notification parameters
data = '643|1|IN|+79165238345|13353941550'

webhook_key = base64.b64decode(bytes(webhook_key_base64,'utf-8'))
print(hmac.new(webhook_key, data.encode('utf-8'), hashlib.sha256).hexdigest())
~~~


Field | Type | Descirption
----|------|-------
hookId | String (UUID) | Unique webhook id
messageId | String (UUID) | Unique notification id
payment | Object | Payment data
payment.txnId | String | QIWI Wallet transaction ID
payment.account | String | For outgoing payments - recipients account number. For incoming payments - sender number, self-service kiosk number, or top-up agent name
payment.signFields | String | A list of fields in `payment` object (separated by comma) to use for HmacSHA256 hash calculation of notification signature (see `hash` field)
payment.personId | Integer | Your wallet number
payment.date | String DateTime | Payment date/time, in Moscow time zone, as `YYYY-MM-DD'T'hh:mm:ss+03:00`
payment.errorCode | String | [Payment error code](#errorCode)
payment.type | String | Payment type:<br>`IN` - wallet topup, <br>`OUT` - payment
payment.status|String|Payment status:<br>`WAITING` - payment in process,<br>`SUCCESS` - successful payment,<br>`ERROR` - payment error.
payment.provider | Integer| Provider ID in QIWI Wallet
payment.comment | String | Transaction comment
payment.sum | Object | Payment amount data
----|----|---
sum.amount|Number(Decimal)|Amount
sum.currency|Integer|Currency code
----|----|----
payment.commission|Object| Payment commission
----|----|----
commission.amount|Number(Decimal)|Commission amount
commission.currency|Integer|Currency code
----|----|----
payment.total|Object|Total payment amount
----|----|----
total.amount|Number(Decimal)|Amount
total.currency|Integer|Currency code
----|----|----
test|Boolean|Flag indicating test notification
version|String|Webhook API version
hash|String| Hash of the notification's digital signature

### Notification signature verification 

To verify signature of a notification, proceed with the following:
  
1. Take values of fields specified in  `payment.signFields` field of the notification JSON (**in the same order**) as Strings.
2. Join them with `|` separator.
3. Encode the resulted string with SHA-256 and [signature key](#hook_key). 
4. Compare obtained value with `hash` field of the notification.

Example of signature verification (see also PHP procedure on the right tab):

1. You get [signature key](#hook_key), encoded in Base64: 
    `JcyVhjHCvHQwufz+IHXolyqHgEc5MoayBfParl6Guoc=`
2. You get notification:
    `{"messageId":"7814c49d-2d29-4b14-b2dc-36b377c76156","hookId":"5e2027d1-f5f3-4ad1-b409-058b8b8a8c22","payment":{"txnId":"13353941550","date":"2018-06-27T13:39:00+03:00","type":"IN","status":"SUCCESS","errorCode":"0","personId":78000008000,"account":"+79165238345","comment":"","provider":7,"sum":{"amount":1,"currency":643},"commission":{"amount":0,"currency":643},"total":{"amount":1,"currency":643},"signFields":"sum.currency,sum.amount,type,account,txnId"},"hash":"76687ffe5c516c793faa46fafba0994e7ca7a6d735966e0e0c0b65eaa43bdca0","version":"1.0.0","test":false}`
3. Join values of the fields specified in `signFields` field (`sum.currency,sum.amount,type,account,txnId`):  
    `643|1|IN|+79165238345|13353941550`
4. The obtained string is encoded with SHA-256 and the Base64-decoded key from step 1:
    `76687ffe5c516c793faa46fafba0994e7ca7a6d735966e0e0c0b65eaa43bdca0`
    Result coincides with `hash` field from the notification. The verification is successful.

## Webhook service registration {#hook_reg}

<h3 class="request method">Request → PUT</h3>

~~~shell
curl -X PUT "https://edge.qiwi.com/payment-notifier/v1/hooks?hookType=1&param=http%3A%2F%2Fecho.fjfalcon.ru%2F&txnType=2" \
     -H "accept: */*" \
     -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
PUT /payment-notifier/v1/hooks?hookType=1&param=http%3A%2F%2Fecho.fjfalcon.ru%2F&txnType=2 HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/payment-notifier/v1/hooks?<a>parameter=value</a></span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>

        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Send parameters in the request query. All parameters are required.</span>
    </li>
</ul>

Parameter|Type|Description
----|-----|------
hookType|Integer|Webhook type. Only `1`.
param|URL-encoded| Service URL. **Please note that its URL original length (before URL-encoding) cannot be longer than 100 symbols**
txnType|String| Choose type of transactions for notifications:<br>0 - only incoming transactions (wallet topup);<br>1 - only outgoing transactions (payments);<br>2 - all transactions

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "hookId":"d63a8729-f5c8-486f-907d-9fb8758afcfc",
  "hookParameters":{
    "url":"http://echo.fjfalcon.ru/"
  },
  "hookType":"WEB",
  "txnType":"BOTH"
}
~~~

Response in JSON.

Field|Type|Description
-----|------|------
hookId|String|Webhook service UUID
hookParameters|Object|Webhook service parameters
hookParameters.url|String| Webhook service URL
hookType|String|Webhook type (only `WEB`)
txnType|String|Transactions type for notifications (`IN` - incoming, `OUT` - outgoing, `BOTH` - all)

## Remove webhook service registration {#hook_remove}

<h3 class="request method">Request → DELETE</h3>

~~~shell
curl -X DELETE "https://edge.qiwi.com/payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc" \
   -H "accept: */*" \
   -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
DELETE /payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
        <li><h3>URL <span>/payment-notifier/v1/hooks/<a>hookId</a></span></h3></li>
        <ul>
             <li><strong>hookId</strong> - webhook UUID</li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>

        </ul>
    </li>
</ul>


<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "response":"Hook deleted"
}
~~~

Response in JSON.

Field|Type|Description
-----|------|------
response|String| Operation result

## Secret key provision {#hook_key}

Each notification contains digital signature encoded by secret key. Use this request to get the key.

<h3 class="request method">Request → GET</h3>


~~~shell
curl -X GET "https://edge.qiwi.com/payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/key" \
   -H "accept: */*" \
    -H "accept: */*" \
    -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
GET /payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/key HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~


<ul class="nestedList url">
    <li><h3>URL <span>/payment-notifier/v1/hooks/<a>hookId</a>/key</span></h3></li>
    <ul>
    <li><strong>hookId</strong> -webhook UUID</li>
    </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "key":"L8UVF3JkLVUr6r70LiE0A9/5WoGGwWKG2pI/e+l/9fs="
}
~~~

Response in JSON.

Field|Type|Description
-----|------|------
key|String| Base64-encoded key 

## Secret key change {#hook_secret}

Use this request to change the secret key for notifications signature.

<h3 class="request method">Request → POST</h3>


~~~shell
curl -X POST "https://edge.qiwi.com/payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/newkey" \
   -H "accept: */*" \
   -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
POST /payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/newkey HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~


<ul class="nestedList url">
    <li><h3>URL <span>/payment-notifier/v1/hooks/<a>hookId</a>/newkey</span></h3></li>
    <ul>
    <li><strong>hookId</strong> - webhook UUID</li>
    </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "key":"OikS4/CcIbSf+yYGnLbnOige8RGoYmGxs/LNMwkJy7Q="
}
~~~

Response in JSON.

Field|Type|Description
-----|------|------
key|String|Base64-encoded new key

## Webhook service data  {#hook_active}

Use this request to get the active webhook service linked to your wallet.

<h3 class="request method">Request → GET</h3>

~~~shell
curl -X GET "https://edge.qiwi.com/payment-notifier/v1/hooks/active" \
   -H "accept: */*" \
   -H "accept: */*" \
   -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
GET /payment-notifier/v1/hooks/active HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/payment-notifier/v1/hooks/active</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "hookId":"d63a8729-f5c8-486f-907d-9fb8758afcfc",
  "hookParameters":{
    "url":"http://echo.fjfalcon.ru/"
  },
  "hookType":"WEB",
  "txnType":"BOTH"
}
~~~

Response in JSON.

Field|Type|Description
-----|------|------
hookId|String|Active webhook UUID
hookParameters|Object|Webhook service parameters
hookParameters.url | String| Webhook URL
hookType|String|Webhook type (only `WEB`)
txnType|String|Transactions type for notifications (`IN` - incoming (wallet topup), `OUT` - outgoing (payments), `BOTH` - all)

## Test webhook service {#hook_test}

Use this request to test your webhook service. As a result of the request, empty notification is sent to the URL of the [active webhook service](#hook_active).

<h3 class="request method">Request → GET</h3>


~~~shell
curl -X GET "https://edge.qiwi.com/payment-notifier/v1/hooks/test" \
   -H "accept: */*" \
   -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
POST /payment-notifier/v1/hooks/test HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/payment-notifier/v1/hooks/test</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "response":"Webhook sent"
}
~~~

Response in JSON.

Field|Type|Description
-----|------|------
response|String| Notification on the request
