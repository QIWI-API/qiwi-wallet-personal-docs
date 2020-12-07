---
title: QIWI Wallet API

search: true

metatitle: QIWI Wallet API

metadescription: QIWI Wallet API  allows you to access your QIWI Wallet account information and make payment operations as well as get payment reports and many more.

category: apiqiwiwallet

language_tabs:
  - shell: cURL
  - php: PHP
  - python: Python
  - http: Request/Response
  
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
*[token]: String for user authentication in API by OAuth 2.0 standard, see RFC 6749, RFC 6750.

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

1. Open <https://qiwi.com/api> page in your browser. You will need to register or authenticate on QIWI Wallet service. Click on **Выпустить новый токен** (please note, interface is Russian only). 
    
    ![Token Issue](/images/apiwallet_get_token.jpg)
2. Select token scopes in the pop-up window and click **Продолжить**:
    * Запрос информации о профиле кошелька - allows use of [person's profile requests](#profile), [identification API](#identification), [limits API](#limits)
    * Запрос баланса кошелька - allows [balance requests](#balance)
    * Просмотр истории платежей - allows [payments history requests](#payments)
    * Проведение платежей без SMS - allows making [payment requests](#payments) with no SMS confirmation (regardless the settings <https://qiwi.com/settings/options/security.action>), using [invoicing's API](#pay_invoice) and [notification service](#webhook)
    ![Token Scopes](/images/apiwallet_token_scopes.jpg)
3. Confirm a token issue and click **Продолжить**.
     
    ![Token Scopes](/images/apiwallet_confirm.jpg)
4. Enter confirmation code from SMS sent to the phone number of your wallet.
     
     ![Token Accept](/images/apiwallet_token_sms.jpg)
4. Copy token string and save it in secure place. [Use the token](#auth_ex) in all QIWI Wallet API requests which require authorization.
     
     ![Token](/images/apiwallet_token_final.jpg)

<aside class="success">Token is valid 180 days from this issuing. You can block the token before its lifetime ends using <a href="https://qiwi.com/settings/apps">this link</a>.</aside>

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
