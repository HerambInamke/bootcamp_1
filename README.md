# IRCTC Sprint Audit & Rescue Plan (`irctc-sprint`)

This repository contains an evidence-based audit and design sprint plan to address critical, high-impact user experience and technical bottlenecks on India's primary railway booking platform, IRCTC (irctc.co.in).

Rather than proposing a superficial visual rebranding, this audit documents real, reproducible broken user flows experienced by over 8 crore registered users daily. It lays down a technical blueprint to rescue the platform from decades of accumulated UX debt and architectural constraints.

## Repository Structure

```directory
irctc-sprint/
├── README.md
├── part-a/
│   └── PROBLEMS.md    # Thorough problem discovery document detailing 6 major issues
├── part-b/            # Engineering specifications and priority matrices (Part B)
│   ├── SPECS.md       # Technical specs for fixing the audited problems
│   ├── AI-FEATURE.md  # Intelligent AI feature proposals
│   └── MATRIX.md      # 2x2 prioritization and impact matrix
└── assets/
    └── screenshots/   # Visual evidence and screenshots of failures
```

## Section Details

### [Part A: Problem Discovery (part-a/PROBLEMS.md)](file:///c:/Projects/bootcamp_1/part-a/PROBLEMS.md)
Contains granular, 5-part audits for 6 key IRCTC issues, documenting what is broken, who is affected, frequency of failure, step-by-step flows, and precise technical failure points:
1. **Tatkal Booking Crashes at 10:00 AM** (Performance/UX)
2. **Search Filters Reliability Issues** (UX/Information Architecture)
3. **Seat Selection Resets** (UX/Mobile)
4. **Aggressive & Silent Session Timeout** (UX/Performance)
5. **Incomplete & Unsaved PNR Status Journey Information** (Information Architecture/UX)
6. **Mobile Web Date Picker Layout & Touch Target Violations** (Mobile/Accessibility)

### [Part B: Feature Specs & Matrix (part-b/)](file:///c:/Projects/bootcamp_1/part-b/)
*(To be completed in Part B)*
*   **SPECS.md**: Concrete technical and UX specifications for addressing the 6 problems.
*   **AI-FEATURE.md**: Product specification for an AI-powered assistant/feature to streamline the booking flow.
*   **MATRIX.md**: A 2x2 prioritization matrix evaluating User Impact vs. Technical Complexity to guide implementation sequencing.