# Платежное API {#payments}

###### [Предложить правки на GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/_payment_ru.html.md)

API предоставляет доступ к платежам в пользу провайдеров услуг, зарегистрированных в сервисах QIWI Кошелька.

<aside class="notice">
Уровень доступа, допустимая сумма платежной операции и лимит на платежи зависят от <a href="https://qiwi.com/settings/identification">уровня идентификации QIWI кошелька</a>.
</aside>

## Комиссионные тарифы {#rates}

 Чтобы узнать комиссию за платеж до его совершения по заданному набору платежных реквизитов, используйте этот запрос. Возвращается полная комиссия QIWI Кошелька за платеж в пользу указанного провайдера с учетом всех тарифов.

**<h3 class="request method">Запрос → POST</h3>

~~~shell
curl -X POST \
  'https://edge.qiwi.com/sinap/providers/99/onlineCommission' \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer <токен API>" \
  -d '{
        "account":"79060766516",
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
~~~**

~~~http
POST /sinap/providers/99/onlineCommission HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer <токен API>
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

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/providers/<a>{ID}</a>/onlineCommission</span></h3>
    </li>
</ul>

**ID** — идентификатор провайдера. Возможные значения:

* 99 — Перевод на QIWI Кошелек.
* 1717 — [Перевод по банковским реквизитам организации](#freepay).
* [Провайдеры банковских переводов](#banks).

Также идентификатор нужного провайдера можно установить [поиском по ключевым словам](#provider-search-words).

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
    <li><h3>Параметры</h3>
    </li>
</ul>

Обязательные параметры в теле запроса:

Название|Тип|Описание
--------|----|----
account| String| Пользовательский идентификатор (номер телефона с международным префиксом, номер карты/счета получателя, и т.д., в зависимости от провайдера)
paymentMethod | Object| Объект, определяющий обработку платежа процессингом QIWI Wallet. Содержит следующие параметры:
paymentMethod.type|String |Метод платежа, только `Account`
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

Рассчитанная сумма комиссии возвращается в поле `qwCommission.amount` JSON-ответа.

<a name="payform"></a>

## Автозаполнение платежных форм {#autocomplete}

Запрос отображает в браузере предзаполненную форму на сайте qiwi.com для совершения платежа.

[Пример ссылки (нажмите для перехода на форму)](https://qiwi.com/payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&amountFraction=0&extra%5B%27comment%27%5D=test123&currency=643&blocked[0]=account)

Если вы не хотите, чтобы пользователь видел номер вашего кошелька на форме, используйте перевод по никнейму:

[Пример ссылки на перевод по никнейму (нажмите для перехода на форму)](https://qiwi.com/payment/form/99999?extra%5B%27account%27%5D=NICKNAME&amountInteger=1&amountFraction=0&currency=643&blocked[0]=account&extra%5B%27accountType%27%5D=nickname)

<aside class="notice">Для приема переводов на свой QIWI кошелек можно использовать <a href="https://qiwi.com/p2p-admin/transfers/link">P2P-форму</a>.</aside>

<h3 class="request method">Запрос → GET</h3>

~~~http
GET /payment/form/99?extra%5B%27account%27%5D=79991112233&amountInteger=1&amountFraction=0&extra%5B%27comment%27%5D=test123&currency=643 HTTP/1.1
Host: qiwi.com

~~~

<ul class="nestedList url">
    <li><h3>URL  <span>https://qiwi.com/<a>{ID}</a>?<a>{parameter}={value}</a></span></h3>
    </li>
</ul>

**ID** — идентификатор провайдера, у которого [набор реквизитов платежа](#payments_model) ограничен только полем `fields.account`. Возможные значения:

* 99 — Перевод на QIWI Кошелек.
* 99999 —  Перевод на QIWI Кошелек по никнейму.
* 1963 — Перевод на карту Visa (карты российских банков).
* 21013 — Перевод на карту MasterCard (карты российских банков).
* 31652 — Перевод на карту МИР.
* 22351 — Перевод на [Виртуальную карту QIWI](https://qiwi.com/cards/qvc).
* 1717 — [Перевод по банковским реквизитам организации](#freepay).

Также идентификатор нужного провайдера можно установить [поиском по ключевым словам](#provider-search-words).

<ul class="nestedList params">
    <li><h3>Параметры</h3></li>
</ul>

В строке URL запроса указываются параметры отображения платежной формы:

Название|Тип|Описание|Поле на форме
---------|--------|---|----
amountInteger|Integer | Целая часть суммы платежа (рубли). Если параметр не указан, поле "Сумма" на форме будет пустым. **Допустимо число не больше 99 999 (ограничение на сумму платежа)** | Сумма 
amountFraction|Integer | Дробная часть суммы платежа (копейки). Если параметр не указан, поле "Сумма" на форме будет пустым.|Сумма
currency|Константа, `643` | Код валюты платежа. **Обязательный параметр, если вы передаете в ссылке сумму платежа** |-
extra['comment'] |URL-encoded string | Комментарий. **Параметр используется только для ID=99** |Комментарий к переводу
extra['account'] |URL-encoded string |  Формат совпадает с форматом параметра `fields.account` при оплате соответствующих провайдеров: для провайдера `99` - номер кошелька получателя; для провайдеров сотовой связи - номер мобильного телефона для пополнения (без префикса 8); для провайдеров перевода на карту - номер банковской карты получателя (без пробелов), для других провайдеров - идентификатор пользователя. Для провайдера `99999` указывается [никнейм](#nickname) или номер кошелька получателя (задайте соответствующее значение параметра `extra['accountType']`).| Номер Кошелька, номер телефона/счета/карты/пользовательский ID получателя.
blocked|Array[String]|Признак неактивного поля формы. Пользователь не сможет менять значение данного поля. Каждый параметр задает соответствующее поле формы и нумеруется начиная с нуля (`blocked[0]`, `blocked[1]` и т.д.). Если не указан, пользователь сможет изменить все поля формы. Допустимые значения:<br>`sum` - поле "сумма платежа", <br>`account` - поле "номер счета/телефона/карты",<br>`comment` - поле "комментарий".<br> Пример (неактивное поле суммы платежа): `blocked[0]=sum` |-
extra['accountType'] | URL-encoded string | **Параметр используется только для ID=99999**. Значение определяет перевод на QIWI кошелек по никнейму или по номеру кошелька.<br>`phone` — для перевода по номеру<br>`nickname` — для перевода по никнейму.  Если вы не хотите, чтобы пользователь видел номер вашего кошелька на форме, используйте это значение.|-

### Как узнать свой никнейм через API {#nickname}

<h3 class="request method">Запрос → GET</h3>

~~~shell
curl -X GET \
  "https://edge.qiwi.com/qw-nicknames/v1/persons/79111234567/nickname" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>"
~~~

~~~http
GET /qw-nicknames/v1/persons/79111234567/nickname HTTP/1.1
Accept: application/json
Authorization: Bearer <токен API>
Host: edge.qiwi.com
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/qw-nicknames/v1/persons/<a>{wallet}</a>/nickname</span></h3>
    </li>
</ul>

**wallet** — номер вашего кошелька без знака `+`.

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

Успешный ответ в формате JSON содержит никнейм вашего кошелька в поле `nickname`.

## Перевод на QIWI Кошелек {#p2p}

<h3 class="request method">Запрос → POST</h3>

~~~shell
curl -X POST \
  'https://edge.qiwi.com/sinap/api/v2/terms/99/payments' \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>" \
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
        "comment":"Комментарий",
        "fields": {
          "account":"+79121112233"
        }
      }'
~~~

~~~http
POST /sinap/api/v2/terms/99/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer <токен API>
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
 "comment":"Комментарий",
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
    <li><h3>URL <span>/sinap/api/v2/terms/99/payments</span></h3></li>
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
    <li><h3>Параметры</h3>
    </li>
</ul>

В теле запроса передается JSON-объект. Структура объекта описана в классе [Payment](#payment_obj). Набор реквизитов платежа передается во вложенном объекте `fields`:

Название|Тип|Описание
--------|----|----
fields.account| String|**Обязательный параметр**. Номер кошелька для перевода

<h3 class="request">Ответ ←</h3>

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

В успешном JSON-ответе возвращается объект со структурой [класса PaymentInfo](#payment_info) с данными о принятом платеже.

## Конвертация {#exchange}

Запрос выполняет перевод средств на валютный счет QIWI Кошелька с конвертацией с вашего рублевого счета. При этом формируются две транзакции: конвертации между счетами вашего кошелька и перевода на другой кошелек. Курс валют для конвертации можно узнать [другим запросом](#CCY).

<h3 class="request method">Запрос → POST</h3>

~~~shell
curl -X POST \
  'https://edge.qiwi.com/sinap/api/v2/terms/1099/payments' \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>" \
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
        "comment":"Комментарий",
        "fields": {
          "account":"+79121112233"
        }
      }'
~~~

~~~http
POST /sinap/api/v2/terms/1099/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer <токен API>
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
	"comment":"Комментарий",
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
    <li><h3>URL <span>/sinap/api/v2/terms/1099/payments</span></h3></li>
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
<li><h3>Параметры</h3>
</li>
</ul>

В теле запроса передается JSON-объект. Структура объекта описана в классе [Payment](#payment_obj). Набор реквизитов платежа передается во вложенном объекте `fields`:

Название|Тип|Описание
--------|----|----
fields.account| String|**Обязательный параметр**. Номер кошелька для перевода

<h3 class="request">Ответ ←</h3>

В успешном JSON-ответе возвращается объект со структурой [класса PaymentInfo](#payment_info) с данными о принятом платеже.

## Курсы валют {#CCY}

Запрос возвращает текущие курсы и кросс-курсы валют КИВИ Банка.

<h3 class="request method">Запрос → GET</h3>

~~~shell
curl "https://edge.qiwi.com/sinap/crossRates" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>"
~~~

~~~http
GET /sinap/crossRates HTTP/1.1
Accept: application/json
Authorization: Bearer <токен API>
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
    <li><h3>URL <span>/sinap/crossRates</span></h3></li>
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

Успешный JSON-ответ содержит список курсов валют в списке `result`. Элемент списка соответствует валютной паре:

Поле ответа|Тип|Описание
--------|----|----
from|String|Валюта покупки
to|String|Валюта продажи
rate|Number|Курс

## Оплата сотовой связи {#cell}

<h3 class="request method">Запрос → POST</h3>

~~~shell
curl -X POST \
  "https://edge.qiwi.com/sinap/api/v2/terms/1/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>" \
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
Authorization: Bearer <токен API>
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
    <li><h3>URL <span>/sinap/api/v2/terms/<a>{ID}</a>/payments</span></h3>
    </li>
</ul>

**ID** — идентификатор провайдера. Определяется с помощью [поиска провайдера](#provider-search-mobile).

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
<li><h3>Параметры</h3>
</li>
</ul>

В теле запроса передается JSON-объект. Структура объекта описана в классе [Payment](#payment_obj). Набор реквизитов платежа передается во вложенном объекте `fields`:

Название|Тип|Описание
--------|----|----
fields.account| String|Номер мобильного телефона для пополнения (без префикса `8`)

<h3 class="request">Ответ ←</h3>

~~~python
send_mobile(api_access_token,'2','9670058909','123','1')
~~~

В успешном JSON-ответе возвращается объект со структурой [класса PaymentInfo](#payment_info) с данными о принятом платеже.

## Перевод на карту {#cards}

### Поиск ID для переводов на банковские карты РФ и Белоруссии {#cards-search-id}

Определение **ID** для переводов на банковские карты РФ и Белоруссии.

<h3 class="request method">Запрос → POST</h3>

~~~http
POST /sinap/api/refs/bd6fb248-2bdf-49ed-bcb2-9b0a789cfde8/containers HTTP/1.1
Accept: application/vnd.qiwi.v1+json
Content-Type: application/json
Host: edge.qiwi.com
Authorization: Bearer <токен API>

{
  "account":"1234 1234 1234 1234"
}
~~~

~~~shell
curl -X POST \
  "https://edge.qiwi.com/sinap/api/refs/bd6fb248-2bdf-49ed-bcb2-9b0a789cfde8/containers" \
  --header "Content-Type: application/json" \
  --header "Accept: application/vnd.qiwi.v1+json" \
  --header "Authorization: Bearer <токен API>" \
  -d '{
    "account":"1234 1234 1234 1234"
  }'
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/refs/bd6fb248-2bdf-49ed-bcb2-9b0a789cfde8/containers</span></h3>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/vnd.qiwi.v1+json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
<li><h3>Параметры</h3>
</li>
</ul>

В поле `account` JSON-тела запроса передается номер карты в строковом формате. В номере после каждой четвертой цифры ставится пробел.

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "elements": [
    { 
      "type": "field",
      "name": "termsId",
      "value": "36390"
    }
  ]
}
~~~

~~~shell
{"elements":[{"type": "field","name": "termsId","value": "36390"}]}
~~~

В поле ответа `elements[].value` возвращается ID провайдера для перевода на банковскую карту.

### Перевод на карты банков РФ и зарубежных банков {#cards-rf-and-foreign}

Запрос выполняет денежный перевод на карты платежных систем Visa, MasterCard или МИР.

<span style="font-weight:bold;color:red;">Платежи на карты Visa и MasterCard, выпущенные иностранными банками, временно остановлены по причине ограничений со стороны платежной системы</span>.

<h3 class="request method">Запрос → POST</h3>

> Пример перевода на карту банка РФ

~~~shell
curl -X POST \
  "https://edge.qiwi.com/sinap/api/v2/terms/1963/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>" \
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
Authorization: Bearer <токен API>
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

> Пример перевода на международную карту

~~~shell
curl -X POST \
  "https://edge.qiwi.com/sinap/api/v2/terms/1960/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>" \
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
POST /sinap/api/v2/terms/1960/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer <токен API>
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

~~~python
import requests
import time

# Перевод на карту
def send_card(api_access_token, payment_data):
    # payment_data - dictionary with all payment data
    s = requests.Session()
    s.headers['Accept'] = 'application/json'
    s.headers['Content-Type'] = 'application/json'
    s.headers['authorization'] = 'Bearer ' + api_access_token
    postjson = {"id":"","sum": {"amount":"","currency":"643"},"paymentMethod": {"type":"Account","accountId":"643"},"fields": {"account":""}}
    postjson['id'] = str(int(time.time() * 1000))
    postjson['sum']['amount'] = payment_data.get('sum')
    postjson['fields']['account'] = payment_data.get('to_card')
    prv_id = payment_data.get('prv_id')
    if payment_data.get('prv_id') in ['1960', '21012']:
        postjson['fields']['rem_name'] = payment_data.get('rem_name')
        postjson['fields']['rem_name_f'] = payment_data.get('rem_name_f')
        postjson['fields']['reg_name'] = payment_data.get('reg_name')
        postjson['fields']['reg_name_f'] = payment_data.get('reg_name_f')
        postjson['fields']['rec_city'] = payment_data.get('rec_address')
        postjson['fields']['rec_address'] = payment_data.get('rec_address')

    res = s.post('https://edge.qiwi.com/sinap/api/v2/terms/' + prv_id + '/payments', json = postjson)
    return res.json()
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/<a>{ID}</a>/payments</span></h3>
    </li>
</ul>

**ID** — идентификатор провайдера. Возможные значения:

* 1963 — Перевод на карту Visa. Для карт, выпущенных российскими банками.
* 1960 — Перевод на карту Visa. Для карт, выпущенных банками стран Албания, Андорра,Аргентина, Армения, Австралия, Австрия, Азербайджан, Беларусь, Бельгия, Бенин, Босния и Герцеговина, Бразилия, Болгария, Китай, Хорватия, Кипр, Чешская Республика, Дания, Египет, Эстония, Финляндия, Франция, Грузия, Германия, Греция, Гонконг (Китай), Венгрия, Исландия, Индия, Индонезия, Израиль, Италия, Япония, Казахстан, Кения, Корея Республика, Кувейт, Кыргызстан, Латвия, Литва, Люксембург, Макао, Китай, Македония, Мадагаскар, Малайзия, Мальдивы, Мальта, Республика Молдова, Монако, Монголия, Черногория, Намибия, Нидерланды, Новая Зеландия, Нигерия, Норвегия, Оман, Парагвай, Польша, Португалия, Катар, Румыния, Саудовская Аравия, Республика Сербия, Сингапур, Словакия, Словения, Южная Африка, Испания, Шри-Ланка, Швеция, Таджикистан, Танзания, Таиланд, Турция, Туркменистан, Объединенные Арабские Эмираты, Великобритания, Узбекистан, Вьетнам, Замбия.
* 21013 — Перевод на карту MasterCard. Для карт, выпущенных российскими банками.
* 21012 — Перевод на карту MasterCard. Для карт, выпущенных банками стран Албания, Аргентина, Армения, Австралия, Австрия, Азербайджан, Бангладеш, Барбадос, Беларусь, Бельгия, Бенин, Босния и Герцеговина, Буркина-Фасо, Бразилия, Болгария, Камбоджа, Камерун Объединенная Республика, Чили, Китай, Колумбия, Конго, Коста-Рика, Хорватия, Кипр, Чешская Республика, Демократическая Республика Конго, Дания, Доминиканская Республика, Эквадор, Сальвадор, Египет, Эстония, Финляндия, Франция, Грузия, Германия, Гана, Греция, Гватемала, Гонконг, Венгрия, Индия, Индонезия, Ирландия, Израиль, Италия, Япония, Иордания, Казахстан, Кения, Корея, Кувейт, Кыргызстан, Латвия, Ливан, Литва, Люксембург, Макао, Македония, Мадагаскар, Малайзия, Мальдивы, Мальта, Мексика, Молдова, Монако, Монголия, Черногория, Марокко, Намибия, Нигерия, Непал, Нидерланды, Новая Зеландия, Нигерия, Норвегия, Оман, Панама, Парагвай, Перу, Филиппины, Польша, Португалия, Румыния, Катар, Саудовская Аравия, Сенегал, Сербия Республика, Сингапур, Словакия, Словения, Южная Африка, Испания, Шри-Ланка, Швеция, Швейцария, Таджикистан, Танзания, Тайланд, Тунис, Турция, Туркменистан, Объединенные Арабские Эмираты, Великобритания, Узбекистан, Вьетнам, Замбия.
* 31652 — Перевод на карту МИР.
* 22351 — Перевод на [Виртуальную карту QIWI](https://qiwi.com/cards/qvc).

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
<li><h3>Параметры</h3>
</li>
</ul>

В теле запроса передается JSON-объект. Структура объекта описана в классе [Payment](#payment_obj). Набор реквизитов платежа в поле `fields` зависит от ID провайдера.

**Параметры для ID 1963, 21013, 31652, 22351**

Название|Тип|Описание
--------|----|----
fields.account| String|Номер банковской карты получателя (без пробелов)

**Параметры для ID 1960, 21012**

Название|Тип|Описание
--------|----|----
fields.account| String|Номер банковской карты получателя (без пробелов)
fields.rem_name|String|Имя отправителя
fields.rem_name_f|String|Фамилия отправителя
fields.rec_address|String|Адрес отправителя (без почтового индекса, в произвольной форме)
fields.rec_city|String|Город отправителя
fields.rec_country|String|Страна отправителя
fields.reg_name|String|Имя **получателя**
fields.reg_name_f|String|Фамилия **получателя**

<h3 class="request">Ответ ←</h3>

В успешном JSON-ответе возвращается объект со структурой [класса PaymentInfo](#payment_info) с данными о принятом платеже.

### Перевод на карты банков Казахстана {#cards-kzt}

Запрос выполняет денежный перевод на карты Visa или MasterCard, выпущенные банками Казахстана.

<h3 class="request method">Запрос → POST</h3>

> Пример перевода на карту

~~~shell
curl -X POST \
  --location \
  "https://edge.qiwi.com/sinap/api/terms/27292/payments" \
  -H "authorization: Bearer <токен API>" \
  -H "accept: application/vnd.qiwi.v2+json" \
  -H "sec-fetch-site: same-site" \
  -H "sec-fetch-mode: cors" \
  -H "sec-fetch-dest: empty" \
  -H "Content-Type: application/json" \
  -d '{
        "id": "<случайный id платежа>",
        "sum": {
          "amount": <сумма перевода>,
          "currency": "398"
        },
        "paymentMethod": {
          "accountId": "398",
          "type": "Account"
        },
        "comment": "",
        "fields": {
          "cardNumber": "<номер карты>",
          "version": "2",
          "transferSum": "100",
          "info": "Для продолжения оплаты, подтвердите, что являетесь держателем указанного банковского счета.",
          "accept": "1",
          "account": "<значение account из подготовительного запроса>",
          "ev_account1": "<значение ev_account1 из подготовительного запроса>"
        }
    }`
~~~

~~~http
POST /sinap/api/terms/27292/payments HTTP/1.1
Content-Type: application/json
Accept: application/vnd.qiwi.v2+json
Authorization: Bearer <токен API>
sec-fetch-site: same-site
sec-fetch-mode: cors
sec-fetch-dest: empty
Host: edge.qiwi.com

{
  "id":"21131343",
  "sum": {
        "amount":1000,
        "currency":"398"
  },
  "paymentMethod": {
      "type":"Account",
      "accountId":"398"
  },
  "fields": {
            "cardNumber": "<номер карты>",
            "version": "2",
            "transferSum": "100",
            "info": "Для продолжения оплаты, подтвердите, что являетесь держателем указанного банковского счета."
            "accept": "1",
            "account": "<значение account из подготовительного запроса>",
            "ev_account1": "<значение ev_account1 из подготовительного запроса>"
  }
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/terms/27292/payments</span></h3>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/vnd.qiwi.v2+json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
             <li>sec-fetch-site: same-site</li>
             <li>sec-fetch-mode: cors</li>
             <li>sec-fetch-dest: empty</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
<li><h3>Параметры</h3>
</li>
</ul>

В теле запроса передается JSON-объект. Структура объекта описана в классе [Payment](#payment_obj). Набор реквизитов платежа передается во вложенном объекте `fields`:

Название|Тип|Описание
--------|----|----
fields.cardNumber| String|Номер банковской карты получателя (без пробелов)
fields.account|String|Значение поля `account` из [ответа на подготовительный запрос](#prepare-card-kzt-transfer)
fields.ev_account1|String|Значение поля `ev_account1` из [ответа на подготовительный запрос](#prepare-card-kzt-transfer)
fields.transferSum|String|Всегда `100`
fields.accept|String|Всегда `1`
fields.info|String|Всегда `Для продолжения оплаты, подтвердите,`<br/>`что являетесь держателем указанного банковского счета.`

<h3 class="request">Ответ ←</h3>

В успешном JSON-ответе возвращается объект со структурой [класса PaymentInfo](#payment_info) с данными о принятом платеже.

### Подготовительный запрос для перевода на карту {#prepare-card-kzt-transfer}

<h3 class="request method">Запрос → POST</h3>

> Пример подготовительного запроса

~~~shell
curl -X POST \
  "https://edge.qiwi.com/sinap/api/refs/a42ebc79-0584-4271-b8a0-15cb4ea8b340/containers" \
  --header "Content-Type: application/json" \
  --header "Accept: application/vnd.qiwi.v1+json" \
  --header "Authorization: Bearer <токен API>" \
  -H 'sec-fetch-site: same-site' \
  -H 'sec-fetch-mode: cors' \
  -H 'sec-fetch-dest: empty' \
  --data-raw '{"cardNumber":"<ваш номер карты>","version":"2","transferSum":"100","src":"sinap"}' \
  --compressed
~~~

~~~http
POST /sinap/api/refs/a42ebc79-0584-4271-b8a0-15cb4ea8b340/containers HTTP/1.1
Content-Type: application/json
Accept: application/vnd.qiwi.v1+json
Authorization: Bearer YUu2qw048gtdsvlk3iu
sec-fetch-site: same-site
sec-fetch-mode: cors
sec-fetch-dest: empty
Host: edge.qiwi.com

{
  "cardNumber":"<ваш номер карты>",
  "version":"2",
  "transferSum":"100",
  "src":"sinap"
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/refs/a42ebc79-0584-4271-b8a0-15cb4ea8b340/containers</span></h3>
    </li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/vnd.qiwi.v1+json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer *** </li>
             <li>sec-fetch-site: same-site</li>
             <li>sec-fetch-mode: cors</li>
             <li>sec-fetch-dest: empty</li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
<li><h3>Параметры</h3>
</li>
</ul>

Название|Тип|Описание
--------|----|----
cardNumber| String|Номер банковской карты получателя (без пробелов)
transferSum|String|Всегда `100`
version|String|Всегда `2`
src|String|Всегда `sinap`

<h3 class="request">Ответ ←</h3>

~~~json
...
"elements":[
  {
    "type":"field",
    "name":"account",
    "value":"<маскированный PAN карты>"
  },
  {
    "type":"field",
    "name":"ev_account1",
    "value":"<длинная строка>"
  }
]
~~~

Успешный JSON-ответ содержит блоки `"name": "account"` и `"name": "ev_account1"`. Сохраните значения из поля `value` для передачи в [платежном запросе](#cards-kzt).

## Банковский перевод {#banks}

Запрос выполняет денежный перевод на карты/счета физических лиц, открытые в российских банках.

### Перевод по номеру карты

Запрос выполняет денежный перевод на карты физических лиц, выпущенные российскими банками.

<h3 class="request method">Запрос → POST</h3>

~~~shell
curl -X POST \
  "https://edge.qiwi.com/sinap/api/v2/terms/464/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>" \
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
          "exp_date": "0422"
        }
    }'
~~~

~~~http
POST /sinap/api/v2/terms/464/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: Bearer <токен API>
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
        "exp_date": "0422"
  }
}
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/sinap/api/v2/terms/<a>{ID}</a>/payments</span></h3>
    </li>
</ul>

**ID** — идентификатор провайдера. Возможные значения:

* 464 — Альфа-Банк
* 804 — АО "ОТП БАНК"
* 810 — АО "РОССЕЛЬХОЗБАНК"
* 815 — Русский Стандарт
* 816 — ВТБ (ПАО)
* 821 — Промсвязьбанк
* 870 — ПАО Сбербанк
* 881 — Ренессанс Кредит
* 1134 — ПАО "МОСКОВСКИЙ КРЕДИТНЫЙ БАНК"

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
<li><h3>Параметры</h3>
</li>
</ul>

В теле запроса передается JSON-объект. Структура объекта описана в классе [Payment](#payment_obj). Набор реквизитов платежа передается во вложенном объекте `fields`:

Название|Тип|Описание
--------|----|----
fields.account| String| Номер банковской карты получателя (без пробелов)
fields.exp_date| String|Срок действия карты, в формате `ММГГ` (например, `0218`). **Параметр указывается только в случае перевода на карту Альфа-Банка (ID 464) и Промсвязьбанка (ID 821).**
fields.account_type| String|Тип банковского идентификатора. Номер карты соответствует типу `1`. Для некоторых банков применяются собственные значения:<br>Россельхозбанк - `5`<br>ВТБ - `5`<br>Промсвязьбанк - `7`<br>Сбербанк - `5`<br>МОСКОВСКИЙ КРЕДИТНЫЙ БАНК - `5`.
fields.mfo| String|БИК соответствующего банка/территориального отделения банка
fields.lname|String|Фамилия получателя
fields.fname|String|Имя получателя
fields.mname|String|Отчество получателя

<h3 class="request">Ответ ←</h3>

В успешном JSON-ответе возвращается объект со структурой [класса PaymentInfo](#payment_info) с данными о принятом платеже.

### Перевод по номеру счета/договора {#money-order-bank-account}

Запрос выполняет денежный перевод на счета физических лиц, открытые в российских банках. Возможен обычный перевод или перевод с использованием сервиса срочного перевода (исполнение в течение часа, с 9:00 до 19:30).

<h3 class="request method">Запрос → POST</h3>

~~~shell
curl -X POST \
  "https://edge.qiwi.com/sinap/api/v2/terms/816/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>" \
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
Authorization: Bearer <токен API>
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
    <li><h3>URL <span>/sinap/api/v2/terms/<a>{ID}</a>/payments</span></h3>
    </li>
</ul>

**ID** — идентификатор провайдера. Возможные значения:

* 313 - ХоумКредит Банк
* 464 - Альфа-Банк
* 821 - Промсвязьбанк
* 804 - АО "ОТП БАНК"
* 810 - АО "РОССЕЛЬХОЗБАНК"
* 816 - ВТБ (ПАО)
* 819 - АО ЮНИКРЕДИТ БАНК
* 868 - КИВИ БАНК (АО)
* 870 - ПАО Сбербанк
* 1134 - ПАО "МОСКОВСКИЙ КРЕДИТНЫЙ БАНК"
* 27324 - АО "РАЙФФАЙЗЕНБАНК"

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
<li><h3>Параметры</h3>
</li>
</ul>

В теле запроса передается JSON-объект. Структура объекта описана в классе [Payment](#payment_obj). Набор реквизитов платежа передается во вложенном объекте `fields`:

Название|Тип|Описание
--------|----|----
fields.account| String| Номер банковского счета получателя
fields.urgent | String | Признак ускоренного перевода. Значение `0` — не использовать; значение `1` — выполнить перевод через Сервис срочного перевода ЦБ РФ. **Внимание! Взимается дополнительная комиссия за ускоренный перевод**
fields.mfo| String|БИК соответствующего банка/территориального отделения банка
fields.account_type| String|Тип банковского идентификатора. Номер счета (`2`) или номер договора (`3`). Для некоторых банков применяются собственные значения:<br>Промсвязьбанк — `9`<br>ВТБ — `5`<br>ХоумКредит Банк — `6`.
fields.lname|String|Фамилия получателя
fields.fname|String|Имя получателя
fields.mname|String|Отчество получателя
fileds.agrnum|String|Номер договора. Только для переводов в ХоумКредит Банк

<h3 class="request">Ответ ←</h3>

В успешном JSON-ответе возвращается объект со структурой [класса PaymentInfo](#payment_info) с данными о принятом платеже.

## Оплата других услуг {#services}

Оплата услуги по идентификатору пользователя. Запрос применяется для провайдеров, использующих в реквизитах единственный пользовательский идентификатор, без проверки номера аккаунта.

<h3 class="request method">Запрос → POST</h3>

~~~shell
curl -X POST \
  "https://edge.qiwi.com/sinap/api/v2/terms/674/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>" \
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
Authorization: Bearer <токен API>
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
    <li><h3>URL <span>/sinap/api/v2/terms/<a>{ID}</a>/payments</span></h3>
    </li>
</ul>

**ID** — идентификатор провайдера. Возможные значения:

* 674 - OnLime.
* 1239 - Фонд Подари жизнь.
* Идентификатор другого интернет-провайдера или благотворительного фонда. Воспользуйтесь [поиском провайдера по ключевым словам](#provider-search-words).

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
<li><h3>Параметры</h3>
</li>
</ul>

В теле запроса передается JSON-объект. Структура объекта описана в классе [Payment](#payment_obj). Набор реквизитов платежа передается во вложенном объекте `fields`:

Название|Тип|Описание
--------|----|----
fields.account| String| Пользовательский идентификатор

<h3 class="request">Ответ ←</h3>

В успешном JSON-ответе возвращается объект со структурой [класса PaymentInfo](#payment_info) с данными о принятом платеже.

## Платеж по свободным реквизитам {#freepay}

Оплата услуг коммерческих организаций по их банковским реквизитам.

<h3 class="request method">Запрос → POST</h3>

~~~shell
curl -X POST \
  "https://edge.qiwi.com/sinap/api/v2/terms/1717/payments" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>" \
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
Authorization: Bearer <токен API>
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
    <li><h3>URL <span>/sinap/api/v2/terms/1717/payments</span></h3></li>
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
<li><h3>Параметры</h3>
</li>
</ul>

В теле запроса передается JSON-объект. Структура объекта описана в классе [Payment](#payment_obj). Набор реквизитов платежа передается во вложенном объекте `fields`:

Название|Тип|Описание
--------|----|----
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

В успешном JSON-ответе возвращается объект со структурой [класса PaymentInfo](#payment_info) с данными о принятом платеже.

## Поиск провайдера по ключевым словам {#provider-search-words}

Используйте этот запрос для поиска идентификатора провайдера. В запросе указывается список ключевых слов (например, название провайдера), разделенных пробелами.

<h3 class="request method">Запрос → GET</h3>

~~~shell
curl -X GET \
  "https://edge.qiwi.com/search/v1/search?query=%D0%91%D0%B8%D0%BB%D0%B0%D0%B9%D0%BD+%D0%B4%D0%BE%D0%BC%D0%B0%D1%88%D0%BD%D0%B8%D0%B9+%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D0%BD%D0%B5%D1%82" \
  --header "Accept: application/json" \
  --header "Authorization: Bearer <токен API>"
~~~

~~~http
GET /search/v1/search?query=%D0%91%D0%B8%D0%BB%D0%B0%D0%B9%D0%BD+%D0%B4%D0%BE%D0%BC%D0%B0%D1%88%D0%BD%D0%B8%D0%B9+%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D0%BD%D0%B5%D1%82 HTTP/1.1
Accept: application/json
Host: edge.qiwi.com
Authorization: Bearer <токен API>
~~~

~~~python
import requests

# поиск на qiwi.com - определение id провайдера по названию
def qiwi_com_search(api_access_token, search_phrase):
    s = requests.Session()
    s.headers['authorization'] = 'Bearer ' + api_access_token
    search = s.get('https://edge.qiwi.com/search/v1/search', params={'query':search_phrase})
    return search.json()['items']
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/search/v1/search?<a>query={value}</a></span></h3>
    </li>
</ul>

**query** — строка ключевых слов, разделенных пробелами.

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "items": [
        {
            "name": "МТС Домашний интернет, ТВ и Телефония РФ",
            "description": "МТС Домашний интернет, ТВ и Телефония РФ",
            "uri": null,
            "data": {
                "id": 23729,
                "logoUrl": "https://static.qiwi.com/img/providers/logoBig/23729_l.png",
                ...
            }
        },
        ...
    ]
}
~~~

~~~python
# Поиск провайдера: парсинг ответа
prv = qiwi_com_search('xxxxxxxxxxxxxxxxxxx','Билайн домашний интернет')[0]['data']['id']
print(str(prv))
~~~

Успешный JSON-ответ содержит идентификаторы найденных провайдеров:

Поле ответа | Тип | Описание
-----|----|-----
items | Array | Список провайдеров
items[].data.id | Number | Идентификатор провайдера

## Поиск сотового оператора по номеру телефона {#provider-search-mobile}

Используйте этот запрос для поиска идентификатора провайдера сотового оператора. В запросе указывается номер мобильного телефона в формате 11 цифр, начинающийся с цифры `7`. Действует только для мобильных операторов РФ и Казахстана.

<h3 class="request method">Запрос → GET</h3>

~~~shell
curl "https://edge.qiwi.com/qw-mobile-providers-resolver/v1/providers?phoneNumber=79277010101" \
--header "Accept: application/json" \
--header "Authorization: Bearer <токен API>" \
~~~

~~~http
GET /qw-mobile-providers-resolver/v1/providers?phoneNumber=79270010101 HTTP/1.1
Accept: application/json
Host: edge.qiwi.com
Authorization: Bearer <токен API>
~~~

~~~python
import requests

# поиск на qiwi.com - определение id провайдера по номеру телефона
def qiwi_com_search_mobile(api_access_token, number):
    s = requests.Session()
    s.headers['authorization'] = 'Bearer ' + api_access_token
    search = s.get('https://edge.qiwi.com/qw-mobile-providers-resolver/v1/providers', params={'phoneNumber':number})
    return search.json()['mobileOperatorProviderList']
~~~

<ul class="nestedList url">
    <li><h3>URL <span>https://edge.qiwi.com/qw-mobile-providers-resolver/v1/providers?phoneNumber={value}</span></h3>
    </li>
</ul>

**value** — номер мобильного телефона в формате 11 цифр, начинающийся с `7`.

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Authorization: Bearer ***</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "mobileOperatorProviderList": [
    { 
      "id": 3,
      "shortName": "МегаФон Столичный филиал",
      "logoUrl": "https://static.qiwi.com/img/providers/logoBig/3_l.png"
    }
  ]
}
~~~

~~~python
# Поиск провайдера: парсинг ответа
prv = qiwi_com_search_mobile('xxxxxxxxxxxxxxxxxxx','79270010101')[0]['id']
print(str(prv))
~~~

Успешный JSON-ответ содержит идентификатор найденного провайдера:

Поле ответа | Тип | Описание
-----|----|-----
mobileOperatorProviderList | Array | Информация о провайдере
---|---|---
id | Number | Идентификатор провайдера
shortName | String | Название провайдера

## Модели данных API {#payments_model}

### Класс Payment {#payment_obj}

Класс, описывающий данные для платежа на провайдера в QIWI Кошельке.

Элемент|Тип|Описание
--------|----|---------
id | String |**Обязательный параметр**. Клиентский ID транзакции (максимум 20 цифр). Должен быть уникальным для каждой транзакции и увеличиваться с каждой последующей транзакцией. Для выполнения этих требований рекомендуется задавать равным 1000*(Standard Unix time в секундах).
sum|Object| **Обязательный параметр**. Данные о сумме платежа
-----|-----|-----
sum.amount|Number|**Обязательный параметр**. Сумма (можно указать рубли и копейки, разделитель `.`). Положительное число, округленное до 2 знаков после десятичной точки. При большем числе знаков значение будет округлено до копеек в меньшую сторону.
sum.currency|String|**Обязательный параметр**. Валюта (только `643`, рубли)
-----|-----|-----
paymentMethod | Object| **Обязательный параметр**. Объект, определяющий обработку платежа процессингом QIWI Wallet.
-----|-----|-----
paymentMethod.type|String |**Обязательный параметр**. Константа, `Account`
paymentMethod.accountId|String| **Обязательный параметр**. Константа, `643`
-----|-----|-----
fields|Object| **Обязательный параметр**. Реквизиты платежа. Состав полей зависит от провайдера.
comment|String|Комментарий к платежу. Используется только для [переводов на QIWI кошелек](#p2p) и при [конвертации](#CCY)

### Класс PaymentInfo {#payment_info}

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
    "comment": "Комментарий"
}
~~~

> Пример ответа с маскированным полем

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

Класс, описывающий данные платежной транзакции в QIWI Кошельке. Возвращается в ответе на запросы к платежному API.

Элемент | Тип | Описание
-----|----|-----
id | Number | Копия параметра `id` из платежного запроса
terms | String | Идентификатор провайдера, на которого был отправлен платеж
fields|Object|Копия объекта `fields` из платежного запроса. **Номер карты (если был выполнен перевод на карту) возвращается в маскированном виде**
sum|Object|Копия объекта `sum` из платежного запроса
source| String| Константа, `account_643`
comment| String | Копия параметра `comment` из платежного запроса (возвращается, если присутствует в запросе)
transaction|Object|Объект с данными о транзакции в процессинге QIWI Wallet.
-----|-----|------
transaction.id|String|ID транзакции в процессинге QIWI Wallet
transaction.state|Object|Объект содержит текущее состояние транзакции в процессинге QIWI Wallet.
-----|-----|------
state.code | String| Текущий статус транзакции, только значение `Accepted` (платеж принят к проведению). Финальный результат транзакции можно узнать в [истории платежей](#payments_history).

# Счета {#invoices}

Счет в QIWI Wallet API — универсальная заявка на платеж или перевод с QIWI кошелька.

В API поддерживаются операции [выставления](#invoice), [оплаты](#paywallet_invoice) и [отмены](#cancel_invoice) счетов, а также [запрос списка неоплаченных счетов](#list_invoice) вашего QIWI кошелька.

## Выставление счета на QIWI кошелек {#invoice}

Для выставления счета на QIWI Кошелек используется протокол [API P2P-счетов](/ru/p2p-payments/#create). Для авторизации используется [токен P2P](#p2p-token).

### Выпуск токена P2P {#p2p-token}

Вы можете получить токен P2P на [p2p.qiwi.com](https://p2p.qiwi.com) в личном кабинете, или использовать представленный ниже запрос. Этим запросом можно также настроить адрес [уведомлений об оплате счетов](/ru/p2p-payments/#notification).

Запрос возвращает в ответе пару токенов P2P:

* поле `PublicKey` — токен для [выставления счета при вызове платежной формы](/ru/p2p-payments/#http);
* поле `SecretKey` — токен для выставления счета через API.

<h3 class="request method">Запрос → POST</h3>

~~~shell
curl -X POST \
  https://edge.qiwi.com/widgets-api/api/p2p/protected/keys/create \
  -H 'Authorization: Bearer <токен API QIWI Кошелька>' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'cache-control: no-cache' \
  -d '{"keysPairName":"Name","serverNotificationsUrl":"https://test.com"}'
~~~

~~~http
POST /widgets-api/api/p2p/protected/keys/create HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer <токен API QIWI Кошелька>
Content-Type: application/json
Accept: application/json
User-Agent: ****

{"keysPairName":"Name", "serverNotificationsUrl":"https://test.com"}
~~~

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{"PublicKey": "XXX", "SecretKey": "YYY"}
~~~

<ul class="nestedList url">
<li><h3>URL <span>/widgets-api/api/p2p/protected/keys/create</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Content-Type: application/json</li>
             <li>Accept: application/json</li>
             <li>Authorization: Bearer <a href="#auth_data">Токен QIWI Wallet API</a></li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>

Для авторизации используется [токен API QIWI Кошелька](#auth_data).

Параметры передаются в теле запроса как JSON:

Название|Тип|Описание
--------|----|----
keysPairName| String| Название пары токенов P2P (произвольная строка)
serverNotificationsUrl|String |URL для [уведомлений об оплате счетов](/ru/p2p-payments/#notification) (необязательный параметр)

## Список счетов {#list_invoice}

Метод получения списка **неоплаченных** счетов вашего кошелька. Список строится в обратном хронологическом порядке. По умолчанию, список разбивается на страницы по 50 элементов в каждой, но вы можете задать другое количество элементов (не более 50). В запросе можно использовать фильтры по времени выставления счета, начальному идентификатору счета.

<h3 class="request method">Запрос → GET</h3>

~~~shell
curl -X GET \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer ***' \
  'https://edge.qiwi.com/checkout-api/api/bill/search?statuses=READY_FOR_PAY&rows=50'
~~~

~~~http
GET /checkout-api/api/bill/search?statuses=READY_FOR_PAY&rows=50 HTTP/1.1
Accept: application/json
Authorization: Bearer ***
Host: edge.qiwi.com
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/checkout-api/api/bill/search?statuses=READY_FOR_PAY&<a>parameter=value</a></span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Authorization: Bearer <a href="#p2p-token">SecretKey</a></li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>

Параметры передаются в строке URL запроса:

Название|Тип|Описание
--------|----|----
statuses|String| Статус неоплаченного счета. Обязательный параметр. Только строка `READY_FOR_PAY`
rows | Integer |Максимальное число счетов в ответе, для разбивки списка на страницы. Целое число от 1 до 50. По умолчанию возвращается не более 50 счетов.
min_creation_datetime|Long|Нижняя временная граница для поиска счетов, Unix-time
max_creation_datetime|Long|Верхняя временная граница для поиска счетов, Unix-time
next_id|Number|Начальный идентификатор счета для поиска. Будет возвращен список счетов с идентификаторами, равными или меньше этого значения. Используется для продолжения списка, разбитого на страницы.
next_creation_datetime|Long|Начальное время для поиска (возвращаются только счета, выставленные ранее этого времени), Unix-time. Используется для продолжения списка, разбитого на страницы.

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
        "short_name": "Интернет-магазин цветов",
        "long_name": "ООО «Ромашка»",
        "logo_url":"https://static.qiwi.com/img/providers/logoBig/480706_l.png"
      },
      "comment": "Пополнение счета 13515573",
      "pay_url":"https://oplata.qiwi.com/form?shop=480706&transaction=102263702405"
    }
  ]
}
~~~

Успешный JSON-ответ содержит список неоплаченных счетов вашего кошелька, соответствующих заданному фильтру:

<a name="invoice_data"></a>

Поле ответа|Тип|Описание
--------|----|----
bills|Array[Object]|Список счетов. <br>Длина списка равна или меньше параметру `rows` из запроса, или максимально 50, если параметр не указан
bills[].id|Integer|Идентификатор счета в QIWI Кошельке
bills[].external_id|String| Идентификатор счета у мерчанта
bills[].creation_datetime|Long|Дата/время создания счета, Unix-time
bills[].expiration_datetime|Long|Дата/время окончания срока действия счета, Unix-time
bills[].sum|Object|Сведения о сумме счета
---|----|----
sum.currency|Integer|Валюта суммы счета
sum.amount|Number|Сумма счета
---|----|----
bills[].status|String | Константа, `READY_FOR_PAY`
bills[].type|String|Константа, `MERCHANT`
bills[].repetitive|Boolean|Служебное поле
bills[].provider|Object|Информация о мерчанте
---|----|----
provider.id|Integer|Идентификатор мерчанта в QIWI
provider.short_name|String|Сокращенное название мерчанта
provider.long_name|String|Полное название мерчанта
provider.logo_url|String|Ссылка на логотип мерчанта
---|----|----
bills[].comment|String|Комментарий к счету
bills[].pay_url|String|Ссылка для оплаты счета на Платежной форме QIWI

## Оплата счета  {#paywallet_invoice}

Метод выполняет безусловную оплату счета без SMS-подтверждения.

<h3 class="request method">Запрос → POST</h3>

~~~shell
curl -X POST \
  --header 'Content-Type: application/json;charset=UTF-8' \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer 68ec21fd52e4244838946dd07ed225a1' \
  -d '{
      "invoice_uid": "1063702405",
      "currency": "643"
    }' \
  'https://edge.qiwi.com/checkout-api/invoice/pay/wallet'
~~~

~~~http
POST /checkout-api/invoice/pay/wallet HTTP/1.1
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
    <li><h3>URL <span>/checkout-api/invoice/pay/wallet</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer <a href="#p2p-token">SecretKey</a></li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>

Обязательные параметры в теле запроса:

Название|Тип|Описание
--------|----|----
invoice_uid | String |ID счета в QIWI; берется из значения `bills[].id` [данных о счете](#invoice_data)
currency|String| Валюта суммы счета; берется из значения `bills[].sum.currency` [данных о счете](#invoice_data)

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

Поле ответа|Тип|Описание
--------|----|----
invoice_status|String|Строка кода статуса оплаты счета, `PAID_STATUS`. Любой другой статус означает неуспех платежной транзакции.
is_sms_confirm|String|Признак подтверждения по SMS

## Отмена неоплаченного счета  {#cancel_invoice}

Метод отклоняет неоплаченный счет, что делает его недоступным для оплаты.

<h3 class="request method">Запрос → POST</h3>

~~~shell
curl -X POST \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer ***' \
  --header 'Content-Type: application/json;charset=UTF-8' \
  -d '{ "id": 1034353453 }' \
  'https://edge.qiwi.com/checkout-api/api/bill/reject'
~~~

~~~http
POST /checkout-api/api/bill/reject HTTP/1.1
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
    <li><h3>URL <span>/checkout-api/api/bill/reject</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Accept: application/json</li>
             <li>Content-type: application/json</li>
             <li>Authorization: Bearer <a href="#p2p-token">SecretKey</a></li>
        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3>
    </li>
</ul>

Обязательный параметр передается в теле запроса в формате JSON:

Название|Тип|Описание
--------|----|----
id | Integer |ID счета для отмены; берется из значения `bills[].id` [данных о счете](#invoice_data)

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
~~~

Успешный ответ содержит HTTP-код `200`.
