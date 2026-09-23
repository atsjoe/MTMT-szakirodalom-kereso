# MTMT Literature Search & Network Analysis Engine (M2-SZIK)

A serverless, client-side scientific literature search and bibliometric network analysis platform connecting the **Hungarian National Scientific Bibliography (MTMT v2)** with the global **OpenAlex** academic knowledge graph.

[Magyar nyelvű leírás (Hungarian README)](README.md) • [English Developer Documentation (DEVELOPER_GUIDE.md)](DEVELOPER_GUIDE.md)

---

## 🌟 Key Features

1. **Elevated Search Bar Token UX:**
   - Pre-filters (Author, Journal, Institution, Publication Year, Publication Role) automatically inject as interactive visual chips/tokens into the primary search bar.
   - Explicit **`AND`** chips depict the Boolean relationship between entities and free-text search terms.
   - One-click instant dismissal (`✕`) synchronizes both the token and filter controls.

2. **Google-Style Smart Query Parser (`okosKeresoFeldolgozo`):**
   - Space-separated terms treated with implicit AND logic.
   - Exact phrase queries supported via double quotes (`"systematic review"`).
   - Exclusion terms supported with prefix minus (`-review*`).

3. **Systematic Literature Review (SLR / PRISMA Support):**
   - Building-block search matrix: rows for synonyms (OR), columns/levels for conceptual dimensions (AND).
   - Client-side Cartesian product generator dispatching parallel asynchronous requests to circumvent MTMT API Boolean tree limitations.

4. **Dedicated Researcher Thesaurus (`szotar.html`):**
   - Micro-service running on browser `localStorage` for custom terminology and synonym trees.
   - Clean TXT export and import capabilities for reproducible sharing among research teams.

5. **Bibliometric Network Discovery (OpenAlex Graph Integration):**
   - **Bibliographic Coupling (Related Works):** Discovers related literature based on shared reference lists.
   - **Co-Citation Algorithm:** Uncovers items frequently co-cited in third-party bibliographies (with rate-limit protected bulk queries).
   - **Network Drawer & Data Fusion:** Normalize and merge external OpenAlex works directly into the active MTMT result set with zero metadata loss.

6. **Academic Export & Reference Management:**
   - Formatted citation generation: **APA, IEEE, MLA, Chicago**.
   - Standard export files: **BibTeX** (.bib with case protection brackets), **RIS** (.ris with internalId fallback for Zotero/EndNote/Mendeley), **CSL-JSON**, and styled **HTML**.
   - Session preservation (**JSON**) with built-in publication alerting / novelty detection.

---

## 🚀 Getting Started & Execution

The application is **100% client-side**, requiring no Node.js backend, no SQL database, and no server configuration:

### Option A: Local Execution
1. Clone or download this repository.
2. Double-click `index.html` in any modern web browser (Chrome, Firefox, Edge, Safari).
3. The application directly and securely queries the public REST APIs of MTMT and OpenAlex.

### Option B: 1-Click Live Web Hosting via GitHub Pages
1. Go to your repository on GitHub: **Settings -> Pages**.
2. Set source to **Deploy from a branch** -> branch: `main` (or `master`), folder: `/ (root)`.
3. Click **Save**. Within 1–2 minutes, your live scientific platform is published at:
   `https://<your-username>.github.io/<repository-name>/index.html`

---

## 📁 Repository Structure

```text
├── index.html                  # Main application UI and client-side logic
├── style.css                   # Complete visual design system (CSS Grid, tokens, modals)
├── sugo.html                   # Detailed user guide (Hungarian)
├── szotar.html                 # Custom thesaurus micro-service (localStorage)
├── Pelda-mentett-kereses.json  # Sample saved search profile for testing
│
├── README.md                   # Hungarian README
├── README.en.md                # English README
├── FEJLESZTOI_DOKUMENTACIO.md  # Hungarian developer documentation
├── DEVELOPER_GUIDE.md          # Comprehensive English technical guide
├── mapping_mtmt_api.json       # MTMT v2 API data field mapping
├── mapping_openalex_api.json   # OpenAlex API data field mapping
└── mapping_export_formats.json # RIS, BibTeX, and CSL-JSON schema crosswalks
```

---

## 📖 Technical Architecture & Engineering Solutions

A comprehensive description of client-side algorithms, rate-limiting guards, and API workarounds (Bulldozer pagination, Dual-query journal matching, Relevance scoring, ISSN regex stripping, and InternalId citation injection) is documented in **[DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md)**.
