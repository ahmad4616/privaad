# privaad
Decentralized Privacy-Preserving Ad Engine
### Enterprise-Grade Homomorphic Encryption & Kinematic Anti-Fraud System

---

## 1. ABSTRACT
This repository contains a production-ready, highly secure infrastructure for zero-knowledge ad targeting and advanced click-fraud prevention. By eliminating cookie dependency, the architecture utilizes client-side Homomorphic Encryption (HE) via WebAssembly to calculate ad relevancy scores on a remote server without decrypting the underlying user profile. Concurrently, it implements a kinematic mouse-tracking algorithm inside a Manifest V3 browser extension to preemptively intercept non-human traffic (bots) before ad interaction occurs. 

---

## 2. SYSTEM ARCHITECTURE

The system is split into two core decoupled components designed to interact securely over authenticated protocols:

### A. Decentralized Client (Chrome Extension - Manifest V3)
Functions as an isolated decentralized node executing user profiling and cryptographic operations locally.
*   `content.js` (UI Monitor): Injected into webpage contexts to track non-invasive browser heuristics and compute an ephemeral user interest vector (`sessionStorage` bound).
*   `background.js` (Cryptographic Service Worker): Operates in a secure, isolated extension context. It receives data from `content.js` via local runtime messaging, executes WebAssembly (WASM) homomorphic encryption routines, and handles network handshakes with the backend API, bypassing rigid host-website Content Security Policies (CSP).

### B. Secure Matching Server (FastAPI Backend)
An encrypted-data computation engine optimized for blind vector mathematics.
*   **Dynamic Data Tier:** Built on a production-hardened SQLite interface for quick, concurrent read operations, designed for seamless migration to PostgreSQL/MySQL.
*   **Defense Layers:** Protected by a dedicated middleware stack including SlowAPI for rate limiting (DDoS defense), domain-restricted CORS rules, and structural API Key header enforcement (`X-API-Key`).

---

## 3. CORE TECHNOLOGICAL INNOVATIONS

### A. Homomorphic Encryption (HE)
To preserve maximum data privacy, raw user profile arrays are never transmitted over the wire in plaintext. The user vector is encrypted locally using the CKKS scheme. The FastAPI backend receives the serialized context and ciphertext, then computes the encrypted Dot Product against targeted campaign vectors loaded dynamically from the secure database.

Mathematically, given an encrypted user profile vector $U$ and an unencrypted ad campaign target vector $V$, the backend calculates the match score $S$ blindly:

$$S = \mathbf{E}(U) \cdot V$$

The server streams back the resulting ciphertext $\mathbf{E}(S)$ to the client extension, which decrypts the result locally to inject the winning banner asset.

### B. Kinematic Fraud Detection
To mitigate programmatic click fraud, the extension records high-frequency mouse coordinates during webpage interactions. The client-side script parses the directional changes to compute a Cumulative Deviation Angle, measuring human micro-jitters against the mathematically perfect trajectories typical of automated scraping or automated script bots.

The system determines the mean angular variance over a sliding kinematic window:

$$\Delta \theta = \frac{1}{N-2} \sum_{i=2}^{N} \left| \arctan\left(\frac{y_i - y_{i-1}}{x_i - x_{i-1}}\right) - \arctan\left(\frac{y_{i-1} - y_{i-2}}{x_{i-1} - x_{i-2}}\right) \right|$$

Where a resulting $\Delta \theta$ close to zero denotes artificial linear path tracking, causing the engine to halt event bubble propagation, cancel the redirect event, and dispatch an out-of-band fraud report to log the abusive IP addresses.

---

## 4. APPLIED SECURITY MEASURES

1.  **Zero-Knowledge Compliance:** The backend server does not hold the private decryption key, making data harvesting impossible during an infrastructure breach.
2.  **Isolated WebAssembly CSP:** The `manifest.json` profile safely configures `wasm-unsafe-eval` strictly inside the extension's execution layer, preventing arbitrary script injection vulnerabilities.
3.  **Rate Limiting & Threat Mitigation:** Restricts endpoints to a configurable threshold (e.g., 5 requests per minute per IP), preventing enumeration attacks and server resource exhaustion.
4.  **Advanced Event Capturing:** The anti-fraud mouse listener hooks into the DOM's early `Capture Phase`, intercepting malicious click behaviors prior to any webpage scripts processing the event.

---

## 5. DEPLOYMENT & OPERATION GUIDE

### Server Infrastructure Requirements
*   Python 3.9 or higher
*   Core dependencies: `fastapi`, `uvicorn`, `tenseal`, `slowapi`, `pydantic`

### Initializing the Core Server
```bash
# 1. Install dependencies
pip install fastapi uvicorn tenseal slowapi pydantic

# 2. Launch production-grade ASGI cluster
uvicorn server:app --host 0.0.0.0 --port 8000 --workers 4


