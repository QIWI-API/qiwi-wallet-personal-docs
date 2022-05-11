
# Уведомления (вебхуки) {#webhook}

###### Последнее обновление: 2020-07-06 | [Предложить правки на GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/_webhook_ru.html.md)

> Исходящие платежи - платеж в проведении

~~~http
POST /some-hook.php HTTP/1.1
Accept: application/json
Content-type: application/json
Host: example.com

{"hash": "50779a03d90c4fa60ac44dfd158dbceec0e9c57fa4cf4f5298450fdde1868945",
 "hookId": "f57f95e2-149f-4278-b2cb-4114bc319727",
 "messageId": "f9a197a8-26b6-4d42-aac4-d86b789c373c",
 "payment": {"account": "myAccount",
             "comment": "Комментарий",
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
Host: example.com

{"hash": "50779a03d90c4fa60ac44dfd158dbceec0e9c57fa4cf4f5298450fdde1868945",
 "hookId": "f57f95e2-149f-4278-b2cb-4114bc319727",
 "messageId": "6e2a0e32-4c8d-4fe2-9eed-fe3b6a726ff4",
 "payment": {"account": "masterDre",
             "comment": "Комментарий",
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
Host: example.com

{"hash": "0637b07b1018d76585db26b0f8077016b12996006429e22a7dc5b6982710a1ef",
 "hookId": "f57f95e2-149f-4278-b2cb-4114bc319727",
 "messageId": "1133873b-9bb6-4adb-9bfe-7be3a9aa999f",
 "payment": {"account": "borya241203",
             "comment": "Комментарий",
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
Host: example.com

{"hash": "a56ed0090fa3fd2fd0b002ed80f85a120037a6a85f840938888275e1631da96f",
 "hookId": "8c79f60d-0272-476b-b120-6e7629467328",
 "messageId": "bba24947-ab5f-4b33-881b-738fc3a4c9e1",
 "payment": {"account": "79042426915",
             "comment": "Пополнение кошелька",
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

Хуки или уведомления с данными о событии (платеже/пополнении) отправляются на ваш сервер. В настоящее время поддерживаются только вебхуки (webhook) - сообщения, адресованные веб-сервисам. Для приема вебхуков вам необходимо настроить свой сервер на прием и обработку POST-запросов ([Формат запросов](#hook_format)).

**От вашего сервера успешный ответ 200 OK на входящий запрос должен поступить в течение 1-2 сек. Не дождавшись ответа, сервис КИВИ отправляет еще одно уведомление через 10 минут, потом еще одно через 1 час.**

Пулы IP-адресов, с которых сервисы QIWI отправляют webhook:

* 79.142.16.0/20
* 195.189.100.0/22
* 91.232.230.0/23
* 91.213.51.0/24

Если ваш сервер обработки вебхуков работает за брандмауэром, необходимо добавить эти IP-адреса в список разрешенных адресов входящих TCP-пакетов.

## Быстрый старт {#quick_hook}

0. Реализуйте веб-сервис обработки [запросов](#hook_format). Особое внимание обратите на реализацию проверки подписи.
1. [Зарегистрируйте свой обработчик](#hook_reg). **Внимание! Длина оригинального (не URL-encoded) адреса сервиса обработчика не должна превышать 100 символов.**
2. Запросите [ключ проверки подписи](#hook_key).
3. Протестируйте прием запросов вашим обработчиком с помощью [тестового запроса](#hook_test). На зарегистрированный в п.2 сервис придет пустое уведомление.

Чтобы сменить адрес сервера для обработки вебхуков:

1. [Удалите обработчик вебхуков](#hook_remove).
2. [Зарегистрируйте новый обработчик](#hook_reg). **Внимание! Длина оригинального (не URL-encoded) адреса сервиса обработчика не должна превышать 100 символов.**
3. Запросите [ключ проверки подписи](#hook_key) для нового обработчика.
4. Протестируйте прием запросов новым обработчиком с помощью [тестового запроса](#hook_test). На зарегистрированный в п.2 сервис придет пустое уведомление.

## Обработка вебхука {#hook_format}

Каждый вебхук посылает уведомления - входящие POST-запросы с JSON-объектом, содержащим данные об одном платеже. Схема объекта:

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

// Base64 encoded ключ для дешифровки вебхуков (метод /hook/{hookId}/key)

$NOTIFY_PWD = "JcyVhjHCvHQwufz+IHXolyqHgEc5MoayBfParl6Guoc=";

// Вычисляем хэш SHA-256 строки параметров и шифруем с ключом для веб-хуков

$reqres = hash_hmac("sha256", $Request[0], base64_decode($NOTIFY_PWD));

// Проверка подписи вебхука

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

# Base64 encoded ключ для расшифровки вебхука (/hook/{hookId}/key)
webhook_key_base64 = 'JcyVhjHCvHQwufz+IHXolyqHgEc5MoayBfParl6Guoc='

# строка параметров
data = '643|1|IN|+79161112233|13353941550'

webhook_key = base64.b64decode(bytes(webhook_key_base64,'utf-8'))
print(hmac.new(webhook_key, data.encode('utf-8'), hashlib.sha256).hexdigest())
~~~

Поле | Тип | Описание
----|------|-------
hookId | String (UUID) | Уникальный id хука
messageId | String (UUID) | Уникальный id уведомления
payment | Object | Данные платежа
payment.txnId | String | ID транзакции в процессинге QIWI Wallet
payment.account | String | Для платежей - номер счета получателя. Для пополнений - номер отправителя, терминала или название агента пополнения кошелька
payment.signFields | String | Список полей объекта `payment` (через `,`), которые хешируются алгоритмом HmacSHA256 для проверки уведомления (см. параметр `hash`)
payment.personId | Integer | Номер кошелька
payment.date | String DateTime | Дата/время платежа, в московской временной зоне. Формат даты `ГГГГ-ММ-ДД'T'чч:мм:сс+03:00`
payment.errorCode | String | [Код ошибки платежа](#errorCode)
payment.type | String | Тип платежа. Возможные значения:<br>`IN` - пополнение, <br>`OUT` - платеж
payment.status|String|Статус платежа. Возможные значения:<br>`WAITING` - платеж проводится,<br>`SUCCESS` - успешный платеж,<br>`ERROR` - ошибка платежа.
payment.provider | Integer| ID провайдера QIWI Wallet
payment.comment | String | Комментарий к транзакции
payment.sum | Object | Данные о сумме платежа или пополнения. Параметры:
sum.amount|Number(Decimal)|Сумма
sum.currency|Number(3)|Код валюты
payment.commission|Object| Данные о комиссии для платежа или пополнения. Параметры:
commission.amount|Number(Decimal)|Сумма
commission.currency|Number(3)|Код валюты
payment.total|Object|Данные об итоговой сумме платежа или пополнения. Параметры:
total.amount|Number(Decimal)|Сумма
total.currency|Number(3)|Код валюты
test|Boolean|Признак тестового сообщения
version|String|Версия API
hash|String| Хэш цифровой подписи уведомления

### Как проверить подпись уведомления

 Реализуйте следующие шаги:

 1. Возьмите значения полей из списка в `payment.signFields` уведомления (**в том же порядке**) в формате String.
 2. Объедините значения в строку с разделителями `|`.
 3. Зашифруйте строку п.2 алгоритмом SHA-256 с [ключом проверки подписи](#hook_key).
 4. Сравните полученное значение со значением поля `hash` уведомления.

Пример расшифровки подписи (см. также функцию PHP на вкладке справа):

1. По [запросу](#hook_key) пользователь получает ключ вебхука, закодированный в Base64: 
    `JcyVhjHCvHQwufz+IHXolyqHgEc5MoayBfParl6Guoc=`
2. Приходит уведомление 
   `{"messageId":"7814c49d-2d29-4b14-b2dc-36b377c76156","hookId":"5e2027d1-f5f3-4ad1-b409-058b8b8a8c22",
   "payment":{"txnId":"13353941550","date":"2018-06-27T13:39:00+03:00","type":"IN","status":"SUCCESS","errorCode":"0","personId":78000008000,"account":"+79161112233","comment":"","provider":7,
   "sum":{"amount":1,"currency":643},
   "commission":{"amount":0,"currency":643},
   "total":{"amount":1,"currency":643},
   "signFields":"sum.currency,sum.amount,type,account,txnId"},
   "hash":"76687ffe5c516c793faa46fafba0994e7ca7a6d735966e0e0c0b65eaa43bdca0","version":"1.0.0","test":false}`
3. Склеиваются требуемые поля платежных данных (указаны в `payment.signFields` - `sum.currency,sum.amount,type,account,txnId`):
   `643|1|IN|+79161112233|13353941550`
4. Поля шифруются методом SHA-256 с Base64-раскодированным ключом из п.1. Результат 
   `76687ffe5c516c793faa46fafba0994e7ca7a6d735966e0e0c0b65eaa43bdca0`
   совпадает с параметром `hash` из запроса.

## Регистрация обработчика вебхуков {#hook_reg}

<h3 class="request method">Запрос → PUT</h3>

~~~shell
curl -X PUT "https://edge.qiwi.com/payment-notifier/v1/hooks?hookType=1&param=http%3A%2F%2Fexample.com%2Fcallbacks%2F&txnType=2" \
     -H "accept: */*" \
     -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
PUT /payment-notifier/v1/hooks?hookType=1&param=http%3A%2F%2Fexample.com%2Fcallbacks%2F&txnType=2 HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/payment-notifier/v1/hooks?<a>parameter=value</a></span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>

        </ul>
    </li>
</ul>

<ul class="nestedList params">
    <li><h3>Параметры</h3><span>Параметры передаются в query запроса. Все параметры обязательны.</span>
    </li>
</ul>

Название|Тип|Описание
----|-----|------
hookType|Integer|Тип хука. Только 1 - вебхук.
param|URL-encoded|Адрес сервера обработки вебхуков. Длина исходного (не URL-encoded) адреса — не более 100 символов. **URL обработчика должен быть доступен из Интернета**.
txnType|String|Тип транзакций, по которым будут включены уведомления. Возможные значения:<br>0 - только входящие транзакции (пополнения);<br>1 - только исходящие транзакции (платежи);<br>2 - все транзакции

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "hookId":"d63a8729-f5c8-486f-907d-9fb8758afcfc",
  "hookParameters":{
    "url":"http://example.com/callbacks/"
  },
  "hookType":"WEB",
  "txnType":"BOTH"
}
~~~

Ответ в формате JSON.

Название|Тип|Описание
-----|------|------
hookId|String|UUID созданного вебхука
hookParameters|Object|Набор параметров вебхука (только URL)
hookType|String|Тип вебхука (только WEB)
txnType|String|Тип транзакций, по которым отсылаются уведомления (`IN` - входящие, `OUT` - исходящие, `BOTH` - все)

## Удаление обработчика вебхуков  {#hook_remove}

<h3 class="request method">Запрос → DELETE</h3>

~~~shell
curl -X DELETE "https://edge.qiwi.com/payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc" \
   -H "accept: */*" \
   -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
DELETE /payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
        <li><h3>URL <span>/payment-notifier/v1/hooks/<a>hookId</a></span></h3></li>
        <ul>
             <li><strong>hookId</strong> - UUID вебхука</li>
        </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>

        </ul>
    </li>
</ul>


<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "response":"Hook deleted"
}
~~~

Формат ответа JSON.

Название|Тип|Описание
-----|------|------
response|String|Описание результата операции

## Получение секретного ключа {#hook_key}

Каждое уведомление содержит цифровую подпись сообщения, зашифрованную ключом. Используйте запрос для получения ключа проверки подписи.

<h3 class="request method">Запрос → GET</h3>


~~~shell
curl -X GET "https://edge.qiwi.com/payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/key" \
   -H "accept: */*" \
    -H "accept: */*" \
    -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
GET /payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/key HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~


<ul class="nestedList url">
    <li><h3>URL <span>/payment-notifier/v1/hooks/<a>hookId</a>/key</span></h3></li>
    <ul>
    <li><strong>hookId</strong> - UUID вебхука</li>
    </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "key":"L8UVF3JkLVUr6r70LiE0A9/5WoGGwWKG2pI/e+l/9fs="
}
~~~

Формат ответа JSON.

Название|Тип|Описание
-----|------|------
key|String|Base64-закодированный ключ

## Изменение секретного ключа  {#hook_secret}

Для смены ключа шифрования уведомлений используйте этот запрос.

<h3 class="request method">Запрос → POST</h3>


~~~shell
curl -X POST "https://edge.qiwi.com/payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/newkey" \
   -H "accept: */*" \
   -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
POST /payment-notifier/v1/hooks/d63a8729-f5c8-486f-907d-9fb8758afcfc/newkey HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~


<ul class="nestedList url">
    <li><h3>URL <span>/payment-notifier/v1/hooks/<a>hookId</a>/newkey</span></h3></li>
    <ul>
    <li><strong>hookId</strong> - UUID вебхука</li>
    </ul>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "key":"OikS4/CcIbSf+yYGnLbnOige8RGoYmGxs/LNMwkJy7Q="
}
~~~

Формат ответа JSON.

Название|Тип|Описание
-----|------|------
key|String|Base64-закодированный новый ключ

## Данные об обработчике уведомлений  {#hook_active}

Список действующих (активных) обработчиков уведомлений, связанных с вашим кошельком, можно получить данным запросом.

Так как сейчас используется только один тип хука - вебхук, то в ответе содержится только один объект данных.

<h3 class="request method">Запрос → GET</h3>

~~~shell
curl -X GET "https://edge.qiwi.com/payment-notifier/v1/hooks/active" \
   -H "accept: */*" \
   -H "accept: */*" \
   -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
GET /payment-notifier/v1/hooks/active HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/payment-notifier/v1/hooks/active</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "hookId":"d63a8729-f5c8-486f-907d-9fb8758afcfc",
  "hookParameters":{
    "url":"http://example.com/callbacks/"
  },
  "hookType":"WEB",
  "txnType":"BOTH"
}
~~~

Формат ответа JSON.

Название|Тип|Описание
-----|------|------
hookId|String|UUID действующего обработчика вебхуков
hookParameters|Object|Набор параметров обработчика (только URL)
hookType|String|Тип вебхука (только WEB)
txnType|String|Тип транзакций, по которым отсылаются уведомления (`IN` - входящие, `OUT` - исходящие, `BOTH` - все)

## Отправка тестового уведомления  {#hook_test}

Для проверки вашего обработчика вебхуков используйте этот запрос. Тестовое уведомление отправляется на адрес, указанный в [параметрах действующего обработчика](#hook_active).

<h3 class="request method">Запрос → GET</h3>

~~~shell
curl -X GET "https://edge.qiwi.com/payment-notifier/v1/hooks/test" \
   -H "accept: */*" \
   -H "authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f"
~~~

~~~http
GET /payment-notifier/v1/hooks/test HTTP/1.1
Host: edge.qiwi.com
Authorization: Bearer 3b7beb2044c4dd4a8f4588d4a6b6c93f
User-Agent: ****
~~~

<ul class="nestedList url">
    <li><h3>URL <span>/payment-notifier/v1/hooks/test</span></h3></li>
</ul>

<ul class="nestedList header">
    <li><h3>HEADERS</h3>
        <ul>
             <li>Authorization: Bearer ***</li>
             <li>Accept: application/json</li>
        </ul>
    </li>
</ul>

<h3 class="request">Ответ ←</h3>

~~~http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "response":"Webhook sent"
}
~~~

Формат ответа JSON.

Название|Тип|Описание
-----|------|------
response|String|Результат запроса
