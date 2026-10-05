# Password-Strength-Analyzer-Security-Suggestion-Tool
**Password Strength Analyzer &amp; Security Suggestion Tool** is a privacy-focused cybersecurity tool that locally analyzes password strength, detects common patterns, provides security suggestions, generates secure passwords, and offers privacy-safe analytics—without storing passwords.
# 🛡️ Password Strength Analyzer & Security Suggestion Tool

> Privacy-focused cybersecurity tool for evaluating password strength using length, predictability, common-password checks, pattern analysis, entropy concepts, and personalized security recommendations.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue) ![Flask](https://img.shields.io/badge/Backend-Flask-black) ![SQLite](https://img.shields.io/badge/Analytics-SQLite-003B57) ![Chart.js](https://img.shields.io/badge/Charts-Chart.js-ff6384) ![Focus](https://img.shields.io/badge/Focus-Defensive%20Security-success)

---

## Overview

Most password meters only check whether a password contains an uppercase letter, a lowercase letter, a number, and a symbol. That approach is misleading: `Password123!` passes every composition rule yet is one of the most predictable passwords possible.

This project evaluates passwords the way modern guidance (NIST SP 800-63B, OWASP ASVS) recommends:

```
LENGTH + UNPREDICTABILITY + PATTERN RESISTANCE + COMMON-PASSWORD CHECKS + CONTEXT = BETTER ASSESSMENT
```

Everything runs **in memory**. Submitted passwords are never stored, logged, or sent to external services.

## Problem Statement

Weak and reused passwords remain a leading cause of account takeover and credential-stuffing attacks. Users overestimate the strength of passwords that look complex but follow human-predictable patterns. Security teams need transparent, explainable feedback that coaches users at the moment of password creation, without collecting the passwords themselves.

## Objectives

- Score passwords from 0–100 with a transparent, explainable rubric.
- Detect common passwords, sequences, keyboard walks, repetition, predictable word+number structures, and personal-information overlap.
- Explain weaknesses with specific, actionable suggestions (never echoing the password).
- Provide a cryptographically secure password generator.
- Evaluate an admin-configurable password policy separately from strength.
- Offer a privacy-safe analytics dashboard built only from aggregate metadata.
- Demonstrate secure password-storage concepts (salted `scrypt`) in a separate, educational function.

## Cybersecurity Relevance

| Area | How this project relates |
|---|---|
| Authentication / IAM | Strength meters, policy enforcement, just-in-time coaching |
| Application Security | Secure input handling, no secret logging, rate limiting, validation |
| SOC / Awareness | Reducing credential-stuffing and account-takeover exposure |
| Secure Coding | Privacy by design, `secrets` CSPRNG, one-way hashing with salt |

Roles this project supports: Cybersecurity Analyst, Application Security Analyst, IAM Analyst, Security Engineer, SOC Analyst, Secure Software Developer.

## Features

- Real-time strength meter (updates as you type, no page refresh)
- Score (0–100) and classification: `VERY WEAK` · `WEAK` · `MODERATE` · `STRONG` · `VERY STRONG`
- Length analysis, character diversity, unique-character ratio
- Common-password detection (local educational list, with leetspeak normalization)
- Sequence detection (ascending/descending, e.g. `1234`, `dcba`)
- Keyboard-pattern detection (`qwer`, `asdf`, `1qaz`, …)
- Repetition detection (repeated characters and substrings)
- Predictable word + number/year structure detection
- Optional personal-context check (name, birth year, college/company), compared locally and never stored
- Entropy-style estimate with clear limitations
- Specific security suggestions and passphrase guidance
- Show/hide password toggle (hidden by default)
- Secure password generator (16 / 20 / 24 characters)
- Password policy checker (`POLICY PASS` / `POLICY FAIL`), separate from strength
- Privacy-safe analytics dashboard with 5 charts
- Educational scrypt hashing demo (`hash_password` / `verify_password`)
- 34 automated tests including privacy tests

## Architecture

```
User
 ↓
Web Interface (type="password" field)
 ↓
POST /api/analyze  (password processed in memory only)
 ↓
┌──────────────────────────────┐
│ Length Analyzer              │
│ Character Analyzer           │
│ Common Password Checker      │
│ Sequence Detector            │
│ Keyboard Pattern Detector    │
│ Repetition Detector          │
│ Predictable-Structure Check  │
│ Context Checker              │
│ Entropy Estimator            │
└──────────────────────────────┘
 ↓
Strength Scoring Engine → Classification → Suggestion Engine → User

Optional (user clicks "Add anonymous result"):
Result metadata only → SQLite → Dashboard
```

The password itself never reaches the analytics layer: `record()` receives only the analysis *result*.

## Technology Stack

| Layer | Technology |
|---|---|
| Backend | Python 3.10+, Flask |
| Analysis engine | Pure Python (`re`, `math`, `secrets`, `hashlib`) |
| Analytics | SQLite (metadata only) |
| Frontend | HTML, CSS, vanilla JavaScript |
| Charts | Chart.js |
| Runtime | Google Colab (single cell) + Cloudflare quick tunnel for a shareable URL |

## Password Analysis

`analyze_password(password, context=None)` returns:

```json
{
  "score": 32,
  "classification": "WEAK",
  "findings": [{"type": "keyboard", "severity": "medium", "description": "..."}],
  "strengths": [],
  "suggestions": ["..."],
  "metrics": {"length": 11, "unique_ratio": 0.909, "theoretical_entropy_bits": 71.8},
  "policy": {"status": "POLICY FAIL", "rules": []},
  "pattern_count": 2
}
```

## Length Analysis

| Length | Band |
|---|---|
| < 8 | Very short |
| 8–11 | Short |
| 12–15 | Better length |
| 16+ | Strong length contribution |

Length alone does not make a password secure: `aaaaaaaaaaaaaaaa` is long but highly predictable, so repetition caps its score.

## Pattern Detection

| Detector | Examples |
|---|---|
| `detect_sequences()` | `1234`, `abcd`, `9876`, `dcba` |
| `detect_keyboard_patterns()` | `qwer`, `asdf`, `zxcv`, `1qaz` |
| `detect_repetition()` | `aaaa`, `ababab`, `abcabcabc` |
| `detect_predictable_structure()` | `welcome123`, `admin2026`, `P@ssw0rd!` |
| `check_context()` | Name, birth year, or organization inside the password |

## Common Password Detection

A small educational list is stored in `data/common_passwords.txt` (no leaked credentials or personal data). Matching includes simple leetspeak normalization (`@→a`, `0→o`, `$→s`, …).

## Entropy Estimation

```
Entropy ≈ L × log2(N)      L = length, N = estimated character pool
```

**Limitation:** this assumes characters are chosen *randomly*. Human-created passwords are not random, so `Password123!` receives an optimistic theoretical value despite being predictable. Entropy is therefore used only as one signal and is combined with pattern analysis. The displayed guess-space label is an **educational estimate only**, not a "time to crack" claim: real resistance depends on the attacker model, predictability, hashing algorithm, work factor, rate limiting, and online vs. offline scenario.

## Strength Scoring

| Component | Max points |
|---|---|
| Length | +35 |
| Character diversity | +15 |
| Unique-character ratio | +10 |
| Pattern resistance (−7 per pattern finding) | +20 |
| Not a common password | +10 |
| Additional unpredictability (entropy, only when pattern-free) | +10 |

| Penalty | Points |
|---|---|
| Common password | −50 |
| Keyboard pattern | −15 |
| Repetition | −15 |
| Common word + number/symbol tail | −15 |
| Personal information | −20 |
| Sequence | −12 |

Transparent caps keep predictable passwords from scoring high on length alone:

| Condition | Maximum score |
|---|---|
| Exact/leet common password | 15 |
| Common-password base + suffix | 40 |
| Dictionary-word base + suffix | 55 |
| Unique-character ratio < 0.35 | 30 |
| Personal information found | 50 |

| Score | Classification |
|---|---|
| 0–20 | VERY WEAK |
| 21–40 | WEAK |
| 41–60 | MODERATE |
| 61–80 | STRONG |
| 81–100 | VERY STRONG |

> These are project-defined scoring bands, not universal security standards. Password strength cannot be perfectly determined by a single formula.

## Security Suggestions

Suggestions are specific and never include the user's password, for example:

- "A predictable numeric/letter sequence was detected."
- "A keyboard pattern (like qwerty or asdf) was detected."
- "Avoid names, birth years, and college/company names."
- "Use a password manager to generate and store a unique password for every account."
- "Enable MFA wherever available."

## Password Generator

Built with Python's `secrets` module and a `SystemRandom` shuffle (OS-level CSPRNG), not `random`, because `random` is predictable and unsuitable for security-sensitive values. Options: length 16/20/24 and uppercase/lowercase/numbers/symbols. Generated passwords are never stored.

## Password Policy Checker

Configurable via `CFG`:

```python
CFG = dict(min_length=12, max_length=128, common_check=True, personal_check=True)
```

Displays `POLICY PASS` / `POLICY FAIL` separately from the strength score, because policy compliance and password strength are different concepts. Aligned with modern guidance: no forced periodic rotation, spaces allowed, long maximum length, common-password rejection.

## Privacy Design

- Passwords are processed in memory only; never written to disk, database, or logs.
- Request logging is minimized; the API never echoes input, even in error messages.
- The password is sent in a JSON POST body, never in a URL.
- The frontend uses `type="password"`, does not use `localStorage`/`sessionStorage`, and only records metadata when the user explicitly clicks the button.
- The database has **no password column** and no password hashes. Even hashes of arbitrary user-submitted passwords create unnecessary privacy risk for an analyzer.
- Analytics store only: score, classification, length, unique-character ratio, weakness count, pattern count, finding types, timestamp.
- In production, serve over HTTPS.

## Installation

This project is designed to run as a **single Google Colab cell**.

1. Open [Google Colab](https://colab.research.google.com/) and create a new notebook.
2. Paste the full project code into one cell.
3. Run the cell (Flask is installed automatically).

## Usage

After the cell finishes, it prints test results and two links:

- 🔎 **Analyzer:** `https://<random>.trycloudflare.com/`
- 📊 **Dashboard:** `https://<random>.trycloudflare.com/dashboard`

Then:

1. Type a **synthetic** password and watch the live meter.
2. Optionally enter demo context (name, year, college).
3. Review findings, suggestions, and the policy result.
4. Generate a secure demo password.
5. Click **Add anonymous result to dashboard** and view the aggregate charts.

> The link is public and is active only while the Colab runtime is running. Use synthetic demo passwords only.

## API Documentation

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/analyze` | Analyze a password (processed transiently) |
| POST | `/api/generate-password` | Generate a secure password |
| GET | `/api/dashboard/stats` | Aggregate dashboard statistics |
| GET | `/api/analytics/weaknesses` | Weakness-type frequency |

**POST `/api/analyze`**

```json
{ "password": "<processed transiently>", "record": false,
  "context": { "name": "", "year": "", "org": "" } }
```

Status codes: `200` success · `400` invalid input (generic message, no echo) · `429` rate limit exceeded (240 requests/min per IP).

## Testing

34 automated checks (T01–T34) run automatically when the cell executes, covering empty/short/numeric/common/repeated/mixed passwords, sequences, keyboard patterns, repetition, word+number, personal-info overlap, passphrases, Unicode, spaces, max length, score boundaries, suggestions, generator, and hashing.

## Security Testing

| Check | Verified by |
|---|---|
| Password not in API response | T30 |
| Password not in database dump | T31 |
| No `password` column in schema | T32 |
| Analytics contain metadata only | T33 |
| Password not handled via URL parameters | T34 |

Manual checks: password field type, no `localStorage`/`sessionStorage` usage, no console logging of the password.

## Results

Expected classifications for synthetic test inputs (confirm against your own run):

| Input | Classification | Reason |
|---|---|---|
| `123456` | VERY WEAK | Common, short, numeric, sequential |
| `Password123!` | WEAK | Diverse characters but common word + predictable tail |
| `aaaaaaaaaaaaaaaa` | WEAK | Long but highly repetitive |
| `qwerty2026!` | WEAK | Keyboard pattern + predictable year |
| Random 20-char generated value | STRONG / VERY STRONG | Long, varied, pattern-free |

## Limitations

- Scoring is heuristic and project-defined; it is not a guarantee of security.
- The common-password list is small and educational, not a full breach corpus.
- Entropy assumes random selection and overestimates human-chosen passwords.
- No live breach-corpus lookup (see Future Improvements).
- The public Colab link is temporary and intended for demonstration only.

## Future Improvements

- Integrate a mature estimator (e.g., zxcvbn-style analysis)
- Privacy-preserving k-anonymity breach checks
- Configurable enterprise password policies
- Password-manager and IAM/SSO integration concepts
- MFA, passkey, and WebAuthn awareness modules
- Localization and accessibility improvements
- Organization-level aggregate reporting without collecting passwords
- FastAPI + React version

## Screenshots

| Screenshot | File |
|---|---|
| Analyzer homepage | `screenshots/01_analyzer_home.png` |
| Hidden password field | `screenshots/02_hidden_password.png` |
| Very Weak / Weak / Moderate / Strong / Very Strong | `screenshots/03_very_weak.png` … `07_very_strong.png` |
| Findings and suggestions | `screenshots/08_findings_suggestions.png` |
| Password generator | `screenshots/09_generator.png` |
| Policy checker | `screenshots/10_policy_checker.png` |
| Analytics dashboard | `screenshots/11_dashboard.png` |
| Test results | `screenshots/12_tests.png` |

## Learning Outcomes

- Password-security fundamentals: strength, entropy, reuse, credential stuffing, MFA
- Why composition rules alone are insufficient
- Secure coding: no secret logging, input validation, rate limiting, CSPRNG usage
- Hashing vs. encryption; salting; slow password hashing (Argon2id, bcrypt, scrypt, PBKDF2)
- Building and testing a privacy-by-design security tool

## Security Disclaimer

This is an educational, defensive project. Strength results are guidance, not a guarantee. Use only synthetic or demo passwords for testing and screenshots. This tool does not crack passwords, attempt logins, or collect credentials. Real systems should also use MFA, rate limiting, secure password hashing, session protection, and monitoring.

## Author

**Debankita Panja**
GitHub: [@your-username](https://github.com/dp2005-lang) · LinkedIn: [your-profile](https://www.linkedin.com/in/debankita-8482a2403/)
