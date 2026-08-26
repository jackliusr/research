# Storylane.io — Complete Tech Stack Analysis

> **Analysis Date:** 2026-07-04
> **Company:** Storylane Inc. — Interactive Product Demo Platform
> **Founded:** 2021 | **YC Batch:** Summer 2021 (S21)
> **Founders:** Akash Bansal & Nalin Senthamil
> **HQ:** San Francisco, CA | **Team:** ~49–55 employees
> **Revenue:** ~$1.9M–$6.3M (estimated)

---

## 1. FRONTEND — Marketing Website (www.storylane.io)

| Technology | Confidence | Evidence |
|---|---|---|
| **Webflow** | ✅ Confirmed | HTTP headers: `x-wf-region: us-east-1`, `x-worker: non-feeds`, `x-lambda-id`, `surrogate-key: storylane.gush.site`. HTML attributes: `data-wf-domain`, `data-wf-page`, `data-wf-site`. Assets served from `cdn.prod.website-files.com`. CSS paths contain `webflow` namespace. |
| **Cloudflare** | ✅ Confirmed | HTTP header: `server: cloudflare`, `cf-ray` trace header, `cf-cache-status: DYNAMIC`. |
| **nginx** (redirect) | ✅ Confirmed | Root domain `storylane.io` responds with `server: nginx` and HTTP 301 redirect to `www.storylane.io`. |
| **Google Fonts** | ✅ Confirmed | Preconnect to `fonts.googleapis.com` and `fonts.gstatic.com`. Loads Montserrat, Familjen Grotesk, Poppins, Roboto via WebFont loader. |
| **Inter Display (custom)** | ✅ Confirmed | Custom `@font-face` declarations loaded from jsDelivr CDN for Inter Display Thin through ExtraBold (100–800 weights). |
| **jQuery 3.5.1** | ✅ Confirmed | Loaded from `d3e54v103j8qbb.cloudfront.net` as part of Webflow's JS runtime. |
| **Webflow JS Runtime** | ✅ Confirmed | Webpack-bundled Webflow JS (`webflow.achunk.*.js` pattern, `r.m` webpack module system). |
| **Tailwind CSS** | 💡 Likely (on app) | Reported by LeadIQ as part of Storylane's tech stack. The marketing site uses Webflow's styling rather than Tailwind; Tailwind is more likely used in the app. |
| **Ahrefs** | ✅ Confirmed | `ahrefs-site-verification` meta tag present in the HTML `<head>`. |

### Analytics & Tracking (Marketing Site)

| Technology | Confidence | Evidence |
|---|---|---|
| **Google Tag Manager** | ✅ Confirmed | GTM container ID `GTM-TWPX6L3` loaded via `googletagmanager.com`. |
| **Google Analytics (GA4)** | ✅ Confirmed | Measurement ID `G-LJ5MSY05XG`, loaded from `js.storylane.io/js/v2/storylane.js` (their own tracker wrapper). |
| **FinSweet Attributes** | ✅ Confirmed | `@finsweet/attributes@2` loaded from jsDelivr for mirror-click functionality. |
| **FinSweet Cookie Consent** | ✅ Confirmed | `@finsweet/cookie-consent@1` loaded from jsDelivr (`fs-cc-mode="informational"`). |
| **Digital Sparks Cookie Consent Boost** | ✅ Confirmed | Custom cookie consent boost script loaded from `cdn.jsdelivr.net/gh/digital-sparks/cookie-consent-boost`. |
| **Leadfeeder (Dealfront)** | ✅ Confirmed | Leadfeeder tracker `lAxoEaKQJz07OYGd` loaded from `sc.lfeeder.com/lftracker_v1_...js`. |
| **HubSpot Analytics** | ✅ Confirmed | HubSpot tracking script loaded from `js.hs-scripts.com/20380174.js`. |
| **Facebook Pixel** | 💡 Likely | Reported by LeadIQ in their tech stack profile; not detected directly in the page source. |
| **Reddit Ads** | 💡 Likely | Reported by LeadIQ. |
| **Canva** | 💡 Likely | Reported by LeadIQ — likely used for design assets, not a technical stack component. |

### Marketing Site Integrations (visible in footer)

| Integration | Notes |
|---|---|
| HubSpot | Hubspot portal ID 20380174 |
| Intercom | Present in TXT DNS SPF record (`include:spf.mail.intercom.io`) |
| SendGrid | Present in TXT DNS SPF record (`include:sendgrid.net`), transactional email |
| Marketo | SDK reads `_mkto_trk` cookie for lead identification |
| RevenueHero | Scheduling widget: `app.revenuehero.io/scheduler.min.js` |

