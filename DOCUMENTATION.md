# GymLog — Dokumentace prototypu

> Verze dokumentu: 1.3 · Březen 2026

---

## 1. Co aplikace dělá

GymLog je webová aplikace pro sledování tréninků v posilovně. Umožňuje rychlý zápis cviků, sérií, vah a opakování přímo ve fitku, zobrazuje historii výkonů a osobní rekordy.

Data jsou uložena v cloudu (Supabase) — přežijí reset telefonu, smazání prohlížeče i přechod na nové zařízení. Přihlášení probíhá bez hesla přes magic link na email. Aplikace je nasazena na Netlify a dostupná z iPhone, Mac i Windows.

---

## 2. Architektura

### Typ aplikace
Single-page application (SPA) jako jediný HTML soubor — bez build procesu, bez závislostí, které by bylo nutné instalovat.

### Technologie

| Vrstva | Technologie | Způsob načtení |
|---|---|---|
| UI framework | React 18 | CDN (unpkg.com) |
| JSX transpilace | Babel Standalone | CDN (unpkg.com) |
| Stylování | Tailwind CSS | CDN (cdn.tailwindcss.com) |
| Grafy | Vlastní SVG (inline) | — |
| Autentizace | Supabase Auth (magic link) | CDN (jsdelivr.net) |
| Databáze | Supabase (PostgreSQL) | API (supabase.co) |
| Offline podpora | Service Worker | sw.js |

### Soubory projektu

```
my-first-prototype/
├── index.html        # Celá aplikace (React komponenty, logika, styly)
├── manifest.json     # PWA manifest (název, ikona, barvy)
├── icon.svg          # Aplikační ikona (činka na indigo pozadí)
├── sw.js             # Service Worker (offline cache)
├── GymLog_PRD_v1.1.md  # Product Requirements Document
├── DOCUMENTATION.md  # Tento soubor
└── WORKLOG.md        # Průběh vývoje a plán dalšího rozvoje
```

### Nasazení
- **GitHub:** `github.com/kgrollova/my-first-prototype` — zdrojový kód
- **Netlify:** `https://aesthetic-bubblegum-3bb722.netlify.app` — produkce (auto-deploy při každém push na `main`)
- **Supabase projekt:** `wzxbrguwzlgwjhnqhixi.supabase.co` — databáze + auth

### Proč jeden HTML soubor?
Cílem byl rychlý funkční prototyp ("vibecoding"). Jeden soubor znamená nulový setup, okamžité spuštění dvojklikem, snadné sdílení a nasazení kamkoli.

---

## 3. Datový model

Data jsou uložena v Supabase (PostgreSQL) se zapnutým Row Level Security — každý uživatel vidí výhradně svá vlastní data.

### Supabase tabulky

#### `workouts`
| Sloupec | Typ | Popis |
|---|---|---|
| `id` | `text` (PK) | UUID generovaný v klientu |
| `user_id` | `uuid` | Vazba na `auth.users` |
| `date` | `timestamptz` | Datum a čas tréninku (ISO 8601) |
| `name` | `text \| null` | Volitelný název tréninku |
| `exercises` | `jsonb` | Pole cviků se sériemi (viz níže) |

#### `custom_exercises`
| Sloupec | Typ | Popis |
|---|---|---|
| `user_id` | `uuid` (PK) | Vazba na `auth.users` |
| `names` | `text[]` | Pole názvů vlastních cviků |

### Struktura `exercises` (JSONB)

### `gymlog_workouts` — pole tréninků

```json
[
  {
    "id": "abc123",
    "date": "2026-03-12T10:30:00.000Z",
    "name": "Leg day",
    "exercises": [
      {
        "id": "def456",
        "name": "Dřep",
        "order": 0,
        "sets": [
          {
            "id": "ghi789",
            "weight": 100,
            "reps": 5,
            "time": null,
            "kcal": null,
            "distance": null,
            "order": 0
          }
        ]
      }
    ]
  }
]
```

### Datové typy polí série (Set)

| Pole | Typ | Použití |
|---|---|---|
| `weight` | `float \| null` | Silové cviky — váha v kg (0–999,5, krok 0,5) |
| `reps` | `integer \| null` | Silové cviky — počet opakování (1–999) |
| `time` | `float \| null` | Kardio — čas v minutách |
| `kcal` | `integer \| null` | Kardio — spálené kalorie |
| `distance` | `float \| null` | Běh — vzdálenost v km |

