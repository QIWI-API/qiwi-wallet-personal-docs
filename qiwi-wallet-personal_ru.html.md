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

includes:
 - qiwi-wallet-personal/profile_ru
 - qiwi-wallet-personal/payment_history_ru
 - qiwi-wallet-personal/balance_ru
 - qiwi-wallet-personal/payment_ru
 - qiwi-wallet-personal/webhook_ru
 - qiwi-wallet-personal/error_ru

---

 *[Токен]: Символьная строка для аутентификации пользователя в API по стандарту OAuth 2.0 RFC 6749, RFC 6750.
 *[токен]: Символьная строка для аутентификации пользователя в API по стандарту OAuth 2.0 RFC 6749, RFC 6750.
 *[API]: Application Programming Interface -  набор готовых методов, предоставляемых приложением (системой) для использования во внешних программных продуктах.
 *[JSON]: JavaScript Object Notation - текстовый формат обмена данными, основанный на *JavaScript*.



# Введение {#intro}

###### Последнее обновление: 2020-07-28 | [Предложить свои правки на GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/blob/master/qiwi-wallet-personal_ru.html.md)

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

Основной URL-адрес для вызова методов API (если не указано иное):

`https://edge.qiwi.com`

Для успешного вызова методов API необходимы:

* Корректные заголовки `Accept` и `Content-Type`. API QIWI Кошелька поддерживает только один MIME-тип: `application/json`. Любое другое значение приведет к [ошибке формата данных](#errors).
* URL, составленный согласно требованиям к нужному запросу.
* OAuth-токен, выданный вам для доступа к вашему QIWI кошельку. Для некоторых запросов его не потребуется.

## Получение OAuth-токена {#auth_data}

API QIWI Кошелька использует открытый протокол [OAuth 2.0](http://tools.ietf.org/html/rfc6749). Согласно протоколу, пользователь авторизуется или регистрируется на сайте <https://qiwi.com> и запрашивает токен OAuth 2.0 Bearer с правом выполнения определённых действий. Выпуск токена подтверждается одноразовым кодом из СМС.

<aside class="warning">Если у вас уже есть действующий токен, то он автоматически заблокируется при выпуске нового токена</aside>

Для выпуска токена выполните следующие шаги:

1. Откройте в браузере страницу <https://qiwi.com/api>. Для этого потребуется авторизоваться или зарегистрироваться в сервисе QIWI Кошелек. Нажмите **Выпустить новый токен**. 
    
   ![Token Issue](/images/apiwallet_get_token.jpg)
2. Во всплывающем окне выберите разрешения на операции с токеном и нажмите **Продолжить**:
    * Запрос информации о профиле кошелька - выполнение запросов [профиля пользователя](#profile), [идентификации](#identification), [лимитов](#limits)
    * Запрос баланса кошелька - выполнение запросов [баланса](#balance)
    * Просмотр истории платежей - выполнение запросов [истории платежей](#payments_history)
    * Проведение платежей без SMS - выполнение [платежных запросов](#payments) без подтверждения по SMS, [оплата счетов](#pay_invoice), использование [уведомлений](#webhook)
    ![Token Scopes](/images/apiwallet_token_scopes.jpg)
3. Подтвердите согласие на выпуск токена и нажмите **Продолжить**.
    
   ![Token Scopes](/images/apiwallet_confirm.jpg)
4. Укажите проверочный код из SMS-сообщения, отправленного на номер вашего кошелька.
    
   ![Token Accept](/images/apiwallet_token_sms.jpg)
5. Скопируйте строку токена и сохраните в безопасном месте. Используйте токен для [запросов к API QIWI Кошелька](#auth_ex).
    
   ![Token](/images/apiwallet_token_final.jpg)

<aside class="success">Токен действует в течение 180 дней с момента выпуска. Вы можете заблокировать токен, не дожидаясь окончания срока его действия. Для этого удалите доступ к вашему кошельку приложению <b>QIWI API</b> на <a href="https://qiwi.com/settings/apps">этой странице</a>.</aside>

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
