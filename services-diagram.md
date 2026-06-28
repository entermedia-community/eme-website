# EME Platform — Service Architecture

> See `services-diagram.png` for the visual diagram.

---

## 1. Infrastructure Layer

| Component | Role |
|-----------|------|
| **Apache Tomcat** | Servlet container / HTTP server (ports 8080, 8005, 8009) |
| **Caucho Resin** | Alternative app server (v3.1) with cluster support |
| **OpenEdit Servlet Filter** | Core framework — intercepts all requests (`/*`) and routes them to the appropriate business module |

All requests flow through Tomcat/Resin → OpenEdit Filter → Business Module.

---

## 2. External (Third-Party) Services

| Service | Purpose |
|---------|---------|
| **AI/LLM Server** (`llamat.emediaworkspace.com`) | OpenAI-compatible API running Qwen3-14B. Powers content creation, classification, chat, semantic search, and 27 AI functions. |
| **Elasticsearch** | Search indexing engine. Cluster `entermedia-testcluster`, single node. Used by the DAM and all entity modules for full-text and faceted search. |
| **Stripe** | Payment processing (Java library `stripe-java-5.25.0.jar`). Handles donations, invoices, and product purchases. |
| **SMTP Mail Server** | Local server on `127.0.0.1:25` for email notifications (orders, chat notifications, alerts). |
| **Google Fonts** | Loads the Inter typeface for the website theme. |
| **emedialibrary.org** | External image/media hosting domain used for user avatars and asset URLs. |

---

## 3. Core Platform — Business Modules

These are the primary data entities managed by EME:

### Digital Asset Management
- **entityasset** — Main DAM entity: images, videos, documents
- **asset** — Raw file storage and metadata
- **entityassetpage** — Page renditions of assets
- **library / librarycollection** — Media libraries and team collections
- **category** — Folder/taxonomy hierarchy
- **searchcategory** — Catalog categories for browsing

### Users & Identity
- **emeprofile** — Top-level user identity/profile
- **user** — User accounts with authentication
- **userprofile** — User preferences and settings
- **socialmediaprofile** — Linked social media accounts
- **entitycompany** — Company/organization records
- **entityperson** — People/personnel records

### Content Management
- **entityarticle** — Articles with pages (`entityarticlepage`)
- **entitydocument** — Documents with pages (`entitydocumentpage`)
- **entitywebcontent** — Web content pages
- **userpost** — Blog posts

### Collaboration
- **userchat** — Chat message history
- **channel** — Communication channels
- **chatterboxattachment** — File attachments in chat
- **chatterboxreaction** — Emoji/message reactions
- **projectgoal** — Project goals and objectives
- **goaltask** — Tasks within goals

### Commerce & Finance
- **collectiveproduct** — E-commerce product catalog
- **order** — Customer orders
- **collectiveinvoice** — Billing/invoice records
- **transaction** — Donation/payment transactions
- **bankaccount** — Financial bank account records
- **collectiveexpense** — Expense tracking
- **collectiveincome** — Cash income records
- **collectiveinvestment** — Investment/share management
- **collectivereimbursement** — Expense reimbursement

---

## 4. AI & Automation Layer

| Automation | Trigger | Action |
|------------|---------|--------|
| **Content Creator** | Manual / API | AI-generated content (text, images) via LLM |
| **Smart Organizer** | Asset upload | Auto-classification and tagging via LLM |
| **Auto Articles** | Scheduled / Event | AI-written articles from source material |
| **Auto Tutorials** | Scheduled / Event | AI-generated tutorial content |
| **Hot Folder Scanning** | File system watch | Monitors folders for new assets and imports them |
| **Chat Detection** | Chat message | Auto-detects conversation topics and suggests actions |
| **Informatics** | Scheduled | Automated content classification |

All automation modules call the external AI/LLM Server for inference.

---

## 5. Sites (Multi-tenancy)

| Site ID | Name | Domain | Purpose |
|---------|------|--------|---------|
| `website` | EnterMedia - Finder DAM | `website.org` | Public-facing DAM portal |
| `catalog` | EME Admin | `localhost.com` | Administrative backend |

Each site shares the same core platform but has its own context root and theming.

---

## 6. Data Flow Summary

```
User Request
    ↓
Tomcat / Resin (HTTP server)
    ↓
OpenEdit Servlet Filter (routing, auth, context)
    ↓
Business Module (entityasset, order, userchat, etc.)
    ↓        ↓           ↓            ↓
Elasticsearch  Stripe    SMTP       AI/LLM Server
(search)     (payments) (email)     (AI features)
```

The platform follows a **modular monolith** architecture — all business modules run in the same application server process but are logically separated by entity type and routed through the OpenEdit core filter.
