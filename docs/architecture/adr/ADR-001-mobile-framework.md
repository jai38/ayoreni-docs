# ADR-001: Mobile Framework — React Native with Expo & Expo Router

**Date:** 2026-10-12  
**Status:** proposed (Gate 1)  
**Deciders:** Principal Architect & Mobile Engineering Lead  

---

## Context

My Health App requires cross-platform mobile delivery for iOS and Android. The mobile client must support:
1. Native camera capture and multi-page document picking.
2. High-performance document zoom/pan previewing.
3. Resilient local device notification scheduling that fires reliably while offline.
4. Hardware-backed secure token storage (`Keychain` / `KeyStore`).
5. Native OS share sheets for PDF exports.
6. A calm, accessible, modern health-tech design system adhering to WCAG 2.2 Level AA.

We need a framework that provides high developer velocity, cross-platform code reuse, type safety, and seamless access to native device primitives without unnecessary native build brittleness.

---

## Decision

We adopt **React Native** with **Expo (Managed Workflow)** and **Expo Router** using **TypeScript in strict mode**.

The directory structure in `myhealthapp/` will follow:
- `app/`: Thin Expo Router file-based route definitions.
- `components/`: Reusable atomic UI primitives (Buttons, Cards, Inputs, Modals, Badges).
- `features/`: Domain-oriented feature modules (`auth/`, `prescriptions/`, `timeline/`, `medications/`, `reports/`).
- `services/`: API client, local notification scheduler, and secure storage adapter.
- `theme/`: Design tokens (typography, color palette, spacing, elevation).

---

## Alternatives Considered

### Alternative 1: Bare React Native (CLI)
- **Pros:** Full manual control over Xcode and Android Studio projects.
- **Cons:** High native maintenance overhead; fragile dependency linking; slower upgrade cycles; lacks file-based routing out-of-the-box.
- **Why rejected:** Managed Expo now supports Config Plugins and Prebuild for any native module requirement, eliminating the historical need for bare workflows.

### Alternative 2: Flutter (Dart)
- **Pros:** High performance canvas rendering, uniform cross-platform behavior.
- **Cons:** Requires a separate language ecosystem (Dart); does not share TypeScript domain models and Zod schemas with the Node.js backend.
- **Why rejected:** Prompt specifies React Native with TypeScript as an approved requirement.

---

## Consequences

### Positive
- Over 90% shared code between iOS and Android.
- File-based routing with deep linking support via Expo Router.
- Built-in access to vetted native modules (`expo-camera`, `expo-document-picker`, `expo-notifications`, `expo-secure-store`, `expo-sharing`).
- TypeScript end-to-end type safety shared with backend schemas.

### Negative
- OTA updates and custom native libraries require EAS Build or local prebuild configurations.

### Risks & Mitigations
- **Risk:** Background notification throttling by Android OEM battery savers.
- **Mitigation:** Implement explicit user permission walkthroughs and battery optimization exemption guides in onboarding.
