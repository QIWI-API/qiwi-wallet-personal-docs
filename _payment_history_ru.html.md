
# История платежей {#payments_history}

###### Последнее обновление: 2020-07-06 | [Предложить правки на GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/_payment_history_ru.html.md)

## Список платежей {#payments_list}

Запрос выгружает список платежей и пополнений вашего кошелька. Можно использовать фильтр по количеству, ID и дате (интервалу дат) транзакций.

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/payment-history-controller-v-2/getPaymentHistoryByUserUsingGET_1)

<aside class="warning">Максимальная частота запросов истории платежей - не более 100 запросов в минуту для <b>одного и того же номера кошелька</b>. При превышении доступ к API блокируется на 5 минут.</aside>

<h3 class="request method">Запрос → GET</h3>

>Пример 1. Последние 10 платежей

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments?rows=10" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

> Пример 2. Платежи за 10.05.2017

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments?rows=50&startDate=2017-05-10T00%3A00%3A00%2B03%3A00&endDate=2017-05-10T23%3A59%3A59%2B03%3A00" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

> Пример 3. Продолжение списка платежей (в предыдущем запросе истории возвращены параметры nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23+03:00)

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12%3A35%3A23%2B03%3A00" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

> Пример 4. Последние 10 платежей с рублевого баланса и с привязанной карты

~~~http
GET /payment-history/v2/persons/79112223344/payments?rows=10&operation=OUT&sources%5B0%5D=QW_RUB&sources%5B1%5D=CARD HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

> Пример 5. Платежи за 10.05.2017 с рублевого счета

~~~http
GET /payment-history/v2/persons/79112223344/payments?rows=50&sources%5B0%5D=QW_RUB&startDate=2017-05-10T00%3A00%3A00%2B03%3A00&endDate=2017-05-10T23%3A59%3A59%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

> Пример 6. Продолжение списка платежей за 10.05.2017 (в Примере 2  возвращены параметры nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23+03:00)

~~~http
GET /payment-history/v2/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12%3A35%3A23%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

~~~python
import requests

# История платежей - последние и следующие n платежей
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
             <li><strong>wallet</strong> - номер вашего кошелька без знака "+"</li>
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
    <li><h3>Параметры</h3><span>Данные параметры передаются в строке запроса:</span>
    </li>
</ul>


Параметр|Тип|Описание
--------|----|----
rows | Integer |Число платежей в ответе, для разбивки отчета на части. Целое число от 1 до 50. Запрос возвращает указанное число платежей в обратном хронологическом порядке, начиная от текущей даты или даты в параметре `startDate`. **Обязательный параметр**
operation|String| Тип операций в отчете, для отбора. Допустимые значения:<br>`ALL` - все операции, <br>`IN` - только пополнения, <br>`OUT` - только платежи, <br>`QIWI_CARD` - только платежи по картам QIWI (QVC, QVP). <br>По умолчанию `ALL`
sources|Array[String]|Список источников платежа, для фильтра. Каждый источник нумеруется, начиная с нуля (`sources[0]`, `sources[1]` и т.д.). Допустимые значения: <br>`QW_RUB` - рублевый счет кошелька, <br>`QW_USD` - счет кошелька в долларах, <br>`QW_EUR` - счет кошелька в евро, <br>`CARD` - привязанные и непривязанные к кошельку банковские карты, <br>`MK` - счет мобильного оператора. Если не указан, учитываются все источники
startDate | DateTime URL-encoded| Начальная дата поиска платежей. **Используется только вместе с `endDate`. Максимальный допустимый интервал между `startDate` и `endDate` - 90 календарных дней.** По умолчанию, равна суточному сдвигу от текущей даты по московскому времени.<br>Дату можно указать в любой временной зоне `TZD` (формат `ГГГГ-ММ-ДД'T'чч:мм:ссTZD`), однако она должна совпадать с временной зоной в параметре `endDate`. Обозначение временной зоны `TZD`: `+чч:мм` или -`чч:мм` (временной сдвиг от GMT). 
endDate | DateTime URL-encoded | Конечная дата поиска платежей. **Используется только вместе со `startDate`. Максимальный допустимый интервал между `startDate` и `endDate` - 90 календарных дней.** По умолчанию, равна текущим дате/времени по московскому времени. Дату можно указать в любой временной зоне `TZD` (формат `ГГГГ-ММ-ДД'T'чч:мм:ссTZD`), однако она должна совпадать с временной зоной в параметре `startDate`. Обозначение временной зоны `TZD`: `+чч:мм` или -`чч:мм` (временной сдвиг от GMT).
nextTxnDate | DateTime URL-encoded| Дата транзакции для начала отчета (должна быть равна параметру `nextTxnDate` в предыдущем списке). **Используется только вместе с `nextTxnId`**
nextTxnId | Long | Номер транзакции для начала отчета (должен быть равен параметру `nextTxnId` в предыдущем списке). **Используется только вместе с `nextTxnDate`**


