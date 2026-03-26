# kuldeepchowhan.com Blog Infrastructure Spec

## Context

This is a Hugo site using the PaperMod theme, deployed to kuldeepchowhan.com. The site currently serves as a personal profile for Kuldeep Chowhan (VP Platform Engineering at Procore, Forbes Technology Council member, published author). There are no actual blog posts — all content pages are either redirect stubs to external sites (Medium, LinkedIn, Forbes) or a publications listing page.

The goal is to transform this into an SEO/GEO-optimized blog while preserving the existing profile homepage and redirect pages.

**Repository:** `~/src/kuldeepme`
**Theme:** PaperMod (in `themes/PaperMod/`, do NOT modify theme files)
**Config:** `config.yml` at root
**Deployment:** Static site on Cloudflare Pages (or similar CDN)

---

## 1. Config Overhaul (`config.yml`)

Replace the existing `config.yml` with an enhanced version. Key changes:

### 1.1 Site-level settings
- Keep `baseURL: https://kuldeepchowhan.com/`
- Keep `theme: PaperMod`
- Add `enableRobotsTXT: false` (we manage our own in `static/robots.txt`)
- Add `paginate: 10`
- Set `params.env: production` so schema/OG/twitter templates render

### 1.2 Taxonomies
```yaml
taxonomies:
  tag: tags
  category: categories
  series: series
```

### 1.3 Menu
```yaml
menu:
  main:
    - identifier: blog
      name: Blog
      url: /blog/
      weight: 10
    - identifier: publications
      name: Publications
      url: /publications/
      weight: 20
    - identifier: about
      name: About
      url: /about/
      weight: 30
    - identifier: search
      name: Search
      url: /search/
      weight: 40
```

### 1.4 Params additions
```yaml
params:
  ShowReadingTime: true
  ShowShareButtons: true
  ShowPostNavLinks: true
  ShowBreadCrumbs: true
  ShowCodeCopyButtons: true
  ShowWordCount: false
  ShowToc: true
  TocOpen: false
  defaultTheme: auto
  comments: false

  # Schema.org configuration
  schema:
    publisherType: Person
    sameAs:
      - "https://www.linkedin.com/in/kuldeepchowhan/"
      - "https://github.com/kchowhan"
      - "https://councils.forbes.com/profile/Kuldeep-Chowhan-Vice-President-Marketplace-Technology-Walmart/b321c354-4586-4af1-932d-f94804a499ae"
      - "https://medium.com/@kuldeep"

  # Author info (used by schema partials)
  author:
    name: "Kuldeep Chowhan"
    url: "https://kuldeepchowhan.com"
    jobTitle: "Vice President, Platform Engineering"
    worksFor: "Procore Technologies"
    sameAs:
      - "https://www.linkedin.com/in/kuldeepchowhan/"
      - "https://github.com/kchowhan"
      - "https://councils.forbes.com/profile/Kuldeep-Chowhan-Vice-President-Marketplace-Technology-Walmart/b321c354-4586-4af1-932d-f94804a499ae"
```

### 1.5 Outputs (keep existing)
```yaml
outputs:
  home:
    - HTML
    - RSS
    - JSON
```

### 1.6 Sitemap (keep existing)
```yaml
sitemap:
  changefreq: weekly
  filename: sitemap.xml
  priority: 0.5
```

### 1.7 Markup settings (keep existing, add)
```yaml
markup:
  goldmark:
    renderer:
      unsafe: true
  highlight:
    noClasses: false
  tableOfContents:
    startLevel: 2
    endLevel: 4
```

---

## 2. Content Structure

### 2.1 Blog section: `content/blog/`

Create `content/blog/_index.md`:
```yaml
---
title: "Blog"
description: "Technical insights on platform engineering, distributed systems, cloud architecture, and engineering leadership by Kuldeep Chowhan."
---
```

### 2.2 About page: `content/about.md`

Create a standalone about page with the following frontmatter and content structure:

```yaml
---
title: "About Kuldeep Chowhan"
description: "Kuldeep Chowhan is VP of Platform Engineering at Procore Technologies, Forbes Technology Council member, and author of Hands on Serverless Computing."
keywords:
  - "kuldeep chowhan"
  - "platform engineering"
  - "technology leadership"
  - "forbes technology council"
author: "Kuldeep Chowhan"
layout: "single"
ShowToc: false
ShowReadingTime: false
ShowShareButtons: false
---
```