---

## 2. FRONTEND — Main Application (app.storylane.io)

| Technology | Confidence | Evidence |
|---|---|---|
| **React** | ✅ Confirmed | JS bundle contains `react-dom`, `react-redux`, React element types (`react.element`, `react.fragment`, `StrictMode`, `Suspense`, `Profiler`), `createElement` calls, `<div id="root">` mount point. |
| **Redux Toolkit (RTK)** | ✅ Confirmed | Bundle contains `react-redux`, Redux Toolkit's `createSlice`, `fetchBaseQuery` (RTK Query), and `@reduxjs` references for API data fetching. |
| **Vite (build tool)** | ✅ Confirmed | Bundle contains `__vite__mapDeps` function, ES module `modulepreload` polyfill, `import.meta.url` pattern, Vite's modulepreload linking strategy. |
| **TypeScript** | 💡 High Likely | Standard for React+Vite projects; all bundle patterns are consistent with compiled TypeScript output. |
| **React Router** | 💡 Likely | SPA with multiple routes; bundle references indicate client-side routing. |
| **Pusher (real-time)** | ✅ Confirmed | Bundle imports `pusher-js` from GitHub for real-time WebSocket functionality — likely used for demo session synchronization and live collaboration. |
| **PostHog** | ✅ Confirmed | Bundle contains `_posthogChunkIds` and references `https://app.posthog.com` (PostHog Cloud). Used for product analytics. |
| **Segment** | ✅ Confirmed | Matched in bundle grep across analytics libraries. Likely used as a CDP for event routing. |
| **Sentry** | ✅ Confirmed | Matched in bundle grep. Used for error tracking and performance monitoring. |
| **Intercom** | ✅ Confirmed | Matched in bundle grep. Used for in-app messaging and customer communication. |
| **HubSpot** | ✅ Confirmed | Matched in bundle grep; also on the marketing site. Used for CRM tracking. |
| **Stripe** | ✅ Confirmed | Bundle references `billing.stripe.com` — used for subscription billing and payment processing. |
| **CSS-in-JS / UI Framework** | 💡 Suspected | Bundle is ~381KB CSS indicating a UI framework or design system; likely custom components with CSS Modules or Emotion/styled-components. |

### App Infrastructure

| Technology | Confidence | Evidence |
|---|---|---|
| **Amazon S3** (hosting) | ✅ Confirmed | Static assets served from `assets.storylane.io/apps/prod/417/` with `server: AmazonS3`, `x-amz-server-side-encryption: AES256`, S3 version IDs, S3 replication enabled (`x-amz-replication-status: COMPLETED`). |
| **Amazon S3 Region** | ✅ Confirmed | `x-amz-bucket-region: us-east-2` (Ohio). |
| **CloudFront (CDN)** | ✅ Confirmed | `x-cache: Hit from cloudfront`, `x-amz-cf-pop`, `x-amz-cf-id`, `via: CloudFront` headers on all asset requests. CloudFront POP: BKK51 (Bangkok). |
| **nginx** (reverse proxy) | ✅ Confirmed | `app.storylane.io` returns `server: nginx/1.28.0` — acts as reverse proxy / API gateway for the SPA and API. |
| **API Backend** | 💡 Likely (Ruby on Rails) | API at `api.storylane.io/api/v1/` returns `x-runtime: 0.009847` and `x-request-id` headers — standard Rails conventions. Also `x-xss-protection: 0` (Rails default with modern CSP). |
| **Staging/Dev** | 💡 Likely | `develop.storylane.io` redirects via nginx/1.28.0 — staging environment. |

---

## 3. BACKEND — API & Infrastructure

| Technology | Confidence | Evidence |
|---|---|---|
| **Ruby on Rails** (backend API) | 💡 High Likely | `x-runtime` header (standard Rails middleware timing header), `x-request-id`, `vary: Origin`, JSON API responses at `https://api.storylane.io/api/v1/`. The API responded with a 404 JSON body in 9.8ms. |
| **API Protocol** | ✅ Confirmed | REST API versioned at `/api/v1/`. RTK Query `fetchBaseQuery` confirms REST-based data fetching from the frontend. |
| **PostgreSQL** (database) | 💡 High Likely | Standard for Rails/Y Combinator startups. LeadIQ also mentions it indirectly. No direct headers confirm this, but very typical for the stack. |
| **Redis** | 💡 Likely | Typical Rails caching/sidekiq stack; Pusher may also use Redis for presence channels. |
| **Docker** | 💡 Likely | Reported by LeadIQ. Likely used for development environments and CI/CD. |
| **AWS** (primary cloud) | ✅ Confirmed | S3 in us-east-2, Route 53 for DNS, nginx/1.28.0 servers. Webflow infrastructure also runs on AWS (us-east-1). |
| **MX Technologies** | 💡 Likely | Listed by LeadIQ — could be a misattribution or a financial integration partner rather than infra. |

