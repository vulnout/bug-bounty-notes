# Security Vulnerability Report — Taskbear

**Reported by:** Abhinab  
**Date:** 02-11-2026

---

## Vulnerability

**OTP Verification Bypass via Response Manipulation**

**Endpoint:** `taskbear.com.au` (POST method)

---

## Steps to Reproduce

1. Navigate to [https://taskbear.com.au/auth/sign-up](https://taskbear.com.au/auth/sign-up?redirect=%2F) and initiate email verification.
2. Enter an invalid OTP and intercept the verification response using Burp Suite.
3. Observe the original `401 Unauthorized` response indicating an invalid or expired OTP.
4. Modify the response status code to `200 OK` and replace the response body with one obtained from a successful verification.
5. Forward the modified response to the browser.
6. Observe that the application accepts the manipulated response and allows registration to proceed.

---

## Impact

An attacker may be able to bypass the frontend email verification flow without supplying a valid OTP. If subsequent backend operations do not independently enforce email verification, this could allow account registration using unverified email addresses.

---

## Proof of Concept

### 1. Registration Page

<p align="center">
  <img src="https://github.com/user-attachments/assets/17a6b7a1-791c-4091-ae82-be56a3761769" width="720" alt="Registration page">
</p>

<br>

### 2. Successful Account Creation (Valid OTP)

<p align="center">
  <img src="https://github.com/user-attachments/assets/deac672d-696d-445f-a78a-750c7742cda9" width="720" alt="Successful account creation">
</p>

<br>

### 3. Entering an Incorrect OTP

<p align="center">
  <img src="https://github.com/user-attachments/assets/34e4434c-aef5-4db6-a760-efa30d2a42fc" width="720" alt="Incorrect OTP submission">
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/95228346-0bb3-4b5b-b52a-33b8c99b26f5" width="720" alt="Invalid OTP response">
</p>

<br>

### 4. Modified Response (Status Code Changed to 200 OK)

<p align="center">
  <img src="https://github.com/user-attachments/assets/90937253-36e2-43f5-8c3c-257c723496b0" width="720" alt="Modified response">
</p>

<br>

### 5. Successful Bypass

<p align="center">
  <img src="https://github.com/user-attachments/assets/265b6a41-b007-45f1-9c79-46457ce195b4" width="720" alt="OTP bypass demonstrated">
</p>

---

## Remediation

1. Enforce OTP validation exclusively on the server.
2. Maintain verification state server-side (session or database flag).
3. Never rely on client-side responses to establish email verification.
4. Validate email verification status before allowing registration to complete.
