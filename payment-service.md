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

# Sequential Diagram

[![](https://mermaid.ink/img/pako:eNqNlVtP2zAUx7_KkaVJoKX0DjQPSKU3IcGICIxp6oubnLYWSZw5ThlDfPedJE6aXpjWlybx_9x-59h-Z570kdkswV8pRh6OBV8pHs6jeQT0456WCp4SVMV7zJUWnoh5pGE6AZ7A2HXgGReHy45bLt89QwMc_hYifXZRbYSHh_qRjJZitWNjPn1qMh46N0cCzzInZbwZ1_jK3-Ck43Xi0yNqp652-ArhZBjQ2lvzGb011_R0WuLIQDSurqYTm5LiylvDMhCrtQYe-RAr6SH6oOWOu9J2OiFLx7VhNnmEZlwokmaIei39pNB8kxpBblCR2sq0LiaJkBE8yhc0fhyX_BRkbHhArQRusIo49EMRgUoDTKDQf_lSmGSwagYLKV9EtCoTAREtpQq5pmiVXWbSMFlfF3oL7m_GFgxDmUbaAuferbIyYIYbLgK-CBBK11WJRyAG6Ok9Ycn_9IAcRauh-x9k8NXEoL6UiO7yKPPoGJ7vPBA-TUyJB5ogU51o6m_2tuABj8pJ3APkHANZa5kzs2GkMHNe1qsVjxLaYhVzZ_ZPbybqtmRnlpf89HBrQS2YoUu9TlVEf75QGeZcVjqu930XtGNvTVLqVTbQcWW1HWjTx0w_kmEc4LayKhenYSp_iv165URUp9VE0H6Ea-5Rv3wY8SBY0HOxYqAca3_TO1RuG5lrjbRBpUjlJ3Ay-fF4upXXu_coKDzRSJKURqVZYUrzvH1zGlJ-10q-ZlBKRHt5FtwNPQLnZRvVNNDQK_WfjHWzbrJTm3Hv7DCEE_dpNJq4LiU9Hd7cTsamQoJZneKB3p6_qefRDlmmwU4m5Dvrpw1jkcQBnZdVpwp9LXcMku15M6XNXuLZd2XmrzY-Sb4Xd1FQnsxiKyV8ZmuVosVCpOqzV_aeSeZMrzHEObPp0efqZc7m0QfZ0Pn9U8qwNFMyXa2ZveSUn8WKxpnrrPqqKBqqUXZ4Mfti0BrkXpj9zn4zu3fW7Q8GvX6vNTjvdC_b_b7F3pjdHZx1Wu3eRbvbvrgY9Fud8w-L_ckDt89arc5lr3vZP79st_udbs9i1Hy6M--KazW_XT_-AvgkWSk?type=png)](https://mermaid.live/edit#pako:eNqNlVtP2zAUx7_KkaVJRQv0Tts8IJXehAQjIjCmqS9uctpaJHHmOGUM8d13kjhpCmVaX5rE_3P7nWP7lXnSR2azBH-lGHk4FXyjeLiMlhHQj3taKnhIUBXvMVdaeCLmkYb5DHgCU9eBR1x9XHbccvnmEU7B4S8h0mcX1U54-FE_kdFabA5szKdPTaZj5-pI4EXmpIy34Bqf-Qs0Ol4nPjmidupqh28QGuOA1l6aj-htuaankxJHBuL04mI-sykprrwtrAOx2WrgkQ-xkh6iD1oeuCtt5zOydFwbFrN7aMaFImmGqLfSTwrNN6kR5A4Vqa1M62KSCBnBvXxC48dxyU9BxoY71ErgDquIYz8UEag0wAQK_ZcvhUkGq2awkvJJRJsyERDRWqqQa4pW2WUmpybry0Jvwe3V1IJxKNNIW-DculVWBsx4x0XAVwFC6boq8QjEAD39TljyP_lAjqLV0P0PMvhqYlBfSkQ3eZRldAzPdx4InyamxANNkKlONPU3e1vxgEflJL4D5BwDWWuZs7BhojBzXtarFY8S2mIVc2fxT28m6r5kZ5GX_HB3bUEtmKFLvU5VRH--UBnmXFY6rvf9ELRj701S6lU20HFltR9o08dMP5FhHOC-sioXxxT-EPv1wgmoTquBoO0Il9yjdvkw4UGwoudixTA51v2m91G572OuNdJTqkQqP4HG7Mf9yV5eb969oPAEI0lSmpRmRSnN8_bNYUj5XSr5nDEpCb3Ls8Bu4BE3L9unpn8GXqn_ZKqbdZOD2ox754AhNNyHyWTmupT0fHx1PZuaCglmdYgHen_8pp5HG2SdBgeZkO-snTZMRRIHdFxWnSr0tdwxSPbHzZz2eonnvSszfrXpSfKteIiC8mQW2yjhM1urFC0WIlWfvbLXTLJkeoshLplNjz5XT0u2jN7Iho7vn1KGpZmS6WbL7DWn_CxWNM7cZtVXRdFQTbKzi9mDUaufe2H2K_vN7N5Ztz8a9fq91ui80x22-7T6wuzu6KzTavcG7W57MBj1W53zN4v9yQO3z1qtzrDXHQ475KzbG1qMek835k1xqeZ369tfrbRYzg)

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
| `/payments/{paymentId}` | GET | Get payment status from payment gateway | Need to check how to get payment status from 2c2p with provided paymentId
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
