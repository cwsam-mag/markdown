# Payment Service – Proposed Architecture

## Objective

Provide a common Payment Service that can be used by multiple applications (Booking, Check-in, Manage Booking, Jounify and more).

---

# Prerequisite

A **Session Token** is generated during Booking / Check-in / Manage Booking initialization.

All Payment Service APIs require the Session Token in the request header.
- need to revise this, as Client such as journify will not have the IBE/SSCI Session-Token

The Payment Service uses the Session Token to retrieve:

- officeId
- pointOfSales
- booking information or any additional information required to process the payment
- client without the Session-Token, they can pass in these data via payload

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
<!--
[![](https://mermaid.ink/img/pako:eNqNVmGP2jgQ_SuWpUqgCwtkYQvRCYkFyiGVbVTgqCq-mMSAtY7NOQ7b7Wr_e8eJkw2Q3pZPMTPzZubNGycvOJAhxR6O6X8JFQEdM7JXJNqIjUDwI4GWCq1iqrLzkSjNAnYkQqM13SISo_HCN4_X9vk6N8_X19bx0J9d_-tPTYxPniMKpynR9Ik8o5obuMd6hbdf9vbJnqLakIPtubmmwYFoeKrnrZgmGoMBlOqhBSUqOKAdZ_uDRkSE6KhkQGmItDzDy4MhCmLnaw9NJ0vUPGYucTOi-iDDOHP68AE9SE2RPFFlIhzjv6BxzKRAS_lIReE3XwPcSIod23voK9WK0RMtMg_DiAmkEk5jdB5iWCsFbKV8ZGKf14OY2EkVEQ0JizgT0rDF32f-DvoyGztoGMlEaAf5XxaZN6TIGRqeCONkyynKsYtWq-jkNNAXnvko6tccQsISiX9KHvrLJoIx5UzN01QbUcXSv4SzEBSUs4SaSCY61jBuc9oSTkDwlTz5VXwWHA0G_jRvwno2hRQrq2d_egkzphrYrGrUn6Z9rr5-dlApQ04sDDpR4hLn761qDmqKhkwBG4afRHEnp38WpoKmOqiXefd9A5eFoASGZ6SeTwyOJxZQswUnFoLtWNK-nbQBGMnoyKkuRJG7-H7DsrI6ppQXwJropNAM7C66JwHMMkQjwvkWnjOL5a1KHs3gypNwXXCySIIAVLJL-Ju9mFMmhBTPwjWgf6nCGNUm35b185CyApYMygQBxHECcmsW6ZK0v7AiVzrHrPu4qMkSUL5Wlmar3uIpj98W_xOM9z3wXerzB8CiBJRpygxoGDwK-QQIcFk-MX1A_yyXPnJbLXvhw2julXwyAsnlcjEiq0wrJSggMPeYXZNmVh4MDXaL8_Q_q6Yc5Hd36Ush4NfLwquWwRJQW6xGo8liATP6NJx9nozNsCYP49nD1I43JcJ2945wTGmQzijeQ2MWHzm8fQotZwGlbv53dmdYtvazleN2eX-D59P0nnofMPO7WLkyqgixg_eKhdjTKqEOjihMyxzxi3HZYH2gEd1gDx5Doh43eCNeIQZest-ljPIwJZP9AXs7AmU6ONsE-71Q_KsgG1Uj82LB3seee5uiYO8F_8Beu3930-13O72-2_vYbvW6YH3GXsPtdG76d3f9rtu77bU6btd9dfDPNHP75rZzB7Zuu9Nrtbptt-dgkB58lcyzD5f0--X1F0hCxz8?type=png)](https://mermaid.live/edit#pako:eNqNVu9v4jgQ_VcsSyuBLpTwo9BGJyQKLIe0dKMFltOJLyYxYNWxOceh26v6v984cdIA2e3yKcYzb2bee3byigMZUuzhmP6bUBHQMSN7RaKN2AgEPxJoqdAqpipbH4nSLGBHIjRa0y0iMRovfPN4vT9f59vz9fXueOjPrv_1pybHJy8RhdWUaPpMXlCtHbSP9Ypovxztkz1FtSGHvZfmmgYHouGpno9ihmgMBtCqhxaUqOCAdpztDxoREaKjkgGlIdLyDC9PhizIna89NJ0sUfOYhcTNiOqDDOMs6NMn9Cg1RfJElclwTPyCxjGTAi3lExVF3HwNcCMpdmzvoW9UK0ZPtKg8DCMmkEo4jdF5imGtlLCV8omJfd4PYmInVUQ0FCzyTErDNv-QxTvo62zsoGEkE6Ed5H9dZNFQImdoeCKMky2nKMcuRq2ik9NAX0TmUtSvOYSCJRJ_lzz0hy0EMuVMzdNSG1HF0nfCWQgOyllCTSQTHWuQ26y2hBMwfCVPfhWfBUeDgT_Nh7CRTSHFyvrZn17CjKkGNqsG9afpnKtvXxxUqpATC0InSlzi_LlVzUFN0ZApYMPwkyju5PTPwtTQVAf1Mu--b-CyFJSAeMbquWKwPLGAmlNwYiHsHUvet0obgJGMjpzqwhR5iO83LCurY0p5AayJTgrPwNlFDyQALUM0Ipxv4TnbsbxV2aMZXEUSrgtOFkkQgEt2CX_fL3TKjJDiWbgGzC9VGKPa5O9l_Tyl7IAlgzbBAHGcgN2aRbkknS-sqJXqmE0fFz1ZAsrXytKcqvd8yuP3g_8Z5P0IfJfG_AawKAFlnjICDYMnIZ8BAS7LZ6YP6K_l0kdt17UXPkjzoOSzMUhulwuJrDOtlaCBwNxj9pg0s_ZANDhbnKf_WTflID-7S18LA79dNl51GCwBtcVqNJosFqDR5-Hsy2RsxJo8jmePUytvSoSd7gPjmNagnHG8h8YsPnJ4-xRezhJK0_xSuzMs2_vZkeP28P4Ez6fpPfUxYBZ3ceTKqCLEDt4rFmJPq4Q6OKKgllniVxOywfpAI7rBHjyGRD1t8Ea8QQ68ZP-RMsrTlEz2B-ztCLTp4Owk2O-F4l8F1agamRcL9vr9fjtFwd4r_oG9xv3dTdft3rY6nfu7btu9c_AL9lrt3o3b6bj9nttp9Xv3bvfNwf-ldVs3nW7vvte7bXXvXPe21YYMMB58k8yzz5b06-Xtf7jJxrc)
-->

[![](https://mermaid.ink/img/pako:eNqdVm1v4jgQ_iujSCuBLhRKKZTothIFlkNauhEvy-qEdDKJAauOzToOPa7qf79x4qS8dff28im2Z54ZP_PMJC9OIEPqeE5MvydUBLTHyFqRaCEWAvAhgZYKZjFV2XpLlGYB2xKhYU6XQGLoTXzzen4-mufHo_n5aa_jD893_YHx8ck-orgaEE2fyR5K9aC-LV-w9g-tfbKmUOpwPNtX5zTYEI1v5fwq5hKV-3tM1YMJJSrYwIqz9UYDESFslQwoDUHLI7zcGb3QdzT3YNCfQnWbmcTViOqNDOPfl6p6n_uN0j3ohBETME44jaECX4Y9uGTVlWLF1mgwZRGVCUalisnQhdn4swv-xM_if_gAj1JTkDuqTDKuSWVC45hJAVP5REVhN5pjphmsB2OqFaM7Wlwqy0qlWR27mIIcOCylfGJinV8VmFhJFRGNAQs_41KxvDxk9q65qQudSCZC4wW-TDJrDJGT39kRxsmSU8ixLYuXK8VpoE8s8yqXz8uDAQ_q81_Jg99sIFTAcYEW4hJLXwlnIYozZwmqgLWLNSrJrJaEE-ylizz57_GZwhuLrqIGOjcc00CqMNUOBtBJ_HH4OJwOO9N-78DRH-R3t35VIcXMdpg_OI3eoxqLEJ9Enm3DH0f2-4-94eMg3VpnzWmNhyHEOCpoeIFyf5Aynip6Ic70gJJLlDhNLY1RQkCmsC6mUonibi4EDGe6luqgfKgA3zdwmQskKCPTz7l2cLljATWtvmMhnm0PGtxqzgB0ZbTlVBfyzE18v2KJtjwVwCk5uRkOKHggAaoKu5twvsT37MSW4pJQq8GZJeG64GSSBAHqdZXwt_Nfq9xk1u32J5ML7pmi03RsNhWVusdQ6n-blo9dDqU8ZXhLVHIcJ9g31SJ6kiYT_nKqOsX7K8P7OB3P-m8QlMdvPp9QIP8D31LxqTP8nPfOGTTh2H3hHixf1VPjo1DDtUDJQ5hsOQtM0PMqogreFpnojYI6wZOQz3gJ_GQ9M72BP6ZTH-q1mv3sonYelHw2Cs71fKIh2zpW6yjzwIx8O1Gqq5QhVBWOIc7TPSv3HOS9L9pL0WGvp4lf6taMUyhZvlAFGWNGDtmwsAJKibC3-4myTWoYzrSkBz0Wbzn-AxTNljkc3OaH0jjCsrkfzQRup8s7eD5NR_rPATO7k5lwiCpCx3XWioWOp1VCXSeiWC2zdF6MycLRGxrRhePha0jU08JZiFf0wV-dP6WMcjclk_XG8VYE03SdrNfsX1uxqzAaVV3zDXa8xu1NM0VxvBfnb8erXNebV9c3jVq71W6027Vao-46e7N_W7tq3F7X71rt-k2t1rpuvrrOP2no66vWLbo0m3ftO4RrNFqug9rDiT_K_h_T38jXfwGRoUMS?type=png)](https://mermaid.live/edit#pako:eNqdVm1v4jgQ_isjSyuBLiyUthSi20oUWA5p6Ua8HKcT0skkBqw6Nus49Liq__3GiUN56-7t5VNsz-szz4z9QkIVMeKThH1LmQxZl9OVpvFcziXgR0OjNEwTpvP1hmrDQ76h0sCMLYAm0B0H9vf8fDgrjoez89NuOxic7wZ9qxPQXcxw1aeGPdMdlOphfVO-IB0cSgd0xaDUFni2q85YuKYG_8pFKjaJyv09hurDmFEdrmEp-GptgMoINlqFjEVg1JG9Qhm1UHc486Hfm0B1k4sk1ZiZtYqSXxe6el_oDbM9aEcxlzBKBUugAl8HXbgk1VFyyVcoMOExUyl6ZZqryIPp6IsHwTjI_X_4AI_KMFBbpm0wng1lzJKEKwkT9cTkXm44w0hzsz6MmNGcbdk-qTwqnUV1rGILcqCwUOqJy1WRKnC5VDqmBh3u9axKxeHykMt7NlMP2rFKpcEEvo5zaXRRgN_eUi7oQjAobDsUL1dKsNCcSBZVLp-XBx0e1Oe_gge_OEfIgOMCzeUllH6ngkdIzgIlqALWLjHIJLtaUEGxly7iFLyHZ2beSnQ0s6YLwRELlY4y7qADkyafBo-DyaA96XUPFIN-kbvTq0olp67Dgv6p9y4zWITkxPN0E33fc9B77A4e-9nWKm9OJzyIIMFRwaILkAf9DPGM0XN5xgekXKrlaWiZjxIa5BrrYiuVauEVREB3tmuZCcuHDAgCay5XgRRpZPu54A4utzxkttW3PMKzzUGDO85ZAx0VbwQze3oWIkFQcUA7nPaGM3AKMRxQ8EBDZBV2NxVigf_5iSvFJaJWwzNJKswek3EahsjXZSrezn-ucuNpp9Mbjy-o54zOwnHRVHSmnkCp98ekfKxySOUJxyyRyUmSYt9U997TLJjop0M1mb2_cnufJqNp780EE8mbzmckyP-w76D43B58KXrnzDQV2H3RDhxe1VPhI1eDlUTKQ5RuBA-t0_MqIgveFjnpLYPa4ZNUz5gEXlnP3Kzht8kkgHqt5q5d5M6DVs-WwQWfTzjkWsdxHWke2pHvJkp1mSGErMIxJES25-heGHnvRnvZd9jraeCXujXHFEoOL2RBjpilQz4sHIEyIFx2P2C2DQ3d2Zb0ocuTjcA3wL7ZcoWDbL5LjSNbLvajmSDcdHnHXsCykf5jg7ncyUw4tCoj4pGV5hHxjU6ZR2KG1bJL8mJF5sSsWczmxMffiOqnOZnLV9TBp86fSsWFmlbpak38JcUwPZL3mnu17Xc1emO6Y-9g4t_c1K8zK8R_IX8Tv_XxplmvNxqt21rrttVs3XlkR_zK9R3uN5p3tfp17Q7PGq8e-Sfze_Xx5qp2W8PDWqt51WzUGx5B4uG4H-aPx-wN-fov4X9CnA)

---

# Retry Job Process Flow Diagram

[![](https://mermaid.ink/img/pako:eNqdVV1v2jAU_SuWn1opFAiBQlQ6UUhZttFlJKjdhBS5iQtWg01tZy1D_Pc5zgelrTp1ebrOPfecc29iewsjFmNoQ4EfUkwjPCJowdFqToF61ohLEpE1ohL40RLHaYI5QAJMseQb8IXdvsZNrjPAyPdU9Do7usiyHtqssF69Rnjj54gxkvgRbcCROTS94znN8ZWX2vn56MIGP1Ks7KwxjQldKDZdK85uef2c4iep3Q4kOOsDyh6PKprRRU0RVGQ28A4ZQEKELMEJY2vgoGj5UidPv7Q1ubZBwMlioQZWNuPSh5TwjfY1dgJQL53Wt0XkxrtSL3sm14rJG1cNFjxCIpmKiqagrQccUYEiSRj1NeIT4zHmV6y_1UGYsAhJxncHGt64Vtj1DuifY1Aiq6yfRhEW4i5N9vnKa0YzW8fqo1X4KY6UujabE_f92XDo-P4b5dm3dKnAvJIbqHZ-E7npT51g-jN8p3TguaqF735QjqnGtbIAR85NcHxYkqHLtgMS3WMJiBApjkG9Mp7qPuIPdyk1X5jz9YPpzPlYp67vz5wwcIdfnWBfiROxV7tEJHnTWUZbOMuHDS4H7jdn9D-zflmpHfiSJEm5T943wLNdF0YspdIA2TYM8zdILRMk9sv3JIDztCb8X70eSB1wG8Uc-s6N504_Ogg_GAQzPxx-HlyND0ZBC0MqgAZccBJDW_IUG3CF-QplS7jNIHMol3iF59BWYYz4_RzO6U7VqLPuF2OrsoyzdLGE9h1SEzBg_usVR3H1lis1zIdZl9BudhotzQLtLXzK1qcnjUa71-62zdNeq9WxDLiBdqd50jK7p1bTssxmr2P2dgb8o3UbJ92W2WtZjW6z3basptk1II6JOh8m-Y2gL4bdX_cy6yk?type=png)](https://mermaid.live/edit#pako:eNqdVV1v2jAU_SuWn1opFAiBQlQ6UUhZttFlJKjdhBS5iQtWg01tZy1D_Pc5zgelrTp1ebr2Pfecc29iZwsjFmNoQ4EfUkwjPCJowdFqToF61ohLEpE1ohL40RLHaYI5QAJMseQb8IXdvsZNrjPAyPdU9Do7usiyHtqssF69Rnjj54gxkvgRbcCROTS94znN8ZWX2vn56MIGP1Ks7KwxjQldKDZdK85uef2c4iep3Q4kOOsDyh6PKprRRU0RVGQ28A4ZQEKELMEJY2vgoGj5UidPv7Q1ubZBwMlioQZWNuPSh5TwjfY1dgJQL53Wt0XkxrtSL3sm14rJG1cNFjxCIpmKgqYgrQccUYEiSRj1df4T4zHmV6y_1UGYsAhJxncHCt64Vpj1DsifY1Aiq6yfRhEW4i5N9vnKaUYzW8fqlVX4KY6UuraaE_f92XDo-P4b5dmbdKnAvJIbqHZ-E7npT51g-jN8p3TguaqF735QDqnGtbIAR85NcHxYkqHLtgMS3WMJiBApjkG9Mp7qPuIPdyk1X5jz9YPpzPlYp67vz5wwcIdfnWBfiROxV7tEJHnTWUZbOMuHDS4H7jdn9D-zflmpHfiSJEl5St43wLMzF0YspdIA2SEM8x2klgkS--V7EsB5WhP-r14PpA64jWIOfefGc6cfHYQfDIKZHw4_D67GB6OghSEVQAMuOImhLXmKDbjCfIWyJdxmkDmUS7zCc2irMEb8fg7ndKdq1E33i7FVWcZZulhC-w6pCRgw__SKi7ja5UoN82HWJbSb7V5bs0B7C5_UunN60miozW7bPO21Wh3LgBtod5onLbN7ajUty2z2OmZvZ8A_Wrdx0m2ZvZbV6Dbbbctqml0D4pio-2GS_w_0b2H3F9IY6r8)

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

<details>
### <sumary>Example Response</sumary>

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
</details>

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
| ~~`/payments/confirmation`~~ | ~~POST~~ | ~~Frontend verifies the final payment status~~ | ~~call dapi payment-records/confirmation~~


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
<details>
## <summary>Admin rules</summary>
```json
{
  "data": {
    "paymentMethods": {
      "attributes": {
        "amadeusCheckoutSdk": {
          "shouldDisplay": true,
          "attributes": {
            "card": {
              "shouldDisplay": true,
              "attributes": {
                "label_mop_creditcard": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_mop_creditcardtok": {
                  "shouldDisplay": true,
                  "attributes": {}
                }
              }
            },
            "onlineBanking": {
              "shouldDisplay": true,
              "attributes": {
                "label_amop_cimbclicks": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_maybank": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_fpx": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_cimbniaga": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_enets": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "Internet Banking": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_ideal": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_netbanking": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_poli": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_rupay": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_upi": {
                  "shouldDisplay": true,
                  "attributes": {}
                }
              }
            },
            "eWallet": {
              "shouldDisplay": true,
              "attributes": {
                "label_amop_boost": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_grabpay": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_touchngo": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_alipay": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_googlepay": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_mobikwik": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_paytm": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_promptpay": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_upiqr": {
                  "shouldDisplay": true,
                  "attributes": {}
                }
              }
            },
            "installment": {
              "shouldDisplay": true,
              "attributes": {
                "label_amop_installmentpayment": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_hoolah": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_humm": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_installmen": {
                  "shouldDisplay": true,
                  "attributes": {}
                }
              }
            },
            "others": {
              "shouldDisplay": true,
              "attributes": {
                "label_amop_paypal": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_cup": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_applepay": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_fnpl": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_iatapay": {
                  "shouldDisplay": true,
                  "attributes": {}
                },
                "label_amop_wechat": {
                  "shouldDisplay": true,
                  "attributes": {}
                }
              }
            }
          }
        },
        "customPayment": {
          "shouldDisplay": true,
          "notificationURLs": {
            "failedURL": "http://example-failed.com",
            "confirmationURL": "http://example-confirmation.com",
            "cancellationURL": "http://example-cancellation.com",
            "backendURL": "http://example-callback.com"
          },
          "attribute": {
            "alipay": {
              "shouldDisplay": true,
              "attributes": {
                "paymentGateway": "2c2p",
                "endpoint": "https://core.demo-paco.2c2p.com/api/2.0/Payment/nonUI"
              }
            },
            "weChatPay": {
              "shouldDisplay": true,
              "attributes": {
                "shouldDisplay": true,
                "paymentGateway": "2c2p",
                "endpoint": "https://core.demo-paco.2c2p.com/api/2.0/Payment/nonUI"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

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



