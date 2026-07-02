# Payment Service – Proposed Architecture

## Objective

Provide a common Payment Service that can be used by multiple applications (Booking, Check-in, Manage Booking).

---

# Prerequisite

A **Session Token** is generated during Booking / Check-in / Manage Booking initialization.

All Payment Service APIs require the Session Token in the request header.

The Payment Service uses the Session Token to retrieve:

- officeId
- pointOfSales
- booking information or any additional information required to process the payment

---

# Overall Flow

```text
Booking / Check-in / Manage Booking
                │
                │ Generate Session Token
                ▼
        Perform booking/check-in process
                │
                │ Proceed to payment page
                ▼
        Payment Service
                │
        (1) Get available payment methods (GET /payments/methods)
                ▼
      Frontend displays payment methods
                │
      User selects payment method
      (WeChat Pay / Alipay / etc.)
                ▼
        (2) Create Payment (POST /payments)
                │
        Return Payment Info (Redirect URL, paymentId)
                ▼
      Redirect to Payment Gateway
                │
      ┌─────────┴───────────────────┐
      │                             │
      ▼                             ▼
Gateway Callback             Browser Redirect
 (Backend)                      (Frontend)
      │                             │
(3) Update payment           (4) Confirm payment
    transaction                   status
(POST /payments/callback)     (GET /payment/{paymentId})
      │                             │
      └─────────┬───────────────────┘
                ▼
        Return payment result
```

---

# API Design

## 1. Get Available Payment Methods

Retrieve all available payment methods for the current session.

### Endpoint

```http
GET /payments/methods
```

### Request Header

```http
Session-Token: <token>
```

### Response

```json
[
  {
    "code": "WECHAT",
    "name": "WeChat Pay"
  },
  {
    "code": "ALIPAY",
    "name": "Alipay"
  }
]
```

### Notes

The Payment Service determines the available payment methods based on information from the Session Token, such as:

- Booking / Check-in / Manage Booking
- Point of Sale (POS)
- Office Id
- Business rules

---

## 2. Create Payment

Called after the user selects a payment method.

### Endpoint

```http
POST /payments
```

### Request

```json
{
  "paymentMethod": "WECHAT"
}
```

### Response

```json
{
  "paymentId": "123456",
  "redirectUrl": "https://payment-gateway/..."
}
```

### Flow

1. Frontend calls the API.
2. Payment Service initiate a request to payment gateway to creates a payment transaction.
3. Payment Service returns the Payment Gateway redirect URL and other required payment info.
4. Frontend capture the payment information and redirects the user to the Payment Gateway.

---

## 3. Payment Gateway Callback

Called by the Payment Gateway.

### Endpoint

```http
POST /payments/callback
```

### Responsibilities

- Validate gateway signature
- Verify transaction result
- Update payment record
- Store gateway transaction details
- Update payment status (Success / Failed)

> A single callback endpoint should handle both successful and failed transactions.

---

## 4. Payment Confirmation

Called by the frontend after the browser is redirected back from the Payment Gateway.

### Endpoint

```http
POST /payments/confirmation
```

### Responsibilities

- Verify the latest payment status
- Return the payment result to the frontend

### Example Response

Success

```json
{
  "status": "SUCCESS"
}
```

Failed

```json
{
  "status": "FAILED"
}
```

---

# Failure Flow

If payment fails:

1. Payment Gateway sends the callback to Payment Service.
2. Payment Service updates the payment status to **FAILED**.
3. Browser redirects back to the frontend.
4. Frontend calls the Confirmation API.
5. Payment Service returns **FAILED**.
6. Frontend displays the payment selection page.

---

# Proposed API Summary

| API | Method | Purpose | Remarks |
|------|--------|---------|-----------|
| `/payments/methods` | GET | Retrieve available payment methods | payment method condition can include value like officeId, pointOfSales and etc
| `/payments` | POST | Create a payment and return the Payment Gateway URL | how to generate paymentId?
| `/payments/callback` | POST | Payment Gateway callback to update payment status | Need to check the payload received from payment gateway
| `/payment/{paymentId}` | GET | Get payment status from payment gateway | Need to check how to get payment status from 2c2p with provided paymentId
| `/payments/confirmation` | POST | Frontend verifies the final payment status | call dapi payment-records/confirmation


---

# Design Considerations

## Generic Endpoints for payment methods
Use a single endpoint:

```http
GET /payments/methods
```

The Session Token currently may not contains the business context (Booking, Check-in, or Manage Booking) to identify the source application, 
this information is required to be included in the Session Token in order to support generic endpoint.

---

application-specific endpoints such as:

```http
GET /payment/booking/method
GET /payment/check-in/method
GET /payment/manage-booking/method
```
