---
title: API QIWI Кошелька

search: true

metatitle: API QIWI Кошелька

metadescription: API QIWI Кошелька позволяет автоматизировать выполнение платежей и получение отчетов о платежах, информации о счёте.

toc_footers:
 - <a href='/'>На главную</a>
 - <a href='mailto:api_help@qiwi.com'>Обратная связь</a>
---

 *[Токен]: Символьная строка для аутентификации пользователя в API.
 *[токен]: Символьная строка для аутентификации пользователя в API.
 *[API]: Application Programming Interface -  набор готовых методов, предоставляемых приложением (системой) для использования во внешних программных продуктах.
 *[JSON]: JavaScript Object Notation - текстовый формат обмена данными, основанный на JavaScript.



# Введение {#intro}

###### Последнее обновление: 2017-08-01 | [Предложить свои правки на GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/qiwicom_ru.html.md)

API QIWI Кошелька позволяет автоматизировать получение информации о вашем счёте в [сервисе Visa QIWI Кошелек](https://qiwi.com) и проводить операции с его помощью.

Методы API доступны после регистрации пользователя в [сервисе Visa QIWI Кошелек](https://qiwi.com). 

## Служебные данные {#auth_param}

<ul class="nestedList params">
    <li><h3>Авторизация</h3>
    </li>
</ul>

Параметр|Описание|Тип
 ---------|--------|---
 Bearer token | Токен для доступа к вашему QIWI кошельку по API. Действие токена заканчивается через 180 дней после [выпуска](#auth_data). Одновременно может действовать только один токен. | String


# Доступ к API {#auth_api}

Для успешного вызова методов API необходимы:

* Корректные заголовки `Accept` и `Content-Type`. API QIWI Кошелька поддерживает только один MIME-тип: `application/json`. Любое другое значение приведет к [ошибке формата данных](#errors).
* URL, составленный согласно требованиям к нужному запросу.
* OAuth-токен, выданный вам для доступа к вашему QIWI кошельку. Некоторые запросы его не требуют.

## Получение OAuth-токена {#auth_data}

API QIWI Кошелька использует открытый протокол [OAuth 2.0](http://tools.ietf.org/html/rfc6749). Согласно протоколу, пользователь авторизуется или регистрируется на сайте <https://qiwi.com> и запрашивает токен OAuth 2.0 Bearer с правом выполнения определённых действий. Выпуск токена подтверждается одноразовым кодом из СМС.

<aside class="warning">Если у вас уже есть действующий токен, то он автоматически заблокируется при выпуске нового токена</aside>

<aside class="warning">Перед выпуском токена проверьте, что на <a href="https://qiwi.com/settings/options/security.action">странице "Настройки безопасности"</a> установлен флаг "Приложения Visa QIWI Кошелька"</aside>

![VQW Security settings](/images/apiwallet_apps.jpg)

Для выпуска токена выполните следующие шаги:

1. Откройте в браузере страницу <https://qiwi.com/api>. Для этого потребуется авторизоваться или зарегистрироваться в сервисе Visa QIWI Кошелек.
  ![Token Issue](/images/apiwallet_get_token.jpg)
2. Нажмите **Выпустить новый токен**. Во всплывающем окне выберите разрешения на операции с токеном:
    * Запрос информации о профиле кошелька - разрешает выполнение запросов [профиля пользователя](#profile)
    * Запрос баланса кошелька - разрешает выполнение запросов [баланса](#balance)
    * Просмотр истории платежей - разрешает выполнение запросов [истории платежей](#payments_history)
    * Проведение платежей без SMS - разрешает выполнение [платежных запросов](#payments) без подтверждения по SMS (независимо от настроек <https://qiwi.com/settings/options/security.action>)
    ![Token Scopes](/images/apiwallet_token_scopes.jpg)
3. Нажмите **Продолжить** и подтвердите выпуск токена кодом из SMS-сообщения.
  ![Token Accept](/images/apiwallet_token_sms.jpg)
4. Скопируйте строку токена и сохраните в безопасном месте. Используйте токен для [запросов к API QIWI Кошелька](#auth_ex).

  ![Token](/images/apiwallet_token_final.jpg)

<aside class="notice">Токен действует в течение 180 дней с момента выпуска. Вы можете заблокировать токен, не дожидаясь окончания срока его действия, на <a href="https://qiwi.com/settings/mine.action">этой странице</a>.</aside>

## Пример вызова API {#auth_ex}

~~~shell
user@server:~$ curl "адрес сервера"
  --header "Authorization: Bearer jMyN22DQxMjM6NDUzRmRnZDQ0Mw11212e"
~~~

Полученный токен следует передавать в заголовке `Authorization` при каждом вызове API, указывая тип токена `Bearer` перед его значением. Пример получения такого заголовка:

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

В заголовке `Authorization` указывается [токен доступа](#auth_api).

Параметры URL запроса:

* `<parameters>` - дополнительные параметры запроса (необязательные):

Параметр|Тип |Описание
--------|----|-------
authInfoEnabled|Boolean | Логический признак выгрузки настроек авторизации пользователя. По умолчанию `true`
contractInfoEnabled|Boolean | Логический признак выгрузки данных о кошельке пользователя. По умолчанию `true`
userInfoEnabled|Boolean | Логический признак выгрузки прочих пользовательских данных. По умолчанию `true`

**Формат ответа**

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
authInfo.personId|Number|Номер кошелька пользователя
authInfo.registrationDate|String|Дата/время регистрации QIWI Кошелька пользователя (через сайт либо мобильноое приложение, либо другим способом)
authInfo.boundEmail|String|E-mail, привязанный к кошельку. Если отсутствует, то `null`
authInfo.ip|String|IP-адрес последней пользовательской сессии
authInfo.lastLoginDate|String|Дата/время последней сессии в QIWI Кошельке
authInfo.mobilePinInfo|Object|Данные о PIN-коде мобильного приложения QIWI Кошелька
mobilePinInfo.mobilePinUsed|Boolean|Логический признак использования PIN-кода (фактически означает, что мобильное приложение используется)
mobilePinInfo.lastMobilePinChange|String|Дата/время последнего изменения PIN-кода мобильного приложения QIWI Кошелька
mobilePinInfo.nextMobilePinChange|String|Дата/время следующего (планового) изменения PIN-кода мобильного приложения QIWI Кошелька
passInfo|Object|Данные о пароле к сайту qiwi.com
passInfo.passwordUsed|Boolean|Логический признак использования пароля (фактически означает, что пользователь заходит на сайт)
passInfo.lastPassChange|String|Дата/время последнего изменения пароля сайта qiwi.com
passInfo.nextPassChange|String|Дата/время следующего (планового) изменения пароля сайта qiwi.com
pinInfo|Object|Данные о PIN-коде к приложению QIWI Кошелька на QIWI терминалах
pinInfo.pinUsed|Boolean|Логический признак использования PIN-кода (фактически означает, что пользователь заходил в приложение)
contractInfo|Object| Информация о кошельке пользователя
contractInfo.blocked|Boolean|Логический признак блокировки кошелька
contractInfo.contractId|Number|Номер кошелька пользователя
contractInfo.creationDate|String|Дата/время создания QIWI Кошелька пользователя (через сайт либо мобильноое приложение, либо при первом пополнении, либо другим способом)
contractInfo.features|Array[Object]|Служебная информация
contractInfo.identificationInfo|Array[Object]|Данные об [идентификации](https://qiwi.com/settings/account/identification.action) пользователя.
identificationInfo[].bankAlias|String|Акроним системы, в которой пользователь получил идентификацию:<br> `QIWI` - QIWI Кошелек.
identificationInfo[].identificationLevel|Текущий уровень идентификации кошелька. Возможные значения:<br>`ANONYMOUS` - без идентификации;<br> `SIMPLE`, `VERIFIED` - упрощенная идентификация;<br> `FULL` - полная идентификация.
userInfo|Object|Прочие пользовательские данные
userInfo.defaultPayCurrency|Number|Код валюты баланса кошелька по умолчанию (number-3 ISO-4217)
userInfo.defaultPaySource|Number|Служебная информация
userInfo.email|String|E-mail пользователя
userInfo.firstTxnId|Number|Номер первой транзакции пользователя после регистрации
userInfo.language|String|Служебная информация
userInfo.operator|String|Название мобильного оператора номера пользователя
userInfo.phoneHash|String|Служебная информация
userInfo.promoEnabled|String|Служебная информация


# Идентификация пользователя {#ident}

Данный запрос позволяет отправить данные для упрощенной идентификации своего QIWI кошелька. [Подробнее об идентификации](https://qiwi.com/settings/account/identification.action)

<aside class="warning">Допускается <b>не</b> идентифицировать не более 5 кошельков на одного владельца. См. п. 3.1.1 ч. III <a href="https://static.qiwi.com/ru/doc/oferta_lk.pdf">Оферты сервиса "Visa QIWI Wallet"</a></aside>

Для получения статуса упрощенно идентифицированного кошелька необходимо предоставить следующие данные о пользователе-владельце кошелька:

* ФИО
* Серия / Номер паспорта
* Дата рождения
* ИНН, СНИЛС или номер полиса ОМС - необязательно.

Для идентификации кошелька вы обязательно должны отправить ФИО, серию/номер паспорта и дату рождения. Если данные прошли проверку, то в ответе будет отображен ваш ИНН и упрощенная идентификация кошелька будет установлена. В случае если данные не прошли проверку, кошелек остается в статусе "Без идентификации".

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/identification/v1/persons/79111234567/identification"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9"
  -d '{
  "birthDate": "1998-02-11",
  "firstName": "Иван",
  "inn": "",
  "lastName": "Иванов",
  "middleName": "Иванович",
  "oms": "",
  "passport": "4400111222",
  "snils": ""
}'
~~~

~~~http
POST /identification/v1/persons/79111234567/identification HTTP/1.1
Accept: application/json
Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9
Content-type: application/json
Host: edge.qiwi.com

{
  "birthDate": "1998-02-11",
  "firstName": "Иван",
  "inn": "",
  "lastName": "Иванов",
  "middleName": "Иванович",
  "oms": "",
  "passport": "4400111222",
  "snils": ""
}
~~~

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/identification/v1/persons/<wallet>/identification`

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указывается [токен доступа](#auth_api).

Параметр URL запроса:

* `<wallet>` - номер кошелька, для которого получен токен доступа (с международным префиксом, но без `+`). **Обязательный параметр**

Параметры JSON-тела запроса:

Параметр|Тип|Описание
--------|----|----
birthDate|String| Дата рождения пользователя (в формате "ГГГГ-ММ-ДД")
firstName|String|Имя пользователя
middleName|String|Отчество пользователя
lastName|String|Фамилия пользователя
passport|String|Серия и номер паспорта пользователя (только цифры)
inn|String|ИНН пользователя
snils|String|Номер СНИЛС пользователя
oms|String|Номер полиса ОМС пользователя

**Формат ответа**

~~~json
{
  "birthDate": "1996-03-18",
  "firstName": "Иван",
  "id": 79111234567,
  "inn": "7710000001",
  "lastName": "Иванов",
  "middleName": "Иванович",
  "oms": "",
  "passport": "1122333000",
  "snils": "",
  "type": "VERIFIED"
}
~~~

Успешный ответ в формате JSON содержит подтверждение идентификации кошелька:

Параметр|Тип|Описание
--------|----|----
id|  Number  |Номер кошелька пользователя
type | String | Текущий уровень идентификации кошелька:<br>`SIMPLE` - без идентификации.<br>`VERIFIED` - упрощенная идентификация (данные для идентификации успешно прошли проверку).<br>`FULL` – если кошелек уже ранее получал полную идентификацию по данным ФИО, номеру паспорта и дате рождения.
birthDate |String | Дата рождения пользователя
firstName| String | Имя пользователя
middleName | String | Отчество пользователя
lastName|  String | Фамилия пользователя
passport | String | Серия и номер паспорта пользователя
inn| String|  ИНН пользователя. Если в запросе параметр не заполнен, но присутствует в ответе, то идентификация кошелька выполнена.
snils |String | Номер СНИЛС пользователя
oms| String | Номер полиса ОМС пользователя

# История платежей {#payments_history}

Запрос выгружает историю платежей и пополнений вашего кошелька.

Тип запроса - GET.

~~~shell
Пример 1. Последние 10 платежей

user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments?rows=10"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"

Пример 2. Платежи за 10.05.2017

user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments?rows=50&startDate=2017-05-10T00%3A00%3A00%2B03%3A00&endDate=2017-05-10T23%3A59%3A59%2B03%3A00"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"

Пример 3. Продолжение списка платежей (в предыдущем запросе истории возвращены параметры nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23+03:00)

user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12%3A35%3A23%2B03%3A00"
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
GET /payment-history/v1/persons/79112223344/payments?rows=50&startDate=2017-05-10T00%3A00%3A00%2B03%3A00&endDate=2017-05-10T23%3A59%3A59%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

~~~text
Пример 3. Продолжение списка платежей 

(в предыдущем запросе истории возвращены параметры nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23+03:00)
~~~

~~~http
GET /payment-history/v1/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12%3A35%3A23%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

URL запроса:

`https://edge.qiwi.com/payment-history/v1/persons/<wallet>/payments?<parameters>`

Параметры URL запроса:

* `<wallet>` - номер кошелька, для которого получен токен доступа (с международным префиксом, но без `+`). **Обязательный параметр**
* `<parameters>` - дополнительные параметры запроса (см. ниже)

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указывается [токен доступа](#auth_api).

Параметры запроса: 

Параметр|Тип|Описание
--------|----|----
rows | Integer |Число платежей в ответе, для разбивки отчета на части. Целое число от 1 до 50. **Обязательный параметр**
operation|String| Тип операций в отчете, для отбора. Допустимые значения:<br>`ALL` - все операции, <br>`IN` - только пополнения, <br>`OUT` - только платежи, <br>`QIWI_CARD` - только платежи по картам QIWI (QVC, QVP). <br>По умолчанию `ALL`
sources|Array[String]|Источники платежа, для отбора. Каждый источник задается как отдельный параметр и нумеруется элементом массива, начиная с нуля (`sources[0]`, `sources[1]` и т.д.). Допустимые значения: <br>`QW_RUB` - рублевый счет кошелька, <br>`QW_USD` - счет кошелька в долларах, <br>`QW_EUR` - счет кошелька в евро, <br>`CARD` - привязанные и непривязанные к кошельку банковские карты, <br>`MK` - счет мобильного оператора. Если не указаны, учитываются все источники
startDate | DateTime URL-encoded| Начальная дата поиска платежей, время московское (формат ГГГГ-ММ-ДДTчч:мм:сс+03:00). По умолчанию, равна вчерашней дате. **Используется только вместе с `endDate`**
endDate | DateTime URL-encoded | Конечная дата поиска платежей, время московское (формат ГГГГ-ММ-ДДTчч:мм:сс+03:00). По умолчанию, равен текущей дате. **Используется только вместе с `startDate`**
nextTxnDate | DateTime URL-encoded| Дата транзакции (формат ГГГГ-ММ-ДДTчч:мм:сс+03:00), для отсчета от предыдущего списка (см. параметр `nextTxnDate` в ответе). **Используется только вместе с `nextTxnId`**
nextTxnId | Long | Номер предшествующей транзакции, для отсчета от предыдущего списка (см. параметр `nextTxnId` в ответе). **Используется только вместе с `nextTxnDate`**


<aside class="notice">Максимальный допустимый интервал между startDate и endDate - 90 календарных дней.</aside>

<aside class="warning">Максимальная интенсивность запросов истории платежей - не более 100 запросов в минуту с одного IP-адреса. При превышении доступ к API блокируется на 5 минут.</aside>

**Формат ответа**

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
  }],
  "nextTxnId":9001,
  "nextTxnDate":"2017-01-31T15:24:10+03:00"
}
~~~

Успешный ответ содержит список платежей из истории кошелька, соответствующих заданному фильтру:

<a name="history_data"></a>

Параметр|Тип|Описание
--------|----|----
data|Array[Object]|Массив платежей. <br>Число платежей равно параметру `rows` из запроса
data[].txnId | Integer |ID транзакции в процессинге
data[].personId|Integer|Номер кошелька
data[].date|DateTime|Дата/время платежа, время московское (в формате ГГГГ-ММ-ДДTчч:мм:сс+03:00)
data[].errorCode|Number(Integer)|Код ошибки платежа
data[].error| String| Описание ошибки
data[].type | String| Тип платежа. Возможные значения:<br>`IN` - пополнение, <br>`OUT` - платеж, <br>`QIWI_CARD` - платеж с карт QIWI (QVC, QVP).
data[].status|String|Статус платежа. Возможные значения:<br>`WAITING` - платеж проводится, <br>`SUCCESS` - успешный платеж, <br>`ERROR` - ошибка платежа.
data[].statusText|String |Текстовое описание статуса платежа
data[].trmTxnId|String|Клиентский ID транзакции
data[].account| String|Номер счета получателя
data[].sum|Object| Данные о сумме платежа. Параметры:
sum.amount|Number(Decimal)|сумма,
sum.currency|String|валюта
data[].commission|Object| Данные о комиссии платежа. Параметры:
commission.amount|Number(Decimal)|сумма,
commission.currency|String|валюта
data[].total|Object| Данные об общей сумме платежа. Параметры:
total.amount|Number(Decimal)|сумма,
total.currency|String|валюта
data[].provider|Object| Данные о провайдере. Параметры:
provider.id|Integer|ID провайдера в процессинге,
provider.shortName|String|краткое наименование провайдера,
provider.longName|String|развернутое наименование провайдера,
provider.logoUrl|String|ссылка на логотип провайдера,
provider.description|String|описание провайдера (HTML),
provider.keys|String|список ключевых слов,
provider.siteUrl|String|сайт провайдера
data[].comment|String|Комментарий к платежу
data[].currencyRate|Number(Decimal)|Курс конвертации (если применяется в транзакции)
data[].extras|Object|Служебная информация
data[].chequeReady| Boolean|Специальное поле
data[].bankDocumentAvailable|Boolean|Специальное поле
data[].bankDocumentReady|Boolean|Специальное поле
data[].repeatPaymentEnabled|Boolean|Специальное поле
data[].favoritePaymentEnabled|Boolean|Специальное поле
data[].regularPaymentEnabled|Boolean|Специальное поле
nextTxnId|Number(Integer)|ID следующей транзакции в полном списке
nextTxnDate|DateTime|Дата/время следующей транзакции в полном списке, время московское (в формате ГГГГ-ММ-ДДTчч:мм:сс+03:00)

## Статистика платежей {#stat}

Для получения статистики по суммам платежей за заданный период используется данный подзапрос запроса истории платежей.

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments/total?startDate=2017-03-01T00%3A00%3A00%2B03%3A00&endDate=2017-03-31T11%3A44%3A15%2B03%3A00"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /payment-history/v1/persons/79112223344/payments/total?startDate=2017-03-01T00%3A00%3A00%2B03%3A00&endDate=2017-03-31T11%3A44%3A15%2B03%3A00 HTTP/1.1
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

В заголовке `Authorization` указывается [токен доступа](#auth_api).

Параметры URL запроса:

* `<wallet>` - номер кошелька, для которого получен токен доступа (с международным префиксом, но без знака "+"). **Обязательный параметр**
* `<parameters>` - дополнительные параметры запроса:

Параметр|Тип |Описание
--------|----|-------
startDate|DateTime URL-encoded | Начальная дата периода статистики, время московское (в формате ГГГГ-ММ-ДДTчч:мм:сс+03:00). **Обязательный параметр**
endDate|DateTime URL-encoded| Конечная дата периода статистики, время московское (в формате ГГГГ-ММ-ДДTчч:мм:сс+03:00). **Обязательный параметр**
operation|String| Тип операций, учитываемых при подсчете статистики. Допустимые значения:<br>`ALL` - все операции, <br>`IN` - только пополнения, <br>`OUT` - только платежи, <br>`QIWI_CARD` - только платежи по картам QIWI (QVC, QVP). <br>По умолчанию `ALL`.|Нет
sources|Array[String]|Источники платежа, учитываемые при подсчете статистики. Каждый источник задается как отдельный параметр и нумеруется элементом массива, начиная с нуля (`sources[0]`, `sources[1]` и т.д.). Допустимые значения: <br>`QW_RUB` - рублевый счет кошелька, <br>`QW_USD` - счет кошелька в долларах, <br>`QW_EUR` - счет кошелька в евро, <br>`CARD` - привязанные и непривязанные к кошельку банковские карты, <br>`MK` - счет мобильного оператора. Если не указан, учитываются все источники платежа.|Нет


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
incomingTotal[].amount | Number(Decimal) |Сумма пополнений за период
incomingTotal[].currency|String|Валюта пополнений
outgoingTotal|Array[Object]|Данные об исходящих платежах, отдельно по каждой валюте
outgoingTotal[].amount | Number(Decimal) |Сумма платежей за период
outgoingTotal[].currency|String|Валюта платежей

## Информация о транзакции {#txn_info}

Для получения подробной информации по определенной транзакции из вашей истории платежей используется данный подзапрос запроса истории платежей.

Тип запроса - GET.

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/transactions/9112223344?type=IN"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /payment-history/v1/transactions/9112223344?type=IN HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

URL запроса:

`https://edge.qiwi.com/payment-history/v1/transactions/<transactionId>?<type>`

Обязательные параметры URL запроса:

* `<transactionId>` - номер транзакции из [истории платежей](#history_data) (параметр `data[].txnId` в ответе).
* `<type>` - тип транзакции из [истории платежей](#history_data) (параметр `data[].type` в ответе).

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указывается [токен доступа](#auth_api).

**Формат ответа**

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
        "shortName": "Visa QIWI Wallet",
        "longName": "Visa QIWI Wallet",
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

Успешный ответ содержит данные о транзакции:

Параметр|Тип|Описание
--------|----|----
txnId | Integer |Копия параметра `<transactionId>` из запроса
personId|Integer|Номер кошелька
date|DateTime|Дата/время платежа, время московское (в формате ГГГГ-ММ-ДДTчч:мм:сс+03:00)
errorCode|Number(Integer)|Код ошибки платежа
error| String| Описание ошибки
type | String| Копия параметра `<type>` из запроса
status|String|Статус платежа. Возможные значения:<br>`WAITING` - платеж проводится, <br>`SUCCESS` - успешный платеж, <br>`ERROR` - ошибка платежа.
statusText|String |Текстовое описание статуса платежа
trmTxnId|String|Клиентский ID транзакции
account| String|Номер счета получателя
sum|Object| Данные о сумме платежа. Параметры:
sum.amount|Number(Decimal)|сумма,
sum.currency|String|валюта
commission|Object| Данные о комиссии платежа. Параметры:
commission.amount|Number(Decimal)|сумма,
commission.currency|String|валюта
total|Object| Данные об общей сумме платежа. Параметры:
total.amount|Number(Decimal)|сумма, 
total.currency|String|валюта 
provider|Object| Данные о провайдере. Параметры:
provider.id|Integer|ID провайдера в процессинге,
provider.shortName|String|краткое наименование провайдера,
provider.longName|String|развернутое наименование провайдера,
provider.logoUrl|String|ссылка на логотип провайдера,
provider.description|String|описание провайдера (HTML),
provider.keys|String|список ключевых слов,
provider.siteUrl|String|сайт провайдера
source|Object|Служебная информация
comment|String|Комментарий к платежу
currencyRate|Number(Decimal)|Курс конвертации (если применяется в транзакции)
extras|Object|Служебная информация
chequeReady| Boolean|Специальное поле
bankDocumentAvailable|Boolean|Специальное поле
repeatPaymentEnabled|Boolean|Специальное поле
favoritePaymentEnabled|Boolean|Специальное поле
regularPaymentEnabled|Boolean|Специальное поле

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

В заголовке `Authorization` указывается [токен доступа](#auth_api).

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
        }
    ]
}
~~~

Успешный ответ содержит список счетов для фондирования платежей и их текущие балансы:

Параметр|Тип|Описание
--------|----|----
accounts|Array[Object]|Массив балансов
accounts[].alias | String |Псевдоним пользовательского баланса
accounts[].fsAlias | String |Псевдоним банковского баланса
accounts[].title|String|Название соответствующего счета кошелька
accounts[].hasBalance|Boolean|Логический признак реального баланса в системе QIWI Кошелек (не привязанная карта, не счет мобильного телефона и т.д.)
accounts[].currency | Number| Код валюты баланса (number-3 ISO-4217). Возвращаются балансы в следующих валютах: 643 - российский рубль, 840 - американский доллар, 978 - евро
accounts[].type|Object|Сведения о счете
type.id, type.title| String| Описание счета
accounts[].balance|Object |Сведения о балансе данного счета
balance.amount|Number|Текущий баланс данного счета
balance.currency | Number| Код валюты баланса (number-3 ISO-4217)

# Платежи {#payments}

## Комиссионные тарифы

### Стандартные тарифы {#standard_rates}

Для провайдеров, оплата которых доступна через API, можно предварительно проверить комиссионные условия (стандартные тарифы).

Тип запроса - GET. Запрос не требует токена доступа.

URL запроса:

`https://edge.qiwi.com/sinap/providers/{id}/form`

~~~shell
user@server:~$ curl "https://edge.qiwi.com/sinap/providers/99/form"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
~~~

~~~http
GET /sinap/providers/99/form HTTP/1.1
Content-Type: application/json
Accept: application/json
Host: edge.qiwi.com
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
    * [Идентификаторы других провайдеров](#charity)
    * 1717 - [платеж по банковским реквизитам](#freepay)

**Формат ответа**

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
commission.ranges | Array[Object] | Массив объектов с граничными условиями комиссий. Каждый объект содержит параметры:
ranges[].bound |  Number | сумма платежа, начиная с которой применяется условие
ranges[].rate | Number |  комиссия (абс.множитель)
ranges[].min | Number | минимальная сумма комиссии
ranges[].max | Number | максимальная сумма комиссии
ranges[].fixed | Number | фиксированная сумма комиссии

### Специальные тарифы {#online-comm}

Запрос предназначен для получения специальных комиссий на платеж (с учетом всех тарифов) по заданному набору платежных реквизитов.

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/providers/{id}/onlineCommission`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/providers/99/onlineCommission"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  -d '{
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

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указывается [токен доступа](#auth_api).

Параметр URL запроса:

* `{id}` - идентификатор провайдера. Возможные значения:
    * 99 - Перевод на Visa QIWI Wallet
    * 1963 - Перевод на карту Visa (карты российских банков)
    * 21013 - Перевод на карту MasterCard (карты российских банков)
    * Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:
        * 1960 – Перевод на карту Visa
        * 21012 – Перевод на карту MasterCard
    * 466 - Тинькофф Банк
    * 464 - Альфа-Банк
    * 821 - Промсвязьбанк
    * 815 - Русский Стандарт
    * [Идентификаторы операторов мобильной связи](#mnp)
    * [Идентификаторы других провайдеров](#charity)
    * 1717 - [платеж по банковским реквизитам](#freepay)

Тело запроса - JSON. Параметры запроса:

Параметр|Тип|Описание
--------|----|----
account| String|Номер телефона (с международным префиксом) или номер карты/счета получателя, в зависимости от провайдера
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
paymentMethod.type|String |Тип платежа, только `Account`
paymentMethod.accountId|String| Идентификатор счета, только `643`.
purchaseTotals | Object |Объект с платежными реквизитами
purchaseTotals.total|Object| Объект, содержащий данные о сумме платежа: 
total.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
total.currency|String|Валюта (только `643`, рубли)

**Формат ответа**

В параметре ответа `qwCommission.amount` возвращается сумма комиссии.

~~~json
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

## Автозаполнение платежных форм {#payform}

Чтобы открыть заполненную форму на сайте qiwi.com для совершения платежа, откройте в браузере ссылку вида:

`https://qiwi.com/payment/form/<ID>?<parameters>`

[Пример ссылки](https://qiwi.com/payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&extra%5B%27comment%27%5D=test123&currency=643)

Обязательный параметр в URL запроса:

* `ID` -  идентификатор провайдера. Возможные значения:
    * 99 - Перевод на Visa QIWI Wallet
    * 1963 - Перевод на карту Visa (карты российских банков)
    * 21013 - Перевод на карту MasterCard (карты российских банков)
    * Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:
        * 1960 – Перевод на карту Visa
        * 21012 – Перевод на карту MasterCard
    * [Идентификаторы операторов мобильной связи](#mnp)
    * [Идентификаторы других провайдеров](#charity)

Параметры в строке запроса `<parameters>`:

* `amountInteger` - целая часть суммы платежа (рубли).
* `amountFraction` - дробная часть суммы платежа (копейки).
* `currency` - константа, `643`.
* `extra['comment']` - комментарий (только для `ID`=99). Имя параметра должно быть URL-закодировано.
* `extra['account']` - номер телефона/счета/карты пользователя. Формат совпадает с форматом параметра `fields.account` в соответствующем платежном запросе. Имя параметра должно быть URL-закодировано.

<aside class="notice">Если параметр не указан, соответствующее поле на форме будет пустым.</aside>

## Перевод на QIWI Кошелек {#p2p}

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/api/v2/terms/99/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/99/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  -d '{
        "id":"11111111111111",
        "sum": {
          "amount":100,
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

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указывается [токен доступа](#auth_api).
 
Тело запроса - JSON. Параметры запроса:

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Данные о сумме платежа: 
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`
paymentMethod.accountId|String| Константа, `643`.
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String|Номер телефона получателя (с международным префиксом)
comment|String|Комментарий к платежу. Необязательный параметр.


### Формат ответа

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

Успешный ответ содержит данные о принятом платеже:

Параметр | Тип | Описание
-----|----|-----
id | String | Копия параметра `id` из исходного запроса
terms | String | Константа, `99`
fields|Object|Копия объекта `fields` из исходного запроса.
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
comment|String|Копия параметра `comment` из исходного запроса (если присутствует в запросе)
transaction|Object|Объект с данными о транзакции в процессинге. Параметры:
transaction.id|String|ID транзакции в процессинге
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

## Оплата сотовой связи {#cell}

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/api/v2/terms/{id}/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  -d '{
        "id":"11111111111111",
        "sum": {
          "amount":100,
          "currency":"643"
        },
        "paymentMethod": {
          "type":"Account",
          "accountId":"643"
        },
        "fields": {
          "account":"9161112233"
        }
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

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указывается [токен доступа](#auth_api).

Параметр URL запроса:

* `{id}` - идентификатор провайдера. Определяется с помощью [проверки мобильного оператора](#mnp).
 
Тело запроса - JSON. Все перечисленные параметры обязательны: 

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Данные о сумме платежа: 
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`
paymentMethod.accountId|String| Константа, `643`.
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String|Номер мобильного телефона для пополнения (без префикса `8`)

~~~json
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

Успешный ответ содержит данные о принятом платеже:

Параметр | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из исходного запроса
terms | String | Параметр `{id}` из URL запроса
fields|Object|Копия объекта `fields` из исходного запроса.
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге. Параметры:
transaction.id|String|ID транзакции в процессинге
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

### Определение оператора {#mnp}

Предварительная проверка мобильного номера выполняется POST-запросом:

`https://qiwi.com/mobile/detect.action`

В ответе возвращается идентификатор оператора для [запроса пополнения телефона](#cell). Запрос не требует токена доступа.

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/x-www-form-urlencoded`

~~~shell
user@server:~$ curl -X POST "https://qiwi.com/mobile/detect.action"
  --header "Accept: application/json" 
  --header "Content-Type: application/x-www-form-urlencoded"
  -d 'phone=%2B79651238341'
~~~

~~~http
POST /mobile/detect.action HTTP/1.1
Host: qiwi.com
Accept: application/json
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache

phone=%2B79651238341
~~~

Параметр запроса:

Параметр|Тип|Описание
--------|----|----
phone | String URL-encoded |Мобильный номер в международном формате

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

`https://edge.qiwi.com/sinap/api/v2/terms/{id}/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1963/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  -d '{
        "id":"21131343",
        "sum":{
          "amount":1000,
          "currency":"643"
        },
        "paymentMethod":{
          "type":"Account",
          "accountId":"643"
        },
        "fields": {
          "account":"4256********1231"
        }
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

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указывается [токен доступа](#auth_api).

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
sum|Object| Данные о сумме платежа: 
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`
paymentMethod.accountId|String| Константа, `643`.
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String|Номер банковской карты получателя

~~~json
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

Успешный ответ содержит данные о принятом платеже:

Параметр | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из исходного запроса
terms | String | Параметр `{id}` из URL запроса
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String|Маскированный номер банковской карты получателя
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге. Параметры:
transaction.id|String|ID транзакции в процессинге
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

### Проверка карты {#card_check}

Определение платежной системы карты выполняется POST-запросом:

`https://qiwi.com/card/detect.action`

В ответе возвращается идентификатор платежной системы для [запроса перевода на карту](#cards). Запрос не требует токена доступа.

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/x-www-form-urlencoded`

~~~shell
user@server:~$ curl -X POST "https://qiwi.com/card/detect.action"
  --header "Accept: application/json" 
  --header "Content-Type: application/x-www-form-urlencoded"
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

`https://edge.qiwi.com/sinap/api/v2/terms/{id}/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/466/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  -d '{
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
          "account_type": "1",
          "account":"4256********1231",
          "exp_date": "MMYY"
        }
      }'
~~~

~~~http
POST /sinap/api/v2/terms/466/payments HTTP/1.1
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

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указывается [токен доступа](#auth_api).

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
sum|Object| Данные о сумме платежа: 
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`
paymentMethod.accountId|String| Константа, `643`.
fields|Object| Реквизиты платежа. Содержит параметры:
fields.account| String| Номер карты/счета получателя
fields.exp_date| String|Срок действия карты, в формате ММГГ (например, `0218`). **Параметр указывается только в случае перевода на карту.**
fields.account_type| String|Тип банковского идентификатора. Допустимые значения:<br>для Тинькофф Банк - карта "1", договор "3"<br>для Альфа-Банка - карта "1", счет "2"<br>для Промсвязьбанка - карта "7", счет "9"<br>для банка Русский Стандарт - карта "1", счет "2", договор "3"


~~~json
{
  "id": "21131343",
  "terms": "466",
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

Параметр | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из исходного запроса
terms | String | Параметр `{id}` из URL запроса
fields|Object|Копия объекта `fields` из исходного запроса. **В случае перевода на банковскую карту, в параметре**
`fields.account` **находится маскированный номер банковской карты получателя**.
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге. Параметры:
transaction.id|String|ID транзакции в процессинге
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

## Оплата других услуг {#charity}

Выполняет оплату услуги по идентификатору пользователя. Данный запрос применяется для провайдеров, использующих в реквизитах единственный пользовательский идентификатор, без проверки номера аккаунта.

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/api/v2/terms/{id}/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/674/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  -d '{
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
          "account":"111000000"
        }
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

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указывается [токен доступа](#auth_api).

Параметр URL запроса:

* `{id}` - идентификатор провайдера. Возможные значения:
    * 674 - OnLime
    * Идентификатор другого интернет-провайдера
    * 1239 - Подари жизнь
    * Идентификатор другого благотворительного фонда

<aside class="notice">Поиск идентификатора выполняется на сайте qiwi.com в поисковой строке. Идентификатор находится в URL ссылки на платежную форму провайдера вида https://qiwi.com/payment/form.action?provider=ID или https://qiwi.com/payment/form/ID</aside>

![Поиск провайдера](/images/provider_id.jpg)

Тело запроса - JSON. Параметры запроса:

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Данные о сумме платежа: 
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`
paymentMethod.accountId|String| Константа, `643`.
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String| Пользовательский идентификатор


~~~json
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

Успешный ответ содержит данные о принятом платеже:

Параметр | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из исходного запроса
terms | String | Параметр `{id}` из URL запроса
fields|Object|Копия объекта `fields` из исходного запроса.
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге. Параметры:
transaction.id|String|ID транзакции в процессинге
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

## Платеж по свободным реквизитам {#freepay}

Данный запрос позволяет оплачивать услуги коммерческих организаций по их банковским реквизитам.

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/api/v2/terms/{id}/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1717/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  --header "User-Agent: ***"
  -d '{
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

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: Bearer ***`

В заголовке `Authorization` указывается [токен доступа](#auth_api).

Параметр URL запроса:

* `{id}` - идентификатор услуги, константа `1717`.

Тело запроса - JSON. Параметры запроса:

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Данные о сумме платежа:
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
paymentMethod.type|String |Тип платежа, только `Account`
paymentMethod.accountId|String| Идентификатор счета, только `643`.
fields|Object| Реквизиты платежа. Содержит параметры:
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


~~~json
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

Успешный ответ содержит данные о принятом платеже:

Параметр | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из исходного запроса
terms | String | Константа, `1717`
fields|Object|Копия объекта `fields` из исходного запроса
sum|Object|Копия объекта `sum` из исходного запроса
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге. Параметры:
transaction.id|String|ID транзакции в процессинге
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments).

# Коды ошибок {#errors}

В случае ошибки API возвращается HTTP-код ошибки.

Код | API | Описание
---|-----|---------
400 | Все | Ошибка синтаксиса запроса (неправильный формат данных)
401 | Все | Неверный токен или истек срок действия токена.
404 | История платежей | Не найдена транзакция или отсутствуют платежи с указанными признаками
404 | Балансы, Профиль пользователя, Идентификация пользователя | Не найден кошелек
423 | История платежей | Слишком много запросов, сервис временно недоступен