---

## 4. Funkce aplikace

### 4.1 Dashboard
- Statistické karty: počet tréninků, sérií a kg v **aktuálním týdnu** a **aktuálním měsíci**
- **Týdenní pruh** (Po–Ne): zobrazuje aktuální týden, zvýrazní dnešek a dny s tréninkem
- Chronologický seznam tréninků (nejnovější nahoře)
- Prázdný stav s výzvou k prvnímu tréninku

### 4.2 Nový trénink
- Volitelný název tréninku
- Datum a čas se zaznamenají automaticky
- Přidávání libovolného počtu cviků
- **Autocomplete** z knihovny 40 cviků — hledá v názvu, zobrazuje kategorii
- Možnost přidat vlastní cvik (uloží se do knihovny pro příští použití)
- **Silové cviky**: váha (kg) + opakování — validace rozsahů
- **Kardio cviky**: čas (min) + kcal — 2 pole
- **Běh** (detekováno podle názvu): čas (min) + km + kcal — 3 pole
- Mazání jednotlivých sérií i celých cviků
- Validace všech vstupů s chybovými hláškami

### 4.3 Detail tréninku
- Souhrnné karty: počet cviků, sérií, celkem kg
- Přehled všech cviků a jejich sérií
- Zobrazení adaptované na typ cviku (silový / kardio / běh)
- Kliknutí na název cviku → přechod na historii daného cviku
- **Smazání série** — křížek (×) na konci každého řádku série
- **Smazání cviku** — ikona koše v hlavičce cviku; odstraní cvik i se všemi sériemi
- Smazání celého tréninku s potvrzovacím dialogem
- Všechny změny se okamžitě synchronizují do Supabase

### 4.4 Detail cviku (progrese)
- Přepínač časového rozsahu: Vše / 30 dní / 90 dní
- **Liniový SVG graf** vývoje výkonu:
  - Silové cviky → maximální váha (kg)
  - Kardio → celkový čas (min)
  - Běh → celková vzdálenost (km)
- Osobní rekord (PR) zobrazen v hlavičce a zvýrazněn v grafu
- **Tabulka** historických výkonů s PR odznakem
  - Silové: datum / max kg / průměr / celkem reps
  - Kardio: datum / čas / kcal
  - Běh: datum / čas / km / kcal

### 4.5 Knihovna cviků (40 přednastavených)

| Kategorie | Cviky |
|---|---|
| Hruď | Bench Press, Incline Bench Press, Kabelové překřížení, Dips, Push-up |
| Záda | Mrtvý tah, Přítahy na hrazdě, Veslování, Lat Pulldown, Seated Row |
| Nohy | Dřep, Leg Press, Rumunský mrtvý tah, Leg Curl, Leg Extension, Výpady |
| Ramena | Overhead Press, Boční zdvihy, Přední zdvihy, Face Pull, Arnold Press |
| Biceps | Bicepsový zdvih s činkou, Kladivový zdvih, Koncentrický zdvih, Preacher Curl |
| Triceps | Tricepsový pushdown, Skull Crusher, Tricepsové kliky, French Press |
| Břicho | Plank, Crunch, Leg Raise, Ab Wheel, Russian Twist |
| Kardio | Běh, Veslařský trenažér, Spinning, Eliptický trenažér, Skipping, Chůze |

---

## 5. Autentizace (Supabase Auth)

### Přihlašovací flow (OTP kód)
1. Uživatel zadá email na přihlašovací obrazovce (email se předvyplní z předchozí session)
2. Supabase pošle email s 6místným jednorázovým kódem (OTP)
3. Uživatel zadá kód přímo v aplikaci — aplikace zavolá `verifyOtp()`
4. Supabase ověří kód, vytvoří session a spustí `onAuthStateChange` event
5. Aplikace načte data z cloudu a zobrazí dashboard

> **Proč OTP místo magic linku:** Na iOS PWA by klik na odkaz v emailu otevřel Safari (oddělená paměť od PWA) a uživatel by byl přihlášen v prohlížeči, ne v nainstalované aplikaci. OTP kód uživatel opíše přímo v PWA bez opuštění aplikace.

