# 📲 OTP Login Flow - Step-by-Step Guide

## 🎯 What This Guide Covers
This guide explains the dual-strategy for OTP Login:
1.  **Email OTP:** Handled by **Backend (Stateless)** using Firebase Admin & Hash Verification.
2.  **Mobile OTP:** Handled by **Frontend (Firebase SDK)** using Native Phone Auth.

---

## 🔄 High-Level Architecture

| Feature | Email Login ✉️ | Mobile Login 📱 |
|---|---|---|
| **Initiator** | Backend API | Client SDK |
| **Storage** | Stateless (HMAC Hash) | Firebase Internal |
| **Delivery** | Firebase Extension (Firestore) | Firebase SMS Gateway |
| **Verification** | Backend Recomputes Hash | Firebase SDK Checks Code |
| **Final Auth** | Custom Token | ID Token |

---

## � Flow 1: Email OTP (Backend Stateless)

### **STEP 1: User Requests Code**
**User:** Enters Email -> Clicks "Get Code".
**Frontend:** Calls `POST /api/auth/otp/send`.

### **STEP 2: Backend Signs & Sends**
**Backend:**
1.  **Validation:** Checks if user exists via `adminAuth.getUserByEmail`.
    *   *Error if not registered.*
2.  **Signing:** Creates a cryptographic signature (Hash) of `email + otp + expiry`.
    *   `Hash = HMAC_SHA256(email + otp + time)`
3.  **Delivery:** Adds document to Firestore `mail` collection -> **Trigger Email Extension** sends it.
4.  **Response:** Returns `{ success: true, hash: "..." }`.

### **STEP 3: User Verifies**
**User:** Enters 6-digit code.
**Frontend:** Calls `POST /api/auth/otp/verify` with `{ email, otp, hash }`.
**Backend:**
1.  **Recompute:** Calculates the hash again using the input.
2.  **Compare:** If `NewHash === OldHash`, it's valid.
3.  **Login:** Generates **Firebase Custom Token**.

---

## 📱 Flow 2: Mobile OTP (Frontend Native)

*Since Firebase Admin SDK cannot directly send SMS without external providers (Twilio, etc), we use the Client SDK for the best experience.*

### **STEP 1: User Requests Code**
**User:** Enters Mobile Number -> Clicks "Get Code".
**Frontend:**
1.  **Check Registration:** Calls `/api/auth/check-status`. If user not found, stops.
2.  **Recaptcha:** Verifies user is human (Invisible Recaptcha).
3.  **Firebase:** Calls `signInWithPhoneNumber(auth, number, verifier)`.

### **STEP 2: Firebase Sends SMS**
**Google/Firebase:** Handles the SMS delivery infrastructure automatically.

### **STEP 3: User Verifies**
**User:** Enters 6-digit code.
**Frontend:**
1.  **Confirm:** Calls `confirmationResult.confirm(otp)`.
2.  **Result:** User is signed in directly on the client.
3.  **Sync:** (Optional) Frontend calls `/api/auth/sync` to ensure Postgres is up to date.

---

## 📊 Summary of Backend Safety (Email)
We use a **Stateless Hash** approach for Email OTP:
- **No DB Storage:** We don't save the OTP in the database.
- **Security:** The matching Hash proves we generated the OTP recently.
- **Expiry:** The timestamp in the hash ensures codes expire after 10 minutes.

**Why different flows?**
- **Email:** We want full control and custom email templates, executed securely by the backend.
- **Mobile:** Firebase's native SMS is free/cheap and handles carrier complexity better than a custom backend solution.
