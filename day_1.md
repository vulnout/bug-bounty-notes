
**Day 01 – Burp Suite Fundamentals & Client-Side Response Manipulation**

**Category:** Web Application Security  
**Focus:** HTTP basics, Burp Suite, authentication testing  
**Tools:** Burp Suite, temporary email  
**Target:** TaskBear (https://taskbear.com.au)

---

### 1. HTTP Fundamentals

HTTP is a request-response protocol.  
Client sends a request → Server processes it → Client receives a response.

```
Client  →  HTTP Request  →  Server
Client  ←  HTTP Response ←  Server
```

Everything we test in Burp (requests, responses, status codes, bodies) lives inside this model.

---

### 2. HTTP Methods (quick reference)

| Method | Purpose |
|--------|---------|
| GET | Retrieve data |
| POST | Submit data (login, registration, OTP check) |
| PUT | Full replacement of a resource |
| PATCH | Partial update |
| DELETE | Remove a resource |

In authentication flows we mostly care about **POST**.

---

### 3. HTTP Status Codes (the ones that matter today)

| Range | Meaning | Common examples |
|-------|---------|-----------------|
| 2xx | Success | 200 OK, 201 Created |
| 4xx | Client error | 401 Unauthorized, 403 Forbidden, 404 Not Found |
| 5xx | Server error | 500, 502, 503 |

During OTP testing we specifically watched for **401 → 200** changes.

---

### 4. Burp Suite – What we actually used

Burp sits between the browser and the server as an intercepting proxy.

Useful today:
- **Proxy → Intercept** – catch requests and responses live
- **Proxy → HTTP history** – review everything that happened
- Ability to edit a response before the browser sees it

That last point is the entire technique of the day.

---

### 5. Practical – Email OTP Verification Bypass (Response Manipulation)

**Goal:** Check whether the registration flow trusts the HTTP response it receives for the OTP check, instead of a server-side verified state.

**High-level steps performed:**

1. Open TaskBear sign-up and request an OTP to a temporary email.
2. Submit a deliberately wrong OTP with Burp intercept on.
3. Observe the real server response: `HTTP/2 401 Unauthorized` + body saying the OTP is invalid/expired.
4. Intercept the same response again, change status to `200 OK` and replace the body with a success-shaped payload (contact + verifiedAt).
5. Forward the modified response to the browser.
6. Continue the registration flow (mobile step was skippable).
7. Account was created successfully even though the correct OTP was never entered.

**Important clarification (kept deliberate):**  
Editing the response only changes what the *browser* sees. It does **not** prove the server’s own verification record was updated. The finding shows client-side trust in the response. Whether the backend independently re-checks verification state at account-creation time was not tested in this exercise.

---

### 6. Key Takeaways

- The frontend treated a forged 200 + success body as proof of verification.
- Subsequent registration steps proceeded as if the email was verified.
- Real impact depends on whether the account-creation endpoint itself enforces a server-side “OTP verified” flag. That check was outside the scope of Day 1.

---

### 7. Security Impact (observed)

- An attacker who can intercept their own traffic can make the UI accept an unverified email.
- This weakens the guarantee that a registered email is controlled by the account owner.
- It is **not** a remote attack against other users; the attacker must control the responses to their own requests.

---

### 8. Recommended Mitigations

1. OTP verification decision must live only on the server.
2. Account-creation endpoint must re-check a server-side verified flag (session-bound).
3. Never treat a client-visible status code or response body as the source of truth.
4. OTP should be single-use and rate-limited.
5. Re-test the final registration request in isolation to confirm server-side enforcement.

---

### 9. Learning Outcomes

After Day 1 you should be able to:

- Explain the basic HTTP request-response model
- Recognise the main HTTP methods and status-code families
- Configure Burp as an intercepting proxy
- Intercept and edit both requests and responses
- Test whether an OTP / verification step is enforced only client-side
- Distinguish “UI accepted the forged response” from “server state was actually changed”

---

### 10. Conclusion

Day 1 introduced the core toolkit (HTTP + Burp) and demonstrated a classic client-side trust issue: the registration UI accepted a manipulated success response for an incorrect OTP and allowed the flow to complete. The exercise reinforces the fundamental rule — authentication and verification decisions belong on the server, not in the browser.

---

**End of Day 01 notes**
