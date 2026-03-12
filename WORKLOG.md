# GymLog — Worklog & Roadmap

> Průběžný záznam vývoje prototypu + plán dalších kroků

---

## Worklog

### Session 1 — Inicializace prototypu
**Co bylo zadáno:** Postav funkční prototyp z PRD GymLog v1.1

**Co bylo rozhodnuto:**
- Architektura jako jediný `index.html` bez build kroku — nejrychlejší cesta k funkčnímu výsledku
- React 18 přes CDN + Babel Standalone (JSX v prohlížeči) místo Vite/CRA — nulový setup
- Tailwind CSS přes CDN
- Vlastní SVG liniový graf místo Recharts — eliminuje další CDN závislost a možné problémy s UMD bundlem
- localStorage jako úložiště — odpovídá MVP rozsahu PRD
- Klientský router přes `useState` (4 pohledy: dashboard / new-workout / workout-detail / exercise-detail)

**Co bylo postaveno:**
- Dashboard se statistickými kartami (týden / měsíc), miniaturním měsíčním kalendářem a seznamem tréninků
- Nový trénink: autocomplete z 40 cviků, přidávání sérií (váha + opakování), validace vstupů
- Detail tréninku: souhrn, přehled sérií, smazání s potvrzením
- Detail cviku: history table s PR odznakem, SVG liniový graf s přepínačem rozsahu (vše / 30d / 90d)
- Prázdné stavy pro dashboard i detail cviku
- Vlastní cviky — ukládají se do localStorage, příště se nabídnou v autocomplete

**Problémy a řešení:**
- _Recharts přes CDN_ — potenciálně nespolehlivý UMD import v kombinaci s Babel Standalone → řešení: vlastní SVG graf (~50 řádků), plně pod kontrolou, bez závislosti
- _Detekce kardio vs. silový cvik_ — původně jen přes `ex.cardio` z knihovny, ale vlastní cviky nemají flag → řešení: detekce ze samotných dat v uložených sériích (přítomnost `time != null`)

---

### Session 2 — Kardio rozšíření
**Co bylo zadáno:** Přidat kardio sledování — čas, kcal, a u běhu také kilometry

**Co bylo rozhodnuto:**
- Tři typy cviků: silový (váha + reps) / kardio (čas + kcal) / běh (čas + km + kcal)
- Běh detekován podle názvu (`includes('běh')` nebo `includes('run')`) — jednodušší než přidávat nový flag do datového modelu
- Grid layout 12 sloupců zůstal, pro běh se přerozdělil na 3+3+4 sloupce
- PR v historii cviku přepnut: silové = max váha, kardio = nejdelší čas, běh = největší vzdálenost
- Graf pro každý typ zobrazuje jinou metriku — jednotka osy Y dynamická (kg / min / km)
- PRD aktualizováno (sekce 3.1 a datový model sekce 6)

**Problémy a řešení:**
- _Detekce kardio v WorkoutDetail_ — komponenta nemá `allExercises` prop → řešení: detekce z dat v sérii (`set.time != null` = kardio, `set.distance != null` = běh), robustnější než lookup v knihovně
- _Hoisting `isCardio`/`isRunning` funkcí_ — definovány až za `validate()` která je volá → v pořádku, protože `validate` je closure volaná až na klik uživatele, nikoli při renderu; v té chvíli jsou všechny `const` v komponentě již přiřazeny

---

### Session 3 — Kompaktní kalendář + PWA
**Co bylo zadáno:** Zmenšit kalendář na 1/3, převést aplikaci do PWA

**Co bylo rozhodnuto:**
- Nahradit celý měsíční mřížek (≈20 řádků JSX, variabilní výška) jediným týdenním pruhem (7 buněk, pevná výška)
- Týdenní pruh: dnešek = tmavě indigo, tréninkový den = světle indigo, ostatní = šedá — zachovává informaci bez zbytečné plochy
- PWA bez build kroku: `manifest.json` + `sw.js` + meta tagy v `<head>`
- Service worker strategie: local-first pro vlastní soubory (cache → network), network-first pro CDN (network → cache jako fallback) — CDN soubory se průběžně cachují po prvním načtení

**Problémy a řešení:**
- _Node.js není nainstalován_ → Netlify CLI nelze použít; řešení: ruční propojení přes Netlify webové rozhraní (GitHub integration = automatický deploy při každém push)
- _PWA ikona_ — bitmapové PNG by vyžadovalo generátor; řešení: SVG ikona (podporováno moderními prohlížeči přes `"sizes": "any"`)

---

## Roadmap — Co dělat dál

Tasky jsou seřazeny od nejhodnotnějšího / nejjednoduššího po komplexnější.

---

### Priorita 1 — Rychlé výhry (hodiny práce, velký dopad)

#### TASK-01: Editace série v uloženém tréninku
**Popis:** Aktuálně lze sérii v uloženém tréninku pouze smazat, ne upravit. Přidat inline editaci (klik na sérii → editable inputs inline).
**Kde:** `WorkoutDetail` komponenta, sekce sérií
**Dopad:** Opraví nejčastější user pain point (překlep při zápisu váhy)

#### TASK-02: Kopírovat poslední trénink jako šablonu
**Popis:** Na obrazovce "Nový trénink" přidat tlačítko "Zkopírovat předchozí" — načte cviky z posledního tréninku se stejnými cviky jako startovní bod.
**Kde:** `NewWorkout`, Dashboard
**Dopad:** Ušetří čas při opakovaných tréninkových dnech

#### TASK-03: Čas trvání tréninku
**Popis:** Zaznamenat čas spuštění (při kliknutí "Nový trénink") a čas uložení, zobrazit délku tréninku v detailu.
**Kde:** Datový model + `WorkoutDetail`
**Dopad:** Motivační statistika, nulová extra práce od uživatele

