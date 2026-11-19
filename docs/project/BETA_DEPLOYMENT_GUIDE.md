# My Health App — Zero-Cost Production Beta Deployment Guide

This guide walks you through deploying the My Health App backend for free ($0/month) and building a standalone shareable Android APK for beta testers.

---

## Architecture Overview

```
┌─────────────────────────────────┐
│     Mobile App (Android APK)    │
│    Built with Expo & React Native│
│  - Dynamic API URL              │
│  - JWT Bearer Auth Storage      │
│  - Branded Icon & Theme         │
└────────────────┬────────────────┘
                 │ HTTPS
                 ▼
┌─────────────────────────────────┐
│     Backend Web Service         │
│     Hosted on Render (Free)     │
│  - Docker Container             │
│  - Express / Node.js 22         │
│  - Built-in Database JWT Auth   │
│  - Auto Database Migrations     │
└────────────────┬────────────────┘
                 │ PostgreSQL (SSL)
                 ▼
┌─────────────────────────────────┐
│     Database Layer (Free Tier)  │
│        Hosted on Supabase       │
│  - 500 MB Free PostgreSQL       │
│  - Automated Prisma Migrations  │
└─────────────────────────────────┘
```

---

## Step 1: Set Up Supabase Database (100% Free)

1. Go to [supabase.com](https://supabase.com) and sign in with GitHub.
2. Click **New Project**:
   - **Name:** `myhealthapp-beta`
   - **Database Password:** Choose a strong password (save this).
   - **Region:** Select a region close to your users (e.g., *East US*, *Central EU*, or *South Asia / Mumbai*).
   - **Pricing Plan:** Free tier ($0).
3. Once the database is provisioned (approx. 1-2 minutes):
   - Navigate to **Project Settings** (gear icon) -> **Database**.
   - Scroll down to **Connection string**.
   - Select **URI** mode.
   - The connection string will look like:
     ```
     postgresql://postgres.[PROJECT_REF]:[YOUR_PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres?sslmode=require
     ```
     *(Or direct connection: `postgresql://postgres:[YOUR_PASSWORD]@db.[PROJECT_REF].supabase.co:5432/postgres?sslmode=require`)*
4. Replace `[YOUR_PASSWORD]` with your database password.

---

## Step 2: Deploy Backend to Render (100% Free)

Render provides free Docker-based Web Services with automatic HTTPS.

### Option A: Using the Render Dashboard (One-Click)
1. Go to [render.com](https://render.com) and sign in.
2. Click **New +** -> **Web Service**.
3. Connect your GitHub repository: `https://github.com/jai38/myhealthapp-backend`.
4. Render will auto-detect the `Dockerfile`:
   - **Name:** `myhealthapp-backend`
   - **Region:** Choose a region matching or close to Supabase.
   - **Instance Type:** **Free** ($0/month).
5. In **Environment Variables**, add:
   - `NODE_ENV`: `production`
   - `PORT`: `4000`
   - `API_PREFIX`: `/api/v1`
   - `AUTH_PROVIDER`: `node-internal`
   - `CLIENT_URL`: `*`
   - `JWT_SECRET`: Generate a random 32+ character string (e.g., `healthapp-prod-jwt-super-secret-key-32chars`)
   - `STORAGE_PROVIDER`: `memory`
   - `ALLOW_EPHEMERAL_STORAGE`: `true`
   - `DATABASE_URL`: Paste the Supabase connection string from Step 1.
6. Click **Create Web Service**.
7. Render will build the Docker container and execute:
   `npx prisma migrate deploy && node dist/index.js`
   When finished, Render gives you a live public HTTPS URL:
   `https://myhealthapp-backend.onrender.com`
8. Verify health check by visiting:
   `https://myhealthapp-backend.onrender.com/healthz` (returns `{"status":"ok"}`).

---

## Step 3: Build Shareable Android APK (EAS Build)

The mobile repository is already configured with `myhealthapp/eas.json` to generate an installable `.apk` file (no Google Play Developer account required).

1. Install EAS CLI globally if not already installed:
   ```bash
   npm install -g eas-cli
   ```
2. In the `myhealthapp` directory, log into your Expo account (create a free account at expo.dev if needed):
   ```bash
   cd myhealthapp
   eas login
   ```
3. Update `myhealthapp/eas.json` with your live Render backend URL:
   ```json
   "env": {
     "EXPO_PUBLIC_API_URL": "https://myhealthapp-backend.onrender.com/api/v1"
   }
   ```
4. Trigger the APK preview build:
   ```bash
   eas build -p android --profile preview
   ```
5. EAS will run the build in the cloud. When complete:
   - EAS provides a download URL for the standalone `.apk`.
   - Download the `.apk` file and share it via Google Drive, WhatsApp, Slack, or direct download link.
   - Testers can install the `.apk` on any Android device immediately by enabling "Install from unknown sources".

---

## Step 4: Testing & Verification Checklist

Once the APK is installed and the backend is live:

1. **Authentication:**
   - Open the app, navigate to **Profile** tab.
   - Tap **Sign In or Create Account**.
   - Switch to **Create Account**, enter name, email, and password.
   - Verify that your user is registered and the badge changes to **Authenticated**.
2. **Prescription Upload:**
   - Navigate to **Prescriptions** tab.
   - Upload a prescription photo or document.
   - Verify AI extraction preview and confirm prescription.
3. **Medical Timeline:**
   - Navigate to **Timeline** tab.
   - Verify confirmed clinical events appear chronologically.
4. **Medications & Reminders:**
   - Navigate to **Medications** tab.
   - Verify medications extracted from the prescription are present.
   - Log a dose as **Taken**.
5. **Medical Report PDF Export:**
   - Navigate to **Profile** -> **Generate Medical PDF Report**.
   - Download or share the doctor-ready PDF.
