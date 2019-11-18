---
title: API QIWI Кошелька

search: true

metatitle: API QIWI Кошелька

metadescription: API QIWI Кошелька позволяет автоматизировать выполнение платежей и получение отчетов о платежах, информации о счёте, идентификации.

language_tabs:
  - shell: cURL
  - php: PHP
  - python: Python
  - http: Запрос/ответ

toc_footers:
 - <a href='/ru/qiwi-wallet-api-release-notes/index.html'>Список изменений</a>
 - <a href='/'>На главную</a>
 - <a href='mailto:api_help@qiwi.com'>Обратная связь</a>
 - <a href='/sandbox/index.html'>Попробовать API</a>

---

 *[Токен]: Символьная строка для аутентификации пользователя в API.
 *[токен]: Символьная строка для аутентификации пользователя в API.
 *[API]: Application Programming Interface -  набор готовых методов, предоставляемых приложением (системой) для использования во внешних программных продуктах.
 *[JSON]: JavaScript Object Notation - текстовый формат обмена данными, основанный на *JavaScript*.



# Введение {#intro}

###### Последнее обновление: 2019-10-17 | [Предложить свои правки на GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/qiwi-wallet-personal_ru.html.md)

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
    * Проведение платежей без SMS - разрешает выполнение [платежных запросов](#payments) без подтверждения по SMS (независимо от настроек <https://qiwi.com/settings/options/security.action>), [оплату счетов](#pay_invoice), использование [уведомлений](#webhook)

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

<h3 class="request method">Запрос → GET</h3>

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

~~~python
import requests
import json

# Профиль пользователя
def get_profile(api_access_token):
    s7 = requests.Session()
    s7.headers['Accept']= 'application/json'
    s7.headers['authorization'] = 'Bearer ' + api_access_token
    p = s7.get('https://edge.qiwi.com/person-profile/v1/profile/current?authInfoEnabled=true&contractInfoEnabled=true&userInfoEnabled=true')
    print(p)
    return json.loads(p.text)
~~~

~~~python
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# Полная информация о профиле пользователя
get_profile(api_access_token)

# Профиль пользователя
# статус блокировки
get_profile(api_access_token)['contractInfo']['blocked']

# Профиль пользователя
# уровень идентификации в Киви Банке
get_profile(api_access_token)['contractInfo']['identificationInfo'][0]['identificationLevel']

# привязанный email
get_profile(api_access_token)['authInfo']['boundEmail']
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/person-profile/v1/profile/current?<a>parameter=value</a></span></h3></li>
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


# Идентификация {#identification}

## Идентификация пользователя {#ident}

Данный запрос позволяет отправить данные для упрощенной идентификации своего QIWI кошелька.

<aside class="warning">Допускается идентифицировать не более 5 кошельков на одного владельца. См. п. 3.1.1 ч. III <a href="https://static.qiwi.com/ru/doc/oferta_lk.pdf">Оферты сервиса "QIWI Wallet"</a></aside>

Для получения статуса упрощенно идентифицированного кошелька необходимо предоставить следующие данные о пользователе-владельце кошелька:

* ФИО
* Серия / Номер паспорта
* Дата рождения
* ИНН, СНИЛС или номер полиса ОМС - необязательно.

Для идентификации кошелька вы обязательно должны отправить ФИО, серию/номер паспорта и дату рождения. Если данные прошли проверку, то в ответе будет отображен ваш ИНН и упрощенная идентификация кошелька будет установлена. В случае если данные не прошли проверку, кошелек остается в статусе "Без идентификации".

<h3 class="request method">Запрос → POST</h3>

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

~~~python
import requests
import json

# идентификация
def get_identification(api_access_token,my_login):
    s = requests.Session()
    s.headers['authorization'] = 'Bearer ' + api_access_token
    res = s.get('https://edge.qiwi.com/identification/v1/persons/'+my_login+'/identification')
    print(res)
    return json.loads(res.text)
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/identification/v1/persons/<a>wallet</a>/identification</span></h3>
        <ul>
        <strong>В pathname POST-запроса используется параметр:</strong>
             <li><strong>wallet</strong> - номер кошелька, для которого получен токен доступа (с международным префиксом без <i>+</i>)</li>
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

~~~python
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'
get_identification(api_access_token,mylogin)

<Response [200]>

{'birthDate': '1984-01-09',
 'firstName': 'Иванов',
 'id': 79262111317,
 'inn': 'xxxxxxx',
 'lastName': 'Иванов',
 'middleName': 'Иванович',
 'oms': None,
 'passport': 'xxxx xxxxxx',
 'snils': None,
 'type': 'FULL'}
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


## Данные идентификации {#ident_data}

Данный запрос позволяет выгрузить маскированные данные и статус идентификации своего QIWI кошелька.

[Подробнее об идентификации](https://qiwi.com/settings/account/identification.action)

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl -X GET "https://edge.qiwi.com/identification/v1/persons/79111234567/identification"
  --header "Accept: application/json"
  --header "Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9"
~~~

~~~http
GET /identification/v1/persons/79111234567/identification HTTP/1.1
Accept: application/json
Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9
Host: edge.qiwi.com
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/identification/v1/persons/<a>wallet</a>/identification</span></h3>
        <ul>
        <strong>В pathname GET-запроса используется параметр:</strong>
             <li><strong>wallet</strong> - номер кошелька, для которого получен токен доступа (с международным префиксом, но без <i>+</i>)</li>
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
  "birthDate": "1996-03-18",
  "firstName": "Иван",
  "id": 79111234567,
  "inn": "77***01",
  "lastName": "Иванов",
  "middleName": "Иванович",
  "oms": "",
  "passport": "43***11",
  "snils": "",
  "type": "VERIFIED"
}
~~~

Успешный ответ в формате JSON содержит маскированные данные идентификации кошелька:

Параметр|Тип|Описание
--------|----|----
id|  Number  |Номер кошелька пользователя
type | String | Текущий уровень идентификации кошелька:<br>`SIMPLE` - без идентификации.<br>`VERIFIED` - упрощенная идентификация (данные для идентификации успешно прошли проверку).<br>`FULL` – если кошелек уже ранее получал полную идентификацию по данным ФИО, номеру паспорта и дате рождения.
birthDate |String | Дата рождения пользователя
firstName| String | Имя пользователя
middleName | String | Отчество пользователя
lastName|  String | Фамилия пользователя
passport | String | Серия и номер паспорта пользователя (первые и последние 2 цифры)
inn| String|  ИНН пользователя (первые и последние 2 цифры)
snils |String | Номер СНИЛС пользователя (первые и последние 2 цифры)
oms| String | Номер полиса ОМС пользователя (первые и последние 2 цифры)

# История платежей {#payments_history}

## Список платежей {#payments_list}

Запрос выгружает список платежей и пополнений вашего кошелька.

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/payment-history-controller-v-2/getPaymentHistoryByUserUsingGET_1)

<h3 class="request method">Запрос → GET</h3>

>Пример 1. Последние 10 платежей

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments?rows=10"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

> Пример 2. Платежи за 10.05.2017

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments?rows=50&startDate=2017-05-10T00%3A00%3A00%2B03%3A00&endDate=2017-05-10T23%3A59%3A59%2B03%3A00"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

> Пример 3. Продолжение списка платежей (в предыдущем запросе истории возвращены параметры nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23+03:00)

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12%3A35%3A23%2B03%3A00"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

> Пример 1. Последние 10 платежей с рублевого баланса и с привязанной карты

~~~http
GET /payment-history/v2/persons/79112223344/payments?rows=10&operation=OUT&sources[0]=QW_RUB&sources[1]=CARD HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

> Пример 2. Платежи за 10.05.2017

