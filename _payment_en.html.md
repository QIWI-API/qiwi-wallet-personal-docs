# Payments API {#payments}

###### Last update: 2020-07-28 | [Edit on GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/_payment_en.html.md)

## Commission rates {#rates}

Use the method to get total commission amount for the payment by the given payment requisites.

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

<h3 class="request method">Запрос → POST</h3>

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/providers/<a>id</a>/onlineCommission</span></h3></li>
        <ul>
             <li><strong>id</strong> - provider's identifier. Possible values:
             <ul><li>99 - QIWI Wallet transfer</li>
             <li>1963 - Visa card transfer (issued by only Russian banks)</li>
             <li>21013 - MasterCard card transfer (issued by only Russian banks)</li>
             <li>For credit card issued by Azerbaijan, Armenia, Belarus, Georgia, Kazakhstan, Kyrgyzstan, Moldova, Tajikistan, Turkmenistan, Ukraine, Uzbekistan:<ul><li>1960 – Visa card transfer</li><li>21012 – MasterCard card transfer</li></ul></li>
             <li>31652 - national payment system MIR card transer</li>
             <li>466 - Tinkoff bank</li>
             <li>464 - Alpha bank</li>
             <li>821 - Promsvyazbank</li>
             <li>815 - Russkiy Standard</li>
             <li><a href="#banks">Other banks</a></li>
             <li><a href="#mnp">Mobile network operators</a></li>
             <li><a href="#search">Other providers</a></li>
             <li>1717 - payment by bank requisites</a></li></ul></li>
        </ul>
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
    <li><h3>Parameters</h3><span>Send it in JSON body. All parameters are required.</span>
    </li>
</ul>

Parameter|Type|Description
--------|----|----
account| String| User's identifier (phone number, card/account number, and other entity depending on provider)
paymentMethod | Object| Object defining payment processing by QIWI Wallet. Includes the following parameters:
paymentMethod.type|String | Payment method, `Account` only
paymentMethod.accountId|String| QIWI Wallet account identifier, `643` only.
purchaseTotals | Object | Object with payment requisites
purchaseTotals.total|Object| Payment amount data:
total.amount|Number | Amount (rubles and kopeks, divided by `.`). Positive number, rounded down to 2 decimals. If you send more decimals, value will be rounded down to kopeks.
total.currency|String|Currency (`643` only, that is rubles)

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

<h3 class="request">Response ←</h3>

Commission rate returns in `qwCommission.amount` field of the JSON response.

<a name="payform"></a>

## Payment form autofilling {#autocomplete}

The request provides autofilled payment form on qiwi.com site.

