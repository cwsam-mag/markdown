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
<!--
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
-->
---

# Overall Custom Payment Flow Diagram

[![](https://mermaid.ink/img/pako:eNqNVmGP2jgQ_SuWpUqgCwtkYQvRCYkFyiGVbVTgqCq-mMSAtY7NOQ7b7Wr_e8eJkw2Q3pZPMTPzZubNGycvOJAhxR6O6X8JFQEdM7JXJNqIjUDwI4GWCq1iqrLzkSjNAnYkQqM13SISo_HCN4_X9vk6N8_X19bx0J9d_-tPTYxPniMKpynR9Ik8o5obuMd6hbdf9vbJnqLakIPtubmmwYFoeKrnrZgmGoMBlOqhBSUqOKAdZ_uDRkSE6KhkQGmItDzDy4MhCmLnaw9NJ0vUPGYucTOi-iDDOHP68AE9SE2RPFFlIhzjv6BxzKRAS_lIReE3XwPcSIod23voK9WK0RMtMg_DiAmkEk5jdB5iWCsFbKV8ZGKf14OY2EkVEQ0JizgT0rDF32f-DvoyGztoGMlEaAf5XxaZN6TIGRqeCONkyynKsYtWq-jkNNAXnvko6tccQsISiX9KHvrLJoIx5UzN01QbUcXSv4SzEBSUs4SaSCY61jBuc9oSTkDwlTz5VXwWHA0G_jRvwno2hRQrq2d_egkzphrYrGrUn6Z9rr5-dlApQ04sDDpR4hLn761qDmqKhkwBG4afRHEnp38WpoKmOqiXefd9A5eFoASGZ6SeTwyOJxZQswUnFoLtWNK-nbQBGMnoyKkuRJG7-H7DsrI6ppQXwJropNAM7C66JwHMMkQjwvkWnjOL5a1KHs3gypNwXXCySIIAVLJL-Ju9mFMmhBTPwjWgf6nCGNUm35b185CyApYMygQBxHECcmsW6ZK0v7AiVzrHrPu4qMkSUL5Wlmar3uIpj98W_xOM9z3wXerzB8CiBJRpygxoGDwK-QQIcFk-MX1A_yyXPnJbLXvhw2julXwyAsnlcjEiq0wrJSggMPeYXZNmVh4MDXaL8_Q_q6Yc5Hd36Ush4NfLwquWwRJQW6xGo8liATP6NJx9nozNsCYP49nD1I43JcJ2945wTGmQzijeQ2MWHzm8fQotZwGlbv53dmdYtvazleN2eX-D59P0nnofMPO7WLkyqgixg_eKhdjTKqEOjihMyxzxi3HZYH2gEd1gDx5Doh43eCNeIQZest-ljPIwJZP9AXs7AmU6ONsE-71Q_KsgG1Uj82LB3seee5uiYO8F_8Beu3930-13O72-2_vYbvW6YH3GXsPtdG76d3f9rtu77bU6btd9dfDPNHP75rZzB7Zuu9Nrtbptt-dgkB58lcyzD5f0--X1F0hCxz8?type=png)](https://mermaid.live/edit#pako:eNqNVu9v4jgQ_VcsSyuBLpTwo9BGJyQKLIe0dKMFltOJLyYxYNWxOceh26v6v984cdIA2e3yKcYzb2bee3byigMZUuzhmP6bUBHQMSN7RaKN2AgEPxJoqdAqpipbH4nSLGBHIjRa0y0iMRovfPN4vT9f59vz9fXueOjPrv_1pybHJy8RhdWUaPpMXlCtHbSP9Ypovxztkz1FtSGHvZfmmgYHouGpno9ihmgMBtCqhxaUqOCAdpztDxoREaKjkgGlIdLyDC9PhizIna89NJ0sUfOYhcTNiOqDDOMs6NMn9Cg1RfJElclwTPyCxjGTAi3lExVF3HwNcCMpdmzvoW9UK0ZPtKg8DCMmkEo4jdF5imGtlLCV8omJfd4PYmInVUQ0FCzyTErDNv-QxTvo62zsoGEkE6Ed5H9dZNFQImdoeCKMky2nKMcuRq2ik9NAX0TmUtSvOYSCJRJ_lzz0hy0EMuVMzdNSG1HF0nfCWQgOyllCTSQTHWuQ26y2hBMwfCVPfhWfBUeDgT_Nh7CRTSHFyvrZn17CjKkGNqsG9afpnKtvXxxUqpATC0InSlzi_LlVzUFN0ZApYMPwkyju5PTPwtTQVAf1Mu--b-CyFJSAeMbquWKwPLGAmlNwYiHsHUvet0obgJGMjpzqwhR5iO83LCurY0p5AayJTgrPwNlFDyQALUM0Ipxv4TnbsbxV2aMZXEUSrgtOFkkQgEt2CX_fL3TKjJDiWbgGzC9VGKPa5O9l_Tyl7IAlgzbBAHGcgN2aRbkknS-sqJXqmE0fFz1ZAsrXytKcqvd8yuP3g_8Z5P0IfJfG_AawKAFlnjICDYMnIZ8BAS7LZ6YP6K_l0kdt17UXPkjzoOSzMUhulwuJrDOtlaCBwNxj9pg0s_ZANDhbnKf_WTflID-7S18LA79dNl51GCwBtcVqNJosFqDR5-Hsy2RsxJo8jmePUytvSoSd7gPjmNagnHG8h8YsPnJ4-xRezhJK0_xSuzMs2_vZkeP28P4Ez6fpPfUxYBZ3ceTKqCLEDt4rFmJPq4Q6OKKgllniVxOywfpAI7rBHjyGRD1t8Ea8QQ68ZP-RMsrTlEz2B-ztCLTp4Owk2O-F4l8F1agamRcL9vr9fjtFwd4r_oG9xv3dTdft3rY6nfu7btu9c_AL9lrt3o3b6bj9nttp9Xv3bvfNwf-ldVs3nW7vvte7bXXvXPe21YYMMB58k8yzz5b06-Xtf7jJxrc)

---

# Retry Job Process Flow Diagram

[![](https://mermaid.ink/img/pako:eNqllF9vmzAUxb-K5adOIl0C-YfVRUpCGmVaNDZaRZp4ceGWWAM7tU1bFuW7zxCgWbNIk-YnW_fnc47NxXsciRgwwQqecuAReIwmkmYhR2bsqNQsYjvKNQqiLcR5ChJRhb6DlgX6LB7OufWmBLzAN7Pzqjcrqz4tMqhW54S_PCWWVMMLLdCVPbf9DyE_8m2WzmTizQj6loOJswMeM54YtWqvunmQHyccXnWVdqrRzSfExctVK-PNOkagFSPI_1MBpUzpBk6F2KEFjbbvfY7l97HWG4LuJEsSc2GNHONPOZNFo1iO9caw_rI9Qk0qTXWuTkF_2alV_YsMTXVbDfIoAqUe8_St3hqWd3a_i83d1ioouJ_PF0HwN3bqr4zp1-CuSdeREAkZqwvCK65AtkGmkWbPTBdvMKQK2pi3lKUQ_1PE2-nqy8L7H9NAszRtPvJFoUgCNbAsm2Yucq6rPqqznLTTiTivT2Am2MKJZDEmWuZg4QxkRssl3pdIiPUWMggxMdOYyp8hDvnB7DGt_0OIrNkmRZ5sMXmkJraF88q7_jNbxJiBrAJiMnJ6lQYme_yKSW84uu52B-5gPLBHruMM-xYuMBn2rh17POr3-n275w5t92DhX5Vr93rs2K7T77rDscEH9sDCEDMt5Pr4PFSvxOE3JGZLVA?type=png)](https://mermaid.live/edit#pako:eNqllFFvmzAUhf-K5adOIhmDhQDqIiUhjTItGhutIk28uHBLrIGd2qYti_LfZwjQrFmkSfOTrfv5nGNz8R4nPAXsYwmPJbAEAkoyQYqYIT12RCia0B1hCkXJFtIyB4GIRN9BiQp95vfn3HpTA0EU6tl5NZjV1ZBUBTSrcyJcnhJLouCZVOjKmlvhu5gd-T7LYDIJZj76VoKOswOWUpZptWavvL4X7ycMXlSTdqrQ9SfE-PNVLxPMBlqgF_NR-KcCyqlUHZxzvkMLkmzf-hzLb2OtNz66FTTL9IV1cpQ9llRUnWI91hvNhsv-CC0pFVGlPAXD5aBVDS8yJFd9NSqTBKR8KPPXem9Y39ndLtV326qg6G4-X0TR39hpuNKmX6PbLt1AQMJFKi8Ir5gE0QeZJoo-UVW9wpBL6GPeEJpD-k8Rb6arL4vgf0wjRfO8-8gXhRIBRMOibpo5L5lq-qjNctJOJ-KsPYGeYANngqbYV6IEAxcgClIv8b5GYqy2UECMfT1NifgZ45gd9B7d-j84L7ptgpfZFvsPRMc2cNl4t39mj2gzEE1A7Dum02hgf49fsP9hbA1Nc-R5Y8txXct1XANX2B_bQ9tyx7bpeJZnOfbBwL8aU3Po2pZnfzQ9x7VtZ2SNDAwpVVysj69D80gcfgPgq0su)

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

---
# Payment Service Database Design

## 1. Payment

Stores the **current/latest state** of a payment. One record per payment.

| Column | Description |
|---------|-------------|
| payment_id | Internal UUID |
| order_no | Merchant order number |
| office_id | Office ID |
| gateway | Payment gateway (e.g. 2C2P) |
| gateway_payment_id | Payment ID returned by gateway |
| gateway_transaction_id | Transaction ID returned by gateway (if any) |
| payment_category | ECOM |
| payment_type | WALLET |
| channel_code | ALIPAY |
| amount | Payment amount |
| currency | Currency code |
| status | Current payment status (INITIATED, PENDING, SUCCESS, FAILED, CANCELLED, EXPIRED) |
| ticket_issued | null/true/false if POST payment-record (EXT) to DAPI is successful after payment success |
| gateway_url | Redirect URL returned by gateway |
| order_locator | Airline PNR |
| order_contact_name | Contact name of the order |
| expiry_datetime | Payment expiry datetime |
| retry_count | Number of inquiry retry attempts |
| next_retry_at | Next scheduled retry time |
| last_retry_at | Last retry timestamp |
| last_error_code | Latest gateway/system error code |
| last_error_message | Latest gateway/system error message |
| instance_name | masg-10dspmwcap10, masg-1dspmwcap10, mahk-2dspmwca10 to handle Preprod, Prod, Dr environment |
| created_at | Record creation timestamp |
| updated_at | Last update timestamp |
| completed_at | Payment completed timestamp (nullable) |

---

## 2. PaymentActivity

Stores the **complete history** of all payment-related events.

One payment can have multiple activity records.

| Column | Description |
|---------|-------------|
| activity_id | Internal UUID |
| payment_id | FK to Payment.payment_id |
| activity_type | Event type |
| source | DSPMW / CALLBACK / INQUIRY / RETRY_JOB / MANUAL |
| status_before | Previous payment status |
| status_after | New payment status |
| gateway_status | Raw status returned by gateway |
| gateway_code | Gateway response/error code |
| gateway_message | Gateway response/error message |
| request_payload | Request JSON (optional) |
| response_payload | Response JSON (optional) |
| created_at | Activity timestamp |

### Example Activity Types

- CREATE_PAYMENT
- GATEWAY_RESPONSE
- CALLBACK_RECEIVED
- CALLBACK_PROCESSED
- STATUS_CHANGED
- INQUIRY
- RETRY_STARTED
- RETRY_SUCCESS
- RETRY_FAILED
- MANUAL_UPDATE

---

# Retry Job Design

The retry job should **read from the Payment table**, not the PaymentActivity table.

Example query:

```sql
SELECT *
FROM Payment
WHERE status IN ('PENDING', 'UNKNOWN') AND instance_name = 'masg-1dspmwcap10'
  AND next_retry_at <= CURRENT_TIMESTAMP
  AND retry_count < 5;
```

Retry flow:

1. Read eligible payments.
2. Call Payment Gateway Inquiry API.
3. Update Payment table with latest status.
4. Increment `retry_count`.
5. Update `next_retry_at` if another retry is required.
6. Insert a PaymentActivity record for the retry attempt.

---

# Status Flow

```text
INITIATED
    │
    ▼
PENDING
    │
    ├────────► SUCCESS
    │
    ├────────► FAILED
    │
    ├────────► CANCELLED
    │
    └────────► EXPIRED
```

---

# Overall Architecture

```text
                 Create Payment
                        │
                        ▼
                  Payment Table
             (Current Payment State)
                        │
                        │ 1 : N
                        ▼
              PaymentActivity Table
          (Complete Audit / Event History)

                        ▲
                        │
      ┌─────────────────┼──────────────────┐
      │                 │                  │
      │                 │                  │
 Gateway Response   Gateway Callback   Inquiry Retry Job
```

## Design Principles

### Payment Table

- One row per payment.
- Always represents the latest payment status.
- Optimized for API queries.
- Contains retry scheduling information.

### PaymentActivity Table

- Append-only history.
- Never update existing records.
- Used for auditing and troubleshooting.
- Records every interaction with the payment gateway and background jobs.

### Retry Job

- Reads from the Payment table.
- Uses:
  - `status`
  - `retry_count`
  - `next_retry_at`
- Writes:
  - Updated Payment record.
  - New PaymentActivity record.

This keeps operational queries fast while maintaining a complete audit trail.



