# 📚 Complete Registration Flow Guide for Beginners

## 🎯 What This Guide Covers

This guide explains **exactly how user registration works** in your application, step-by-step, using the **Firebase Admin SDK** (Current Implementation).

---

## 🔥 Current Implementation: WITH Firebase Admin SDK

### **Overview: The Complete Journey**

```
User fills form → Frontend validates → API creates account → Email sent → User verifies → Login enabled
```

### **Step-by-Step Flow**

#### **STEP 1: User Fills Registration Form** 👤

**Location:** [app/register/page.jsx](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/app/register/page.jsx) (Frontend - Browser)

**What happens:**
- User enters:
  - First Name (required)
  - Last Name (optional)
  - Email (required)
  - Mobile Number (required, 10 digits)
  - Password (required, min 6 characters)
  - Confirm Password (required)

**Code:**
```javascript
const [formData, setFormData] = useState({
  firstName: '',
  lastName: '',
  email: '',
  mobileNumber: '',
  password: '',
  confirmPassword: ''
});
```

---

#### **STEP 2: Frontend Validation** ✅

**Location:** [app/register/page.jsx](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/app/register/page.jsx) (Frontend)

**What happens:**
- Validates all fields before sending to server
- Checks:
  - Email format is valid
  - Password is at least 6 characters
  - Passwords match
  - Mobile number is exactly 10 digits

**Code:**
```javascript
const validation = validateRegisterForm(formData);
if (!validation.isValid) {
  setError(Object.values(validation.errors)[0]);
  return;
}
```

---

#### **STEP 3: Call Registration Function** 📞

**Location:** [lib/services/auth.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js) (Frontend)

**What happens:**
- Frontend calls [register()](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js#71-138) function
- This function coordinates the entire registration process

**Code:**
```javascript
const response = await register({
  firstName: formData.firstName.trim(),
  lastName: formData.lastName.trim(),
  email: formData.email.trim().toLowerCase(),
  mobileNumber: formData.mobileNumber.trim(),
  password: formData.password
});
```

---

#### **STEP 4: Check Email/Mobile Uniqueness** 🔍

**Location:** [lib/services/auth.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js) (Frontend → API)

**What happens:**
- Calls API to check if email already exists in Firebase
- Calls API to check if mobile number already exists in Firebase
- **Uses Firebase Admin SDK** to search all users

**Code:**
```javascript
// Check Email
const emailStatus = await api.post('/auth/check-status', {
  identifier: email.trim().toLowerCase()
});
if (emailStatus.exists) {
  throw new Error("Email already registered");
}

// Check Mobile
const mobileStatus = await api.post('/auth/check-status', {
  identifier: mobileNumber.trim()
});
if (mobileStatus.exists) {
  throw new Error("Mobile already registered");
}
```

**API Endpoint:** `app/api/auth/check-status/route.js`
- Uses `adminAuth.getUserByEmail()` ✅ (Firebase Admin)
- Uses `adminAuth.getUserByPhoneNumber()` ✅ (Firebase Admin)

---

#### **STEP 5: Create User in Firebase Auth** 🔐

**Location:** [lib/services/auth.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js) (Frontend)

**What happens:**
- Creates user account in Firebase Authentication
- **Client SDK** creates the account
- User is automatically logged in (temporarily)

**Code:**
```javascript
const userCredential = await createUserWithEmailAndPassword(
  auth, 
  email, 
  password
);
const user = userCredential.user;
```

**Result:**
- User created with:
  - UID (unique ID)
  - Email
  - Password (hashed by Firebase)
  - `emailVerified: false` (not verified yet)

---

#### **STEP 6: Set Display Name** 👤

**Location:** [lib/services/auth.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js) (Frontend)

**What happens:**
- Updates user's display name in Firebase
- Combines first name + last name

**Code:**
```javascript
await updateFirebaseProfile(user, {
  displayName: `${firstName} ${lastName}`.trim()
});
```

---

#### **STEP 7: Set Mobile Number (Admin SDK)** 📱

**Location:** [lib/services/auth.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js) → `app/api/auth/set-mobile/route.js`

**What happens:**
- **Cannot set phone number from client!**
- Calls API endpoint that uses **Firebase Admin SDK**
- Admin SDK updates user's phone number

**Frontend Code:**
```javascript
await api.post('/auth/set-mobile', {
  uid: user.uid,
  mobileNumber: mobileNumber.trim()
});
```

**Backend Code (API):**
```javascript
await adminAuth.updateUser(uid, {
  phoneNumber: mobileNumber.startsWith('+') 
    ? mobileNumber 
    : `+91${mobileNumber}`
});
```

**Why Admin SDK?**
- ✅ Client SDK **cannot** set phone number
- ✅ Only Admin SDK can update phone number
- ✅ This is a privileged operation

---

#### **STEP 8: Send Email Verification** 📧

**Location:** [lib/services/auth.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js) (Frontend)

**What happens:**
- Firebase sends verification email to user
- Email contains a verification link
- User must click link to verify email

**Code:**
```javascript
await sendEmailVerification(user);
```

**Email sent by:** Firebase (automatic)
**Email contains:** Verification link

---

#### **STEP 9: Sign Out User** 🚪

**Location:** [lib/services/auth.js](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/lib/services/auth.js) (Frontend)

**What happens:**
- User is signed out immediately
- They cannot login until email is verified

**Code:**
```javascript
await signOut(auth);
```

**Why?**
- User must verify email first
- Prevents unverified users from accessing the app

---

#### **STEP 10: Show Success Message** ✅

**Location:** [app/register/page.jsx](file:///c:/Users/ADMIN/OneDrive/Desktop/SwadnGUI/frontend/app/register/page.jsx) (Frontend)

**What happens:**
- Shows success screen
- Tells user to check their email
- Provides "Go to Login" button

**Code:**
```javascript
setSuccessMsg(
  'Account created! Please check your email to activate your account.'
);
```