Content should be a concise professional bio (3-4 paragraphs) covering:
- Current role at Procore
- Previous roles (Walmart, SoFi, Expedia, Microsoft) — brief summary, not full resume
- Publications and speaking (link to /publications/)
- Forbes Technology Council membership
- Areas of expertise
- A note that the blog covers topics in platform engineering, distributed systems, cloud architecture, and engineering leadership

Source the bio content from the existing `static/llms.txt` file. Keep it human and readable, not resume-formatted.

### 2.3 Move search page

Move `content/posts/search.md` to `content/search.md` (same content, just relocate). Delete the `content/posts/` directory if empty after the move.

### 2.4 Keep existing pages

Do NOT modify these existing files (they work as-is):
- `content/linkedin.md`
- `content/linkedinprofile.md`
- `content/chaosengblog.md`
- `content/serverlessblog.md`
- `content/forbes.md`
- `content/medium.md`
- `content/publications.md`

---

## 3. Blog Post Archetype

Create `archetypes/blog.md` — this is the template for new posts created via `hugo new blog/my-post.md`:

```yaml
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
lastmod: {{ .Date }}
draft: true
description: ""
summary: ""
keywords: []
tags: []
categories: []
series: []
author: "Kuldeep Chowhan"

# Cover image (optional)
cover:
  image: ""
  alt: ""
  caption: ""
  relative: true

# SEO/GEO
ShowToc: true
TocOpen: false
ShowReadingTime: true
ShowShareButtons: true
ShowPostNavLinks: true

# FAQ Schema (optional — populate to auto-generate FAQPage schema)
# Each item generates a Question/Answer pair in structured data
faq:
  # - question: "What is platform engineering?"
  #   answer: "Platform engineering is the discipline of designing and building toolchains and workflows that enable self-service capabilities for software engineering organizations."

# HowTo Schema (optional — populate to auto-generate HowTo schema)
# howto:
#   name: ""
#   description: ""
#   totalTime: "PT30M"
#   steps:
#     - name: "Step 1"
#       text: "Description of step 1"
#     - name: "Step 2"
#       text: "Description of step 2"
---

<!--
GEO WRITING GUIDELINES:
1. First 40-60 words should directly answer the post's core question
2. Include a statistic or data point every 150-200 words
3. Use H2/H3 headings as questions where possible (AI engines extract these)
4. Cite authoritative sources with links
5. End with a clear summary/takeaway section
6. If applicable, include an FAQ section at the bottom (and populate the faq frontmatter above)
-->
```

---

## 4. Schema/Structured Data Partials

**IMPORTANT:** All schema partials go in `layouts/partials/` (project-level overrides). Do NOT modify any files inside `themes/PaperMod/`.

### 4.1 Override schema: `layouts/partials/templates/schema_json.html`

This file overrides PaperMod's default `schema_json.html`. It must include:

**For the homepage (`{{ if .IsHome }}`):**
- `@type: Person` (not Organization) with:
  - `name`, `url`, `description`, `jobTitle`, `worksFor` (from `site.Params.author`)
  - `sameAs` array (LinkedIn, GitHub, Forbes, Medium from `site.Params.author.sameAs`)
  - `image` pointing to `/Kuldeep.webp`
  - `alumniOf` with Berkeley Haas and ISB
  - `memberOf` for Forbes Technology Council
  - `knowsAbout` array with key technical domains

**For blog posts (`{{ if .IsPage }}`):**
- `@type: BlogPosting` with all fields from PaperMod's existing template PLUS:
  - Enhanced `author` block: `@type: Person` with `url`, `jobTitle`, `sameAs` (not just name)
  - `publisher` as `@type: Person` (not Organization) with same author details
  - `datePublished` and `dateModified` in ISO 8601 format
  - `articleSection` from the page's section
  - `wordCount`
  - `timeRequired` calculated from word count (e.g., "PT5M" for a 5-minute read)

- **Conditional FAQPage schema:** If `.Params.faq` exists and has items, output an ADDITIONAL `<script type="application/ld+json">` block with:
  ```json
  {
    "@context": "https://schema.org",
    "@type": "FAQPage",
    "mainEntity": [
      {
        "@type": "Question",
        "name": "question text",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "answer text"
        }
      }
    ]
  }
  ```

