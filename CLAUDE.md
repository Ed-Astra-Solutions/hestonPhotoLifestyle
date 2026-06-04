# Lifestyle Site — Admin-Driven Content Map

This document maps every section of the Lifestyle site to the data an admin needs to
upload/edit from the Admin Panel ("Lifestyle Site Controls"). Use it as the contract
between the static markup, the backend content model, and the admin UI.

- **Frontend (static today):** [index.html](index.html) + [services/](services/) (`headshots`, `family`, `sport`, `schools`, `events`, `product`, `food`, `property`)
- **Backend model:** `backend/models/lifestyle.js` (`LifestyleContent` singleton, `LifestylePricing`, `LifestyleBooking`, `LifestyleEvent`)
- **Backend actions:** `backend/methods/lifestyleActions.js`
- **Routes:** `backend/routes/lifestyle.js`
- **Admin UI:** `frontend-admin/index.html` → `#lifestyleSection` (tabs: Services, Portfolio, Testimonials, Event Galleries, Bookings)
- **Public API base:** `https://hestonadminapi.edastra.in/api` (localhost: `http://localhost:3000/api`)

> **Status legend:** ✅ wired (model + admin UI exist) · ⚠️ model exists, no admin UI · ❌ not modelled yet (static-only, needs schema + UI).

---

## Public read endpoints (what the frontend should fetch)

