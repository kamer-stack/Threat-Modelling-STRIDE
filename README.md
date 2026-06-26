# 🧩 Threat Modelling with STRIDE — Three Progressive Scenarios

> Application of the STRIDE threat modelling framework across three progressively complex systems using OWASP Threat Dragon, covering authentication, payment API integration, and a microservices healthcare platform.

![Security](https://img.shields.io/badge/Category-Threat%20Modelling-blueviolet)
![Framework](https://img.shields.io/badge/Framework-STRIDE-blue)
![Tool](https://img.shields.io/badge/Tool-OWASP%20Threat%20Dragon-orange)
![Threats](https://img.shields.io/badge/Threats%20Identified-20%2B-critical)
![Institution](https://img.shields.io/badge/Institution-PUCIT-green)

---

## 📋 Overview

Threat modelling is a proactive security discipline that identifies and mitigates threats at the **design stage** — before they reach production. This project applies the **STRIDE framework** (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege) across three scenarios of increasing complexity, using **OWASP Threat Dragon** for Data Flow Diagrams (DFDs).

The scenarios escalate from a basic login system to a full microservices-based healthcare application with multi-stage attack chains.

---

## 🎯 Scenarios

---

### Scenario 1 — User Authentication System *(Easy)*

**System:** Simple web application with username/password login, session tokens, relational database.

**DFD Components:** User → Web Server → Session Manager → User Database

**Trust Boundary:** External user vs internal application infrastructure

| # | Threat | STRIDE Category |
|---|--------|----------------|
| 1 | Credential Spoofing (no MFA) | Spoofing |
| 2 | Session Token Tampering | Tampering |
| 3 | Credential Leak from Database | Information Disclosure |
| 4 | Login Endpoint Flooding | Denial of Service |

**Key mitigations:** TOTP-based MFA, HMAC-signed JWTs, bcrypt/Argon2 password hashing, rate limiting + CAPTCHA

---

### Scenario 2 — Third-Party Payment API Integration *(Medium)*

**System:** E-commerce platform with payment gateway REST API, internal transaction database, and third-party email service.

**Trust Boundaries:** Customer ↔ Frontend; Backend ↔ Third-party services

| # | Threat | STRIDE Category | Severity |
|---|--------|----------------|----------|
| 1 | API Key Theft via Code Repository | Spoofing | High |
| 2 | Payment Data Tampering in Transit | Tampering | High |
| 3 | Transaction Repudiation | Repudiation | Medium |
| 4 | Card Data Exposure in Backend Logs | Information Disclosure | High |
| 5 | Email Service Abuse for Phishing | Elevation of Privilege | Medium |
| 6 | Payment Gateway Denial of Service | Denial of Service | Medium |

**Critical trust boundary violation:** Backend ↔ Payment Gateway — stolen API key gives attacker full backend identity at the gateway; no way to distinguish legitimate from attacker.

**Key mitigations:** HashiCorp Vault for secrets, mutual TLS + certificate pinning, PCI-DSS compliant immutable audit logs, DMARC/DKIM/SPF

---

### Scenario 3 — Microservices Healthcare Application *(Hard)*

**System:** Patient management platform with 6 microservices — Patient Portal, API Gateway, Identity Service, Medical Records Service, Appointment Service, Notification Service. All patient data is PHI subject to HIPAA.

**Trust Boundaries:**
- TB1: Public Internet ↔ DMZ
- TB2: DMZ ↔ Internal Services
- TB3: Internal Services ↔ Each Other

| # | Threat | STRIDE | Severity | Component |
|---|--------|--------|----------|-----------|
| 1 | JWT Token Forgery | Spoofing | Critical | Identity Service, API GW |
| 2 | Patient Record Tampering | Tampering | Critical | Medical Records Service |
| 3 | No Audit Trail for PHI Access | Repudiation | High | Medical Records, Identity |
| 4 | JWT Signing Secret Exposure | Information Disclosure | Critical | Identity Service |
| 5 | PHI Leakage via Notification Payload | Information Disclosure | High | Notification Service |
| 6 | API Gateway DDoS | Denial of Service | High | API Gateway |
| 7 | OAuth Token Replay Attack | Spoofing | High | Identity Service |
| 8 | Appointment IDOR | Tampering | Medium | Appointment Service |
| 9 | Broken Access Control on Medical Records | Elevation of Privilege | Critical | Medical Records Service |
| 10 | SMS Provider Interception | Tampering | Medium | Notification Service |

---

## ⛓️ Attack Chain Analysis

### Chain 1: Full PHI Exfiltration via Token Compromise

```
JWT Secret Exposed (T4)
    → Forge JWT with admin role (T1)
        → Bypass API Gateway, access Medical Records directly (T9)
            → All patient PHI exfiltrated
```

**Mitigation:** Asymmetric JWT signing (RS256), per-service authorization, anomaly detection on unusual role claims.

---

### Chain 2: DDoS-Masked Data Exfiltration

```
API Gateway DDoS (T6) — consumes security team attention
    → OAuth Token Replay while team is distracted (T7)
        → Trigger bulk PHI export via Notification Service (T5)
            → PHI silently exfiltrated under cover of DDoS noise
```

**Mitigation:** Automated DDoS mitigation independent of human response, short-lived tokens, DLP on Notification Service.

---

## 📁 Repository Structure

```
threat-modelling-stride/
├── README.md
├── report/
│   └── Threat_Modelling_Assignment_Khadija_Amer.pdf
└── diagrams/
    ├── scenario1_auth_system.json       (Threat Dragon DFD)
    ├── scenario2_payment_api.json       (Threat Dragon DFD)
    └── scenario3_healthcare_micro.json  (Threat Dragon DFD)
```

---

## 💡 Key Lessons

- Trust boundaries are the most critical analysis points in any DFD — threats are most severe where systems of differing trust levels meet
- No single control is sufficient; **defence-in-depth** is essential for sensitive systems
- Architectural complexity amplifies risk — a vulnerability in one microservice can cascade into systemic compromise when inter-service trust is implicit
- Threat modelling delivers maximum value when performed **iteratively during design**, not after deployment

---

## ⚠️ Disclaimer

All systems modelled in this project are fictional scenarios created for educational purposes as part of a university assignment.

---

## 👩‍💻 Author

**Khadija Amer**
Punjab University College of Information Technology (PUCIT)  
Course: Information Security | Spring 2026  
Instructor: Sir Shehryar Raza
