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

- `biller_info` description:
  - `Name`: Name of the service.
  - `PmtType`: Payment type (e.g., `POST`).
  - `ServiceType`: Type of the service (e.g., `BILLS`).
  - `ServiceName`: Name of the bill.
  - `BillTypeAcctLabel`: Label for the account.
  - `IsHidden`: Whether the service is hidden.
  - `PaymentRules`: Rules governing payment.
  - `BillTypeStatus`: Status of the bill type.
  - `AllowRctRePrint`: Allow receipt reprint.
  - `OTPEnabled`: Whether OTP is enabled.
  - `ValidationEnabled`: Whether validation is required.
  - `Timeout`: Timeout rule.
  - `OTPRequired`: Whether OTP is required.
  - `IsTermsConditionReq`: Whether terms and conditions are required.

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