### Persistence OTP kroku na iOS
- Po odeslání emailu se `step='otp'` a `email` uloží do `sessionStorage`
- iOS PWA může být při přepnutí do emailové aplikace restartována — `sessionStorage` přežije
- Po zadání správného kódu se `sessionStorage` automaticky vyčistí
- E-mail se navíc ukládá do `localStorage` (`gymlog_last_email`) pro předvyplnění při příštím přihlášení

### Správa session
- Session je uložena v `localStorage` pod klíčem Supabase (`sb-*`) — přežije refresh stránky i restart PWA
- `onAuthStateChange` poslouchá změny (přihlášení, odhlášení, expiraci)
- Odhlášení: tlačítko v pravém horním rohu dashboardu (kroužek s iniciálou emailu)

### Migrace z localStorage
Při prvním přihlášení aplikace detekuje existující data v `localStorage`. Zobrazí dialog s možností:
- **Přenést do cloudu** — data se importují do Supabase, localStorage se vyčistí
- **Zahodit** — localStorage se vyčistí, začne se s prázdnou databází

### Supabase konfigurace
- **Site URL:** `https://aesthetic-bubblegum-3bb722.netlify.app`
- **Redirect URL:** `https://aesthetic-bubblegum-3bb722.netlify.app/**`
- Nastavení: Supabase → Authentication → URL Configuration

---

## 6. PWA (Progressive Web App)

Aplikace je PWA-ready:

- **Instalovatelná** na telefon nebo desktop (tlačítko "Přidat na plochu")
- **Offline funkce** — po prvním načtení funguje bez internetu
- Service worker cachuje: `index.html`, `manifest.json`, `icon.svg`
- CDN skripty (React, Tailwind) se průběžně cachují po prvním načtení

> Service worker funguje pouze přes HTTPS nebo `localhost`.

---

## 7. Jak spustit

### Lokálně (rychlé)
Dvojklik na `index.html` — otevře se v prohlížeči. Data se uloží do localStorage.

> PWA funkce (offline, instalace) vyžadují HTTP server, ne přímé otevření souboru.

### Lokálně s HTTP serverem (pro PWA)
Pokud máš Python:
```bash
python -m http.server 8080
# otevři http://localhost:8080
```

### Produkčně
Nasazeno na Netlify — každý `git push` na `main` se automaticky promítne.

---

## 8. SMTP a emailové limity

Supabase free tier omezuje odesílání emailů na ~2–3 za hodinu. Pro produkční použití je doporučeno napojit vlastní SMTP.

### Nastavení vlastního SMTP (Resend.com — doporučeno)

1. Vytvoř účet na `resend.com` (free: 100 emailů/den, 3 000/měsíc)
2. V Resend dashboardu: **API Keys → Create API Key** → zkopíruj klíč
3. V Supabase: **Project Settings → Authentication → SMTP Settings → Enable Custom SMTP**
4. Vyplň:
   - **Host:** `smtp.resend.com`
   - **Port:** `465`
   - **Username:** `resend`
   - **Password:** [Resend API klíč]
   - **Sender email:** `gymlog@resend.dev` (nebo vlastní doména)
   - **Sender name:** `GymLog`
5. Uložit — od teď posílá Resend, žádný hodinový limit

---

## 9. Omezení prototypu

| Oblast | Omezení | Řešení v budoucnu |
|---|---|---|
| Úložiště | Supabase free tier: 500 MB — při extrémním objemu dat zvážit upgrade | Supabase Pro plán |
| Editace | Nelze editovat sérii v uloženém tréninku (jen smazat) | Formulář pro editaci série (TASK-01) |
| Synchronizace | ✅ Vyřešeno — Supabase cloud sync napříč zařízeními | — |
| Bezpečnost | ✅ OTP auth + Row Level Security | — |
| Výkon | Babel Standalone transpiluje JSX v prohlížeči — pomalejší první načtení | Build krok (Vite + React, TASK-11) |
| Offline | Nové tréninky nelze uložit bez připojení (Supabase vyžaduje internet) | Offline queue + sync při obnovení spojení |
| Email limity | Supabase free tier ~2–3 emaily/hodinu | Vlastní SMTP přes Resend.com (viz sekce 8) |
