# Error Codes {#errors}

API returns the following HTTP codes in case of errors.

HTTP code | API | Description
---|-----|---------
400 | All | Wrong data format of request
401 | All | Wrong API token or token expired
403 | All | Not enough rights of API token for the request
404 | [Payments history](#payments_history), [Transaction info](#txn_info), [Receipt](#payment_receipt) | Transaction not found or no payments with such data
404 | [Balances](#balance), [User's profile](#profile), [Identification](#identification) | Wallet not found
404 | [Callbacks](#webhook) | Active webhook not found
404 | [Pay/Cancel invoice](#paywallet_invoice) | Invoice not found
422 | [Webhook registration](#hook_reg) | Wrong domain/subnet/host in new webhook URL, wrong webhook type or transactions type, or webhook already exists and is active
423 | All | Too many requests, service temporary unavailable
500 | All | Internal service error (webhook URL too long, infrastructure maintenance, resource is unavailable and so on)

<a name="errorCode"/>

The following errors return in `errorCode` field in responses to [payments history](#payments_history) and [transaction info](#txn_info) requests:

errorCode | Description
----|------
0 |OK
3 |Technical error. Repeat the request later
4 |Incorrect format of phone or account number. Check the data
5 |No such number. Check the data and try again
8 |Technical problem on the recipient's bank side. Try again later
57 | Recipient's wallet status not allowing the money transfer. Ask them to enter passport data in QIWI Wallet to increase status level.
131 |Payment type unavailable for your country
166 |Your wallet status not allowing the money transfer. Enter passport data in QIWI Wallet to increase status level.
167 |Recipient's wallet status not allowing the money transfer. Ask them to enter passport data in QIWI Wallet to increase status level.
202 |Technical error. Repeat the payment later
204 |Your wallet status not allowing the cash topup. Enter passport data in QIWI Wallet to increase status level.
220 |Not enough funds. Replenish your wallet
241 |Payment amount must be larger than 1 ruble
242 |Payment amount larger than maximum allowed
254 |Payment amount must be larger than 1 ruble
271 |Technical issue on the recipient's bank side. Try again later
300 |Technical error. Repeat the payment later
303 |Wrong phone number - enter 10 digits
319 |Your wallet status not allowing the money transfer. Enter passport data in QIWI Wallet to increase status level.
407 |Not enough funds on your card
408 |You already have the same payment - pay or cancel it
455 |Payment is not possible due to limit on minimum balance
461 |Time to confirm operation is expired. Try again later
472 |Not enough funds on your wallet - replenish it
500 |Technical error on the recipient's bank side. Contact the bank's Support service
522 |Recipient's card wrong number or expiration date. Check data and try again
547 |Recipient's card wrong expiration date. Check data and try again
548 |Recipient's card expired
558 |Payment amount larger than maximum allowed
561 |Bank where the money is transferring does not accept the payment. Contact the bank's Support service
700 |Limit for your wallet current status is exceeded. Increase your status level or check your current limit in [Profile section](https://qiwi.com/settings)
702 |Payment is not possible due to recipient's limit. Its balance's limit is exceeded. Recipient has to contact with our Support
704 |The monthly limit on your wallet has been exceeded. To remove the restrictions, increase your wallet status in [Profile section](https://qiwi.com/settings)
705 |The monthly limit on your wallet has been exceeded. To remove the restrictions, increase your wallet status in [Profile section](https://qiwi.com/settings)
710 | Transfer is not possible - the weekly limit of payments for the same recipient have been exceeded
711 | Transfer is not possible. You have exceeded the monthly payment limit for such transactions
716 |You have exceeded the monthly limit on withdrawals from the card. To remove the restrictions, increase your wallet status in [Profile section](https://qiwi.com/settings)
717 |You have exceeded the daily limit on withdrawals from the card. To remove the restrictions, increase your wallet status in [Profile section](https://qiwi.com/settings)
746 |Transfer is impossible - the limit for the same recipient has been exceeded
747 |Transfer is not possible. The number of operations for the same recipient has been exceeded
749 |Technical error. Contact our Support
750 |Technical error. Repeat the payment later
757 |The limit on the number of payments has been exceeded. To remove the restrictions, increase your wallet status in [Profile section](https://qiwi.com/settings)
797 |Payment has been cancelled, the money is returned to your wallet
852 |Transfer is impossible - the limit for the same recipient has been exceeded
866 |Payment not processed. Limit on outgoing transfers has been exceeded - 5 000 RUB from RUB, USD, EUR accounts to KZT monthly. Increase your wallet status in [Profile section](https://qiwi.com/settings) and pay without limits
867 |Payment not processed. Limit on incoming transfers has been exceeded - 5 000 RUB from RUB, USD, EUR accounts to KZT monthly. Increase your wallet status in [Profile section](https://qiwi.com/settings) and pay without limits
893 |Transfer rejected. Date is expired
901 |The code to confirm the payment has expired. Repeat the payment
943 |The limit on transfers per month is exceeded. Increase your wallet status in [Profile section](https://qiwi.com/settings) and transfer without restrictions
1050 |The limit on such operations is exceeded. Increase your wallet status in [Profile section](https://qiwi.com/settings) and expand your options
7000 |Payment rejected. Check card's details and repeat the payment
7600 |Payment rejected. Contact the bank that issued the card
 |
 |
 |
