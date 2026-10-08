# 📘 LearningSteps — Origins

## Overview
Manual deployment of the LearningSteps journal API on **Microsoft Azure** using a secure two-tier architecture. This project represents Phase 1 of the LearningSteps series and Part 1 of a three-part cloud security series.

**Original Curriculum:** [CyberstepsDE/learningsteps](https://github.com/CyberstepsDE/learningsteps)  
**My Implementation:** Two-tier Azure architecture with NSG segmentation, jump host pattern, PostgreSQL configuration.

---

## 🎯 Objective
Deploy a FastAPI + PostgreSQL journal application on Azure with:
- Public-facing API tier (web VM)
- Isolated database tier (private VM, no public IP)
- Network security enforced via NSGs at both subnet levels

---

## 🏗️ Architecture

                        Internet                           │
                    ┌──────▼───────┐
                    │  Web-vm-ip   │  (Public IP address)
                    └──────┬───────┘
                           │
┌──────────────────────────▼────────────────────────────────┐
│ LearningSteps-vn (VNet) 10.0.0.0/16                       │
│                                                           │
│ ┌─────────── Public subnet 10.0.0.0/24 ───────────────┐   │
│ │  NSG-sg (NSG — API tier)                            │   │
│ │  Inbound: SSH(22, key-auth) · HTTP(8000)            │   │
│ │  ┌────────────────────┐                             │   │
│ │  │      Web-vm        │──► FastAPI Journal API      │   │
│ │  └────────┬───────────┘                             │   │
│ └───────────┼──────────────────────────────────────────┘  │
│             │ (jump-host SSH admin, SSH only via web-vm)  │
│ ┌───────────▼─── Private subnet 10.0.1.0/24 ──────────┐   │
│ │  NSG-db (NSG — Database tier)                       │   │
│ │  Inbound: SSH(22) · PostgreSQL(5432)                │   │
│ │           — source restricted to Web-vm private IP  │   │
│ │  ┌────────────────────┐                             │   │
│ │  │       DB-vm        │──► PostgreSQL :5432         │   │
│ │  │  (no public IP)    │    10.0.1.4                 │   │
│ │  └────────────────────┘                             │   │
│ └─────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────┘

---

## 🔒 Security Decisions

| Decision | Rationale |
|----------|-----------|
| **Two subnets (public/private)** | Segmentation. Compromising web tier doesn't grant direct internet access to database |
| **DB VM has no public IP** | Database unreachable from internet by construction |
| **Private NSG: inbound 5432 allowed only from web VM** | Least privilege. Only app needs database access |
| **Public NSG: SSH key-based only** | Removes password brute-force risk |
| **Dedicated DB user with limited privileges** | Limits SQL injection blast radius |

---

## 🛠️ Key Implementation Steps

1. **Database Server (Private Tier)**
   - Installed PostgreSQL 16 with JSONB + GIN indexes for flexible API schema
   - Configured `pg_hba.conf` to allow connections from web VM's private IP only
   - Enabled auto-start: `sudo systemctl enable postgresql.service`

2. **Application Server (Public Tier)**
   - Cloned reference implementation from upstream repo
   - Configured `.env` pointing to DB's private IP
   - Started with `./start.sh` (uvicorn)

3. **Verification**
   - POST/GET requests verified via `/docs` Swagger interface
   - Database isolation confirmed: no public endpoint, NSG enforcement
   - Jump host pattern for admin access to private VM

---

## ⚠️ Challenges

- **Git workflow with upstream branches:** Fetching and tracking the reference branch correctly
- **Key-based SSH:** Ensuring keys work consistently across both tiers

---

## 📊 Success Criteria

| Criterion | Status |
|-----------|--------|
| API reachable via public IP | ✅ |
| DB unreachable from public internet | ✅ |
| CRUD via API endpoints | ✅ |
| Data persistence (survives reboot) | ✅ |
| Least-privilege NSGs | ✅ |

---

## 💡 Key Learnings

> A resource group in Azure is the organizational boundary. Azure security enforcement happens at NSG level, complementing OS-level controls like `pg_hba.conf`. Defense in depth means both must agree.

> The jump host pattern trades convenience for security — acceptable for administration, not for production apps.

---

## 🚀 Next Stage
Phase 2 automated this architecture using Terraform + Kubernetes + CI/CD with security gates. See: [LearningSteps — Evolution](https://github.com/e-Itohan/learningsteps-evolution)

---

## 📃 Full Report
[View](https://github.com/e-Itohan/e-Itohan/blob/main/reports/LearningSteps_Origins.pdf)