- **Conditional HowTo schema:** If `.Params.howto` exists, output an ADDITIONAL `<script type="application/ld+json">` block with:
  ```json
  {
    "@context": "https://schema.org",
    "@type": "HowTo",
    "name": "howto name",
    "description": "howto description",
    "totalTime": "PT30M",
    "step": [
      {
        "@type": "HowToStep",
        "name": "step name",
        "text": "step text"
      }
    ]
  }
  ```

**For section/list pages (`{{ if .IsSection }}`):**
- `@type: CollectionPage` with name, description, url
- Keep existing BreadcrumbList schema from PaperMod

**BreadcrumbList:** Keep PaperMod's existing BreadcrumbList logic for all non-home pages (copy it from the theme's `schema_json.html` and include it in the override).

### 4.2 Enhanced extend_head: `layouts/partials/extend_head.html`

Override the existing `extend_head.html`. Keep the existing publications CSS conditional, and ADD:

- A `<link rel="alternate" type="application/rss+xml">` tag for the blog RSS feed
- An `llms.txt` reference: `<link rel="alternate" type="text/plain" href="/llms.txt" title="LLMs.txt">`
- Preconnect hints for any CDN assets if needed

---

## 5. Custom Shortcodes

Create these shortcodes in `layouts/shortcodes/`:

### 5.1 `faq.html`

A shortcode for rendering FAQ sections in blog posts with proper HTML structure. Usage in markdown:

```
{{</* faq question="What is platform engineering?" */>}}
Platform engineering is the discipline of designing and building toolchains and workflows...
{{</* /faq */>}}
```

Output HTML:
```html
<div class="faq-item" itemscope itemprop="mainEntity" itemtype="https://schema.org/Question">
  <h3 itemprop="name">What is platform engineering?</h3>
  <div itemscope itemprop="acceptedAnswer" itemtype="https://schema.org/Answer">
    <div itemprop="text">
      Platform engineering is the discipline of designing and building toolchains and workflows...
    </div>
  </div>
</div>
```

### 5.2 `callout.html`

A shortcode for highlighted callout/summary boxes (useful for GEO — AI engines extract these):

```
{{</* callout type="summary" */>}}
Key takeaway text here.
{{</* /callout */>}}
```

Supported types: `summary`, `tldr`, `warning`, `tip`. Output should be a styled `<aside>` or `<div>` with appropriate CSS class.

### 5.3 `cite.html`

A shortcode for inline citations (builds credibility for GEO):

```
{{</* cite source="AWS Documentation" url="https://docs.aws.amazon.com/..." */>}}
```

Output: a styled inline citation link.

---

## 6. CSS Additions

Create `assets/css/extended/custom.css` (PaperMod automatically picks up CSS files from this directory):

Add minimal styles for:
- `.faq-item` — slight left border, padding, margin between items
- `.callout` — background tint, left border accent, padding. Different colors for each type (summary=blue, tldr=green, warning=amber, tip=purple)
- `.cite-inline` — small, muted text style for citation links
- Keep it minimal and consistent with PaperMod's existing design language and CSS variables (use `var(--primary)`, `var(--secondary)`, `var(--border)`, etc.)

---

## 7. Sample Blog Post

Create one sample blog post to validate the entire pipeline: `content/blog/platform-engineering-in-2026.md`

Requirements:
- Real, substantive content (not lorem ipsum) — 800-1200 words
- Topic: "What Platform Engineering Actually Means in 2026" or similar
- Written from Kuldeep's perspective (first person, drawing on his experience at Procore, Walmart, Expedia)
- Follow the GEO writing guidelines from the archetype:
  - Answer-first opening paragraph (40-60 words directly addressing the topic)
  - Statistics/data points sprinkled throughout
  - H2/H3 headings phrased as questions where natural
  - Citations to authoritative sources
  - A summary/takeaway section at the end
  - An FAQ section at the bottom using the `faq` shortcode AND the `faq` frontmatter field (both, so schema generates from frontmatter and visible content renders via shortcode)
- Populate all frontmatter fields including tags, categories, description, summary, keywords, cover image placeholder
- Set `draft: false` so it renders

