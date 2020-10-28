# API QIWI Мастер {#qiwi-master}

###### Последнее обновление: 2020-10-09 | [Предложить правки на GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/_master_ru.html.md)

API предоставляет доступ к управлению пакетом QIWI Мастер. Данный пакет услуг позволяет выпускать до пяти бесплатных виртуальных карт QIWI и перевыпускать карты неограниченное число раз.

Для вызова методов API вам потребуется токен API QIWI Wallet с разрешениями на действия *Управление виртуальными картами*, *Запрос информации о профиле кошелька*, *Просмотр истории платежей*, *Проведение платежей без SMS*. Отметьте данные разрешения при [выпуске токена API QIWI Wallet](#auth_data). 

![Token Scopes](/images/apiwallet_token_scopes_qiwi-master.jpg)

## Покупка пакета QIWI Мастер {#buy-qiwi-master}

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST 'https://edge.qiwi.com/sinap/api/v2/terms/28004/payments' \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  -d '{ \
        "id":"1600884280003", \
        "sum": { \
          "amount":2999, \
          "currency":"643" \
        }, \
        "paymentMethod": { \
          "type":"Account", \
          "accountId":"643" \
        },
        "comment":"test", \
        "fields": { \
          "account":"79121112233", \
          "vas_alias":"qvc-master" \
        } \
      }'
~~~

~~~http
POST /sinap/api/v2/terms/28004/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com

{
 "id":"1600884280003",
 "sum": {
  "amount":2999,
  "currency":"643"
 },
 "paymentMethod": {
  "type":"Account",
  "accountId":"643"
 },
 "comment":"test",
 "fields": {
   "account":"79121112233",
   "vas_alias":"qvc-master"
 }
}
~~~

~~~python
import requests
import time

# Перевод на QIWI Кошелек
def buy_qiwi_master(api_access_token, qw):
    s = requests.Session()
    s.headers = {'content-type': 'application/json'}
    s.headers['authorization'] = 'Bearer ' + api_access_token
    s.headers['User-Agent'] = 'Android v3.2.0 MKT'
    s.headers['Accept'] = 'application/json'
    postjson = {"id":"","sum":{"amount":"","currency":""},"paymentMethod":{"type":"Account","accountId":"643"}, "fields":{"account":"", "vas_alias":"qvc-master"}}
    postjson['id'] = str(int(time.time() * 1000))
    postjson['sum']['amount'] = 2999
    postjson['sum']['currency'] = '643'
    postjson['fields']['account'] = qw
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/28004/payments',json = postjson)
    return res.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/28004/payments</span></h3></li>
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
    <li><h3>Параметры</h3><span>В теле запроса передается JSON-объект <a href="#payment_obj">Payment</a>.<br>Набор реквизитов платежа в поле <code>fields</code>:</span>
    </li>
</ul>

Название|Тип|Описание|Обяз.
--------|----|----|------
fields.account| String|Номер кошелька для покупки пакета QIWI Мастер |+
fields.vas_alias| String|Строка "qvc-master"|+

<h3 class="request">Ответ ←</h3>

~~~python
print(buy_qiwi_master(mylogin,api_access_token,'+79261112233','comment',99.01))

>> Response
{'fields': {'account': '79261112233'},
 'id': '1514296828893',
 'source': 'account_643',
 'sum': {'amount': 2999.00, 'currency': '643'},
 'terms': '28004',
 'transaction': {'id': '11982501857', 'state': {'code': 'Accepted'}}}
~~~