<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{"data":
  [ 
   {
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
        "currency":"RUB"
        },
    "commission":{
        "amount":0,
        "currency":"RUB"
        },
    "total":{
        "amount":70,
        "currency":"RUB"
        },
    "provider":{
      ...
    },
    "source": {},
    "comment":null,
    "currencyRate":1,
    "extras":null,
    "chequeReady":true,
    "bankDocumentAvailable":false,
    "bankDocumentReady":false,
    "repeatPaymentEnabled":false,
    "favoritePaymentEnabled": true,
    "regularPaymentEnabled": true
   }
  ],
  "nextTxnId":9001,
  "nextTxnDate":"2017-01-31T15:24:10+03:00"
}
~~~

~~~python
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# последние 20 платежей
lastPayments = payment_history_last(mylogin, api_access_token, '5','','')

# дата и время следующего платежа
nextTxnDate = lastPayments['nextTxnDate']

# id транзакции следующего платежа
nextTxnId = lastPayments['nextTxnId']

# История платежей - последние и следующие n платежей
orderedPayments = payment_history_last(mylogin, api_access_token, '5', nextTxnId, nextTxnDate)
~~~

Успешный JSON-ответ содержит список платежей из истории кошелька, соответствующих заданному фильтру:

<a name="history_data"></a>

Поле ответа|Тип|Описание
--------|----|----
data|Array[Object]|Список [объектов Transaction](#txnid). <br>Число транзакций в списке меньше или равно параметру `rows` из запроса
nextTxnId|Number(Integer)|ID следующей транзакции в полном списке
nextTxnDate|DateTime|Дата/время следующей транзакции в полном списке, время московское (в формате `ГГГГ-ММ-ДД'T'чч:мм:сс+03:00`)

## Статистика платежей {#stat}

Данный запрос используется для получения сводной статистики по суммам платежей за заданный период.

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/payment-history-controller-v-2/getPaymentHistoryTotalByUserUsingGET_1)

<aside class="notice">Максимальный период для выгрузки статистики - 90 календарных дней.</aside>

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments/total?startDate=2017-03-01T00%3A00%3A00%2B03%3A00&endDate=2017-03-31T11%3A44%3A15%2B03%3A00" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /payment-history/v2/persons/79112223344/payments/total?startDate=2017-03-01T00%3A00%3A00%2B03%3A00&endDate=2017-03-31T11%3A44%3A15%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

~~~python
import requests

# История платежей - сумма за диапазон дат
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
             <li><strong>wallet</strong> - номер вашего кошелька без знака "+"</li>
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
    <li><h3>Параметры</h3><span>Данные параметры передаются в строке запроса:</span>
    </li>
</ul>

Параметр|Тип |Описание
--------|----|-------
startDate|DateTime URL-encoded | Начальная дата периода статистики. Дату можно указать в любой временной зоне `TZD` (формат `ГГГГ-ММ-ДД'T'чч:мм:ссTZD`), однако она должна совпадать с временной зоной в параметре `endDate`. Обозначение временной зоны `TZD`: `+чч:мм` или -`чч:мм` (временной сдвиг от GMT). **Обязательный параметр**
endDate|DateTime URL-encoded| Конечная дата периода статистики. Дату можно указать в любой временной зоне `TZD` (формат `ГГГГ-ММ-ДД'T'чч:мм:ссTZD`), однако она должна совпадать с временной зоной в параметре `startDate`. Обозначение временной зоны `TZD`: `+чч:мм` или -`чч:мм` (временной сдвиг от GMT). **Обязательный параметр**
operation|String| Тип операций, учитываемых при подсчете статистики. Допустимые значения:<br>`ALL` - все операции, <br>`IN` - только пополнения, <br>`OUT` - только платежи, <br>`QIWI_CARD` - только платежи по картам QIWI (QVC, QVP). <br>По умолчанию `ALL`.
sources|Array[String]|Источники платежа, по которым вернутся данные. Каждый источник нумеруется, начиная с нуля (`sources[0]`, `sources[1]` и т.д.). Допустимые значения: <br>`QW_RUB` - рублевый счет кошелька, <br>`QW_USD` - счет кошелька в долларах, <br>`QW_EUR` - счет кошелька в евро, <br>`CARD` - привязанные и непривязанные к кошельку банковские карты, <br>`MK` - счет мобильного оператора. Если не указан, учитываются все источники платежа.


