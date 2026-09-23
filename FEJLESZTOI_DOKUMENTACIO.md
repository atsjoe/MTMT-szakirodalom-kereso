# MTMT Szakirodalom Kereső - Fejlesztői Dokumentáció
**Verzió:** 2.3 (Robust Journal & Relevancy Engine + Elevated Search Bar Edition)  
**Dátum:** 2026  
**Architektúra:** Kliens-oldali (Client-side) Single Page Application (SPA) vanilla HTML/CSS/JS alapokon  

---

## 1. Projekt Architektúra és Fájlszerkezet

A rendszer teljes mértékben a böngészőben (kliens oldalon) fut, háttérszerver (Backend) és relációs adatbázis nélkül. Az adatokat közvetlenül a publikus MTMT REST API-tól (`https://m2.mtmt.hu/api/`) és az OpenAlex API-tól kéri le.

### 1.1. Architektúra és Technológiák részletezése
*   **Környezet:** Kliensoldali webalkalmazás (Serverless / Static SPA). Minden adat a böngésző memóriájában (`localStorage` a szótárnak, DOM és JS memória a találatoknak) él.
*   **Nyelv:** Vanilla JavaScript (ES6+ aszinkron funkciókkal, párhuzamos Promise kezeléssel), HTML5, CSS3.
*   **UI/UX Minta:** SPA felület CSS Grid alapú elrendezéssel (szűrő oszlop és találati oszlop). Az előszűrőkben kijelölt entitások (szerző, folyóirat, intézmény, év, szerep) interaktív tokenként emelődnek be a fő keresősávba.
*   **Külső API-k:**
    *   *MTMT v2 API:* Publikációs listák, szerzők, folyóiratok és intézmények lekérése aszinkron lapozással és komplex feltételrendszerrel (`cond` operátorok).
    *   *OpenAlex API:* Művek (works) kötegelt lekérdezése DOI, OpenAlex ID és `cites` (ko-citáció) szűrők alapján a hálózati kutatáshoz.

### 1.2. Ajánlott Fájlszerkezet
* `index.html`: A fő alkalmazás, tartalmazza a teljes UI-t, a keresőmotorokat, az előszűrőket és a formázó logikát.
* `style.css`: Az alkalmazás egységes vizuális megjelenése (UI/UX, modális ablakok, beemelt tokenek).
* `szotar.html`: Önálló mikroszolgáltatás a `localStorage` alapú saját tezaurusz kezelésére (szöveges import/export képességgel).
* `sugo.html`: Részletes felhasználói dokumentáció.
* `mapping_mtmt_api.json`: Az MTMT v2 API objektummodell mezőleírása.
* `mapping_openalex_api.json`: Az OpenAlex API objektummodell mezőleírása.
* `mapping_export_formats.json`: A bibliográfiai kimenetek (RIS, BibTeX, CSL-JSON) mezőmegfeleltetései.

---

## 2. Keresőmotorok (Search Engines)

### 2.1. Egyszerű kereső (`okosKeresoFeldolgozo`)
* **Logika:** String parsing regex segítségével (`/(-?"[^"]+")|(-?[^\s]+)/g`).
* **Működés:** Szétválasztja a kötelező (tartalmazza) és a tiltott (kizarja) elemeket. A szóközöket implicit AND kapcsolatként kezeli, az idézőjeleket Exact Phrase-ként (pontos kifejezés).

### 2.2. Szisztematikus SLR Kereső (`generaljKombinaciokat`)
* **Logika:** Kliens-oldali Descartes-szorzat (Cartesian product) generálás.
* **Probléma:** Az MTMT API `cond` paramétere nem ismeri a komplex logikai fák (Zárójelek, vegyes AND/OR) fogalmát.
* **Megoldás:** A JS a blokkokban (sorokban) lévő szinonimákból (OR) és a sorok közötti feltételekből (AND) legenerálja az összes lehetséges logikai útvonalat, majd ezeket *párhuzamos aszinkron kérésekkel* kilövi a szerverre. A válaszokat egy közös `Set` egyesíti az `mtid` alapján, kiküszöbölve a redundanciát.

---

## 3. MTMT API Integrációs Megoldások (Workarounds & Hacks)

Az MTMT Java-alapú szervere bizonyos típusú lekérdezéseknél (pl. többszavas, szóközös keresések) hajlamos összeomlani, vagy "büntetésből" fals adatokat visszaadni. A szoftver az alábbi architektúrákkal küszöböli ki ezeket:

### 3.1. Kettős Lekérdezési Stratégia (Dual-Query) és Kliens-oldali Védelem
A Modal ablakos Intelligens entitás-keresőben (Szerző/Folyóirat/Intézmény megadása) egy speciális logika dolgozik:
* **Feldarabolt részleges keresés (AND logika):** A beírt stringet szóközök mentén daraboljuk, és minden szót külön-külön `cond=label;any` operátorként adunk át. Ez stabil, nem akasztja ki a szervert.
* **Mágikus Exact Match (Folyóiratoknál):** Mivel a darabolás néha túl tág találatot ad, a folyóiratok (`journal` és `series`) lekérdezésekor egy második, aszinkron kérés is elindul a `cond=title;eq` operátorral, amely egyben hagyja a kifejezést. Ez garantálja, hogy a pontos címek (pl. "Magyar Tudomány") is leérkezzenek.
* **Szigorú Kliens-oldali ÉS-szűrő:** A szerverről letöltött halmazt JavaScriptben egy szigorú ciklussal utószűrjük, amely eldobja az összes olyan találatot, amiben nem szerepel *minden* beírt keresőszó.

