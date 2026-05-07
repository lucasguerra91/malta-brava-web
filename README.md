# Malta Brava — Public Website

[![Astro](https://img.shields.io/badge/Astro-5.x-FF5D01?style=flat-square&logo=astro&logoColor=white)](https://astro.build)
[![Firebase](https://img.shields.io/badge/Firebase-Firestore-FFCA28?style=flat-square&logo=firebase&logoColor=black)](https://firebase.google.com)
[![CSS](https://img.shields.io/badge/CSS-Pure-1572B6?style=flat-square&logo=css3&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/CSS)
[![JavaScript](https://img.shields.io/badge/JavaScript-Vanilla-F7DF1E?style=flat-square&logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![GA4](https://img.shields.io/badge/Analytics-GA4-E37400?style=flat-square&logo=googleanalytics&logoColor=white)](https://analytics.google.com)
[![Production](https://img.shields.io/badge/Status-Production-4CAF50?style=flat-square)](https://maltabrava.com)

Public-facing website for **Malta Brava**, a craft brewery and taproom in Encarnación, Paraguay. I'm both the developer and a co-founder — so this isn't a client project, it's a product I care about shipping correctly.

Live: **[maltabrava.com](https://maltabrava.com)**

---

<img src="https://github.com/user-attachments/assets/5c1e523c-3957-453e-933f-9eb5f062d2b5" width="100%" alt="Beer catalog section — 5-column CSS Grid with amber neon cards" />

<p>
  <img src="https://github.com/user-attachments/assets/8a830354-61b8-49a5-b0f8-0d2292e69565" width="68%" alt="Spotify playlists section — facade pattern, one embed injected on click" />
  <img src="https://github.com/user-attachments/assets/1f0ca7b0-7d16-4c95-96f8-1516a68dc711" width="30%" alt="Loyalty program modal — Firebase/Firestore integration" />
</p>

---

## Why this stack

The site needed to be fast, cheap to host, and maintainable without a backend team. The audience is local — mobile-first, variable connection quality — so performance wasn't optional.

- **Astro** for static output with zero client-side JS by default. Pages ship as plain HTML/CSS. Component islands only where interactivity is actually needed.
- **Pure CSS** — no utility frameworks. Keeps the bundle lean, forces intentional design decisions, and avoids fighting a framework's opinions on specificity.
- **Vanilla JS** — the interactive parts (modals, carousels, lazy embeds) are simple enough that a framework would add overhead without adding value.
- **Firebase/Firestore** only where real-time data is required (loyalty program lookup). Everything else is static.
- **Hostinger** for hosting — static folder drop via FTP. No server to maintain.

This is a deliberate tradeoff: no CMS, no admin panel, no hot reload in production. Updates require a local rebuild and manual deploy. That's acceptable for a site that changes infrequently and where build time is under 10 seconds.

---

## Architecture decisions worth noting

### Static by default, dynamic where it matters

The site is a single-page layout with section-based navigation. There's no routing complexity, no hydration overhead. The one exception is the loyalty points lookup, which queries Firestore client-side at the time of the request. Everything else — including the beer catalog, gallery, and venue listings — is rendered at build time from local data files.

### Single source of truth for content

Structured content (beer catalog, venue list) lives in JS data files under `src/data/`. Components import from there. This avoids the classic problem of the same information hardcoded in three places and updated in two.

### Facade pattern for third-party embeds

Spotify playlist embeds are not injected on page load. The DOM renders a static placeholder; the actual `<iframe>` is injected only when the user clicks. This keeps the initial load free of third-party network requests and eliminates layout shift from embeds that take time to resolve.

### `overflow-x: clip` instead of `overflow: hidden`

`overflow: hidden` creates a new block formatting context that breaks `position: sticky` and clips box shadows on child elements. On a dark-themed site with multi-layer glow effects on cards, that's a recurring problem. `overflow-x: clip` contains horizontal overflow without the side effects. Took one bug to establish the pattern; now it's a project-wide convention.

### Progressive enhancement for scroll animations

Sections are visible by default. JS adds animation classes after the `IntersectionObserver` fires — so if JS is slow or blocked (Googlebot, low-end devices), content is never hidden. Each element is unobserved immediately after animating to avoid redundant callbacks. `prefers-reduced-motion` is respected throughout.

---

## Integrations

| Integration | Purpose | Notes |
|---|---|---|
| Firebase / Firestore | Loyalty points lookup | Client-side SDK, read-only from the site's perspective |
| Google Analytics 4 | Traffic and conversion tracking | Loaded via `is:inline` to avoid blocking render |
| Spotify Embed API | Playlist section | Injected on interaction, not on load |
| Google Maps | Venue locations | Embedded per location |
| WhatsApp deep links | Order / contact CTAs | URL-encoded message generation, no API |

---

## Interesting problems

**Scroll containers inside a centered layout**

The design called for horizontal scroll tracks in some sections that bleed to the viewport edges — not clipped to the `.container` boundary. Achieving that while keeping the section header and footer centered requires deliberately breaking the container for the scroll track, then restoring padding on the non-scroll elements. Using `max-width` on the scroll container itself breaks overflow; the correct approach is `calc((100vw - var(--container-max)) / 2)` for dynamic lateral padding.

**Consistent neon glow effects across components**

The visual identity uses a three-layer `box-shadow` on interactive cards (inner white glow, mid amber halo, outer diffusion). Getting this to look consistent across sections built months apart required locking the values into CSS custom properties and auditing usages rather than relying on copy-paste.

**Modal accessibility without a library**

Every modal on the site implements scroll lock, backdrop-click-to-close, ESC key handling, and focus management manually. Not rocket science, but it's the kind of thing that's easy to get 80% right and ship broken. The pattern is consistent across all modals — open/close logic follows the same structure everywhere.

**Age gate cookie handling**

The age verification overlay persists state via a cookie. Implementation required handling the edge case where the cookie is set but the user navigates back — making sure the gate doesn't re-render on pages where the session is already verified.

---

## Project structure (abbreviated)

```
src/
  components/     → Astro components, one per section
  data/           → JS data files (single source of truth for structured content)
  layouts/        → Base layout with head, meta tags, analytics
  pages/          → index + dynamic routes for venue menus
  styles/         → global.css (single stylesheet)
  assets/         → images (WebP, optimized by Astro's asset pipeline)
```

---

## What I'd do differently

- **i18n from day one.** Adding multilingual support to an existing site is significantly more work than designing for it upfront. The site will eventually need Spanish/English/Portuguese at minimum.
- **Content from a headless CMS for the parts that change.** The beer catalog and venue list are stable enough that build-time data files work fine. If those updated more frequently, the rebuild-and-deploy cycle would become a real friction point.
- **Automated deploys.** Manual FTP deploys are fine for now. As the site grows, a CI/CD pipeline (even a simple GitHub Actions → rsync setup) would remove a manual step from every release.

---

## Notes

This is a production site for a real business. The repository does not include environment variables, Firebase configuration, or internal data structures. If you're looking at this for the code patterns rather than the product, the interesting bits are in the component architecture and the CSS conventions.

---

*Built and maintained by Lucas — co-founder @ Malta Brava, developer by necessity and by choice.*