<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
 "incomingTotal":[
  {
  "amount":3500,
  "currency":"RUB"
  }],
 "outgoingTotal":[
  {
  "amount":3497.5,
  "currency":"RUB"
  }]
}
~~~

~~~python
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# История платежей - сумма за диапазон
# не более 90 дней с 12 апреля по 11 июля 2019 года
print(payment_history_summ_dates(mylogin, api_access_token, '2019-04-12T00:00:00Z','2019-07-11T23:59:59Z'))

{'incomingTotal': [{'amount': 3.33, 'currency': 840},
  {'amount': 3481, 'currency': 643}],
 'outgoingTotal': [{'amount': 3989.98, 'currency': 643},
  {'amount': 3.33, 'currency': 840}]}
~~~

Успешный JSON-ответ содержит статистику платежей за выбранный период:

Поле ответа|Тип|Описание
--------|----|----
incomingTotal|Array[Object]|Данные о входящих платежах (пополнениях), отдельно по каждой валюте
incomingTotal[].amount | Number(Decimal) |Сумма пополнений за период
incomingTotal[].currency|String|Валюта пополнений
outgoingTotal|Array[Object]|Данные об исходящих платежах, отдельно по каждой валюте
outgoingTotal[].amount | Number(Decimal) |Сумма платежей за период
outgoingTotal[].currency|String|Валюта платежей

## Информация о транзакции {#txn_info}

Запрос используется для получения информации по определенной транзакции из вашей истории платежей.

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/payment-history-controller-v-2/getPaymentHistoryByTransactionUsingGET_1)

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/transactions/9112223344" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /payment-history/v2/transactions/9112223344 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

~~~python
import requests

# История платежей - информация по транзакции
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
             <li><strong>transactionId</strong> - номер транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].txnId</i> в ответе)</li>
             <li><strong>type</strong> - тип транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].type</i> в ответе). Данный параметр является необязательным</li>
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

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

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
    "comment": null,
    "currencyRate": 1,
    "extras": [],
    "chequeReady": false,
    "bankDocumentAvailable": false,
    "bankDocumentReady": false,
    "repeatPaymentEnabled": false,
    "favoritePaymentEnabled": false,
    "regularPaymentEnabled": false
}
~~~

~~~python
# номер кошелька в формате 79992223344
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# История платежей - информация по транзакции
transactionInfo = payment_history_transaction(api_access_token, '11181101215', 'OUT')

# История платежей - информация по транзакции из истории платежей
lastPayments = payment_history_last(mylogin, api_access_token, '20','','')
last_txn_id = lastPayments['data'][5]['txnId']
last_txn_type = lastPayments['data'][5]['type']

transactionInfo = payment_history_transaction(api_access_token, str(last_txn_id), last_txn_type)
~~~

Успешный JSON-ответ содержит [объект Transaction](#txnid) с данными о транзакции.


## Квитанция платежа {#payment_receipt}

Данный запрос используется для получения электронной квитанции (чека) по определенной транзакции из вашей истории платежей в формате PDF/JPEG в виде файла или почтовым сообщением на заданный e-mail.

### Файл квитанции {#receipt_file}

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/cheque-controller-v-1/getChequeBytesUsingGET)

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/transactions/9112223344/cheque/file?type=IN&format=PDF" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /payment-history/v1/transactions/9112223344/cheque/file?type=IN&format=PDF HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

~~~python
import requests

# История платежей - получение текста чека в файле
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
             <li><strong>transactionId</strong> - номер транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].txnId</i> в ответе)</li>
             <li><strong>type</strong> - тип транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].type</i> в ответе)</li>
             <li><strong>format</strong> - тип файла, в который сохраняется квитанция. Допустимые значения: <i>JPEG</i>, <i>PDF</i></li>
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

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

[
  ""
]
~~~

Успешный JSON-ответ содержит файл выбранного формата в бинарном виде.

