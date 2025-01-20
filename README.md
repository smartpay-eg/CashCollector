# SmartPay Payment Services API Guide

## Overview

This guide provides detailed instructions for integrating and using the SmartPay Payment Services API. It includes endpoints for user management, service interactions, payments, wallets, and transactions, as well as explanations of request formats, headers, and response structures.

---

## Authentication

### Get Token

**Path:** `/api/auth/token`

**Method:** `GET`

**Headers:**

- `login: <username>` (The username for authentication.)
- `password: <password>` (The password for authentication.)
- `machine_serial: <machine_serial>` (A unique identifier for the machine, typically read from the POS.)

**Request:** None

**Response:**

```json
{
    "uid": 30,
    "user_context": {
        "lang": "ar_AA",
        "tz": "Africa/Cairo",
        "uid": 30
    },
    "company_id": 1,
    "access_token": "access_token_4bfdb0a31410c799c3514d5033b044c36287a348",
    "expires_in": "86400",
    "Force_Update": false
}
```

---

### Get Profile

**Path:** `/api/get_user_profile`

**Method:** `POST`

**Headers:**

- `charset: utf-8`
- `access_token: <access_token>`
- `Content-Type: application/x-www-form-urlencoded`

**Request:**

```text
fields=["name","phone","ref"]
```

**Response:**

```json
{
    "count": 1,
    "data": [
        {
            "id": 99,
            "name": "Staging user",
            "phone": "01028855777",
            "ref": "000011"
        }
    ]
}
```

---

## Services

### Get Categories

**Path:** `/api/get_sevice_categories`

**Method:** `POST`

**Headers:**

- `charset: utf-8`
- `access_token: <access_token>`
- `Content-Type: application/x-www-form-urlencoded`

**Request:**

```text
lang=ar_AA
limit=0
offset=0
```

- `lang`: Language code (e.g., `ar_AA` for Arabic or `en_US` for English).
- `limit`: Maximum number of records to retrieve.
- `offset`: Number of records to skip.

**Response:**

```json
{
    "count": 8,
    "data": [
        {
            "id": 205,
            "image": "/web/image?model=product.category&field=image_medium&id=205",
            "name": "شحن رصيد"
        },
        {
            "id": 13,
            "image": "/web/image?model=product.category&field=image_medium&id=13",
            "name": "كروت الشحن"
        }
    ]
}
```

---

### Get Service Providers

**Path:** `/api/get_sevice_billers`

**Method:** `POST`

**Headers:**

- `charset: utf-8`
- `access_token: <access_token>`
- `Content-Type: application/x-www-form-urlencoded`

**Request:**

```text
lang=ar_AA
domain=[("parent_id.id", "=", 205)]
offset=0
limit=0
```

- `lang`: Language code.
- `domain`: Filter to retrieve providers under a specific category (e.g., `205`). If empty, retrieves all available providers.
- `offset`: Number of records to skip.
- `limit`: Maximum number of records to retrieve.

**Response:**

```json
{
    "count": 4,
    "data": [
        {
            "id": 6,
            "categ_id": 205,
            "categ_name": "Mobile Recharge",
            "image": "/web/image?model=product.category&field=image_medium&id=6",
            "name": "فودافون"
        }
    ]
}
```

---

### Get Services

**Path:** `/api/get_sevices`

**Method:** `POST`

**Headers:**

- `charset: utf-8`
- `access_token: <access_token>`
- `Content-Type: application/x-www-form-urlencoded`

**Request:**

```text
lang=ar_AA
domain=[("categ_id.id", "=", 926)]
offset=0
limit=0
```

- `lang`: Language code.
- `domain`: Filter to retrieve services under a specific provider ID. If empty, retrieves all services.
- `offset`: Number of records to skip.
- `limit`: Maximum number of records to retrieve.

**Response:**

```json
{
    "count": 1,
    "data": [
        {
            "id": 21,
            "categ_id": 21,
            "categ_name": "Orange",
            "image": "/web/image?model=product.template&field=image_medium&id=21",
            "name": "فاتورة اورنج",
            "biller_info": {
                "Name": "فاتورة اورنج",
                "PmtType": "POST",
                "ServiceType": "BILLS",
                "ServiceName": "فواتير الموبايل",
                "BillTypeAcctLabel": "رقم المحمول",
                "AcctInputMethod": "KP",
                "BillTypeExtraRefKeys": [
                    {
                        "key": "contract_number",
                        "label": "Contract Number",
                        "input_method": "KP",
                        "required": true
                    },
                    {
                        "key": "service_area",
                        "label": "Service Area",
                        "input_method": "SC",
                        "required": false
                    }
                ],
                "IsHidden": false,
                "PaymentRules": {
                    "IsInqRqr": false,
                    "IsMobNtfy": false,
                    "IsFracAcpt": true,
                    "IsPrtAcpt": false,
                    "IsOvrAcpt": false,
                    "IsAdvAcpt": true,
                    "IsAcptCardPmt": true
                },
                "BillTypeStatus": "Available",
                "AllowRctRePrint": true,
                "OTPEnabled": false,
                "ValidationEnabled": false,
                "Timeout": "SRETRY",
                "OTPRequired": false,
                "IsTermsConditionReq": false
            }
        }
    ]
}
```