### 3.2. A "Bulldózer" Lapozás (`fetchAllPages`)
* **Probléma:** Bizonyos API kondíciók esetén az MTMT szervere figyelmen kívül hagyja a `size=2000` paraméterünket, és önkényesen az alapértelmezett 20 tételt adja vissza.
* **Megoldás:** A `fetchAllPages` függvény az első letöltés (`page=1`) után ellenőrzi a szerver válaszában a `paging.totalPages` paramétert. Ha ez nagyobb mint 1, a szoftver aszinkron módon, párhuzamos Promise hívásokkal letölti az összes hátralévő oldalt (limitálva max 50 oldalra / 1000 elemre), kikényszerítve a hiánytalan adatkészletet.

### 3.3. Cache-Buster Mechanizmus
* **Probléma:** A böngészők memóriája hajlamos a korábbi, megegyező URL-ű szerver-válaszokat azonnal, hálózati kérés nélkül visszadobni.
* **Megoldás:** Minden Modal-alapú keresés generál egy időbélyeget: `let cacheBuster = Date.now();`. Ezt az `&_t=` paraméterbe fűzve garantáljuk az egyedi HTTP GET kérést.

### 3.4. Relevancia Motor (`calcScore`) és DOM-védelem
* **ISSN-levágó Hack:** Az MTMT a folyóiratok nevét mindig hozzáfűzött ISSN számmal adja vissza. A relevancia motor egy Regex kifejezéssel (`/\s?[0-9]{4}-[0-9]{3}[0-9xX]/g`) levágja ezt a pontozás előtt, így az egyezés 100%-os tud lenni.
* **Pontozás:** 100 pont (Exact Match) -> 80 pont (Ezzel kezdődik) -> 60 pont (Egybefüggően szerepel bárhol) -> 10 pont (Szétesve szerepel). Azonos pontszám esetén `localeCompare('hu')` ABC-rendezés lép életbe.
* **DOM-védelem (Anti-Freeze):** A böngésző memóriájának védelme érdekében a rendszer rendezi a teljes listát, de **csak a legrelevánsabb 500 elemet** rendereli ki a Modal ablakba.

---

## 4. Dinamikus Szűrés és Keresősáv Integráció

### 4.1. Előszűrők (Pre-filters) és Keresősáv Tokenek
A felület bal felső sarkában lévő Előszűrők adatai automatikusan beemelődnek a keresősávba tokenként (szerző, folyóirat, intézmény, év, közlemény szerepe), vizuálisan megjelenítve az **`ÉS`** kapcsolatokat. A fetch kérés előtt ezek a paraméterek bekerülnek az URL-be (`cond=authorships.author;eq;...`, `publishedYear;ge;...`, stb.).

### 4.2. Utószűrők (Post-filters)
A letöltött halmazon a kliens JavaScript végzi el az azonnali szűrést:
* **Szigorú szókereső (`pontosKereses`):** A regex kifejezés (`(^|[^a-záéíóöőúüűA-ZÁÉÍÓÖŐÚÜŰ0-9])`) biztosítja a szóhatárok betartását.
* **Közlemény szerepe (`getPubAllapot`):** A `pub.core` és `pub.citation` boolean értékekből állítja elő a státuszt.

---

## 5. Adatperzisztencia, Szótár és Újdonságfigyelés
* **Munkamenet mentése:** A teljes GUI állapotát JSON formátumba csomagolja, és kliens oldali Blob-ként menti le (`.json`).
* **Újdonságfigyelés (Alerting):** A mentett JSON betöltésekor a rendszer összeveti a mentés dátumát a publikációk `created` mezőjével, lehetővé téve a csak új rekordok szűrését.
* **Saját Szótár:** A `szotar.html` a böngésző `localStorage` (kulcs: `mtmtSajatSzotar`) API-ját használja TXT import/export képességgel.

---

## 6. Export Modul (Citation Formatter & InternalId Hack)
* `HTML`: APA, IEEE, MLA és Chicago formázás.
* `BibTeX`: Standard címvédő kapcsos zárójelek `{{...}}` a kis/nagybetűk megőrzésére.
* `RIS / BibTeX InternalId Injekció`: Ha egy folyóiratcikknek nincsenek oldalszámai, az iparági hack szerint az MTMT `internalId` értékét a Start Page (`SP`), `M1` és `artno` mezőkbe injektálja, megvédve a hivatkozások épségét a Zotero/Mendeley referenciakezelőkben.

---

## 7. Hálózatkutató Modul (OpenAlex API Integráció)
*   **Adatforrás:** OpenAlex REST API (`https://api.openalex.org/works`).
*   **Híd a rendszerek között:** DOI (`identifiers` tömbből) és OpenAlex Work ID alapján.
*   **Bibliográfiai Csatolás:** Rokon cikkek felkutatása az irodalomjegyzékek metszete alapján (`related_works`).
*   **Ko-citációs Algoritmus & Rate Limit Védelem:** Az OpenAlex `429 Too Many Requests` hibájának kivédésére egyetlen kötegelt hívás indul (`filter=cites:W1|W2|...`).
*   **Hálózati Tálca & Adatfúzió:** Az OpenAlex elemek negatív mű-azonosítót (`oaFakeIdCounter`) kapnak, normalizálódnak az MTMT sémára, és a `+ Hozzáadás` gombbal a fő találatok közé emelhetők.
