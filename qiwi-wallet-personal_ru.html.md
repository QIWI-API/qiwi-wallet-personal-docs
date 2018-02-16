---
title: API QIWI Кошелька

search: true

metatitle: API QIWI Кошелька

metadescription: API QIWI Кошелька позволяет автоматизировать выполнение платежей и получение отчетов о платежах, информации о счёте, идентификации.

toc_footers:
 - <a href='/ru/qiwi-wallet-api-release-notes/index.html'>Список изменений</a>
 - <a href='/'>На главную</a>
 - <a href='mailto:api_help@qiwi.com'>Обратная связь</a>
---

 *[Токен]: Символьная строка для аутентификации пользователя в API.
 *[токен]: Символьная строка для аутентификации пользователя в API.
 *[API]: Application Programming Interface -  набор готовых методов, предоставляемых приложением (системой) для использования во внешних программных продуктах.
 *[JSON]: JavaScript Object Notation - текстовый формат обмена данными, основанный на JavaScript.



# Введение {#intro}

###### Последнее обновление: 2018-01-15 | [Предложить свои правки на GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/qiwi-wallet-personal_ru.html.md)

API QIWI Кошелька позволяет автоматизировать получение информации о вашем счёте в [сервисе QIWI Кошелек](https://qiwi.com) и проводить операции с его помощью.

Методы API доступны после регистрации пользователя в [сервисе QIWI Кошелек](https://qiwi.com).

## Авторизация запросов {#auth_param}

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

<aside class="warning">Перед выпуском токена проверьте, что на <a href="https://qiwi.com/settings/options/security.action">странице "Настройки безопасности"</a> установлен флаг "Приложения QIWI Кошелька"</aside>

![VQW Security settings](/images/apiwallet_apps.jpg)

Для выпуска токена выполните следующие шаги:

* Откройте в браузере страницу <https://qiwi.com/api>. Для этого потребуется авторизоваться или зарегистрироваться в сервисе QIWI Кошелек.

![Token Issue](/images/apiwallet_get_token.jpg)

* Нажмите **Выпустить новый токен**. Во всплывающем окне выберите разрешения на операции с токеном:
    * Запрос информации о профиле кошелька - разрешает выполнение запросов [профиля пользователя](#profile)
    * Запрос баланса кошелька - разрешает выполнение запросов [баланса](#balance)
    * Просмотр истории платежей - разрешает выполнение запросов [истории платежей](#payments_history)
    * Проведение платежей без SMS - разрешает выполнение [платежных запросов](#payments) без подтверждения по SMS (независимо от настроек <https://qiwi.com/settings/options/security.action>)

![Token Scopes](/images/apiwallet_token_scopes.jpg)

* Нажмите **Продолжить** и подтвердите выпуск токена кодом из SMS-сообщения.

![Token Accept](/images/apiwallet_token_sms.jpg)

* Скопируйте строку токена и сохраните в безопасном месте. Используйте токен для [запросов к API QIWI Кошелька](#auth_ex).

![Token](/images/apiwallet_token_final.jpg)

<aside class="notice">Токен действует в течение 180 дней с момента выпуска. Вы можете заблокировать токен, не дожидаясь окончания срока его действия, на <a href="https://qiwi.com/settings/mine.action">этой странице</a>.</aside>

## Пример вызова API {#auth_ex}

~~~shell
user@server:~$ curl "адрес сервера"
  --header "Authorization: Bearer jMyN22DQxMjM6NDUzRmRnZDQ0Mw11212e"
~~~

Полученный токен следует передавать в заголовке `Authorization` при каждом вызове API, указывая тип токена `Bearer` перед его значением. Пример получения такого заголовка:

* В результате авторизации на сайте QIWI Кошелек и выпуска токена [получен](#auth_data) токен, представляющий собой строку:

`U1QtOTkwMTAyLWNud3FpdWhmbzg3M`

* Токен добавляется в заголовок `Authorization: Bearer `

* Итоговый заголовок, добавляемый в каждый запрос к API QIWI Кошелька:

`Authorization: Bearer U1QtOTkwMTAyLWNud3FpdWhmbzg3M`

# Профиль пользователя {#profile}

~~~shell
user@server:~$ curl "https://edge.qiwi.com/person-profile/v1/profile/current?authInfoEnabled=false"
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

<h3 class="request method">Запрос → GET</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/person-profile/v1/profile/current?<a>parameter=value</a></span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Данные параметры передаются в строке запроса и не являются обязательными:</span>
    </li>
</ul>

Параметр|Тип |Описание
--------|----|-------
authInfoEnabled|Boolean | Логический признак выгрузки настроек авторизации пользователя.<br>По умолчанию `true`
contractInfoEnabled|Boolean | Логический признак выгрузки данных о кошельке пользователя.<br>По умолчанию `true`
userInfoEnabled|Boolean | Логический признак выгрузки прочих пользовательских данных.<br>По умолчанию `true`

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

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

Успешный JSON-ответ содержит данные о профиле пользователя:

Параметр|Тип|Описание
--------|----|----
authInfo|Object|Текущие настройки авторизации пользователя. Объект может отсутствовать, в зависимости от признака `authInfoEnabled` в запросе.
authInfo.personId|Number|Номер кошелька пользователя
authInfo.registrationDate|String|Дата/время регистрации QIWI Кошелька пользователя (через сайт/мобильное приложение, либо другим способом)
authInfo.boundEmail|String|E-mail, привязанный к кошельку. Если отсутствует, то `null`
authInfo.ip|String|IP-адрес последней пользовательской сессии
authInfo.lastLoginDate|String|Дата/время последней сессии в QIWI Кошельке
authInfo.mobilePinInfo|Object|Данные о PIN-коде мобильного приложения QIWI Кошелька
mobilePinInfo.mobilePinUsed|Boolean|Логический признак использования PIN-кода (фактически означает, что мобильное приложение используется)
mobilePinInfo.lastMobilePinChange|String|Дата/время последнего изменения PIN-кода мобильного приложения QIWI Кошелька
mobilePinInfo.nextMobilePinChange|String|Дата/время следующего (планового) изменения PIN-кода мобильного приложения QIWI Кошелька
authInfo.passInfo|Object|Данные о пароле к сайту qiwi.com
passInfo.passwordUsed|Boolean|Логический признак использования пароля (фактически означает, что пользователь заходит на сайт)
passInfo.lastPassChange|String|Дата/время последнего изменения пароля сайта qiwi.com
passInfo.nextPassChange|String|Дата/время следующего (планового) изменения пароля сайта qiwi.com
authInfo.pinInfo|Object|Данные о PIN-коде к приложению QIWI Кошелька на QIWI терминалах
pinInfo.pinUsed|Boolean|Логический признак использования PIN-кода (фактически означает, что пользователь заходил в приложение)
contractInfo|Object| Информация о кошельке пользователя. Объект может отсутствовать, в зависимости от признака `contractInfoEnabled` в запросе.
contractInfo.blocked|Boolean|Логический признак блокировки кошелька
contractInfo.contractId|Number|Номер кошелька пользователя
contractInfo.creationDate|String|Дата/время создания QIWI Кошелька пользователя (через сайт/мобильное приложение, либо при первом пополнении, либо другим способом)
contractInfo.features|Array[Object]|Служебная информация
contractInfo.identificationInfo|Array[Object]|Данные об [идентификации](https://qiwi.com/settings/account/identification.action) пользователя.
identificationInfo[].bankAlias|String|Акроним системы, в которой пользователь получил идентификацию:<br> `QIWI` - QIWI Кошелек.
identificationInfo[].identificationLevel|String|Текущий уровень идентификации кошелька. Возможные значения:<br>`ANONYMOUS` - без идентификации;<br> `SIMPLE`, `VERIFIED` - упрощенная идентификация;<br> `FULL` - полная идентификация.
userInfo|Object|Прочие пользовательские данные. Объект может отсутствовать, в зависимости от признака `userInfoEnabled` в запросе.
userInfo.defaultPayCurrency|Number|Код валюты баланса кошелька по умолчанию (number-3 ISO-4217)
userInfo.defaultPaySource|Number|Служебная информация
userInfo.email|String|E-mail пользователя
userInfo.firstTxnId|Number|Номер первой транзакции пользователя после регистрации
userInfo.language|String|Служебная информация
userInfo.operator|String|Название мобильного оператора номера пользователя
userInfo.phoneHash|String|Служебная информация
userInfo.promoEnabled|String|Служебная информация


# Идентификация пользователя {#ident}

Данный запрос позволяет отправить данные для упрощенной идентификации своего QIWI кошелька.

<aside class="warning">Допускается <b>не</b> идентифицировать не более 5 кошельков на одного владельца. См. п. 3.1.1 ч. III <a href="https://static.qiwi.com/ru/doc/oferta_lk.pdf">Оферты сервиса "QIWI Wallet"</a></aside>

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

<h3 class="request method">Запрос → POST</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/identification/v1/persons/<a>wallet</a>/identification</span></h3></li>
        <ul>
        <strong>В pathname POST-запроса используется параметр:</strong>
             <li><strong>wallet</strong> - номер кошелька, для которого получен токен доступа (с международным префиксом, но без <i>+</i>)</li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Данные параметры передаются в JSON-теле запроса:</span>
    </li>
</ul>


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

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

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

## Список платежей

Запрос выгружает список платежей и пополнений вашего кошелька.

~~~shell
Пример 1. Последние 10 платежей

user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments?rows=10"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"

Пример 2. Платежи за 10.05.2017

user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments?rows=50&startDate=2017-05-10T00%3A00%3A00%2B03%3A00&endDate=2017-05-10T23%3A59%3A59%2B03%3A00"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"

Пример 3. Продолжение списка платежей (в предыдущем запросе истории возвращены параметры nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23+03:00)

user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12%3A35%3A23%2B03%3A00"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~text
Пример 1. Последние 10 платежей с рублевого баланса и с привязанной карты
~~~

~~~http
GET /payment-history/v2/persons/79112223344/payments?rows=10&operation=OUT&sources[0]=QW_RUB&sources[1]=CARD HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

~~~text
Пример 2. Платежи за 10.05.2017
~~~

~~~http
GET /payment-history/v2/persons/79112223344/payments?rows=50&startDate=2017-05-10T00%3A00%3A00%2B03%3A00&endDate=2017-05-10T23%3A59%3A59%2B03%3A00 HTTP/1.1
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
GET /payment-history/v2/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12%3A35%3A23%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

<h3 class="request method">Запрос → GET</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/payment-history/v2/persons/<a>wallet</a>/payments?<a>parameter=value</a></span></h3></li>
        <ul>
        <strong>В pathname GET-запроса используется параметр:</strong>
             <li><strong>wallet</strong> - номер кошелька, для которого получен токен доступа (с международным префиксом, но без <i>+</i>)</li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Данные параметры передаются в строке запроса:</span>
    </li>
</ul>


Параметр|Тип|Описание
--------|----|----
rows | Integer |Число платежей в ответе, для разбивки отчета на части. Целое число от 1 до 50. **Обязательный параметр**
operation|String| Тип операций в отчете, для отбора. Допустимые значения:<br>`ALL` - все операции, <br>`IN` - только пополнения, <br>`OUT` - только платежи, <br>`QIWI_CARD` - только платежи по картам QIWI (QVC, QVP). <br>По умолчанию `ALL`
sources|Array[String]|Источники платежа, для отбора. Каждый источник задается как отдельный параметр и нумеруется элементом массива, начиная с нуля (`sources[0]`, `sources[1]` и т.д.). Допустимые значения: <br>`QW_RUB` - рублевый счет кошелька, <br>`QW_USD` - счет кошелька в долларах, <br>`QW_EUR` - счет кошелька в евро, <br>`CARD` - привязанные и непривязанные к кошельку банковские карты, <br>`MK` - счет мобильного оператора. Если не указаны, учитываются все источники
startDate | DateTime URL-encoded| Начальная дата поиска платежей. Дату можно указать в любой временной зоне `TZD` (формат `ГГГГ-ММ-ДД'T'чч:мм:ссTZD`), однако она должна совпадать с временной зоной в параметре `endDate`. Обозначение временной зоны `TZD`: `+чч:мм` или -`чч:мм` (временной сдвиг от GMT). **Используется только вместе с `endDate`**. По умолчанию, равна суточному сдвигу от текущей даты по московскому времени.
endDate | DateTime URL-encoded | Конечная дата поиска платежей. Дату можно указать в любой временной зоне `TZD` (формат `ГГГГ-ММ-ДД'T'чч:мм:ссTZD`), однако она должна совпадать с временной зоной в параметре `startDate`. Обозначение временной зоны `TZD`: `+чч:мм` или -`чч:мм` (временной сдвиг от GMT). **Используется только вместе с `startDate`**. По умолчанию, равна текущим дате/времени по московскому времени.
nextTxnDate | DateTime URL-encoded| Дата транзакции для отсчета от предыдущего списка (равна параметру `nextTxnDate` в предыдущем списке). **Используется только вместе с `nextTxnId`**
nextTxnId | Long | Номер предшествующей транзакции для отсчета от предыдущего списка (равен параметру `nextTxnId` в предыдущем списке). **Используется только вместе с `nextTxnDate`**


<aside class="notice">Максимальный допустимый интервал между <i>startDate</i> и <i>endDate</i> - 90 календарных дней.</aside>

<aside class="warning">Максимальная частота запросов истории платежей - не более 100 запросов в минуту для <b>одного и того же номера кошелька</b>. При превышении доступ к API блокируется на 5 минут.</aside>

<h3 class="request">Ответ ←</h3>

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

Успешный JSON-ответ содержит список платежей из истории кошелька, соответствующих заданному фильтру:

<a name="history_data"></a>

Параметр|Тип|Описание
--------|----|----
data|Array[Object]|Массив платежей. <br>Число платежей равно параметру `rows` из запроса
data[].txnId | Integer |ID транзакции в процессинге QIWI Wallet
data[].personId|Integer|Номер кошелька
data[].date|DateTime|Дата/время платежа, во временной зоне запроса (см. параметр `startDate`). Формат даты `ГГГГ-ММ-ДД'T'чч:мм:сс+03:00`
data[].errorCode|Number(Integer)|[Код ошибки платежа](#errors)
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
provider.id|Integer|ID провайдера в QIWI Wallet,
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
nextTxnDate|DateTime|Дата/время следующей транзакции в полном списке, время московское (в формате `ГГГГ-ММ-ДД'T'чч:мм:сс+03:00`)

## Статистика платежей {#stat}

Данный запрос используется для получения сводной статистики по суммам платежей за заданный период.

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments/total?startDate=2017-03-01T00%3A00%3A00%2B03%3A00&endDate=2017-03-31T11%3A44%3A15%2B03%3A00"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /payment-history/v2/persons/79112223344/payments/total?startDate=2017-03-01T00%3A00%3A00%2B03%3A00&endDate=2017-03-31T11%3A44%3A15%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

<h3 class="request method">Запрос → GET</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/payment-history/v2/persons/<a>wallet</a>/payments/total?<a>parameter=value</a></span></h3></li>
        <ul>
        <strong>В pathname GET-запроса используется параметр:</strong>
             <li><strong>wallet</strong> - номер кошелька, для которого получен токен доступа (с международным префиксом, но без <i>+</i>)</li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
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
sources|Array[String]|Источники платежа, учитываемые при подсчете статистики. Каждый источник задается как отдельный параметр и нумеруется элементом массива, начиная с нуля (`sources[0]`, `sources[1]` и т.д.). Допустимые значения: <br>`QW_RUB` - рублевый счет кошелька, <br>`QW_USD` - счет кошелька в долларах, <br>`QW_EUR` - счет кошелька в евро, <br>`CARD` - привязанные и непривязанные к кошельку банковские карты, <br>`MK` - счет мобильного оператора. Если не указан, учитываются все источники платежа.


<aside class="notice">Максимальный период для выгрузки статистики - 90 календарных дней.</aside>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

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

Успешный JSON-ответ содержит статистику платежей за выбранный период:

Параметр|Тип|Описание
--------|----|----
incomingTotal|Array[Object]|Данные о входящих платежах (пополнениях), отдельно по каждой валюте
incomingTotal[].amount | Number(Decimal) |Сумма пополнений за период
incomingTotal[].currency|String|Валюта пополнений
outgoingTotal|Array[Object]|Данные об исходящих платежах, отдельно по каждой валюте
outgoingTotal[].amount | Number(Decimal) |Сумма платежей за период
outgoingTotal[].currency|String|Валюта платежей

## Информация о транзакции {#txn_info}

Данный запрос используется для получения информации по определенной транзакции из вашей истории платежей.

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/transactions/9112223344"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /payment-history/v2/transactions/9112223344 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

<h3 class="request method">Запрос → GET</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/payment-history/v2/transactions/<a>transactionId</a>?<a>type=value</a></span></h3></li>
        <ul>
        <strong>В pathname GET-запроса используются два параметра:</strong>
             <li><strong>transactionId</strong> - номер транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].txnId</i> в ответе)</li>
             <li><strong>type</strong> - тип транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].type</i> в ответе). Данный параметр является необязательным и может не указываться</li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

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

Успешный JSON-ответ содержит данные о транзакции:

Параметр|Тип|Описание
--------|----|----
txnId | Integer |Копия параметра `transactionId` из запроса
personId|Integer|Номер кошелька
date|DateTime|Дата/время платежа, время московское (в формате `ГГГГ-ММ-ДД'T'чч:мм:сс+03:00`)
errorCode|Number(Integer)|[Код ошибки платежа](#errors)
error| String| Описание ошибки
type | String| Копия параметра `type` из запроса
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
provider.id|Integer|ID провайдера в QIWI Wallet,
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

## Квитанция платежа

Данный запрос используется для получения электронной квитанции (чека) по определенной транзакции из вашей истории платежей в формате PDF/JPEG в виде файла или почтовым сообщением на заданный e-mail.

### Файл квитанции

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/transactions/9112223344/cheque/file?type=IN&format=PDF"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /payment-history/v1/transactions/9112223344/cheque/file?type=IN&format=PDF HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com
~~~

<h3 class="request method">Запрос → GET</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/payment-history/v1/transactions/<a>transactionId</a>/cheque/file?<a>type=value&format=value</a></span></h3></li>
        <ul>
        <strong>В pathname GET-запроса используются три параметра:</strong>
             <li><strong>transactionId</strong> - номер транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].txnId</i> в ответе)</li>
             <li><strong>type</strong> - тип транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].type</i> в ответе)</li>
             <li><strong>format</strong> - тип файла, в который сохраняется квитанция. Допустимые значения: <i>JPEG</i>, <i>PDF</i></li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

~~~json
[
  ""
]
~~~

Успешный JSON-ответ содержит файл выбранного формата в бинарном виде.

### Отправка квитанции

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/payment-history/v1/transactions/9112223344/cheque/send?type=IN"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
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

<h3 class="request method">Запрос → POST</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/payment-history/v1/transactions/<a>transactionId</a>/cheque/send?<a>type=value</a></span></h3></li>
        <ul>
        <strong>В pathname POST-запроса используются два параметра:</strong>
             <li><strong>transactionId</strong> - номер транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].txnId</i> в ответе)</li>
             <li><strong>type</strong> - тип транзакции из <a href="#history_data">истории платежей</a> (параметр <i>data[].type</i> в ответе)</li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
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

Успешный JSON-ответ содержит HTTP-код отправки файла.

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

<h3 class="request method">Запрос → GET</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/funding-sources/v1/accounts/current</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

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
                "title": "QIWI Wallet"
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

Успешный ответ содержит JSON-массив ваших счетов QIWI Кошелька для фондирования платежей и их текущие балансы:

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

## Комиссионные тарифы {#rates}

Для провайдеров, оплата которых доступна через API, можно запросить комиссионные условия (стандартные тарифы).

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

<h3 class="request method">Запрос → GET</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/providers/<a>id</a>/form</span></h3></li>
        <ul>
        <strong>В pathname GET-запроса используется параметр:</strong>
             <li><strong>id</strong> - идентификатор провайдера. Возможные значения:
             <ul><li>99 - Перевод на QIWI Wallet</li>
             <li>1963 - Перевод на карту Visa (карты российских банков)</li>
             <li>21013 - Перевод на карту MasterCard (карты российских банков)</li>
             <li>Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:<ul><li>1960 – Перевод на карту Visa</li><li>21012 – Перевод на карту MasterCard</li></ul></li>
             <li>31652 - национальная платежная система МИР</li>
             <li>466 - Тинькофф Банк</li>
             <li>464 - Альфа-Банк</li>
             <li>821 - Промсвязьбанк</li>
             <li>815 - Русский Стандарт</li>
             <li><a href="#mnp">Идентификаторы операторов мобильной связи</a></li>
             <li><a href="#charity">Идентификаторы других провайдеров</a></li>
             <li>1717 - <a href="#freepay">платеж по банковским реквизитам</a></li></ul></li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

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

Успешный ответ содержит JSON-фрагмент с данными о ставках комиссии:

Параметр | Тип | Описание
-----|-----|------
commission | Object | Объект, содержащий данные об условиях комиссий.
commission.ranges | Array[Object] | Массив объектов с граничными условиями комиссий. Каждый объект содержит параметры:
ranges[].bound |  Number | сумма платежа, начиная с которой применяется условие
ranges[].rate | Number |  комиссия (абс.множитель)
ranges[].min | Number | минимальная сумма комиссии
ranges[].max | Number | максимальная сумма комиссии
ranges[].fixed | Number | фиксированная сумма комиссии

Для расчета полной комиссии за платеж (с учетом всех тарифов) по заданному набору платежных реквизитов используйте другой запрос (требует [авторизации](#auth_api)).

~~~shell
user@server:~$ curl -X POST 'https://edge.qiwi.com/sinap/providers/99/onlineCommission'
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

<h3 class="request method">Запрос → POST</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/providers/<a>id</a>/onlineCommission</span></h3></li>
        <ul>
        <strong>В pathname POST-запроса используется параметр:</strong>
             <li><strong>id</strong> - идентификатор провайдера. Возможные значения:
             <ul><li>99 - Перевод на QIWI Wallet</li>
             <li>1963 - Перевод на карту Visa (карты российских банков)</li>
             <li>21013 - Перевод на карту MasterCard (карты российских банков)</li>
             <li>Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:<ul><li>1960 – Перевод на карту Visa</li><li>21012 – Перевод на карту MasterCard</li></ul></li>
             <li>31652 - национальная платежная система МИР</li>
             <li>466 - Тинькофф Банк</li>
             <li>464 - Альфа-Банк</li>
             <li>821 - Промсвязьбанк</li>
             <li>815 - Русский Стандарт</li>
             <li><a href="#mnp">Идентификаторы операторов мобильной связи</a></li>
             <li><a href="#charity">Идентификаторы других провайдеров</a></li>
             <li>1717 - <a href="#freepay">платеж по банковским реквизитам</a></li></ul></li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON. Все параметры обязательны.</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
account| String|Номер телефона (с международным префиксом) или номер карты/счета получателя, в зависимости от провайдера
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
paymentMethod.type|String |Тип платежа, только `Account`
paymentMethod.accountId|String| Идентификатор счета, только `643`.
purchaseTotals | Object |Объект с платежными реквизитами
purchaseTotals.total|Object| Объект, содержащий данные о сумме платежа:
total.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
total.currency|String|Валюта (только `643`, рубли)

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~
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


<h3 class="request">Ответ ←</h3>

В параметре JSON-ответа `qwCommission.amount` возвращается рассчитанная сумма комиссии.

## Автозаполнение платежных форм {#payform}

[Пример ссылки (нажмите для перехода на форму)](https://qiwi.com/payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&extra%5B%27comment%27%5D=test123&currency=643)

Данный запрос отображает в браузере заполненную форму на сайте qiwi.com для совершения платежа.

~~~shell
https://qiwi.com/payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&extra%5B%27comment%27%5D=test123&currency=643
~~~

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>В пути ссылки и в строке запроса указываются параметры платежной формы.</span></li>
</ul>

Параметр|Описание|Тип|Обяз.
---------|--------|---|----
ID | Идентификатор провайдера (указывается в пути ссылки).<br>Возможные значения:<br>`99` - [Перевод на QIWI Wallet](#p2p)<br>`1963` - [Перевод на карту Visa](#cards) (карты российских банков)<br>`21013` - [Перевод на карту MasterCard](#cards) (карты российских банков)<br>Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:<br>`1960` – [Перевод на карту Visa](#cards)<br>`21012` – [Перевод на карту MasterCard](#cards)<br>[Идентификаторы операторов мобильной связи](#mnp)<br>[Идентификаторы других провайдеров](#charity) | Integer | +
amountInteger | Целая часть суммы платежа (рубли). Указывается в строке запроса. Если параметр не указан, соответствующее поле на форме будет пустым.|Integer| -
amountFraction | Дробная часть суммы платежа (копейки). Указывается в строке запроса. Если параметр не указан, соответствующее поле на форме будет пустым.|Integer| -
currency | Код валюты платежа. Указывается в строке запроса. **Обязательный параметр, если вы передаете в ссылке сумму платежа**|Константа, `643`|+
extra['comment'] | Комментарий (только для `ID`=99). Указывается в строке запроса. Имя параметра должно быть URL-закодировано.|URL-encoded string |-
extra['account'] |Номер телефона/счета/карты пользователя. Формат совпадает с форматом параметра `fields.account` в соответствующем платежном запросе. Имя параметра должно быть URL-закодировано.|URL-encoded string |-


## Перевод на QIWI Кошелек {#p2p}

~~~shell
user@server:~$ curl -X POST 'https://edge.qiwi.com/sinap/api/v2/terms/99/payments'
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


<h3 class="request method">Запрос → POST</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/99/payments</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON. Все параметры обязательны.</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Данные о сумме платежа:
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`
paymentMethod.accountId|String| Константа, `643`.
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String|Номер телефона получателя (с международным префиксом)
comment|String|Комментарий к платежу. Необязательный параметр.

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

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

Успешный JSON-ответ содержит данные о принятом платеже:

Параметр | Тип | Описание
-----|----|-----
id | String | Копия параметра `id` из исходного запроса
terms | String | Константа, `99`
fields|Object|Копия объекта `fields` из исходного запроса.
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
comment|String|Копия параметра `comment` из исходного запроса (если присутствует в запросе)
transaction|Object|Объект с данными о транзакции в процессинге QIWI Wallet. Параметры:
transaction.id|String|ID транзакции в процессинге QIWI Wallet
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге QIWI Wallet. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

## Оплата сотовой связи {#cell}

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

<h3 class="request method">Запрос → POST</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/<a>ID</a>/payments</span></h3></li>
        <ul>
        <strong>В pathname POST-запроса используется параметр:</strong>
             <li><strong>ID</strong> - идентификатор оператора. Определяется с помощью <a href="#mnp">проверки мобильного оператора</a></li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON. Все параметры обязательны.</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Данные о сумме платежа:
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`
paymentMethod.accountId|String| Константа, `643`.
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String|Номер мобильного телефона для пополнения (без префикса `8`)

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

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

Успешный JSON-ответ содержит данные о принятом платеже:

Параметр | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из исходного запроса
terms | String | Копия параметра `ID` из URL запроса
fields|Object|Копия объекта `fields` из исходного запроса.
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге QIWI Wallet. Параметры:
transaction.id|String|ID транзакции в процессинге QIWI Wallet
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге QIWI Wallet. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

Предварительное определение оператора мобильного номера выполняется вспомогательным запросом. В ответе возвращается идентификатор оператора для [запроса пополнения телефона](#cell).

~~~shell
user@server:~$ curl -X POST "https://qiwi.com/mobile/detect.action"
  --header "Accept: application/json"
  --header "Content-Type: application/x-www-form-urlencoded"
  -d "phone=%2B79651238341"
~~~

~~~http
POST /mobile/detect.action HTTP/1.1
Host: qiwi.com
Accept: application/json
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache

phone=%2B79651238341
~~~

<h3 class="request method" id="mnp">Запрос → POST</h3>

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
    <li><h3>Параметры</h3><span>Параметр передается в теле запроса как formdata.</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
phone | String URL-encoded |Мобильный номер в международном формате. Обязательный параметр.

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

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

~~~text
Не удалось определить мобильного оператора
~~~

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~


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

<h3 class="request">Ответ ←</h3>

Ответ с HTTP Status 200 и параметром `code.value` = 0 является признаком успешной проверки. Идентификатор оператора находится в параметре `message`.

Ответ с HTTP Status 200 и параметром `code.value` = 2 означает, что невозможно определить оператора.

## Перевод на карту {#cards}

Запрос выполняет перевод на карты платежных систем Visa или MasterCard. Предварительно можно [проверить тип платежной системы](#card_check).

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

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1960/payments"
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
          "account": "402865XXXXXXXXXX",
          "rec_address": "Ленинский проспект 131, 56",
          "rec_city": "Москва",
          "rec_country": "Россия",
          "reg_name": "Виктор",
          "reg_name_f": "Петров",
          "rem_name": "Сергей",
          "rem_name_f": "Иванов"
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

<h3 class="request method">Запрос → POST</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/<a>ID</a>/payments</span></h3></li>
        <ul>
        <strong>В pathname POST-запроса используется параметр:</strong>
             <li><strong>ID</strong> - идентификатор провайдера. Возможные значения:
             <ul>
             <li>1963 - Перевод на карту Visa (карты российских банков)</li>
             <li>21013 - Перевод на карту MasterCard (карты российских банков)</li>
             <li>Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:
             <ul><li>1960 – Перевод на карту Visa</li>
             <li>21012 – Перевод на карту MasterCard</li></ul></li>
             <li>31652 - национальная платежная система МИР</li>
             </ul></li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON. Все параметры обязательны.</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Данные о сумме платежа:
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`
paymentMethod.accountId|String| Константа, `643`.
fields|Object| Реквизиты платежа. Содержит параметры:
fields.account| String|Номер банковской карты получателя
fields.rem_name|String|Имя отправителя. Требуется только для ID (идентификатор провайдера в запросе) 1960, 21012
fields.rem_name_f|String|Фамилия отправителя. Требуется только для ID (идентификатор провайдера в запросе) 1960, 21012
fields.rec_address|String|Адрес отправителя (без почтового индекса, в произвольной форме). Требуется только для ID (идентификатор провайдера в запросе) 1960, 21012
fields.rec_city|String|Город отправителя. Требуется только для ID (идентификатор провайдера в запросе) 1960, 21012
fields.rec_country|String|Страна отправителя. Требуется только для ID (идентификатор провайдера в запросе) 1960, 21012
fields.reg_name|String|Имя **получателя**. Требуется только для ID (идентификатор провайдера в запросе) 1960, 21012
fields.reg_name_f|String|Фамилия **получателя**. Требуется только для ID (идентификатор провайдера в запросе) 1960, 21012

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~
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

Успешный ответ содержит JSON-данные о принятом платеже:

Параметр | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из исходного запроса
terms | String | Параметр **ID** из URL запроса
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String|Маскированный номер банковской карты получателя
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге QIWI Wallet. Параметры:
transaction.id|String|ID транзакции в процессинге QIWI Wallet
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге QIWI Wallet. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

Определение платежной системы карты выполняется отдельным запросом. В ответе возвращается идентификатор платежной системы для [запроса перевода на карту](#cards).

Запрос не требует  [токена доступа](#auth_param).

~~~shell
user@server:~$ curl -X POST "https://qiwi.com/card/detect.action"
  --header "Accept: application/json"
  --header "Content-Type: application/x-www-form-urlencoded"
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

<h3 class="request method" id="card_check">Запрос → POST</h3>

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
cardNumber | String |Немаскированный номер карты. Обязательный параметр

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

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

~~~text
Не удалось определить платежную систему карты
~~~

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

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

<h3 class="request">Ответ ←</h3>

Ответ с HTTP Status 200 и параметром `code.value` = 0 является признаком успешной проверки. Идентификатор платежной системы находится в параметре `message`.

Ответ с HTTP Status 200 и параметром `code.value` = 2 означает, что в номере карты ошибка.

## Банковский перевод {#banks}

Запрос выполняет перевод на банковские карты/счета российских банков.

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

<h3 class="request method">Запрос → POST</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/<a>ID</a>/payments</span></h3></li>
        <ul>
        <strong>В pathname POST-запроса используется параметр:</strong>
             <li><strong>ID</strong> - идентификатор провайдера. Возможные значения:
             <ul><li>466 - Тинькофф Банк</li>
             <li>464 - Альфа-Банк</li>
             <li>821 - Промсвязьбанк</li>
             <li>815 - Русский Стандарт</li>
             </ul></li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON. Все параметры обязательны.</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Данные о сумме платежа:
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`
paymentMethod.accountId|String| Константа, `643`.
fields|Object| Реквизиты платежа. Содержит параметры:
fields.account| String| Номер карты/счета получателя
fields.exp_date| String|Срок действия карты, в формате `ММГГ` (например, `0218`). **Параметр указывается только в случае перевода на карту.**
fields.account_type| String|Тип банковского идентификатора. Допустимые значения:<br>для Тинькофф Банк - карта `1`, договор `3`<br>для Альфа-Банка - карта `1`, счет `2`<br>для Промсвязьбанка - карта `7`, счет `9`<br>для банка Русский Стандарт - карта `1`, счет `2`, договор `3`.

<h3 class="request">Ответ ←</h3>

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

Успешный JSON-ответ содержит данные о принятом платеже:

Параметр | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из исходного запроса
terms | String | Параметр **ID** из URL запроса
fields|Object|Копия объекта `fields` из исходного запроса. **В случае перевода на банковскую карту, в параметре** `fields.account` **находится маскированный номер банковской карты получателя**.
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге QIWI Wallet. Параметры:
transaction.id|String|ID транзакции в процессинге QIWI Wallet
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге QIWI Wallet. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

## Оплата других услуг {#charity}

Оплата услуги по идентификатору пользователя. Данный запрос применяется для провайдеров, использующих в реквизитах единственный пользовательский идентификатор, без проверки номера аккаунта.

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

<h3 class="request method">Запрос → POST</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/<a>ID</a>/payments</span></h3></li>
        <ul>
        <strong>В pathname POST-запроса используется параметр:</strong>
             <li><strong>ID</strong> - идентификатор провайдера. Возможные значения:
             <ul><li>674 - OnLime</li>
             <li>Идентификатор другого интернет-провайдера</li>
             <li>1239 - Подари жизнь</li>
             <li>Идентификатор другого благотворительного фонда</li>
             </ul></li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON. Все параметры обязательны.</span>
    </li>
</ul>

<aside class="notice">Поиск идентификатора выполняется на сайте qiwi.com в поисковой строке. Идентификатор находится в URL ссылки на платежную форму провайдера вида <a>https://qiwi.com/payment/form.action?provider=ID</a> или <a>https://qiwi.com/payment/form/ID</a></aside>

![Поиск провайдера](/images/provider_id.jpg)

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Данные о сумме платежа:
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`
paymentMethod.accountId|String| Константа, `643`.
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String| Пользовательский идентификатор

<h3 class="request">Ответ ←</h3>

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

Успешный JSON-ответ содержит данные о принятом платеже:

Параметр | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из исходного запроса
terms | String | Параметр **ID** из URL запроса
fields|Object|Копия объекта `fields` из исходного запроса.
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге QIWI Wallet. Параметры:
transaction.id|String|ID транзакции в процессинге QIWI Wallet
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге QIWI Wallet. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

## Платеж по свободным реквизитам {#freepay}

Оплата услуг коммерческих организаций по их банковским реквизитам.

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

<h3 class="request method">Запрос → POST</h3>

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/1717/payments</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON. Все параметры обязательны.</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| Данные о сумме платежа:
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|Валюта (только `643`, рубли)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
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

<h3 class="request">Ответ ←</h3>

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

Успешный JSON-ответ содержит данные о принятом платеже:

Параметр | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из исходного запроса
terms | String | Константа, `1717`
fields|Object|Копия объекта `fields` из исходного запроса
sum|Object|Копия объекта `sum` из исходного запроса
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге QIWI Wallet. Параметры:
transaction.id|String|ID транзакции в процессинге QIWI Wallet
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге QIWI Wallet. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments).

# Коды ошибок {#errors}

В случае ошибки API возвращается HTTP-код ошибки.

HTTP Код | Секция API | Описание
---|-----|---------
400 | Все | Ошибка синтаксиса запроса (неправильный формат данных)
401 | Все | Неверный токен или истек срок действия токена
403 | Все | Нет прав на данный запрос (недостаточно разрешений у токена)
404 | История платежей, Информация о транзакции, Отправка квитанции | Не найдена транзакция или отсутствуют платежи с указанными признаками
404 | Балансы, Профиль пользователя, Идентификация пользователя | Не найден кошелек
423 | История платежей | Слишком много запросов, сервис временно недоступен

Следующие ошибки возвращаются в запросах [истории платежей](#payments_history) и [транзакции](#txn_info) в параметре `errorCode`:

errorCode | Описание
----|------
0 |OK
3 |Техническая ошибка, нельзя отправить запрос провайдеру
4 |Неверный формат счета/телефона
5 |Номер не принадлежит оператору
8 |Прием платежа запрещен по техническим причинам
131 |Платежи на выбранного провайдера запрещено проводить из данной страны.
202 |Ошибка в параметрах запроса
220 |Недостаточно средств
241 |Сумма платежа меньше минимальной
242 |Сумма платежа больше максимальной
319 |Платеж невозможен
500 |По техническим причинам этот платеж не может быть выполнен. Для совершения платежа обратитесь, пожалуйста, в свой обслуживающий банк
522 |Неверный номер или срок действия карты получателя
547 | Ошибка в сроке действия карты получателя
548 |Истек срок действия карты получателя
561 |Платеж отвергнут оператором банка получателя
702 |Платеж не проведен из-за ограничений у получателя. Подробности по телефону: 8-800-707-77-59
705 |Ежемесячный лимит платежей и переводов для статуса Стандарт - 200 000 р. Для увеличения лимита пройдите идентификацию.
746 |Превышен лимит по платежам в пользу провайдера
852 |Превышен лимит по платежам в пользу провайдера
893 |Срок действия перевода истек
1050 | Превышен лимит на операции, либо превышен дневной лимит на переводы на карты Visa/MasterCard
 |
 |
 |