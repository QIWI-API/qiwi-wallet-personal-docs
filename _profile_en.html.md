
# User's Profile API {#profile}

###### Last update: 2020-07-28 | [Edit on GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/_profile_en.html.md)

The API returns information about your profile - a set of user data and settings of your QIWI Wallet.

<h3 class="request method">Request → GET</h3>

~~~shell
curl "https://edge.qiwi.com/person-profile/v1/profile/current?authInfoEnabled=false" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API token>"
~~~

~~~http
GET /person-profile/v1/profile/current HTTP/1.1
Accept: application/json
Authorization: Bearer <API token>
Host: edge.qiwi.com
~~~

~~~python
import requests

# User profile
def get_profile(api_access_token):
    s7 = requests.Session()
    s7.headers['Accept']= 'application/json'
    s7.headers['authorization'] = 'Bearer ' + api_access_token
    p = s7.get('https://edge.qiwi.com/person-profile/v1/profile/current?authInfoEnabled=true&contractInfoEnabled=true&userInfoEnabled=true')
    return p.json()
~~~

~~~python
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# Full info
profile = get_profile(api_access_token)

# Blocking status
profile['contractInfo']['blocked']

# Identification level
profile['contractInfo']['identificationInfo'][0]['identificationLevel']

# Linked email
profile['authInfo']['boundEmail']
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/person-profile/v1/profile/current?<a>parameter=value</a></span></h3></li>
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
    <li><h3>Parameters</h3><span>These options are transmitted in the URL query:</span>
    </li>
</ul>

Name|Type |Description
--------|----|-------
authInfoEnabled|Boolean | Flag to get authorization settings.<br>By default, `true`
contractInfoEnabled|Boolean | Flag to get your QIWI wallet data.<br>By default, `true`
userInfoEnabled|Boolean | Flag to get other user data.<br>By default, `true`

<h3 class="request">Response ←</h3>

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
        "identificationLevel": "SIMPLE",
        "passportExpired": false
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

Successful JSON-response has the following fields:

Field|Type|Description
--------|----|----
authInfo|Object|Current authorization settings. The object may be missing, depending on the `authInfoEnabled` parameter in the request.
authInfo.personId|Number|Wallet number
authInfo.registrationDate|String|QIWI Wallet date/registration time (via website/mobile app, or otherwise)
authInfo.boundEmail|String| E-mail linked to your QIWI wallet. If not, `null`
authInfo.ip|String|IP address of the last user session
authInfo.lastLoginDate|String|The date/time of the last session in QIWI Wallet service
authInfo.mobilePinInfo|Object|PIN data of the mobile app
mobilePinInfo.mobilePinUsed|Boolean|Flag of using a PIN-code (actually means that the mobile app is being used)
mobilePinInfo.lastMobilePinChange|String|The date/time of the last time of the PIN change in the mobile app
mobilePinInfo.nextMobilePinChange|String|The date/time of the next (planned) change of the PIN in the mobile app
authInfo.passInfo|Object|Password usage data on the qiwi.com site
passInfo.passwordUsed|Boolean|Flag of using password (actually means using the site qiwi.com)
passInfo.lastPassChange|String|The date/time of the last password change on qiwi.com
passInfo.nextPassChange|String|The date/time of the next (planned) password change on qiwi.com
authInfo.pinInfo|Object|PIN usage data to the wallet app on the self-service kiosks
pinInfo.pinUsed|Boolean|Flag of using a PIN for the terminal (actually means using the wallet app on the self-service kiosk)
contractInfo|Object| Information about the wallet. The object may be missing, depending on the `contractInfoEnabled` parameter in the request.
contractInfo.blocked|Boolean|Flag of wallet block
contractInfo.contractId|Number|Wallet number
contractInfo.creationDate|String|The date/time to create a wallet (via website/mobile app, either at first topup or otherwise)
contractInfo.features|Array[Object]|Service info
contractInfo.identificationInfo|Array[Object]|User's [identification data](https://qiwi.com/settings/identification)
identificationInfo[].bankAlias|String|String's acronym of the system, in which the user has received the identification::<br> `QIWI` - QIWI Wallet.
identificationInfo[].identificationLevel|String|Current level of the wallet identification. Possible values:<br>`ANONYMOUS` - no identification;<br> `SIMPLE`, `VERIFIED` - simplified identification;<br> `FULL` - full identification.
identificationInfo[].passportExpired|Boolean|Validity of passport data of the wallet' owner (`true` means that the passport data is invalid).
userInfo|Object|Other user data. The object may be missing, depending on the `userInfoEnabled` parameter in the request.
userInfo.defaultPayCurrency|Number(3)|Default wallet balance currency code (ISO-4217)
userInfo.defaultPaySource|Number|Service info
userInfo.email|String|User's e-mail
userInfo.firstTxnId|Number|Identifier of the first transaction after registration
userInfo.language|String|Service info
userInfo.operator|String|Name of the mobile operator of the user's number
userInfo.phoneHash|String|Service info
userInfo.promoEnabled|String|Service info

