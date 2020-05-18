# Платежи {#payments}

## Комиссионные тарифы {#rates}

Для расчета полной комиссии за платеж (с учетом всех тарифов) по заданному набору платежных реквизитов используйте данный запрос (требуется [авторизация](#auth_api)).

~~~shell
user@server:~$ curl -X POST 'https://edge.qiwi.com/sinap/providers/99/onlineCommission' \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
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

# Тарифные комиссии
def get_commission(api_access_token, to_account, prv_id, sum_pay):
    s = requests.Session()
    s.headers = {'content-type': 'application/json'}
    s.headers['authorization'] = 'Bearer ' + api_access_token  
    postjson = {"account":"","paymentMethod":{"type":"Account","accountId":"643"}, "purchaseTotals":{"total":{"amount":"","currency":"643"}}}
    postjson['account'] = to_account
    postjson['purchaseTotals']['total']['amount'] = sum_pay
    c_online = s.post('https://edge.qiwi.com/sinap/providers/'+prv_id+'/onlineCommission',json = postjson)
    return c_online.json()['qwCommission']['amount']
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
             <li><a href="#banks">Прочие банки</a></li>
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

# Комиссия за перевод на QIWI кошелек
print(get_commission(api_access_token,'+380000000000','99',5000))
# Комиссия за перевод на карту
print(get_commission(api_access_token,'4890xxxxxxxx1698','22351',1000))
~~~

<h3 class="request">Ответ ←</h3>

В параметре JSON-ответа `qwCommission.amount` возвращается рассчитанная сумма комиссии.

<a name="payform"></a>

## Автозаполнение платежных форм {#autocomplete}

Данный запрос отображает в браузере предзаполненную форму на сайте qiwi.com для совершения платежа.

<h3 class="request method">Запрос → GET</h3>

[Пример ссылки (нажмите для перехода на форму)](https://qiwi.com/payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&amountFraction=0&extra%5B%27comment%27%5D=test123&currency=643&blocked[0]=account)

~~~http
POST /payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&amountFraction=0&extra%5B%27comment%27%5D=test123&currency=643 HTTP/1.1
Host: qiwi.com

~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://qiwi.com/ID?<a>parameter=value</a></span></h3></li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>В пути ссылки и в строке запроса указываются параметры платежной формы.</span></li>
</ul>

Параметр|Тип|Описание|Поле на форме| Обяз.
---------|--------|---|----
ID | Integer | Идентификатор провайдера (указывается в пути ссылки).<br>Возможные значения:<br>`99` - [Перевод на QIWI Wallet](#p2p)<br>`99999` -  Перевод на QIWI Wallet по никнейму<br>`1963` - [Перевод на карту Visa](#cards) (карты российских банков)<br>`21013` - [Перевод на карту MasterCard](#cards) (карты российских банков)<br>Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:<br>`1960` – [Перевод на карту Visa](#cards)<br>`21012` – [Перевод на карту MasterCard](#cards)<br>`31652` - [Перевод на карту МИР](#cards)<br>[Идентификаторы операторов мобильной связи](#mnp)<br>[Идентификаторы других провайдеров](#charity) | - | +
amountInteger|Integer | Целая часть суммы платежа (рубли). Указывается в строке запроса. Если параметр не указан, поле "Сумма" на форме будет пустым. **Допустимо число не больше 99 999 (ограничение на сумму платежа)** | Сумма | -
amountFraction|Integer | Дробная часть суммы платежа (копейки). Указывается в строке запроса. Если параметр не указан, поле "Сумма" на форме будет пустым.|Сумма | -
currency|Константа, `643` | Код валюты платежа. Указывается в строке запроса. **Обязательный параметр, если вы передаете в ссылке сумму платежа** |-|+
extra['comment'] |URL-encoded string | Комментарий. Указывается в строке запроса. Имя параметра должно быть URL-закодировано. **Параметр используется только для ID=99** |Комментарий к переводу | -
extra['account'] |URL-encoded string |  Формат совпадает с форматом параметра `fields.account` в соответствующем платежном запросе. Для провайдера `99999` указывается никнейм или номер кошелька (в зависимости от параметра `extra['accountType']`).<br>Имя параметра должно быть URL-закодировано.|Номер Кошелька, Номер телефона/счета/карты получателя.|-
blocked|Array[String]|Признак неактивного поля формы. Пользователь не сможет менять значение данного поля. Каждое поле задается соответствующим именем параметра и нумеруется элементом массива, начиная с нуля (`blocked[0]`, `blocked[1]` и т.д.). Если параметр не указан, пользователь сможет изменить все поля формы. Допустимые значения:<br>`sum` - поле "сумма платежа", <br>`account` - поле "номер счета/телефона/карты",<br>`comment` - поле "комментарий".<br> Пример (неактивное поле суммы платежа): `blocked[0]=sum` |-|-
extra['accountType'] | URL-encoded string | **Параметр используется только для ID=99999**. Значение определяет перевод на QIWI кошелек по никнейму или по номеру кошелька.<br>`phone` - для перевода по номеру<br>`nickname` - для перевода по никнейму.

### Как узнать свой никнейм через API {#nickname}

Используйте данный запрос:

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl -X GET "https://edge.qiwi.com/qw-nicknames/v1/persons/79111234567/nickname" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9"
~~~

~~~http
GET /qw-nicknames/v1/persons/79111234567/nickname HTTP/1.1
Accept: application/json
Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9
Host: edge.qiwi.com
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/qw-nicknames/v1/persons/<a>wallet</a>/nickname</span></h3>
        <ul>
        <strong>В pathname GET-запроса используется параметр:</strong>
             <li><strong>wallet</strong> - номер вашего кошелька (с международным префиксом, но без <i>+</i>)</li>
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
  "canChange": true,
  "canUse": true,
  "description": "",
  "nickname": "NICKNAME"
}
~~~

Успешный ответ в формате JSON содержит ваш никнейм в поле `nickname`.

## Перевод на QIWI Кошелек {#p2p}

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST 'https://edge.qiwi.com/sinap/api/v2/terms/99/payments' \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  -d '{ \
        "id":"11111111111111", \
        "sum": { \
          "amount":100, \
          "currency":"643" \
        }, \
        "paymentMethod": { \
          "type":"Account", \
          "accountId":"643" \
        },
        "comment":"test", \
        "fields": { \
          "account":"+79121112233" \
        } \
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
import time

# Перевод на QIWI Кошелек
def send_p2p(api_access_token, to_qw, comment, sum_p2p):
    s = requests.Session()
    s.headers = {'content-type': 'application/json'}
    s.headers['authorization'] = 'Bearer ' + api_access_token
    s.headers['User-Agent'] = 'Android v3.2.0 MKT'
    s.headers['Accept'] = 'application/json'
    postjson = {"id":"","sum":{"amount":"","currency":""},"paymentMethod":{"type":"Account","accountId":"643"}, "comment":"'+comment+'","fields":{"account":""}}
    postjson['id'] = str(int(time.time() * 1000))
    postjson['sum']['amount'] = sum_p2p
    postjson['sum']['currency'] = '643'
    postjson['fields']['account'] = to_qw
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/99/payments',json = postjson)
    return res.json()
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
print(send_p2p(mylogin,api_access_token,'+79261112233','comment',99.01))

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

Метод выполняет перевод средств на валютный счет QIWI Кошелька с конвертацией с вашего рублевого счета. При этом формируются две транзакции: конвертации между счетами вашего кошелька и перевода на другой кошелек.

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST 'https://edge.qiwi.com/sinap/api/v2/terms/1099/payments' \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
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
POST /sinap/api/v2/terms/1099/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer YUu2qw048gtdsvlk3iu
Host: edge.qiwi.com

{
  "id":"11111111111111",
  "sum": {
		"amount":10.00,
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

~~~python
import requests
import time

# Конвертация в QIWI Кошельке (currency - код валюты String)
def exchange(api_access_token, sum_exchange, currency, to_qw):
    s = requests.Session()
    currencies = ['398', '840', '978']
    if currency not in currencies:
      print('This currency not available')
      return
    s.headers = {'content-type': 'application/json'}
    s.headers['authorization'] = 'Bearer ' + api_access_token
    s.headers['User-Agent'] = 'Android v3.2.0 MKT'
    s.headers['Accept'] = 'application/json'
    postjson = {"id":"","sum":{"amount":"","currency":""},"paymentMethod":{"type":"Account","accountId":"643"}, "comment":"'+comment+'","fields":{"account":""}}
    postjson['id'] = str(int(time.time() * 1000))
    postjson['sum']['amount'] = sum_exchange
    postjson['sum']['currency'] = currency
    postjson['fields']['account'] = to_qw
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/1099/payments',json = postjson)
    return res.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/sinap/api/v2/terms/1099/payments</span></h3></li>
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
sum.amount|Number|Сумма в валюте конвертации. Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.|+
sum.currency|String|Код валюты (number-3 ISO-4217)|+
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
paymentMethod.type|String |Константа, `Account`|+
paymentMethod.accountId|String| Константа, `643`|+
fields|Object| Реквизиты платежа. Содержит параметр:
fields.account| String|Номер кошелька для перевода|+
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

## Курс валют {#exchange}

Метод возвращает текущие курсы и кросс-курсы валют КИВИ Банка.

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl "https://edge.qiwi.com/sinap/crossRates" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9"
~~~

~~~http
GET /sinap/crossRates HTTP/1.1
Accept: application/json
Authorization: Bearer 5c4b25xx93aa435d9cb8cd17480356f9
Content-type: application/json
Host: edge.qiwi.com
~~~

~~~python
import requests

# Курс пары валют (коды валют в String)
def exchange(api_access_token, currency_to, currency_from):
    s = requests.Session()
    s.headers = {'content-type': 'application/json'}
    s.headers['authorization'] = 'Bearer ' + api_access_token
    s.headers['User-Agent'] = 'Android v3.2.0 MKT'
    s.headers['Accept'] = 'application/json'
    res = s.get('https://edge.qiwi.com/sinap/crossRates')

    # все курсы
    rates = res.json()['result']

    # запрошенный курс
    rate = [x for x in rates if x['from'] == currency_from and x['to'] == currency_to]
    if (len(rate) == 0):
        print('No rate for this currencies!')
        return
    else:
        return rate[0]['rate']
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
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
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
import time

# Оплата мобильного телефона
def send_mobile(api_access_token, prv_id, to_account, comment, sum_pay):
    s = requests.Session()
    s.headers['Accept'] = 'application/json'
    s.headers['Content-Type'] = 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    postjson = {"id":"","sum": {"amount":"","currency":"643"},"paymentMethod": {"type":"Account","accountId":"643"},"comment":"","fields": {"account":""}}
    postjson['id'] = str(int(time.time() * 1000))
    postjson['sum']['amount'] = sum_pay
    postjson['fields']['account'] = to_account
    postjson['comment'] = comment
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/'+prv_id+'/payments', json = postjson)
    return res.json()
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
user@server:~$ curl -X POST "https://qiwi.com/mobile/detect.action" \
  --header "Accept: application/json" \
  --header "Content-Type: application/x-www-form-urlencoded" \
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

def mobile_operator(phone_number):
    s = requests.Session()
    res = s.post('https://qiwi.com/mobile/detect.action', data = {'phone': phone_number })
    s.headers['Accept'] = 'application/json'
    s.headers['Content-Type'] = 'application/x-www-form-urlencoded'
    return res.json()['message']
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
    <li><h3>Параметры</h3><span>Параметр передается в теле запроса как <code>formdata</code>.</span>
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
print(mobile_operator(79652468447))
~~~

Ответ с HTTP Status 200 и параметром `code.value` = 0 является признаком успешной проверки. Идентификатор оператора находится в параметре `message`.

Ответ с HTTP Status 200 и параметром `code.value` = 2 означает, что невозможно определить оператора.

## Перевод на карту {#cards}

Запрос выполняет денежный перевод на карты платежных систем Visa, MasterCard или МИР. Предварительно можно [проверить тип платежной системы](#card_check).

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1963/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
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
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1960/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
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
             <li>31652 - Перевод на карту национальной платежной системы МИР</li>
             <li>22351 - Перевод на <a href="https://qiwi.com/qvc/help.action">Виртуальную карту QIWI</a></li>
             <li>Для карт, выпущенных банками стран Азербайджан, Армения, Белоруссия, Грузия, Казахстан, Киргизия, Молдавия, Таджикистан, Туркменистан, Украина, Узбекистан:
             <ul><li>1960 – Перевод на карту Visa</li>
             <li>21012 – Перевод на карту MasterCard</li></ul></li>
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
fields.rem_name|String|Имя отправителя. Требуется только для ID (идентификатор провайдера в запросе) `1960`, `21012`
fields.rem_name_f|String|Фамилия отправителя. Требуется только для ID (идентификатор провайдера в запросе) `1960`, `21012`
fields.rec_address|String|Адрес отправителя (без почтового индекса, в произвольной форме). Требуется только для ID (идентификатор провайдера в запросе) `1960`, `21012`
fields.rec_city|String|Город отправителя. Требуется только для ID (идентификатор провайдера в запросе) `1960`, `21012`
fields.rec_country|String|Страна отправителя. Требуется только для ID (идентификатор провайдера в запросе) `1960`, `21012`
fields.reg_name|String|Имя **получателя**. Требуется только для ID (идентификатор провайдера в запросе) `1960`, `21012`
fields.reg_name_f|String|Фамилия **получателя**. Требуется только для ID (идентификатор провайдера в запросе) `1960`, `21012`

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

Запрос не требует авторизации.

<h3 class="request method" id="card_check">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://qiwi.com/card/detect.action" \
  --header "Accept: application/json" \
  --header "Content-Type: application/x-www-form-urlencoded" \
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

def card_system(card_number):
    s = requests.Session()
    res = s.post('https://qiwi.com/card/detect.action', data = {'cardNumber': card_number })
    return res.json()['message']
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
print(card_system(4890xxxxxxxx1698))
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

Запрос выполняет денежный перевод на карты/счета физических лиц, открытые в российских банках.

### Перевод по номеру карты

Запрос выполняет денежный перевод на карты физических лиц, выпущенные российскими банками.

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/464/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
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
POST /sinap/api/v2/terms/464/payments HTTP/1.1
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
             <ul><li>464 - Альфа-Банк</li>
             <li>804 - АО "ОТП БАНК"</li>
             <li>810 - АО "РОССЕЛЬХОЗБАНК"</li>
             <li>815 - Русский Стандарт</li>
             <li>816 - ВТБ (ПАО)</li>
             <li>821 - Промсвязьбанк</li>
             <li>870 - ПАО Сбербанк</li>
             <li>881 - Ренессанс Кредит</li>
             <li>1134 - ПАО "МОСКОВСКИЙ КРЕДИТНЫЙ БАНК"</li>
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
fields.account| String| Номер банковской карты получателя
fields.exp_date| String|Срок действия карты, в формате `ММГГ` (например, `0218`). **Параметр указывается только в случае перевода на карту Альфа-Банка (ID 464) и Промсвязьбанка (ID 821).**
fields.account_type| String|Тип банковского идентификатора. Для каждого банка применяется собственное значение:<br>Альфа-Банк - `1`<br>ОТП банк - `1`<br>Русский Стандарт - `1`<br>Россельхозбанк - `5`<br>ВТБ - `5`<br>Промсвязьбанк - `7`<br>Сбербанк - `5`<br>МОСКОВСКИЙ КРЕДИТНЫЙ БАНК - `5`<br>Ренессанс Кредит - `1`.
fields.mfo| String|БИК соответствующего банка/территориального отделения банка
fields.lname|String|Фамилия получателя
fields.fname|String|Имя получателя
fields.mname|String|Отчество получателя


<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "21131343",
  "terms": "464",
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
fields|Object|Копия объекта `fields` из исходного запроса. **В параметре** `fields.account` **находится маскированный номер банковской карты получателя**.
sum|Object|Копия объекта `sum` из исходного запроса.
source| String| Константа, `account_643`
transaction|Object|Объект с данными о транзакции в процессинге QIWI Wallet. Параметры:
transaction.id|String|ID транзакции в процессинге QIWI Wallet
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге QIWI Wallet. Параметр:
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).


### Перевод по номеру счета/договора

Запрос выполняет денежный перевод на счета физических лиц, открытые в российских банках. Возможен обычный перевод или перевод с использованием сервиса срочного перевода (исполнение в течение часа, с 9:00 до 19:30).

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/816/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
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
POST /sinap/api/v2/terms/816/payments HTTP/1.1
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
             <ul><li>313 - ХоумКредит Банк</li>
             <li>464 - Альфа-Банк</li>
             <li>821 - Промсвязьбанк</li>
             <li>804 - АО "ОТП БАНК"</li>
             <li>810 - АО "РОССЕЛЬХОЗБАНК"</li>
             <li>816 - ВТБ (ПАО)</li>
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
fields.urgent | String | Признак ускоренного перевода. Значение 0 - не использовать; значение 1 - выполнить перевод через Сервис срочного перевода ЦБ РФ. **Внимание! Взимается дополнительная комиссия за ускоренный перевод**
fields.mfo| String|БИК соответствующего банка/территориального отделения банка
fields.account_type| String|Тип банковского идентификатора. Для каждого банка применяется собственное значение:<br>Альфа-Банк - `2`<br>ОТП банк - `2`<br>Россельхозбанк - `2`<br>Русский Стандарт - `2`<br>ВТБ - `2`<br>Промсвязьбанк - `9`<br>ЮНИКРЕДИТ БАНК - `2`<br>КИВИ БАНК - `2`<br>Сбербанк - `2`<br>МОСКОВСКИЙ КРЕДИТНЫЙ БАНК - `2`<br>РАЙФФАЙЗЕНБАНК -`2`<br>Россельхозбанк - `2`<br>ВТБ - `5`<br>Промсвязьбанк - `9`<br>Ренессанс Кредит - `2`<br>ХоумКредит Банк - `6`.
fields.lname|String|Фамилия получателя
fields.fname|String|Имя получателя
fields.mname|String|Отчество получателя
fileds.agrnum|String|Номер договора - для переводов в ХоумКредит Банк

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "21131343",
  "terms": "464",
  "fields": {
          "account": "407121010910909011",
          "account_type": "2"
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
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/674/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
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
import time

# оплата простого провайдера

def pay_simple_prv(api_access_token, prv_id, to_account, sum_pay):
    s = requests.Session()
    s.headers['Accept'] = 'application/json'
    s.headers['Content-Type'] = 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    postjson = {"id":"","sum": {"amount":"","currency":"643"},"paymentMethod": {"type":"Account","accountId":"643"},"fields": {"account":""}}
    postjson['id'] = str(int(time.time() * 1000))
    postjson['sum']['amount'] = sum_pay
    postjson['fields']['account'] = to_account
    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/'+prv_id+'/payments', json = postjson)
    return res.json()
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

<aside class="notice">Поиск идентификатора выполняется на сайте qiwi.com в поисковой строке. Идентификатор находится в URL ссылки на платежную форму провайдера вида <a>https://qiwi.com/payment/form/ID</a> или <a>https://qiwi.com/payment/form/ID</a></aside>

![Поиск провайдера](/images/provider_id.jpg)

~~~python
import requests

# поиск на qiwi.com - определение id провайдера по названию
def qiwi_com_search(search_phrase):
    s = requests.Session()
    search = s.post('https://qiwi.com/search/results/json.action', params={'searchPhrase':search_phrase})
    return search.json()['data']['items']

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
# Платёж на провайдера
print(pay_simple_prv(api_access_token,'26386','2166191','10'))

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
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/api/v2/terms/1717/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer YUu2qw048gtdsvlk3iu" \
  --header "User-Agent: ***" \
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

# Счета {#invoices}

## Выставление счета {#invoice}

Для выставления счета на QIWI Кошелек используется протокол [API P2P-счетов](https://developer.qiwi.com/ru/p2p-payments/#create). Для получения ключей авторизации запросов вы можете зайти на [p2p.qiwi.com](https://p2p.qiwi.com) в личный кабинет, или использовать представленный запрос. Этим запросом можно также настроить адрес [уведомлений об оплате счетов](https://developer.qiwi.com/ru/p2p-payments/#notification)

Данный запрос возвращает пару ключей в параметрах ответа `PublicKey` и `SecretKey`.


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

Для заголовка Authorization используется токен API QIWI Кошелька.

<ul class="nestedList url">
<li><h3>URL</h3> <span>https://edge.qiwi.com/widgets-api/api/p2p/protected/keys/create</span></li>
</ul>

Параметр|Тип|Описание
--------|----|----
keysPairName| String| Название пары ключей
serverNotificationsUrl|String |URL для [уведомлений об оплате счетов](https://developer.qiwi.com/ru/p2p-payments/#notification) (необязательный параметр)

## Список счетов  {#list_invoice}

Метод получения списка неоплаченных счетов вашего кошелька. Список строится в обратном хронологическом порядке. Можно использовать фильтры по времени выставления счета, начальному идентификатору счета.

<h3 class="request method">Запрос → GET</h3>

~~~shell
user@server:~$ curl -X GET --header 'Accept: application/json' \
   --header 'Authorization: Bearer ***' \
   'https://edge.qiwi.com/checkout/api/bill/search?statuses=READY_FOR_PAY&rows=50'
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
             <li>Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f</li>
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
statuses|String| Статус неоплаченного счета. Только строка `READY_FOR_PAY`
min_creation_datetime|Long|Нижняя временная граница для поиска счетов, Unix-time
max_creation_datetime|Long|Верхняя временная граница для поиска счетов, Unix-time
next_id|Number|Начальный идентификатор счета для поиска, чтобы возврат списка выполнялся начиная с этого значения
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
      "pay_url":"https://oplata.qiwi.com/form?shop=480706&transaction=102263702405"
    }
  ]
}
~~~

Успешный JSON-ответ содержит список неоплаченных счетов вашего кошелька, соответствующих заданному фильтру:

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

Выполнение безусловной оплаты счета без SMS-подтверждения.

<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST --header 'Content-Type: application/json;charset=UTF-8' \
   --header 'Accept: application/json' \
   --header 'Authorization: Bearer 68ec21fd52e4244838946dd07ed225a1' \
   -d '{ \
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
             <li>Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в теле запроса в формате JSON. Все параметры обязательны.</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
invoice_uid | String |ID счета в QIWI (берется из значения `bills[].id` [данных о счете](#invoice_data)
currency|String| Валюта суммы счета (берется из значения `bills[].sum.currency` [данных о счете](#invoice_data))


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
invoice_status|String|Строка кода статуса оплаты счета, `PAID_STATUS`. Любой другой статус означает неуспех платежной транзакции.
is_sms_confirm|String|Признак подтверждения по SMS

## Отмена неоплаченного счета  {#cancel_invoice}

Метод отклоняет неоплаченный счет. При этом счет становится недоступным для оплаты.


<h3 class="request method">Запрос → POST</h3>

~~~shell
user@server:~$ curl -X POST --header 'Accept: application/json' \
                            --header 'Authorization: Bearer ***' \
                            'https://edge.qiwi.com/checkout/api/bill/reject' \
                            -d '{ "id": 1034353453 }'
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
             <li>Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Данный обязательный параметр передается в теле запроса в формате JSON:</span>
    </li>
</ul>

Параметр|Тип|Описание
--------|----|----
id | Integer |ID счета для отмены (берется из значения `bills[].id` [данных о счете](#invoice_data)

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
~~~

Успешный ответ содержит HTTP-код 200.