#### TASK-04: Poznámka k tréninku / sérii
**Popis:** Volitelné textové pole na konci tréninku (celková poznámka) a/nebo u každé série.
**Kde:** `NewWorkout`, datový model
**Dopad:** Kontext k datům ("unavená, málo spánku"), důležité pro progress tracking

#### TASK-05: Swipe to delete na mobilu
**Popis:** Na mobilním prohlížeči přidat touch swipe-left gesto pro smazání série nebo cviku.
**Kde:** Komponenty se séries v `NewWorkout` a `WorkoutDetail`
**Dopad:** Výrazně lepší UX na telefonu

---

### Priorita 2 — Zlepšení UX a datové kvality

#### TASK-06: Přednaplnění váhy z posledního tréninku
**Popis:** Když uživatel přidá cvik, automaticky přednastavit váhu a reps z poslední zaznamenané série daného cviku.
**Kde:** `addSet` + lookup přes `workouts` v localStorage
**Dopad:** Výrazně rychlejší zápis — typicky se trénuje se stejnou nebo lehce vyšší váhou

#### TASK-07: Pořadí sérií drag & drop
**Popis:** Umožnit přeuspořádání cviků v probíhajícím tréninku přetažením.
**Kde:** `NewWorkout`
**Dopad:** Flexibilita při změně plánu v průběhu tréninku

#### TASK-08: Filtr a vyhledávání v historii tréninků
**Popis:** Searchbar na Dashboardu pro filtrování tréninků podle data nebo názvu cviku.
**Kde:** `Dashboard`
**Dopad:** Použitelnost roste s počtem tréninků

#### TASK-09: Stránka "Cviky" — přehled všech cviků s poslední aktivitou
**Popis:** Seznam všech cviků (z knihovny + vlastní) s datumem posledního záznamu a aktuální PR — rozcestník do detailu cviku.
**Kde:** Nový view `ExerciseLibrary`, navigace
**Dopad:** Usnadní procházení pokroku bez nutnosti otevírat konkrétní tréninky

#### TASK-10: Export dat (CSV)
**Popis:** Tlačítko "Exportovat data" stáhne CSV se všemi tréninky. Ochrana dat uživatele a záloha před výmazem localStorage.
**Kde:** Dashboard nebo Settings stránka
**Dopad:** Důvěra uživatele, bezpečnost dat

---

### Priorita 3 — Technická modernizace

#### TASK-11: Migrovat na Vite + React (build krok)
**Popis:** Přejít z CDN + Babel Standalone na standardní Vite projekt. Výsledek: rychlejší načítání (tree-shaking, minifikace), TypeScript podpora, Hot Module Replacement při vývoji.
**Effort:** ~2h migrace, žádná změna funkcionality
**Dopad:** Produkční výkon, lepší developer experience

#### TASK-12: Přejít na IndexedDB
**Popis:** Nahradit localStorage za IndexedDB (přes knihovnu `idb`). Odstraní ~5MB limit, umožní ukládání více dat (obrázky, přílohy v budoucnu).
**Effort:** ~4h, nutná migrace stávajících dat
**Dopad:** Škálovatelnost pro aktivní uživatele (denní tréninky × roky)

#### TASK-13: Přidat bottom navigation bar
**Popis:** Pevná navigace dole na mobilu: Dashboard | Cviky | Nový trénink. Nahradí back buttons a zlepší orientaci.
**Kde:** `App` (router)
**Dopad:** Standardní mobilní UX pattern

---

### Priorita 4 — Větší funkce (vyžadují backend)

#### TASK-14: Uživatelský účet + cloud synchronizace
**Popis:** Přihlášení (Google OAuth nebo email/heslo), data synchronizovaná přes Supabase. Přístup ze všech zařízení.
**Stack:** Supabase (PostgreSQL + Auth), přihlašovací flow
**Effort:** ~2–3 dny implementace

#### TASK-15: Sdílení PR a tréninku
**Popis:** Generovat share link nebo obrázek pro sdílení na sociálních sítích ("Dnes jsem zvedla PR: 100 kg Bench Press!").
**Effort:** ~1 den

#### TASK-16: Šablony tréninků
**Popis:** Uložit trénink jako šablonu, spustit nový trénink z šablony s přednastavenými cviky a váhami.
**Effort:** ~1 den (pokud jsou data v localStorage)

#### TASK-17: Push notifikace / připomínky
**Popis:** "Dnes jsi ještě netrénovala" nebo "Uplynuly 3 dny od posledního tréninku."
**Stack:** Web Push API, vyžaduje backend pro odesílání
**Effort:** ~1 den + backend

#### TASK-18: Dark mode
**Popis:** Tmavý režim podle systémového nastavení (`prefers-color-scheme`) nebo manuální přepínač.
**Effort:** ~4h s Tailwind dark: variantami

---

## Technický dluh

| # | Problém | Závažnost | Kdy řešit |
|---|---|---|---|
| TD-01 | Babel Standalone transpiluje JSX v prohlížeči — zpomaluje první načtení (~500ms navíc) | Střední | Při migraci na Vite (TASK-11) |
| TD-02 | Žádné testy — jakákoli změna může rozbít existující funkce | Střední | Po migraci na Vite, přidat Vitest |
| TD-03 | Detekce "Běh" je string matching (`includes('běh')`) — fragile | Nízká | Přidat `isRunning` flag do datového modelu cviku |
| TD-04 | Validace kardio sérií nevyžaduje vyplnění alespoň jednoho pole | Nízká | Přidat cross-field validaci |
| TD-05 | localStorage data nemají verzování schématu — migrace při změně modelu | Střední | Přidat `schema_version` klíč a migration logic |
