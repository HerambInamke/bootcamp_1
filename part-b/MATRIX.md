# Impact vs Effort Matrix

This document presents a product management prioritization matrix for the six proposed IRCTC solutions, evaluating the trade-offs between User Impact and Technical Implementation Effort.

---

## The Matrix

| | Low Effort (1–3) | High Effort (4–5) |
|---|---|---|
| **High Impact (4–5)** | **🚀 Quick Wins**<br>• Persistent Search Filters (P2)<br>• Seat Selection State Retention (P3)<br>• Session Timeout Warning (P4)<br>• Mobile Date Picker Redesign (P6) | **🏗 Major Projects**<br>• Tatkal Waiting Queue (P1)<br>• Consolidated PNR Travel Assistant (P5) |
| **Low Impact (1–3)** | **🧩 Fill-Ins**<br>*(None identified in this critical audit)* | **❌ Time Sinks**<br>*(None identified in this critical audit)* |

---

## How I Scored Each Dimension

### Impact Scoring (1–5)
I scored Impact based on:
*   **Number of users affected:** Based on the frequency analysis in Part A.
*   **Core booking flow relevance:** Does it block ticket purchasing or lead to booking failures?
*   **Severity of consequence:** Does it lead to financial loss, missed trips, or accessibility blocks?

### Effort Scoring (1–5)
I scored Effort based on:
*   **System components touched:** Frontend only vs. Backend, Gateway, and Database.
*   **New infrastructure required:** Does it need a queue cluster (Redis), new API integrations, or real-time web socket layers?
*   **Risk of breaking existing flows:** Is it a major architectural change?
*   **Railway API dependencies:** Does it rely on CRIS or NTES external endpoints?

---

## Scoring Summary Table

| Problem | Title | User Impact (1-5) | Tech Effort (1-5) | Quadrant |
|---|---|:---:|:---:|---|
| **P1** | Tatkal Waiting Queue | 5 | 5 | 🏗 Major Project |
| **P2** | Persistent Search Filters | 4 | 2 | 🚀 Quick Win |
| **P3** | Seat Selection State Retention | 4 | 3 | 🚀 Quick Win |
| **P4** | Session Timeout Warning | 4 | 2 | 🚀 Quick Win |
| **P5** | Consolidated PNR Travel Assistant | 4 | 4 | 🏗 Major Project |
| **P6** | Mobile Date Picker Redesign | 4 | 2 | 🚀 Quick Win |

---

## Placement Justifications

### P1: Tatkal Waiting Queue — 🏗 Major Project (Impact: 5, Effort: 5)
*   **Impact Justification:** This solution directly targets the 10:00 AM server crash affecting 20–40 lakh users daily, resolving the single biggest performance blocker on the platform.
*   **Effort Justification:** Implementing a virtual waiting room requires a dedicated Redis BullMQ system, WebSocket gateways, and a decoupled asynchronous billing pipeline.
*   **Prioritisation Meaning:** This is a core architectural milestone that must be scheduled and designed carefully with a dedicated backend sprint.

### P2: Persistent Search Filters — 🚀 Quick Win (Impact: 4, Effort: 2)
*   **Impact Justification:** Eliminating filter resets saves 8–15 minutes of manual scrolling for 8 crore users, making the search flow significantly cleaner.
*   **Effort Justification:** The implementation is entirely frontend-driven, using local router states and `sessionStorage` with zero database or backend API dependencies.
*   **Prioritisation Meaning:** This should be built in the first sprint as it provides immediate usability returns with negligible technical risk.

### P3: Seat Selection State Retention — 🚀 Quick Win (Impact: 4, Effort: 3)
*   **Impact Justification:** Retaining selected seats protects families and senior citizens from being separated, improving checkout confidence for 30–40% of sessions.
*   **Effort Justification:** It requires a lightweight Redis cache layer to manage temporary 10-minute TTL locks on seats, coupled with a frontend state reference.
*   **Prioritisation Meaning:** A high-value feature that can be deployed independently within a single sprint.

### P4: Session Timeout Warning — 🚀 Quick Win (Impact: 4, Effort: 2)
*   **Impact Justification:** Adding a countdown bar and warning modal prevents sudden form resets and checkout drop-offs for 15–20% of users.
*   **Effort Justification:** The solution requires a frontend React hook and modal container, with a single lightweight auth API heartbeat endpoint to refresh tokens.
*   **Prioritisation Meaning:** A straightforward UX enhancement that solves a major source of customer support complaints.

### P5: Consolidated PNR Travel Assistant — 🏗 Major Project (Impact: 4, Effort: 4)
*   **Impact Justification:** Consolidating live location, platform numbers, and coach positions on one page eliminates passenger anxiety and the need for third-party apps.
*   **Effort Justification:** This requires integrating external NTES APIs, resolving caching policies, and designing responsive SVG coach layout components.
*   **Prioritisation Meaning:** A major value-add project that should be scheduled as the next core feature sprint once baseline UX is stabilized.

### P6: Mobile Date Picker Redesign — 🚀 Quick Win (Impact: 4, Effort: 2)
*   **Impact Justification:** Large 48px touch targets and gesture swiping eliminate date selection misclicks for mobile web users who represent >60% of total site traffic.
*   **Effort Justification:** It is a frontend-only responsive design improvement with zero backend code changes.
*   **Prioritisation Meaning:** An essential accessibility upgrade that can be launched immediately alongside the search filter fixes.

---

## Recommended Sprint Order

1.  **Sprint 1: Baseline Mobile UX (P2 & P6)**
    *   *Why:* Persistent search filters and the redesigned mobile date picker are both frontend-only changes. They solve massive day-to-day search friction with near-zero backend risk.
2.  **Sprint 2: Session and Booking State Stabilization (P3 & P4)**
    *   *Why:* Implementing session timeout warnings and seat lease state management solves checkout drop-offs and seat reset issues, securing the core passenger entry flow.
3.  **Sprint 3: High-Throughput Infrastructure (P1)**
    *   *Why:* Designing the Tatkal virtual queue requires significant backend preparation, testing, and load-simulation, which should build on top of a stable frontend booking flow.
4.  **Sprint 4: Post-Booking Experience (P5)**
    *   *Why:* The consolidated PNR assistant is a post-booking utility. While highly impactful, it is sequenced last as it is not blocking the core ticket-purchasing funnel.