# Identification API {#identification}

Use methods of this API to identify and check identification status of your wallet in QIWI Wallet service. You need identification to get access to increased limits for balances and transactions.

[Identification details (in Russian)](https://qiwi.com/settings/identification#ru)

## User identification {#ident}

This request allows you to start the identification of your QIWI Wallet.

<aside class="warning">It is permissible to identify no more than 5 wallets per owner. See 3.1.1(III) of the <a href="https://static.qiwi.com/ru/doc/oferta_lk.pdf">QIWI Wallet Offer</a></aside>

To obtain Main wallet status, you must provide the following data about the owner of the wallet:

* The name
* Series / Passport No.
* Date of birth
* TIN, SNILS or OMS policy number is optional.

To identify your wallet, you must send your name, series/passport number and date of birth. If the data has been verified, the answer will show your TIN and simplified wallet identification will be established. If the data has not been verified, the wallet remains in the status of "No Identification."

<h3 class="request method">Request → POST</h3>

~~~shell
curl -X POST \
  "https://edge.qiwi.com/identification/v1/persons/79111234567/identification" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer <API token>" \
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
Authorization: Bearer <API token>
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

def get_identification(api_access_token, my_login):
    s = requests.Session()
    s.headers['authorization'] = 'Bearer ' + api_access_token
    res = s.get('https://edge.qiwi.com/identification/v1/persons/'+my_login+'/identification')
    return res.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/identification/v1/persons/<a>wallet</a>/identification</span></h3>
        <ul>
             <li><strong>wallet</strong> - number of your QIWI wallet without <i>+</i> sign</li>
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
    <li><h3>Parameters</h3><span>Send in JSON body of the request:</span>
    </li>
</ul>

Name|Type|Description
--------|----|----
birthDate|String| Date of birth (in "YYYY-MM-DD" format)
firstName|String| User's first name
middleName|String| User's surname
lastName|String| User's last name
passport|String| Series / Passport No. (only digits)
inn|String| User's TIN
snils|String| User's SNILS
oms|String| User's medical insurance number (OMS)

<h3 class="request">Response ←</h3>

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

Successful JSON response confirms wallet identification data:

Response field|Type|Description
--------|----|----
id|  Number  | User's QIWI wallet number
type | String | Current identification level of the wallet:<br>`SIMPLE` - no identification, wallet status "MINIMAL".<br>`VERIFIED` - wallet status "MAIN"  (identification data has been successfully verified).<br>`FULL` – the wallet already got "FULL" status by the provided personally verified name, passport and date of birth.
birthDate |String | Date of birth
firstName|String| User's first name
middleName|String| User's surname
lastName|String| User's last name
passport|String| Series / Passport No. (only digits)
inn|String| User's TIN. If the parameter is returned but not present in the original request, then the wallet identification is established
snils|String| User's SNILS
oms|String| User's medical insurance number (OMS)

## Identification data {#ident_data}

Gets masked private data and identification status of your QIWI Wallet.

<h3 class="request method">Request → GET</h3>

~~~shell
curl -X GET \
  "https://edge.qiwi.com/identification/v1/persons/79111234567/identification" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API token>"
~~~

~~~http
GET /identification/v1/persons/79111234567/identification HTTP/1.1
Accept: application/json
Authorization: Bearer <API token>
Host: edge.qiwi.com
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/identification/v1/persons/<a>wallet</a>/identification</span></h3>
        <ul>
             <li><strong>wallet</strong> - your QIWI wallet number without <i>+</i> sign</li>
        </ul>
    </li>
</ul>

<ul class="nestedList url">
    <li><h3>URL <span>/qw-limits/v1/persons/<a>personId</a>/actual-limits?<a>parameter=value</a></span></h3></li>
        <ul>
             <li><strong>personId</strong> - your QIWI wallet number without <i>+</i> sign</li>
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

<h3 class="request">Response ←</h3>

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

Successful JSON response contains masked data used for the wallet identification:

Response field|Type|Description
--------|----|----
id|  Number  | User's QIWI wallet number
type | String | Current identification level of the wallet:<br>`SIMPLE` - no identification, status "MINIMAL".<br>`VERIFIED` - status "MAIN" (identification data has been successfully verified).<br>`FULL` – "FULL" status, the wallet already got full identification by the provided name, passport and date of birth.
birthDate |String | Date of birth
firstName|String| User's first name
middleName|String| User's surname
lastName|String| User's last name
passport|String| Series / Passport No. (first and last two digits)
inn|String| User's TIN (first and last two digits)
snils|String| User's SNILS (first and last two digits)
oms|String| User's medical insurance number (OMS) (first and last two digits)

# QIWI Wallet Limits API {#limits}

## Limit levels {#limit-levels}

By using this API, you can get current limits for operations in your QIWI wallet. Limits apply on accessible amount of the operations.

<h3 class="request method">Request → GET</h3>

~~~shell
curl "https://edge.qiwi.com/qw-limits/v1/persons/79115221133/actual-limits?types%5B0%5D=TURNOVER" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <API token>"
~~~

~~~http
GET /qw-limits/v1/persons/79115221133/actual-limits?types%5B0%5D=TURNOVER HTTP/1.1
Accept: application/json
Authorization: Bearer <API token>
Host: edge.qiwi.com
~~~

~~~python
import requests

def limits(login, api_access_token):
    types = [ 'TURNOVER', 'REFILL', 'PAYMENTS_P2P', 'PAYMENTS_PROVIDER_INTERNATIONALS', 'PAYMENTS_PROVIDER_PAYOUT', 'WITHDRAW_CASH']
    s = requests.Session()
    s.headers['Accept']= 'application/json'
    s.headers['Content-Type']= 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    parameters = {}
    for i, type in enumerate(types):
        parameters['types[' + str(i) + ']'] = type
    b = s.get('https://edge.qiwi.com/qw-limits/v1/persons/' + login + '/actual-limits', params = parameters)
    return b.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/qw-limits/v1/persons/<a>personId</a>/actual-limits?<a>parameter=value</a></span></h3>
        <ul>
             <li><strong>personId</strong> - your QIWI wallet number without <i>+</i> sign</li>
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
    <li><h3>Parameters</h3><span>Send in the request path:</span>
    </li>
</ul>

Name|Type|Description
--------|----|----
types|Array[String]| A list of the types of operations that limits are requested for. Each type is numbered by an array element, starting from zero (`types[0]`, `types[1]`, etc.). Acceptable types of transactions:<br>`REFILL` - balance in the account<br>`TURNOVER` - turnover per month<br>`PAYMENTS_P2P` - transfers to other wallets per month<br>`PAYMENTS_PROVIDER_INTERNATIONALS` - payments to foreign companies per month<br>`PAYMENTS_PROVIDER_PAYOUT` - transfers to bank accounts and cards, wallets of other systems<br>`WITHDRAW_CASH` - cash withdrawals per month.<br>At least one type of operation must be specified.

<h3 class="request">Response ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "limits":{
      "RU" :[
        {
            "type": "TURNOVER",
            "currency": "RUB",
            "rest": 200.00,
            "max": 40000.00,
            "spent": 39800.00,
            "interval": {
                "dateFrom": "2019-11-01T:00:00",
                "dateTill": "2019-12-01T00:00"
            }
        },
        ...
    ]
  }
}
~~~

