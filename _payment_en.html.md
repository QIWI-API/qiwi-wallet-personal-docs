# Payments API {#payments}

###### Last update: 2022-01-20 | [Edit on GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/_payment_en.html.md)

## Commission rates {#rates}

Returns total commission amount for the payment by the given payment requisites.

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST 'https://edge.qiwi.com/sinap/providers/99/onlineCommission' \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  -d '{ \
        "account":"380995238345", \
        "paymentMethod":{ \
          "type":"Account", \
          "accountId":"643" \
        },\
        "purchaseTotals":{ \
          "total":{ \
            "amount":10, \
            "currency":"643" \
          } \
        } \
  }'
~~~

~~~http
POST /sinap/providers/99/onlineCommission HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com

{
  "account":"380995238345",
  "paymentMethod":{
    "type":"Account",
    "accountId":"643"
  },
  "purchaseTotals":{
    "total":{
      "amount":10,
      "currency":"643"
    }
  }
}
~~~

~~~python
import requests

# Тарифные комиссии
def get_commission(api_access_token, to_account, prv_id, sum_pay):
    s = requests.Session()
    s.headers = {'content-type': 'application/json'}
    s.headers['authorization'] = 'Bearer ' + api_access_token  
    postjson = {"account":"","paymentMethod":{"type":"Account","accountId":"643"}, "purchaseTotals":{"total":{"amount":"","currency":"643"}}}
    postjson['account'] = to_account
    postjson['purchaseTotals']['total']['amount'] = sum_pay
    c_online = s.post('https://edge.qiwi.com/sinap/providers/'+prv_id+'/onlineCommission',json = postjson)
    return c_online.json()['qwCommission']['amount']
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/providers/<a>ID</a>/onlineCommission</span></h3>
    </li>
</ul>

**ID** — provider's identifier. Possible values:

