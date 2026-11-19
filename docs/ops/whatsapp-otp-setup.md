# WhatsApp OTP setup

**Owner decision, 19 November 2026 (D1, revised):** sign-in uses a mobile number plus a one-time code sent **by WhatsApp only** (S01–S02). There is no SMS for now, so no DLT registration is needed yet; `dlt-registration.md` is kept for when an SMS fallback is added. This is the owner's task. Meta's steps and prices change, so check each one in Meta Business Manager before paying.

## What you need

- **A registered business,** for Meta business verification (GST, incorporation, Udyam, or a utility bill in the business name), plus the business website and email.
- **A phone number used only for WhatsApp Business.** It must be able to receive a verification call or SMS once, and it can't be on the normal WhatsApp app at the same time.
- **The display name** "Ayoreni".

## Steps

1. **Create a Meta Business account** (business.facebook.com) and start **business verification** (Business Settings → Security Center). It usually takes a few days to about two weeks.
2. **Choose how to send.**
   - **Meta Cloud API directly:** no middleman fee, and you pay Meta's per-message rate. The backend calls Meta's API.
   - **A provider** (Gupshup, Twilio, MSG91, Interakt and others): easier onboarding, but they add a markup.
   - **AWS End User Messaging Social:** WhatsApp through your AWS account, so billing and keys stay on AWS.
   - Recommendation: **AWS End User Messaging Social or the Meta Cloud API**, to avoid a markup. Decide this with the backend session.
3. **Add the WhatsApp Business phone number** to the account, and get the **display name "Ayoreni" approved**.
4. **Create an "Authentication" message template.** Meta fixes the wording ("{code} is your verification code."). You choose:
   - Add the security line ("For your security, do not share this code.").
   - Add the expiry line ("This code expires in 10 minutes.").
   - The button type: **zero-tap** or **one-tap autofill** on Android, and **copy code** for iPhone. Android autofill needs the app's package name and signing-key hash:
     - **Package name:** `com.ayoreni.app`. The dev build is a separate app, `com.ayoreni.app.dev`, and needs its own entry if it should autofill too. (UI session, 19 November 2026.)
     - **Signing-key hash: not ready yet.** CI builds the APK with `expo prebuild` and `gradlew assembleRelease`, which signs it with the public React Native debug key. A hash from that key proves nothing. First set up a real release key (EAS-managed, or a keystore kept as a CI secret); then the UI session computes the hash. Until then, create the template with **copy code** only and add autofill later.
   - Use English first; add Hindi templates later.
   - Approval usually takes minutes to hours.
5. **Create the access token.** A permanent token comes from a system user in Business Settings. Store it in AWS Parameter Store yourself, never in chat or git, the same way as the Gemini key. For AWS End User Messaging Social, you link the WhatsApp account in AWS instead.
6. **Record the non-secret identifiers in this file:** WhatsApp Business Account ID, phone number ID, template name and language, and the chosen route. The backend session then builds `POST /auth/otp/request` and `POST /auth/otp/verify` in Phase 5.

## What this means for the build (Phase 5)

- **Security:** rate limits per number and per device; a 10-minute expiry; codes stored only as a hash; lockout after repeated wrong codes; no phone numbers or codes in logs.
- **A number with no WhatsApp:** the app needs a calm "We couldn't send a WhatsApp code to this number" state, with help text. There is no SMS fallback yet, so this case must be designed: ask the designer to add the state to S01/S02.
- **After the first sign-in, offer a passkey or app lock,** because a phone number alone is weak against number takeover.
- **Cost watch:** add a messaging spend alarm next to the AWS budget alarm.

## Status

| Step | Done | Notes |
|---|---|---|
| Meta Business account created | | |
| Business verification approved | | |
| Sending route chosen | | |
| WhatsApp number added, display name approved | | |
| Authentication template approved (name, language) | | |
| Token stored in Parameter Store | | |
