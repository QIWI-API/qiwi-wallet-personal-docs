---
title: QIWI Wallet API

search: true

metatitle: API QIWI Кошелька

metadescription: API QIWI Кошелька позволяет автоматизировать выполнение платежей и получение отчетов о платежах, информации о счёте.

toc_footers:
 - <a href='/'>На главную</a>
 - <a href='mailto:api_help@qiwi.com'>Обратная связь</a>
---

# Введение

API QIWI Кошелька позволяет автоматизировать получение информации о вашем счёте в [сервисе Visa QIWI Кошелек](https://qiwi.com) и проводить операции с его помощью.

Методы API доступны после регистрации пользователя в [сервисе Visa QIWI Кошелек](https://qiwi.com). 

## Служебные данные {#auth_param}

<ul class="nestedList params">
    <li><h3>Авторизация</h3>
    </li>
</ul>

Параметр|Описание|Тип
 ---------|--------|---
 token | [Токен](#auth_data) для авторизации запросов в API. Действие токена заканчивается через 1 месяц после [выпуска](#auth_data). | String


# Авторизация {#auth_api}

## Данные для авторизации {#auth_data}

API QIWI Wallet использует открытый протокол авторизации OAuth 2.0. Согласно протоколу, пользователь авторизуется или регистрируется на сайте <https://qiwi.com> и запрашивает токен с правом выполнения определённых действий. Выпуск токена подтверждается одноразовым кодом из СМС.

1. Откройте в браузере страницу <https://qiwi.com/api>. Для этого потребуется авторизоваться или зарегистрироваться в сервисе Visa QIWI Кошелек.
  ![Token Issue](images/apiwallet_get_token.png)
2. Нажмите **Выпустить новый токен**. Во всплывающем окне выберите разрешения на операции с токеном:
  ![Token Scopes](images/apiwallet_token_scopes.png)
    * Запрос информации о профиле кошелька - разрешает выполнение запросов [профиля пользователя](#profile)
    * Запрос баланса кошелька - разрешает выполнение запросов [баланса](#balance)
    * Просмотр истории платежей - разрешает выполнение запросов [истории платежей](#payments)
    * Проведение платежей без SMS - разрешает выполнение платежных запросов без подтверждения по SMS (независимо от настроек <https://qiwi.com/settings/options/security.action>)
3. Нажмите **Продолжить** и подтвердите выпуск токена кодом из SMS-сообщения.
4. Скопируйте строку токена и сохраните в безопасном месте. Используйте токен для [запросов к API QIWI Кошелька](#auth_ex).

Токен действует в течение 1 месяца с момента выпуска. Вы можете заблокировать токен, не дожидаясь окончания срока его действия, на странице <https://qiwi.com/settings/mine.action>.

## Пример авторизации {#auth_ex}

~~~shell
user@server:~$ curl "адрес сервера"
  --header "Authorization: Bearer jMyN22DQxMjM6NDUzRmRnZDQ0Mw11212e"
~~~

<aside class="notice">
Для авторизации в запрос добавляется заголовок Authorization, значение которого представлено как: "Bearer <a href="#auth_data">token</a>"
</aside>


* В результате авторизации на сайте Visa QIWI Кошелек и выпуска токена [получен](#auth_data) токен, представляющий собой строку:

`U1QtOTkwMTAyLWNud3FpdWhmbzg3M`

* Токен добавляется в заголовок `Authorization: Bearer `

* Итоговый заголовок, добавляемый в каждый запрос к API QIWI Кошелька: 

`Authorization: Bearer U1QtOTkwMTAyLWNud3FpdWhmbzg3M`

# Профиль пользователя {#profile}

~~~shell
user@server:~$ curl "https://edge.qiwi.com/person-profile/v1/profile/current"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9"
~~~

~~~http
GET /person-profile/v1/profile/current HTTP/1.1
Accept: application/json
Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9
Content-type: application/json
Host: edge.qiwi.com
~~~

Тип запроса - GET.

URL запроса:

`https://edge.qiwi.com/person-profile/v1/profile/current?<parameters>`

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

Параметры URL запроса:

* `<parameters>` - дополнительные параметры запроса (необязательные):

Параметр|Тип |Описание
--------|----|-------
authInfoEnabled|Boolean | Логический признак выгрузки настроек авторизации пользователя. По умолчанию `true`
contractInfoEnabled|Boolean | Логический признак выгрузки данных о кошельке пользователя. По умолчанию `true`
userInfoEnabled|Boolean | Логический признак выгрузки прочих пользовательских данных. По умолчанию `true`

## Формат ответа

~~~json
{
  "authInfo": {
    "boundEmail": "m@ya.ru",
    "ip": "81.210.201.22",
    "lastLoginDate": "2017-07-27T06:51:06.099Z",
    "mobilePinInfo": {
      "lastMobilePinChange": "2017-07-13T11:22:06.099Z",
      "mobilePinUsed": true,
      "nextMobilePinChange": "2017-11-27T06:51:06.099Z"
    },
    "passInfo": {
      "lastPassChange": "2017-07-21T09:25:06.099Z",
      "nextPassChange": "2017-08-21T09:25:06.099Z",
      "passwordUsed": true
    },
    "personId": 79683851815,
    "pinInfo": {
      "pinUsed": true
    },
    "registrationDate": "2017-01-07T16:51:06.100Z"
  },
  "contractInfo": {
    "blocked": false,
    "contractId": 79683851815,
    "creationDate": "2017-01-07T16:51:06.100Z",
    "features": [
      ...
    ],
    "identificationInfo": [
      {
        "bankAlias": "AKB",
        "identificationLevel": "ANONYMOUS"
      },
      {
        "bankAlias": "QIWI",
        "identificationLevel": "SIMPLE"
      }
    ]
  },
  "userInfo": {
    "defaultPayCurrency": 643,
    "defaultPaySource": 7,
    "email": null,
    "firstTxnId": 10807097143,
    "language": "string",
    "operator": "Beeline",
    "phoneHash": "lgsco87234f0287",
    "promoEnabled": null
  }
}
~~~

Успешный ответ содержит данные о профиле пользователя:

Параметр|Тип|Описание
--------|----|----
authInfo|Object|Настройки авторизации пользователя
personId|Number|Номер кошелька пользователя
registrationDate|String|Дата/время регистрации QIWI Кошелька пользователя (через сайт либо мобильноое приложение, либо другим способом)
boundEmail|String|E-mail, привязанный к кошельку. Если отсутствует, то `null`
ip|String|IP-адрес последней пользовательской сессии
lastLoginDate|String|Дата/время последней сессии в QIWI Кошельке
mobilePinInfo|Object|Данные о PIN-коде мобильного приложения QIWI Кошелька
mobilePinUsed|Boolean|Логический признак использования PIN-кода (фактически означает, что мобильное приложение используется)
lastMobilePinChange|String|Дата/время последнего изменения PIN-кода мобильного приложения QIWI Кошелька
nextMobilePinChange|String|Дата/время следующего (планового) изменения PIN-кода мобильного приложения QIWI Кошелька
passInfo|Object|Данные о пароле к сайту qiwi.com
passwordUsed|Boolean|Логический признак использования пароля (фактически означает, что пользователь заходит на сайт)
lastPassChange|String|Дата/время последнего изменения пароля сайта qiwi.com
nextPassChange|String|Дата/время следующего (планового) изменения пароля сайта qiwi.com
pinInfo|Object|Данные о PIN-коде к приложению QIWI Кошелька на QIWI терминалах
pinUsed|Boolean|Логический признак использования PIN-кода (фактически означает, что пользователь заходил в приложение)
contractInfo|Object| Информация о кошельке пользователя
blocked|Boolean|Логический признак блокировки кошелька
contractId|Number|Номер кошелька пользователя
creationDate|String|Дата/время создания QIWI Кошелька пользователя (через сайт либо мобильноое приложение, либо при первом пополнении, либо другим способом)
features|Array[Object]|Служебная информация
identificationInfo|Array[Object]|Данные об [идентификации](https://qiwi.com/settings/account/identification.action) пользователя.
bankAlias|String|Акроним системы, в которой пользователь получил идентификацию. `QIWI` - QIWI Кошелек, `AKB` - QIWI Кошелек в Казахстане.
identificationLevel|Уровень идентификации. `ANONYMOUS` - без идентификации, `SIMPLE` - упрощенная идентификация, `FULL` - полная идентификация.
userInfo|Object|Прочие пользовательские данные
defaultPayCurrency|Number|Код валюты баланса кошелька по умолчанию (number-3 ISO-4217)
defaultPaySource|Number|Служебная информация
email|String|E-mail пользователя
firstTxnId|Number|Номер первой транзакции пользователя после регистрации
language|String|Служебная информация
operator|String|Название мобильного оператора номера пользователя
phoneHash|String|Служебная информация
promoEnabled|String|Служебная информация


# История платежей {#payments}

История платежей и пополнений вашего кошелька.

Тип запроса - GET.

~~~shell
Пример 1. Последние 10 платежей

user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments?rows=10"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"

Пример 2. Платежи за 10.05.2017

user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments?rows=50&startDate=2017-05-10T00%3A00%3A00Z&endDate=2017-05-10T23%3A59%3A59Z"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"

Пример 3. Продолжение списка платежей (в предыдущем запросе истории возвращены параметры nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23+03:00)

user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12:35:23Z"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~text
Пример 1. Последние 10 платежей с рублевого баланса и с привязанной карты
~~~

~~~http
GET /payment-history/v1/persons/79112223344/payments?rows=10&operation=OUT&sources[0]=QW_RUB&sources[1]=CARD HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

~~~text
Пример 2. Платежи за 10.05.2017
~~~

~~~http
GET /payment-history/v1/persons/79112223344/payments?rows=50&startDate=2017-05-10T00%3A00%3A00Z&endDate=2017-05-10T23%3A59%3A59Z HTTP/1.1

Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

~~~text
Пример 3. Продолжение списка платежей 

(в предыдущем запросе истории возвращены параметры nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23)
~~~

~~~http
GET /payment-history/v1/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12%3A35%3A23Z HTTP/1.1

Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

URL запроса:

`https://edge.qiwi.com/payment-history/v1/persons/<wallet>/payments?<parameters>`

Параметры URL запроса:

* `<wallet>` - номер кошелька, для которого получена авторизация (с международным префиксом, но без `+`), **обязательный параметр**
* `<parameters>` - дополнительные параметры запроса (см. ниже)

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

Параметры запроса: 

Параметр|Тип|Описание
--------|----|----
rows | Integer |Число платежей в ответе, для разбивки отчета на части. Целое число от 1 до 50. **Обязательный параметр**
operation|String| Тип операций в отчете, для отбора. Допустимые значения:<br>`ALL` - все операции, <br>`IN` - только пополнения, <br>`OUT` - только платежи, <br>`QIWI_CARD` - только платежи по картам QIWI (QVC, QVP). <br>По умолчанию `ALL`
sources|Array[String]|Источники платежа, для отбора. Каждый источник задается как отдельный параметр и нумеруется элементом массива, начиная с нуля (`sources[0]`, `sources[1]` и т.д.). Допустимые значения: <br>`QW_RUB` - рублевый счет кошелька, <br>`QW_KZT` - счет кошелька в тенге, <br>`QW_USD` - счет кошелька в долларах, <br>`CARD` - привязанные и непривязанные к кошельку банковские карты, <br>`MK` - счет мобильного оператора. Если не указаны, учитываются все источники
startDate | DateTime URL-encoded| Начальная дата поиска платежей (формат ГГГГ-ММ-ДДTчч:мм:ссZ). По умолчанию, равен вчерашней дате. **Используется только вместе с `endDate`**
endDate | DateTime URL-encoded | Конечная дата поиска платежей (формат ГГГГ-ММ-ДДTчч:мм:ссZ). По умолчанию, равен текущей дате. **Используется только вместе с `startDate`**
nextTxnDate | DateTime URL-encoded| Дата транзакции (формат ГГГГ-ММ-ДДTчч:мм:ссZ), для отсчета от предыдущего списка (см. параметр `nextTxnDate` в ответе). Используется только вместе с `nextTxnId`
nextTxnId | Long | Номер предшествующей транзакции, для отсчета от предыдущего списка (см. параметр `nextTxnId` в ответе). **Используется только вместе с `nextTxnDate`**


<aside class="notice">Максимальный допустимый интервал между startDate и endDate - 90 календарных дней.</aside>

## Формат ответа

~~~json
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
                       "id":26476,
                       "shortName":"Yandex.Money",
                       "longName":"ООО НКО \"Яндекс.Деньги\"",
                       "logoUrl":"https://static.qiwi.com/img/providers/logoBig/26476_l.png",
                       "description":"Яндекс.Деньги",
                       "keys":"***",
                       "siteUrl":null
                      },
    "comment":null,
    "currencyRate":1,
    "extras":null,
    "chequeReady":true,
    "bankDocumentAvailable":false,
    "bankDocumentReady":false,
                "repeatPaymentEnabled":false
    }
  ],
  "nextTxnId":9001,
  "nextTxnDate":"2017-01-31T15:24:10+03:00"
}
~~~

Успешный ответ содержит список платежей из истории кошелька, соответствующих заданному фильтру:

Параметр|Тип|Описание
--------|----|----
data|Array[Object]|Массив платежей. <br>Число платежей равно параметру `rows` из запроса
txnId | Integer |ID транзакции в процессинге
personId|Integer|Номер кошелька
date|DateTime|Дата/время платежа (в формате ГГГГ-ММ-ДДTчч:мм:ссTMZ)
errorCode|Integer|Код ошибки платежа
error| String| Описание ошибки
status|String|Статус платежа. Возможные значения:<br>`WAITING` - платеж проводится, <br>`SUCCESS` - успешный платеж, <br>`ERROR` - ошибка платежа.
type | String| Тип платежа (соответствует параметру запроса `operation`)
statusText|String |Описание статуса платежа
trmTxnId|String|Клиентский ID транзакции
account| String|Номер счета получателя
sum|Object| Объект с суммой платежа. Параметры:
amount|Decimal|сумма, 
currency|String|валюта 
commission|Object| Объект с комиссией платежа. Параметры:
amount|Decimal|сумма, 
currency|String|валюта 
total|Object| Объект с общей суммой платежа. Параметры:
amount|Decimal|сумма, 
currency|String|валюта 
provider|Object| Объект с описанием провайдера. Параметры:
id|Integer|ID провайдера в процессинге,
shortName|String|краткое наименование провайдера,
longName|String|развернутое наименование провайдера,
logoUrl|String|ссылка на логотип провайдера,
description|String|описание провайдера (HTML),
keys|String|список ключевых слов,
siteUrl|String|сайт провайдера
comment|String|Комментарий к платежу
currencyRate|Decimal|Курс конвертации (если применяется в транзакции)
extras|Object|Дополнительные поля платежа, список не фиксирован
chequeReady| Boolean|Специальное поле
bankDocumentAvailable|Boolean|Специальное поле
bankDocumentReady|Boolean|Специальное поле
repeatPaymentEnabled|Boolean|Специальное поле
nextTxnId|Integer|ID следующей транзакции в полном списке
nextTxnDate|DateTime|Дата/время следующей транзакции в полном списке

## Статистика платежей {#stat}

Для получения статистики по суммам платежей за заданный период используется подзапрос запроса истории.

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments/total?startDate=2017-03-01T00%3A00%3A00Z&endDate=2017-03-31T11%3A44%3A15Z"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /payment-history/v1/persons/79112223344/payments/total?startDate=2017-03-01T00%3A00%3A00Z&endDate=2017-03-31T11%3A44%3A15Z HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

Тип запроса - GET.

URL запроса:

`https://edge.qiwi.com/payment-history/v1/persons/<wallet>/payments/total?<parameters>`

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

Параметры URL запроса:

* `<wallet>` - номер кошелька, для которого получена авторизация (с международным префиксом, но без знака "+"), **обязательный параметр**
* `<parameters>` - дополнительные параметры запроса:

Параметр|Тип |Описание
--------|----|-------
startDate|DateTime URL-encoded | Начальная дата периода статистики (в формате ГГГГ-ММ-ДДTчч:мм:ссZ). **Обязательный параметр**
endDate|DateTime URL-encoded| Конечная дата периода статистики (в формате ГГГГ-ММ-ДДTчч:мм:ссZ). **Обязательный параметр**
operation|String| Тип операций, учитываемых при подсчете статистики. Допустимые значения:<br>`ALL` - все операции, <br>`IN` - только пополнения, <br>`OUT` - только платежи, <br>`QIWI_CARD` - только платежи по картам QIWI (QVC, QVP). <br>По умолчанию `ALL`.|Нет
sources|Array[String]|Источники платежа, учитываемые при подсчете статистики. Каждый источник задается как отдельный параметр и нумеруется элементом массива, начиная с нуля (`sources[0]`, `sources[1]` и т.д.). Допустимые значения: <br>`QW_RUB` - рублевый счет кошелька, <br>`QW_KZT` - счет кошелька в тенге, <br>`QW_USD` - счет кошелька в долларах, <br>`CARD` - привязанные и непривязанные к кошельку банковские карты, <br>`MK` - счет мобильного оператора. Если не указан, учитываются все источники платежа.|Нет


<aside class="notice">Максимальный период для выгрузки статистики - 90 календарных дней.</aside>

~~~json
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

Успешный ответ содержит статистику платежей за соответствующий период:

Параметр|Тип|Описание
--------|----|----
incomingTotal|Array[Object]|Данные о входящих платежах (пополнениях), отдельно по каждой валюте
amount | Decimal |Сумма пополнений за период
currency|String|Валюта пополнений
outgoingTotal|Array[Object]|Данные об исходящих платежах, отдельно по каждой валюте
amount | Decimal |Сумма платежей за период
currency|String|Валюта платежей


# Баланс QIWI Кошелька {#balance}

~~~shell
user@server:~$ curl "https://edge.qiwi.com/funding-sources/v1/accounts/current"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /funding-sources/v1/accounts/current HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

Тип запроса - GET.

URL запроса:

`https://edge.qiwi.com/funding-sources/v1/accounts/current`

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

## Формат ответа

~~~json
{
    "accounts": [
        {
            "alias": "mc_beeline_rub",
            "fsAlias": "qb_mc_beeline",
            "title": "MC",
            "type": {
                "id": "MC",
                "title": "Счет мобильного кошелька"
            },
            "hasBalance": false,
            "balance": null,
            "currency": 643
        },
        {
            "alias": "qw_wallet_rub",
            "fsAlias": "qb_wallet",
            "title": "WALLET",
            "type": {
                "id": "WALLET",
                "title": "Visa QIWI Wallet"
            },
            "hasBalance": true,
            "balance": {
                "amount": 8.74,
                "currency": 643
            },
            "currency": 643
        },
        {
            "alias": "qw_wallet_kzt",
            "fsAlias": "akb_wallet",
            "title": "WALLET",
            "type": {
                "id": "WALLET",
                "title": "Visa QIWI Wallet"
            },
            "hasBalance": true,
            "balance": {
                "amount": 0.53,
                "currency": 398
            },
            "currency": 398
        },
        {
            "alias": "qw_wallet_usd",
            "fsAlias": "qb_wallet",
            "title": "WALLET",
            "type": {
                "id": "WALLET",
                "title": "Visa QIWI Wallet"
            },
            "hasBalance": true,
            "balance": {
                "amount": 0,
                "currency": 840
            },
            "currency": 840
        },
        {
            "alias": "qw_wallet_eur",
            "fsAlias": "qb_wallet",
            "title": "WALLET",
            "type": {
                "id": "WALLET",
                "title": "Visa QIWI Wallet"
            },
            "hasBalance": true,
            "balance": {
                "amount": 0.01,
                "currency": 978
            },
            "currency": 978
        }
    ]
}
~~~

Успешный ответ содержит список счетов для фондирования платежей и их текущие балансы:

Параметр|Тип|Описание
--------|----|----
accounts|Array[Object]|Массив балансов
alias | String |Псевдоним пользовательского баланса
fsAlias | String |Псевдоним банковского баланса
title|String|Название соответствующего счета кошелька
hasBalance|Boolean|Логический признак реального баланса в системе QIWI Кошелек (не привязанная карта, не счет мобильного телефона и т.д.)
currency | Number| Код валюты баланса (number-3 ISO-4217). Возвращаются балансы в следующих валютах: 398 - казахский тенге, 643 - российский рубль, 840 - американский доллар, 978 - евро
type|Object|Сведения о счете
id, title| String| Описание счета
balance|Object |Сведения о балансе данного счета
amount|Decimal|Текущий баланс данного счета
currency | Number| Код валюты баланса (number-3 ISO-4217)

# Платежи

## Комиссионные тарифы {#commission}

Для провайдеров, оплата которых доступна через API, можно предварительно проверить комиссионные условия (стандартные тарифы).

Тип запроса - GET. Запрос не требует авторизации.

URL запроса:

`https://edge.qiwi.com/sinap/providers/{id}/form`

~~~shell
user@server:~$ curl "https://edge.qiwi.com/sinap/providers/99/form"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "User-Agent: ***"
~~~

~~~http
GET /sinap/providers/99/form HTTP/1.1
Content-Type: application/json
Accept: application/json
Host: edge.qiwi.com
Origin: qiwi.com
User-Agent: ****
~~~

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`

Параметр URL запроса:

* `{id}` - идентификатор провайдера. Возможные значения:
    * 99 - Перевод на Visa QIWI Wallet
    * 1963 - Перевод на карту Visa (карты российских банков)
    * 21013 - Перевод на карту MasterCard (карты российских банков)
    * Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:
        * 1960 – Перевод на карту Visa
        * 21012 – Перевод на карту MasterCard
    * 31652 - национальная платежная система МИР
    * 466 - Тинькофф Банк
    * 464 - Альфа-Банк
    * 821 - Промсвязьбанк
    * 815 - Русский Стандарт
    * [Идентификаторы операторов мобильной связи](#mnp)

### Формат ответа

~~~json
{
  "content": {
      "terms": {
         "commission": {
                "ranges": [{
                     "bound": 0,
                     "fixed": 50.0,
                     "rate": 0.02
                }]
           }
       }
    }
}
~~~

Успешный ответ содержит фрагмент с данными о комиссии:

Параметр | Тип | Описание
-----|-----|------
commission | Object | Объект, содержащий данные об условиях комиссий.
ranges | Array[Object] | Массив объектов с граничными условиями комиссий. Каждый объект содержит параметры:
bound |  Number | сумма платежа, начиная с которой применяется условие
rate | Number |  комиссия (абс.множитель)
min | Number | минимальная сумма комиссии
max | Number | максимальная сумма комиссии
fixed | Number | фиксированная сумма комиссии


## Перевод на QIWI Кошелек {#p2p}

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/terms/99/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/terms/99/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  --header "User-Agent: ***"
  -d '{"id":"11111111111111", "sum": {"amount":100, "currency":"643"}, "source": "account_643", 
       "paymentMethod": {"type":"Account", "accountId":"643"},
       "comment":"test", "fields": {"account":"+79121112233"}}'
~~~

~~~http
POST /sinap/terms/99/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
User-Agent: ****
 
{
	"id":"11111111111111",
	"sum": {
				"amount":100,
				"currency":"643"
			},
	"source": "account_643", 
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

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).
 
Тело запроса - JSON. Параметры запроса:

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Объект, содержащий данные о сумме платежа: 
amount|Decimal|Сумма
currency|String|Валюта (только `643`, рубли)
source| String| Источник фондирования платежа. Допускается только следующее значение:<br>`account_643` - рублевый счет QIWI Кошелька отправителя
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
type|String |Тип платежа, только `Account`
accountId|String| Идентификатор счета, только `643`.
comment| String| Комментарий к платежу.
fields|Object| Объект с параметрами перевода. Допускается только следующий параметр:
account| String|Номер телефона получателя (с международным префиксом)


### Формат ответа

~~~json
{
    "id": ":11111111111111",
 	"fields": {
    	    "account": "+79121112233"
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

Успешный ответ содержит данные о принятом платеже:

Параметр | Описание
-----|--------
id | ID транзакции (максимум 20 цифр)
sum|Копия объекта `sum` из исходного запроса.
fields|Копия объекта `fields` из исходного запроса.
source| Копия параметра `source` из исходного запроса.
transaction|Объект с данными о транзакции в процессинге. Параметры:
id|ID транзакции в процессинге
state|Объект содержит текущее состояние транзакции в процессинге. Содержит только параметр `code` со значением `Accepted` (платеж принят) 

## Оплата сотовой связи {#cell}

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/terms/{id}/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/terms/1/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  --header "User-Agent: ***"
  -d '{"id":"11111111111111", "sum": {"amount":100, "currency":"643"}, "source": "account_643", 
       "paymentMethod": {"type":"Account", "accountId":"643"}, "fields": {"account":"9161112233"}}'
~~~

~~~http
POST /sinap/terms/1/payments HTTP/1.1
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
  "source": "account_643", 
  "paymentMethod": {
      "type":"Account",
      "accountId":"643"
      },
  "fields": {
        "account":"9161112233"
      }
}
~~~

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

Параметр URL запроса:

* `{id}` - идентификатор провайдера. Определяется с помощью [проверки мобильного оператора](#mnp).
 
Тело запроса - JSON. Все перечисленные параметры обязательны: 

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Объект, содержащий данные о сумме платежа: 
amount|Decimal|Сумма
currency|String|Валюта (только `643`, рубли)
source| String| Источник фондирования платежа. Допускается только следующее значение:<br>`account_643` - рублевый счет QIWI Кошелька отправителя
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
type|String |Тип платежа, только `Account`
accountId|String| Идентификатор счета, только `643`.
fields|Object| Объект с параметрами перевода. Допускается только следующий параметр:
account| String|Номер мобильного телефона для пополнения (без префикса `8`)

~~~json
{
    "id": "21131343",
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

Успешный ответ содержит данные о принятом платеже:

Параметр | Описание
-----|--------
id | ID транзакции (максимум 20 цифр)
sum|Копия объекта `sum` из исходного запроса.
fields|Копия объекта `fields` из исходного запроса.
source| Копия параметра `source` из исходного запроса.
transaction|Объект с данными о транзакции в процессинге. Параметры:
id|ID транзакции в процессинге
state|Объект, содержит текущее состояние транзакции в процессинге. Содержит только параметр `code` со значением `Accepted` (платеж принят)

### Определение оператора {#mnp}

Предварительная проверка мобильного номера выполняется POST-запросом:

`https://qiwi.com/mobile/detect.action`

В ответе возвращается идентификатор оператора для [запроса пополнения телефона](#cell). Запрос не требует авторизации.

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/x-www-form-urlencoded`

~~~shell
user@server:~$ curl -X POST "https://qiwi.com/mobile/detect.action"
  --header "Accept: application/json" 
  --header "Content-Type: application/x-www-form-urlencoded"
  --header "User-Agent: ***"
  -d 'phone=+79651238341'
~~~

~~~http
POST /mobile/detect.action HTTP/1.1
Host: qiwi.com
Accept: application/json
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache

phone=79651238341
~~~

Параметр запроса:

Параметр|Тип|Описание
--------|----|----
phone | String |Мобильный номер в международном формате

**Формат ответа**

Успешная проверка - Ответ с HTTP Status 200, параметр `code.value` = 0. 

Идентификатор оператора находится в параметре `message`.

~~~json
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

Невозможно определить оператора - Ответ с HTTP Status 200, параметр `code.value` = 2.

~~~json
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


## Перевод на карту {#cards}

Выполняет перевод на карты платежных систем Visa или MasterCard. Предварительно можно [проверить тип платежной системы](#card_check).

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/terms/{id}/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/terms/1963/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  --header "User-Agent: ***"
  -d '{"id":"21131343", "sum": {"amount":1000, "currency":"643"}, "source": "account_643", 
       "paymentMethod": {"type":"Account", "accountId":"643"}, "fields": {"account":"4256********1231"}}'
~~~

~~~http
POST /sinap/terms/1963/payments HTTP/1.1
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
  "source": "account_643", 
  "paymentMethod": {
      "type":"Account",
      "accountId":"643"
      },
  "fields": {
        "account":"4256********1231"
      }
}
~~~

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

Параметр URL запроса:

* `{id}` - идентификатор платежной системы:
    * 1963 - Visa (карты российских банков)
    * 21013 - MasterCard (карты российских банков)
    * Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:
        * 1960 – Visa
        * 21012 – MasterCard
    * 31652 - национальная платежная система МИР

Тело запроса - JSON. Параметры запроса: 

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Объект, содержащий данные о сумме платежа: 
amount|Decimal|Сумма
currency|String|Валюта (только `643`, рубли)
source| String| Источник фондирования платежа. Допускается только следующее значение:<br>`account_643` - рублевый счет QIWI Кошелька отправителя
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
type|String |Тип платежа, только `Account`
accountId|String| Идентификатор счета, только `643`.
fields|Object| Объект с параметрами перевода. Допускается только следующий параметр:
account| String|Номер банковской карты получателя

~~~json
{
  "id": "21131343",
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

Успешный ответ содержит данные о принятом платеже:

Параметр | Описание
-----|--------
id | ID транзакции (максимум 20 цифр)
sum|Копия объекта `sum` из исходного запроса.
fields|Копия объекта `fields` из исходного запроса.
source| Копия параметра `source` из исходного запроса.
transaction|Объект с данными о транзакции в процессинге. Параметры:
id|ID транзакции в процессинге
state|Объект, содержит текущее состояние транзакции в процессинге. Содержит только параметр `code` со значением `Accepted` (платеж принят)

### Проверка карты {#card_check}

Определение платежной системы карты выполняется POST-запросом:

`https://qiwi.com/card/detect.action`

В ответе возвращается идентификатор платежной системы для [запроса перевода на карту](#cards). Запрос не требует авторизации.

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/x-www-form-urlencoded`

~~~shell
user@server:~$ curl -X POST "https://qiwi.com/card/detect.action"
  --header "Accept: application/json" 
  --header "Content-Type: application/x-www-form-urlencoded"
  --header "User-Agent: ***"
  -d 'cardNumber=4256********1231'
~~~

~~~http
POST /card/detect.action HTTP/1.1
Host: qiwi.com
Accept: application/json
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache

cardNumber=4256********1231
~~~

Параметр запроса:

Параметр|Тип|Описание
--------|----|----
cardNumber | String |Немаскированный номер карты

**Формат ответа**

Успешная проверка - Ответ с HTTP Status 200, параметр `code.value` = 0. 

Идентификатор платежной системы находится в параметре `message`.

~~~json
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

Ошибка в номере карты - Ответ с HTTP Status 200, параметр `code.value` = 2.

~~~json
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

## Банковский перевод {#banks}

Выполняет перевод на банковские карты/счета российских банков.

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/terms/{id}/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/terms/466/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  --header "User-Agent: ***"
  -d '{"id":"21131343", "sum": {"amount":1000,"currency":"643"}, "source": "account_643", 
       "paymentMethod": {"type":"Account","accountId":"643"}, 
       "fields": {"account_type": "1","account":"4256********1231","exp_date": "MMYY"}}'
~~~

~~~http
POST /sinap/terms/466/payments HTTP/1.1
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
  "source": "account_643", 
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

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

Параметр URL запроса:

* `{id}` - идентификатор банка. Возможные значения:
    * 466 - Тинькофф Банк
    * 464 - Альфа-Банк
    * 821 - Промсвязьбанк
    * 815 - Русский Стандарт

Тело запроса - JSON. Параметры запроса: 

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Объект, содержащий данные о сумме платежа: 
amount|Decimal|Сумма
currency|String|Валюта (только `643`, рубли)
source| String| Источник фондирования платежа. Допускается только следующее значение:<br>`account_643` - рублевый счет QIWI Кошелька отправителя
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
type|String |Тип платежа, только `Account`
accountId|String| Идентификатор счета, только `643`.
fields|Object| Объект с параметрами перевода:
account| String| Номер карты/счета получателя
exp_date| String|Срок действия карты, в формате ММГГ (например, `0218`). **Только для перевода на карту.**
account_type| String|Тип банковского идентификатора. Допустимые значения:<br>для Тинькофф Банк - карта "1", договор "3"<br>для Альфа-Банка - карта "1", счет "2"<br>для Промсвязьбанка - карта "7", счет "9"<br>для банка Русский Стандарт - карта "1", счет "2", договор "3"


~~~json
{
  "id": "21131343",
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

Успешный ответ содержит данные о принятом платеже:

Параметр | Описание
-----|--------
id | ID транзакции (максимум 20 цифр)
sum|Копия объекта `sum` из исходного запроса.
fields|Копия объекта `fields` из исходного запроса. **Номер карты в ответе маскирован.**
source| Копия параметра `source` из исходного запроса.
transaction|Объект с данными о транзакции в процессинге. Параметры:
id|ID транзакции в процессинге
state|Объект, содержит текущее состояние транзакции в процессинге. Содержит только параметр `code` со значением `Accepted` (платеж принят)


# Коды ошибок

В случае ошибки API возвращается HTTP-код ошибки.

Код | API | Описание
---|-----|---------
401 | Все | Ошибка авторизации
404 | История платежей | Не найдена транзакция или отсутствуют платежи с указанными признаками
404 | Балансы, Профиль пользователя | Не найден кошелек
423 | История платежей | Слишком много запросов, сервис временно недоступен
