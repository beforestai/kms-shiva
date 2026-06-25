# Beforest KMS Prototype Logic Mapping

This document explains the current logic of the Beforest KMS static prototype in `kms-prototype.html`: what each module does, what data it uses, how users interact with it, and what the feature logic is meant to represent in a future live system.

## 1. Prototype Architecture

The prototype is a single static HTML application. It does not use a backend, database, authentication service, or routing framework yet.

Current architecture:

- UI routes are handled by showing and hiding `<section>` blocks.
- Sidebar buttons use `data-view` to open a matching section ID.
- Internal links use `data-view-link` to open another section.
- Demo data is stored in JavaScript arrays.
- User-created pages are saved to browser `localStorage`.
- Collective page overviews can call the OpenAI API if an API key is manually available in the browser.

Main route logic:

```text
Sidebar click -> setView(viewId) -> matching section becomes active -> nav item becomes active
```

## 2. Core Data Model

Every KMS page follows the same basic page object structure:

```text
id
title
slug
team
category
summary
content
fileType
fileName
tags
relatedTeams
sourceLink
updated
```

Data sources:

- `seedDocuments`: fixed demo pages included inside the prototype.
- `localStorage.beforest-kms-demo-documents`: pages created through the Create Page modal.
- `getDocuments()`: merges seed pages and locally created pages.

Important logic:

```text
getDocuments() = seedDocuments + saved local pages
renderDocuments() = sort pages + apply filters + update visible page lists
```

## 3. App Shell And Navigation

### Purpose

The app shell provides the overall KMS structure: sidebar navigation, top search, demo user identity, and Create Page access.

### Sidebar Modules

Current sidebar modules:

- Knowledge Home
- Team Spaces
- All Pages
- Recently Updated
- Collective-wise Information
- Glossary
- Templates

### Interaction Logic

When a user clicks a sidebar item:

```text
Click sidebar item
-> read data-view value
-> call setView(viewId)
-> hide all .view sections
-> show the selected section
-> update active sidebar state
```

### Live System Logic

In a live system, this same navigation logic can become real routes, for example:

```text
/kms/home
/kms/teams
/kms/pages
/kms/shared/collective-wise-information
/kms/shared/glossary
/kms/templates
```

## 4. Knowledge Home

### Purpose

Knowledge Home is the main dashboard. It gives users a quick view of what is happening across the KMS.

### What It Shows

- Greeting for the demo user.
- Total team count.
- Total page count.
- Shared space count.
- Quick filters.
- Recently updated pages.
- Team knowledge space preview.
- Common references.
- Recent activity.

### Data Flow

```text
seedDocuments + localStorage pages
-> getDocuments()
-> getSortedDocuments()
-> renderDocuments()
-> home recent list
-> doc count

teams array + documents
-> renderTeams()
-> team cards
-> team count

sorted documents
-> renderActivity()
-> recent activity card
```

### Interaction Logic

- Clicking `View all` opens Recently Updated.
- Clicking `See all teams` opens Team Spaces.
- Clicking Collective-wise Information in Common References opens the Collective-wise Information page.
- Quick filters update the visible page lists.

## 5. Search And Quick Filters

### Purpose

Search and filters help users find pages by topic, team, category, keywords, and content.

### Search Logic

The search input checks against a combined searchable string:

```text
title + team + category + summary + content + fileType + fileName + tags + relatedTeams
```

The matching is case-insensitive.

### Quick Filter Logic

Current quick filters:

- All teams
- My team
- SOPs
- FAQs
- Reports

`My team` currently maps to Marketing because the demo user is shown as `Demo User - Marketing`.

### Live System Logic

In the live system, `My team` should use the logged-in user's actual team mapping.

```text
Logged-in user -> user.team -> My team filter -> pages where page.team == user.team
```

## 6. Team Spaces

### Purpose

Team Spaces shows all pilot teams and how many pages each team owns.

### Current Teams

The main team list includes:

- Business Intelligence
- Be Wild
- Collective Design & Support
- Community Engagement
- Admin
- HR
- Finance
- Collective Operations
- Hospitality
- Regolith
- Marketing
- Content & Storytelling
- Legal & Liaisoning

### Data Flow

```text
teams array
+ getDocuments()
-> count documents where doc.team == team name
-> render team cards
```

### Interaction Logic

When a user clicks a team card:

```text
Click team card
-> set hidden team filter to that team
-> clear quick filters
-> open All Pages
-> render documents owned by selected team
```

### Live System Logic

In the live system, Team Spaces should show team-owned pages from the database, not from local demo arrays.

## 7. All Pages

### Purpose

All Pages is the main page listing for created and seeded KMS pages.