- **`biller_info` field descriptions:**
  - `Name`: Name of the service.
  - `PmtType`: Payment type:
    - `POST`: Post-payment, requires inquiry.
    - `PREP`: Prepaid payment, check `PaymentRules.IsInqRqr` to determine if an inquiry is required.
    - `VOCH`: Voucher, payment is direct without inquiry.
  - `ServiceType`: Type of the service (e.g., `BILLS`).
  - `ServiceName`: Name of the bill.
  - `BillTypeAcctLabel`: Label for the primary input field (POST and PREP types only).
  - `AcctInputMethod`: Input method for the primary field:
    - `KP`: Keypad (default).
    - `SC`: Smart card (e.g., for prepaid utilities).
    - `GSC`: Gas smart card.
    - `WSC`: Water smart card.
  - `BillTypeExtraRefKeys`: Array of additional input fields, each with:
    - `key`: Field key.
    - `label`: Field label.
    - `input_method`: Input method (e.g., `KP`, `SC`).
    - `required`: Whether the field is mandatory.
  - `IsHidden`: Whether the service is hidden.
  - `PaymentRules`: Rules governing payment.
    - `IsInqRqr`: Whether inquiry is required before payment.
    - Other rules define acceptance criteria (e.g., fractional amounts, advanced payments).
  - `BillTypeStatus`: Status of the bill type.
  - `AllowRctRePrint`: Whether receipt reprinting is allowed.
  - `OTPEnabled`: Whether OTP is enabled.
  - `ValidationEnabled`: Whether validation is required.
  - `Timeout`: Timeout rule for the transaction.
  - `OTPRequired`: Whether OTP is required.
  - `IsTermsConditionReq`: Whether terms and conditions must be acknowledged.
Here's a clean table representation of your data for better readability:

| **Element Name**         | **Type**      | **Cardinality** | **Description**                                   |
|---------------------------|---------------|------------------|-------------------------------------------------|
| **PmtType**              | Enum          | +                | POST, PREP, VOCH                                |
| **BillTypeAcctLabel**    | String        | +                |                                                 |
| **IsHidden**             | Boolean       | +/-              |                                                 |
| **AcctInputMethod**      | Enum          | +/-              |                                                 |
| **BillTypeExtraRefKeys** | Array         | +/-              |                                                 |
| **BillTypeRefKey**       | Object        | -                |                                                 |
| **BillTypeRefKey -> Key** | String       | +                |                                                 |
| **BillTypeRefKey -> Label** | String     | +                |                                                 |
| **BillTypeRefKey -> Required** | Boolean | +                |                                                 |
| **BillTypeRefKey -> IsCnfrmRequired** | Boolean | +/-       |                                                 |
| **BillTypeRefKey -> InputMethod** | Enum  | +/-              |                                                 |
| **BillTypeRefKey -> IsInputMasked** | Boolean | +/-          |                                                 |
| **PaymentRules**         | Object        | +/-              |                                                 |
| **PaymentRules -> IsInqRqr** | Boolean   | +/-              |                                                 |
| **PaymentRules -> IsFracAcpt** | Boolean | +/-              | Default: false                                  |
| **PaymentRules -> IsPrtAcpt** | Boolean | +/-              |                                                 |
| **PaymentRules -> IsOvrAcpt** | Boolean | +/-              |                                                 |
| **PaymentRules -> IsAdvAcpt** | Boolean | +/-              |                                                 |
| **PaymentRules -> IsAcptCardPmt** | Boolean | +/-            | Default: false                                  |
| **Type**                 | Enum          | +/-              | CASHOUT / CASHININT                             |
| **PaymentRanges**        | Object        | +/-              |                                                 |
| **PaymentRanges -> PaymentRangeType** | Array | +/-          |                                                 |
| **PaymentRanges -> PaymentRangeType -> Lower** | Object | +   |                                                 |
| **PaymentRanges -> PaymentRangeType -> Lower -> Amt** | Decimal | + |                                                 |
| **PaymentRanges -> PaymentRangeType -> Upper** | Object | +   |                                                 |
| **PaymentRanges -> PaymentRangeType -> Upper -> Amt** | Decimal | + |                                                 |
| **HasCorrelation**       | Boolean       | +/-              | Default: false                                  |
---

## Payments

### Service Inquiry

**Path:** `/api/create_machine_request`

**Method:** `POST`

**Headers:**

- `charset: utf-8`
- `access_token: <access_token>`
- `machine_serial: <machine_serial>`
- `Content-Type: application/x-www-form-urlencoded`

**Request:**

```text
requestType=service_bill_inquiry
productId=3007
billingAcct=27905071700083
lang=ar_AA
```

