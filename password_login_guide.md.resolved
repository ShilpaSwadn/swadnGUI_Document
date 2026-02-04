# 🔐 Password Login Flow - Step-by-Step Guide

## 🎯 What This Guide Covers

This guide explains **exactly how users log in with a password** using either their **Email** or **Mobile Number**.

---

## 🔄 Overview

```
User enters credentials → Resolve Email (if mobile) → Firebase Auth → Check Verification → Sync with Server → Login Success
```

---

## 📝 Step-by-Step Flow

### **STEP 1: User Enters Credentials** 👤

**Location:** [app/login/page.jsx](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/app/login/page.jsx) (Frontend)

**User enters:**
- **Identifier:** Email address OR Mobile Number (10 digits)
- **Password:** Their password

---

### **STEP 2: Handle Login Submission** 🚀

**Location:** [app/login/page.jsx](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/app/login/page.jsx) (Frontend)

**What happens:**
- Frontend detects if input is Email or Mobile
- Calls [loginWithPasswordDirect()](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js#173-222)

**Code:**
```javascript
const handlePasswordLogin = async (e) => {
  // ... validation ...
  await loginWithPasswordDirect(identifier, password);
}
```

---

### **STEP 3: Resolve Mobile to Email (If needed)** 📱➡️📧

**Location:** [lib/services/auth.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js) (Frontend)

**Why:** Firebase's `signInWithEmailAndPassword` **ONLY** accepts emails. It does **NOT** accept phone numbers.

**Logic:**
- **If Email:** Proceed directly.
- **If Mobile:** Call API to find the email attached to this phone number.

**Code:**
```javascript
if (!identifier.includes('@')) {
  // It's a mobile number, ask server for the email
  const response = await api.post('/auth/check-status', { identifier });
  targetEmail = response.email; 
}
```

**Server API (`/auth/check-status`):** Uses Firebase Admin SDK to look up user by phone number and return their email.

---

### **STEP 4: Authenticate with Firebase** 🔥

**Location:** [lib/services/auth.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js) (Frontend)

**What happens:**
- Client SDK sends credentials to Firebase.
- Firebase validates password (bcrypt).
- Returns User Credential if valid.

**Code:**
```javascript
const userCredential = await signInWithEmailAndPassword(
  auth, 
  targetEmail, 
  password
);
const user = userCredential.user;
```

---

### **STEP 5: Check Email Verification** ✅

**Location:** [lib/services/auth.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js) (Frontend)

**What happens:**
- **CRITICAL:** We verify if the user's email is actually verified.
- If not verified, we **block** the login and sign them out immediately.

**Code:**
```javascript
await user.reload(); // Get latest status

if (!user.emailVerified) {
  await signOut(auth); // Kick them out
  throw new Error("Email not verified. Please verify your email first.");
}
```

---

### **STEP 6: Synchronize with Server (The "Sync" Step)** 📡

**Location:** [lib/services/auth.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js) (Frontend → Backend)

**Why:** We need our PostgreSQL database to know the user just logged in and ensure their data is up-to-date.

**What happens:**
1.  Frontend gets Firebase **ID Token**.
2.  Sends ID Token to `/api/auth/login`.

**Code:**
```javascript
const idToken = await user.getIdToken();
const response = await api.post('/auth/login', { idToken });
```

---

### **STEP 7: Server Verifies & Syncs** 🖥️

**Location:** [app/api/auth/login/route.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/app/api/auth/login/route.js) (Backend)

**What happens:**
1.  **Verify Token:** Backend uses Admin SDK to verify the ID token is valid and coming from Firebase.
2.  **Sync User:** Upserts (Update or Insert) user data into **PostgreSQL** `users` table.

**Code:**
```javascript
// 1. Verify Token
const decodedToken = await authService.verifyFirebaseToken(idToken);

// 2. Sync to Postgres
const user = await authService.syncUser(decodedToken, profileData);
```

---

### **STEP 8: Login Success & Redirect** 🎉

**Location:** [app/login/page.jsx](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/app/login/page.jsx) (Frontend)

**What happens:**
- Session data saved to LocalStorage.
- User redirected to Dashboard.

**Code:**
```javascript
router.push('/dashboard');
```

---

## 📊 Summary

| Step | Action | Logic Location | Key Tech |
|---|---|---|---|
| 1 | User enters ID/Pass | Browser | React |
| 2 | Resolve Mobile->Email | API (`check-status`) | Admin SDK |
| 3 | Authenticate | Browser | **Firebase Client SDK** |
| 4 | Check Verification | Browser | Firebase User Object |
| 5 | Sync User | API (`/auth/login`) | **PostgreSQL** |
| 6 | Redirect | Browser | Next.js Router |

**Key Takeaway:**
We use **Firebase** for security (passwords) but **PostgreSQL** for our main application data. The "Sync" step connects them!
