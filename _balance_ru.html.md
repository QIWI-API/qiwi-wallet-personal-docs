
# Баланс QIWI Кошелька {#balance}

###### Последнее обновление: 2020-07-06 | [Предложить правки на GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/_balance_ru.html.md)

Методы данного API предназначены для управления балансами вашего QIWI кошелька.

## Список балансов {#balances_list}

Запрос выгружает текущие балансы счетов вашего QIWI Кошелька.

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/getByAliasUsingGET_1)

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/funding-sources/v2/persons/79115221133/accounts" \
  --header "Accept: application/json" \
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

# Баланс QIWI Кошелька
def balance(login, api_access_token):
    s = requests.Session()
    s.headers['Accept']= 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token  
    b = s.get('https://edge.qiwi.com/funding-sources/v2/persons/' + login + '/accounts')
    return b.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/funding-sources/v2/persons/<a>personId</a>/accounts</span></h3>
        <ul>
             <li><strong>personId</strong> - номер вашего кошелька без знака "+"</li>
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
balances = balance(mylogin,api_access_token)['accounts']

# рублевый баланс
rubAlias = [x for x in balances if x['alias'] == 'qw_wallet_rub']
rubBalance = rubAlias[0]['balance']['amount']
~~~

> Повторный запрос, если в ответе пришел пустой объект balance и поле "hasBalance": true

~~~http
GET /funding-sources/v2/persons/79115221133/accounts?timeout=1000&alias=qw_wallet_rub HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~


Успешный ответ содержит JSON-массив счетов вашего QIWI Кошелька для фондирования платежей и текущие балансы счетов:

Поле ответа|Тип|Описание
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
accounts[].balance|Object |Сведения о балансе данного счета.<br>Если объект пустой и при этом поле `accounts[].hasBalance` равно `true`, повторите запрос с дополнительными параметрами:<br>`timeout=1000` и `alias=accounts[].alias` (псевдоним этого баланса)
balance.amount|Number|Текущий баланс данного счета
balance.currency | Number| Код валюты баланса (number-3 ISO-4217)

## Создание баланса {#balance_create}

Запрос создает новый счет и баланс в вашем QIWI Кошельке. Список доступных для создания счетов можно получить [другим запросом](#funding_offer).

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/createAccountUsingPOST)

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/funding-sources/v2/persons/79115221133/accounts" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
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
    <li><h3>URL <span>/funding-sources/v2/persons/<a>personId</a>/accounts</span></h3>
        <ul>
             <li><strong>personId</strong> - номер вашего кошелька без знака "+"</li  >
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
    <li><h3>Параметры</h3><span>Данный параметр передается в JSON-теле запроса:</span>
    </li>
</ul>


Название|Тип|Описание
--------|----|----
alias|String| Псевдоним нового счета (см. [запрос доступных счетов](#funding_offer))

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 201 Created
Content-Type: application/json
~~~

Успешный ответ содержит HTTP-код 201.

## Запрос доступных счетов {#funding_offer}

Запрос отображает псевдонимы счетов, доступных для создания в вашем QIWI Кошельке.

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/getAccountsOfferUsingGET)

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl -X GET "https://edge.qiwi.com/funding-sources/v2/persons/79115221133/accounts/offer" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu"
~~~

~~~http
GET /funding-sources/v2/persons/79115221133/accounts/offer HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/funding-sources/v2/persons/<a>personId</a>/accounts/offer</span></h3>
        <ul>
             <li><strong>personId</strong> - номер вашего кошелька без знака "+"</li>
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

{ { "alias": "qw_wallet_eur", "currency": 978 }, {} }
~~~

Успешный JSON-ответ содержит данные о счетах, которые можно создать:

Поле ответа|Тип|Описание
--------|----|----
{} | Object |Коллекция описаний счетов
Object.alias|String|Псевдоним счета
Object.currency|Integer|ID валюты счета

## Установка баланса по умолчанию  {#default_balance}

Запрос устанавливает для вашего QIWI Кошелька счет, баланс которого будет использоваться для фондирования всех платежей по умолчанию. Счет должен содержаться в [списке счетов](#balances_list)

[Потестировать](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/saveAccountAttributesUsingPATCH)

<h3 class="request method">Запрос → PATCH</h3>

~~~shell
user@server:~$ curl -X PATCH "https://edge.qiwi.com/funding-sources/v2/persons/79115221133/accounts/qw_wallet_usd" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
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
    <li><h3>URL <span>/funding-sources/v2/persons/<a>personId</a>/accounts/<a>accountAlias</a></span></h3>
        <ul>
             <li><strong>personId</strong> - номер вашего кошелька без знака "+"</li>
             <li><strong>accountAlias</strong> - псевдоним счета в кошельке из <a href="#balances_list">списка счетов</a> (параметр <i>accounts[].alias</i> в ответе)</li>
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
    <li><h3>Параметры</h3><span>Данный параметр передается в JSON-теле запроса:</span>
    </li>
</ul>


Название|Тип|Описание
--------|----|----
defaultAccount|Boolean| Признак установки счета по умолчанию

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 204 Modified
Content-Type: application/json
~~~

Успешный ответ содержит HTTP-код 204.