- `requestType`: Inquiry type (e.g., `service_bill_inquiry`).
- `productId`: ID of the product/service.
- `billingAcct`: Billing account number.
- `lang`: Language code.

**Response:**

```json
{
    "count": 17,
    "data": {
        "message": "Service Bill Inquiry request was submitted successfully.",
        "provider": "Smart pay",
        "statusCode": 200,
        "statusDescription": "Success",
        "trxNumber": "2411123471021621",
        "additionalInfo": "Details of the inquiry."
    }
}
```

---

### Pay Service

**Path:** `/api/create_machine_request`

**Method:** `POST`

**Headers:**

- `charset: utf-8`
- `access_token: <access_token>`
- `machine_serial: <machine_serial>`
- `Content-Type: application/x-www-form-urlencoded`

**Request:**

```text
requestType=pay_service_bill
productId=23
billingAcct=01110000000
trxAmt=20
inquiryTrxNumber=2411123471661622
feesAmt=5.0
pmtAmts=[{"Sequence": 1, "AmtDue": 20}]
```

- `requestType`: Payment request type.
- `productId`: ID of the product/service.
- `billingAcct`: Billing account number.
- `trxAmt`: Transaction amount.
- `inquiryTrxNumber`: Inquiry transaction number.
- `feesAmt`: Fees associated with the transaction.
- `pmtAmts`: List of amounts to be paid.

**Response:**

```json
{
    "count": 19,
    "data": {
        "message": "Pay Service Bill request was submitted successfully.",
        "trxNumber": "2411123471681622",
        "billingAcct": "01110000000",
        "trxAmt": 20.0,
        "feesAmt": 5.0
    }
}
```

---

### Cancel Payment

**Path:** `/api/cancel_request`

**Method:** `POST`

**Headers:**

- `charset: utf-8`
- `access_token: <access_token>`
- `machine_serial: <machine_serial>`
- `Content-Type: application/x-www-form-urlencoded`

**Request:**

```text
inquiryTransactionId=2410108603009
```

- `inquiryTransactionId`: ID of the inquiry transaction to be canceled.

**Response:**

```json
{
    "count": 47,
    "data": "Cancel REQ Number (2410109043011) successfully!"
}
```

---

## Wallets

### Get Wallet Balance

**Path:** `/api/get_my_wallets`

**Method:** `POST`

**Headers:**

- `charset: utf-8`
- `access_token: <access_token>`
- `machine_serial: <machine_serial>`

**Request:** None

**Response:**

```json
{
    "count": 2,
    "data": [
        {
            "id": 1324,
            "name": "Cash wallet",
            "wallet_balance": 964.58,
            "available_amount": 864.58
        }
    ]
}
```

---

### Transfer Wallet Recharge

**Path:** `/api/recharge_mobile_wallet`

**Method:** `POST`

**Headers:**

- `charset: utf-8`
- `access_token: <access_token>`
- `machine_serial: <machine_serial>`

**Request:**

```text
transfer_to=000011
trans_amount=500
wallet_id=77
wallet_dest_id=43
```

- `transfer_to`: Reference of the user to whom the wallet recharge is directed.
- `trans_amount`: Amount to be transferred.
- `wallet_id`: Source wallet ID.
- `wallet_dest_id`: Destination wallet ID.

**Response:**

```json
{
    "count": 121,
    "data": "Wallet for User (Test user) recharged successfully with amount 500.0 EGP."
}
```

---

### Get Wallet Transactions

**Path:** `/api/get_wallet_trans`

**Method:** `POST`

**Headers:**

- `charset: utf-8`
- `access_token: <access_token>`
- `machine_serial: <machine_serial>`
- `Content-Type: application/x-www-form-urlencoded`

**Request:**

```text
limit=0
fields=["request_id", "label", "amount", "create_date"]
domain=[("wallet_id","!=",False)]
offset=0
```

- `limit`: Maximum number of transactions to retrieve.
- `fields`: Fields to be included in the response.
- `domain`: Filter criteria for the transactions.
- `offset`: Number of records to skip.

**Response:**

```json
{
    "count": 3,
    "data": [
        {
            "id": 8344871,
            "label": "Pay Service Bill",
            "amount": "25.0",
            "create_date": "2021-11-16T22:36:46.817262"
        }
    ]
}
```

---

### Get Wallet Transaction Summary

**Path:** `/api/get_wallet_trans_summary`

**Method:** `POST`

**Headers:**

- `charset: utf-8`
- `access_token: <access_token>`
- `machine_serial: <machine_serial>`
- `Content-Type: application/x-www-form-urlencoded`

**Request:**

```text
domain=[("create_date", ">=", "2022-11-01"), ("create_date", "<=", "2022-11-27")]
```

- `domain`: Date range for filtering transactions.

**Response:**

```json
{
    "count": 4,
    "data": {
        "open": {
            "opening_balance": {
                "label": "Opening Balance",
                "value": "-839.54"
            }
        },
        "end": {
            "ending_balance": {
                "label": "Ending Balance",
                "value": "-964.58"
            }
        }
    }
}
```

---