[Link example (click to see the form)](https://qiwi.com/payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&amountFraction=0&extra%5B%27comment%27%5D=test123&currency=643&blocked[0]=account)

If you don't want to show your wallet number, use transfer to wallet nickname:

[Link example for nickname transfer (click to see the form)](https://qiwi.com/payment/form/99999?extra%5B%27account%27%5D=NICKNAME&amountInteger=1&amountFraction=0&currency=643&blocked[0]=account&extra%5B%27accountType%27%5D=nickname)

To get payments to your QIWI Wallet, you can use also [P2P-form](https://qiwi.com/p2p-admin/transfers/link).

<h3 class="request method">Request → GET</h3>

~~~http
GET /payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&amountFraction=0&extra%5B%27comment%27%5D=test123&currency=643 HTTP/1.1
Host: qiwi.com

~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://qiwi.com/<a>ID</a>?<a>parameter=value</a></span></h3></li>
<ul>
<li><strong>id</strong> - provider's identifier. Possible values:
<ul><li>99 - QIWI Wallet transfer</li>
<li>99999 - QIWI Wallet nickname transfer</li>
<li>1963 - Visa card transfer (issued by only Russian banks)</li>
<li>21013 - MasterCard card transfer (issued by only Russian banks)</li>
<li>For credit card issued by Azerbaijan, Armenia, Belarus, Georgia, Kazakhstan, Kyrgyzstan, Moldova, Tajikistan, Turkmenistan, Ukraine, Uzbekistan:<ul><li>1960 – Visa card transfer</li><li>21012 – MasterCard card transfer</li></ul></li>
<li>31652 - national payment system MIR card transer</li>
<li>22351 - <a href="https://qiwi.com/cards/qvc">QIWI Virtual card</a> transfer</li>
<li><a href="#banks">Transfer to bank account</a></li>
<li><a href="#mnp">Mobile network operators</a></li>
<li><a href="#search">Other providers</a></li>
<li>1717 - payment by bank requisites</a></li></ul></li>
</ul></li></ul>

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Send in URL query to properly fill the form fields.</span></li>
</ul>

Parameter | Type | Description | Form field | Required
---------|--------|---|----|----
amountInteger|Integer | Integer part of the payment amount (in rubles). If absent, "Amount" field will be empty. **The number not greater than 99 999 (payment amount limit)** | Anmount | -
amountFraction|Integer | Fractional part of the payment amount (kopeks). If absent,  Дробная часть суммы платежа (копейки). Если параметр не указан, "Amount" field will be empty.|Сумма | -
currency|Constant string, `643` | Payment currency code. **Required if you send the payment amount in the link** |-|+
extra['comment'] |URL-encoded string | Payment comment. **Use it only for ID=99** | Comment to the transfer | -
extra['account'] |URL-encoded string |  Field format is the same as `fields.account` of the corresponding payment request: for provider `99` - recipient's wallet number; for mobile network operators - phone number for top-up (without international prefix); for card transfer - recipient's card number without spaces, for other providers - user's identifier. For provider `99999` use nickname or number of the wallet and use appropriate value of `extra['accountType']` parameter.| Wallet number, phone number, user ID.|-
blocked|Array[String]|Name of blocked (inactive) form field. User will not be able to change this field. Each parameter corresponds to respective form field and should be numbered starting from zero (`blocked[0]`, `blocked[1]`, and so on). If absent, user can change all form fields. Possible values:<br>`sum` - "Payment amount" field, <br>`account` - "Phone number/Account number" field,<br>`comment` - "Comment" field.<br> Example (for payment amount field): `blocked[0]=sum` |-|-
extra['accountType'] | URL-encoded string | **Use only for ID=99999**. The value determines transfer to QIWI wallet by nickname or wallet number. If you don't want to show your wallet number, use transfer to wallet nickname.<br>`phone` - for transfer to wallet number<br>`nickname` - for transfer to wallet nickname. 

### How to get your wallet nickname {#nickname}

Use the following request:

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
    <li><h3>Parameters</h3><span>Send JSON object [Payment](#payment_obj) in the request's body. Payment requisites in <code>fields</code> field:</span>
    </li>
</ul>

Parameter|Type|Description|Required
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

## Conversion {#CCY}

API method to transfer funds to currency account in QIWI wallet with conversion from your QIWI wallet ruble account. Two transactions are created: conversion between accounts of your QIWI wallet, and transfer to another wallet. You can get currency rates from [another API method](#exchange).

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
<li><h3>Parameters</h3><span>Send JSON-object [Payment](#payment_obj) in the request's body. Payment's requisites in <code>fields</code> JSON field:</span>
</li>
</ul>

Parameter|Type|Description|Required
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

## Currency rates {#exchange}

API method returns current QIWI Bank currency rates and cross-rates.

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

Successful response containt JSON array of currency rates in `result` field. An element of the list corresponds to currency pair:

Parameter|Type|Description
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

# Cell phone topup
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
    <li><h3>URL <span>/sinap/api/v2/terms/<a>ID</a>/payments</span></h3></li>
        <ul>
             <li><strong>ID</strong> - provider identifier. <a href="#mnp">Check mobile operator</a> to get the proper ID</li>
        </ul>
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
<li><h3>Parameters</h3><span>Send JSON-object [Payment](#payment_obj) in the request's body. Payments requisites in JSON-field <code>fields</code>:</span>
</li>
</ul>

Parameter|Type|Description
--------|----|----
fields.account| String|Cell phone number to topup (without `8` prefix)

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

This request makes money transfer to Visa, MasterCard, or MIR credit cards. You can preliminary check [card system](#card_check).

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

# Card money transer
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
    <li><h3>URL <span>/sinap/api/v2/terms/<a>ID</a>/payments</span></h3></li>
        <ul>
             <li><strong>ID</strong> - QIWI provider identifier. Possible values:
             <ul>
             <li>1963 - Visa card money transfer (issued by only Russian banks)</li>
             <li>21013 - MasterCard card money transfer (issued by only Russian banks)</li>
             <li>31652 - national payment system MIR card money transfer</li>
             <li>22351 - money transfer to <a href="https://qiwi.com/cards/qvc">QIWI Virtual Card</a></li>
             <li>For credit card issued by Azerbaijan, Armenia, Belarus, Georgia, Kazakhstan, Kyrgyzstan, Moldova, Tajikistan, Turkmenistan, Ukraine, Uzbekistan:<ul><li>1960 – Visa card money transfer</li><li>21012 – MasterCard card money transfer</li></ul></li>
        </ul>
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
<li><h3>Parameters</h3><span>Send JSON-object [Payment](#payment_obj) in the request' body. Payment requisites in <code>fields</code> parameter:</span>
</li>
</ul>

Parameter | Type | Description
--------|----|----
fields.account| String| Recipient's card number (no spaces)
fields.rem_name|String| Sender's first name. Required for ID  `1960`, `21012` only
fields.rem_name_f|String| Sender's last name. Required for ID  `1960`, `21012` only
fields.rec_address|String| Sender's address (no zip code, only text). Required for ID  `1960`, `21012` only
fields.rec_city|String| Sender's city. Required for ID  `1960`, `21012` only
fields.rec_country|String| Sender's country. Required for ID  `1960`, `21012` only
fields.reg_name|String| Recipient's first name. Required for ID  `1960`, `21012` only
fields.reg_name_f|String| Recipient's last name. Required for ID  `1960`, `21012` only

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

To make money transfer to bank cards/accounts of persons opened in Russian banks, use the following methods.

### Card nunmber transfer

The request makes money transfer to cards issued by Russian banks.

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
          "exp_date": "MMYY" \
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
        "exp_date": "MMYY"
  }
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/<a>ID</a>/payments</span></h3></li>
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
<li><h3>Parameters</h3><span>Send JSON-object [Payment](#payment_obj) in the request' body. Payment requisites in <code>fields</code> parameter:</span>
</li>
</ul>

Parameter | Type | Description
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
      "exp_date": "MMYY"
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


### Bank account transfer

The request makes money transfer to personal accounts opened in Russian banks. You can use quick transfer service (within an hour if transaction is made from 9:00 until 19:30).

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
    <li><h3>URL <span>/sinap/api/v2/terms/<a>ID</a>/payments</span></h3></li>
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
<li><h3>Parameters</h3><span>Send JSON-object [Payment](#payment_obj) in the request' body. Payment requisites in <code>fields</code> parameter:</span>
</li>
</ul>

Parameter | Type | Description
--------|----|----
fields.account| String| Recipient's bank account number
fields.urgent | String | Quick transfer flag. For `0` - not used; for `1` - make quick transfer by Urgent transfer service of Central Bank of Russia. **Extra commission is paid for quick transfer**
fields.mfo| String| Bank MFO (BIK)
fields.account_type| String| Bank identifier type. For each bank specfic value applies:<br>ID 464 - `2`<br>ID 804 - `2`<br>ID 810 - `2`<br>ID 815 - `2`<br>ID 816 - `2`<br>ID 821 - `9`<br>ID 819 - `2`<br>ID 868 - `2`<br>ID 870 - `2`<br>ID 1134 - `2`<br>ID 27324 -`2`<br>ID 810 - `2`<br>ID 816 - `5`<br>ID 821 - `9`<br>ID 881 - `2`<br>ID 313 - `6`.
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

You can pay for services by user identifier. This request applies for QIWI providers with one user identifier and without account number online check.

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
    <li><h3>URL <span>/sinap/api/v2/terms/<a>ID</a>/payments</span></h3></li>
        <ul>
             <li><strong>ID</strong> - QIWI provider identifier. Possible values:
             <ul><li>674 - OnLime</li>
             <li>Other Internet providers</li>
             <li>1239 - Podari zhizn Charitable Foundation</li>
             <li>Other charitable foundations identifiers</li>
             <li><a href="#search">How to find a service provider identifier</a></li>
             </ul></li>
        </ul>
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
<li><h3>Parameters</h3><span>Send JSON-object [Payment](#payment_obj) in the request' body. Payment requisites in <code>fields</code> parameter:</span>
</li>
</ul>

Parameter | Type | Description
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

This method makes payments for commercial services by their bank details.

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
<li><h3>Parameters</h3><span>Send JSON-object [Payment](#payment_obj) in the request' body. Payment requisites in <code>fields</code> parameter:</span>
</li>
</ul>

Parameter | Type | Description
--------|----|----
fields.name|String|Наименование банка получателя (кавычки экранируются символом `\`)
fields.extra_to_bik|String|БИК банка получателя
fields.to_bik|String|БИК банка получателя
fields.city|String|Город местонахождения получателя
fields.info|String|Константа, `Коммерческие организации`
fields.is_commercial|String|Служебная информация, константа `1`
fields.to_name|String|Наименование организации (кавычки экранируются символом `\`)
fields.to_inn|String|ИНН организации
fields.to_kpp|String|КПП организации
fields.nds|String|Признак уплаты НДС. Если вы оплачиваете квитанцию и в ней не указан НДС, то строка `НДС не облагается`. В ином случае, строка `В т.ч. НДС`.
fields.goal|String|Назначение платежа
fields.urgent|String|Признак срочного платежа (`0` - нет, `1` - да). Срочный платеж выполняется от 10 минут. Возможен по будням с 9:00 до 20:30 по московскому времени. Стоимость услуги — 25 рублей.
fields.account| String| Номер счета получателя
fields.from_name|String|Имя плательщика
fields.from_name_p|String|Отчество плательщика
fields.from_name_f|String|Фамилия плательщика
fields.requestProtocol|String|Служебная информация, константа `qw1`
fields.toServiceId|String|Служебная информация, константа `1717`

<h3 class="request">Ответ ←</h3>

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


## Поиск провайдера {#search}

## Поиск провайдера по строке

Используйте API для поиска идентификатора провайдера.

<h3 class="request method">Запрос → POST</h3>

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

# поиск на qiwi.com - определение id провайдера по названию
def qiwi_com_search(search_phrase):
    s = requests.Session()
    search = s.post('https://qiwi.com/search/results/json.action', params={'searchPhrase':search_phrase})
    return search.json()['data']['items']
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://qiwi.com/search/results/json.action?<a>searchPhrase=value</a></span></h3></li>
        <ul>
             <li><strong>searchPhrase</strong> - строка ключевых слов для поиска провайдера.</li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>


<h3 class="request">Ответ ←</h3>

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
# Поиск провайдера
prv = qiwi_com_search('Билайн домашний интернет')[0]['item']['id']['id']
print(prv)
~~~

Успешный JSON-ответ содержит идентификаторы найденных провайдеров:

Параметр | Тип | Описание
-----|----|-----
data.items | Array | Список провайдеров
items[].item.id.id | String | Идентификатор провайдера

### Определение мобильного оператора {#mnp}

Предварительное определение оператора мобильного номера выполняется данным запросом. В ответе возвращается идентификатор провайдера для [запроса пополнения телефона](#cell).

<h3 class="request method" id="mnp">Запрос → POST</h3>

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
    <li><h3>Параметры</h3><span>Параметр передается в теле запроса как <code>formdata</code>.</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
phone | String URL-encoded |Мобильный номер в международном формате без знака `+`. Обязательный параметр.

<h3 class="request">Ответ ←</h3>

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

> Не удалось определить мобильного оператора

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
print(mobile_operator(79652468447))
~~~

Ответ с HTTP Status 200 и параметром `code.value` = 0 является признаком успешной проверки. Идентификатор оператора находится в параметре `message`.

Ответ с HTTP Status 200 и параметром `code.value` = 2 означает, что невозможно определить оператора.

### Определение провайдера перевода на карту {#card_check}

Определение провайдера перевода на карту выполняется данным запросом. В ответе возвращается идентификатор провайдера для [запроса перевода на карту](#cards).

Запрос не требует авторизации.

<h3 class="request method" id="card_check">Запрос → POST</h3>

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
    <li><h3>Параметры</h3><span>Параметр передается в теле запроса как formdata.</span>
    </li>
</ul> 

Параметр|Тип|Описание
--------|----|----
cardNumber | String |Немаскированный номер карты (без пробелов). Обязательный параметр

<h3 class="request">Ответ ←</h3>

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
print(card_system(4890xxxxxxxx1698))
~~~

> Не удалось определить платежную систему карты

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

Ответ с HTTP Status 200 и параметром `code.value` = 0 является признаком успешной проверки. Идентификатор [платежной системы](#cards) находится в параметре `message`.

Ответ с HTTP Status 200 и параметром `code.value` = 2 означает, что в номере карты ошибка или платежная система не определена.

## Модели данных API 

### Класс Payment {#payment_obj}

Объект, описывающий данные для платежа на провайдера в QIWI Кошельке.

Параметр|Тип|Описание|Обяз.
--------|----|----|------
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).|+
sum|Object| Данные о сумме платежа:
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.|+
sum.currency|String|Валюта (только `643`, рубли)|+
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`|+
paymentMethod.accountId|String| Константа, `643`|+
fields|Object| Реквизиты платежа. Состав полей зависит от провайдера.
comment|String|Комментарий к платежу. Используется только для [переводов на QIWI кошелек](#p2p) и при [конвертации](#CCY) |-


### Класс PaymentInfo {#payment_info}

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
    "comment": "test"
}
~~~

Объект, описывающий данные платежной транзакции в QIWI Кошельке. Возвращается в ответ на запросы к платежному API.

Параметр | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из платежного запроса
terms | String | Идентификатор провайдера, на которого был отправлен платеж
fields|Object|Копия объекта `fields` из платежного запроса. **Номер карты (если был выполнен перевод на карту) возвращается в маскированном виде**
sum|Object|Копия объекта `sum` из платежного запроса
source| String| Константа, `account_643`
comment| String | Копия параметра `comment` из платежного запроса (возвращается, если присутствует в запросе)
transaction|Object|Объект с данными о транзакции в процессинге QIWI Wallet. Параметры:
transaction.id|String|ID транзакции в процессинге QIWI Wallet
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге QIWI Wallet. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments).


# Счета {#invoices}

## Выставление счета {#invoice}

Для выставления счета на QIWI Кошелек используется протокол [API P2P-счетов](https://developer.qiwi.com/ru/p2p-payments/#create).Для авторизации используется токен P2P. 

Вы можете получить токен P2P на [p2p.qiwi.com](https://p2p.qiwi.com) в личном кабинете, или использовать представленный ниже запрос. Этим запросом можно также настроить адрес [уведомлений об оплате счетов](https://developer.qiwi.com/ru/p2p-payments/#notification)

Данный запрос возвращает пару токенов P2P (для выставления счета при вызове платежной формы и через API, соответственно) в параметрах ответа `PublicKey` и `SecretKey`. Для авторизации используется [токен API QIWI Кошелька](#auth_data).

<h3 class="request method">Запрос → POST</h3>

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
<li><h3>URL</h3> <span>https://edge.qiwi.com/widgets-api/api/p2p/protected/keys/create</span></li>
</ul>

Параметр|Тип|Описание
--------|----|----
keysPairName| String| Название пары ключей P2P
serverNotificationsUrl|String |URL для [уведомлений об оплате счетов](https://developer.qiwi.com/ru/p2p-payments/#notification) (необязательный параметр)

## Список счетов  {#list_invoice}

Метод получения списка неоплаченных счетов вашего кошелька. Список строится в обратном хронологическом порядке. Можно использовать фильтры по времени выставления счета, начальному идентификатору счета.

<h3 class="request method">Запрос → GET</h3>

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
    <li><h3>URL <span>https://edge.qiwi.com/checkout-api/api/bill/search?statuses=READY_FOR_PAY&rows=50</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Данные параметры передаются в строке запроса:</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
rows | Integer |Максимальное число счетов в ответе, для разбивки списка на части. Целое число от 1 до 50. По умолчанию возвращается не более 50 счетов.
statuses|String| Статус неоплаченного счета. Только строка `READY_FOR_PAY`
min_creation_datetime|Long|Нижняя временная граница для поиска счетов, Unix-time
max_creation_datetime|Long|Верхняя временная граница для поиска счетов, Unix-time
next_id|Number|Начальный идентификатор счета для поиска, чтобы возврат списка выполнялся начиная с этого значения
next_creation_datetime|Long|Начальное время для поиска (возвращаются только счета, выставленные ранее этого времени), Unix-time.

<h3 class="request">Ответ ←</h3>

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
        "short_name": "Букмекерская контора ФОНБЕТ",
        "long_name": "ООО «Ф.О.Н.»",
        "logo_url":"https://static.qiwi.com/img/providers/logoBig/480706_l.png"
      },
      "comment": "Deposit to FON 13515573",
      "pay_url":"https://oplata.qiwi.com/form?shop=480706&transaction=102263702405"
    }
  ]
}
~~~

Успешный JSON-ответ содержит список неоплаченных счетов вашего кошелька, соответствующих заданному фильтру:

<a name="invoice_data"></a>

Параметр|Тип|Описание
--------|----|----
bills|Array[Object]|Список платежей. <br>Число платежей равно параметру `rows` из запроса, или максимально 50, если параметр не указан
bills[].id|Integer|Идентификатор счета в QIWI Кошельке
bills[].external_id|String| Идентификатор счета у мерчанта
bills[].creation_datetime|Long|Дата/время создания счета, Unix-time
bills[].expiration_datetime|Long|Дата/время окончания срока действия счета, Unix-time
bills[].sum|Object|Сведения о сумме счета
sum.currency|Integer|Валюта суммы счета
sum.amount|Number|Сумма счета
bills[].status|String | Константа, `READY_FOR_PAY`
bills[].type|String|Константа, `MERCHANT`
bills[].repetitive|Boolean|Служебное поле
bills[].provider|Object|Информация о мерчанте
provider.id|Integer|Идентификатор мерчанта в QIWI
provider.short_name|String|Сокращенное название мерчанта
provider.long_name|String|Полное название мерчанта
provider.logo_url|String|Ссылка на логотип мерчанта
bills[].comment|String|Комментарий к счету
bills[].pay_url|String|Ссылка для оплаты счета в интерфейсе QIWI

## Оплата счета  {#paywallet_invoice}

Выполнение безусловной оплаты счета без SMS-подтверждения.

<h3 class="request method">Запрос → POST</h3>

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
    <li><h3>URL <span>https://edge.qiwi.com/checkout-api/invoice/pay/wallet</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON. Все параметры обязательны.</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
invoice_uid | String |ID счета в QIWI (берется из значения `bills[].id` [данных о счете](#invoice_data)
currency|String| Валюта суммы счета (берется из значения `bills[].sum.currency` [данных о счете](#invoice_data))


<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "invoice_status": "PAID_STATUS",
  "is_sms_confirm": false,
  "WALLET_ACCEPT_PAY_RESULT": {}
}
~~~

Успешный JSON-ответ содержит статус оплаченного счета:

Параметр|Тип|Описание
--------|----|----
invoice_status|String|Строка кода статуса оплаты счета, `PAID_STATUS`. Любой другой статус означает неуспех платежной транзакции.
is_sms_confirm|String|Признак подтверждения по SMS

## Отмена неоплаченного счета  {#cancel_invoice}

Метод отклоняет неоплаченный счет. При этом счет становится недоступным для оплаты.


<h3 class="request method">Запрос → POST</h3>

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
    <li><h3>URL <span>https://edge.qiwi.com/checkout-api/api/bill/reject</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Данный обязательный параметр передается в теле запроса в формате JSON:</span>
    </li>
</ul>


Параметр|Тип|Описание
--------|----|----
id | Integer |ID счета для отмены (берется из значения `bills[].id` [данных о счете](#invoice_data)

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

Успешный ответ содержит HTTP-код 200.
