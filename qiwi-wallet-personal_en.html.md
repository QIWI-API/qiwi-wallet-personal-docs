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
 - <a href='https://t.me/qiwi_api_help_bot'>P2P-operations support</a>
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

# General Information

###### [Propose corrections on GitHub](https://github.com/QIWI-API/qiwi-wallet-personal-docs/)

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

# API Access {#api-access}

<span style="font-weight:bold;color:red;">We have stopped issuing QIWI OAuth-tokens.</span>

Main URL address to call API methods (unless explicitly stated):

`https://edge.qiwi.com`

To make a successful request of API methods, you need:

* Correct HTTP headers: `Accept` and `Content-Type`, as designated in a method description.
* URL composed according to the method reference
* OAuth token issued for your QIWI Wallet. Some requests do not require it though.

# Authorization {#auth_api}

## Authorization data {#auth_data}

<span style="font-weight:bold;color:red;">We have stopped OAuth token issue.</span>

QIWI Wallet API implements OAuth 2.0 open authorization protocol specification. A user registers or authenticates on <https://qiwi.com> QIWI Wallet site and requests a token with a certain scopes. Token issue is confirmed by SMS code.

<aside class="success">Token is valid 180 days from this issuing. You can block the token before its lifetime ends using <a href="https://qiwi.com/settings/apps">this link</a>.</aside>

## Authorization example {#auth_ex}

<span style="font-weight:bold;color:red;">We have stopped OAuth token issue.</span>

~~~shell
curl "server address" \
  --header "Authorization: Bearer jMyN22DQxMjM6NDUzRmRnZDQ0Mw11212e"
~~~

<aside class="notice">
Add `Authorization` HTTP-header to the request with its value as "Bearer <a href="#auth_data">token</a>"
</aside>

* As a result of authentication in [QIWI Wallet site](https://qiwi.com/api), you [got](#auth_data) the token:

`U1QtOTkwMTAyLWNud3FpdWhmbzg3M`

* Add the token to `Authorization: Bearer ` HTTP header.

* The header has to be added to each API request:

`Authorization: Bearer U1QtOTkwMTAyLWNud3FpdWhmbzg3M`