* 99 — QIWI Wallet transfer.
* [Providers of money transfer to cards](#cards).
* [Providers of bank wire transfer](#banks).
* [Mobile network operators](#mnp).
* 1717 — [Payment by bank requisites to commercial organization](#freepay).

You can also search for a required identifier [through API by keywords](#provider-search).

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Required parameters in JSON body:

Name|Type|Description
--------|----|----
account| String| User's identifier (phone number, card/account number, and other entity depending on provider)
paymentMethod | Object| Object defining payment processing by QIWI Wallet. Includes the following parameters:
paymentMethod.type|String | Payment method, `Account` only
paymentMethod.accountId|String| QIWI Wallet account identifier, `643` only.
purchaseTotals | Object | Object with payment requisites
purchaseTotals.total|Object| Payment amount data:
total.amount|Number | Amount (rubles and kopeks, divided by `.`). Positive number, rounded down to 2 decimals. If you send more decimals, value will be rounded down to kopeks.
total.currency|String|Currency (`643` only, that is rubles)

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "providerId": 99,
    "withdrawSum": {
        "amount": 1011.01,
        "currency": "643"
    },
    "enrollmentSum": {
        "amount": 1001,
        "currency": "643"
    },
    "qwCommission": {
        "amount": 10.01,
        "currency": "643"
    },
    "fundingSourceCommission": {
        "amount": 0,
        "currency": "643"
    },
    "withdrawToEnrollmentRate": 1
}
~~~

~~~python
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# QIWI wallet transfer commission
print(get_commission(api_access_token,'+380000000000','99',5000))
# Card transfer commission
print(get_commission(api_access_token,'4890xxxxxxxx1698','22351',1000))
~~~

Commission rate returns in `qwCommission.amount` field of the JSON response.

<a name="payform"></a>

## Payment form auto-filling {#autocomplete}

Provides auto-filled payment form on qiwi.com site.

[Link example (click to see the form)](https://qiwi.com/payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&amountFraction=0&extra%5B%27comment%27%5D=test123&currency=643&blocked[0]=account)

If you don't want to show your wallet number, use transfer to the wallet nickname:

[Link example for nickname transfer (click to see the form)](https://qiwi.com/payment/form/99999?extra%5B%27account%27%5D=NICKNAME&amountInteger=1&amountFraction=0&currency=643&blocked[0]=account&extra%5B%27accountType%27%5D=nickname)

To get payments to your QIWI Wallet, you can use also [P2P-form](https://qiwi.com/p2p-admin/transfers/link).

<h3 class="request method">Request → GET</h3>

~~~http
GET /payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&amountFraction=0&extra%5B%27comment%27%5D=test123&currency=643 HTTP/1.1
Host: qiwi.com
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://qiwi.com/<a>ID</a>?<a>{parameter}={value}</a></span></h3>
    </li>
</ul>

**ID** — provider's identifier. Only providers with [payment requisites](#payments_model) limited to `fields.account` are accepted. Possible values:

* 99 — QIWI Wallet transfer.
* 99999 — QIWI Wallet nickname transfer.
* 1963 — Visa card transfer (issued by only Russian banks).
* 21013 — MasterCard card transfer (issued by only Russian banks).
* 31652 — MIR national payment system card transfer.
* 22351 — [QIWI Virtual card](https://qiwi.com/cards/qvc) transfer.
* 1717 — [Payment by bank requisites to commercial organization](#freepay).
* [Mobile network operators](#mnp).

You can also search for a required identifier [through API by keywords](#provider-search).

<ul class="nestedList params">
    <li><h3>Parameters</h3></li>
</ul>

Parameters in URL query to fill the form fields:

Name             | Type                   | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                             | Form field                            | Required
-----------------|------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------|---------
amountInteger    | Integer                | Integer part of the payment amount (in rubles). If absent, the "Amount" field on the form will be empty. **The number not greater than 99 999 (payment amount limit)**                                                                                                                                                                                                                                                                                                  | Amount                                | -
amountFraction   | Integer                | Fractional part of the payment amount (kopeks). If absent, the "Amount" field on the form will be empty.                                                                                                                                                                                                                                                                                                                                                                | Amount                                | -
currency         | Constant string, `643` | Payment currency code. **Required if you send the payment amount in the link**                                                                                                                                                                                                                                                                                                                                                                                          | -                                     | +
extra['comment'] | URL-encoded string     | Payment comment. **Use it only for ID=99**                                                                                                                                                                                                                                                                                                                                                                                                                              | Comment to the transfer               | -
extra['account'] | URL-encoded string     | Field format is the same as `fields.account` of the corresponding payment request: for provider `99` - recipient's wallet number; for mobile network operators - phone number for top-up (without international prefix); for card transfer - recipient's card number without spaces, for other providers - user's identifier. For provider `99999` use [nickname](#nickname) or number of the wallet and use appropriate value of `extra['accountType']` parameter.                  | Wallet number, phone number, user ID. | -
blocked          | Array[String]          | Name of blocked (inactive) form field. User will not be able to change this field. Each parameter corresponds to respective form field and should be numbered starting from zero (`blocked[0]`, `blocked[1]`, and so on). If absent, user can change all form fields. Possible values:<br>`sum` - "Payment amount" field, <br>`account` - "Phone number/Account number" field,<br>`comment` - "Comment" field.<br> Example (for payment amount field): `blocked[0]=sum` | -                                     | -
extra['accountType'] | URL-encoded string | **Use only for ID=99999**. The value determines transfer to QIWI wallet by nickname or wallet number.<br>`phone` - for transfer to wallet number<br>`nickname` - for transfer to wallet nickname. If you don't want to show your wallet number on the form, use this value.

### How to retrieve your wallet nickname {#nickname}

<h3 class="request method">Request → GET</h3>

~~~shell
user@server:~$ curl -X GET "https://edge.qiwi.com/qw-nicknames/v1/persons/79111234567/nickname" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9"
~~~

~~~http
GET /qw-nicknames/v1/persons/79111234567/nickname HTTP/1.1
Accept: application/json
Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9
Host: edge.qiwi.com
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/qw-nicknames/v1/persons/<a>wallet</a>/nickname</span></h3>
        <ul>
             <li><strong>wallet</strong> - your wallet number without <i>+</i> sign</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "canChange": true,
  "canUse": true,
  "description": "",
  "nickname": "NICKNAME"
}
~~~

Successful response contains nickname of your wallet in JSON field `nickname`.

## QIWI wallet transfer {#p2p}

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST 'https://edge.qiwi.com/sinap/api/v2/terms/99/payments' \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  -d '{ \
        "id":"11111111111111", \
        "sum": { \
          "amount":100, \
          "currency":"643" \
        }, \
        "paymentMethod": { \
          "type":"Account", \
          "accountId":"643" \
        },
        "comment":"test", \
        "fields": { \
          "account":"+79121112233" \
        } \
      }'
~~~

~~~http
POST /sinap/api/v2/terms/99/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com

{
 "id":"11111111111111",
 "sum": {
  "amount":100.50,
  "currency":"643"
 },
 "paymentMethod": {
  "type":"Account",
  "accountId":"643"
 },
 "comment":"test",
 "fields": {
	"account":"+79121112233"
 }
}
~~~

~~~python
import requests
import time

# QIWI wallet transfer
def send_p2p(api_access_token, to_qw, comment, sum_p2p):
    s = requests.Session()
    s.headers = {'content-type': 'application/json'}
    s.headers['authorization'] = 'Bearer ' + api_access_token
    s.headers['User-Agent'] = 'Android v3.2.0 MKT'
    s.headers['Accept'] = 'application/json'
    postjson = {"id":"","sum":{"amount":"","currency":""},"paymentMethod":{"type":"Account","accountId":"643"}, "comment":"'+comment+'","fields":{"account":""}}
    postjson['id'] = str(int(time.time() * 1000))
    postjson['sum']['amount'] = sum_p2p
    postjson['sum']['currency'] = '643'
    postjson['fields']['account'] = to_qw
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/99/payments',json = postjson)
    return res.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/99/payments</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Send JSON object [Payment](#payment_obj) in the request's body. Payment requisites in `fields` object:

Name|Type|Description|Required
--------|----|----|------
fields.account| String|Wallet number of the recipient |+

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": "150217833198900",
    "terms": "99",
    "fields": {
        "account": "79121238345"
    },
    "sum": {
        "amount": 100,
        "currency": "643"
    },
    "transaction": {
        "id": "11155897070",
        "state": {
            "code": "Accepted"
        }
    },
    "source": "account_643",
    "comment": "test"
}
~~~

~~~python
print(send_p2p(mylogin,api_access_token,'+79261112233','comment',99.01))

{'comment': 'comment',
 'fields': {'account': '+79261112233'},
 'id': '1514296828893',
 'source': 'account_643',
 'sum': {'amount': 99.01, 'currency': '643'},
 'terms': '99',
 'transaction': {'id': '11982501857', 'state': {'code': 'Accepted'}}}
~~~

Successful response contains JSON-object [PaymentInfo](#payment_info) with accepted payment data.

## Conversion {#exchange}

Transfers funds to currency account in QIWI wallet with conversion from your QIWI wallet ruble account. Two transactions are created: conversion between accounts of your QIWI wallet, and transfer to another wallet. You can get currency rates from [another API method](#CCY).

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST 'https://edge.qiwi.com/sinap/api/v2/terms/1099/payments' \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  -d '{ \
        "id":"11111111111111", \
        "sum": { \
          "amount":100, \
          "currency":"398" \
        }, \
        "paymentMethod": { \
          "type":"Account", \
          "accountId":"643" \
        }, \
        "comment":"test", \
        "fields": { \
          "account":"+79121112233" \
        } \
      }'
~~~

~~~http
POST /sinap/api/v2/terms/1099/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com

{
  "id":"11111111111111",
  "sum": {
		"amount":10.00,
		"currency":"398"
	},
	"paymentMethod": {
		"type":"Account",
		"accountId":"643"
	},
	"comment":"test",
	"fields": {
	 	"account":"+79121112233"
	}
}
~~~

~~~python
import requests
import time

# Conversion in QIWI wallet (currency as currency integer code as String)
def exchange(api_access_token, sum_exchange, currency, to_qw):
    s = requests.Session()
    currencies = ['398', '840', '978']
    if currency not in currencies:
      print('This currency not available')
      return
    s.headers = {'content-type': 'application/json'}
    s.headers['authorization'] = 'Bearer ' + api_access_token
    s.headers['User-Agent'] = 'Android v3.2.0 MKT'
    s.headers['Accept'] = 'application/json'
    postjson = {"id":"","sum":{"amount":"","currency":""},"paymentMethod":{"type":"Account","accountId":"643"}, "comment":"'+comment+'","fields":{"account":""}}
    postjson['id'] = str(int(time.time() * 1000))
    postjson['sum']['amount'] = sum_exchange
    postjson['sum']['currency'] = currency
    postjson['fields']['account'] = to_qw
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/1099/payments',json = postjson)
    return res.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/1099/payments</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
<li><h3>Parameters</h3>
</li>
</ul>

Send JSON object [Payment](#payment_obj) in the request's body. Payment requisites in `fields` object:

Name|Type|Description|Required
--------|----|----|------
fields.account| String|Wallet number for the conversion|+

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": "150217833198900",
    "terms": "99",
    "fields": {
        "account": "79121238345"
    },
    "sum": {
        "amount": 100,
        "currency": "398"
    },
    "transaction": {
        "id": "11155897070",
        "state": {
            "code": "Accepted"
        }
    },
    "source": "account_643",
    "comment": "test"
}
~~~

Successful response contains JSON-object [PaymentInfo](#payment_info) with accepted payment data.

## Currency rates {#CCY}

Returns current QIWI Bank currency rates and cross-rates.

<h3 class="request method">Request → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/sinap/crossRates" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9"
~~~

~~~http
GET /sinap/crossRates HTTP/1.1
Accept: application/json
Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9
Content-type: application/json
Host: edge.qiwi.com
~~~

~~~python
import requests

# Currencies rate (currency pair codes in String)
def exchange(api_access_token, currency_to, currency_from):
    s = requests.Session()
    s.headers = {'content-type': 'application/json'}
    s.headers['authorization'] = 'Bearer ' + api_access_token
    s.headers['User-Agent'] = 'Android v3.2.0 MKT'
    s.headers['Accept'] = 'application/json'
    res = s.get('https://edge.qiwi.com/sinap/crossRates')

    # all rates
    rates = res.json()['result']

    # requested rate
    rate = [x for x in rates if x['from'] == currency_from and x['to'] == currency_to]
    if (len(rate) == 0):
        print('No rate for this currencies!')
        return
    else:
        return rate[0]['rate']
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/crossRates</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "result": [
        {
            "set": "General",
            "from": "398",
            "to": "643",
            "rate": 6.22665
        },
        {
            "set": "General",
            "from": "398",
            "to": "756",
            "rate": 412.0174305
        },
        ...,
        {
            "set": "General",
            "from": "980",
            "to": "978",
            "rate": 31.4680914
        }
    ]
}
~~~

Successful response contains JSON array of currency rates in `result` field. An element of the list corresponds to currency pair:

Response field|Type|Description
--------|----|----
from|String| Base currency
to|String| Quote currency
rate|Number|Rate

## Mobile network payment {#cell}

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  -d '{ \
        "id":"11111111111111", \
        "sum": { \
          "amount":100, \
          "currency":"643" \
        }, \
        "paymentMethod": { \
          "type":"Account", \
          "accountId":"643" \
        }, \
        "fields": { \
          "account":"9161112233" \
        } \
      }'
~~~

~~~http
POST /sinap/api/v2/terms/1/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com

{
  "id":"21131343",
  "sum": {
        "amount":1000,
        "currency":"643"
  },
  "paymentMethod": {
      "type":"Account",
      "accountId":"643"
  },
  "fields": {
        "account":"9161112233"
  }
}
~~~

~~~python
import requests
import time

# Cell phone top-up
def send_mobile(api_access_token, prv_id, to_account, comment, sum_pay):
    s = requests.Session()
    s.headers['Accept'] = 'application/json'
    s.headers['Content-Type'] = 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    postjson = {"id":"","sum": {"amount":"","currency":"643"},"paymentMethod": {"type":"Account","accountId":"643"},"comment":"","fields": {"account":""}}
    postjson['id'] = str(int(time.time() * 1000))
    postjson['sum']['amount'] = sum_pay
    postjson['fields']['account'] = to_account
    postjson['comment'] = comment
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/'+prv_id+'/payments', json = postjson)
    return res.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/<a>ID</a>/payments</span></h3>
        <ul>
             <li><strong>ID</strong> - QIWI provider identifier. <a href="#mnp">How to get provider ID</a></li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
<li><h3>Parameters</h3>
</li>
</ul>

Send JSON object [Payment](#payment_obj) in the request's body. Payment requisites in `fields` object:

Name|Type|Description
--------|----|----
fields.account| String|Cell phone number to top-up (without `8` prefix)

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": "21131343",
    "terms": "1",
    "fields": {
          "account": "9161112233"
    },
    "sum": {
         "amount": 1000,
         "currency": "643"
    },
    "source": "account_643",
    "transaction": {
         "id": "4969142201",
         "state": {
            "code": "Accepted"
          }
    }
}
~~~

~~~python
send_mobile(api_access_token,'2','9670058909','123','1')
~~~

Successful response contains JSON-object [PaymentInfo](#payment_info) with accepted payment data.

## Card money transfer {#cards}

Makes money transfer to Visa, MasterCard, or MIR credit cards. To make sure you choose the correct identifier, check [provider ID by card number](#card_check) first.

<span style="font-weight:bold;color:red;">Money transfers to Visa and MasterCard cards issued by foreign banks are suspended due to restrictions from the payment system</span>.

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1963/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  -d '{ \
        "id":"21131343", \
        "sum":{ \
          "amount":1000, \
          "currency":"643" \
        }, \
        "paymentMethod":{ \
          "type":"Account", \
          "accountId":"643" \
        }, \
        "fields": { \
          "account":"4256********1231" \
        } \
      }'
~~~

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1960/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  -d '{ \
        "id":"21131343", \
        "sum":{ \
          "amount":1000, \
          "currency":"643"
        }, \
        "paymentMethod":{ \
          "type":"Account", \
          "accountId":"643" \
        }, \
        "fields": { \
          "account": "402865XXXXXXXXXX", \
          "rec_address": "Ленинский проспект 131, 56", \
          "rec_city": "Москва", \
          "rec_country": "Россия", \
          "reg_name": "Виктор", \
          "reg_name_f": "Петров", \
          "rem_name": "Сергей", \
          "rem_name_f": "Иванов" \
        } \
      }'
~~~

~~~http
POST /sinap/api/v2/terms/1963/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com

{
  "id":"21131343",
  "sum": {
        "amount":1000,
        "currency":"643"
  },
  "paymentMethod": {
      "type":"Account",
      "accountId":"643"
  },
  "fields": {
        "account":"4256XXXXXXXX1231"
  }
}
~~~

~~~http
POST /sinap/api/v2/terms/1960/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com

{
  "id":"21131343",
  "sum": {
        "amount":1000,
        "currency":"643"
  },
  "paymentMethod": {
      "type":"Account",
      "accountId":"643"
  },
  "fields": {
      "account": "402865XXXXXXXXXX",
      "rec_address": "Ленинский проспект 131, 56",
      "rec_city": "Москва",
      "rec_country": "Россия",
      "reg_name": "Виктор",
      "reg_name_f": "Петров",
      "rem_name": "Сергей",
      "rem_name_f": "Иванов"
  }
}
~~~

~~~python
import requests
import time

# Card money transfer
def send_card(api_access_token, payment_data):
    # payment_data - dictionary with all payment data
    s = requests.Session()
    s.headers['Accept'] = 'application/json'
    s.headers['Content-Type'] = 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    postjson = {"id":"","sum": {"amount":"","currency":"643"},"paymentMethod": {"type":"Account","accountId":"643"},"fields": {"account":""}}
    postjson['id'] = str(int(time.time() * 1000))
    postjson['sum']['amount'] = payment_data.get('sum')
    postjson['fields']['account'] = payment_data.get('to_card')
    prv_id = payment_data.get('prv_id')
    if payment_data.get('prv_id') in ['1960', '21012']:
        postjson['fields']['rem_name'] = payment_data.get('rem_name')
        postjson['fields']['rem_name_f'] = payment_data.get('rem_name_f')
        postjson['fields']['reg_name'] = payment_data.get('reg_name')
        postjson['fields']['reg_name_f'] = payment_data.get('reg_name_f')
        postjson['fields']['rec_city'] = payment_data.get('rec_address')
        postjson['fields']['rec_address'] = payment_data.get('rec_address')
        
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/' + prv_id + '/payments', json = postjson)
    return res.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/<a>ID</a>/payments</span></h3>
    </li>
</ul>

**ID** — QIWI provider identifier. Possible values:

* 1963 — Visa money transfer to cards issued by Russian banks only.
* 21013 — MasterCard money transfer to cards issued by Russian banks only.
* 1960 — Visa money transfer to credit cards issued by banks in Albania, Andorra, Argentina, Armenia, Australia, Austria, Azerbaijan, Belarus, Belgium, Benin, Bosnia and Herzegovina, Brazil, Bulgaria, China, Croatia, Cyprus, Czech Republic, Denmark, Egypt, Estonia, Finland, France, Georgia, Germany , Greece, Hong Kong (China), Hungary, Iceland, India, Indonesia, Israel, Italy, Japan, Kazakhstan, Kenya, Korea Republic, Kuwait, Kyrgyzstan, Latvia, Lithuania, Luxembourg, Macao, China, Macedonia, Madagascar, Malaysia, Maldives , Malta, Republic of Moldova, Monaco, Mongolia, Montenegro, Namibia, Netherlands, New Zealand, Nigeria, Norway, Oman, Paraguay, Poland, Portugal, Qatar, Romania, Saudi Arabia, Republic of Serbia, Singapore, Slovakia, Slovenia, South Africa, Spain, Sri Lanka, Sweden, Tajikistan, Tanzania, Thailand, Turkey, Turkmenistan, United Arab Emirates, United Kingdom, Uzbekistan, Vietnam, Zambia.
* 21012 — MasterCard money transfer to credit cards issued by banks in Albania, Argentina, Armenia, Australia, Austria, Azerbaijan, Bangladesh, Barbados, Belarus, Belgium, Benin, Bosnia and Herzegovina, Burkina Faso, Brazil, Bulgaria, Cambodia, Cameroon United Republic, Chile, China, Colombia, Congo, Costa Rica, Croatia, Cyprus, Czech Republic, Democratic Republic of the Congo, Denmark, Dominican Republic, Ecuador, El Salvador, Egypt, Estonia, Finland, France, Georgia, Germany, Ghana, Greece, Guatemala, Hong Kong, Hungary, India, Indonesia, Ireland, Israel, Italy, Japan, Jordan, Kazakhstan, Kenya, Korea, Kuwait, Kyrgyzstan, Latvia, Lebanon, Lithuania, Luxembourg, Macao, Macedonia, Madagascar, Malaysia, Maldives, Malta, Mexico, Moldova, Monaco, Mongolia, Montenegro, Morocco, Namibia, Nigeria, Nepal, Netherlands, New Zealand, Nigeria, Norway, Oman, Panama, Paraguay, Peru, Philippines, Poland, Portugal, Romania, Qatar, Russian Federation, Saudi Arabia, Senegal, Serbia Republic, Singapore, Slovakia, Slovenia, South Africa, Spain, W ri-Lanka, Sweden, Switzerland, Tajikistan, Tanzania, Thailand, Tunisia, Turkey, Turkmenistan, United Arab Emirates, United Kingdom, Uzbekistan, Vietnam, Zambia.
* 31652 — Transfer to MIR national payment system cards.
* 22351 — Transfer to [QIWI Virtual Card](https://qiwi.com/cards/qvc).

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
<li><h3>Parameters</h3>
</li>
</ul>

Send JSON-object [Payment](#payment_obj) in the request' body. Payment requisites in `fields` object depend on the provider ID.

### For ID 1963, 21013, 31652, 22351

Name | Type | Description
--------|----|----
fields.account| String| Recipient's card number (no spaces)

### For ID 1960, 21012

Name | Type | Description
--------|----|----
fields.account| String| Recipient's card number (no spaces)
fields.rem_name|String| Sender's first name
fields.rem_name_f|String| Sender's last name. Required for ID  `1960`, `21012` only
fields.rec_address|String| Sender's address (no zip code, only text)
fields.rec_city|String| Sender's city
fields.rec_country|String| Sender's country
fields.reg_name|String| Recipient's first name
fields.reg_name_f|String| Recipient's last name

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "21131343",
  "terms": "1963",
  "fields": {
          "account": "4256********1231"
    },
    "sum": {
         "amount": 1000,
         "currency": "643"
    },
    "source": "account_643",
    "transaction": {
         "id": "4969142201",
         "state": {
            "code": "Accepted"
          }
    }
}
~~~

~~~python
payment_data = {'prv_id': '1963', 'to_card' : '41548XXXXXXXX008', 'sum': 100}

jss = send_card(token, payment_data)
~~~

Successful response contains JSON-object [PaymentInfo](#payment_info) with accepted payment data.

## Bank money transfer {#banks}

Makes money transfer to personal bank cards/accounts opened in Russian banks.

### Transfer to card number {#card-transfer-bank}

Makes money transfer to cards issued by Russian banks.

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/464/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  -d '{ \
        "id":"21131343", \
        "sum": { \
          "amount":1000, \
          "currency":"643" \
        }, \
        "paymentMethod": { \
          "type":"Account", \
          "accountId":"643" \
        }, \
        "fields": { \
          "account_type": "1", \
          "account":"4256********1231", \
          "exp_date": "0623" \
        } \
      }'
~~~

~~~http
POST /sinap/api/v2/terms/464/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com

{
  "id":"21131343",
  "sum": {
        "amount":1000,
        "currency":"643"
  },
  "paymentMethod": {
      "type":"Account",
      "accountId":"643"
  },
  "fields": {
        "account":"4256********1231",
        "account_type": "1",
        "exp_date": "0623"
  }
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/<a>ID</a>/payments</span></h3>
        <ul>
             <li><strong>ID</strong> - QIWI provider identifier. Possible values:
             <ul><li>464 - Alfa Bank</li>
             <li>804 - OTP Bank</li>
             <li>810 - Rosselkhozbank</li>
             <li>815 - Russkiy Standart</li>
             <li>816 - VTB</li>
             <li>821 - Promsvyazbank</li>
             <li>870 - Sberbank</li>
             <li>881 - Renaissance Credit</li>
             <li>1134 - Moskovskiy Creditnyi Bank</li>
             </ul></li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
<li><h3>Parameters</h3>
</li>
</ul>

Send JSON object [Payment](#payment_obj) in the request's body. Payment requisites in `fields` object:

Name | Type | Description
--------|----|----
fields.account| String| Recipient's card number (no spaces)
fields.exp_date| String| Card expiry date, as `MMYY` (for example, `0218`). **Parameter is required for ID 464 and 821.**
fields.account_type| String| Bank identifier type. For each bank specific value applies:<br>ID 464 - `1`<br>ID 084 - `1`<br>ID 815 - `1`<br>ID 810 - `5`<br>ID 816 - `5`<br>ID 821 - `7`<br>ID 870 - `5`<br>ID 1134 - `5`<br>ID 881 - `1`.
fields.mfo| String| Bank MFO (BIK)
fields.lname|String| Recipient's last name
fields.fname|String| Recipient's first name
fields.mname|String| Recipient's middle name

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "21131343",
  "terms": "464",
  "fields": {
      "account": "4256********1231",
      "account_type": "1",
      "exp_date": "0423"
  },
  "sum": {
      "amount": 1000,
      "currency": "643"
  },
  "source": "account_643",
  "transaction": {
      "id": "4969142201",
      "state": {
        "code": "Accepted"
      }
  }
}
~~~

Successful response contains JSON-object [PaymentInfo](#payment_info) with accepted payment data.

### Transfer to bank account {#transfer-bank-account}

Makes money transfer to personal accounts opened in Russian banks. You can use quick transfer service (within an hour if transaction is made from 9:00 until 19:30).

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/816/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  -d '{ \
        "id":"21131343", \
        "sum": { \
          "amount":1000, \
          "currency":"643" \
        }, \
        "paymentMethod": { \
          "type":"Account", \
          "accountId":"643" \
        }, \
        "fields": { \
          "account_type": "2", \
          "urgent": "0", \
          "lname": "Иванов", \
          "fname": "Иван", \
          "mname": "Иванович", \
          "mfo": "046577795", \
          "account":"40817***" \
        } \
      }'
~~~

~~~http
POST /sinap/api/v2/terms/816/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com

{
  "id":"21131343",
  "sum": {
        "amount":1000,
        "currency":"643"
  },
  "paymentMethod": {
      "type":"Account",
      "accountId":"643"
  },
  "fields": {
          "account_type": "2",
          "urgent": "0",
          "lname": "Иванов",
          "fname": "Иван",
          "mname": "Иванович",
          "mfo": "046577795",
          "account":"40817***"
  }
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/<a>ID</a>/payments</span></h3>
        <ul>
             <li><strong>ID</strong> - QIWI provider identifier. Possible values:
             <ul><li>313 - HomeCredit Bank</li>
             <li>464 - Alfa Bank</li>
             <li>821 - Promsvyazbank</li>
             <li>804 - OTP Bank</li>
             <li>810 - Rosselkhozbank</li>
             <li>816 - VTB</li>
             <li>819 - Unicredit Bank</li>
             <li>868 - QIWI Bank</li>
             <li>870 - Sberbank</li>
             <li>815 - Russkiy Standart</li>
             <li>881 - Renaissance Credit</li>
             <li>1134 - Moskovskiy Creditnyi Bank</li>
             <li>27324 - Raiffeisen Bank</li>
             </ul></li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
<li><h3>Parameters</h3>
</li>
</ul>

Send JSON object [Payment](#payment_obj) in the request's body. Payment requisites in `fields` object:

Name | Type | Description
--------|----|----
fields.account| String| Recipient's bank account number
fields.urgent | String | Quick transfer flag. For `0` - not used; for `1` - make quick transfer by Urgent transfer service of Central Bank of Russia. **Extra commission is paid for quick transfer**
fields.mfo| String| Bank MFO (BIK)
fields.account_type| String| Bank identifier type. For each bank specific value applies:<br>ID 464 - `2`<br>ID 804 - `2`<br>ID 810 - `2`<br>ID 815 - `2`<br>ID 816 - `2`<br>ID 821 - `9`<br>ID 819 - `2`<br>ID 868 - `2`<br>ID 870 - `2`<br>ID 1134 - `2`<br>ID 27324 -`2`<br>ID 810 - `2`<br>ID 816 - `5`<br>ID 821 - `9`<br>ID 881 - `2`<br>ID 313 - `6`.
fields.lname|String| Recipient's last name
fields.fname|String| Recipient's first name
fields.mname|String| Recipient's middle name
fileds.agrnum|String| Bank agreement number - for ID 313

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "21131343",
  "terms": "464",
  "fields": {
          "account": "407121010910909011",
          "account_type": "2"
  },
  "sum": {
         "amount": 1000,
         "currency": "643"
  },
  "source": "account_643",
  "transaction": {
         "id": "4969142201",
         "state": {
            "code": "Accepted"
          }
  }
}
~~~

Successful response contains JSON-object [PaymentInfo](#payment_info) with accepted payment data.

## Other services {#services}

You can pay for services by user identifier. This request applies for QIWI providers with one user identifier and without requirement of account number online check.

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/674/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  -d '{ \
        "id":"21131343", \
        "sum": { \
          "amount":100, \
          "currency":"643" \
        },\
        "paymentMethod": { \
          "type":"Account", \
          "accountId":"643" \
        }, \
        "fields": { \
          "account":"111000000" \
        } \
      }'
~~~

~~~http
POST /sinap/api/v2/terms/674/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com

{
  "id":"21131343",
  "sum": {
        "amount":100,
        "currency":"643"
  },
  "paymentMethod": {
      "type":"Account",
      "accountId":"643"
  },
  "fields": {
        "account":"111000"
  }
}
~~~

~~~python
import requests
import time

# payment for service provider

def pay_simple_prv(api_access_token, prv_id, to_account, sum_pay):
    s = requests.Session()
    s.headers['Accept'] = 'application/json'
    s.headers['Content-Type'] = 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    postjson = {"id":"","sum": {"amount":"","currency":"643"},"paymentMethod": {"type":"Account","accountId":"643"},"fields": {"account":""}}
    postjson['id'] = str(int(time.time() * 1000))
    postjson['sum']['amount'] = sum_pay
    postjson['fields']['account'] = to_account
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/'+prv_id+'/payments', json = postjson)
    return res.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/<a>ID</a>/payments</span></h3>
        <ul>
             <li><strong>ID</strong> - QIWI provider identifier. Possible values:
             <ul><li>674 - OnLime</li>
             <li>Other Internet providers</li>
             <li>1239 - Podari zhizn Charitable Foundation</li>
             <li>Other charitable foundations identifiers</li>
             <li><a href="#provider-search">Find a service provider identifier</a></li>
             </ul></li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
<li><h3>Parameters</h3>
</li>
</ul>

Send JSON object [Payment](#payment_obj) in the request's body. Payment requisites in `fields` object:

Name | Type | Description
--------|----|----
fields.account| String| User identifier

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "21131343",
  "terms": "674",
  "fields": {
          "account": "111000"
  },
  "sum": {
         "amount": 100,
         "currency": "643"
  },
  "source": "account_643",
  "transaction": {
         "id": "4969142201",
         "state": {
            "code": "Accepted"
          }
  }
}
~~~

Successful response contains JSON-object [PaymentInfo](#payment_info) with accepted payment data.

## Payment by any requisites {#freepay}

Makes payments for commercial services by their bank details.

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1717/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  --header "User-Agent: ***" \
  -d '{ \
  "id":"21131343", \
  "sum": { \
        "amount":1000, \
        "currency":"643" \
  }, \
  "paymentMethod": { \
      "type":"Account", \
      "accountId":"643" \
  }, \
  "fields": { \
         "extra_to_bik":"044525201", \
         "requestProtocol":"qw1", \
         "city":"МОСКВА", \
         "name":"ПАО АКБ \"АВАНГАРД\"", \
         "to_bik":"044525201", \
         "urgent":"0", \
         "to_kpp":"772111001", \
         "is_commercial":"1", \
         "nds":"НДС не облагается", \
         "goal":" Оплата товара по заказу №090738231", \
         "from_name_p":"Николаевич", \
         "from_name":"Иван", \
         "from_name_f":"Михайлов", \
         "info":"Коммерческие организации", \
         "to_name":"ООО \"Технический Центр ДЕЛЬТА\"", \
         "to_inn":"7726111111", \
         "account":"40711100000012321", \
         "toServiceId":"1717" \
  } \
}'
~~~

~~~http
POST /sinap/api/v2/terms/1717/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
User-Agent: ****

{
  "id":"21131343",
  "sum": {
        "amount":1000,
        "currency":"643"
  },
  "paymentMethod": {
      "type":"Account",
      "accountId":"643"
  },
  "fields": {
         "extra_to_bik":"044525201",
         "requestProtocol":"qw1",
         "city":"МОСКВА",
         "name":"ПАО АКБ \"АВАНГАРД\"",
         "to_bik":"044525201",
         "urgent":"0",
         "to_kpp":"772111001",
         "is_commercial":"1",
         "nds":"НДС не облагается",
         "goal":" Оплата товара по заказу №090738231",
         "from_name_p":"Николаевич",
         "from_name":"Иван",
         "from_name_f":"Михайлов",
         "info":"Коммерческие организации",
         "to_name":"ООО \"Технический Центр ДЕЛЬТА\"",
         "to_inn":"7726111111",
         "account":"40711100000012321",
         "toServiceId":"1717"
  }
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/1717/payments</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
<li><h3>Parameters</h3>
</li>
</ul>

Send JSON object [Payment](#payment_obj) in the request's body. Payment requisites in `fields` object:

Name | Type | Description
--------|----|----
fields.name|String|Recipient's bank name (escape quotes with `\`)
fields.extra_to_bik|String| Recipient's bank MFO (BIK)
fields.to_bik|String| Recipient's bank MFO (BIK)
fields.city|String| Recipient's city of placement
fields.info|String| Constant, `Коммерческие организации` (in Russian)
fields.is_commercial|String| Service info, constant `1`
fields.to_name|String| Recipient's organization name  (escape quotes with `\`)
fields.to_inn|String| Organization's TIN
fields.to_kpp|String| Organization's KPP (code for the reason in the tax service regisration)
fields.nds|String| Value-added tax flag. If you pay for invoice and there is no VAT, then put the string `НДС не облагается` (in Russian). Otherwise, put the string `В т.ч. НДС` (in Russian).
fields.goal|String| Payment appointment
fields.urgent|String| Urgent payment (`0` - no, `1` - yes). Urgent payment is made in 10 minutes or more. It is applicable for weekdays from 9:00 to 20:30, Moscow time zone. Extra fee for the service is 25 rubles.
fields.account| String| Recipient's account number
fields.from_name|String| Recipient's first name
fields.from_name_p|String| Recipient's middle name
fields.from_name_f|String| Recipient's last name
fields.requestProtocol|String| Service info, constant `qw1`
fields.toServiceId|String| Service info, QIWI provider ID `1717`

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "21131343",
  "terms": "1717",
  "fields": {
         "extra_to_bik":"044525201",
         "requestProtocol":"qw1",
         "city":"МОСКВА",
         "name":"ПАО АКБ \"АВАНГАРД\"",
         "to_bik":"044525201",
         "urgent":"0",
         "to_kpp":"772111001",
         "is_commercial":"1",
         "nds":"НДС не облагается",
         "goal":" Оплата товара по заказу №090738231",
         "from_name_p":"Николаевич",
         "from_name":"Иван",
         "from_name_f":"Михайлов",
         "info":"Коммерческие организации",
         "to_name":"ООО \"Технический Центр ДЕЛЬТА\"",
         "to_inn":"7726111111",
         "account":"40711100000012321",
         "toServiceId":"1717"
  },
  "sum": {
         "amount": 1000,
         "currency": "643"
  },
  "source": "account_643",
  "transaction": {
         "id": "10969142201",
         "state": {
            "code": "Accepted"
          }
  }
}
~~~

Successful response contains JSON-object [PaymentInfo](#payment_info) with accepted payment data.

## QIWI provider search {#search}

### Search by string {#provider-search}

Performs search of QIWI provider's ID for [payment methods](#services) by keywords (for example, provider's name).

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://qiwi.com/search/results/json.action?searchPhrase=%D0%91%D0%B8%D0%BB%D0%B0%D0%B9%D0%BD+%D0%B4%D0%BE%D0%BC%D0%B0%D1%88%D0%BD%D0%B8%D0%B9+%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D0%BD%D0%B5%D1%82" \
  --header "Accept: application/json"
~~~

~~~http
POST /search/results/json.action?searchPhrase=%D0%91%D0%B8%D0%BB%D0%B0%D0%B9%D0%BD+%D0%B4%D0%BE%D0%BC%D0%B0%D1%88%D0%BD%D0%B8%D0%B9+%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D0%BD%D0%B5%D1%82 HTTP/1.1
Accept: application/json
Host: qiwi.com
~~~

~~~python
import requests

# provider id by its name
def qiwi_com_search(search_phrase):
    s = requests.Session()
    search = s.post('https://qiwi.com/search/results/json.action', params={'searchPhrase':search_phrase})
    return search.json()['data']['items']
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://qiwi.com/search/results/json.action?<a>searchPhrase={value}</a></span></h3>
        <ul>
             <li><strong>searchPhrase</strong> - keywords for provider's searching.</li>
        </ul>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
  
{
  "data": {
    ...
    "items": [
      {
        "item": {
          "id": {
            "id": "120",
            ...
          },
          ...
        },
        ...
      }
    ]
  }
}
~~~

~~~python
# get Beeline Internet  provider ID
prv = qiwi_com_search('Билайн домашний интернет')[0]['item']['id']['id']
print(prv)
~~~

Successful JSON-response contains IDs of the found QIWI providers:

Response field | Type | Description
-----|----|-----
data.items | Array | List of providers
items[].item.id.id | String | Provider's ID in the array's element

### Mobile network operator {#mnp}

Determines mobile network operator ID by the client's mobile number. Response returns QIWI provider ID for using in the API method of the client's [mobile phone replenishment](#cell).

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://qiwi.com/mobile/detect.action" \
  --header "Accept: application/json" \
  --header "Content-Type: application/x-www-form-urlencoded" \
  -d "phone=79651238341"
~~~

~~~http
POST /mobile/detect.action HTTP/1.1
Host: qiwi.com
Accept: application/json
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache

phone=79651238341
~~~

~~~python
import requests

def mobile_operator(phone_number):
    s = requests.Session()
    res = s.post('https://qiwi.com/mobile/detect.action', data = {'phone': phone_number })
    s.headers['Accept'] = 'application/json'
    s.headers['Content-Type'] = 'application/x-www-form-urlencoded'
    return res.json()['message']
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://qiwi.com/mobile/detect.action</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/x-www-form-urlencoded</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Send parameter in the request's body as `formdata`:

Name | Type | Description
--------|----|----
phone | String URL-encoded | Client's mobile phone number, international format without `+`. Required.

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "code": {
    "value": "0",
    "_name": "NORMAL"
  },
  "data": null,
  "message": "3",
  "messages": null
}
~~~

> Cannot determine the mobile operator

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "code": {
    "value": "2",
    "_name": "ERROR"
  },
 "data": null,
 "message": "По указанному номеру невозможно определить оператора сотовой связи. Воспользуйтесь поиском.",
 "messages": {}
}
~~~

~~~python
print(mobile_operator('79652468447'))
~~~

Response with HTTP Status 200 and `code.value` = 0 means successful operator determination. QIWI provider ID is the value of `message` field.

Response with HTTP Status 200 and `code.value` = 2 means that operator determination is not possible.

### Card transfer provider {#card_check}

Gets QIWI provider ID for [money transfer to credit card](#cards).

No authorization is required.

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://qiwi.com/card/detect.action" \
  --header "Accept: application/json" \
  --header "Content-Type: application/x-www-form-urlencoded" \
  -d "cardNumber=4256********1231"
~~~

~~~http
POST /card/detect.action HTTP/1.1
Host: qiwi.com
Accept: application/json
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache

cardNumber=4256********1231
~~~

~~~python
import requests

def card_system(card_number):
    s = requests.Session()
    res = s.post('https://qiwi.com/card/detect.action', data = {'cardNumber': card_number })
    return res.json()['message']
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://qiwi.com/card/detect.action</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/x-www-form-urlencoded</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Send parameter in the request's body as `formdata`:

Name | Type | Description
--------|----|----
cardNumber | String | Full card number (no spaces). Required

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "code": {
    "value": "0",
    "_name": "NORMAL"
  },
  "data": null,
  "message": "1963",
  "messages": null
}
~~~

~~~python
print(card_system('4890xxxxxxxx1698'))
~~~

> Cannot get provider ID for credit card money transfer

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "code": {
        "value": "2",
        "_name": "ERROR"
    },
    "data": null,
    "message": "Неверно введен номер банковской карты. Попробуйте ввести номер еще раз.",
    "messages": {}
}
~~~

Response with HTTP Status 200 and `code.value` = 0 means successful ID determination. QIWI provider ID is the value of `message` field.

Response with HTTP Status 200 and `code.value` = 2 means that ID determination is not possible (wrong card number or payment system is not supported).

## API data models

### Payment class {#payment_obj}

~~~json
{
  "id":"21131343",
  "sum": {
        "amount":1000,
        "currency":"643"
  },
  "paymentMethod": {
      "type":"Account",
      "accountId":"643"
  },
  "fields": {
         "extra_to_bik":"044525201",
         "requestProtocol":"qw1",
         "city":"МОСКВА",
         "name":"ПАО АКБ \"АВАНГАРД\"",
         "to_bik":"044525201",
         "urgent":"0",
         "to_kpp":"772111001",
         "is_commercial":"1",
         "nds":"НДС не облагается",
         "goal":" Оплата товара по заказу №090738231",
         "from_name_p":"Николаевич",
         "from_name":"Иван",
         "from_name_f":"Михайлов",
         "info":"Коммерческие организации",
         "to_name":"ООО \"Технический Центр ДЕЛЬТА\"",
         "to_inn":"7726111111",
         "account":"40711100000012321",
         "toServiceId":"1717"
  }
}
~~~

Object describes payment data for QIWI Wallet provider.

Name|Type|Description|Required
--------|----|----|------
id | String | Client transaction ID (max 20 digits). Must be unique for each transaction. Increment with each following transaction. To satisfy these requirements, set it to 1000*(Standard Unix time in seconds).|+
sum|Object| Payment amount data
---|--|--
sum.amount|Number|Payment amount value (rubles and kopeks, separator `.`). Positive number rounded down to 2 decimals. If you specify more decimals, our system will round the number down to the same precision.|+
sum.currency|String|Payment currency (only rubles, `643`)|+
-----|-----|-----
paymentMethod | Object| QIWI wallet account to fund the payment
-----|-----|-----
paymentMethod.type|String |Constant, `Account`|+
paymentMethod.accountId|String| Constant, `643`|+
-----|-----|-----
fields|Object| Payment requisites. Object fields depend on provider ID.
comment|String| Payment comment. Used for [QIWI wallet transfer](#p2p) or  [conversion](#CCY) only |-

### PaymentInfo class {#payment_info}

~~~json
{
    "id": "150217833198900",
    "terms": "99",
    "fields": {
        "account": "79121238345"
    },
    "sum": {
        "amount": 100,
        "currency": "643"
    },
    "transaction": {
        "id": "11155897070",
        "state": {
            "code": "Accepted"
        }
    },
    "source": "account_643",
    "comment": "My comment"
}
~~~

Object describes QIWI wallet transaction data and returns in response from Payment API.

Name|Type|Description
-----|----|-----
id | Number | `id` parameter from the original request
terms | String | QIWI provider ID used for the payment
fields|Object| `fields` object from the original request. **Card number returns in masked form**
sum|Object| `sum` object from the original request
source| String| Always constant, `account_643`
comment| String | `comment` parameter from the original request (if exists in the request)
transaction|Object| Object with QIWI transaction data
---|---|---
transaction.id|String|QIWI transaction ID
transaction.state|Object|Current state of the transaction
---|---|---
state.code | String| Current status of the transaction. Only `Accepted` is returned (it means that the payment is accepted for processing). Actual transaction status can be obtained from [Payments history API](#payments_history).

# Invoices {#invoices}

Invoice is the universal request for payment or money transfer in QIWI Wallet system.

API provides operations of [invoice creation](#invoice) (only P2P invoices for money transfer to another QIWI wallet), [payment](#paywallet_invoice), [rejection](#cancel_invoice), and also [method for requesting list of unpaid invoices](#list_invoice) issued to your QIWI wallet.

## Invoice creation and P2P token {#invoice}

You can issue invoices to any QIWI wallet by using [P2P invoices API](/en/p2p-payments/#create). Use [special P2P token](#p2p-token) for authorization in P2P invoices API.

### How to get P2P token {#p2p-token}

Authorize on [p2p.qiwi.com](https://p2p.qiwi.com), or use the given request. You may also specify [Invoice payment callbacks URL](/en/p2p-payments/#notification) in this request.

The method returns P2P tokens for using with [Payment form](/en/p2p-payments/#http) and with P2P API in `PublicKey` and `SecretKey` fields of the response, respectively.

Use [QIWI Wallet API token](#auth_data) for authorization.

<h3 class="request method">Request → POST</h3>

~~~shell
curl -X POST \
  https://edge.qiwi.com/widgets-api/api/p2p/protected/keys/create \
  -H 'Authorization: Bearer ec74********' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{"keysPairName":"Name","serverNotificationsUrl":"https://test.com"}'
~~~

~~~http
POST /widgets-api/api/p2p/protected/keys/create HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
Content-Type: application/json
User-Agent: ****

{"keysPairName":"Name", "serverNotificationsUrl":"https://test.com"}
~~~

<ul class="nestedList url">
<li><h3>URL <span>/widgets-api/api/p2p/protected/keys/create</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Authorization: Bearer XXX</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Send parameters in JSON-body:

Name|Type|Description
--------|----|----
keysPairName| String| Name for the couple of P2P tokens
serverNotificationsUrl|String | [Invoice payment callbacks URL](/en/p2p-payments/#notification) (optional)

## List of invoices  {#list_invoice}

Returns unpaid invoices issued to your wallet. Invoices are placed in the list in reverse chronological order.

By default, the list is paginated on 50 elements on each page. You can specify the number of elements on each page.
You may use filters: invoice creation period of dates and starting invoice ID.

<h3 class="request method">Request → GET</h3>

~~~shell
user@server:~$ curl -X GET --header 'Accept: application/json' \
   --header 'Authorization: Bearer ***' \
   'https://edge.qiwi.com/checkout-api/api/bill/search?statuses=READY_FOR_PAY&rows=50'
~~~

~~~http
GET /checkout-api/api/bill/search?statuses=READY_FOR_PAY&rows=50 HTTP/1.1
Accept: application/json
Authorization: Bearer ***
Host: edge.qiwi.com
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/checkout-api/api/bill/search?statuses=READY_FOR_PAY&<a>parameter=value</a></span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Authorization: Bearer XXX (<a href="#p2p-token">P2P API Token</a>)</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Send parameters in the request query URL:

Name|Type|Description
--------|----|----
statuses|String| Unpaid invoice status. Only `READY_FOR_PAY`. Required parameter.
rows | Integer | Maximum number of invoices to be returned, for list pagination. Integer number from 1 to 50. Default value is 50.
min_creation_datetime|Long| Lower time limit for invoice search, Unix-time
max_creation_datetime|Long| Greater time limit for invoice search, Unix-time
next_id|Number| Starting invoice identifier for invoice search. Only invoices with IDs equal or smaller than this ID are returned. Use it for obtaining next invoices in the paginated list.
next_creation_datetime|Long| Starting date for invoice search, Unix-time. Only invoices created before this date are returned. Use it for obtaining next invoices in the paginated list.

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "bills": [
    {
      "id": 1063702405,
      "external_id": "154140605",
      "creation_datetime": 1523025585000,
      "expiration_datetime": 1523026003808,
      "sum": {
        "currency": 643,
        "amount": 100
      },
      "status": "READY_FOR_PAY",
      "type": "MERCHANT",
      "repetitive": false,
      "provider": {
        "id": 480706,
        "short_name": "Интернет-магазин цветов",
        "long_name": "ООО «Цветы»",
        "logo_url":"https://static.qiwi.com/img/providers/logoBig/480706_l.png"
      },
      "comment": "Пополнение счета 13515573",
      "pay_url":"https://oplata.qiwi.com/form?shop=480706&transaction=102263702405"
    }
  ]
}
~~~

Successful JSON response includes a list of unpaid invoices according to the conditions of the request:

<a name="invoice_data"></a>

Response field| Type| Description
--------|----|----
bills|Array[Object]|List of invoices.<br>List length is `rows` parameter from the original request, or 50, if it is absent
bills[].id|Integer| QIWI Wallet invoice ID
bills[].external_id|String| Merchant's invoice ID
bills[].creation_datetime|Long| Date/time of the invoice creation, Unix-time
bills[].expiration_datetime|Long| Date/time of the invoice expiration, Unix-time
bills[].sum|Object| Invoice amount data
---|----|----
sum.currency|Integer| Invoice currency
sum.amount|Number| Invoice amount
---|----|----
bills[].status|String | Constant, `READY_FOR_PAY`
bills[].type|String| Constant, `MERCHANT`
bills[].repetitive|Boolean|Service data
bills[].provider|Object| Merchant information
---|----|----
provider.id|Integer| QIWI identifier
provider.short_name|String| Short name
provider.long_name|String| Full name
provider.logo_url|String| Logo URL
---|----|----
bills[].comment|String| Invoice comment
bills[].pay_url|String| URL to pay for the invoice on QIWI Payment Form

## Invoice payment {#paywallet_invoice}

Makes invoice payment immediately without SMS confirmation.

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST --header 'Content-Type: application/json;charset=UTF-8' \
   --header 'Accept: application/json' \
   --header 'Authorization: Bearer 68ec21fd52e4244838946dd07ed225a1' \
   -d '{ \
         "invoice_uid": "1063702405", \
         "currency": "643" \
        }' 'https://edge.qiwi.com/checkout-api/invoice/pay/wallet'
~~~

~~~http
POST /checkout-api/invoice/pay/wallet HTTP/1.1
Accept: application/json
Content-type: application/json
Authorization: Bearer ***
Host: edge.qiwi.com
User-Agent: ****

{
   "invoice_uid": "1063702405",
   "currency": "643"
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/checkout-api/invoice/pay/wallet</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer XXX (<a href="#p2p-token">P2P API Token</a>)</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3>
    </li>
</ul>

Required parameters in JSON body:

Name|Type|Description
--------|----|----
invoice_uid | String |QIWI invoice ID (take it from `bills[].id` field of [invoice data](#invoice_data)
currency|String| Invoice currency (take it from `bills[].sum.currency` field of [invoice data](#invoice_data))

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "invoice_status": "PAID_STATUS",
  "is_sms_confirm": false,
  "WALLET_ACCEPT_PAY_RESULT": {}
}
~~~

Successful response contains JSON with paid invoice status:

Field|Type|Description
--------|----|----
invoice_status|String|Invoice payment status, `PAID_STATUS`. Any other status means unsuccessful transaction.
is_sms_confirm|String|SMS confirmation flag

## Unpaid invoice cancelling {#cancel_invoice}

Rejects an unpaid invoice. This makes the invoice unavailable for payment.

<h3 class="request method">Request → POST</h3>

~~~shell
user@server:~$ curl -X POST --header 'Accept: application/json' \
                            --header 'Authorization: Bearer ***' \
                            'https://edge.qiwi.com/checkout-api/api/bill/reject' \
                            -d '{ "id": 1034353453 }'
~~~

~~~http
POST /checkout-api/api/bill/reject HTTP/1.1
Accept: application/json
Authorization: Bearer ***
Content-type: application/json
Host: edge.qiwi.com
User-Agent: ****

{
  "id": 1034353453
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/checkout-api/api/bill/reject</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer XXX (<a href="#p2p-token">P2P API Token</a>)</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Parameter</h3>
    </li>
</ul>

Required parameter in JSON body:

Name|Type|Description
--------|----|----
id | Integer |Invoice ID to reject (take it from `bills[].id` field of [invoice data](#invoice_data)

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

Successful response has HTTP code 200.