~~~http
GET /payment-history/v2/persons/79112223344/payments?rows=50&startDate=2017-05-10T00%3A00%3A00%2B03%3A00&endDate=2017-05-10T23%3A59%3A59%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

> Пример 3. Продолжение списка платежей (в предыдущем запросе истории возвращены параметры nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23+03:00)

~~~http
GET /payment-history/v2/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12%3A35%3A23%2B03%3A00 HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

~~~python
import requests
import json

# История платежей - последние и следующие n платежей
def payment_history_last(my_login, api_access_token, rows_num, next_TxnId, next_TxnDate):
    s = requests.Session()
    s.headers['authorization'] = 'Bearer ' + api_access_token  
    parameters = {'rows': rows_num, 'nextTxnId': next_TxnId, 'nextTxnDate': next_TxnDate}
    h = s.get('https://edge.qiwi.com/payment-history/v2/persons/' + my_login + '/payments', params = parameters)
    print(h)
    return json.loads(h.text)
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/payment-history/v2/persons/<a>wallet</a>/payments?<a>parameter=value</a></span></h3></li>
        <ul>
        <strong>В pathname GET-запроса используется параметр:</strong>
             <li><strong>wallet</strong> - номер кошелька, для которого получен токен доступа (с международным префиксом без <i>+</i>)</li>
        </ul>
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

~~~python
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# последние 20 платежей
payment_history_last(mylogin, api_access_token, '20','','')

# дата и время следующего платежа
nextTxnDate = payment_history_last(mylogin, api_access_token, '3','','')['nextTxnDate']

# id транзакции следующего платежа
nextTxnId = payment_history_last(mylogin, api_access_token, '3','','')['nextTxnId']

# История платежей - последние и следующие n платежей
payment_history_last(mylogin, api_access_token, '3', nextTxnId, nextTxnDate)
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
data[].account| String|Для платежей - номер счета получателя. Для пополнений - номер отправителя, терминала или название агента пополнения кошелька
data[].sum|Object| Данные о сумме платежа или пополнения. Параметры:
sum.amount|Number(Decimal)|сумма,
sum.currency|String|валюта
data[].commission|Object| Данные о комиссии платежа. Параметры:
commission.amount|Number(Decimal)|сумма,
commission.currency|String|валюта
data[].total|Object| Данные о фактической сумме платежа или пополнения. Параметры:
total.amount|Number(Decimal)|сумма (равна сумме платежа `sum.amount` и комиссии `commission.amount`),
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

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/payment-history-controller-v-2/getPaymentHistoryTotalByUserUsingGET_1)

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/persons/79112223344/payments/total?startDate=2017-03-01T00%3A00%3A00%2B03%3A00&endDate=2017-03-31T11%3A44%3A15%2B03%3A00"
  --header "Accept: application/json"
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
import json

# История платежей - сумма за диапазон дат
def payment_history_summ_dates(my_login, api_access_token, start_Date, end_Date):
    s = requests.Session()
    s.headers['authorization'] = 'Bearer ' + api_access_token
    parameters = {'startDate': start_Date,'endDate': end_Date}
    h = s.get('https://edge.qiwi.com/payment-history/v2/persons/' + my_login + '/payments/total', params = parameters)
    print(h)
    return json.loads(h.text)
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/payment-history/v2/persons/<a>wallet</a>/payments/total?<a>parameter=value</a></span></h3></li>
        <ul>
        <strong>В pathname GET-запроса используется параметр:</strong>
             <li><strong>wallet</strong> - номер кошелька, для которого получен токен доступа (с международным префиксом без <i>+</i>)</li>
        </ul>
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
sources|Array[String]|Источники платежа, учитываемые при подсчете статистики. Каждый источник задается как отдельный параметр и нумеруется элементом массива, начиная с нуля (`sources[0]`, `sources[1]` и т.д.). Допустимые значения: <br>`QW_RUB` - рублевый счет кошелька, <br>`QW_USD` - счет кошелька в долларах, <br>`QW_EUR` - счет кошелька в евро, <br>`CARD` - привязанные и непривязанные к кошельку банковские карты, <br>`MK` - счет мобильного оператора. Если не указан, учитываются все источники платежа.


<aside class="notice">Максимальный период для выгрузки статистики - 90 календарных дней.</aside>

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
payment_history_summ_dates(mylogin, api_access_token, '2019-04-12T00:00:00Z','2019-07-11T23:59:59Z')

{'incomingTotal': [{'amount': 3.33, 'currency': 840},
  {'amount': 3481, 'currency': 643}],
 'outgoingTotal': [{'amount': 3989.98, 'currency': 643},
  {'amount': 3.33, 'currency': 840}]}
~~

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

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/payment-history-controller-v-2/getPaymentHistoryByTransactionUsingGET_1)

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v2/transactions/9112223344"
  --header "Accept: application/json"
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
import json

# История платежей - информация по транзакции
def payment_history_transaction(api_access_token, transaction_id, transaction_type):
    s = requests.Session()
    s.headers['authorization'] = 'Bearer ' + api_access_token  
    parameters = {'type': transaction_type} # transaction_type 'IN' 'OUT'
    h = s.get('https://edge.qiwi.com/payment-history/v1/transactions/'+transaction_id, params = parameters)
    print(h)
    return json.loads(h.text)
~~~

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
payment_history_transaction(api_access_token, '11181101215', 'OUT')

# История платежей - информация по транзакции из истории платежей
last_txn_id = payment_history_last(mylogin, api_access_token, '20','','')['data'][5]['txnId']
last_txn_type = payment_history_last(mylogin, api_access_token, '20','','')['data'][5]['type']

payment_history_transaction(api_access_token, str(last_txn_id), last_txn_type)
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
account| String|Для платежей - номер счета получателя. Для пополнений - номер отправителя, терминала или название агента пополнения кошелька
sum|Object| Данные о сумме платежа или пополнения. Параметры:
sum.amount|Number(Decimal)|сумма,
sum.currency|String|валюта
commission|Object| Данные о комиссии платежа. Параметры:
commission.amount|Number(Decimal)|сумма,
commission.currency|String|валюта
total|Object| Данные о фактической сумме платежа или пополнения. Параметры:
total.amount|Number(Decimal)|сумма (равна сумме платежа `sum.amount` и комиссии `commission.amount`),
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

## Квитанция платежа {#payment_receipt}

Данный запрос используется для получения электронной квитанции (чека) по определенной транзакции из вашей истории платежей в формате PDF/JPEG в виде файла или почтовым сообщением на заданный e-mail.

### Файл квитанции {#receipt_file}

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/cheque-controller-v-1/getChequeBytesUsingGET)

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/transactions/9112223344/cheque/file?type=IN&format=PDF"
  --header "Accept: application/json"
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
import json

# История платежей - получение текста чека в файле
def payment_history_cheque_file(transaction_id, transaction_type, filename, api_access_token):
    s = requests.Session()
    s.headers['Accept'] ='application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    parameters = {'type': transaction_type,'format': 'PDF'}
    h = s.get('https://edge.qiwi.com/payment-history/v1/transactions/'+transaction_id+'/cheque/file', params=parameters)
    print(h)
    with open(filename + '.pdf', 'wb') as f:
        f.write(h.content)
~~~

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

~~~python
import requests
import json

# История платежей - отправить чек на email
def payment_history_cheque_send(transaction_id, transaction_type, email, api_access_token):
    s = requests.Session()
    s.headers['content-type'] ='application/json'
    s.headers['Accept'] ='application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    postjson = {'email':email}
    h = s.post('https://edge.qiwi.com/payment-history/v1/transactions/' + transaction_id + '/cheque/send?type=' + transaction_type, json=postjson)
    print(h)
~~~

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