### What It Shows

Each page card shows:

- Page title
- Summary
- Team owner
- File type
- Last updated date
- Open page link
- View details button

### Data Flow

```text
seed pages + local pages
-> sorted by updated date
-> filtered by search/team/quick filter
-> rendered as page cards
```

### Interaction Logic

- `Open page` opens a generated page URL pattern.
- `View details` opens the Page Details modal.

Generated page URL pattern:

```text
/kms/{team-slug}/{page-slug}
```

Example:

```text
/kms/hospitality/guest-stay-information-pack
```

### Live System Logic

In the live system, this should point to actual page routes where the full page can be opened and cited by the future KMS chat.

## 8. Recently Updated

### Purpose

Recently Updated shows the latest changed pages across the KMS.

### Data Flow

```text
getDocuments()
-> sort by updated date descending
-> apply filters
-> render recent rows
```

### Interaction Logic

Clicking a recent row opens the Page Details modal.

### Live System Logic

The live version should use actual `updated_at` timestamps from the database.

## 9. Shared Knowledge Spaces

### Purpose

Shared Knowledge Spaces are organization-level reference areas used across teams.

### Current Shared Areas In Prototype

- Collective-wise Information
- Templates
- Glossary is available from the sidebar as a coming-soon module.

### Interaction Logic

The Shared Knowledge Spaces landing page has cards that open shared modules.

Current card behavior:

```text
Open Collective-wise Information -> setView("collective")
Open Templates -> setView("templates")
```

## 10. Collective-wise Information

### Purpose

This module maps collective-related knowledge to actual Beforest source webpages.

It is meant for users who want information about collectives, such as:

- What is a collective?
- Poomaale 1.0
- Poomaale 2.0
- Hyderabad Collective
- Hammiyala Collective
- Bhopal Collective
- Mumbai Collective

### Page Structure

The page has two columns:

- Left: selected collective page and generated overview.
- Right: grouped web references.

### Dropdown Logic

The dropdown contains source webpages under `Collective Webpages`.

When the user selects a page:

```text
Select page
-> check overview cache for selected URL
-> if cached, show saved overview
-> if not cached, show placeholder
```

### Generate Overview Logic

When `Generate overview` is clicked:

```text
Click Generate overview
-> get selected URL
-> check local overview cache
-> check OpenAI API key
-> show loading state
-> send URL to OpenAI chat completions endpoint
-> ask model to read page and write 3-5 sentence overview
-> save returned overview in localStorage cache
-> show Source verified badge
```

Cache key:

```text
localStorage.beforest-kms-collective-overviews
```

### Web References Sidebar

The sidebar groups official references by category:

- Reference
- Collective Webpages
- Experiences
- 10 percent lifestyle
- Hospitality
- Bewild
- Beforest

Clicking a web reference opens the URL in a new browser tab.

### Live System Logic

In the live version, this module should become a source-backed reference space. The system can periodically fetch the listed URLs, index the page content, and make it available for search or future chat citations.

## 11. Glossary

### Purpose

Glossary is currently a coming-soon module. It explains the intended AI-powered glossary flow without requiring manual glossary entry today.

### Current Page Sections

- Coming soon banner for AI-powered glossary.
- Three-step flow:
  - New page saved
  - Agent extracts terms
  - Definition published
- Empty Glossary terms card.
- Disabled search field.

### Current Logic

There is no glossary data or extraction logic yet. The page is static.

### Future Logic

The intended live flow is:

```text
New KMS page saved
-> agent reads the page
-> agent identifies Beforest-specific terms
-> agent checks source context
-> agent generates accurate definition
-> glossary term appears automatically
```

The important product decision is that teams do not manually maintain glossary terms.

## 12. Templates

### Purpose

Templates explain how KMS pages should be structured and what details are useful for search, reuse, and future citations.

### Current Content

The template card lists fields such as:

- title
- team_owner
- category
- topic
- summary
- rich_text_content
- keywords
- related_teams
- source_link
- last_updated

### Logic

This page is static. It does not generate or enforce templates yet.

### Live System Logic

In a live version, these fields can become validation rules or form fields in the Create Page flow.

## 13. Create Page Modal

### Purpose

Create Page lets team members create a knowledge page directly. No approval is required in the prototype logic.

### Current Form Fields

In order:

1. Title
2. Team owner
3. Topic / summary
4. Page content rich text editor
5. Keywords
6. Related teams
7. Source link
8. Supporting file

### Team Owner Logic

The Create Page dropdown uses a smaller list:

- Business Intelligence
- Be Wild
- Collective Design & Support
- Community Engagement
- Admin
- HR
- Finance
- Collective Operations
- Hospitality
- Marketing

