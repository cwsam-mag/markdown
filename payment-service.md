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

<!--
[![](https://mermaid.ink/img/pako:eNqdVm1v4kYQ_iurlU5KVBNILiFg9SIR4CjSkbN4KVWFVC32AqusvXS9JqVR_ntn38xr0l79yd6deWb2mWdm_YpjkVAc4pz-WdAsph1GlpKks2yWIXhIrIREk5xK-70mUrGYrUmm0JTOEclRZxTp15_nsvpwEZFtSmEvIkt6eeoymHqPwdQ6PArxzLJlgJxngNoiWzBY6DyeAei0ov7patTTsD52jyj6QrYW_ya-WZ_BiaJ9D52tNW9x2N9WpzReEQVvl54ITUHl4QEOGqIRJTJeoQVny5VCJEvQWoqY0gQpcYDpncELfAfTEPW6Y1RdW5O8mlK1EkluQnu_gVlDrSRlGRoWnOaogr73O-icleUKDMYspaKAqFQykQRoMvwGlI4iG__TJ_QkFEViQ6VOJtCpjGieM5GhsXimWWk3mEKmFjZEQ6okoxtaHspmJU1Why66MHsOc1tWf1TEsoWQKVEQsPTTLhXHSykDOGmAWqkotBSi7yNrDSE8-a0NYZzMOUUe27F4vlKcxurIErkqX56WBwLu1ee_kod-coFAAYcFmmXnWPqVcJaASD1LqIqgdrkCJemvOeEEOvEsT9F7fBp4bdGWVEN7wyGNhUyMdiCAKvIv_af-uN8adzt7jlHPn935VTORTVynRb3j6B2qoAj5UeTJOvk4ctR96vSfemZpaZvUGfcTlMOgockZyqOeYdwoepad6AEkV8jsODXbzQDIJNRFV6qQPPBCgHC6a6mKL_cVEEUazrqgAmSk-9lrBz43LKa61Tcsgb31XoM7zWmAtkjXnKpSnt4kiiqOaMdTCWzI8WYwpNAjiUFV0N2E8zm82x1XinNCrcYnloSrkpNREceg10XBd_s_VrnRpN3ujkZn3K2iTToum4o07jm66P42vjx02ZfymMEpQcl5XkDfVMvohUkm-eFUlcH7w-J9GQ8n3R0E5fnO5ysI5H_gOyq-tvrffO-cQBMO3ZdskeOremx8EKq_zEDyKCnWnMU66GkVQQW7Dyt6raBW_JyJFzjEkqIXplbol_E4Qje1mru0QTuPUrxoBXs9H2nItY7TOsg81iPfTZTqwjAEqoIxxLlZc3L3INCh711qr2WTvR3Yv9-zlll04VgDLVjetCjsyNiT0VFYf9FAlaiE4K_mBWK_HRNnIvuJm5gxYU0MyY65f-kaHRygdLuHqMPyNSfbXSNbhz2mPpTdAZZj5GDecDe5dnjA4gFkRM2NUW5-BGtNj6bOfq5ZggO8lCzBoZIFDXBKQQ_6E79qkxlWK5rSGQ7hNSHyeYZn2Rv4wA_V70Kk3k2KYrnC4YJApgG23ez-KstVCdGobOtbHofXzXrNoODwFf-Fw0q9dnX9udFo3jWbtbvabaN-G-AtDu-vr-r1WgOe22b9pllvvgX4bxP4-uq-dlu7q99_bjZq99c3zQCDtOFCGdifW_OP-_YPvchzCQ?type=png)](https://mermaid.live/edit#pako:eNqdVv1v4kYQ_VdGlk5KVBMgl5Bg9SIR4CjSkVsFKFWFVC32AqusvXS9Tkqj_O-d_TDfSXv1T7Z35r3ZN2_Wfg1imbAgCnL2Z8GymHU4XSiaTrNpBnjRWEsF45wp97yiSvOYr2imYcJmQHPoDIm5_XmmqndnhK5ThmuELtj5ccpgUmYMJnsJIbRlNueLEDr3J_I6LdI_fkt6Bq2k7FHNXujawV7Gl6sTOITsZpgiXXhL4Pq6OmHxkmq8Oy_3b3ZeubvD_UUwZFTFS5gLvlhqoFkCKyVjxhLQcg-zTMYszB1MIuh1R1BduZC8mjK9lEluqcu8gX0HrSTlGTwWguVQge_9DpyKclphwIinTBbIyhSXSQjjx28hkCFx_J8-wYPUDOQzU6aY0JQyZHnOZQYj-cSyTdxggpU62AgemVacPbPNplxVyla1n2Ias5Mwk_KJZ4tyq8CzuVQp1Ui4yTMpFa_LvYsPzU5DaKWyMFYg34cuGilK8VvPlAs6EwxKbK_i6U4JFuuDSPBdPj9uDxLu9Oe_igc_eSJ0wH6DptkplX6lgido0lIlqAL2LtfoJPM0o4LiAJ7Uibynp4U3EW3FDHQZ-MhiqRLrHSTQRf6l_9Af9VujbmcnkfTKvfu8aiazsZ800jtk7zCNTcgPmMer5GNm0n3o9B969tXCDakP7ieQ4_nCkhOSk55V3Dp6mh35AS1XqOywNDfNCMgV9sV0qlAiLI2AdGZqmY7Pdx1AiIFzKVCgjcw8l97Bx2ceMzPqzzzBtdXOgHvPGYC2TFeC6Y09yxBCKl5or9MG2IpThuEhBfc0RlfhdFMhZnjvVnwrThm1Gh9FUqE3mgyLOEa_zguxXf-xzg3H7XZ3ODyR7hxty_HVVJRNz-Gs-9vofD9l18ojjrtEJ-d5gXNT3bAXtpjkh0vVFu8Ph_dl9DjubiGYyLc5X9Eg_wPfS_G11f9Wzs4RNBU4fckavF7Vw-A9qv4iQ8tDUqwEjw3pcRfRBdsHZ3rjoFb8lMkX3MSCwQvXS_hlNCJwWav5bzV6517JF-Pg0s8HHvKj472ONo_Nke9PlOrcKoSuwmNICPvO270EwQl976P2uhmyt73492fWKQtnXjX0gtPNmMIdGTs2OqAtPzTYJaaQ_NXeIPfboXCWuTxxE3tMuBArslfuX6bGkCOUGfcIOjxfCbreDrJL2FHqQ9vtYXlF9s4b4U-uLR6quAdJmP1ibBY_gnWhB6fObq1ZEoTBQvEkiLQqWBikDP1gHoNXEzIN9JKlbBpEeJtQ9TQNptkb5uAP1e9SpmWaksViGURzipWGgZtm_zO5eauQjam2-coHUb15eWVRgug1-CuIKo3aRf3z7W3zutmsXdeubhu4vA6im_pFo1G7xeuq2bhsNppvYfC3Ja5f3NSuateNm9urm2at_rkeBmht_KAM3D-t_bV9-wcszW_u)-->
[![](https://mermaid.ink/img/pako:eNq1WGtv4kYU_SsjSysFlcSER0JQk4oEliJtsm4gpaqQosEeYBQzw47tpDTiv_fOw8Y25rGJ6k9-3Pc998yM3y2Xe8RqWQH5ERHmkg7FM4EXYzZmCC7shlygp4AI_bzEIqQuXWIWolH3FuEAdQYOGpHJrxNh35w4eLUg8M3BM1LaVrkfxRr3I61wy_kLZbMyMppldMfZlM4KlDvaHQ7xBAek4Hvb6W-_dXpSK46rh0Pyhlfad9WtLgv8OI7UaPvwtLJHxJ3jEO7iishanN7cQPItNCBYuHM09elsHiLMPLQU3CXEQyFH6VLEyqAFuvejFup1h8heapHAXpBwzr1AhRXr3at3qO0tKEOPkU8CdIq-9zuoSEoXDQSGdEF4BF6JoNwro6fHb1DbgaP9f_mCHnhIEH8lQgZTlqEMSBBQztCQvxCWyN2PIFJttoUeSSgoeSVJUjoqoaLKqsgupBQmur9xqoiyKRcLHILDRE-qnJq6JHiATMuoveCRxITzfaClwUVc_PYrpj6e-ATFtk0VN6UGUdmuFurQYOnjFcI7dIpa6xM3zImhE42K0nY_IcJUQ4-tNvrFOALIZDs6ZkVl_RP71AMIx2VFNoJmByFATz5NsI9hhgsL6-xpgPIghTqQeZ9BGcJE_pG4XHgKc-AnjILr_kN_2G8Pu52kJRDeluKYbXKXX7UyukYp9TFTyk4vrp_RtRlnTzDLTi8ffYeE0EE9KCeCeFRA8Z6EX0YzPdlGsA_QPzs7KyU-ZAhPS1W7QwE63YdO_6GXVEc5M-afTYefqQeS73mn64LGOz3VdzWIcVdTMIZJiQTbn6DESySTXMaOFNmQ0C2lceg40pxWQRE0Q9JQjGB4fKUukQz1Sj34tkzxkkG-NHDHF0ufhMmExCKOc2paZcqYGFZ1i8WASNEtdgHbQErY9ydwr78oI73CcbHdLUnsbyA4iFwXpmYa-ZvvGeTlG5sWKmry9eDp7q47GBSZU3OmwjPRnQo1AQE66f5Vymqk52tIIWkYryCIYJjtJPhIxeZ9OnLI3p9S35dy16GIyM7gE-6FqCUGdGAsWkzgSeIGK1bd6BM_2DD7V4DfkeGiIyr9td3_FjPFli_sC4K9FTLtsPPCiW9Z4AduaokE7FUA5KkgAW2bBz1cEqlt94XxN0hnRtAbDefo9-HQQdVKxWxuoBa3gr_JSYnnpgCryUjBNLlyQTT0aU9VqQC8wLm-r97JSJacbsYmxnI8C0nOaqcQBPlSb4qYpr1YyYAJTVZoe2RUaRM_jIdQKJfQ18JuyvLIPchQYBZgRTEDM8hp2W0GHmTGvXBci4ZrO7dHAhS1xRVxXWA5yxrIgrQAKJ_3wHYhCvj8Y2joSjTk4RAvuLIFf0RErHJUiugUupc0WME00BRoJE6Gt3elfHgH-wRih1gVRI6iJyN3HLdmjB5Lr0bpQwz7qSz28CzI76XK_X7RceXLQzvv9BBnpoM4RJvSNvMOYmQvRDSCd9VWGjOJZbZVKZEMJRXNg9oLSa7qM5nAyt7irN_USvfAr9_VzbPPXQzn1vW60NnuWdkV2REg-ulx-PBIfHYsPpfV_vGQVzfHnN0swlJbX7Ua1yrVgsVX4ibNrlmmzh2j4zOman5gv2sUwG58vXOPpI_fEZzecUAOKqbLrTdVntqmb5mPM9stVHgsTWeq9uX6iE_0qS5u6IlHp1Mi5C0Rggskf91AwxfA4pvfLclaOS3YyGW8m5NH5ozgm9OGDEIqbNS7W0ShymnOMD-xJur_ISk7H-hm9lR8fDNzZ-LiNmnj-1uZlWl7nlkMkKM3F8k5MMWvO9df23Q6Efs_gFLahLRrDSvyfAgkxVYPJpQya4LP7YE2tuG7VbZmgnpWSxJO2VoQKIF8tN6lyNgK52RBxlYLbj0sXsbWmK1BZ4nZ35wvYjXBo9ncak0xRFq2ND2a353JWygatPtOno-sVu2qWasqM1br3frHap2eVy7OGvWLRr1xcVVvnF_U62VrBe8bleZZs3Feq15WzpuVSrO2Llv_KtfnZ82r2lXtsn7RbFSbl_XLZtmCaYHV6V7_eFX_X9f_AYqqfig?type=png)](https://mermaid.live/edit#pako:eNq1WGtv4kYU_SsjSysFlYQECCGo2YoEliJtsm4gpaqQVoM9wCj2DDu2k9KI_947Dz8xj01Uf_Ljvu-5Z2b8ZjncJVbHCsiPiDCH9CheCOxP2ZQhuLATcoGeAiL08wqLkDp0hVmIJv1bhAPUG9loQma_zkTt84mN1z6BbzZekMq2yv0k1rifaIVbzp8pW1SR0ayiO87mdFGi3NPucIhnOCAl37v2cPutPZBacVwDHJJXvNa-6059VeLHtqVG14OndW1CnCUO4S6uiKzF6efPkHwHjQgWzhLNPbpYhggzF60EdwhxUchRthSxMmiB7v2kgwb9MaqttEhQ80m45G6gwor17tU71HV9ytBj5JEAnaJvwx4qk9JFA4Ex9QmPwCsRlLtV9PT4FWo7srX_T5_QAw8J4i9EyGCqMpQRCQLKGRrzZ8ISufsJRKrNdtAjCQUlLyRJSkclVFR5FdmFjMJM9zdOFVE258LHIThM9KTKqalLggfItIq6Po8kJuxvIy0NLuLid18w9fDMIyi2baqYlhpEZbs6qEeDlYfXCO_QKWutR5ywIIZONCoq2_2ECDMNPbba6BfjCCCT7-iUlZX1T-xRFyAclxXVEDQ7CAF68mmGPQwzXFpYe08DlAcp1IPMhwzKECbyj8ThwlWYAz9hFNwMH4bjYXfc7yUtgfC2FKcszV1-1croBmXUp0wp24O4fka3xjh7glm2B8XoeySEDupBORHEpQKK9yS8KlroyTaCQ4D-2dlZJfEhQ3haqdodCtDuP_SGD4OkOsqZMf_ddPg7dUHyreh0U9J4e6D6rgYx7moGxjApkWD7E5R4iWSSq9iRIhsSOpUsDm1bmtMqKIJmSBqKEQyPL9QhkqFeqAvfVhleMsiXBu64v_JImExILGLbp6ZVpoyJYVW3WAyIFN1iB7ANpIQ9bwb3-osyMigdl5qzJYm9FIKjyHFgauaRl37PIa_Y2KxQWZNvRk93d_3RqMycmjMVnonuVKgJCNBJ_69KXiM7X2MKScN4BUEEw1xLgo9UbO6HI4fsvTn1PCl3E4qI7Aw-4V6IWmJAB8YifwZPEjdYsWqqT7wgZfYvAL8jw0VHVPpLd_g1ZootX9gTBLtrZNpRKwonvmWBH7ipJRKwVwGQZ4IEtKUPergkUrvOM-OvkM6CoFcaLtHv47GN6ufnZnMDtbgV_FVOSjw3JVhNRgqmyZELoqHP2lyVCsALnOt56p2MZMVpOjYxluNZSHJWO4UgKJY6LWKW9mIlAyY0W6PtkVGlTfwwHkKhHEJfSrspyyP3IGOBWYAVxYzMIGdltxl4lBv30nEtG67t3B4JUNQWV8R1geUsbyAP0hKgfNwD24Uo4PP3oaEv0VCEQ7zgyhb8ERGxLlAponPoXtJgBdNAU6CROBnf3lWK4R3sE4gdYlUQOYqejNxx3Jozeiy9GqV3MeyHstjDsyC_lyr3-0XHla8I7aLTQ5yZDeIQbUrbzD2Ikb0Q0QjeVVtpzCSW21ZlRHKUVDYPai8kuWrIZALr2hZn_aZWugd-86ZuvnvcwXBu3WxKne2elV2RHQGinx6Hd4_ER8fiY1ntHw959QvM2c8jLLP1Vatx47xesvhK3GTZNc_UhWN0fMZUzQ9qbxoFsBvf7Nwj6eN3BKd3HJCDitly602Vq7bpW-bjzHYLlR5Ls5mqfbk-4hN9qosbeuLS-ZwIeUuE4ALJXzfQcB9YPP3dkqyV85KNXM67OXnkzgieOW3IIKRCqt7fIgpVTnOG-Yk1Uf8Pydh5Rzfzp-Ljm1k4E5e3SRvf38q8TNd1zWKAbL25SM6BGX7duf7WTKcTsf8DKJU0pF1rWJnnQyApt3owoYxZE3xhD5Tahu9W1VoI6lodSThVyydQAvlovUmRqRUuiU-mVgduXSyep9aUbUBnhdnfnPuxmuDRYml15hgirVqaHs3vzuQtFA3afSfPR1ancd2unyszVufN-sfqnF6ct84um63L5mXrunl50Wo2q9Ya3jeur87alxeNRr1-3Wo0LtqtTdX6V_m-OGtfN64bV81W-7LevmpetasWjAssT_f6z6v6Abv5Dwdtfmk)

---

# Retry Job Process Flow Diagram
<!--
[![](https://mermaid.ink/img/pako:eNqdVV1v2jAU_SuWn1opFAiBQlQ6UUhZttFlJKjdhBS5iQtWg01tZy1D_Pc5zgelrTp1ebrOPfecc29iewsjFmNoQ4EfUkwjPCJowdFqToF61ohLEpE1ohL40RLHaYI5QAJMseQb8IXdvsZNrjPAyPdU9Do7usiyHtqssF69Rnjj54gxkvgRbcCROTS94znN8ZWX2vn56MIGP1Ks7KwxjQldKDZdK85uef2c4iep3Q4kOOsDyh6PKprRRU0RVGQ28A4ZQEKELMEJY2vgoGj5UidPv7Q1ubZBwMlioQZWNuPSh5TwjfY1dgJQL53Wt0XkxrtSL3sm14rJG1cNFjxCIpmKiqagrQccUYEiSRj1NeIT4zHmV6y_1UGYsAhJxncHGt64Vtj1DuifY1Aiq6yfRhEW4i5N9vnKa0YzW8fqo1X4KY6UujabE_f92XDo-P4b5dm3dKnAvJIbqHZ-E7npT51g-jN8p3TguaqF735QjqnGtbIAR85NcHxYkqHLtgMS3WMJiBApjkG9Mp7qPuIPdyk1X5jz9YPpzPlYp67vz5wwcIdfnWBfiROxV7tEJHnTWUZbOMuHDS4H7jdn9D-zflmpHfiSJEm5T943wLNdF0YspdIA2TYM8zdILRMk9sv3JIDztCb8X70eSB1wG8Uc-s6N504_Ogg_GAQzPxx-HlyND0ZBC0MqgAZccBJDW_IUG3CF-QplS7jNIHMol3iF59BWYYz4_RzO6U7VqLPuF2OrsoyzdLGE9h1SEzBg_usVR3H1lis1zIdZl9BudhotzQLtLXzK1qcnjUa71-62zdNeq9WxDLiBdqd50jK7p1bTssxmr2P2dgb8o3UbJ92W2WtZjW6z3basptk1II6JOh8m-Y2gL4bdX_cy6yk?type=png)](https://mermaid.live/edit#pako:eNqdVV1v2jAU_SuWn1opFAiBQlQ6UUhZttFlJKjdhBS5iQtWg01tZy1D_Pc5zgelrTp1ebr2Pfecc29iZwsjFmNoQ4EfUkwjPCJowdFqToF61ohLEpE1ohL40RLHaYI5QAJMseQb8IXdvsZNrjPAyPdU9Do7usiyHtqssF69Rnjj54gxkvgRbcCROTS94znN8ZWX2vn56MIGP1Ks7KwxjQldKDZdK85uef2c4iep3Q4kOOsDyh6PKprRRU0RVGQ28A4ZQEKELMEJY2vgoGj5UidPv7Q1ubZBwMlioQZWNuPSh5TwjfY1dgJQL53Wt0XkxrtSL3sm14rJG1cNFjxCIpmKgqYgrQccUYEiSRj1df4T4zHmV6y_1UGYsAhJxncHCt64Vpj1DsifY1Aiq6yfRhEW4i5N9vnKaUYzW8fqlVX4KY6UuraaE_f92XDo-P4b5dmbdKnAvJIbqHZ-E7npT51g-jN8p3TguaqF735QDqnGtbIAR85NcHxYkqHLtgMS3WMJiBApjkG9Mp7qPuIPdyk1X5jz9YPpzPlYp67vz5wwcIdfnWBfiROxV7tEJHnTWUZbOMuHDS4H7jdn9D-zflmpHfiSJEl5St43wLMzF0YspdIA2SEM8x2klgkS--V7EsB5WhP-r14PpA64jWIOfefGc6cfHYQfDIKZHw4_D67GB6OghSEVQAMuOImhLXmKDbjCfIWyJdxmkDmUS7zCc2irMEb8fg7ndKdq1E33i7FVWcZZulhC-w6pCRgw__SKi7ja5UoN82HWJbSb7V5bs0B7C5_UunN60miozW7bPO21Wh3LgBtod5onLbN7ajUty2z2OmZvZ8A_Wrdx0m2ZvZbV6Dbbbctqml0D4pio-2GS_w_0b2H3F9IY6r8)
-->
[![](https://mermaid.ink/img/pako:eNrdV2tv4jgU_SuWpZHaXVoeBdpGQysKKYt2aTMkqLMrJGQSQ60mdsZxOmUR_31s5zE0ZIFq99P6k3Guzz333EfCGrrMw9CAEf4WY-riPkFLjoIpBXKFiAvikhBRAcaPd48OQBGw0CrA6oDNmUjsPn16Zzp6UnZ925K7XaB-1xqWnN7pO0igOYrw7nNrsO17gAT-jlbgpNFrWKdTmthrimc3N_07A5ivmK9ACwSExgJHn-e8evMlVmchph6hS4musSJQBQtEfOyBRez7C-L72kUWmb5J8ZsYY8FXXQE-dwBl309OwS8uooDjgL3iijwSgAQh40KyzQj1784kHU3LyLn7JMoNfMZCYCL3OWOTHKuFfBl0StUqPkxFzwIePRnA4WS5xDx3M6TfYsJXmv7AdEA1i7e6TndDb5PxyFYGaA0MkKqVwkUCiTjaYZDDp-6qDkc0Qq4gjNr6yi3jHuYPrLPWm5nPXCQY32zeY1mDEqn6WMjMFElqZVILO3ZdHEUyc-9tiuUwCT1ZMuVCqvXA5FOZSA6UdRJsx570eqZt70GWpSzZPtpOptMZx66MMwIn5tfT3YvqwlacDnFfsKybKIpl-VXzqGLN1vsvQ9qu7Y7gMd69sFVQSWCq4gl-lSgqc0AkbGkczOUvRD2AAhYXXWM_ykmBe91YH4kDHJmb--7wD7O_x3XWO2rtj_UYDqrmzLeQcJmni1o2VQBaCEkr6xGXY6QKXyoVYMkzwCcuowuyjDma-7ikHj6a03-Uw_xqDcdFPXJNbCEznw2-Q6hpbepxBwgFavaBkPm-UtNduX5J6UjkUo1Lud4mS4-OTNPeM3ZfbgvpVKDFiXNM80vHco2eypM71h2qve_r8wLIkEaY5667csC9ErHqjE1n_OfsIMDeOeGcll3MRoWe7R-ZE0dHn_TzLEHtOOOJWZrDQxIMbXtizpxh73fTKTI5OAzKySZ5AWU9fiSpJC9lACmppCWsspYo58RVP8xcNfAqwEeRmCUnSBzGz0bHv_NTOdjsRwhjO11nYs96v3UfBiXSFPtYx5IkDtxvvUL-n-9E3RZghN66QuAglIqThRRdZTL9RBTPmIIwjp6BYOCLfBP8Crq-1Pmnq1xCuYEVuOTEg4YCr8AA8wCpn3CtTKZQogV4Cg259RB_mcIp3cg78oP3L8aC7Bpn8fIZGgskc1GBiQ7pZ3p-yqU3zHuqaqDRbFzWNAo01vANGo1a-7xdq7Xr9dZVq1lrtBsVuILGWaPWOG_WGxfXrWar3r5qN682Ffi39lw_r19dNluXjdbFdfOydn1dgdgj8rttlPxd0P8aNj8AxuyHkQ?type=png)](https://mermaid.live/edit#pako:eNrdV2tv4jgU_SuWpZHaXVooBUqjoRWFlEW7tBkS1NkVEjKJoVYTO-M4nbKI_z628xgaskC1-2n9yTjX55577iNhDV3mYWjACH-LMXVxn6AlR8GUArlCxAVxSYioAOPHu0cHoAhYaBVgdcDmTCR2nz69Mx09Kbu-bcndLlC_aw1LTu_0HSTQHEV497k12PY9QAJ_RytwUu_VrdMpTew1xbObm_6dAcxXzFegCQJCY4Gjz3NevfkSq7MQU4_QpUTXWBGoggUiPvbAIvb9BfF97SKLTN-k-E2MseCrrgCfO4Cy7yen4BcXUcBxwF5xRR4JQIKQcSHZZoT6d2eSjqZl5Nx9EuUGPmMhMJH7nLFJjtVCvgw6pWoVH6aiZwGPngzgcLJcYp67GdJvMeErTX9gOqCaxVtdp7uht8l4ZCsDtAYGSNVK4SKBRBztMMjhU3dVhyMaIVcQRm195ZZxD_MH1lnrzcxnLhKMbzbvsaxBiVR9LGRmiiS1MqmFHbsujiKZufc2xXKYhJ4smXIh1Xpg8qlMJAfKOgm2Y096PdO29yDLUpZsH20n0-mMY1fGGYET8-vp7kV1YStOh7gvWNZNFMWy_Kp5VLFm6_2XIW3XdkfwGO9e2CqoJDBV8QS_ShSVOSAStjQO5vIXoh5AAYuLrrEf5aTAvW6sj8QBjszNfXf4h9nf4zrrHbX2x3oMB1Vz5ltIuMzTZS2bKgAthKSV9YjLMVKFL5UKsOQZ4BOX0QVZxhzNfVxSDx_N6T_KYX61huOiHrkmtpCZzwbfIdS0NvW4A4QCNftAyHxfqemuXL-kdCRyqcalXG-TpUdHpmnvGbsvt4V0KtDixDmm-aVjuUZP5ckd6w7V3vf1eQFkSCPMc9ddOeBeiVh1xqYz_nN2EGDvnHBOyy5mo0LP9o_MiaOjT_p5lqB2nPHELM3hIQmGtj0xZ86w97vpFJkcHAblZJO8gLIeP5JUkpcygJRU0hJWWUuUc-KqH2auGngV4KNIzJITJA7jZ6Pj3_mpHGz2I4Sxna4zsWe937oPgxJpin2sY0kSB-63XiH_z3eibgswQm9dIXAQSsXJQoquMpl-IopnTEEYR89AMPBFvgl-BV1f6vzTVS6h3MAKXHLiQUOBV2CAeYDUT7hWJlMo0QI8hYbceoi_TOGUbuQd-cH7F2NBdo2zePkMjQWSuajARIf0Mz0_5dIb5j1VNdBo1Jt1jQKNNXyDxsV18_y6fd2ut-u12kWr3mxW4AoaZ_VW47xWbzRazYt2_apx1dpU4N_a8cX55eVlq1WrXdVq7WajdtWuQOwR-d02Sv4u6H8Nmx_OlIeV)
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

<details>
  <summary><h3>Response</h3></summary>
  
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
</details>
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
<details>
  <summary>Request</summary>

  ```json
  {
    "paymentMethod": "WECHAT"
  }
  ```
</details>
  
<details>
    <summary><h3>Response</h3></summary>
  
  ```json
  {
    "paymentId": "123456",
    "redirectUrl": "https://payment-gateway/..."
  }
  ```
</details>
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
  <summary><h3>Example Response</h3></summary>

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
  <summary><h2>Admin rules</h2></summary>
  
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
| txn_date | yyyymmdd |
| order_no | Merchant order number / IATA one order id |
| merchant_ref| merchant unique payment id, idempotency merchant_ref + merchant_id |
| office_id | Office ID |
| merchant_id | dsp mid |
| merchant_name | dsp merchant name |
| gateway | Payment gateway (e.g. 2C2P) |
| gateway_mid | Payment gateway mid (e.g. 2C2P) |
| gateway_payment_id | Payment ID returned by gateway |
| gateway_transaction_id | Transaction ID returned by gateway (if any) |
| payment_category | ECOM |
| payment_type | WALLET / POINTRedeem |
| channel_code | ALIPAY |
| pay_amount | Payment amount |
| ori_amount | Original amount |
| pay_currency | Payment Currency code |
| ori_currency | Original Currency code |
| status | Current payment status (INITIATED, PENDING, SUCCESS, FAILED, EXPIRED) REFUND/REVERSAL another table, post to data brick on SUCCESS, FAILED, EXPIRED |
| sub_status |  |
| gateway_status | status from 2c2p |
| gateway_status_code | status code from 2c2p |
| gateway_status_desc | status desc from 2c2p |
| fullfillment | null/true/false if POST payment-record (EXT) to DAPI is successful after payment success |
| gateway_url | Redirect URL returned by gateway (not mandatory) |
| platform | web, mobile_browser, mobile_ios, mobile_android, mobile_huawei |
| order_locator | Airline PNR |
| order_created_date | Original merchant order created date (not MMB order data) |
| order_contact_name | Contact name of the order |
| order_contact_no | Contact phone no of the order |
| order_contact_email | Contact email of the order |
| expiry_datetime | DSP Payment expiry datetime, Payment Robot will check this field and set to expired during retry |
| retry_count | Number of inquiry retry attempts |
| last_retry_at | Last retry timestamp |
| last_error_code | Latest gateway/system error code |
| last_error_message | Latest gateway/system error message |
| instance_name | masg-10dspmwcap10, masg-1dspmwcap10, mahk-2dspmwca10 to handle Preprod, Prod, Dr environment |
| created_at | Record creation timestamp |
| updated_at | Last update timestamp |
| completed_at | Payment completed timestamp (nullable), on succesful/failed/expired |

-- type (IBE / BOOKING / SSCI / NON IBE, source?)
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
### Do we need database table?

| Question | Payment Table | PaymentActivity | Application Insights |
|----------|:-------------:|:---------------:|:--------------------:|
| What is the payment status now? | ✅ | ❌ | ❌ |
| What happened to this payment? | ❌ | ✅ | Partially |
| Why did the API fail? | ❌ | ❌ | ✅ |
| Which container processed it? | ❌ | ❌ | ✅ |
| How long did 2C2P take to respond? | ❌ | ❌ | ✅ |
| Can I reconcile payments with Finance? | ✅ | ✅ | ❌ |
| Can I recover after restart? | ✅ | ✅ | ❌ |

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

## Edge Cases & Discussion
1. DSP Payment status EXPIRED but callback come late and success
2. If payment has failed, do we need to update DAPI?
3. DSP Payment SUCCESS, Fullfillment failed -> Q30 + Alert

