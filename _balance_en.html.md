
# Wallet Balances API {#balance}

###### Last update: 2020-07-28 | [Edit on GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/_balance_en.html.md)

API provides methods to control balances of your QIWI wallet.

## List of balances {#balances_list}

The request provides current balances of your QIWI Wallet.

[Interactive API](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/getByAliasUsingGET_1)

<h3 class="request method">Request → GET</h3>

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

# QIWI Wallet balances
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
             <li><strong>personId</strong> - your wallet number (without  <i>+</i> sign)</li>
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
# wallet number as 79992223344
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# All balances
balances = balance(mylogin,api_access_token)['accounts']

# Ruble account balance
rubAlias = [x for x in balances if x['alias'] == 'qw_wallet_rub']
rubBalance = rubAlias[0]['balance']['amount']
~~~

> Repeated request when you got empty "balance" object and "hasBalance": true in response

~~~http
GET /funding-sources/v2/persons/79115221133/accounts?timeout=1000&alias=qw_wallet_rub HTTP/1.1
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com
~~~

Successful response is JSON array of all active balances of your QIWI wallet used for payment funding:

Response field | Type | Description
--------|----|----
accounts|Array[Object]|Array of balances
accounts[].alias | String | User balance alias
accounts[].fsAlias | String |  Bank account alias
accounts[].bankAlias | String | Bank alias
accounts[].title|String| Wallet account name
accounts[].hasBalance|Boolean|Flag of actual QIWI Wallet balance (not a linked card or cell phone balance or something like that)
accounts[].currency | Number| Currency of the balance (number-3 ISO-4217). Only balances in following currencies are returned: 643 - Russian ruble, 840 - USD, 978 - Euro
accounts[].type|Object| Account information
type.id, type.title| String| Account title
accounts[].balance|Object | Balance data.<br>If `null` is returned and  `accounts[].hasBalance` is `true`, repeat the request with additional parameters:<br>`timeout=1000` and `alias="accounts[].alias"` (alias of that balance)
balance.amount|Number|Текущий баланс данного счета
balance.currency | Number| Код валюты баланса (number-3 ISO-4217)

## Creating balance {#balance_create}

The request creates a new account and its balance in your QIWI wallet. List of account types which you can create is provided by [other request](#funding_offer).

[Interactive API](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/createAccountUsingPOST)

<h3 class="request method">Request → POST</h3>

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
             <li><strong>personId</strong> - your wallet number without <i>+</i> sign</li>
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
    <li><h3>Parameters</h3><span>In JSON body of the request:</span>
    </li>
</ul>


Name|Type|Description
--------|----|----
alias|String| Alias of the new account (taken from [Available accounts](#funding_offer))

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 201 Created
Content-Type: application/json
~~~

Successful response contains HTTP code 201.

## Available accounts {#funding_offer}

The request provides all possible account aliases for your QIWI wallet.

[Interactive API](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/getAccountsOfferUsingGET)

<h3 class="request method">Request → GET</h3>

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
             <li><strong>personId</strong> - your wallet number without <i>+</i> sign</li>
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

{ { "alias": "qw_wallet_eur", "currency": 978 }, {} }
~~~

Successful JSON response contains a list of accounts available for creation:

Response field|Type|Description
--------|----|----
{} | Object | List of accounts
Object.alias|String|Alias of the account
Object.currency|Integer|Account's currency ID

## Default balance  {#default_balance}

The request sets up default account in your QIWI wallet for funding all payments. The account must be in the list of  [available accounts](#balances_list)

[Interactive API](https://developer.qiwi.com/sandbox/index.html#!/account-controller-v-2/saveAccountAttributesUsingPATCH)

<h3 class="request method">Request → PATCH</h3>

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
             <li><strong>personId</strong> - your wallet number without <i>+</i> sign</li>
             <li><strong>accountAlias</strong> - account's alias in the wallet, from <a href="#balances_list">a list</a> (see parameter <i>accounts[].alias</i> in response)</li>
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
    <li><h3>Parameters</h3><span>Parameter in JSON body:</span>
    </li>
</ul>


Name|Type|Description
--------|----|----
defaultAccount|Boolean| Flag of default account

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 204 Modified

~~~

Successful response has HTTP code 204.