### DNS & Networking

| Technology | Confidence | Evidence |
|---|---|---|
| **AWS Route 53** | ✅ Confirmed | NS records: `ns-1081.awsdns-07.org`, `ns-1656.awsdns-15.co.uk`, `ns-234.awsdns-29.com`, `ns-896.awsdns-48.net`. |
| **A Record** | 18.218.215.163 | AWS us-east-2 IP (pre-Cloudflare direct). |
| **WWW CNAME** | `client.gushwork.net` | Webflow hosting alias (Gushwork = Webflow's infrastructure partner). |
| **Google Workspace** (email) | ✅ Confirmed | MX records pointing to `aspmx.l.google.com` and alt servers. SPF includes `_spf.google.com`. |
| **SendGrid** | ✅ Confirmed | In SPF record: `include:sendgrid.net`. Transactional email delivery. |
| **Intercom** | ✅ Confirmed | In SPF record: `include:spf.mail.intercom.io`. Email and in-app messaging. |
| **HubSpot** | ✅ Confirmed | TXT: `hubspot-developer-verification=...` and tracking script. |

---

## 4. DOCUMENTATION (docs.storylane.io)

| Technology | Confidence | Evidence |
|---|---|---|
| **GitBook** | ✅ Confirmed | HTTP headers: `x-gitbook-route-site: docs.storylane.io/`, `x-gitbook-route-type: static`, `x-gitbook-target: 2v`, `x-matched-path`. |
| **Next.js** (under GitBook) | ✅ Confirmed | `x-nextjs-prerender: 1` header — GitBook uses Next.js under the hood. |
| **Cloudflare** | ✅ Confirmed | `server: cloudflare` on docs subdomain. |
| **Content Security Policy** | ✅ Confirmed | CSP: `default-src 'self' *` allowing broad resource loading for embed content. |

---

## 5. STATUS PAGE (status.storylane.io)

| Technology | Confidence | Evidence |
|---|---|---|
| **Atlassian Statuspage** | ✅ Confirmed | `x-statuspage-version: 6e08ceff324f2b596e4c1a7fcfee88906b146fcb`, `server: AtlassianEdge`, `server-timing: atl-edge;desc="aws-us-east-1"`. |
| **AtlassianEdge CDN** | ✅ Confirmed | Servers indicate Atlassian's edge network in AWS us-east-1. |

---

## 6. JS EMBED SDK (js.storylane.io)

| Technology | Confidence | Evidence |
|---|---|---|
| **Amazon S3** | ✅ Confirmed | `server: AmazonS3` on `js.storylane.io`. |
| **CloudFront** | ✅ Confirmed | `x-cache: Hit from cloudfront`, `via: CloudFront`. |
| **Custom JS SDK (~17KB)** | ✅ Confirmed | Self-hosted embed SDK (`storylane.js` v2) for embedding interactive demos on third-party sites. Handles custom events: `storylane-demo-event`, `storylane-embed-demo`, `storylane-token-submit`, `storylane-tracking`, `storylane-host-info`, `storylane-demo-host`, `storylane-close-demo-player`. |
| **Lead Enrichment** | ✅ Confirmed | SDK reads HubSpot (`hubspotutk`), Marketo (`_mkto_trk`), and generic visitor ID cookies for lead identification. |

### Embed SDK Capabilities
- Demo player embedding with iframe-based architecture
- Lead tracking through CRM cookie reading
- Custom event dispatching for host page communication
- Close-modal UI with branded styling (`sl-close-btn` CSS)

---

## 7. CHROME EXTENSION

| Technology | Confidence | Evidence |
|---|---|---|
| **Chrome Extension** | ✅ Confirmed | Published on Chrome Web Store: `ongmhighpnfpojfidgigcjflgdkjfdeb` |
| **Purpose** | ✅ Confirmed | "Capture your product application pages easily, and build interactive product demo in Storylane" — screenshots and page capture for demo creation |
| **Support** | `support-extension@storylane.io` |
| **Likely Tech** | 💡 Likely | Built with React + TypeScript + WebExtensions API (standard for modern Chrome extensions) |

---

## 8. INTEGRATIONS & ECOSYSTEM

| Integration | Type | Details |
|---|---|---|
| **Salesforce** | CRM | Native integration for lead/pipeline sync |
| **HubSpot** | CRM | Native integration with lead tracking; HubSpot portal ID 20380174 |
| **Marketo** | Marketing Automation | Cookie-based lead identification (`_mkto_trk`), Zapier connector available |
| **Intercom** | Customer Comms | In-app messaging and email via SPF |
| **Segment** | CDP | Customer Data Pipeline for event routing |
| **PostHog** | Product Analytics | Product analytics and feature tracking |
| **Zapier** | Automation | 9,000+ app integrations available |
| **SendGrid** | Email | Transactional email delivery |
| **RevenueHero** | Scheduling | Meeting scheduling widget |
| **Stripe** | Billing | Subscription billing and payment processing |
| **Pusher** | Real-time | Demo session synchronization, live events |
| **Sentry** | Monitoring | Error tracking and performance monitoring |
| **Google Analytics (GA4)** | Analytics | Marketing site and demo engagement tracking |
| **Google Tag Manager** | Tag Management | Centralized tag deployment |

---

## 9. DEPLOYMENT & CI/CD (Inferred)

| Technology | Confidence | Evidence |
|---|---|---|
| **S3 + CloudFront static hosting** | ✅ Confirmed | App assets deployed to `assets.storylane.io/apps/prod/{build_number}/` with S3 versioning |
| **S3 Cross-Region Replication** | ✅ Confirmed | `x-amz-replication-status: COMPLETED` header on assets — DR/redundancy |
| **S3 Server-Side Encryption** | ✅ Confirmed | `x-amz-server-side-encoding: AES256` |
| **Immutable Cache Policy** | ✅ Confirmed | CSS assets: `cache-control: public,max-age=31536000,immutable` (1 year with content hashing) |
| **Nginx reverse proxy** | ✅ Confirmed | Version 1.28.0 on app and API subdomains |
| **Build Numbering** | ✅ Confirmed | App deployed at version/build path `/417/` — indicates CI/CD pipeline with sequential builds |

---

## 10. COMPLETE TECH STACK SUMMARY

### Confirmed (direct evidence from headers, source, or bundles)

| Layer | Technology |
|---|---|
| **Marketing Site CMS** | Webflow |
| **Marketing Site CDN** | Cloudflare |
| **App Frontend Framework** | React |
| **App State Management** | Redux Toolkit (createSlice, RTK Query) |
| **App Build Tool** | Vite |
| **App Real-time** | Pusher (pusher-js) |
| **App Static Hosting** | Amazon S3 (us-east-2) |
| **App CDN** | Amazon CloudFront |
| **App Reverse Proxy** | nginx 1.28.0 |
| **Error Monitoring** | Sentry |
| **Product Analytics** | PostHog |
| **Customer Data Pipeline** | Segment |
| **In-App Messaging** | Intercom |
| **CRM** | HubSpot |
| **Billing** | Stripe |
| **Scheduling** | RevenueHero |
| **Documentation** | GitBook (Next.js) |
| **Status Page** | Atlassian Statuspage |
| **DNS** | AWS Route 53 |
| **Email (Business)** | Google Workspace |
| **Email (Transactional)** | SendGrid |
| **Cookie Consent** | FinSweet + Digital Sparks |

### Highly Likely (strong circumstantial evidence)

| Layer | Technology |
|---|---|
| **Backend Framework** | Ruby on Rails (`x-runtime` header, RESTful API pattern) |
| **Database** | PostgreSQL (standard Rails/YC stack) |
| **Caching / Queue** | Redis / Sidekiq (standard Rails stack) |
| **CI/CD** | GitHub Actions or similar (standard for YC startups) |
| **Containerization** | Docker (LeadIQ report) |

### Suspected / Possible

| Layer | Technology |
|---|---|
| **CSS Framework (App UI)** | Custom design system with Tailwind-like utility patterns |
| **API Type** | REST (confirmed), with possible GraphQL augmentation |
| **Testing** | Jest + React Testing Library (standard React stack) |
| **Monitoring** | Datadog or similar APM (standard Rails production stack) |
| **AI Features** | RepX product (likely uses LLM APIs — OpenAI, Anthropic, etc.) |

---

## 11. ARCHITECTURE DIAGRAM (Textual)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           END USERS (Browser)                               │
└────────┬────────────────────────┬─────────────────────────────┬────────────┘
         │                        │                             │
         ▼                        ▼                             ▼
┌─────────────────┐   ┌─────────────────────┐   ┌─────────────────────────┐
│  www.storylane.io│   │  app.storylane.io   │   │  Embedded Demos          │
│  (Marketing)     │   │  (Product App)      │   │  (Customer Sites via     │
│                  │   │                     │   │   js.storylane.io SDK)   │
│  Webflow CMS     │   │  React SPA (Vite)   │   │                          │
│  Cloudflare CDN  │   │  Redux Toolkit      │   │  iframe + PostMessage    │
│  Google Analytics │   │  Pusher (real-time) │   │  Cookie-based lead track │
│  GTM, HubSpot    │   │  PostHog, Sentry,   │   │  (HubSpot/Marketo)      │
│  Leadfeeder      │   │  Intercom, Segment  │   │                          │
└────────┬─────────┘   └─────────┬───────────┘   └────────────┬────────────┘
         │                      │                            │
         │              ┌───────┴────────┐                   │
         │              │  nginx 1.28.0  │                   │
         │              │  (proxy)       │                   │
         │              └───────┬────────┘                   │
         │                      │                            │
         │              ┌───────┴────────────────────────┐   │
         │              │  api.storylane.io/api/v1/      │   │
         │              │  Ruby on Rails Backend         │   │
         │              │  (REST API)                    │   │
         │              └───────┬────────────────────────┘   │
         │                      │                            │
         │              ┌───────┴────────┐                   │
         │              │  PostgreSQL    │                   │
         │              │  + Redis       │                   │
         │              └────────────────┘                   │
         │                                                   │
         ▼                                                   ▼
┌──────────────────────┐                        ┌──────────────────────┐
│ Cloudflare           │                        │ Amazon CloudFront    │
│ (Marketing site CDN) │                        │ (App assets CDN)     │
│                      │                        │ S3 origin (us-east-2)│
└──────────────────────┘                        └──────────────────────┘

INFRASTRUCTURE:
┌──────────────────────────────────────────────────────────────────┐
│ AWS (Primary Cloud Provider)                                      │
│   ├── S3 (Static assets, us-east-2, with CRR + SSE-AES256)       │
│   ├── CloudFront (CDN for app assets)                             │
│   ├── Route 53 (DNS)                                              │
│   └── EC2/ECS (Likely Rails app servers)                          │
│                                                                    │
│ Third-Party Services:                                              │
│   Cloudflare (marketing site CDN & security)                       │
│   Atlassian Statuspage (status.storylane.io)                       │
│   GitBook / Next.js (docs.storylane.io)                            │
│   Pusher (real-time WebSocket infrastructure)                       │
│   Stripe (billing & subscriptions)                                 │
│   Sentry (error monitoring)                                        │
│   PostHog Cloud (product analytics)                                │
│   Segment (customer data pipeline)                                 │
└──────────────────────────────────────────────────────────────────┘
```

---

## 12. DATA SOURCES & METHODS

- **Direct HTTP requests** to all storylane.io subdomains (headers analysis)
- **DNS record inspection** (A, AAAA, MX, NS, TXT, CNAME records)
- **HTML source extraction** from marketing site, app login page, and integrations page
- **JavaScript bundle analysis** of the main app bundle (`index-HiaTl6Wk.js`, ~2.2MB) and Webflow runtime
- **JS SDK analysis** (`storylane.js` v2 embed script)
- **LeadIQ company profile** (crowd-sourced tech stack data)
- **BuiltWith.com** profile reference
- **Y Combinator company directory** (batch, founders, team size)
- **Crunchbase / Growjo / Tracxn** (funding, revenue, headcount)
- **Webflow blog post** regarding Storylane embed
- **Chrome Web Store** listing analysis

---

## 13. CONFIDENCE LEGEND

| Badge | Meaning |
|---|---|
| ✅ Confirmed | Direct, unambiguous evidence from headers, source code, or authoritative source |
| 💡 High Likely | Strong circumstantial evidence or multiple corroborating signals |
| 💡 Likely | Standard practice / typical stack for similar companies, with some corroboration |
| 💡 Suspected | Plausible based on stack patterns, no direct confirmation |

---

*Report generated via comprehensive multi-source recon. Stack may have evolved since analysis date. For the most current info, re-scan headers and bundles periodically.*