~~~python
# wallet number like 79992223344
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# list of all limits
limits = limits(mylogin,api_access_token)['limits']['RU']

# turnover limit
turnoverInfo = [x for x in limits if x['type'] == 'TURNOVER']
turnoverLimit = turnoverInfo[0]['rest']
~~~

Successful JSON response contains an array of limits for your QIWI wallet operations:

Response field|Type|Description
--------|----|----
limits|Object| Limits data
limits[].'RU'|Array[Object]| An array of limits for operations
type | String | Operation type where the limit is applied
currency | String | Currency of the operation
max | String | Value of the limit
spent|String| Amount already spent in the operations
rest|Boolean| The rest of the limit which can be used in the given period (see `interval` field)
interval|Object| Details of the limit's period
interval.dateFrom, interval.dateTill| String| Period's start and end, as `YYYY-MM-DDTHH:MM:SStmz` string

## Person-to-person transaction limit {#p2p-limit}

The API returns the value of the person-to-person transaction number for the current month.

<h3 class="request method">Request → GET</h3>

~~~shell
curl "https://edge.qiwi.com/qw-limits/v1/persons/79999999999/p2p-payment-count-limit" \
  --header "Accept: application/json" \
  --header "Authorization:  Bearer <API token>"
~~~

~~~http
GET /qw-limits/v1/persons/79999999999/p2p-payment-count-limit HTTP/1.1
Accept: application/json
Authorization: Bearer <API token>
Host: edge.qiwi.com
~~~

~~~python
import requests

# Number of person-to-person transactions
def get_p2p_payment_count(login, api_access_token):
    s = requests.Session()
    s.headers['Accept']= 'application/json'
    s.headers['Content-Type']= 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    b = s.get('https://edge.qiwi.com/qw-limits/v1/persons/' + login + '/p2p-payment-count-limit')
    return b.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/qw-limits/v1/persons/<a>personId</a>/p2p-payment-count-limit</span></h3>
        <ul>
             <li><strong>personId</strong> - your QIWI wallet number without "+"</li>
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
    "p2pPaymentCountLimit": 1
}
~~~

~~~python
mylogin = '79999999999'
api_access_token = '975efd8e8376xxxb95fa7cb213xxx04'

# P2p payment count for the current month
print(get_p2p_payment_count(api_access_token, mylogin))

{'p2pPaymentCountLimit': 1}

~~~

Successful JSON response contains an information of your person-to-person transactions:

Response field|Type|Description
--------|--------|----
p2pPaymentCountLimit| Number |Person-to-person transaction count per month