| Endpoint | Returns | Drives |
|---|---|---|
| `GET /api/lifestyle/content` | `LifestyleContent` singleton (hero, about, services, portfolio, testimonials, contact) | Landing page [index.html](index.html) |
| `GET /api/lifestyle/pricing` | `LifestylePricing[]` (active, sorted by `serviceSlug, sortOrder`) | Service-page pricing/backdrop section |
| `POST /api/lifestyle/booking` | Creates `LifestyleBooking` | Contact form (already wired in [index.html](index.html#L1760)) |

> ⚠️ **Gap:** The landing page and service pages are currently **hard-coded**. To make them admin-driven, each section below must be rendered from `GET /api/lifestyle/content` / `GET /api/lifestyle/pricing` instead of static HTML.

---

# PART A — Landing page (`index.html`)

Backed by the **`LifestyleContent`** singleton (`singletonKey: 'lifestyle'`).
Edit sections via `PUT /api/admin/lifestyle/content/:section` (section = `hero` | `about` | `services` | `portfolio` | `testimonials` | `contact`).

## A1. Hero — `content.hero`  ⚠️
Markup: [index.html:1159](index.html#L1159)

| Field | Type | Markup target |
|---|---|---|
| `tag` | string | `.hero-tag` |
| `headline` | string (may contain `<em>`) | `.hero h1` |
| `tagline` | string | `.hero-tagline` |
| `primaryCtaLabel` / `primaryCtaHref` | string | first `.hero-cta-row .btn` |
| `secondaryCtaLabel` / `secondaryCtaHref` | string | second `.hero-cta-row .btn` |
| `backgroundImages[]` | `{ imageUrl, s3Key, caption, altText, sortOrder }` | `.hero-grid img` (12 tiles) |

## A2. About / Intro — `content.about`  ⚠️
Markup: [index.html:1287](index.html#L1287)

| Field | Type | Markup target |
|---|---|---|
| `sectionLabel` | string | `.intro-content .section-label` |
| `headline` | string (`<em>`) | `.intro-content .section-title` |
| `paragraphs[]` | string[] | `.intro-content p` (one `<p>` each) |
| `coverImageUrl` / `coverImageS3Key` | string | `.intro-image img` |
| `stats[]` | `{ number, label }` (3 items) | `.intro-stats .stat-item` + `.intro-image-badge` |

## A3. Services grid — `content.services`  ✅ (Admin: **Services** tab)
Markup: [index.html:1195](index.html#L1195). Section headings (`sectionLabel`, `sectionTitle`, `sectionSubtitle`) via `PUT .../content/services`.

Each card = one `services.entries[]` item (`lifestyleServiceSchema`):

| Field | Type | Markup target |
|---|---|---|
| `slug` | string (links to `services/<slug>.html`) | card `href` |
| `tag` | string (e.g. `01 · Corporate`) | `.service-card-tag` |
| `title` | string | `.service-card-content h3` |
| `description` | string | `.service-card-content p` |
| `coverImageUrl` / `coverImageS3Key` | string | `.service-card-img` |
| `galleryImages[]` | image[] | reused on the service detail page gallery |
| `sortOrder`, `isActive` | number / bool | ordering / visibility |

Endpoints: `POST/PUT/DELETE /api/admin/lifestyle/content/services/entry[/:id]`,
gallery: `POST .../services/:id/gallery`, `DELETE .../services/:id/gallery/:imageId`.

## A4. Approach / "How We Work" — 4 cards  ❌
Markup: [index.html:1323](index.html#L1323). **Not modelled.** Either keep static or add `content.approach` with `{ sectionLabel, sectionTitle, sectionSubtitle, cards:[{ icon, title, description }] }`.

## A5. Portfolio mosaic — `content.portfolio`  ✅ (Admin: **Portfolio** tab)
Markup: [index.html:1364](index.html#L1364). Each tile = `portfolio.entries[]` (`lifestylePortfolioSchema`):

| Field | Type | Markup target |
|---|---|---|
| `imageUrl` / `s3Key` | string | `.mosaic-item img` |
| `label` | string | `.mosaic-label` |
| `serviceSlug` | string | optional link target |
| `layoutClass` | `'' \| 'mosaic-tall' \| 'mosaic-wide'` | tile size |
| `sortOrder`, `isActive` | number / bool | ordering / visibility |

Endpoints: `POST/PUT/DELETE /api/admin/lifestyle/content/portfolio/entry[/:id]`.

## A6. Testimonials — `content.testimonials`  ✅ (Admin: **Testimonials** tab)
Markup: [index.html:1408](index.html#L1408). Each = `testimonials.entries[]` (`lifestyleTestimonialSchema`):

| Field | Type | Markup target |
|---|---|---|
| `quote` | string | `.testimonial-card .quote` |
| `name` | string | `.testimonial-author .name` |
| `role` | string | `.testimonial-author .role` |
| `rating` | number 1–5 | `.testimonial-stars` (count) |
| `sortOrder`, `isActive` | number / bool | ordering / visibility |

Endpoints: `POST/PUT/DELETE /api/admin/lifestyle/content/testimonials/entry[/:id]`.

## A7. Contact block — `content.contact`  ⚠️
Markup: [index.html:1496](index.html#L1496) (`.contact-details`). The form itself posts to `/api/lifestyle/booking`.

| Field | Type | Markup target |
|---|---|---|
| `phone` | string | `tel:` link |
| `email` | string | `mailto:` link |
| `addressLine` | string | Studio line |
| `hours` | string | Hours line |

## A8. Booking form → `LifestyleBooking`  ✅ (Admin: **Bookings** tab)
Submission already wired ([index.html:1737](index.html#L1737)). Stored fields: `name, company, email, phone, serviceSlug, serviceLabel, message, source`, plus admin-side `status` (`new → contacted → quoted → booked → completed → rejected`), `adminNotes`.
Endpoints: `GET/PUT/DELETE /api/admin/lifestyle/bookings[/:id]`.

---

# PART B — Service detail pages (`services/<slug>.html`)

All 8 pages share one template. **Only `headshots.html` currently has a pricing/backdrop
section**; the other 7 do not. None of the per-page body content is modelled yet.

Shared sections per page:

| # | Section | Markup class | Source today |
|---|---|---|---|
| B1 | Hero (bg image, breadcrumb, tag, title, lede) | `.service-hero` | static |
| B2 | Intro text + "What's included" bullets | `.service-intro` / `.service-bullets` | static |
| B3 | Pricing / "Choose Your Backdrop" | `.backdrop-section` | static (**headshots only**) |
| B4 | Gallery (4–6 images) | `.service-gallery` / `.gallery-item` | static |
| B5 | CTA banner | `.service-cta` | static |
| B6 | Related services (4 cards) | `.related-services` | static |

### Recommended model extension (❌ not present yet)
Extend `lifestyleServiceSchema` (the `services.entries[]` used by A3) so one record drives
both the landing card **and** the full detail page:

```js
// add to lifestyleServiceSchema in backend/models/lifestyle.js
heroLede:        { type: String, default: '' },   // B1 .service-hero-lede
heroImageUrl:    { type: String, default: '' },   // B1 .service-hero-bg
heroImageS3Key:  { type: String, default: '' },
introHeadline:   { type: String, default: '' },   // B2 h2 (supports <em>)
introParagraphs: { type: [String], default: [] }, // B2 .service-intro-text p
includedBullets: { type: [String], default: [] }, // B2 .service-bullets li
ctaHeadline:     { type: String, default: '' },   // B5
ctaText:         { type: String, default: '' },   // B5
seoTitle:        { type: String, default: '' },   // <title>
seoDescription:  { type: String, default: '' },   // meta description + schema
// galleryImages[] already exists → drives B4
```

- **B3 Pricing** is already modelled separately as **`LifestylePricing`** (per `serviceSlug`):
  fields `title, tier, description, price, currency, unit, features[], featured, sortOrder, isActive`.
  ⚠️ No admin UI yet — add a **Pricing** tab and render B3 from `GET /api/lifestyle/pricing` filtered by slug.
- **B6 Related services** can be derived automatically from other active `services.entries`
  (no separate upload needed).

---

# Implementation checklist (to make sections admin-driven)

1. **Backend:** add `content.approach` (A4) and the B1/B2/B5/SEO fields to `lifestyleServiceSchema`; expose pricing in admin (`getAllPricing`/`createPricing` already exist).
2. **Admin UI** (`frontend-admin/index.html`): add tabs/forms for **Hero, About, Contact, Approach** (singleton sections) and **Pricing**; extend the Service editor with the detail-page fields above.
3. **Frontend:** replace hard-coded markup in [index.html](index.html) and each `services/*.html` with a fetch from `GET /api/lifestyle/content` (+ `/pricing`), rendering each section from the fields mapped above. Keep the current static HTML as the fallback/skeleton.
4. **Images:** upload via the existing S3 flow — every image field is a `{ imageUrl, s3Key }` pair; public reads re-sign URLs (`refreshSignedUrl` / `refreshSubArrayUrls`).

---

## Quick reference — section → admin location

| Page section | Admin tab / endpoint | Status |
|---|---|---|
| Landing Hero | (needs) `PUT content/hero` | ⚠️ |
| Landing About | (needs) `PUT content/about` | ⚠️ |
| Landing Services grid | **Services** tab | ✅ |
| Landing Approach | (needs new model) | ❌ |
| Landing Portfolio | **Portfolio** tab | ✅ |
| Landing Testimonials | **Testimonials** tab | ✅ |
| Landing Contact details | (needs) `PUT content/contact` | ⚠️ |
| Booking submissions | **Bookings** tab | ✅ |
| Service page hero/intro/bullets/CTA | (needs service-schema fields) | ❌ |
| Service page pricing (B3) | `LifestylePricing` (needs UI) | ⚠️ |
| Service page gallery | Service editor `galleryImages[]` | ✅ (model) |
| Event galleries (passcode) | **Event Galleries** tab | ✅ |
