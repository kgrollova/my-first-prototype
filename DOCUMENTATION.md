# GymLog — Dokumentace prototypu

> Verze dokumentu: 1.0 · Březen 2026

---

## 1. Co aplikace dělá

GymLog je webová aplikace pro sledování tréninků v posilovně. Umožňuje rychlý zápis cviků, sérií, vah a opakování přímo ve fitku, zobrazuje historii výkonů a osobní rekordy.

Aplikace běží celá v prohlížeči — nepotřebuje server, backend ani připojení k internetu (po prvním načtení).

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
| Ukládání dat | localStorage | Prohlížeč |
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

### Proč jeden HTML soubor?
Cílem byl rychlý funkční prototyp ("vibecoding"). Jeden soubor znamená nulový setup, okamžité spuštění dvojklikem, snadné sdílení a nasazení kamkoli.

---

## 3. Datový model

Data jsou uložena v `localStorage` ve dvou klíčích:

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

### `gymlog_custom_exercises` — pole vlastních cviků

```json
["Můj vlastní cvik", "Kettlebell swing"]
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
- Smazání celého tréninku s potvrzovacím dialogem

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

## 5. PWA (Progressive Web App)

Aplikace je PWA-ready:

- **Instalovatelná** na telefon nebo desktop (tlačítko "Přidat na plochu")
- **Offline funkce** — po prvním načtení funguje bez internetu
- Service worker cachuje: `index.html`, `manifest.json`, `icon.svg`
- CDN skripty (React, Tailwind) se průběžně cachují po prvním načtení

> Service worker funguje pouze přes HTTPS nebo `localhost`.

---

## 6. Jak spustit

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

## 7. Omezení prototypu

| Oblast | Omezení | Řešení v budoucnu |
|---|---|---|
| Úložiště | localStorage ~5 MB, při stovkách tréninků může dojít místo | IndexedDB nebo cloud (Supabase) |
| Editace | Nelze editovat sérii v uloženém tréninku (jen smazat) | Formulář pro editaci série |
| Synchronizace | Data jsou pouze lokální, vázaná na prohlížeč | Uživatelský účet + cloud sync |
| Bezpečnost | Žádná autentizace — data vidí každý, kdo má přístup k zařízení | Přihlášení + šifrování |
| Výkon | Babel Standalone transpiluje JSX v prohlížeči — pomalejší první načtení | Build krok (Vite + React) |
