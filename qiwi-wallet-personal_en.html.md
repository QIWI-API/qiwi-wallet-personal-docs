---
title: QIWI Wallet API

search: true

metatitle: QIWI Wallet API

metadescription: QIWI Wallet API  allows you to access your QIWI Wallet account information and make payment operations as well as get payment reports and many more.

toc_footers:
- <a href='/en/'>Main page</a>
- <a href='mailto:api_help@qiwi.com'>Feedback</a>
- <a href='/sandbox/index.html'>Interactive API</a>
 
 includes:
  - qiwi-wallet-personal/profile_en
  - qiwi-wallet-personal/payment_history_en
  - qiwi-wallet-personal/balance_en
  - qiwi-wallet-personal/payment_en
  - qiwi-wallet-personal/webhook_en
  - qiwi-wallet-personal/error_en
---

*[Token]: String for user authentication in API by OAuth 2.0 standard RFC 6749, RFC 6750.
*[token]: String for user authentication in API by **OAuth 2.0** standard, see RFC 6749, RFC 6750.

# Introduction

###### Last update: 2020-07-28 | [Edit on GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/qiwi-wallet-personal_en.html.md)

QIWI Wallet API makes it easy to automate getting info on your account's state in [QIWI Wallet service](https://qiwi.com) and making financial operations.

API uses HTTPS requests and JSON-formatted responses. 

API methods are accessible after the user is registered in [QIWI Wallet service](https://qiwi.com).

## Service data {#auth_param}

<ul class="nestedList params">
    <li><h3>Authorization</h3>
    </li>
</ul>

Parameter|Description|Type
 ---------|--------|---
 token | [Token](#auth_data) to authorize API requests. Token is valid within one month after its [issuing](#auth_data). | String

# API Access 

Main URL address to call API methods (unless explicitly stated):

`https://edge.qiwi.com`

To make a successful request of API methods, you need:

* Correct HTTP headers: `Accept` and `Content-Type`, as designated in a method description.
* URL composed according to the method reference
* OAuth token issued for your QIWI Wallet. Some requests do not require it though.

# Authorization {#auth_api}

## Authorization data {#auth_data}

QIWI Wallet API implements OAuth 2.0 open authorization protocol specification. A user registers or authenticates on <https://qiwi.com> QIWI Wallet site and requests a token with a certain scopes. Token issue is confirmed by SMS code.

**How to get a token**

1. Open <https://qiwi.com/api> page in your browser. You will need to register or authenticate on QIWI Wallet service.
  ![Token Issue](/images/apiwallet_get_token.jpg)
2. Click on **Выпустить новый токен** (please note, interface is Russian only). Select token scopes in the pop-up window:
    * Запрос информации о профиле кошелька - allows use of [person's profile requests](#profile), [identification API](#identification), [limits API](#limits)
    * Запрос баланса кошелька - allows [balance requests](#balance)
    * Просмотр истории платежей - allows [payments history requests](#payments)
    * Проведение платежей без SMS - allows making [payment requests](#payments) with no SMS confirmation (regardless the settings <https://qiwi.com/settings/options/security.action>), using [invoicing's API](#pay_invoice) and [notification service](#webhook)
  ![Token Scopes](/images/apiwallet_token_scopes.jpg)
3. Click **Продолжить**, confirm a token issuing and enter confirmation code from SMS.
  ![Token Accept](/images/apiwallet_token_sms.jpg)
4. Copy token string and save it in secure place. [Use the token](#auth_ex) in all QIWI Wallet API requests which require authorization.

  ![Token](/images/apiwallet_token_final.jpg)

<aside class="success">Token is valid 180 days from this issuing. You can block the token before its lifetime ends using this link <https://qiwi.com/settings/apps>.</aside>

## Authorization example {#auth_ex}

~~~shell
user@server:~$ curl "server address"
  --header "Authorization: Bearer jMyN22DQxMjM6NDUzRmRnZDQ0Mw11212e"
~~~

<aside class="notice">
Add <code>Authorization</code> header to the request with its value as "Bearer <a href="#auth_data">token</a>"
</aside>


* As a result of authentication in [QIWI Wallet site](https://qiwi.com/api), you [got](#auth_data) the token:

`U1QtOTkwMTAyLWNud3FpdWhmbzg3M`

* Add the token to `Authorization: Bearer ` HTTP header.

* The header has to be added to each API request:

`Authorization: Bearer U1QtOTkwMTAyLWNud3FpdWhmbzg3M`

# QIWI Wallet Balances {#balance}

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
                "title": "QIWI Wallet"
            },
            "hasBalance": true,
            "balance": {
                "amount": 8.74,
                "currency": 643
            },
            "currency": 643
        },
        {
            "alias": "qw_wallet_usd",
            "fsAlias": "qb_wallet",
            "title": "WALLET",
            "type": {
                "id": "WALLET",
                "title": "QIWI Wallet"
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
                "title": "QIWI Wallet"
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
currency | Number| Код валюты баланса (number-3 ISO-4217). Возвращаются балансы в следующих валютах: 643 - российский рубль, 840 - американский доллар, 978 - евро
type|Object|Сведения о счете
id, title| String| Описание счета
balance|Object |Сведения о балансе данного счета
amount|Decimal|Текущий баланс данного счета
currency | Number| Код валюты баланса (number-3 ISO-4217)

# Payments

## Commission rates {#commission}

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
    * 99 - Перевод на QIWI Wallet
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


## Peer-to-Peer QIWI Wallet Transfer {#p2p}

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

## Cell phone top-up {#cell}

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

### Wireless operator check {#mnp}

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


## Card transfer {#cards}

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

### Card system check {#card_check}

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

## Wire transfer {#banks}

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


# Error Codes

В случае ошибки API возвращается HTTP-код ошибки.

Код | API | Описание
---|-----|---------
401 | Все | Ошибка авторизации. Неверный токен или истек срок действия токена.
404 | История платежей | Не найдена транзакция или отсутствуют платежи с указанными признаками
404 | Балансы, Профиль пользователя | Не найден кошелек
423 | История платежей | Слишком много запросов, сервис временно недоступен