### Rich Text Editor Logic

The editor supports:

- Heading
- Bold
- Italic
- Unordered list
- Ordered list
- Table
- Link
- Image upload

Table logic:

```text
Click table icon
-> insert simple HTML table into page content
```

Link logic:

```text
Click link icon
-> ask for URL
-> insert link into selected content
```

Image logic:

```text
Click image icon
-> open local image picker
-> read selected image as data URL
-> insert image into rich text editor
```

Supporting file logic:

```text
Choose supporting file
-> store file name and inferred file type in page object
```

The prototype does not upload files to a server. It only records the selected file name and type.

### Save / Publish Logic

Both Save draft and Publish page currently save the page locally.

```text
Submit form
-> validate page content exists
-> build page object
-> add object to localStorage
-> clear form
-> close modal
-> re-render teams, pages, and activity
-> open All Pages
-> show toast
```

Toast behavior:

- Save draft shows `Draft saved.`
- Publish page shows `Page published.`

### Live System Logic

In the live system:

- Save draft should save with draft status if drafts are needed.
- Publish page should create a live page immediately.
- The page should be indexed after save/publish.
- The page should become available for search and future chat citations.

## 14. Page Details Modal

### Purpose

The Page Details modal shows the full details of a selected page.

### What It Shows

- Summary
- Page content
- Cited page link
- Supporting file
- Keywords
- Related teams
- Source link
- Last updated

### Interaction Logic

The modal opens when:

- A recently updated row is clicked.
- A page card's View details button is clicked.

### Cited Link Logic

The prototype generates a citable page path:

```text
/kms/{team-slug}/{page-slug}
```

### Live System Logic

This link should become the page URL used by the future KMS chat for citations.

## 15. Local Storage

### Purpose

Local storage makes the static demo feel persistent without a backend.

### Keys Used

```text
beforest-kms-demo-documents
beforest-kms-collective-overviews
```

### Page Storage Logic

```text
Create page
-> save custom page array to localStorage
-> getDocuments() merges it with seedDocuments
```

### Overview Cache Logic

```text
Generate collective overview
-> save overview by URL
-> next visit reuses cached overview
```

### Limitation

Data is saved only in the user's current browser. It is not shared across users or devices.

## 16. Future KMS Chat And Citation Logic

The prototype is designed so the future chat layer can cite KMS pages instead of only answering directly.

Intended flow:

```text
User creates or publishes page
-> page is saved in KMS database
-> indexing automation reads page content
-> page content is chunked and vectorized
-> user asks question in KMS chat
-> search retrieves matching page chunks
-> chat answers with citations
-> citations link back to KMS pages
```

Important product logic:

- Users create pages like internal blog/wiki pages.
- Supporting documents are optional.
- URLs can be used as source references.
- The KMS should cite internal pages as links.
- Glossary terms should eventually be generated automatically by an agent.

## 17. Current Prototype Limitations

The current prototype is intentionally static/local.

Limitations:

- No real login.
- No backend database.
- No real file upload storage.
- No shared user state.
- No real approval workflow.
- No real draft/published separation yet.
- No production search index.
- No vector database.
- No live chatbot interface yet.
- OpenAI overview generation only works if an API key is manually available in the browser.

## 18. Suggested Live System Mapping

| Prototype Module | Live System Equivalent |
| --- | --- |
| `seedDocuments` | Database records |
| `localStorage` pages | User-created KMS pages |
| Sidebar `setView()` | Real app routing |
| Demo user - Marketing | Authenticated user profile |
| My team filter | User-team mapping |
| Create Page modal | Page creation form |
| Supporting file name | Uploaded file storage |
| Rich text content | Stored HTML / editor document format |
| Generated page path | Real page route |
| Collective URL list | Source registry |
| Generate overview | Source-reading automation |
| Glossary coming soon | Agent-built glossary service |
| Search filter | Database search + vector search |
| Future citations | Links to internal KMS pages |

## 19. End-To-End User Flow

### Create And Discover A Page

```text
User clicks + Create Page
-> fills title, owner, summary, content, keywords, related teams
-> optionally adds image, source link, and supporting file
-> clicks Publish page
-> page is saved locally
-> page appears in All Pages
-> page count updates
-> team page count updates
-> recent activity updates
-> page can be opened through generated citable link
```

### Find Collective Information

```text
User opens Collective-wise Information
-> selects a collective webpage
-> generates or views cached overview
-> opens source links from Web references if needed
```

### Future Glossary Flow

```text
User publishes new page
-> glossary agent reads new content
-> Beforest-specific terms are identified
-> definitions are generated from source context
-> terms appear in Glossary automatically
```

