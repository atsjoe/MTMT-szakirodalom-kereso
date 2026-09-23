# MTMT Szakirodalom Kereső és Hálózatkutató Modul (M2-SZIK)

Kliensoldali (Serverless), tisztán böngészőben futó tudományos szakirodalom-kutató és bibliográfiai elemző platform a **Magyar Tudományos Művek Tára (MTMT v2)** és az **OpenAlex** globális tudományos gráfjához.

[English documentation (README.en.md)](README.en.md) • [English Developer Guide (DEVELOPER_GUIDE.md)](DEVELOPER_GUIDE.md)

---

## 🌟 Főbb Képességek és Funkciók

1. **Beemelt Keresősáv (Elevated Search Bar Token UX):**
   - Az előszűrőkben kijelölt szempontok (Szerző, Folyóirat, Intézmény, Évszám, Közlemény szerepe) közvetlenül beemelődnek a fő keresősávba interaktív jelvényként (token).
   - Vizuális **`ÉS`** logikai kapcsolat jelzi a feltételek összefüggését a szabadszöveges kulcsszavakkal.
   - Egy kattintással (`✕`) törölhető a keresősávból és az előszűrőkből egyaránt.

2. **Google-stílusú egyszerű kereső (`okosKeresoFeldolgozo`):**
   - Szóközzel elválasztott szavak (implicit ÉS kapcsolat).
   - Idézőjeles pontos kifejezés-keresés (`"systematic review"`).
   - Kizáró feltételek mínusz jellel (`-post*`).

3. **Szisztematikus Irodalomkutatás (SLR / PRISMA):**
   - Építőkocka-módszer: sorokon belüli szinonimák (VAGY) és sorok közötti szempontok (ÉS).
   - Kliensoldali Descartes-szorzat generálás, amellyel a böngésző párhuzamos lekérdezésekkel hidalja át az MTMT API logikai fa korlátait.

4. **Saját Szótár és Tezaurusz Kezelő (`szotar.html`):**
   - Böngészőben (`localStorage`) perzisztált saját szinonimaszótár.
   - Szöveges (TXT) export és import lehetőség a szótár mentésére és megosztására.

5. **Hálózatkutató Modul (OpenAlex API Integráció):**
   - **Bibliográfiai csatolás (Related Works):** rokon publikációk felderítése a közös hivatkozások alapján.
   - **Ko-citációs algoritmus:** olyan művek azonosítása, amelyeket a harmadik fél gyakran idéz együtt a kiválasztott cikkekkel (rate-limit védett kötegelt lekérdezéssel).
   - **Hálózati Tálca & Adatfúzió:** a kiválasztott OpenAlex cikkek normalizálva átemelhetők az MTMT találati listába.

6. **Tudományos Export & Referenciakezelés:**
   - Formázott bibliográfia: **APA, IEEE, MLA, Chicago** szabványok.
   - Fájlexportok: **BibTeX** (.bib - case protection címekkel), **RIS** (.ris - EndNote, Zotero, Mendeley kompatibilis oldalszám/internalId pótlásokkal), **CSL-JSON** és **HTML**.
   - Munkamenet mentése és visszatöltése (**JSON**), beépített újdonságfigyelő (Alerting) szűrővel.

---

## 🚀 Használat és Futtatás

Az alkalmazás **tisztán kliensoldali**, így semmilyen szervert, Node.js-t, adatbázist vagy telepítést nem igényel:

### 1. Futtatás helyi gépen:
1. Töltse le a repository fájljait (vagy klónozza a repót).
2. Kattintson duplán az `index.html` fájlra a böngészőjében (Chrome, Firefox, Edge, Safari).
3. Az alkalmazás azonnal működik és kommunikál az MTMT és OpenAlex nyilvános API-jaival.

### 2. Élő futtatás GitHub Pages-ről:
Kapcsolja be a GitHub repository-ban a **Settings -> Pages** menüpont alatt a GitHub Pages hosztolást:
- Forrás: `Deploy from a branch` -> `main` / `root`.
- A mentés után a weboldal azonnal elérhető nyilvános címen: `https://<felhasznalonev>.github.io/<repo-nev>/index.html`

---

## 📁 Repository Fájlszerkezet

```text
├── index.html                  # A fő alkalmazás felülete és kliensoldali logikája
├── style.css                   # Az alkalmazás teljes egységes stíluslapja (UI/UX)
├── sugo.html                   # Részletes felhasználói kézikönyv és útmutató
├── szotar.html                 # Saját szinonimaszótár mikroszolgáltatás (localStorage)
├── Pelda-mentett-kereses.json  # Mintafájl a keresések mentésének és betöltésének teszteléséhez
│
├── FEJLESZTOI_DOKUMENTACIO.md  # Részletes műszaki és architekturális leírás (Hacks & Workarounds)
├── mapping_mtmt_api.json       # MTMT v2 API mezőleképezési specifikáció
├── mapping_openalex_api.json   # OpenAlex API adatmodell specifikáció
└── mapping_export_formats.json # RIS, BibTeX és CSL-JSON mezőmegfeleltetések
```

---

## 📖 Műszaki és Fejlesztői Részletek

A rendszer fejlesztése során alkalmazott speciális API védelmi mechanizmusokról (Bulldózer lapozás, Dual-query folyóiratkeresés, Relevancia pontozó motor, ISSN regex tisztítás, InternalId injekció) a részletes leírás a **[FEJLESZTOI_DOKUMENTACIO.md](FEJLESZTOI_DOKUMENTACIO.md)** fájlban található.
