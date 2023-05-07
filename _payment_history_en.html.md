
# Payments History API {#payments_history}

###### Last update: 2020-07-28 | [Edit on GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/_payment_history_en.html.md)

The API gives access to the history of transactions in your QIWI wallet.

## List of payments {#payments_list}

Provides a list of payments and top-ups of your wallet. You can use the filter by number, ID and date (date interval) of transactions.

[Interactive API](https://developer.qiwi.com/sandbox/index.html#!/payment-history-controller-v-2/getPaymentHistoryByUserUsingGET_1)

<aside class="warning">The maximum frequency of the payment history requests is no more than 100 requests per minute for <b>the same wallet number</b>. If exceeded, access to the API is blocked for 5 minutes.</aside>

<h3 class="request method">Request → GET</h3>

>Example 1. Last 10 payments

~~~shell
curl "https://edge.qiwi.com/payment-history/v2/persons/<wallet>/payments?rows=10" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API token>"
~~~

> Example 2. Payments for 10.05.2017

~~~shell
curl "https://edge.qiwi.com/payment-history/v2/persons/<wallet>/payments?rows=50&startDate=2017-05-10T00%3A00%3A00%2B03%3A00&endDate=2017-05-10T23%3A59%3A59%2B03%3A00" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API token>"
~~~

> Example 3. Continuing payments list (parameters nextTxnId=9103121 and nextTxnDate=2017-05-11T12:35:23+03:00 returned in the previous history request)

~~~shell
curl "https://edge.qiwi.com/payment-history/v2/persons/<wallet>/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12%3A35%3A23%2B03%3A00" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API token>"
~~~

> Example 4. Last 10 payments from the ruble account and from the linked card

~~~http
GET /payment-history/v2/persons/<wallet>/payments?rows=10&operation=OUT&sources%5B0%5D=QW_RUB&sources%5B1%5D=CARD HTTP/1.1
Accept: application/json
Authorization: Bearer <API token>
Host: edge.qiwi.com
~~~

> Example 5. Payments for 10.05.2017 from the ruble account

~~~http
GET /payment-history/v2/persons/<wallet>/payments?rows=50&sources%5B0%5D=QW_RUB&startDate=2017-05-10T00%3A00%3A00%2B03%3A00&endDate=2017-05-10T23%3A59%3A59%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer <API token>
Host: edge.qiwi.com
~~~

> Example 6. Continuing payments list for 10.05.2017 (see Example 2, parameters nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23+03:00 returned)

~~~http
GET /payment-history/v2/persons/<wallet>/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12%3A35%3A23%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer <API token>
Host: edge.qiwi.com
~~~

~~~python
import requests

# Payments history - the most recent and next n payments
def payment_history_last(my_login, api_access_token, rows_num, next_TxnId, next_TxnDate):
    s = requests.Session()
    s.headers['authorization'] = 'Bearer ' + api_access_token  
    parameters = {'rows': rows_num, 'nextTxnId': next_TxnId, 'nextTxnDate': next_TxnDate}
    h = s.get('https://edge.qiwi.com/payment-history/v2/persons/' + my_login + '/payments', params = parameters)
    return h.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/payment-history/v2/persons/<a>wallet</a>/payments?<a>parameter=value</a></span></h3>
        <ul>
             <li><strong>wallet</strong> - your QIWI wallet number without <i>+</i> sign</li>
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

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>These are transmitted in the query path:</span>
    </li>
</ul>

Name | Type | Description
--------|----|----
rows | Integer | The number of payments in response to break down the report into chunks. It must be from 1 to 50. The request returns a specified number of payments in reverse chronological order, starting from the current date or date in the `startDate` option. **Required**
operation|String| The type of operations in the report, for selection. Acceptable values:<br>`ALL` - all transactions, <br>`IN` - only top-ups, <br>`OUT` - only payments, <br>`QIWI_CARD` - only payments from QIWI cards (QVC, QVP). <br>By default, `ALL` is used
sources|Array[String]| A list of payment sources for the filter. Each source is numbered from scratch (`sources[0]`, `sources[1]` etc). Acceptable values: <br>`QW_RUB` - ruble account of your QIWI wallet, <br>`QW_USD` - USD account of your QIWI wallet, <br>`QW_EUR` - euro account of your QIWI wallet, <br>`CARD` - cards linked to the wallet and other credit cards, <br>`MK` - the corresponding account on the mobile operator. If not specified, all sources are considering
startDate | DateTime URL-encoded| The initial date for the search for payments. **It is only used with `endDate`. The maximum allowable interval between `startDate` and `endDate` is 90 calendar days.** By default, equal to the daily shift from the current date to Moscow time. The date can be specified in any time zone `TZD` (in `YYYY-MM-DD'T'hh:mm:ssTZD` format), but it must coincide with the time zone in the `endDate` parameter. Timezone designation: `+hh:mm` or -`hh:mm` (time shift from GMT). 
endDate | DateTime URL-encoded | The final date for the search for payments. **It is only used with `startDate`. The maximum allowable interval between `startDate` and `endDate` is 90 calendar days.** By default, equal to current date/time in Moscow time. The date can be specified in any time zone `TZD` (in `YYYY-MM-DD'T'hh:mm:ssTZD` format), but it must coincide with the time zone in the `startDate` parameter. Timezone designation: `+hh:mm` or -`hh:mm` (time shift from GMT).
nextTxnDate | DateTime URL-encoded| The transaction date to start the report (should be equal to `nextTxnDate` in the previous report). **Used only with `nextTxnId`**
nextTxnId | Long | The transaction number to start the report (should be equal to `nextTxnId` in the previous report). **Used only with `nextTxnDate`**

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

~~~json
{"data":
  [{
    "txnId":9309,
    "personId":79112223344,
    "date":"2017-01-21T11:41:07+03:00",
    "errorCode":0,
    "error":null,
    "status":"SUCCESS",
    "type":"OUT",
    "statusText":"Успешно",
    "trmTxnId":"1489826461807",
    "account":"0003***",
    "sum":{
        "amount":70,
        "currency":643
        },
    "commission":{
        "amount":0,
        "currency":643
        },
    "total":{
        "amount":70,
        "currency":643
        },
    "provider":{
      ...
    },
    "source": {},
    "comment":"",
    "currencyRate":1
  }],
  "nextTxnId":9001,
  "nextTxnDate":"2025-01-31T15:24:10+03:00"
}
~~~

~~~python
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# last 20 payments
lastPayments = payment_history_last(mylogin, api_access_token, '5','','')

# date and time of the next payment (to use in the next request)
nextTxnDate = lastPayments['nextTxnDate']

# transaction id of the next payment (to use in the next request)
nextTxnId = lastPayments['nextTxnId']

# the most recent and next n payments
orderedPayments = payment_history_last(mylogin, api_access_token, '5', nextTxnId, nextTxnDate)
~~~

Successful JSON-response contains a list of payments corresponding to the request's filter:

<a name="history_data"></a>

Field|Type|Description
--------|----|----
data|Array[Object]| A list of [PaymentHistoryItem objects](#payment-history-item).<br>Number of objects in the list is less than or equals to `rows` value from the request
nextTxnId|Number(Integer)| Next transaction ID, in the complete list of your transactions
nextTxnDate|DateTime| Next transaction date/time, in the complete list of your transactions, Moscow time (in `YYYY-MM-DD'T'hh:mm:ss+03:00`)

## Statistics {#stat}

Provides aggregate statistics on the amount of payments for a given period.

[Interactive API](https://developer.qiwi.com/sandbox/index.html#!/payment-history-controller-v-2/getPaymentHistoryTotalByUserUsingGET_1)

<aside class="notice">Maximum period for getting statistics is 90 days.</aside>

<h3 class="request method">Request → GET</h3>

~~~shell
curl "https://edge.qiwi.com/payment-history/v2/persons/<wallet>/payments/total?startDate=2017-03-01T00%3A00%3A00%2B03%3A00&endDate=2017-03-31T11%3A44%3A15%2B03%3A00" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API token>"
~~~

~~~http
GET /payment-history/v2/persons/<wallet>/payments/total?startDate=2017-03-01T00%3A00%3A00%2B03%3A00&endDate=2017-03-31T11%3A44%3A15%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer <API token>
Host: edge.qiwi.com
~~~

~~~python
import requests

# Sum of payments for a range of dates
def payment_history_summ_dates(my_login, api_access_token, start_Date, end_Date):
    s = requests.Session()
    s.headers['authorization'] = 'Bearer ' + api_access_token
    parameters = {'startDate': start_Date,'endDate': end_Date}
    h = s.get('https://edge.qiwi.com/payment-history/v2/persons/' + my_login + '/payments/total', params = parameters)
    return h.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/payment-history/v2/persons/<a>wallet</a>/payments/total?<a>parameter=value</a></span></h3>
        <ul>
             <li><strong>wallet</strong> - the number of your QIWI wallet without <i>+</i> sign</li>
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

<ul class="nestedList params">
    <li><h3>Parameters</h3><span>Send them in the query path:</span>
    </li>
</ul>

Name| Type | Description
--------|----|-------
startDate|DateTime URL-encoded | Start date of the period, in any time zone `TZD` (date format `YYYY-MM-DD'T'hh:mm:ssTZD`). Time zone must coincide with `endDate` time zone. Designation `TZD`: `+hh:mm` or -`hh:mm` (time shift from GMT). **Required**
endDate|DateTime URL-encoded| Final date of th period, in any time zone `TZD` (date format `YYYY-MM-DD'T'hh:mm:ssTZD`). Time zone must coincide with `startDate` time zone. Designation `TZD`: `+hh:mm` or -`hh:mm` (time shift from GMT). **Required**
operation|String| Operations to take into account when accumulating statistics. Possible values:<br>`ALL` - all operations, <br>`IN` - only top-ups, <br>`OUT` - only payments, <br>`QIWI_CARD` - only payments from QIWI cards (QVC, QVP). <br>Default value is `ALL`.
sources|Array[String]|Payment sources to filter data. Each source is enumerated starting from zero (`sources[0]`, `sources[1]` and so on). Possible values of each source: <br>`QW_RUB` - ruble QIWI Wallet account, <br>`QW_USD` - USD  QIWI Wallet account, <br>`QW_EUR` - euro  QIWI Wallet account, <br>`CARD` - credit cards, both linked to QIWI Wallet and others, <br>`MK` - mobile operator account. If not specified, all sources are collected.

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

~~~json
{
 "incomingTotal":[
  {
    "amount":3500,
    "currency":643
  }
 ],
 "outgoingTotal":[
  {
   "amount":3497.5,
   "currency":643
  }
 ]
}
~~~

~~~python
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# Payments amount from April 12 till July 11
print(payment_history_summ_dates(mylogin, api_access_token, '2019-04-12T00:00:00Z','2019-07-11T23:59:59Z'))

{'incomingTotal': [{'amount': 3.33, 'currency': 840},
  {'amount': 3481, 'currency': 643}],
 'outgoingTotal': [{'amount': 3989.98, 'currency': 643},
  {'amount': 3.33, 'currency': 840}]}
~~~

Successful JSON-response contains statistics data for a specified period:

Field| Type | Description
--------|----|----
incomingTotal|Array[Object]| Array of total amounts of incoming payments (top-up payments) separated by payment's currency
incomingTotal[].amount | Number(Decimal) |Top-ups amount for the period
incomingTotal[].currency|Number(3)| Currency of the operations (ISO-4217)
outgoingTotal|Array[Object]| Array of total amounts of payments separated by payment's currency
outgoingTotal[].amount | Number(Decimal) | Payments amount for the period
outgoingTotal[].currency|Number(3)| Currency of the operations (ISO-4217)

## Transaction details {#txn_info}

Returns details on a specific transaction from your payments history.

[Interactive API](https://developer.qiwi.com/sandbox/index.html#!/payment-history-controller-v-2/getPaymentHistoryByTransactionUsingGET_1)

<h3 class="request method">Request → GET</h3>

~~~shell
curl "https://edge.qiwi.com/payment-history/v2/transactions/9112223344" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API token>"
~~~

~~~http
GET /payment-history/v2/transactions/9112223344 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

~~~python
import requests

# Transaction details
def payment_history_transaction(api_access_token, transaction_id, transaction_type):
    s = requests.Session()
    s.headers['authorization'] = 'Bearer ' + api_access_token  
    parameters = {'type': transaction_type} # transaction_type 'IN' 'OUT'
    h = s.get('https://edge.qiwi.com/payment-history/v1/transactions/'+transaction_id, params = parameters)
    return h.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/payment-history/v2/transactions/<a>transactionId</a>?<a>type=value</a></span></h3>
        <ul>
             <li><strong>transactionId</strong> - transaction ID from <a href="#history_data">Payments history</a> report (<i>txnId</i> field of Transaction object)</li>
             <li><strong>type</strong> - transaction type from <a href="#history_data">Payments history</a> report (<i>type</i> field of Transaction object). This is optional parameter</li>
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
~~~

~~~json
{
    "txnId": 11233344692,
    "personId": 79161122331,
    "date": "2017-08-30T14:38:09+03:00",
    "errorCode": 0,
    "error": null,
    "status": "WAITING",
    "type": "OUT",
    "statusText": "Запрос обрабатывается",
    "trmTxnId": "11233344691",
    "account": "15040930424823121081",
    "sum": {
        "amount": 1,
        "currency": 643
    },
    "commission": {
        "amount": 0,
        "currency": 643
    },
    "total": {
        "amount": 1,
        "currency": 643
    },
    "provider": {
        "id": 1,
        "shortName": "MTS",
        "longName": "MTS",
        "logoUrl": null,
        "description": null,
        "keys": null,
        "siteUrl": null,
        "extras": []
    },
    "source": {
        "id": 7,
        "shortName": "QIWI Wallet",
        "longName": "QIWI Wallet",
        "logoUrl": null,
        "description": null,
        "keys": "мобильный кошелек, кошелек, перевести деньги, личный кабинет, отправить деньги, перевод между пользователями",
        "siteUrl": null,
        "extras": []
    },
    "comment": "",
    "currencyRate": 1,
    "paymentExtras": [],
    "serviceExtras": {},
    "view": {},
    "features": {
      "chequeReady": false,
      "bankDocumentAvailable": false,
      "bankDocumentReady": false,
      "repeatPaymentEnabled": false,
      "favoritePaymentEnabled": false,
      "regularPaymentEnabled": false
    }
}
~~~

~~~python
# wallet number like 79992223344
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# Transaction ID 11181101215 details
transactionInfo = payment_history_transaction(api_access_token, '11181101215', 'OUT')

# Transaction number 5 details
lastPayments = payment_history_last(mylogin, api_access_token, '20','','')
last_txn_id = lastPayments['data'][5]['txnId']
last_txn_type = lastPayments['data'][5]['type']

transactionInfo = payment_history_transaction(api_access_token, str(last_txn_id), last_txn_type)
~~~

Successful JSON-response contains [Transaction object](#txnid).

## Payment receipt {#payment_receipt}

Returns electronic receipt for a certain transaction in PDF/JPEG format, either as binary file or via e-mail to specified address.

### Receipt file {#receipt_file}

[Interactive API](https://developer.qiwi.com/sandbox/index.html#!/cheque-controller-v-1/getChequeBytesUsingGET)

<h3 class="request method">Request → GET</h3>

~~~shell
curl "https://edge.qiwi.com/payment-history/v1/transactions/9112223344/cheque/file?type=IN&format=PDF" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API token>"
~~~

~~~http
GET /payment-history/v1/transactions/9112223344/cheque/file?type=IN&format=PDF HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

~~~python
import requests

# Get receipt text in file
def payment_history_cheque_file(transaction_id, transaction_type, filename, api_access_token):
    s = requests.Session()
    s.headers['Accept'] ='application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    parameters = {'type': transaction_type,'format': 'PDF'}
    h = s.get('https://edge.qiwi.com/payment-history/v1/transactions/'+transaction_id+'/cheque/file', params=parameters)
    h.status_code
    with open(filename + '.pdf', 'wb') as f:
        f.write(h.content)
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/payment-history/v1/transactions/<a>transactionId</a>/cheque/file?<a>type=value&format=value</a></span></h3>
        <ul>
             <li><strong>transactionId</strong> - transaction ID from <a href="#history_data">Payments history</a> report (<i>txnId</i> field in Transaction object)</li>
             <li><strong>type</strong> - transaction type from <a href="#history_data">Payments history</a> report (<i>type</i> field in Transaction object)</li>
             <li><strong>format</strong> - file type for receipt export. Possible values: <i>JPEG</i>, <i>PDF</i></li>
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

[
  ""
]
~~~

Successful JSON-response contains binary file.

### Receipt sending {#receipt_send}

[Interactive API](https://developer.qiwi.com/sandbox/index.html#!/cheque-controller-v-1/sendChequeUsingPOST)

<h3 class="request method">Request → POST</h3>

~~~shell
curl -X POST \
  "https://edge.qiwi.com/payment-history/v1/transactions/9112223344/cheque/send?type=IN" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer <API token>" \
  -d '{"email": "my@example.com"}'
~~~

~~~http
POST /payment-history/v1/transactions/9112223344/cheque/send?type=IN HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com

{"email": "my@example.com"}
~~~

~~~python
import requests

# Send receipt to email
def payment_history_cheque_send(transaction_id, transaction_type, email, api_access_token):
    s = requests.Session()
    s.headers['content-type'] ='application/json'
    s.headers['Accept'] ='application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    postjson = {'email':email}
    h = s.post('https://edge.qiwi.com/payment-history/v1/transactions/' + transaction_id + '/cheque/send?type=' + transaction_type, json = postjson)
    h.status_code
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/payment-history/v1/transactions/<a>transactionId</a>/cheque/send?<a>type=value</a></span></h3>
        <ul>
        <li><strong>transactionId</strong> - transaction ID from <a href="#history_data">Payments history</a> report (<i>txnId</i> field in Transaction object)</li>
        <li><strong>type</strong> - transaction type from <a href="#history_data">Payments history</a> report (<i>type</i> field in Transaction object)</li>
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
    <li><h3>Parameter</h3><span>Send this parameter in JSON body of the request:</span>
    </li>
</ul>

Name|Type|Description
--------|----|----
email|String| Email address

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 201 Created
~~~

~~~python
# wallet number like 79992223344
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

lastPayments = payment_history_last(mylogin, api_access_token, '20','','')
last_txn_id = lastPayments['data'][5]['txnId']
last_txn_type = lastPayments['data'][5]['type']

# Send receipt to email
payment_history_cheque_send(str(last_txn_id), last_txn_type, 'mmd@yandex.ru', api_access_token)
~~~

Successful JSON-response contains HTTP result code of file sending operation.

## API data models

### PaymentHistoryItem class {#payment-history-item}

Object contains information about existing QIWI Wallet payment.

Name| Type | Description
--------|----|-------
txnId | Integer | Transaction ID in QIWI Wallet processing
personId|Integer| Wallet number
date|DateTime| For payments history reports - Payment date/time, in the request time zone (see `startDate` parameter). Date format `YYYY-MM-DD'T'hh:mm:ss+03:00`<br>For transaction details - Payment date/time, in Moscow time zone (date format: `YYYY-MM-DD'T'hh:mm:ss+03:00`)
errorCode|Number(Integer)|[Payment error code](#errorCode)
error| String| Error description
type | String| Payment type. Possible values:<br>`IN` - top-up, <br>`OUT` - payment, <br>`QIWI_CARD` - payment from QIWI card (QVC, QVP).
status|String| Payment status. Possible values:<br>`WAITING` - payment is processing, <br>`SUCCESS` - successful payment, <br>`ERROR` - payment error.
statusText|String | Text description of the status
trmTxnId|String| Transaction's client ID (assigned on the client device)
account| String| For payments, recipient's identifier (masked card number, phone number, account number etc.). For top-ups, sender's or terminal's identifier, or top-up agent name
sum|Object| Payment's amount data.
-----|-----|-----
sum.amount|Number(Decimal)|amount,
sum.currency|Number(3)| currency (ISO-4217)
-----|-----|-----
commission|Object| Payment's commission data
-----|-----|-----
commission.amount|Number(Decimal)|amount,
commission.currency|Number(3)| currency (ISO-4217)
-----|-----|-----
total|Object| Total amount of transaction.
-----|-----|-----
total.amount|Number(Decimal)|amount, it is `sum.amount` plus `commission.amount`,
total.currency|Number(3)| currency  (ISO-4217)
-----|-----|-----
provider|Object| Provider's data
-----|-----|-----
provider.id|Integer| Provider ID in QIWI Wallet system,
provider.shortName|String| Provider's short name,
provider.longName|String | Provider's extended name,
provider.logoUrl|String | Provider's logo URL,
provider.description|String | Provider's description (in HTML),
provider.keys|String | Provider's keywords list,
provider.siteUrl|String | Provider's site
-----|-----|-----
comment|String | Comment to the payment
currencyRate|Number(Decimal) | Currency exchange rate (if applied to transaction)

### Transaction class {#txnid}

Object contains information about existing QIWI Wallet transaction.

Name| Type | Description
--------|----|-------
txnId | Integer | Transaction ID in QIWI Wallet processing
personId|Integer| Wallet number
date|DateTime| For payments history reports - Payment date/time, in the request time zone (see `startDate` parameter). Date format `YYYY-MM-DD'T'hh:mm:ss+03:00`<br>For transaction details - Payment date/time, in Moscow time zone (date format: `YYYY-MM-DD'T'hh:mm:ss+03:00`)
errorCode|Number(Integer)|[Payment error code](#errorCode)
error| String| Error description
type | String| Payment type. Possible values:<br>`IN` - top-up, <br>`OUT` - payment, <br>`QIWI_CARD` - payment from QIWI card (QVC, QVP).
status|String| Payment status. Possible values:<br>`WAITING` - payment is processing, <br>`SUCCESS` - successful payment, <br>`ERROR` - payment error.
statusText|String | Text description of the status
trmTxnId|String| Transaction's client ID (assigned on the client device)
account| String| For payments, recipient's identifier (masked card number, phone number, account number etc.). For top-ups, sender's or terminal's identifier, or top-up agent name
sum|Object| Payment's amount data.
-----|-----|-----
sum.amount|Number(Decimal)|amount,
sum.currency|Number(3)| currency (ISO-4217)
-----|-----|-----
commission|Object| Payment's commission data
-----|-----|-----
commission.amount|Number(Decimal)|amount,
commission.currency|Number(3)| currency (ISO-4217)
-----|-----|-----
total|Object| Total amount of transaction.
-----|-----|-----
total.amount|Number(Decimal)|amount, it is `sum.amount` plus `commission.amount`,
total.currency|Number(3)| currency  (ISO-4217)
-----|-----|-----
provider|Object| Provider's data
-----|-----|-----
provider.id|Integer| Provider ID in QIWI Wallet system,
provider.shortName|String| Provider's short name,
provider.longName|String | Provider's extended name,
provider.logoUrl|String | Provider's logo URL,
provider.description|String | Provider's description (in HTML),
provider.keys|String | Provider's keywords list,
provider.siteUrl|String | Provider's site
-----|-----|-----
comment|String | Comment to the payment
currencyRate|Number(Decimal) | Currency exchange rate (if applied to transaction)
paymentExtras|Array of Object | Service information
features|Object|Set of special fields
-----|-----|-----
features.chequeReady| Boolean|Special field
features.bankDocumentReady|Boolean|Special field
features.bankDocumentAvailable|Boolean|Special field
features.repeatPaymentEnabled|Boolean|Special field
features.favoritePaymentEnabled|Boolean|Special field
features.regularPaymentEnabled|Boolean|Special field
features.chatAvailable|Boolean|Special field
features.greetingCardAttached|Boolean|Special field
-----|-----|-----
serviceExtras|Object|Служебная информация
view|Object|Служебная информация
