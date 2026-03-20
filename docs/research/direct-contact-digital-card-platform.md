# Direct-Contact Digital Business Card Platform: A GOAP-Driven Feasibility Study

## Eliminating Intermediary Networks in NFC/QR Contact Sharing

---

**Research Classification:** Applied Technology & Product Architecture
**Methodology:** Goal-Oriented Action Planning (GOAP) with A* State-Space Analysis
**Date:** March 2026
**Status:** Active Research

---

## Abstract

This document presents a comprehensive feasibility analysis for building a direct-to-contact digital business card platform that eliminates intermediary network dependencies (e.g., Popl Network). Using Goal-Oriented Action Planning (GOAP) methodology, we evaluate the current state of contact-sharing technologies, identify the optimal architectural path from profile creation to native iPhone/Android contact storage, and propose a privacy-first, network-free implementation. The platform targets a digital business card market valued at $229M (2025) growing at 9.78-12.2% CAGR, with a differentiated position centered on data sovereignty and zero vendor lock-in.

---

## Table of Contents

1. [GOAP State Assessment](#1-goap-state-assessment)
2. [Problem Definition: The Intermediary Network Problem](#2-problem-definition)
3. [Technical Landscape Analysis](#3-technical-landscape-analysis)
4. [Competitive Intelligence](#4-competitive-intelligence)
5. [Architecture Design](#5-architecture-design)
6. [iPhone Contact Integration: Technical Deep Dive](#6-iphone-contact-integration)
7. [NFC Hardware & Protocol Specifications](#7-nfc-hardware--protocol-specifications)
8. [Privacy & Compliance Framework](#8-privacy--compliance-framework)
9. [Market Analysis & Positioning](#9-market-analysis--positioning)
10. [GOAP Action Plan & Implementation Roadmap](#10-goap-action-plan--implementation-roadmap)
11. [Risk Assessment](#11-risk-assessment)
12. [References](#12-references)

---

## 1. GOAP State Assessment

### 1.1 Current State (World State)

| State Variable | Current Value |
|---|---|
| `has_digital_card_platform` | false |
| `contacts_go_directly_to_phone` | false |
| `intermediary_network_required` | true (Popl model) |
| `user_owns_contact_data` | false (platform-dependent) |
| `supports_iphone_native_contacts` | false |
| `nfc_hardware_available` | true (NTAG213/215/216) |
| `vcard_standard_mature` | true (RFC 6350) |
| `qr_native_ios_support` | true (iOS 11+) |
| `web_nfc_ios_support` | false (Apple declined) |
| `pwa_contact_api_ios` | false (experimental only) |

### 1.2 Goal State

| State Variable | Target Value |
|---|---|
| `has_digital_card_platform` | true |
| `contacts_go_directly_to_phone` | true |
| `intermediary_network_required` | false |
| `user_owns_contact_data` | true |
| `supports_iphone_native_contacts` | true |
| `supports_android_native_contacts` | true |
| `supports_nfc_tap` | true |
| `supports_qr_scan` | true |
| `supports_wallet_pass` | true |
| `gdpr_ccpa_compliant` | true |

### 1.3 State Gap Analysis

The critical gap is the **intermediary network dependency**. Current platforms (Popl, Linq, HiHello) route contact exchanges through proprietary databases, creating vendor lock-in and privacy concerns. The goal is to achieve direct device-to-contact transfer using open standards (vCard RFC 6350, NFC Forum NDEF, QR ISO/IEC 18004) while maintaining feature parity with commercial platforms.

---

## 2. Problem Definition

### 2.1 The Intermediary Network Problem

When a user taps a Popl NFC card or scans a Popl QR code, the following occurs:

```
[NFC Tap / QR Scan] → [Popl Server] → [Popl Profile Page] → [User clicks "Save"] → [Popl Network connection created] → [Optional: vCard download]
```

**Problems with this model:**

1. **Data Sovereignty:** Contact exchange data is stored on Popl's servers, not the user's device
2. **Vendor Lock-in:** Contacts exist within the Popl ecosystem; migration requires manual export
3. **Privacy Exposure:** Every contact interaction is logged, tracked, and analyzed by the platform
4. **Network Dependency:** Profile viewing requires Popl's servers to be operational
5. **Friction:** Multiple steps between scan and contact save; intermediary profile page adds latency

### 2.2 Target Architecture

```
[NFC Tap / QR Scan] → [Hosted Profile Page (self-owned)] → [One-tap vCard Download] → [Native Contacts App]
```

**Or even more direct:**

```
[NFC Tap / QR Scan] → [Direct vCard Download] → [Native Contacts App]
```

### 2.3 Core Requirements

| Requirement | Priority | Rationale |
|---|---|---|
| Direct-to-contacts vCard delivery | P0 | Core differentiator; eliminates network dependency |
| iPhone native Contacts integration | P0 | Primary target platform |
| QR code generation & scanning | P0 | Universal compatibility (100% device support) |
| NFC tag programming (NTAG215) | P0 | Premium tap-to-share experience |
| Rich profile page (self-hosted) | P1 | Feature parity with Popl |
| Apple Wallet / Google Wallet pass | P1 | Native wallet integration |
| Analytics (privacy-preserving) | P2 | Business intelligence without tracking individuals |
| CRM webhook integration | P2 | Enterprise lead capture |
| Team management dashboard | P3 | Enterprise features |

---

## 3. Technical Landscape Analysis

### 3.1 iOS Contact Creation from Web: Available Methods

#### 3.1.1 vCard (.vcf) File Delivery (RECOMMENDED - Production Ready)

The vCard standard (RFC 6350) is the **only production-ready method** for web-to-iPhone contact creation.

**How it works:**
- Web server generates a `.vcf` file with MIME type `text/vcard; charset=utf-8`
- `Content-Disposition: attachment; filename="contact.vcf"` header triggers download
- iOS Safari recognizes the file type and prompts "Add to Contacts"
- User confirms; contact is written directly to native Contacts app

**Technical implementation:**
```
Content-Type: text/vcard; charset=utf-8
Content-Disposition: attachment; filename="Stuart_Kerr.vcf"

BEGIN:VCARD
VERSION:3.0
FN:Stuart Kerr
ORG:ISO Vision AI
TITLE:Founder & Owner
TEL;TYPE=CELL:+13129539668
EMAIL:sikerr@gmail.com
URL:https://isovision.ai
ADR:;;3201 S Ocean Blvd #401;Highland Beach;FL;33487;USA
END:VCARD
```

**Advantages:** Universal iOS support (all versions), no app required, standards-based, offline capable
**Limitations:** Requires user confirmation tap; cannot auto-add contacts

**Libraries:** vCards-js (npm), ez-vcard (Java), vobject (Python)

#### 3.1.2 Apple Wallet Pass (PKPass) - Supplementary

- ZIP archive containing JSON metadata + images
- Can store contact information as a "generic" pass type
- Double-tap Side Button for instant access
- **Not a replacement for vCard** — Wallet passes don't create Contacts entries
- Best used as a supplementary always-accessible card

#### 3.1.3 Web NFC API - NOT AVAILABLE on iOS

- Apple explicitly declined implementation (June 2020)
- Cited fingerprinting and privacy concerns
- Only available in Chrome on Android
- **Implication:** NFC on iOS requires physical NTAG hardware, not Web NFC API

#### 3.1.4 Contact Picker API - NOT PRODUCTION READY on iOS

- Experimental in Safari iOS 17.4+ (requires manual feature flag)
- Read-only (can pick existing contacts, cannot create new ones)
- Not enabled by default on any iOS version
- **Verdict:** Not viable for contact creation

#### 3.1.5 URL Schemes - NO CONTACT CREATION SUPPORT

- `tel://`, `mailto://`, `sms://` exist but no `contacts://` scheme for creating contacts
- No undocumented URL scheme exists for this purpose
- **Verdict:** Not viable

### 3.2 Summary: iOS Contact Creation Viability Matrix

| Method | iOS Support | Creates Contact | Production Ready | App Required |
|---|---|---|---|---|
| vCard (.vcf) download | All versions | Yes | Yes | No |
| Apple Wallet (PKPass) | iOS 6+ | No (wallet only) | Yes | No |
| Web NFC API | Not supported | N/A | No | N/A |
| Contact Picker API | Experimental | No (read-only) | No | No |
| URL Scheme | No scheme exists | No | No | N/A |
| NameDrop (P2P) | iOS 17+ | Yes | Yes | No (native) |

### 3.3 QR Code: Native iOS Camera Support

- Built-in scanning since iOS 11 (2017)
- Works on iPhone 6s and all newer models
- Over 1 billion compatible devices worldwide
- iOS 17+ improved UX with fixed-position link button
- QR can encode: direct vCard data (up to ~3KB), or URL pointing to vCard endpoint

### 3.4 Technology Comparison: QR vs NFC

| Dimension | QR Code | NFC |
|---|---|---|
| Device compatibility | Universal (all smartphones) | Flagship devices only |
| Transaction speed | 5.94ms | 1,074ms |
| User action | Open camera, point, tap | Tap device to card |
| Cost per unit | Near zero (printable) | $0.15-$2.00 per tag |
| Offline capability | Yes (if vCard embedded) | Yes (data on tag) |
| Analytics capability | Built-in via URL tracking | Requires separate infra |
| Maximum data | ~3KB (vCard fits) | 504 bytes (NTAG215) |
| Security | URL preview before visit | No preview; proximity-based |

---

## 4. Competitive Intelligence

### 4.1 Market Leaders

| Platform | Model | NFC | QR | Wallet | CRM | Direct Contact | Pricing |
|---|---|---|---|---|---|---|---|
| **Popl** | Hardware + SaaS | Yes | Yes | Yes | Yes | Via network | $7.99/mo |
| **HiHello** | Software-first | Optional | Yes | Yes | Yes | vCard download | Free-$6/mo |
| **Blinq** | Freemium | Yes | Yes | Yes | Limited | vCard download | Free-$5.89/mo |
| **Mobilo** | Hardware-first | Required | Yes | No | Yes | Via platform | $4/mo + card |
| **Linq** | Enterprise NFC | Yes | Yes | No | Yes | Via platform | Enterprise |
| **V1CE** | Premium hardware | Yes | Yes | Yes | Yes | vCard download | Premium |
| **Dot** | Essentials-only | Yes | No | No | No | Direct tap | One-time |

### 4.2 Open-Source Alternatives

| Project | Stack | Features | Maturity |
|---|---|---|---|
| **Swiish** | Self-hostable | Theming, PWA, QR, privacy controls | Active |
| **EnBizCard** | HTML/CSS | Responsive cards, embeddable | Stable |
| **Cardie** | Web | Design, QR, print, wallet | Active |
| **DigiCard** | Flutter | Cross-platform mobile | Early |

### 4.3 Competitive Differentiation Opportunity

**No major platform offers a true "zero-network" direct-to-contact experience.** Even platforms that offer vCard downloads (HiHello, Blinq) still route through their servers and track interactions. The opportunity is:

1. **Direct vCard delivery** — Contact goes straight to phone, no intermediary profile required
2. **Self-hosted profiles** — User controls their data endpoint
3. **Privacy-first analytics** — Aggregate counts without individual tracking
4. **Open standards only** — vCard (RFC 6350), NFC Forum NDEF, QR (ISO 18004)
5. **No account required for recipients** — Zero friction contact exchange

---

## 5. Architecture Design

### 5.1 Proposed System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    SHARING LAYER                             │
│                                                              │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│   │ NFC Tag  │  │ QR Code  │  │  Wallet   │  │ URL Link  │  │
│   │ NTAG215  │  │ ISO18004 │  │  PKPass   │  │ Direct    │  │
│   └────┬─────┘  └────┬─────┘  └────┬──────┘  └─────┬─────┘  │
│        │              │             │               │        │
│        └──────────────┴─────────────┴───────────────┘        │
│                           │                                  │
└───────────────────────────┼──────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    DELIVERY LAYER                            │
│                                                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              Profile Page (PWA)                      │   │
│   │  ┌─────────┐  ┌──────────┐  ┌───────────────────┐  │   │
│   │  │ Profile  │  │ Links &  │  │  "Save Contact"   │  │   │
│   │  │ Display  │  │ Socials  │  │  Button (vCard)    │  │   │
│   │  └─────────┘  └──────────┘  └───────────────────┘  │   │
│   └─────────────────────────────────────────────────────┘   │
│                           │                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │         Direct vCard Endpoint                        │   │
│   │         /api/contact/{id}/vcard                      │   │
│   │         Content-Type: text/vcard                     │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                              │
└───────────────────────────┬──────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    STORAGE LAYER                             │
│                                                              │
│   ┌───────────────┐  ┌────────────┐  ┌──────────────────┐  │
│   │  Edge KV /    │  │  CDN for   │  │  Serverless      │  │
│   │  Database     │  │  Static    │  │  Functions        │  │
│   │  (profiles)   │  │  Assets    │  │  (vCard gen)      │  │
│   └───────────────┘  └────────────┘  └──────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────┐
│                 RECIPIENT'S DEVICE                            │
│                                                               │
│   ┌──────────────────────────────────────────────────────┐   │
│   │              Native Contacts App                      │   │
│   │   ┌──────┐ ┌───────┐ ┌────────┐ ┌───────────────┐   │   │
│   │   │ Name │ │ Phone │ │ Email  │ │ Social Links  │   │   │
│   │   └──────┘ └───────┘ └────────┘ └───────────────┘   │   │
│   └──────────────────────────────────────────────────────┘   │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 5.2 Technology Stack

| Layer | Technology | Rationale |
|---|---|---|
| **Frontend** | Next.js / Nuxt 3 + Tailwind CSS | SSR for SEO, PWA capable, fast rendering |
| **Backend** | Serverless (Vercel/Netlify Functions) | Zero-ops, auto-scaling, global edge |
| **Database** | Edge KV (Cloudflare KV / Vercel KV) | Sub-10ms reads globally |
| **vCard Engine** | Custom (RFC 6350 compliant) | Full control over output format |
| **QR Generation** | qrcode.js / node-qrcode | Client-side generation, no server dependency |
| **NFC Programming** | NFC Tools / Web NFC (Android) | NTAG215 programming for physical cards |
| **Wallet Pass** | passkit-generator (Node.js) | Apple Wallet + Google Wallet passes |
| **CDN** | Cloudflare / Vercel Edge | Global <50ms profile delivery |
| **Analytics** | Privacy-preserving counters | Aggregate only, no individual tracking |

### 5.3 Data Flow: NFC Tap to Contact Save

```
1. User taps NFC card to recipient's iPhone
2. iPhone reads NDEF URL record: https://card.example.com/s/stuart-kerr
3. Safari opens profile page (PWA, server-rendered)
4. Profile displays: photo, name, title, company, bio, social links
5. Recipient taps "Save Contact" button
6. JavaScript generates vCard blob with ALL contact data + social URLs
7. Browser triggers download of .vcf file
8. iOS prompts: "Add to Contacts?" with preview
9. Recipient taps "Create New Contact"
10. Contact saved to native Contacts app with all fields populated
```

**Total steps from tap to saved contact: 3 user actions** (tap card, tap Save, tap Create)
**Popl equivalent: 4-5 actions** (tap card, wait for load, tap Save, handle Popl network prompt, confirm)

---

## 6. iPhone Contact Integration: Technical Deep Dive

### 6.1 vCard 3.0 Specification for Maximum iOS Compatibility

vCard 3.0 provides the broadest iOS compatibility (all versions support it). Key fields for a rich contact:

```vcard
BEGIN:VCARD
VERSION:3.0
N:Kerr;Stuart;;;
FN:Stuart Kerr
ORG:ISO Vision AI
TITLE:Founder & Owner
TEL;TYPE=CELL:+13129539668
EMAIL;TYPE=INTERNET:sikerr@gmail.com
ADR;TYPE=WORK:;;3201 S Ocean Blvd #401;Highland Beach;FL;33487;USA
URL;TYPE=WORK:https://isovision.ai
URL;TYPE=LinkedIn:https://linkedin.com/in/stuartkerr
URL;TYPE=Twitter:https://twitter.com/sikerr
NOTE:Stuart Kerr, with 30+ years in tech, had exec roles at 8 early-stage
 companies, driving 2 to $B+ IPO's and 5 to strategic sales.
X-SOCIALPROFILE;TYPE=linkedin:https://linkedin.com/in/stuartkerr
X-SOCIALPROFILE;TYPE=twitter:https://twitter.com/sikerr
END:VCARD
```

### 6.2 iOS-Specific vCard Extensions

Apple supports proprietary `X-` extensions that enhance contact display:

| Extension | Purpose | Example |
|---|---|---|
| `X-SOCIALPROFILE` | Social media links (renders with icons) | `X-SOCIALPROFILE;TYPE=linkedin:https://...` |
| `X-ABLabel` | Custom field labels | `X-ABLabel:Calendly` |
| `X-ABRELATEDNAMES` | Related contacts | `X-ABRELATEDNAMES;TYPE=pref:Assistant Name` |
| `PHOTO;ENCODING=b;TYPE=JPEG` | Embedded contact photo | Base64-encoded image |

### 6.3 Social Links in vCard: iOS Rendering

iOS Contacts natively renders social profiles with platform icons when using `X-SOCIALPROFILE`:

```vcard
X-SOCIALPROFILE;TYPE=linkedin:https://linkedin.com/in/stuartkerr
X-SOCIALPROFILE;TYPE=twitter:https://twitter.com/sikerr
```

For custom links (Calendly, Udemy, WhatsApp), use labeled URLs:

```vcard
item1.URL:https://tidycal.com/stuartkerr/
item1.X-ABLabel:Schedule Meeting
item2.URL:https://wa.me/13129539668
item2.X-ABLabel:WhatsApp
item3.URL:https://www.udemy.com/course/microsoft-excel-roi-selling-for-sales-professionals/
item3.X-ABLabel:Udemy Course
```

### 6.4 Client-Side vCard Generation (JavaScript)

```javascript
function generateVCard(profile) {
  const lines = [
    'BEGIN:VCARD',
    'VERSION:3.0',
    `N:${profile.lastName};${profile.firstName};;;`,
    `FN:${profile.firstName} ${profile.lastName}`,
    `ORG:${profile.company}`,
    `TITLE:${profile.title}`,
    `TEL;TYPE=CELL:${profile.phone}`,
    `EMAIL;TYPE=INTERNET:${profile.email}`,
    `ADR;TYPE=WORK:;;${profile.address}`,
    `URL;TYPE=WORK:${profile.website}`,
    `NOTE:${profile.bio}`,
    // Social profiles (iOS renders with icons)
    ...profile.socials.map(s =>
      `X-SOCIALPROFILE;TYPE=${s.type}:${s.url}`
    ),
    // Custom labeled links
    ...profile.customLinks.map((link, i) =>
      `item${i+1}.URL:${link.url}\r\nitem${i+1}.X-ABLabel:${link.label}`
    ),
    'END:VCARD'
  ];

  const blob = new Blob([lines.join('\r\n')], { type: 'text/vcard' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `${profile.firstName}_${profile.lastName}.vcf`;
  a.click();
  URL.revokeObjectURL(url);
}
```

---

## 7. NFC Hardware & Protocol Specifications

### 7.1 Recommended NFC Tags

| Specification | NTAG213 | NTAG215 (Recommended) | NTAG216 |
|---|---|---|---|
| Total memory | 144 bytes | 504 bytes | 888 bytes |
| User data | ~64 bytes | ~464 bytes | ~848 bytes |
| Pages | 45 | 135 | 231 |
| NFC Forum | Type 2 | Type 2 | Type 2 |
| ISO compliance | ISO/IEC 14443A | ISO/IEC 14443A | ISO/IEC 14443A |
| NDEF support | Full | Full | Full |
| Cost (bulk) | $0.08-0.15 | $0.15-0.30 | $0.25-0.50 |

**NTAG215 is recommended** because:
- 504 bytes accommodates a URL NDEF record (our approach)
- Compatible with all NFC-enabled iPhones (XR and newer)
- Cost-effective for business card form factors
- Widely available in card, sticker, and keyfob formats

### 7.2 NDEF Record Programming

For our direct-contact architecture, the NFC tag stores a **single NDEF URL record**:

```
NDEF Record:
  TNF: 0x01 (Well-Known)
  Type: "U" (URI)
  Payload: 0x04 + "card.example.com/s/stuart-kerr"
  (0x04 = "https://" prefix abbreviation)
```

**Why URL, not vCard on tag:**
1. iPhone does not natively parse vCard NDEF records — only URL records
2. URL allows dynamic profile updates without reprogramming the tag
3. URL approach enables analytics (tap counting)
4. 504 bytes is tight for a full vCard with photo; URL is ~50 bytes

### 7.3 NFC + iPhone Interaction Flow

```
iPhone (XR+) brought within 4cm of NTAG215
  → iPhone NFC reader activates
  → Reads NDEF URL record
  → Displays notification banner: "card.example.com"
  → User taps banner
  → Safari opens profile URL
  → Profile page loads (server-rendered, <200ms via edge CDN)
  → User taps "Save Contact"
  → vCard download triggers
  → iOS Contacts import dialog appears
```

---

## 8. Privacy & Compliance Framework

### 8.1 GDPR Compliance (EU)

| Requirement | Platform Approach |
|---|---|
| Lawful basis | Legitimate interest (contact sharing is the explicit purpose) |
| Data minimization | Only contact fields the user chooses to share |
| Right to erasure | User can delete profile; no recipient data stored |
| Data portability | vCard IS the portable format (RFC 6350) |
| Privacy by design | No intermediary network; no recipient tracking |
| Breach notification | Minimal attack surface (no recipient PII stored) |

### 8.2 CCPA Compliance (California)

| Requirement | Platform Approach |
|---|---|
| Transparency | Clear privacy policy; no hidden data collection |
| Opt-out rights | No data collected from recipients to opt out of |
| Data deletion | Profile owner controls all data |
| Non-discrimination | No tiered service based on privacy choices |

### 8.3 Privacy Architecture Advantages

**Compared to Popl/Linq/HiHello:**

| Dimension | Traditional Platforms | Direct-Contact Platform |
|---|---|---|
| Recipient data stored | Yes (connection logs) | No |
| Interaction tracking | Individual-level | Aggregate counters only |
| Third-party data sharing | Often (analytics, ads) | Never |
| Account required (recipient) | Sometimes | Never |
| Data location | Platform servers | User-controlled hosting |
| Vendor lock-in | High | None (open standards) |

---

## 9. Market Analysis & Positioning

### 9.1 Market Size & Growth

| Metric | Value | Source |
|---|---|---|
| 2024 market size | $204.54M | Market.us |
| 2025 market size | $229-238M | Multiple analysts |
| CAGR (2024-2030) | 9.78-12.2% | ResearchNester, Mordor Intelligence |
| 2030 projected size | $317.75M | ResearchNester |
| North America share | 46.2% | Market.us |
| Active Popl users | 2.5M+ professionals | Popl.co |
| Fortune 500 adoption | 90% | Popl.co (claimed) |

### 9.2 Target Segments

| Segment | Pain Point | Value Proposition |
|---|---|---|
| **Privacy-conscious professionals** | Don't want contact data on third-party servers | Zero-network, self-hosted profiles |
| **Solo entrepreneurs** | Don't need/want CRM overhead | Simple tap-to-save, no platform |
| **Enterprise security teams** | Compliance concerns with SaaS contact platforms | Self-hosted, auditable, GDPR-native |
| **Tech-forward networkers** | Frustrated by multi-step Popl flow | 3-action tap-to-contact experience |
| **International professionals** | Platform availability varies by region | Open-standard, globally deployable |

### 9.3 Pricing Strategy

| Tier | Price | Features |
|---|---|---|
| **Free** | $0 | 1 profile, QR code, vCard download, basic page |
| **Pro** | $4.99/mo | Custom domain, analytics, multiple profiles, Wallet pass |
| **Team** | $3.99/user/mo | Team management, branded templates, CRM webhooks |
| **Self-hosted** | One-time $49 | Full source code, deploy anywhere, unlimited profiles |

The **self-hosted tier** is the key differentiator — no other platform offers this.

---

## 10. GOAP Action Plan & Implementation Roadmap

### 10.1 GOAP Action Inventory

| Action | Preconditions | Effects | Cost |
|---|---|---|---|
| `build_vcard_engine` | RFC 6350 knowledge | `vcard_generation = true` | 2 days |
| `build_profile_page` | Design system | `profile_display = true` | 3 days |
| `implement_qr_generation` | Profile URL exists | `qr_sharing = true` | 1 day |
| `implement_nfc_programming` | NTAG215 hardware | `nfc_sharing = true` | 2 days |
| `build_wallet_pass` | Apple Developer account | `wallet_pass = true` | 2 days |
| `deploy_edge_infrastructure` | Cloud account | `global_delivery = true` | 1 day |
| `implement_analytics` | Profile page live | `analytics = true` | 1 day |
| `build_admin_dashboard` | Database schema | `profile_management = true` | 3 days |
| `implement_crm_webhooks` | API design | `crm_integration = true` | 2 days |
| `gdpr_compliance_audit` | All features built | `compliant = true` | 1 day |

### 10.2 Optimal Plan (A* Path)

**Phase 1: Core (Week 1-2)** — Minimum viable direct-contact platform

1. `build_vcard_engine` — RFC 6350 compliant generator with iOS extensions
2. `build_profile_page` — Server-rendered profile with "Save Contact" button
3. `implement_qr_generation` — QR codes on profile page and for printing
4. `deploy_edge_infrastructure` — Global edge deployment

**Phase 2: Hardware (Week 3)** — NFC and Wallet integration

5. `implement_nfc_programming` — NTAG215 URL record writing tool
6. `build_wallet_pass` — Apple Wallet + Google Wallet passes

**Phase 3: Business (Week 4)** — Analytics and management

7. `implement_analytics` — Privacy-preserving tap/scan counters
8. `build_admin_dashboard` — Profile CRUD, team management
9. `implement_crm_webhooks` — Salesforce, HubSpot, Zapier integration

**Phase 4: Compliance (Week 5)** — Audit and launch

10. `gdpr_compliance_audit` — Full compliance review and documentation

### 10.3 OODA Monitoring Loop

```
OBSERVE: Monitor deployment metrics, user feedback, conversion rates
ORIENT:  Compare against Popl feature matrix; identify gaps
DECIDE:  Prioritize features by user demand and competitive pressure
ACT:     Implement next highest-value feature; return to OBSERVE
```

---

## 11. Risk Assessment

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| Apple changes vCard handling in future iOS | Low | High | Monitor WebKit changelogs; maintain fallback URL approach |
| NFC tag costs increase | Low | Medium | QR-first strategy; NFC is supplementary |
| Competitor copies zero-network approach | Medium | Medium | First-mover advantage; self-hosted tier creates moat |
| Enterprise buyers require SOC 2 | Medium | High | Plan SOC 2 Type II certification by Year 2 |
| vCard size limits for rich profiles | Low | Low | Client-side generation avoids server limits; photo compression |
| Web NFC eventually comes to iOS | Low | Positive | Would eliminate need for physical NFC tags entirely |

---

## 12. References

### Standards & Specifications
1. RFC 6350 — vCard Format Specification. IETF. https://datatracker.ietf.org/doc/html/rfc6350
2. NFC Forum Type 2 Tag Operation Specification. NFC Forum. https://nfc-forum.org/build/specifications
3. ISO/IEC 18004:2015 — QR Code. International Organization for Standardization.
4. ISO/IEC 14443 — Identification cards, Contactless integrated circuit cards. ISO.
5. NXP NTAG213/215/216 Datasheet. NXP Semiconductors. https://www.nxp.com/docs/en/data-sheet/NTAG213_215_216.pdf

### Market Research
6. Digital Business Card Market Size & Growth. Market.us. https://market.us/report/digital-business-card-market/
7. Digital Business Card Market Analysis. Mordor Intelligence. https://www.mordorintelligence.com/industry-reports/digital-business-card-market
8. Digital Business Card Market Growth 2035. ResearchNester. https://www.researchnester.com/reports/digital-business-card-market/6782

### Technical References
9. Web NFC API — MDN Web Docs. https://developer.mozilla.org/en-US/docs/Web/API/Web_NFC_API
10. Contact Picker API — MDN Web Docs. https://developer.mozilla.org/en-US/docs/Web/API/Contact_Picker_API
11. Apple Wallet Developer Guide. Apple. https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/PassKit_PG/
12. PWA iOS Limitations and Safari Support 2026. MagicBell. https://www.magicbell.com/blog/pwa-ios-limitations-safari-support-complete-guide
13. vCards-js Library. GitHub. https://github.com/enesser/vCards-js
14. MIME Types for vCard. Univik. https://univik.com/blog/mime-types-vcard/

### Competitive Analysis
15. Popl Digital Business Card Platform. https://popl.co/
16. HiHello Digital Business Cards. https://www.hihello.com/
17. Blinq Digital Business Cards. https://blinq.me/
18. V1CE Smart Business Cards. https://v1ce.co/
19. Mobilo Smart Business Cards. https://www.mobilocard.com/

### Academic
20. NFiD: An NFC based system for Digital Business Cards. IRJET. https://www.irjet.net/archives/V4/i10/IRJET-V4I10277.pdf

### Patent Landscape
21. US20160059532A1 — NFC Card Manufacturing. Google Patents.
22. U.S. Patent No. 10,333,356 — Electronic Business Card Information Exchange Method. Justia Patents.

### Privacy & Compliance
23. GDPR — General Data Protection Regulation. https://gdpr.eu/
24. CCPA — California Consumer Privacy Act. https://oag.ca.gov/privacy/ccpa
25. BigID — CCPA vs GDPR Compliance. https://bigid.com/blog/ccpa-vs-gdpr-compliance/

---

*This document was generated using GOAP (Goal-Oriented Action Planning) methodology with A* state-space search, applied to the domain of digital contact sharing platform architecture. The analysis eliminates the intermediary network dependency present in current market solutions (Popl, Linq, HiHello) and proposes a direct-to-contact architecture built on open standards (RFC 6350, NFC Forum NDEF, ISO 18004).*