Успешный JSON-ответ содержит [объект PaymentInfo](#payment_info) с данными о принятом платеже.

## Выпуск виртуальной карты QIWI Мастер {#qiwi-master-issue-card}

Для выпуска виртуальной карты к пакету QIWI Мастер вам необходимо выполнить следующие запросы в указанной последовательности.

### Создание заказа {#order-card}

~~~http
POST /cards/v2/persons/78000008024/orders HTTP/1.1
Accept: application/json
Authorization: Bearer f80f0875d8e45af7bdd244c7df3f1a3f
Content-Type: application/json
Host: edge.qiwi.com

{
 "cardAlias": "qvc-cpa"
}
~~~

> Ответ

~~~json
{
   "id": "<номер заказа>",
   "cardAlias": "qvc-cpa",
   "status": "DRAFT",
   "price": null,
   "cardId": null
}
~~~

Отправьте POST-запрос с токеном API QIWI кошелька на адрес 

`/cards/v2/persons/<номер кошелька>/orders`

В ссылке запроса укажите номер кошелька с пакетом QIWI Мастер. В теле запроса укажите JSON с параметром:

Название|Тип|Описание|Обяз.
--------|----|----|------
cardAlias| String|Строка "qvc-cpa"|+

В ответе придет номер заказа. Формат ответа - JSON:

Поле ответа|Тип|Описани
--------|----|----
id| String|Номер заказа
cardAlias| String|Строка "qvc-cpa"
status|String| Статус заказа
price|Object|Не заполняется
cardId|String|Не заполняется


### Подтверждение заказа {#card-order-confirm}

~~~http
PUT /cards/v2/persons/78000008024/orders/920fa383-6209-4743-a5d1-883f473f7f95/submit HTTP/1.1
Accept: application/json
Authorization: Bearer f80f0875d8e45af7bdd244c7df3f1a3f
Content-Type: application/json
Host: edge.qiwi.com

~~~

> Ответ, если карта бесплатная

~~~json
{
   "id": "<номер заказа>",
   "cardAlias": "qvc-cpa",
   "status": "COMPLETED",
   "price": {
     "amount": 0,
     "currency": 643
   },
   "cardId": <ID карты> 
}
~~~

> Ответ, если карта платная

~~~json
{
   "id": "<номер заказа>",
   "cardAlias": "qvc-cpa",
   "status": "PAYMENT_REQUIRED",
   "price": {
     "amount": <стоимость>,
     "currency": 643
   },
   "cardId": null
}
~~~

Отправьте PUT-запрос с токеном API QIWI кошелька  на адрес:

`/cards/v2/persons/<номер кошелька>/orders/<номер заказа из ответа предыдущего шага>/submit`

В ссылке запроса укажите номер кошелька с пакетом QIWI Мастер и номер заказа из ответа предыдущего шага (поле `id`). В теле запроса ничего не указывайте.

В ответе придет статус заказа. Формат ответа - JSON:

Поле ответа|Тип|Описани
--------|----|----
id| String|Номер заказа
cardAlias| String|Строка "qvc-cpa"
status|String| Статус заказа. Если карта бесплатная, то `COMPLETED`. Если карта платная, то `PAYMENT_REQUIRED`.
price|Object|Сведения о платеже
---|---|---
amount|Number| Сумма покупки
currency|Number|Валюта платежа
-----|---|----
cardId|String|Номер карты. **Не заполняется, если карта платная**.


### Покупка карты {#card-buy}

~~~http
POST /sinap/api/v2/terms/32064/payments HTTP/1.1
Accept: application/json
Authorization: Bearer 68944212761e25f6fce457661cabba6c
Content-Type: application/json
Host: edge.qiwi.com

{
 "id": "1600884290004",
 "sum": {
   "amount": 99,
   "currency": "643"
 },
 "paymentMethod": {
   "type": "Account",
   "accountId": "643"
 },
 "fields": {
   "account": "78000008024",
   "order_id":"920fa383-6209-4743-a5d1-883f473f7f95"
 }
}
~~~


Отправьте POST-запрос с токеном API QIWI кошелька  на адрес 

`/sinap/api/v2/terms/32064/payments`

В теле запроса передается JSON-объект <a href="#payment_obj">Payment</a>. Набор реквизитов платежа в поле `fields`:

Название|Тип|Описание|Обяз.
--------|----|----|------
fields.account| String|Номер кошелька |+
fields.order_id| String| Номер заказа карты из ответа на [запрос](#order-card) |+


Успешный JSON-ответ содержит [объект PaymentInfo](#payment_info) с данными о принятом платеже.

## Список карт QIWI Мастер {#qiwi-master-list}

~~~http
GET /cards/v1/cards?vas-alias=qvc-master HTTP/1.1
Accept: application/json
Authorization: Bearer b15ba2d82db883697e8a35877e60e680
Host: edge.qiwi.com
~~~

Запрос позволяет вам получить список всех ваших карт, выпущенных в QIWI по тарифу QIWI Мастер.

Отправьте GET-запрос с токеном API QIWI Кошелька на адрес

`/cards/v1/cards/?vas-alias=qvc-master`

~~~json
[
  {
    "qvx": {
      "id": 133789472,
      "maskedPan": "****9078",
      "status": "ACTIVE",
      "cardExpire": "2022-01-31T00:00:00+03:00",
      "cardType": "VIRTUAL",
      "cardAlias": "Facebook",
      "cardLimit": null,
      "activated": "2020-01-29T11:10:59+03:00",
      "smsResended": "2020-01-29T11:35:01+03:00",
      "postNumber": null,
      "blockedDate": null,
      "fullPan": null,
      "cardId": 2001291110576200000,
      "txnId": "2001291110576200000",
      "cardExpireMonth": "01",
      "cardExpireYear": "2022"
    },
    "balance": null,
    "info": {
      "id": 12,
      "name": "Виртуальная карта QIWI",
      "alias": "qvc-cpa",
      "price": {
        "amount": 99.0000,
        "currency": 643
      },
      "period": "за год",
      "type": "QVC_CPA",
      "details": {
        "info": "99 ₽, действует 1 год",
        "description": "",
        "tariffLink": "https://static.qiwi.com/qcms/files/1582791401478_5_JJ5vJe1L0szXzKb.pdf",
        "offerLink": "https://static.qiwi.com/ru/doc/qvc.pdf",
        "features": [
          "Покупки без комиссии"
        ],
        "requisites": [
          {
            "name": "Получатель",
            "value": "КИВИ Банк (АО)"
          },
          {
            "name": "ИНН",
            "value": "3123011520"
          },
          {
            "name": "Банк получателя",
            "value": "КИВИ Банк (АО)"
          },
          {
            "name": "БИК",
            "value": "044525416"
          },
          {
            "name": "КПП",
            "value": "772601001"
          },
          {
            "name": "Счет",
            "value": "47416810600000000004"
          },
          {
            "name": "Корр. счет",
            "value": "30101810645250000416 (открыт в ГУ Банка России по Центральному федеральному округу)"
          },
          {
            "name": "Назначение платежа",
            "value": "Пополнение QIWI Кошелька\nN +79258150000"
          }
        ]
      },
      "features": []
    }
  }
]
~~~

В ответ придет JSON массив с данными выпущенных карт.

Поле ответа | Тип | Описание
----|-----|-----
qvx | Object | Общая информация о карте
----|-----|----
id | Number | ID карты
maskedPan | String | Маскированный номер карты (отображаются только последние 4 цифры)
status | String | Текущий статус карты. Возможные значения: "ACTIVE", "SENDED_TO_BANK", "SENDED_TO_USER", "BLOCKED", "UNKNOWN"
cardExpire |String | Срок действия карты
cardType | String | Тип карты: "VIRTUAL" (виртуальная), "PLASTIC" (обычная)
cardAlias | String | Пользовательское имя карты
cardLimit | Object | Лимиты на карту
-----|-----|------
value | Number | Значение лимита 
currencyCode | Number | Код валюты по ISO
----|---|---
activated | String | Дата активации карты
smsResended | String | Дата высылки СМС с реквизитами
postNumber | String | Номер почтового отправления (для обычных карт)
blockedDate | String | Дата блокировки
txnId | String | ID транзакции заказа карты
cardExpireMonth | String | Месяц окончания действия карты
cardExpireYear | String | Год окончания действия карты
----|---|---
balance | Object | Данные баланса карты
---|----|-----
amount|Number| Сумма баланса
currency|Number| Код валюты баланса (по ISO)
----|-----|----
info | Object | Тарифы и банковские реквизиты карты
-----|-----|----
price|Object|Тариф карты
-----|-----|----
amount|Number|Стоимость обслуживания
currency|Number| Код валюты баланса (по ISO)
-----|-----|----
period|String|Период обслуживания (по тарифу)
tariffLink | String | Ссылка на описание тарифа
offerLink | String | Ссылка на договор оферты на выпуск карты
requisites | Array | Список пар "ключ-значение" с данными банковских реквизитов для пополнения карты

## Выписка по карте {#card-payments}

~~~http
GET /payment-history/v1/persons/78000008024/cards/158618787/statement?from=2020-01-01T00%3A00%3A00%2B03%3A00&till=2020-09-23T23%3A59%3A59%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer b15ba2d82db883697e8a35877e60e680
Host: edge.qiwi.com
~~~

Запрос позволяет вам получить операции по определенной карте за указанный период в тарифе QIWI Мастер.

<aside class="notice">Выписку можно запросить за период не более 90 дней</aside>

Отправьте GET-запрос с токеном API QIWI Кошелька на адрес:

`/payment-history/v1/persons/<номер пользователя>/cards/<ID карты>/statement?from=<дата начала>&till=<дата окончания>`

В ссылке укажите номер кошелька, ID карты, полученный при выпуске или из ответа на запрос списка карт, и интервал дат для выписки.

Успешный JSON-ответ содержит файл с выпиской формата PDF в бинарном виде.

## Блокировка карты {#card-block}

~~~http
PUT /cards/v1/cards/158619365/block HTTP/1.1
Accept: application/json
Authorization: Bearer 68944212761e25f6fce457661cabba6c
Host: edge.qiwi.com
~~~

Запрос позволяет вам заблокировать выбранную карту тарифа QIWI Мастер. 
 
Отправьте PUT-запрос с токеном API QIWI Кошелька на адрес:

`/cards/v1/cards/<ID карты>/block`

 В ссылке укажите ID карты, полученный при выпуске или из ответа на запрос списка карт.
 
 В ответе придет JSON с результатом операции:
 
 Поле ответа | Тип | Описание
 ----|-----|-----
 status | String | Статус выполнения запроса (OK, FAIL)
 error | String | Описание ошибки

 
## Получение реквизитов карты {#card-details}

~~~http
PUT /cards/v1/cards/158619365/details HTTP/1.1
Accept: application/json
Authorization: Bearer 68944212761e25f6fce457661cabba6c
Content-Type: application/json
Host: edge.qiwi.com

{
   "operationId": "43555447-a026-4c17-b56d-6956a09249c9"
}

~~~

Отправьте PUT-запрос с токеном API QIWI Кошелька на адрес

`/cards/v1/cards/<ID карты>/details`

В теле запроса укажите JSON с параметром:

Название|Тип|Описание|Обяз.
--------|----|----|------
operationId| String|Произвольный UUID|+

В ответ придет JSON с PAN и CVV карты.

Поле ответа | Тип | Описание
----|-----|-----
status | String | Статус операции ("OK", "FAIL", "CONFIRMATION_REQUIRED", "CONFIRMATION_LIMIT_EXCEED")
cvv	| String | CVV карты
pan | String | PAN карты	
errorCode	| String | Код ошибки


## Переименование карты {#qvc-rename}


~~~http
PUT /cards/v1/cards/158619365/alias HTTP/1.1
Accept: application/json
Authorization: Bearer 68944212761e25f6fce457661cabba6c
Content-Type: application/json
Host: edge.qiwi.com

{
   "alias": "new card name"
}
~~~

Отправьте PUT-запрос с токеном API QIWI Кошелька на адрес

`/cards/v1/cards/<ID карты>/alias`

В теле запроса укажите JSON с параметром:

Название|Тип|Описание|Обяз.
--------|----|----|------
alias| String|Новое пользовательское имя карты|+

В ответ придет JSON со статусом операции.

Поле ответа | Тип | Описание
----|-----|-----
status | String | Статус операции ("OK", "FAIL")
error	| String | Текстовое описание ошибки
errorCode	| String | Код ошибки
