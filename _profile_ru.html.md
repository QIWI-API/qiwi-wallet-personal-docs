
# Профиль пользователя {#profile}

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/person-profile/v1/profile/current?authInfoEnabled=false" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
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

# Профиль пользователя
def get_profile(api_access_token):
    s7 = requests.Session()
    s7.headers['Accept']= 'application/json'
    s7.headers['authorization'] = 'Bearer ' + api_access_token
    p = s7.get('https://edge.qiwi.com/person-profile/v1/profile/current?authInfoEnabled=true&contractInfoEnabled=true&userInfoEnabled=true')
    return p.json()
~~~

~~~python
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# Полная информация о профиле пользователя
profile = get_profile(api_access_token)

# Профиль пользователя
# статус блокировки
profile['contractInfo']['blocked']

# Профиль пользователя
# уровень идентификации в Киви Банке
profile['contractInfo']['identificationInfo'][0]['identificationLevel']

# привязанный email
profile['authInfo']['boundEmail']
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
contractInfo.identificationInfo|Array[Object]|Данные об [идентификации](https://qiwi.com/settings/identification#ru) пользователя.
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
user@server:~$ curl -X POST "https://edge.qiwi.com/identification/v1/persons/79111234567/identification" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9" \
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

# идентификация
def get_identification(api_access_token, my_login):
    s = requests.Session()
    s.headers['authorization'] = 'Bearer ' + api_access_token
    res = s.get('https://edge.qiwi.com/identification/v1/persons/'+my_login+'/identification')
    return res.json()
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
print(get_identification(api_access_token, mylogin))

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

[Подробнее об идентификации](https://qiwi.com/settings/identification#ru)

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl -X GET "https://edge.qiwi.com/identification/v1/persons/79111234567/identification" \
  --header "Accept: application/json" \
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