---

## 8. llms.txt Update

Update `static/llms.txt` to append a section at the bottom:

```
## Blog

Kuldeep writes about platform engineering, distributed systems, cloud architecture, and engineering leadership on his blog at https://kuldeepchowhan.com/blog/. Topics include lessons from scaling engineering organizations, cloud migration strategies, and building developer platforms.

## Blog Feed

RSS: https://kuldeepchowhan.com/blog/index.xml
```

---

## 9. Robots.txt Update

The existing `static/robots.txt` is excellent. Add one line to reference the blog sitemap if Hugo generates a separate one, or verify the existing sitemap reference covers blog posts.

No other changes needed.

---

## 10. Verification Checklist

After implementing everything, verify:

1. **`hugo server`** runs without errors
2. **Homepage** still shows the profile page with Person schema in page source
3. **`/blog/`** shows the blog list page with the sample post
4. **`/blog/platform-engineering-in-2026/`** renders correctly with:
   - Table of contents
   - Reading time
   - Share buttons
   - Breadcrumbs
   - FAQ section visible in content
5. **View page source** on the sample blog post and confirm:
   - `BlogPosting` JSON-LD with enhanced author
   - `FAQPage` JSON-LD with questions from frontmatter
   - `BreadcrumbList` JSON-LD
   - Open Graph meta tags
   - Twitter card meta tags
   - Canonical URL
6. **`/about/`** renders the bio page
7. **`/search/`** works (PaperMod's Fuse.js search)
8. **`/publications/`** still renders correctly
9. **Redirect pages** (`/linkedin`, `/forbes`, `/medium`, etc.) still work
10. **`/sitemap.xml`** includes all blog posts, about page, publications
11. **`/index.xml`** (RSS) includes blog posts
12. **Navigation menu** shows Blog, Publications, About, Search
13. **Google Rich Results Test**: Copy the BlogPosting JSON-LD into https://search.google.com/test/rich-results and confirm no errors
14. **Schema Markup Validator**: Test at https://validator.schema.org/ for the homepage Person schema

---

## Architecture Notes

- **Never modify files inside `themes/PaperMod/`** — all customizations go in project-level `layouts/`, `assets/`, `archetypes/`, `static/`
- PaperMod supports project-level overrides: any file in `layouts/` with the same path as a theme file takes precedence
- The `extend_head.html` and `extend_footer.html` partials are specifically designed by PaperMod for user customization
- CSS files in `assets/css/extended/` are automatically concatenated by PaperMod's head partial
- The `params.env: production` setting is required for schema/OG/twitter templates to render (see `head.html` line 165)

---

## File Tree (what should exist after implementation)

```
kuldeepme/
├── archetypes/
│   └── blog.md                          # NEW: blog post template
├── assets/
│   └── css/
│       └── extended/
│           └── custom.css               # NEW: FAQ, callout, cite styles
├── config.yml                           # MODIFIED: enhanced config
├── content/
│   ├── about.md                         # NEW: about page
│   ├── blog/
│   │   ├── _index.md                    # NEW: blog section index
│   │   └── platform-engineering-in-2026.md  # NEW: sample post
│   ├── search.md                        # MOVED from content/posts/search.md
│   ├── publications.md                  # UNCHANGED
│   ├── linkedin.md                      # UNCHANGED
│   ├── linkedinprofile.md               # UNCHANGED
│   ├── chaosengblog.md                  # UNCHANGED
│   ├── serverlessblog.md                # UNCHANGED
│   ├── forbes.md                        # UNCHANGED
│   └── medium.md                        # UNCHANGED
├── layouts/
│   ├── partials/
│   │   ├── extend_head.html             # MODIFIED: add RSS + llms.txt links
│   │   ├── index_profile.html           # UNCHANGED
│   │   └── templates/
│   │       └── schema_json.html         # NEW: override PaperMod schema
│   └── shortcodes/
│       ├── faq.html                     # NEW
│       ├── callout.html                 # NEW
│       └── cite.html                    # NEW
├── static/
│   ├── llms.txt                         # MODIFIED: add blog section
│   ├── robots.txt                       # UNCHANGED (verify only)
│   └── ... (existing assets)            # UNCHANGED
└── themes/
    └── PaperMod/                        # NEVER MODIFY
```