last_txn_id = payment_history_last(mylogin, api_access_token, '20','','')['data'][5]['txnId']
last_txn_type = payment_history_last(mylogin, api_access_token, '20','','')['data'][5]['type']

# История платежей - оптравить чек на email
payment_history_cheque_send(str(last_txn_id), last_txn_type, 'mmd@yandex.ru', api_access_token)
~~~

Успешный JSON-ответ содержит HTTP-код отправки файла.

# Баланс QIWI Кошелька {#balance}

## Список балансов {#balances_list}

Запрос выгружает текущие балансы ваших счетов QIWI Кошелька.

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/getByAliasUsingGET_1)

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/funding-sources/v2/persons/79115221133/accounts"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /funding-sources/v2/persons/79115221133/accounts HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

~~~python
import requests
import json

# Баланс QIWI Кошелька
def balance(login, api_access_token):
    s = requests.Session()
    s.headers['Accept']= 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token  
    b = s.get('https://edge.qiwi.com/funding-sources/v2/persons/' + login + '/accounts')
    print(b)
    return json.loads(b.text)
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/funding-sources/v2/persons/<a>personId</a>/accounts</span></h3></li>
        <ul>
        <strong>В pathname запроса используется параметр:</strong>
             <li><strong>personId</strong> - номер кошелька, для которого получен токен доступа (с международным префиксом без <i>+</i>)</li>
        </ul>
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
    "accounts": [
        {
            "alias": "mc_beeline_rub",
            "fsAlias": "qb_mc_beeline",
            "bankAlias": "QIWI",
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
            "bankAlias": "QIWI",
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

~~~python
# номер кошелька в формате 79992223344
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# все балансы
balance(mylogin,api_access_token)

# рублевый баланс
balances = balance(mylogin,api_access_token)['accounts']
rubAlias = [x for x in balances if x.alias =='qw_wallet_rub']
rubAlias['balance']['amount']
~~~

Успешный ответ содержит JSON-массив ваших счетов QIWI Кошелька для фондирования платежей и текущие балансы счетов:

Параметр|Тип|Описание
--------|----|----
accounts|Array[Object]|Массив балансов
accounts[].alias | String |Псевдоним пользовательского баланса
accounts[].fsAlias | String |Псевдоним банковского баланса
accounts[].bankAlias | String |Псевдоним банка
accounts[].title|String|Название соответствующего счета кошелька
accounts[].hasBalance|Boolean|Логический признак реального баланса в системе QIWI Кошелек (не привязанная карта, не счет мобильного телефона и т.д.)
accounts[].currency | Number| Код валюты баланса (number-3 ISO-4217). Возвращаются балансы в следующих валютах: 643 - российский рубль, 840 - американский доллар, 978 - евро
accounts[].type|Object|Сведения о счете
type.id, type.title| String| Описание счета
accounts[].balance|Object |Сведения о балансе данного счета.<br>Если вернулся `null` и при этом параметр `accounts[].hasBalance` равен `true`, повторите запрос с дополнительными параметрами:<br>`timeout=1000` и `alias=accounts[].alias` (псевдоним этого баланса).<br>Например<br>`GET /funding-sources/v1/accounts/current?timeout=1000&alias=qw_wallet_rub`
balance.amount|Number|Текущий баланс данного счета
balance.currency | Number| Код валюты баланса (number-3 ISO-4217)

## Создание баланса {#balance_create}

Запрос создает новый счет и баланс в вашем QIWI Кошельке. Список доступных для создания счетов можно получить [другим запросом](#funding_offer).

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/createAccountUsingPOST)

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/funding-sources/v2/persons/79115221133/accounts"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  -d '{  "alias": "qw_wallet_eur"}'
~~~

~~~http
POST /funding-sources/v2/persons/79115221133/accounts HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com

{ "alias": "qw_wallet_eur" }
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/funding-sources/v2/persons/<a>personId</a>/accounts</span></h3></li>
        <ul>
        <strong>В теле запроса используется параметр:</strong>
             <li><strong>personId</strong> - номер кошелька, для которого получен токен доступа (с международным префиксом без <i>+</i>)</li  >
        </ul>
</ul>

<ul class="nestedList params">
    <li><h3>Параметр</h3><span>Данный параметр передается в JSON-теле запроса:</span>
    </li>
</ul>


Параметр|Тип|Описание
--------|----|----
alias|String| Псевдоним нового счета (см. [запрос доступных счетов](#funding_offer))

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 201 Created
Content-Type: application/json
~~~

Успешный ответ содержит HTTP-код запроса 201.

## Запрос доступных счетов {#funding_offer}

Запрос отображает псевдонимы счетов, доступных для создания в вашем QIWI Кошельке.

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/getAccountsOfferUsingGET)

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl -X GET "https://edge.qiwi.com/funding-sources/v2/persons/79115221133/accounts/offer"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /funding-sources/v2/persons/79115221133/accounts/offer HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/funding-sources/v2/persons/<a>personId</a>/accounts/offer</span></h3></li>
        <ul>
        <strong>В теле запроса используется параметр:</strong>
             <li><strong>personId</strong> - номер кошелька, для которого получен токен доступа (с международным префиксом без <i>+</i>)</li>
        </ul>
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

{ { "alias": "qw_wallet_eur", "currency": 978 }, {} }
~~~

Успешный JSON-ответ содержит данные о счетах, которые можно создать:

Параметр|Тип|Описание
--------|----|----
{} | Object |Коллекция описаний счетов
Object.alias|String|Псевдоним счета
Object.currency|Integer|ID валюты счета

## Установка баланса по умолчанию  {#default_balance}

Запрос устанавливает для вашего QIWI Кошелька счет, баланс которого будет использоваться для фондирования всех платежей по умолчанию. Счет должен содержаться в [списке счетов](#balances_list)

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/saveAccountAttributesUsingPATCH)

<h3 class="request method">Запрос → PATCH</h3>

~~~shell
user@server:~$ curl -X PATCH "https://edge.qiwi.com/funding-sources/v2/persons/79115221133/accounts/qw_wallet_usd"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  -d '{ "defaultAccount": true }'
~~~

~~~http
PATCH /funding-sources/v2/persons/79115221133/accounts/qw_wallet_usd HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Content-type: application/json
Host: edge.qiwi.com

{ "defaultAccount": true }
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/funding-sources/v2/persons/<a>personId</a>/accounts/<a>accountAlias</a></span></h3></li>
        <ul>
        <strong>В теле запроса используются обязательные параметры:</strong>
             <li><strong>personId</strong> - номер кошелька, для которого получен токен доступа (с международным префиксом без <i>+</i>)</li>
             <li><strong>accountAlias</strong> - псевдоним счета в кошельке из <a href="#balances_list">списка счетов</a> (параметр <i>accounts[].alias</i> в ответе)</li>
        </ul>
</ul>

<ul class="nestedList params">
    <li><h3>Параметр</h3><span>Данный параметр передается в JSON-теле запроса:</span>
    </li>
</ul>


Параметр|Тип|Описание
--------|----|----
defaultAccount|Boolean| Признак установки счета по умолчанию

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

Успешный ответ содержит HTTP-код запроса 200.

# Платежи {#payments}

## Комиссионные тарифы {#rates}

Для провайдеров, оплата которых доступна через API, можно запросить комиссионные условия (стандартные тарифы).

<h3 class="request method">Запрос → GET</h3>

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

~~~python
import requests
import json

# Стандартные комиссии
def get_prv_commission(prv_id):
    s = requests.Session()
    s.headers['Accept']='application/vnd.qiwi.sso-v1+json'
    s.headers['content-type'] = 'application/json'
    с = s.get('https://qiwi.com/sinap/providers/'+prv_id+'/form/proxy.action')
    return json.loads(с.text)['data']['body']['content']['terms']['commission']
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/providers/<a>id</a>/form</span></h3></li>
        <ul>
        <strong>В pathname GET-запроса используется параметр:</strong>
             <li><strong>id</strong> - идентификатор провайдера. Возможные значения:
             <ul><li>99 - Перевод на QIWI Wallet</li>
             <li>1963 - Перевод на карту Visa (карты российских банков)</li>
             <li>21013 - Перевод на карту MasterCard (карты российских банков)</li>
             <li>Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:<ul><li>1960 – Перевод на карту Visa</li><li>21012 – Перевод на карту MasterCard</li></ul></li>
             <li>31652 - Перевод на карту национальной платежной системы МИР</li>
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

{
  "content": {
      "terms": {
         "commission": {
                "ranges": [{
                     "bound": 0,
                     "fixed": 50.0,
                     "rate": 0.02
                },
                "limits": [{
                     "currency": "643",
                     "min": 10,
                     "max": 15000
                }]
            }
         }
     }
}     
~~~

~~~python
get_prv_commission('466') # Альфа-Банк
get_prv_commission('1') # МТС
~~~

Успешный ответ содержит JSON-фрагмент с данными о ставках комиссии и ограничениях на сумму платежа (с учетом комиссии) для провайдера:

Параметр | Тип | Описание
-----|-----|------
commission | Object | Объект, содержащий данные об условиях комиссий.
commission.ranges | Array[Object] | Массив объектов с граничными условиями комиссий. Каждый объект содержит параметры:
ranges[].bound |  Number | сумма платежа, начиная с которой применяется условие
ranges[].rate | Number |  комиссия (абс.множитель)
ranges[].min | Number | минимальная сумма комиссии
ranges[].max | Number | максимальная сумма комиссии
ranges[].fixed | Number | фиксированная сумма комиссии
commission.limits | Array[Object] | Массив объектов с ограничениями на полную сумму платежа (вместе с комиссией). Каждый объект содержит параметры:
limits[].currency |  Number | валюта, в которой выражены ограничения (рубли `643`)
limits[].min | Number | минимальная сумма платежа
limits[].max | Number | максимальная сумма платежа


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

~~~python
import requests
import json

# Сложные комиссии
def get_online_comission(api_access_token,to_qw,prv_id,sum_pay):
    s = requests.Session()
    s.headers = {'content-type': 'application/json'}
    s.headers['authorization'] = 'Bearer ' + api_access_token  
    postjson = json.loads('{"account":"","paymentMethod":{"type":"Account","accountId":"643"},"purchaseTotals":{"total":{"amount":"","currency":"643"}}}')
    postjson['account']=to_qw
    postjson['purchaseTotals']['total']['amount']=sum_pay
    c_online = s.post('https://edge.qiwi.com/sinap/providers/'+prv_id+'/onlineCommission',json=postjson)
    return json.loads(c_online.text)
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
             <li>31652 - Перевод на карту национальной платежной системы МИР</li>
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
             <li>Authorization: Bearer *** </li>
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

# Сложные комиссии
get_online_comission(api_access_token,'+380000000000','99',5000)
get_online_comission(api_access_token,'4890xxxxxxxx1698','22351',1000)
get_online_comission(api_access_token,'42767xxxxxxxx268','1963',200)
~~~

<h3 class="request">Ответ ←</h3>

В параметре JSON-ответа `qwCommission.amount` возвращается рассчитанная сумма комиссии.

<a name="payform"></a>

## Автозаполнение платежных форм

[Пример ссылки (нажмите для перехода на форму)](https://qiwi.com/payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&amountFraction=0&extra%5B%27comment%27%5D=test123&currency=643&blocked[0]=account)

Данный запрос отображает в браузере заполненную форму на сайте qiwi.com для совершения платежа.

~~~shell
https://qiwi.com/payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&amountFraction=0&extra%5B%27comment%27%5D=test123&currency=643
~~~

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>В пути ссылки и в строке запроса указываются параметры платежной формы.</span></li>
</ul>

Параметр|Тип|Описание|Поле на форме| Обяз.
---------|--------|---|----
ID | Integer | Идентификатор провайдера (указывается в пути ссылки).<br>Возможные значения:<br>`99` - [Перевод на QIWI Wallet](#p2p)<br>`1963` - [Перевод на карту Visa](#cards) (карты российских банков)<br>`21013` - [Перевод на карту MasterCard](#cards) (карты российских банков)<br>Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:<br>`1960` – [Перевод на карту Visa](#cards)<br>`21012` – [Перевод на карту MasterCard](#cards)<br>`31652` - [Перевод на карту МИР](#cards)<br>[Идентификаторы операторов мобильной связи](#mnp)<br>[Идентификаторы других провайдеров](#charity) | - | +
amountInteger|Integer | Целая часть суммы платежа (рубли). Указывается в строке запроса. Если параметр не указан, поле "Сумма" на форме будет пустым. **Допустимо число не больше 99 999 (ограничение на сумму платежа)** | Сумма | -
amountFraction|Integer | Дробная часть суммы платежа (копейки). Указывается в строке запроса. Если параметр не указан, поле "Сумма" на форме будет пустым.|Сумма | -
currency|Константа, `643` | Код валюты платежа. Указывается в строке запроса. **Обязательный параметр, если вы передаете в ссылке сумму платежа** |-|+
extra['comment'] |URL-encoded string | Комментарий. Указывается в строке запроса. Имя параметра должно быть URL-закодировано. **Параметр используется только для ID=99** |Комментарий к переводу | -
extra['account'] |URL-encoded string |  Формат совпадает с форматом параметра `fields.account` в соответствующем платежном запросе. Имя параметра должно быть URL-закодировано.|Номер Кошелька, Номер телефона/счета/карты получателя.|-
blocked|Array[String]|Признак неактивного поля формы. Пользователь не сможет менять значение данного поля. Каждое поле задается соответствующим именем параметра и нумеруется элементом массива, начиная с нуля (`blocked[0]`, `blocked[1]` и т.д.). Если параметр не указан, пользователь сможет изменить все поля формы. Допустимые значения:<br>`sum` - поле "сумма платежа", <br>`account` - поле "номер счета/телефона/карты",<br>`comment` - поле "комментарий". Пример (неактивное поле суммы платежа): `blocked[0]=sum` |-|-


## Перевод на QIWI Кошелек {#p2p}

<h3 class="request method">Запрос → POST</h3>

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

~~~python
import requests
import json
import time

# Перевод на QIWI Кошелек
def send_p2p(my_login,api_access_token,to_qw,comment,sum_p2p):
    s = requests.Session()
    s.headers = {'content-type': 'application/json'}
    s.headers['authorization'] = 'Bearer ' + api_access_token
    s.headers['User-Agent'] = 'Android v3.2.0 MKT'
    s.headers['Accept']= 'application/json'
    postjson = json.loads('{"id":"","sum":{"amount":"","currency":""},"paymentMethod":{"type":"Account","accountId":"643"},"comment":"'+comment+'","fields":{"account":""}}')
    postjson['id']=str(int(time.time() * 1000))
    postjson['sum']['amount']=sum_p2p
    postjson['sum']['currency']='643'
    postjson['fields']['account']=to_qw
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/99/payments',json=postjson)
    print(res)
    return json.loads(res.text)
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/99/payments</span></h3></li>
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
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON.</span>
    </li>
</ul>

Параметр|Тип|Описание|Обяз.
--------|----|----|------
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).|+
sum|Object| Данные о сумме платежа:
sum.amount|Number|Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.|+
sum.currency|String|Валюта (только `643`, рубли)|+
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`|+
paymentMethod.accountId|String| Константа, `643`|+
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String|Номер телефона получателя (с международным префиксом)|+
comment|String|Комментарий к платежу|-

<h3 class="request">Ответ ←</h3>

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
send_p2p(mylogin,api_access_token,'+79261112233','comment',99.01)

<Response [200]>
{'comment': 'comment',
 'fields': {'account': '+79261112233'},
 'id': '1514296828893',
 'source': 'account_643',
 'sum': {'amount': 99.01, 'currency': '643'},
 'terms': '99',
 'transaction': {'id': '11982501857', 'state': {'code': 'Accepted'}}}
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

## Конвертация {#CCY}

Метод выполняет перевод средств на валютный счет QIWI Кошелька с конвертацией с рублевого счета.

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST 'https://edge.qiwi.com/sinap/api/v2/terms/99/payments'
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
  -d '{
        "id":"11111111111111",
        "sum": {
          "amount":100,
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
				"amount":100.00,
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

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/99/payments</span></h3></li>
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
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON.</span>
    </li>
</ul>

Параметр|Тип|Описание|Обяз.
--------|----|----|------
id | String |Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).|+
sum|Object| Данные о сумме платежа:
sum.amount|Number|Сумма в валюте. Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.|+
sum.currency|String|Код валюты (number-3 ISO-4217)|+
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`|+
paymentMethod.accountId|String| Константа, `643`|+
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String|Номер кошелька на котором происходит конвертация|+
comment|String|Комментарий к платежу|-

<h3 class="request">Ответ ←</h3>

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

## Курс валют

Метод возвращает текущие курсы и кросс-курсы валют КИВИ Банка.

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/sinap/crossRates"
  --header "Accept: application/json"
  --header "Content-Type: application/json"
  --header "Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9"
~~~

~~~http
GET /sinap/crossRates HTTP/1.1
Accept: application/json
Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9
Content-type: application/json
Host: edge.qiwi.com
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/crossRates</span></h3></li>
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

<h3 class="request">Ответ ←</h3>

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

Успешный JSON-ответ содержит массив данных о курсах валют:

Параметр|Тип|Описание
--------|----|----
from|String|Валюта покупки
to|String|Валюта продажи
rate|Number|Курс


## Оплата сотовой связи {#cell}

<h3 class="request method">Запрос → POST</h3>

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

~~~python
import requests
import json

# Оплата мобильного телефона
def send_mobile(api_access_token,prv_id,to_account,comment,sum_pay):
    s = requests.Session()
    s.headers['Accept']= 'application/json'
    s.headers['Content-Type']= 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    postjson = {"id":"","sum": {"amount":"","currency":"643"},"paymentMethod": {"type":"Account","accountId":"643"},"comment":"","fields": {"account":""}}
    postjson['id']=str(int(time.time() * 1000))
    postjson['sum']['amount']=sum_pay
    postjson['fields']['account']=to_account
    postjson['comment']=comment
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/'+prv_id+'/payments', json=postjson)
    print(res)
    return json.loads(res.text)
~~~

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
             <li>Authorization: Bearer *** </li>
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

<h3 class="request method" id="mnp">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://qiwi.com/mobile/detect.action"
  --header "Accept: application/json"
  --header "Content-Type: application/x-www-form-urlencoded"
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
import json

def mobile_operator(phone_number):
    s = requests.Session()
    res = s.post('https://qiwi.com/mobile/detect.action', data = {'phone': phone_number })
    s.headers['Accept'] = 'application/json'
    s.headers['Content-Type'] = 'application/x-www-form-urlencoded'
    print(res)
    return json.loads(res.text)['message']
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
    <li><h3>Параметры</h3><span>Параметр передается в теле запроса как formdata.</span>
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
mobile_operator(79652468447)
~~~

Ответ с HTTP Status 200 и параметром `code.value` = 0 является признаком успешной проверки. Идентификатор оператора находится в параметре `message`.

Ответ с HTTP Status 200 и параметром `code.value` = 2 означает, что невозможно определить оператора.

## Перевод на карту {#cards}

Запрос выполняет перевод на карты платежных систем Visa или MasterCard. Предварительно можно [проверить тип платежной системы](#card_check).

<h3 class="request method">Запрос → POST</h3>

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

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/<a>ID</a>/payments</span></h3></li>
        <ul>
        <strong>В pathname POST-запроса используется параметр:</strong>
             <li><strong>ID</strong> - идентификатор провайдера. Возможные значения:
             <ul>
             <li>1963 - Перевод на карту Visa (карты российских банков)</li>
             <li>21013 - Перевод на карту MasterCard (карты российских банков)</li>
             <li>22351 - Перевод на <a href="https://qiwi.com/qvc/help.action">Виртуальную карту QIWI</a></li>
             <li>Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:
             <ul><li>1960 – Перевод на карту Visa</li>
             <li>21012 – Перевод на карту MasterCard</li></ul></li>
             <li>31652 - Перевод на карту национальной платежной системы МИР</li>
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

<h3 class="request method" id="card_check">Запрос → POST</h3>

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

~~~python
import requests
import json

def card_system(card_number):
    s = requests.Session()
    res = s.post('https://qiwi.com/card/detect.action', data = {'cardNumber': card_number })
    print(res)
    print(res.text)
    return json.loads(res.text)['message']
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
cardNumber | String |Немаскированный номер карты. Обязательный параметр

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
card_system(4890xxxxxxxx1698)
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

Ответ с HTTP Status 200 и параметром `code.value` = 0 является признаком успешной проверки. Идентификатор платежной системы находится в параметре `message`.

Ответ с HTTP Status 200 и параметром `code.value` = 2 означает, что в номере карты ошибка.

## Банковский перевод {#banks}

Запрос выполняет перевод на банковские карты/счета российских банков.

<h3 class="request method">Запрос → POST</h3>

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
             <li>Authorization: Bearer *** </li>
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

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

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


## Перевод по банковским реквизитам {#banks_wire}

Запрос выполняет перевод на счета российских банков по полным реквизитам счета. Возможен обычный перевод или перевод через систему БЭСП (ускоренная обработка платежа, не более 1 операционного дня).

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/382/payments"
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
          "account_type": "2",
          "urgent": "0",
          "lname": "Иванов",
          "fname": "Иван",
          "mname": "Иванович",
          "mfo": "046577795",
          "account":"40817***"
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
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/<a>ID</a>/payments</span></h3></li>
        <ul>
        <strong>В pathname POST-запроса используется параметр:</strong>
             <li><strong>ID</strong> - идентификатор провайдера. Возможные значения:
             <ul><li>466 - Тинькофф Банк</li>
             <li>464 - Альфа-Банк</li>
             <li>821 - Промсвязьбанк</li>
             <li>804 - АО "ОТП БАНК"</li>
             <li>810 - АО "РОССЕЛЬХОЗБАНК"</li>
             <li>816 - ВТБ 24 (ПАО)</li>
             <li>819 - АО ЮНИКРЕДИТ БАНК</li>
             <li>868 - КИВИ БАНК (АО)</li>
             <li>870 - ПАО Сбербанк</li>
             <li>1134 - ПАО "МОСКОВСКИЙ КРЕДИТНЫЙ БАНК"</li>
             <li>27324 - АО "РАЙФФАЙЗЕНБАНК"</li>
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
fields.account| String| Номер банковского счета получателя
fields.mfo| String|БИК соответствующего банка/территориального отделения банка
fields.urgent| String|Признак ускоренного перевода по системе БЭСП. Значение `0` - не использовать; значение `1` - выполнить перевод через систему БЭСП (в течение операционного дня). **Внимание! Взимается дополнительная комиссия за перевод в системе БЭСП**
fields.account_type| String|Тип банковского идентификатора. Для каждого банка применяется собственное значение:<br>Тинькофф Банк - `3`<br>Альфа-Банк - `2`<br>ОТП банк - `2`<br>Россельхозбанк - `2`<br>ВТБ 24 - `2`<br>Промсвязьбанк - `9`<br>ЮНИКРЕДИТ БАНК - `2`<br>КИВИ БАНК - `2`<br>Сбербанк - `2`<br>МОСКОВСКИЙ КРЕДИТНЫЙ БАНК - `2`<br>РАЙФФАЙЗЕНБАНК -`2`.
fields.lname|String|Фамилия получателя
fields.fname|String|Имя получателя
fields.mname|String|Отчество получателя

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "21131343",
  "terms": "466",
  "fields": {
          "account": "407121010910909011",
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
fields|Object|Копия объекта `fields` из исходного запроса
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге QIWI Wallet. Параметры:
transaction.id|String|ID транзакции в процессинге QIWI Wallet
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге QIWI Wallet. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

## Оплата других услуг {#charity}

Оплата услуги по идентификатору пользователя. Данный запрос применяется для провайдеров, использующих в реквизитах единственный пользовательский идентификатор, без проверки номера аккаунта.

<h3 class="request method">Запрос → POST</h3>

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

~~~python
import requests
import json
import time

# оплата простого провайдера

def pay_simple_prv(api_access_token,prv_id,to_account,sum_pay):
    s = requests.Session()
    s.headers['Accept']= 'application/json'
    s.headers['Content-Type']= 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    postjson = {"id":"","sum": {"amount":"","currency":"643"},"paymentMethod": {"type":"Account","accountId":"643"},"fields": {"account":""}}
    postjson['id']=str(int(time.time() * 1000))
    postjson['sum']['amount']=sum_pay
    postjson['fields']['account']=to_account
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/'+prv_id+'/payments', json=postjson)
    print(res)
    return json.loads(res.text)
~~~

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
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON. Все параметры обязательны.</span>
    </li>
</ul>

<aside class="notice">Поиск идентификатора выполняется на сайте qiwi.com в поисковой строке. Идентификатор находится в URL ссылки на платежную форму провайдера вида <a>https://qiwi.com/payment/form.action?provider=ID</a> или <a>https://qiwi.com/payment/form/ID</a></aside>

![Поиск провайдера](/images/provider_id.jpg)

~~~python
import requests
import json

# поиск на qiwi.com - опредление id провайдера по названию
def qiwi_com_search(search_phrase):
    s = requests.Session()
    search = s.post('https://qiwi.com/search/results/json.action', params={'searchPhrase':search_phrase})
    print(search)
    return json.loads(search.text)['data']['items']

qiwi_com_search('Билайн домашний интернет')[0]['item']['id']['id']
~~~

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

~~~python
# zenit
pay_simple_prv(api_access_token,'26386','2166191','10')

<Response [200]>
{'fields': {'account': '2166191'},
 'id': '1509031806148',
 'source': 'account_643',
 'sum': {'amount': 10, 'currency': '643'},
 'terms': '26386',
 'transaction': {'id': '11585129951', 'state': {'code': 'Accepted'}}}
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

<h3 class="request method">Запрос → POST</h3>

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

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/1717/payments</span></h3></li>
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

# Уведомления (хуки) {#webhook}

> Исходящие платежи - платеж в проведении

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

> Исходящие платежи - успешный платеж

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

> Исходящие платежи - неуспешный платеж

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

> Входящие платежи - успешный платеж

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


Хуки или уведомления отправляются на сервер клиента с данными о событии (платеже/пополнении). В настоящее время поддерживаются только веб-хуки (webhook) - сообщения, адресованные веб-сервисам. Для приема веб-хуков пользователю-владельцу токена доступа к кошельку необходимо настроить свой сервер на прием и обработку POST-запросов ([Формат запросов](#hook_format)).

**От вашего сервера успешный ответ 200 OK на входящий запрос должен поступить в течение 1-2 сек. Не дождавшись ответа, сервис КИВИ отправляет еще одно уведомление через 10 минут, потом еще одно через 1 час.**

Сервера, с которых сервисы QIWI отправляют webhook:

* 91.232.231.36
* 91.232.231.35

Если ваш сервер обработки веб-хуков работает за брандмауэром, необходимо добавить эти IP-адреса в список разрешенных адресов входящих TCP-пакетов.

## Быстрый старт {#quick_hook}

0. Реализуйте веб-сервис обработки [запросов](#hook_format). Особое внимание обратите на реализацию проверки подписи.
1. Зарегистрируйте свой обработчик [PUT-запросом](#hook_reg). **Внимание! Длина адреса сервиса обработчика не должна превышать 100 символов.**
2. Запросите [ключ проверки подписи хука](#hook_key).
3. Протестируйте прием запросов вашим обработчиком с помощью [тестового запроса](#hook_test). На зарегистрированный PUT-запросом сервис придет пустой веб-хук.


## Формат webhook {#hook_format}

Каждый веб-хук - POST-запрос с JSON-объектом, содержащий данные об одном платеже. Схема объекта:


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

//Функция возвращает упорядоченную строку значений параметров webhook и хэш подписи webhook для проверки
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

    // Строка параметров
    $reqparams = $decoded['payment']['sum']['currency'] . '|' . $decoded['payment']['sum']['amount'] . '|'. $decoded['payment']['type'] . '|' . $decoded['payment']['account'] . '|' . $decoded['payment']['txnId'];
    // Подпись из запроса
    foreach ($decoded as $name=>$value) {
       if ($name == 'hash') {
            $SIGN_REQ = $value;
       }
    }

    return [$reqparams, $SIGN_REQ];
}

// Список параметров и подпись

$Request = getReqParams();

// Base64 encoded ключ для уведомлений webhook (метод /hook/{hookId}/key)

$NOTIFY_PWD = "JcyVhjHCvHQwufz+IHXolyqHgEc5MoayBfParl6Guoc=";

// Вычисляем хэш SHA-256 строки параметров и шифруем с ключом для уведомлений

$reqres = hash_hmac("sha256", $Request[0], base64_decode($NOTIFY_PWD));

// Проверка подписи запроса

if (hash_equals($reqres, $Request[1])) {
    $error = array('response' => 'OK');
}
else $error = array('response' => 'error');

//Ответ

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

# Base64 encoded ключ для уведомлений webhook (/hook/{hookId}/key)
webhook_key_base64 = 'JcyVhjHCvHQwufz+IHXolyqHgEc5MoayBfParl6Guoc='

# строка параметров
data = '643|1|IN|+79165238345|13353941550'

webhook_key = base64.b64decode(bytes(webhook_key_base64,'utf-8'))
print(hmac.new(webhook_key, data.encode('utf-8'), hashlib.sha256).hexdigest())
~~~


Поле | Тип | Описание
----|------|-------
hookId | String (UUID) | Уникальный id хука
messageId | String (UUID) | Уникальный id отправленного коллбэка
payment | Object | Данные платежа
payment.txnId | String | ID транзакции в процессинге QIWI Wallet
payment.account | String | Для платежей - номер счета получателя. Для пополнений - номер отправителя, терминала или название агента пополнения кошелька
payment.signFields | String | Список полей объекта `payments` (через `,`), которые хешируются алгоритмом HmacSHA256 для проверки хука (см. параметр `hash`)
payment.personId | Integer | Номер кошелька
payment.date | String DateTime | Дата/время платежа, в московской временной зоне. Формат даты `ГГГГ-ММ-ДД'T'чч:мм:сс+03:00`
payment.errorCode | String | Код ошибки платежа
payment.type | String | Тип платежа. Возможные значения:<br>`IN` - пополнение, <br>`OUT` - платеж
payment.status|String|Статус платежа. Возможные значения:<br>`WAITING` - платеж проводится,<br>`SUCCESS` - успешный платеж,<br>`ERROR` - ошибка платежа.
payment.provider | Integer| ID провайдера QIWI Wallet
payment.comment | String | Комментарий к транзакции
payment.sum | Object | Данные о сумме платежа или пополнения. Параметры:
sum.amount|Number(Decimal)|Сумма
sum.currency|Integer|Код валюты
payment.commission|Object| Данные о комиссии для платежа или пополнения. Параметры:
commission.amount|Number(Decimal)|Сумма
commission.currency|Integer|Код валюты
payment.total|Object|Данные об итоговой сумме платежа или пополнения. Параметры:
total.amount|Number(Decimal)|Сумма
total.currency|Integer|Код валюты
test|Boolean|Признак тестового сообщения
version|String|Версия API
hash|String| Хэш цифровой подписи веб-хука. Как проверить хэш: берутся значения полей из списка payment.signFields (**в том же порядке**) в формате String, конкатенируются с разделителем `|` и шифруются алгоритмом SHA-256 с [ключом проверки подписи](#hook_key). Полученное значение сравнивается с тем, что пришло в поле `hash`.

Пример расшифровки подписи (см. также функцию PHP на вкладке справа):

1. По запросу пользователь получает свой ключ, закодированный в Base64: `JcyVhjHCvHQwufz+IHXolyqHgEc5MoayBfParl6Guoc=`
2. Приходит веб-хук `{"messageId":"7814c49d-2d29-4b14-b2dc-36b377c76156","hookId":"5e2027d1-f5f3-4ad1-b409-058b8b8a8c22","payment":{"txnId":"13353941550","date":"2018-06-27T13:39:00+03:00","type":"IN","status":"SUCCESS","errorCode":"0","personId":78000008000,"account":"+79165238345","comment":"","provider":7,"sum":{"amount":1,"currency":643},"commission":{"amount":0,"currency":643},"total":{"amount":1,"currency":643},"signFields":"sum.currency,sum.amount,type,account,txnId"},"hash":"76687ffe5c516c793faa46fafba0994e7ca7a6d735966e0e0c0b65eaa43bdca0","version":"1.0.0","test":false}`
3. Конкатенируются требуемые поля платежных данных: `643|1|IN|+79165238345|13353941550`
4. Поля шифруются методом SHA-256 с раскодированным ключом из п.1. Результат `76687ffe5c516c793faa46fafba0994e7ca7a6d735966e0e0c0b65eaa43bdca0` совпадает с параметром `hash` из запроса.

## Регистрация обработчика webhook {#hook_reg}

<h3 class="request method">Запрос → PUT</h3>

~~~shell
curl -X PUT "https://edge.qiwi.com/payment-notifier/v1/hooks?hookType=1&param=http%3A%2F%2Fecho.fjfalcon.ru%2F&txnType=2"
     -H "accept: */*"
     -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
PUT /payment-notifier/v1/hooks?hookType=1&param=http%3A%2F%2Fecho.fjfalcon.ru%2F&txnType=2 HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL</h3><span>https://edge.qiwi.com/payment-notifier/v1/hooks</span></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в query запроса. Все параметры обязательны.</span>
    </li>
</ul>

Название|Тип|Описание
----|-----|------
hookType|Integer|Тип хука. Только 1 - веб-хук.
param|URL-encoded|Адрес сервера обработки веб-хуков. **Внимание! Длина исходного (не URL-encoded) адреса сервиса обработчика не должна превышать 100 символов.**
txnType|String|Тип транзакций, по которым будут включены уведомления. Возможные значения:<br>0 - только входящие транзакции (пополнения)<br>1 - только исходящие транзакции (платежи)<br>2 - все транзакции

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{"hookId":"d63a8729-f5c8-486f-907d-9fb8758afcfc","hookParameters":{"url":"http://echo.fjfalcon.ru/"},"hookType":"WEB","txnType":"BOTH"}
~~~

Ответ в формате JSON

Название|Тип|Описание
-----|------|------
hookId|String|UUID созданного веб-хука
hookParameters|Object|Набор параметров веб-хука (только URL)
hookType|String|Тип веб-хука (только WEB)
txnType|String|Тип транзакций, по которым отсылаются веб-хуки (`IN` - входящие, `OUT` - исходящие, `BOTH` - все)

## Удаление webhook  {#hook_remove}

<h3 class="request method">Запрос → DELETE</h3>

~~~shell
curl -X DELETE "https://edge.qiwi.com/payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc" -H "accept: */*" -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
DELETE /payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
        <li><h3>URL </h3><span>https://edge.qiwi.com/payment-notifier/v1/hooks/{hookId}</span></li>
        <ul>
        <strong>В pathname запроса используется параметр:</strong>
             <li><strong>hookId</strong> - UUID веб-хука</li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>


<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{"response":"Hook deleted"}
~~~

Формат ответа JSON.

Название|Тип|Описание
-----|------|------
response|String|Описание результата операции

## Получение секретного ключа {#hook_key}

Каждый веб-хук содержит цифровую подпись сообщения, зашифрованную ключом. Для получения ключа проверки подписи используйте данный запрос.

<h3 class="request method">Запрос → GET</h3>


~~~shell
curl -X GET "https://edge.qiwi.com/payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/key" -H "accept: */*" -H "accept: */*" -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
GET /payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/key HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~


<ul class="nestedList url">
    <li><h3>URL </h3><span>https://edge.qiwi.com/payment-notifier/v1/hooks/{hookId}/key</span></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметр передается в pathname запроса.</span></li>
        <ul>
        <strong>В pathname запроса используется параметр:</strong>
             <li><strong>hookId</strong> - UUID веб-хука</li>
        </ul>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 201 Created
Content-Type: application/json

{"key":"L8UVF3JkLVUr6r70LiE0A9/5WoGGwWKG2pI/e+l/9fs="}
~~~

Формат ответа JSON.

Название|Тип|Описание
-----|------|------
key|String|Base64-закодированный ключ

## Изменение секретного ключа  {#hook_secret}

Для смены ключа шифрования сообщений веб-хука используйте данный запрос.

<h3 class="request method">Запрос → POST</h3>


~~~shell
curl -X POST "https://edge.qiwi.com/payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/newkey" -H "accept: */*" -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
POST /payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/newkey HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~


<ul class="nestedList url">
    <li><h3>URL </h3><span>https://edge.qiwi.com/payment-notifier/v1/hooks/{hookId}/newkey</span></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметр передается в pathname запроса.</span></li>
        <ul>
        <strong>В pathname запроса используется параметр:</strong>
             <li><strong>hookId</strong> - UUID веб-хука</li>
        </ul>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 201 Created
Content-Type: application/json

{"key":"OikS4/CcIbSf+yYGnLbnOige8RGoYmGxs/LNMwkJy7Q="}
~~~

Формат ответа JSON.

Название|Тип|Описание
-----|------|------
key|String|Base64-закодированный новый ключ

## Данные об активном веб-хуке   {#hook_active}

Список действующих (активных) веб-хуков, связанных с токеном (номером кошелька), можно получить данным запросом.

Так как сейчас используется только один тип хуков - веб-хуки, то в ответе содержится только один объект данных.

<h3 class="request method">Запрос → GET</h3>

~~~shell
curl -X GET "https://edge.qiwi.com/payment-notifier/v1/hooks/active" -H "accept: */*" -H "accept: */*" -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
GET /payment-notifier/v1/hooks/active HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL </h3><span>https://edge.qiwi.com/payment-notifier/v1/hooks/active</span></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры запроса отсутствуют.</span></li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{"hookId":"d63a8729-f5c8-486f-907d-9fb8758afcfc","hookParameters":{"url":"http://echo.fjfalcon.ru/"},"hookType":"WEB","txnType":"BOTH"}
~~~

Формат ответа JSON.

Название|Тип|Описание
-----|------|------
hookId|String|UUID активного веб-хука
hookParameters|Object|Набор параметров веб-хука (только URL)
hookType|String|Тип веб-хука (только WEB)
txnType|String|Тип транзакций, по которым отсылаются уведомления (`IN` - входящие, `OUT` - исходящие, `BOTH` - все)

## Отправка тестового веб-хука  {#hook_test}

Для проверки вашего обработчика сообщений веб-хуков используйте данный запрос. Запрос отправляется на адрес, указанный в активном веб-хуке.

<h3 class="request method">Запрос → GET</h3>


~~~shell
curl -X GET "https://edge.qiwi.com/payment-notifier/v1/hooks/test" -H "accept: */*" -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
POST /payment-notifier/v1/hooks/test HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL </h3><span>https://edge.qiwi.com/payment-notifier/v1/hooks/test</span></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры запроса отсутствуют.</span></li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{"response":"Webhook sent"}
~~~

Формат ответа JSON.

Название|Тип|Описание
-----|------|------
response|String|Результат запроса


# Выставление счета {#invoice}

Для выставления счета на QIWI Кошелек используется протокол [API QIWI Кассы](https://developer.qiwi.com/ru/bill-payments/). Для получения ключей авторизации запросов вы можете зайти на [p2p.qiwi.com](https://p2p.qiwi.com) в личный кабинет, или использовать представленный запрос.

Данный запрос возвращает пару ключей (PublicKey и SecretKey) и настраивает URL уведомлений (если нужно).


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

Для заголовка Authorization используется токен Open API.

<ul class="nestedList url">
<li><h3>URL</h3> <span>https://edge.qiwi.com/widgets-api/api/p2p/protected/keys/create</span></li>
</ul>

Параметр|Тип|Описание
--------|----|----
keysPairName| String| Название пары ключей
serverNotificationsUrl|String |URL для уведомлений об оплате счетов


# Оплата счета {#pay_invoice}

## Список счетов  {#list_invoice}

Получение списка неоплаченных счетов кошелька пользователя.

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl -X GET --header 'Accept: application/json' --header 'Authorization: Bearer ***' 'https://edge.qiwi.com/checkout/api/bill/search?statuses=READY_FOR_PAY&rows=50'
~~~

~~~http
GET /checkout/api/bill/search?statuses=READY_FOR_PAY&rows=50 HTTP/1.1
Accept: application/json
Authorization: Bearer ***
Host: edge.qiwi.com
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/checkout/api/bill/search?statuses=READY_FOR_PAY&rows=50</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
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
rows | Integer |Максимальное число счетов в ответе, для разбивки списка на части. Целое число от 1 до 50. По умолчанию возвращается не более 50 счетов.
statuses|String| Статус неоплаченного счета. Строка `READY_FOR_PAY`
min_creation_datetime|Long|Нижняя временная граница для поиска счетов, Unix-time
max_creation_datetime|Long|Верхняя временная граница для поиска счетов, Unix-time
next_id|Number|Начальный идентификатор для поиска, чтобы возврат списка выполнялся начиная с этого значения
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
      "pay_url":"https://bill.qiwi.com/order/external/form.action?from=480706&to=79262468447&order=1063702405&billref=site"
    }
  ]
}
~~~

Успешный JSON-ответ содержит список неоплаченных счетов кошелька, соответствующих заданному фильтру:

<a name="invoice_data"></a>

Параметр|Тип|Описание
--------|----|----
bills|Array[Object]|Массив платежей. <br>Число платежей равно параметру `rows` из запроса, или максимально 50, если параметр не указан
bills[].id|Integer|Идентификатор счета в QIWI Кошельке
bills[].external_id|String| Идентификатор счета у мерчанта
bills[].creation_datetime|Long|Дата/время создания счета, Unix-time
bills[].expiration_datetime|Long|Дата/время окончания срока действия счета, Unix-time
bills[].sum|Object|Сведения о сумме счета
sum.currency|Integer|Валюта суммы счета
sum.amount|Number|Сумма счета
bills[].status|String | Константа, READY_FOR_PAY
bills[].type|String|Константа, MERCHANT
bills[].repetitive|Boolean|Служебное поле
bills[].provider|Object|Информация о мерчанте
provider.id|Integer|Идентификатор мерчанта в QIWI
provider.short_name|String|Сокращенное название мерчанта
provider.long_name|String|Полное название мерчанта
provider.logo_url|String|Ссылка на логотип мерчанта
bills[].comment|String|Комментарий к счету
bills[].pay_url|String|Ссылка для оплаты счета в интерфейсе QIWI

## Оплата счета  {#paywallet_invoice}

Выполнение оплаты счета без подтверждения.

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST --header 'Content-Type: application/json;charset=UTF-8' --header 'Accept: application/json' --header 'Authorization: Bearer 68ec21fd52e4244838946dd07ed225a1' -d '{ \
   "invoice_uid": "1063702405", \
   "currency": "643" \
 }' 'https://edge.qiwi.com/checkout/invoice/pay/wallet'
~~~

~~~http
POST /checkout/invoice/pay/wallet HTTP/1.1
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
    <li><h3>URL <span>https://edge.qiwi.com/checkout/invoice/pay/wallet</span></h3></li>
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
invoice_uid | String |ID счета в QIWI (параметр bills[].id в [данных о счете](#invoice_data)
currency|String| Валюта суммы счета (параметр bills[].sum.currency в [данных о счете](#invoice_data))


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
invoice_status|String|Строка кода статуса оплаты счета, PAID_STATUS. Любой другой статус означает неуспех платежной транзакции.
is_sms_confirm|String|Признак подтверждения по SMS

## Отмена неоплаченного счета  {#cancel_invoice}

Метод позволяет отклонить неоплаченный счет. При этом счет становится недоступным для оплаты.


<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST --header 'Accept: application/json' --header 'Authorization: Bearer ***' 'https://edge.qiwi.com/checkout/api/bill/reject' -d '{ "id": 1034353453 }'
~~~

~~~http
POST /checkout/api/bill/reject HTTP/1.1
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
    <li><h3>URL <span>https://edge.qiwi.com/checkout/api/bill/reject</span></h3></li>
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
    <li><h3>Параметры</h3><span>Данный обязательный параметр передается в теле запроса в формате JSON:</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
id | Integer |ID счета для отмены

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

Успешный ответ содержит HTTP-код 200.


# Коды ошибок {#errors}

В случае ошибки API возвращается HTTP-код ошибки.

HTTP Код | Секция API | Описание
---|-----|---------
400 | Все | Ошибка синтаксиса запроса (неправильный формат данных)
401 | Все | Неверный токен или истек срок действия токена
403 | Все | Нет прав на данный запрос (недостаточно разрешений у токена)
404 | История платежей, Информация о транзакции, Отправка квитанции | Не найдена транзакция или отсутствуют платежи с указанными признаками
404 | Балансы, Профиль пользователя, Идентификация пользователя | Не найден кошелек
404 | Веб-хуки | Не найден активный веб-хук
404 | Оплата/Отмена счета | Не найден счет
422 | Регистрация веб-хука | Неправильно указаны домен/подсеть/хост веб-хука(в параметре URL), неправильно указаны тип хука или тип транзакции, попытка создать хук при наличии уже созданного
423 | История платежей | Слишком много запросов, сервис временно недоступен
500 | Веб-хуки | Внутренняя ошибка сервиса (превышена длина URL)

Следующие ошибки возвращаются на запросы [истории платежей](#payments_history) и [информации о транзакции](#txn_info) в параметре `errorCode` ответа:

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