### Отправка квитанции {#receipt_send}

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/cheque-controller-v-1/sendChequeUsingPOST)

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/payment-history/v1/transactions/9112223344/cheque/send?type=IN" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
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

# История платежей - отправить чек на email
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
             <li><strong>transactionId</strong> - номер транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].txnId</i> в ответе)</li>
             <li><strong>type</strong> - тип транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].type</i> в ответе)</li>
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
    <li><h3>Параметр</h3><span>Данный параметр передается в JSON-теле запроса:</span>
    </li>
</ul>


Параметр|Тип|Описание
--------|----|----
email|String| Адрес для отправки электронной квитанции

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 201 Created
Content-Type: application/json
~~~

~~~python
# номер кошелька в формате 79992223344
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

lastPayments = payment_history_last(mylogin, api_access_token, '20','','')
last_txn_id = lastPayments['data'][5]['txnId']
last_txn_type = lastPayments['data'][5]['type']

# История платежей - отправить чек на email
payment_history_cheque_send(str(last_txn_id), last_txn_type, 'mmd@yandex.ru', api_access_token)
~~~

Успешный JSON-ответ содержит HTTP-код результата операции отправки файла.

## Модели данных API {#history_model}

### Класс Transaction {#txnid}

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
    "comment": null,
    "currencyRate": 1,
    "extras": [],
    "chequeReady": false,
    "bankDocumentAvailable": false,
    "bankDocumentReady": false,
    "repeatPaymentEnabled": false,
    "favoritePaymentEnabled": false,
    "regularPaymentEnabled": false
}
~~~

Объект, описывающий существующую транзакцию в сервисе QIWI Кошелек.

Параметр|Тип|Описание
--------|----|----
txnId | Integer |ID транзакции в сервисе QIWI Кошелек
personId|Integer|Номер кошелька
date|DateTime|Для запросов истории платежей - Дата/время платежа, во временной зоне запроса (см. параметр `startDate`). Формат даты `ГГГГ-ММ-ДД'T'чч:мм:сс+03:00`<br>Для запросов данных о транзакции - Дата/время платежа, время московское (в формате `ГГГГ-ММ-ДД'T'чч:мм:сс+03:00`)
errorCode|Number(Integer)|[Код ошибки платежа](#errorCode)
error| String| Описание ошибки
type | String| Тип платежа. Возможные значения:<br>`IN` - пополнение, <br>`OUT` - платеж, <br>`QIWI_CARD` - платеж с карт QIWI (QVC, QVP).
status|String|Статус платежа. Возможные значения:<br>`WAITING` - платеж проводится, <br>`SUCCESS` - успешный платеж, <br>`ERROR` - ошибка платежа.
statusText|String |Текстовое описание статуса платежа
trmTxnId|String|Клиентский ID транзакции
account| String|Для платежей - номер счета получателя. Для пополнений - номер отправителя, терминала или название агента пополнения кошелька
sum|Object| Данные о сумме платежа или пополнения.
-----|-----|-----
sum.amount|Number(Decimal)|сумма платежа
sum.currency|String|валюта платежа
-----|-----|-----
commission|Object| Данные о комиссии платежа
-----|-----|-----
commission.amount|Number(Decimal)|сумма
commission.currency|String|валюта
-----|-----|-----
total|Object| Данные о фактической сумме платежа или пополнения.
-----|-----|-----
total.amount|Number(Decimal)|сумма (равна сумме платежа `sum.amount` и комиссии `commission.amount`)
total.currency|String|валюта
-----|-----|-----
provider|Object| Данные о провайдере.
-----|-----|-----
provider.id|Integer|ID провайдера в QIWI Wallet
provider.shortName|String|краткое наименование провайдера
provider.longName|String|развернутое наименование провайдера
provider.logoUrl|String|ссылка на логотип провайдера
provider.description|String|описание провайдера (HTML)
provider.keys|String|список ключевых слов
provider.siteUrl|String|сайт провайдера
-----|-----|-----
source|Object|Служебная информация
comment|String|Комментарий к платежу
currencyRate|Number(Decimal)|Курс конвертации (если применяется в транзакции)
extras|Object|Служебная информация
chequeReady| Boolean|Специальное поле
bankDocumentAvailable|Boolean|Специальное поле
repeatPaymentEnabled|Boolean|Специальное поле
favoritePaymentEnabled|Boolean|Специальное поле
regularPaymentEnabled|Boolean|Специальное поле
