# 🛡️ DoS Vulnerability Assessment & Mitigation Lab

**Target Application:** OWASP Juice Shop  
**Tools Used:** Kali Linux, Artillery (Node.js), Express Rate Limit  
**Course:** Secure Software Development (Fall 2025)

---

## 📖 Project Overview
This project demonstrates a **Denial of Service (DoS)** simulation against a vulnerable Node.js application (OWASP Juice Shop). The goal was to identify performance bottlenecks, exploit them to crash the server, and implement a code-level defense mechanism to mitigate the attack.

## 🛠️ Tools & Technologies
*   **Target:** [OWASP Juice Shop](https://github.com/juice-shop/juice-shop) (Intentionally insecure web app)
*   **Attack Tool:** [Artillery.io](https://www.artillery.io/) (Load testing framework)
*   **Environment:** Kali Linux (VMware)
*   **Mitigation:** `express-rate-limit` (Node.js Middleware)

---

## 📂 Repository Structure

| File | Description |
| :--- | :--- |
| `products-test.yml` | 📄 Artillery script targeting the **Products Search** endpoint (Heavy Read operation). |
| `login-test.yml` | 📄 Artillery script targeting the **Login** endpoint (Sensitive operation). |
| `feedback-test.yml` | 📄 Artillery script targeting the **Feedback** endpoint (Database Write operation). |
| `products-report.json` | 📊 Raw data showing server crash/high latency **BEFORE** the fix. |
| `products-fixed-final.json` | 📊 Raw data showing HTTP 429 blocked requests **AFTER** the fix. |
| `Lab8_Report.pdf` | 📑 Full documentation with screenshots and OWASP analysis. |

---

## 💥 Vulnerability Analysis (Before Fix)
During the initial stress test, the application exhibited the following behaviors:

1.  **High Latency:** Response times spiked to **>5000ms**.
2.  **Server Crash:** The Node.js process crashed with `FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed` (Heap Out of Memory).
3.  **Connection Timeouts:** Thousands of `ETIMEDOUT` and `ECONNREFUSED` errors were recorded.

**Vulnerabilities Mapped to OWASP Top 10:**
*   **A01: Broken Access Control:** Lack of rate limiting allowed unlimited resource consumption.
*   **A05: Security Misconfiguration:** Verbose error logging revealed stack traces during the crash.

---

## 🛡️ Mitigation Strategy
To fix the DoS vulnerability, I implemented **Rate Limiting** at the application layer.

**Code Changes (`server.ts`):**
I installed `express-rate-limit` (v6) and applied the following middleware to the Express server:

```typescript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes window
  max: 100, // Limit each IP to 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
});

app.use(limiter); // Apply globally
```

*Note: Used version 6 (`npm install express-rate-limit@6`) to avoid 'Trust Proxy' validation errors in the specific Juice Shop environment.*

---

## 📊 Verification Results (After Fix)
After rebuilding and restarting the server, the attack was re-run.

*   **Result:** The server did **not** crash.
*   **Outcome:** 3,050 requests were blocked.
*   **HTTP Code:** The server returned **HTTP 429 (Too Many Requests)**.

| Metric | Before Fix ❌ | After Fix ✅ |
| :--- | :--- | :--- |
| **Status Code** | 200 (then Crash/Timeout) | 429 (Too Many Requests) |
| **Server Status** | Offline / Crashed | Online / Stable |
| **Resource Usage** | 100% CPU / OOM | Normal |

---

## 🚀 How to Run
1.  **Clone the Repo:** `git clone https://github.com/YourUsername/OWASP-JuiceShop-DoS-Lab.git`
2.  **Install Artillery:** `npm install -g artillery`
3.  **Run Attack:** `artillery run products-test.yml`

---

### ⚠️ Disclaimer
*This project is for educational purposes only. The attacks were performed on a locally hosted instance of OWASP Juice Shop in a controlled environment.*
