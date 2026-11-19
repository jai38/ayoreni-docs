# DLT registration for phone OTP (India)

> **Deferred (19 November 2026):** launch uses WhatsApp OTP only (see `whatsapp-otp-setup.md`). Use this checklist only if an SMS fallback is added later.

**Owner decision, 19 November 2026 (D1):** sign-in uses a mobile number plus an OTP sent by SMS (S01–S02). In India, every business SMS must go through TRAI's DLT system, so this has to be done before Phase 5 can go live. It is the owner's task. Fees, timings and rules change: check each step on the portal before paying.

## Before you start: what you need

- **A registered business.** Individuals generally can't register as a sender. Any one of: certificate of incorporation (Pvt Ltd or LLP), GST certificate, Udyam registration, or shop and establishment licence.
- **The business PAN,** plus the authorised signatory's ID proof (PAN or Aadhaar).
- **A letter of authorisation** on the company letterhead, naming the person who will manage DLT. Every portal has a template.
- **A company email address and mobile number** for the account. A business-domain email helps approval.
- **The brand name exactly as it should appear,** for the sender ID.

## Steps

1. **Register as a Principal Entity (PE)** on ONE operator's DLT portal: Jio TrueConnect, Airtel DLT, Vodafone Idea "Vilpower", BSNL or Tata/Smartping. A PE registration is shared across all operators, so one is enough.
   - Upload the documents above and pay the fee. It's usually a few thousand rupees plus GST, and some portals charge yearly.
   - You get an **Entity ID**, normally within about 1 to 7 working days.
2. **Register a header (sender ID).**
   - For OTP and service SMS it is **6 letters**, linked to the brand, for example `AYRENI` or `AYORNI`. Pick two or three options in case one is taken.
   - Category: **Service Implicit**. OTPs to your own users are "service implicit". The "Transactional" category is reserved for banks.
3. **Register the content template(s).**
   - Each SMS text must match an approved template exactly, with `{#var#}` for the parts that change. Use no links (a URL must be whitelisted separately), and keep the brand name in the text. Suggested template (the second variable is the Android auto-read hash):
     ```
     {#var#} is your Ayoreni verification code. It expires in 10 minutes. Do not share it with anyone. {#var#}
     ```
   - Category: Service Implicit, type OTP. Approval usually takes 1 to 3 days. Note the **Template ID**.
4. **Choose an SMS provider** (the "telemarketer"), for example MSG91, Gupshup, Kaleyra or Twilio's Indian route. AWS End User Messaging also sends to India once you register the Entity ID and Template IDs there.
   - Compare the price per SMS (Indian aggregators are usually a fraction of a rupee per OTP), delivery speed, and a fallback such as WhatsApp or voice OTP.
   - Ask whether they handle DLT "chain binding". Under the newer TRAI rules, the PE has to approve its telemarketer chain on the DLT portal.
5. **Link the provider to your PE** on the DLT portal (telemarketer binding or chain declaration), and give the provider your Entity ID, header and Template ID(s).
6. **Send to Claude only** the non-secret identifiers, recorded in this repo: the Entity ID, header and Template ID, plus the provider's name.
   - The provider's API key goes into AWS Parameter Store, done by you, never in chat or git, in the same way as the Gemini key.
   - The backend session then builds `/auth/otp/request` and `/auth/otp/verify` in Phase 5.

## What this means for the build (Phase 5)

- **Android auto-read:** OTP autofill on Android uses the SMS Retriever API, which needs an 11-character app hash in the message. The debug and release builds have different hashes, which is why it's a template variable.
- **Security, required for health data:**
  - rate limits per number and per device;
  - OTP expiry (10 minutes), with the code stored only as a hash;
  - lockout after repeated wrong codes;
  - no OTPs or phone numbers in logs.
  - A phone number alone is weaker against SIM swap, so Phase 5 should still offer a passkey or app lock after the first sign-in.
- **Cost watch:** add an SMS spend alarm next to the AWS budget alarm.

## Status

| Step | Done | Notes |
|---|---|---|
| Business documents ready | | |
| PE registered (Entity ID) | | |
| Header approved | | |
| OTP template approved (Template ID) | | |
| SMS provider chosen and linked | | |
| Key stored in Parameter Store | | |
