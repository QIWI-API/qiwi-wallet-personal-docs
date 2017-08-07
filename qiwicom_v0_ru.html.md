---
title: QIWI Wallet API

search: true

metatitle: API QIWI Кошелька

metadescription: API QIWI Кошелька позволяет автоматизировать выполнение платежей и получение отчетов о платежах, информации о счёте.

toc_footers:
 - <a href='/'>На главную</a>
 - <a href='mailto:api_help@qiwi.com'>Обратная связь</a>
---

# Введение

API QIWI Кошелька позволяет автоматизировать получение информации о вашем счёте в [сервисе Visa QIWI Кошелек](https://qiwi.com) и проводить операции с его помощью.

Методы API доступны после регистрации пользователя в [сервисе Visa QIWI Кошелек](https://qiwi.com). 

## Служебные данные {#auth_param}

<ul class="nestedList params">
    <li><h3>Авторизация</h3>
    </li>
</ul>

Параметр|Описание|Тип
 ---------|--------|---
 st-ticket | [ST-ticket](#sts) для авторизации запроса в API. Каждый запрос к API требует выпуска нового st-ticket | Integer
 service-name | Строка `https://qiwi.com/j_cas_security_check`| String
 SECRET_KEY | Base64 хешированный набор параметров вида: `st-ticket:service-name`| String


# Авторизация {#auth_api}

Для авторизации используется [SECRET_KEY](#auth_param), уникальный для каждого запроса к API.<br>

~~~shell
user@server:~$ curl "адрес сервера"
  --header "Authorization: ST MjMyNDQxMjM6NDUzRmRnZDQ0Mw=="
~~~

<aside class="notice">
Заголовок представляет собой параметр Authorization, значение которого представлено как: "ST <a href="#auth_param">SECRET_KEY</a>"
</aside>

## Пример авторизации

* В результате авторизации в QIWI Wallet [получен](#tgts) TGT-ticket.

* По TGT-ticket [получен](#sts) ST-ticket.

`ST-990102-cnwqiuhfo8732q4f`

* Составьте строку

`ST-990102-cnwqiuhfo8732q4f:https://qiwi.com/j_cas_security_check`

* Закодируйте строку преобразованием *Base64()*.

`Base64("ST-990102-cnwqiuhfo8732q4f:https://qiwi.com/j_cas_security_check") = U1QtOTkwMTAyLWNud3FpdWhmbzg3MzJxNGY6aHR0cHM6Ly9xaXdpLmNvbS9qX2Nhc19zZWN1cml0eV9jaGVjaw==`

* Закодированная строка добавляется в заголовок `Authorization: ST `

* Итоговый заголовок: 

`Authorization: ST U1QtOTkwMTAyLWNud3FpdWhmbzg3MzJxNGY6aHR0cHM6Ly9xaXdpLmNvbS9qX2Nhc19zZWN1cml0eV9jaGVjaw==`

## Данные для авторизации {#auth_data}

Авторизация использует стандарт CAS. См. <https://apereo.github.io/cas/5.0.x/protocol/CAS-Protocol-Specification.html>.

### Этап 1. tgt-ticket {#tgts}

TGT (ticket-granting ticket) выпускается при успешной авторизации клиента сервером CAS. Данный тикет используется многократно для [выпуска st-ticket](#sts).

~~~shell
user@server:~$ сurl "https://auth.qiwi.com/cas/tgts"
  -X POST -v 
  -d '{ "login":"<login>","password":"<password>"}' 
  --header "Accept: application/vnd.qiwi.sso-v1+json" 
  --header "Content-Type: application/json" 
~~~

~~~http
POST /cas/tgts HTTP/1.1
Accept: application/vnd.qiwi.sso-v1+json
Content-Type: application/json
Host: auth.qiwi.com
User-Agent: ****

{ "login":"<wallet_login>","password":"<wallet_password>"}
~~~

Тип запроса - POST. 

URL запроса:

`https://auth.qiwi.com/cas/tgts`

Заголовки запроса:

* `Accept: application/vnd.qiwi.sso-v1+json`
* `Content-Type: application/json`

Тело запроса - JSON. Параметры запроса: 

Параметр|Тип|Описание
--------|----|----
login | String |Номер QIWI Кошелька (с международным кодом `7`)
password|String| Пароль от QIWI Кошелька

TGT содержится в поле `ticket` JSON-ответа.

~~~json
{
  "entity": {
    "user": "<wallet_login>",
    "ticket": "TGT-***"
  },
  "links": [
    {
      "rel": "sts",
      "href": "https://auth.qiwi.com/cas/sts"
    }
  ]
}
~~~

Тикет действует в течение пользовательской сессии работы с API. 

<aside class="notice">
Максимальное время жизни TGT-тикета составляет 8 часов. При отсутствии <a href="#sts">запросов ST-ticket</a> сессия завершается через 2 часа.
</aside>

Для проверки наличия действующего TGT-ticket можно использовать запрос:

`GET /cas/tgts`

~~~shell
user@server:~$ сurl "https://auth.qiwi.com/cas/tgts"
  -v --header "Accept: application/vnd.qiwi.sso-v1+json" 
  --header "Content-Type: application/json" 
~~~

~~~http
GET /cas/tgts HTTP/1.1
Host: auth.qiwi.com
User-Agent: ****
~~~

В ответе в Cookie `CASTGC` возвращается действующий TGT-тикет.

Для завершения действия тикета (завершения сессии) необходимо отправить запрос:

`DELETE /cas/tgts`

~~~shell
user@server:~$ сurl "https://auth.qiwi.com/cas/tgts"
  -X DELETE -v 
  -d '{"service":"https://qiwi.com/j_spring_cas_security_check","ticket":"TGT-11223-***"}' 
  --header "Accept: application/vnd.qiwi.sso-v1+json" 
  --header "Content-Type: application/json" 
~~~

### Этап 2. st-ticket {#sts}

ST-ticket (service ticket) выпускается по действующему [TGT-тикету](#tgts). 

<aside class="notice">
На каждый запрос к API необходимо получать новый ST-ticket и использовать его для <a href="#auth_api">авторизации</a>.
</aside>

~~~shell
user@server:~$ curl -X POST -v "https://auth.qiwi.com/cas/sts
  -d '{ "service":"https://qiwi.com/j_spring_cas_security_check","ticket":"TGT-***"}' 
  --header "Accept: application/vnd.qiwi.sso-v1+json" 
  --header "Content-Type: application/json" 
~~~

~~~http
POST /cas/sts HTTP/1.1
Accept: application/vnd.qiwi.sso-v1+json
Content-Type: application/json
Host: auth.qiwi.com
User-Agent: ****

{ "service":"https://qiwi.com/j_spring_cas_security_check","ticket":"TGT-***"}
~~~

Тип запроса - POST. 

URL запроса:

`https://auth.qiwi.com/cas/sts`

Заголовки запроса:

* `Accept: application/vnd.qiwi.sso-v1+json`
* `Content-Type: application/json`

Тело запроса - JSON. Параметры запроса: 

Параметр|Тип|Описание
--------|----|----
service | String |Идентификатор сервиса.<br>Строка`https://qiwi.com/j_spring_cas_security_check`.
ticket|String| Полученный [TGT-ticket](#tgts)

ST-ticket содержится в параметре `ticket` ответа.

~~~json
{
  "entity": {
    "ticket": "ST-***"
  },
  "links": []
}
~~~


# Комиссионные тарифы {#commission}

Для провайдеров, оплата которых доступна через API, можно предварительно проверить комиссионные условия.

## Стандартные комиссии

Общие комиссионные условия (стандартные тарифы) можно получить для провайдеров следующим запросом.

Тип запроса - GET.

URL запроса:

`https://edge.qiwi.com/sinap/providers/{id}/form`

~~~shell
user@server:~$ curl "https://edge.qiwi.com/sinap/providers/99/form"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: ST YUu2qw048gtdsvlk3iu=="
  --header "User-Agent: ***"
~~~

~~~http
POST /sinap/providers/99/form HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: ST YUu2qw048gtdsvlk3iu==
Host: edge.qiwi.com
Origin: qiwi.com
User-Agent: ****
~~~

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: ST ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

Параметр URL запроса:

* `{id}` - идентификатор провайдера. Возможные значения:
    * 99 - Перевод на Visa QIWI Wallet
    * 1963 - Перевод на карту Visa (карты российских банков)
    * 21013 - Перевод на карту MasterCard (карты российских банков)
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
         ...,
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

## Сложные комиссии

Сложные комиссии на платеж (с учетом всех тарифов) можно получить для заданного набора платежных данных следующим запросом.

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/providers/{id}/onlineCommission`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/providers/99/onlineCommission"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: ST YUu2qw048gtdsvlk3iu=="
  --header "User-Agent: ***"
  -d '{"account":"79165238345","paymentMethod":{"type":"Account","accountId":"643"},
       "purchaseTotals":{"total":{"amount":"10","currency":"643"}}}'
~~~

~~~http
POST /sinap/providers/99/onlineCommission HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: ST YUu2qw048gtdsvlk3iu==
Host: edge.qiwi.com
User-Agent: ****

{"account":"79165238345","paymentMethod":{"type":"Account","accountId":"643"},"purchaseTotals":{"total":{"amount":"10","currency":"643"}}}
~~~

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: ST ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

Параметр URL запроса:

* `{id}` - идентификатор провайдера. Возможные значения:
    * 99 - Перевод на Visa QIWI Wallet
    * 1963 - Перевод на карту Visa (карты российских банков)
    * 21013 - Перевод на карту MasterCard (карты российских банков)
    * 466 - Тинькофф Банк
    * 464 - Альфа-Банк
    * 821 - Промсвязьбанк
    * 815 - Русский Стандарт
    * [Идентификаторы операторов мобильной связи](#mnp)

Тело запроса - JSON. Параметры запроса:

Параметр|Тип|Описание
--------|----|----
account| String|Номер телефона (без международного префикса) или номер карты/счета получателя, в зависимости от провайдера
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
type|String |Тип платежа, только `Account`
accountId|String| Идентификатор счета, только `643`.
purchaseTotals | Object |Объект с платежными данными
total|Object| Объект, содержащий данные о сумме платежа: 
amount|Decimal|Сумма
currency|String|Валюта (только `643`, рубли)

### Формат ответа

В параметре ответа `data.body.qwCommission.amount` возвращается сумма комиссии.

~~~json
{
...
  "data": {
    "body": {
     ...
      "qwCommission": {
        "amount": 100,
        "currency": "643"
      }
      ...
    },
    "status": 200
  }
}
~~~


# Перевод на QIWI Кошелек {#p2p}

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/terms/99/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/terms/99/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: ST YUu2qw048gtdsvlk3iu=="
  --header "User-Agent: ***"
  -d '{"id":"11111111111111", "sum": {"amount":100, "currency":"643"}, "source": "account_643", 
       "paymentMethod": {"type":"Account", "accountId":"643"},
       "comment":"test", "fields": {"account":"+79121112233"}}'
~~~

~~~http
POST /sinap/terms/99/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: ST YUu2qw048gtdsvlk3iu==
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

<aside class="notice">
Для выполнения запроса необходимо отключить <a href="https://qiwi.com/settings/options/security.action">подтверждение платежей по SMS</a>.
</aside>

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: ST ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).
 
Тело запроса - JSON. Параметры запроса:

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр)
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


## Формат ответа

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

# Оплата сотовой связи {#cell}

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/terms/{id}/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/terms/1/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: ST YUu2qw048gtdsvlk3iu=="
  --header "User-Agent: ***"
  -d '{"id":"11111111111111", "sum": {"amount":100, "currency":"643"}, "source": "account_643", 
       "paymentMethod": {"type":"Account", "accountId":"643"}, "fields": {"account":"79161112233"}}'
~~~

~~~http
POST /sinap/terms/1/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: ST YUu2qw048gtdsvlk3iu==
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
        "account":"79161112233"
      }
}
~~~

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: ST ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

<aside class="notice">
Для выполнения запроса необходимо отключить <a href="https://qiwi.com/settings/options/security.action">подтверждение платежей по SMS</a>.
</aside>

Параметр URL запроса:

* `{id}` - идентификатор провайдера. Определяется с помощью [проверки мобильного оператора](#mnp).
 
Тело запроса - JSON. Все перечисленные параметры обязательны: 

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр)
sum|Object| Объект, содержащий данные о сумме платежа: 
amount|Decimal|Сумма
currency|String|Валюта (только `643`, рубли)
source| String| Источник фондирования платежа. Допускается только следующее значение:<br>`account_643` - рублевый счет QIWI Кошелька отправителя
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
type|String |Тип платежа, только `Account`
accountId|String| Идентификатор счета, только `643`.
fields|Object| Объект с параметрами перевода. Допускается только следующий параметр:
account| String|Номер мобильного телефона получателя

## Формат ответа

~~~json
{
    "id": "21131343",
  "fields": {
          "account": "79161112233"
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

## Определение оператора {#mnp}

Предварительная проверка мобильного номера выполняется POST-запросом:

`https://qiwi.com/mobile/detect.action`

В ответе возвращается идентификатор оператора для [запроса пополнения телефона](#cell).

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

Запрос не требует авторизации. Параметр запроса:

Параметр|Тип|Описание
--------|----|----
phone | URL-String |Мобильный номер в международном формате

### Формат ответа

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


# Перевод на карту {#cards}

Выполняет перевод на карты платежных систем Visa или MasterCard. Предварительно можно [проверить тип платежной системы](#card_check).

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/terms/{id}/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/terms/1963/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: ST YUu2qw048gtdsvlk3iu=="
  --header "User-Agent: ***"
  -d '{"id":"21131343", "sum": {"amount":1000, "currency":"643"}, "source": "account_643", 
       "paymentMethod": {"type":"Account", "accountId":"643"}, "fields": {"account":"4256********1231"}}'
~~~

~~~http
POST /sinap/terms/1963/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: ST YUu2qw048gtdsvlk3iu==
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
* `Authorization: ST ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

<aside class="notice">
Для выполнения запроса необходимо отключить <a href="https://qiwi.com/settings/options/security.action">подтверждение платежей по SMS</a>.
</aside>

Параметр URL запроса:

* `{id}` - идентификатор платежной системы:
    * 1963 - Visa (карты российских банков)
    * 21013 - MasterCard (карты российских банков)

Тело запроса - JSON. Параметры запроса: 

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр)
sum|Object| Объект, содержащий данные о сумме платежа: 
amount|Decimal|Сумма
currency|String|Валюта (только `643`, рубли)
source| String| Источник фондирования платежа. Допускается только следующее значение:<br>`account_643` - рублевый счет QIWI Кошелька отправителя
paymentMethod | Object| Объект, определяющий обработку платежа процессингом. Содержит следующие параметры:
type|String |Тип платежа, только `Account`
accountId|String| Идентификатор счета, только `643`.
fields|Object| Объект с параметрами перевода. Допускается только следующий параметр:
account| String|Номер банковской карты получателя

## Формат ответа

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

## Проверка карты {#card_check}

Определение платежной системы карты выполняется POST-запросом:

`https://qiwi.com/card/detect.action`

В ответе возвращается идентификатор платежной системы для [запроса перевода на карту](#cards).

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

Запрос не требует авторизации. Параметр запроса:

Параметр|Тип|Описание
--------|----|----
cardNumber | String |Немаскированный номер карты

### Формат ответа

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

# Банковский перевод {#banks}

Выполняет перевод на банковские карты/счета российских банков. Предварительно можно [проверить номер карты/счета](#bank_check).

Тип запроса - POST.

URL запроса:

`https://edge.qiwi.com/sinap/terms/{id}/payments`

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/terms/466/payments"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: ST YUu2qw048gtdsvlk3iu=="
  --header "User-Agent: ***"
  -d '{"id":"21131343", "sum": {"amount":1000,"currency":"643"}, "source": "account_643", 
       "paymentMethod": {"type":"Account","accountId":"643"}, 
       "fields": {"account_type": "1","account":"4256********1231","exp_date": "MMYY"}}'
~~~

~~~http
POST /sinap/terms/466/payments HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: ST YUu2qw048gtdsvlk3iu==
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
* `Authorization: ST ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

<aside class="notice">
Для выполнения запроса необходимо отключить <a href="https://qiwi.com/settings/options/security.action">подтверждение платежей по SMS</a>.
</aside>

Параметр URL запроса:

* `{id}` - идентификатор банка. Возможные значения:
    * 466 - Тинькофф Банк
    * 464 - Альфа-Банк
    * 821 - Промсвязьбанк
    * 815 - Русский Стандарт

Тело запроса - JSON. Параметры запроса: 

Параметр|Тип|Описание
--------|----|----
id | String |Клиентский ID транзакции (максимум 20 цифр)
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


## Формат ответа

~~~json
{
  "id": "21131343",
  "fields": {
          "account": "4256********1231"
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

## Проверка идентификатора {#bank_check}

Предварительная проверка номера карты/счета выполняется POST-запросом:

`https://edge.qiwi.com/sinap/terms/{id}/validations`

Заголовки, параметр URL запроса и параметры JSON-тела запроса идентичны [переводу на банковский счет](#banks).

~~~shell
user@server:~$ curl -X POST "https://edge.qiwi.com/sinap/terms/466/validations"
  --header "Content-Type: application/json"
  --header "Accept: application/json"
  --header "Authorization: ST YUu2qw048gtdsvlk3iu=="
  --header "User-Agent: ***"
  -d '{"id":"21131343", "sum": {"amount":1000, "currency":"643"}, "source": "account_643", 
       "paymentMethod": {"type":"Account", "accountId":"643"}, 
       "fields": {"account_type": "1", "account":"4256********1231", "exp_date": "MMYY"}}'
~~~

~~~http
POST /sinap/terms/466/validations HTTP/1.1
Content-Type: application/json
Accept: application/json
Authorization: ST YUu2qw048gtdsvlk3iu==
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
        "account_type": "1",
        "account":"4256********1231",
        "exp_date": "MMYY"
      }
}
~~~

### Формат ответа

Успешная проверка - Ответ с HTTP Status 200.

~~~json
{
 "code": {
        "_name": "NORMAL", 
        "value": "0"},
 "data": {
        "body": {}, 
        "status": 200},
 "message": null,
 "messages": null
}
~~~

Неверный номер счета/карты - Ответ с HTTP Status 400.

~~~json
{
 "code": {
        "_name": "NORMAL", 
        "value": "0"},
 "data": {
        "body": {
          "code": "QWPRC-542",
          "message": "Неверный номер карты получателя"
        }, 
        "status": 400},
 "message": null,
 "messages": null
}
~~~


# История платежей {#payments}

История платежей и пополнений вашего кошелька.

Тип запроса - GET.

~~~shell
Пример 1. Последние 10 платежей

user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments?rows=10"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: ST YUu2qw048gtdsvlk3iu=="

Пример 2. Платежи за 10.05.2017

user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments?rows=50&startDate=2017-05-10T00%3A00%3A00&endDate=2017-05-10T23%3A59%3A59"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: ST YUu2qw048gtdsvlk3iu=="

Пример 3. Продолжение списка платежей (в предыдущем запросе истории возвращены параметры nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23)

user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12:35:23"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: ST YUu2qw048gtdsvlk3iu=="
~~~

~~~text
Пример 1. Последние 10 платежей с рублевого баланса и с привязанной карты
~~~

~~~http
GET /payment-history/v1/persons/79112223344/payments?rows=10&operation=OUT&sources[0]=QW_RUB&sources[1]=CARD HTTP/1.1
Accept: application/json
Authorization: ST YUu2qw048gtdsvlk3iu==
Content-type: application/json
Host: edge.qiwi.com
~~~

~~~text
Пример 2. Платежи за 10.05.2017
~~~

~~~http
GET /payment-history/v1/persons/79112223344/payments?rows=50&startDate=2017-05-10T00%3A00%3A00&endDate=2017-05-10T23%3A59%3A59 HTTP/1.1
Accept: application/json
Authorization: ST YUu2qw048gtdsvlk3iu==
Content-type: application/json
Host: edge.qiwi.com
~~~

~~~text
Пример 3. Продолжение списка платежей (в предыдущем запросе истории возвращены параметры nextTxnId=9103121 и nextTxnDate=2017-05-11T12:35:23)
~~~

~~~http
GET /payment-history/v1/persons/79112223344/payments?rows=50&nextTxnId=9103121&nextTxnDate=2017-05-11T12:35:23 HTTP/1.1
Accept: application/json
Authorization: ST YUu2qw048gtdsvlk3iu==
Content-type: application/json
Host: edge.qiwi.com
~~~

URL запроса:

`https://edge.qiwi.com/payment-history/v1/persons/<wallet>/payments?<parameters>`

Параметры URL запроса:

* `<wallet>` - номер кошелька, для которого получена авторизация (с международным префиксом, но без `+`), **обязательный параметр**
* `<parameters>` - дополнительные параметры запроса (см. ниже)

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: ST ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

Параметры запроса: 

Параметр|Тип|Описание
--------|----|----
rows | Integer |Число платежей в ответе, для разбивки отчета на части. Целое число от 1 до 50. **Обязательный параметр**
operation|String| Тип операций в отчете, для отбора. Допустимые значения:<br>`ALL` - все операции, <br>`IN` - только пополнения, <br>`OUT` - только платежи, <br>`QIWI_CARD` - только платежи по картам QIWI (QVC, QVP). <br>По умолчанию `ALL`
sources|Array[String]|Источники платежа, для отбора. Каждый источник задается как отдельный параметр и нумеруется элементом массива, начиная с нуля (`sources[0]`, `sources[1]` и т.д.). Допустимые значения: <br>`QW_RUB` - рублевый счет кошелька, <br>`QW_USD` - счет кошелька в долларах, <br>`CARD` - привязанные и непривязанные к кошельку банковские карты, <br>`MK` - счет мобильного оператора. Если не указаны, учитываются все источники
startDate | DateTime URL-encoded| Начальная дата поиска платежей (формат ГГГГ-ММ-ДДTчч:мм:ссZ). По умолчанию, равен вчерашней дате. Используется только вместе с `endDate`
endDate | DateTime URL-encoded | Конечная дата поиска платежей (формат ГГГГ-ММ-ДДTчч:мм:ссZ). По умолчанию, равен текущей дате. Используется только вместе с `startDate`
nextTxnDate | DateTime URL-encoded| Дата транзакции (формат ГГГГ-ММ-ДДTчч:мм:ссZ), для отсчета от предыдущего списка (см. параметр `nextTxnDate` в ответе). Используется только вместе с `nextTxnId`
nextTxnId | Long | Номер предшествующей транзакции, для отсчета от предыдущего списка (см. параметр `nextTxnId` в ответе). Используется только вместе с `nextTxnDate`


<aside class="notice">Максимальный допустимый интервал между startDate и endDate - 90 календарных дней.</aside>

## Формат ответа

~~~json
{"data":
	[
		{
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
                       "id":26476,
                       "shortName":"Yandex.Money",
                       "longName":"ООО НКО \"Яндекс.Деньги\"",
                       "logoUrl":"https://static.qiwi.com/img/providers/logoBig/26476_l.png",
                       "description":"Яндекс.Деньги",
                       "keys":"***",
                       "siteUrl":null
                      },
		"comment":null,
		"currencyRate":1,
		"extras":null,
		"chequeReady":true,
		"bankDocumentAvailable":false,
		"bankDocumentReady":false,
                "repeatPaymentEnabled":false
		}
	],
	"nextTxnId":9001,
	"nextTxnDate":"2017-01-31T15:24:10+03:00"
}
~~~

Успешный ответ содержит список платежей из истории кошелька, соответствующих заданному фильтру:

Параметр|Тип|Описание
--------|----|----
data|Array[Object]|Массив платежей. <br>Число платежей равно параметру `rows` из запроса
txnId | Integer |ID транзакции в процессинге
personId|Integer|Номер кошелька
date|DateTime|Дата/время платежа (в формате ГГГГ-ММ-ДДTчч:мм:ссTMZ)
errorCode|Integer|Код ошибки платежа
error| String| Описание ошибки
status|String|Статус платежа. Возможные значения:<br>`WAITING` - платеж проводится, <br>`SUCCESS` - успешный платеж, <br>`ERROR` - ошибка платежа.
type | String| Тип платежа (соответствует параметру запроса `operation`)
statusText|String |Описание статуса платежа
trmTxnId|String|Клиентский ID транзакции
account| String|Номер счета получателя
sum|Object| Объект с суммой платежа. Параметры:
amount|Decimal|сумма, 
currency|String|валюта 
commission|Object| Объект с комиссией платежа. Параметры:
amount|Decimal|сумма, 
currency|String|валюта 
total|Object| Объект с общей суммой платежа. Параметры:
amount|Decimal|сумма, 
currency|String|валюта 
provider|Object| Объект с описанием провайдера. Параметры:
id|Integer|ID провайдера в процессинге,
shortName|String|краткое наименование провайдера,
longName|String|развернутое наименование провайдера,
logoUrl|String|ссылка на логотип провайдера,
description|String|описание провайдера (HTML),
keys|String|список ключевых слов,
siteUrl|String|сайт провайдера
comment|String|Комментарий к платежу
currencyRate|Decimal|Курс конвертации (если применяется в транзакции)
extras|Object|Дополнительные поля платежа, список не фиксирован
chequeReady| Boolean|Специальное поле
bankDocumentAvailable|Boolean|Специальное поле
bankDocumentReady|Boolean|Специальное поле
repeatPaymentEnabled|Boolean|Специальное поле
nextTxnId|Integer|ID следующей транзакции в полном списке
nextTxnDate|DateTime|Дата/время следующей транзакции в полном списке

## Статистика платежей {#stat}

Для получения статистики по суммам платежей за заданный период используется подзапрос запроса истории.

~~~shell
user@server:~$ curl "https://edge.qiwi.com/payment-history/v1/persons/79112223344/payments/total?startDate=2017-03-01T00%3A00%3A00%2B03%3A00Z&endDate=2017-03-31T11%3A44%3A15%2B03%3A00Z"
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: ST ***"
~~~

~~~http
GET /payment-history/v1/persons/79112223344/payments/total?startDate=2017-03-01T00%3A00%3A00%2B03%3A00Z&endDate=2017-03-31T11%3A44%3A15%2B03%3A00Z HTTP/1.1
Accept: application/json
Authorization: ST ***
Content-type: application/json
Host: edge.qiwi.com
~~~

Тип запроса - GET.

URL запроса:

`https://edge.qiwi.com/payment-history/v1/persons/<wallet>/payments/total?<parameters>`

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: ST ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

Параметры URL запроса:

* `<wallet>` - номер кошелька, для которого получена авторизация (с международным префиксом, но без знака "+"), **обязательный параметр**
* `<parameters>` - дополнительные параметры запроса:

Параметр|Тип |Описание
--------|----|-------
operation|String| Тип операций, учитываемых в отчете. Допустимые значения:<br>`ALL` - все операции, <br>`IN` - только пополнения, <br>`OUT` - только платежи, <br>`QIWI_CARD` - только платежи по картам QIWI (QVC, QVP). <br>По умолчанию `ALL`.
sources|Array[String]|Источники платежа, учитываемые в отчете. Каждый источник задается как отдельный параметр и нумеруется элементом массива, начиная с нуля (`sources[0]`, `sources[1]` и т.д.). Допустимые значения: <br>`QW_RUB` - рублевый счет кошелька, <br>`QW_USD` - счет кошелька в долларах, <br>`CARD` - привязанные и непривязанные к кошельку банковские карты, <br>`MK` - счет мобильного оператора. Если не указан, учитываются все источники платежа.
startDate|DateTime URL-encoded | Начальная дата периода статистики (в формате ГГГГ-ММ-ДДTчч:мм:ссZ). **Обязательный параметр.**
endDate|DateTime URL-encoded| Конечная дата периода статистики (в формате ГГГГ-ММ-ДДTчч:мм:ссZ). **Обязательный параметр.**

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
amount | Decimal |Сумма пополнений 
currency|String|Валюта пополнений
outgoingTotal|Array[Object]|Данные об исходящих платежах, отдельно по каждой валюте
amount | Decimal |Сумма платежей 
currency|String|Валюта платежей


# Баланс QIWI Кошелька {#balance}

~~~shell
user@server:~$ curl "https://edge.qiwi.com/funding-sources/v1/accounts/current
  --header "Accept: application/json" 
  --header "Content-Type: application/json"
  --header "Authorization: Token ST-***"
~~~

~~~http
GET /funding-sources/v1/accounts/current HTTP/1.1
Accept: application/json
Authorization: ST ***
Content-type: application/json
Host: edge.qiwi.com
~~~

Тип запроса - GET.

URL запроса:

`https://edge.qiwi.com/funding-sources/v1/accounts/current`

Заголовки запроса:

* `Accept: application/json`
* `Content-Type: application/json`
* `Authorization: ST ***`

В заголовке `Authorization` указываются [параметры авторизации](#auth_api).

## Формат ответа

~~~json
{
  "accounts": [
    {
      "fsAlias": "qb_wallet",
      "title": "Qiwi Account",
      "type": {
        "id": "WALLET",
        "title": "Visa QIWI Wallet"
      },
      "currencyId": 643,
      "balance": {
        "amount": 3.15,
        "currencyId": 643
      }
    }
  ]
}
~~~

Успешный ответ содержит список источников фондирования платежей и балансы:

Параметр|Тип|Описание
--------|----|----
accounts|Array[Object]|Массив балансов
fsAlias | String |Название банковского баланса
title|String|Название счета кошелька
type|Object|Сведения о счете
id, title| String| Описание счета
currencyId | Integer| Код валюты счета (number-3 ISO-4217)
balance|Object |Сведения о балансе данного счета
amount|Decimal|Текущий баланс данного счета
currencyId | Integer| Код валюты счета (number-3 ISO-4217)

# Коды ошибок

В случае ошибки API возвращается HTTP-код ошибки.

Код | API | Описание
---|-----|---------
401 | Все | Ошибка авторизации
404 | История платежей | Не найдена транзакция или отсутствуют платежи с указанными признаками
404 | Балансы | Не найден кошелек
423 | История платежей | Слишком много запросов, сервис временно недоступен
