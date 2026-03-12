**GymLog**

Webová aplikace pro sledování tréninků

*Product Requirements Document · v1.1 · březen 2026*

**1. Přehled produktu**

GymLog je jednoduchá, přehledná webová aplikace určená pro jednotlivce,
kteří chtějí zaznamenávat a sledovat svůj pokrok v posilování. Aplikace
umožňuje rychlý zápis cviků, vah a opakování přímo ve fitku, a zobrazuje
přehledné tabulky a grafy pro sledování progrese v čase.

  --------------- ---------------------------------------------------------
  **Platforma**   Webová aplikace (prohlížeč -- desktop i mobilní)

  --------------- ---------------------------------------------------------

  -------------- ---------------------------------------------------------
  **Uživatel**   Jednotlivec cvičící v posilovně, bez nutnosti registrace
                 v MVP

  -------------- ---------------------------------------------------------

  ------------- ---------------------------------------------------------
  **Ukládání    Lokálně v prohlížeči (localStorage) v MVP; cloud
  dat**         synchronizace jako budoucí rozšíření

  ------------- ---------------------------------------------------------

  ------------- ---------------------------------------------------------
  **Design**    Světlý (light mode), minimalistický, estetický

  ------------- ---------------------------------------------------------

**2. Cíle a úspěch produktu**

**Primární cíle**

-   Umožnit uživateli rychle a snadno zaznamenat trénink -- cviky, počet
    opakování a váhu

-   Zobrazit přehledný přehled minulých tréninků a pokroku

-   Poskytnout napovídání základních cviků, zároveň umožnit vlastní
    cviky

-   Motivovat uživatele vizuální reprezentací zlepšení

**Metriky úspěchu**

-   Uživatel je schopen zaznamenat celý trénink do 3 minut

-   Dashboard zobrazuje osobní rekord a výkon z posledního tréninku bez
    nutnosti kliknutí

-   Aplikace funguje plynule i na mobilním prohlížeči (bez
    horizontálního scrollování, bez překryvu prvků)

**3. Funkční požadavky**

**3.1 Zápis tréninku**

-   Uživatel může spustit nový trénink jedním kliknutím

-   Ke každému tréninku se automaticky zaznamená datum a čas

-   V rámci tréninku lze přidávat neomezený počet cviků

-   Každý cvik obsahuje: název cviku, váhu (kg) na sérii a počet
    opakování (reps) na sérii

-   Uživatel může pro jeden cvik přidat více sérií s různou váhou a
    počtem opakování

-   Váha je desetinné číslo v rozsahu 0--999,5 kg (krok 0,5 kg); hodnota
    0 je povolena pro cviky bez zátěže

-   Počet opakování je celé číslo v rozsahu 1--999

-   U kardio cviků jsou pole váha a opakování nahrazena kardio poli:
    čas (min) a spálené kalorie (kcal) jsou dostupné pro všechny kardio
    cviky; u běhu je navíc dostupné pole vzdálenost (km)

**3.1a Editace a mazání**

-   Uživatel může kdykoli upravit nebo smazat jakoukoli sérii v rámci
    probíhajícího i uloženého tréninku

-   Uživatel může smazat celý cvik z tréninku včetně všech jeho sérií

-   Uživatel může smazat celý trénink; akce vyžaduje potvrzení dialogem

-   Po smazání tréninku se data nevratně odstraní z localStorage

**3.2 Napovídání cviků**

-   Při psaní názvu cviku se zobrazí autocomplete nabídka ze základní
    knihovny

-   Knihovna obsahuje minimálně 40 běžných cviků rozdělených do
    kategorií (hruď, záda, nohy, ramena, biceps, triceps, břicho,
    kardio)

-   Uživatel může kdykoli zadat vlastní název cviku, který není v
    knihovně

-   Vlastní cviky se uloží a příště se také zobrazí v nabídce

**3.3 Přehled tréninků (Dashboard)**

-   Hlavní stránka zobrazuje seznam všech zaznamenaných tréninků
    seřazených od nejnovějšího

-   U každého tréninku je vidět datum, počet cviků a celkový počet sérií

-   Kliknutím na trénink se zobrazí jeho detail se všemi cviky, sériemi
    a vahami

**3.4 Tabulka pokroku**

-   Pro každý cvik existuje stránka s historií výkonů

-   Tabulka zobrazuje: datum, maximální váhu, průměrnou váhu, celkový
    počet opakování

-   Zvýraznění osobního rekordu (PR) v tabulce

**3.5 Graf vývoje váhy**

-   Pro vybraný cvik se zobrazí liniový graf vývoje maximální váhy v
    čase (maximum = nejtěžší série daného dne, reprezentuje nejlepší
    výkon)

-   Graf obsahuje popisky os (datum, kg) a legendu

-   Uživatel může přepínat mezi zobrazením: vše / posledních 30 dní /
    posledních 90 dní

-   Pokud má cvik méně než 2 záznamy, graf se nezobrazí -- místo něj se
    ukáže výzva k přidání dalšího tréninku

**3.6 Týdenní a měsíční přehled**

-   Týden je definován jako pondělí--neděle (ISO týden); měsíc jako
    kalendářní měsíc

-   Statistická karta zobrazující: počet tréninků v aktuálním týdnu /
    měsíci

-   Celkový počet sérií a zdvihnutých kg v daném období

-   Přehledný kalendář s vyznačenými dny, kdy proběhl trénink

**3.7 Prázdné stavy (Empty States)**

-   Dashboard bez tréninků: zobrazí se uvítací zpráva s výzvou
    \'Zaznamenej svůj první trénink\' a tlačítko Začít

-   Detail cviku bez historie: zobrazí se text \'Zatím žádné záznamy --
    přidej cvik do tréninku\' místo tabulky a grafu

-   Prázdný výsledek vyhledávání v knihovně: text \'Cvik nenalezen --
    přidej vlastní\'

**4. Uživatelské příběhy (User Stories)**

  ----------------------------------------------------------------------------
  **ID**   **Jako uživatel chci\...**           **Aby\...**
  -------- ------------------------------------ ------------------------------
  US-01    spustit nový trénink jedním          ztratila minimum času na
           kliknutím                            začátku

  US-02    vidět napovídání cviků při psaní     nemusela cviky psát celé ručně

  US-03    přidat vlastní cvik který není v     nebyla omezena nabídkou
           knihovně                             aplikace

  US-04    zapsat více sérií u jednoho cviku s  přesně zachytila průběh
           různou váhou                         tréninku

  US-05    vidět historii konkrétního cviku v   sledovala svůj pokrok a osobní
           tabulce                              rekordy

  US-06    zobrazit graf vývoje váhy pro cvik   vizuálně viděla zlepšení v
                                                čase

  US-07    vidět kolik jsem trénovala tento     měla motivaci dodržovat plán
           týden/měsíc                          

  US-08    aplikaci pohodlně používat na mobilu nemusela nosit notebook
           v posilovně                          

  US-09    upravit nebo smazat sérii po uložení mohla opravit chybu při zápisu
           tréninku                             

  US-10    vidět prázdný dashboard s výzvou k   věděla jak začít a nebyla
           prvnímu tréninku                     zmatená
  ----------------------------------------------------------------------------

**5. Design a UX požadavky**

**5.1 Vizuální styl**

-   Light mode -- bílé pozadí, jemné šedé prvky

-   Primární barva: indigo/fialová (#4F46E5) pro akcenty, tlačítka,
    grafy

-   Čistá typografie -- jeden font (Inter nebo podobný), velký
    whitespace

-   Minimální ikony, bez přeplnění informacemi na jedné obrazovce

-   Karty (cards) jako základní UI prvek pro cviky a tréninky

**5.2 Klíčové obrazovky**

  -----------------------------------------------------------------------
  **Obrazovka**         **Obsah**
  --------------------- -------------------------------------------------
  Dashboard             Statistické karty (týden/měsíc), kalendář, seznam
                        tréninků

  Nový trénink          Postupné přidávání cviků a sérií, tlačítko
                        Dokončit trénink

  Detail tréninku       Přehled všech cviků, sérií, vah -- read-only s
                        možností editace

  Detail cviku          Tabulka historických výkonů + liniový graf vývoje
                        váhy

  Knihovna cviků        Seznam cviků dle kategorie, vlastní cviky,
                        vyhledávání
  -----------------------------------------------------------------------

**5.3 Responzivita**

-   Plná funkčnost na mobilním prohlížeči (min. 360px šíře)

-   Velká dotyková plocha tlačítek (min. 44px výška)

-   Formulář pro přidání série optimalizovaný pro mobilní klávesnici
    (numerické inputy)

**6. Datový model**

Základní datové entity aplikace:

  --------------------------------------------------------------------------
  **Entita**        **Atributy a typy**      **Validace / poznámky**
  ----------------- ------------------------ -------------------------------
  Workout (trénink) id: string (UUID) datum: Název je volitelný; datum se
                    ISO 8601 string název:   generuje automaticky
                    string \| null           

  WorkoutExercise   id: string (UUID)        Pořadí určuje zobrazení cviků v
                    workout_id: string       tréninku
                    exercise_id: string      
                    pořadí: integer          

  Set (série)       id: string (UUID)        Váha: 0--999,5 kg, krok 0,5;
                    workout_exercise_id:     null u kardio. Reps: 1--999;
                    string váha: float \|    null u kardio. Čas: kladné
                    null reps: integer \|    desetinné číslo (min). Kcal:
                    null čas: float \| null  kladné celé číslo. Vzdálenost:
                    kcal: integer \| null    kladné desetinné číslo (km),
                    vzdálenost: float \|     pouze u běhu. Alespoň jedno
                    null pořadí: integer     pole musí být vyplněno.

  Exercise (cvik)   id: string (UUID) název: Kategorie: hruď \| záda \| nohy
                    string kategorie: enum   \| ramena \| biceps \| triceps
                    cardio: boolean custom:  \| břicho \| kardio.
                    boolean                  Custom=true pro uživatelské
                                             cviky.
  --------------------------------------------------------------------------

*⚠️ Known limitation MVP: localStorage má limit \~5 MB. Při intenzivním
používání (každodenní tréninky, roky dat) může být limit překročen.
Doporučuje se přejít na IndexedDB nebo cloud při \>500 tréninzích.*

**7. Nefunkční požadavky**

-   Výkon: Stránka se načte do 2 sekund na běžném připojení

-   Dostupnost: Offline mód -- aplikace funguje i bez internetu (PWA
    ready)

-   Bezpečnost: V MVP jsou data pouze lokální; žádná citlivá data se
    neposílají na server

-   Kompatibilita: Chrome, Safari, Firefox -- poslední 2 verze

-   Přístupnost: Dostatečný kontrast barev, čitelné velikosti písma
    (min. 16px na mobilu)

**8. MVP -- rozsah první verze**

**Zahrnuto v MVP ✓**

-   Zápis tréninku: cviky, série, váha, opakování

-   Validace vstupů (rozsahy, povinná pole)

-   Editace a mazání sérií, cviků i celých tréninků

-   Autocomplete z knihovny 40+ cviků + vlastní cviky

-   Prázdné stavy (empty states) pro dashboard a detail cviku

-   Dashboard se seznamem tréninků

-   Detail cviku s tabulkou historických výkonů

-   Graf vývoje váhy pro cvik (max. váha)

-   Týdenní a měsíční statistiky (pondělí--neděle / kalendářní měsíc)

-   Responzivní design (mobil + desktop)

-   Ukládání dat do localStorage / IndexedDB

**Budoucí rozšíření (mimo MVP)**

-   Uživatelský účet a cloud synchronizace

-   Sdílení tréninků a přátelé

-   Šablony tréninků a plánování

-   Export dat (CSV, PDF)

-   Push notifikace / připomínky tréninku

-   Tmavý režim (dark mode)

**9. Základní knihovna cviků (ukázka)**

Aplikace bude předvyplněna nejběžnějšími cviky v kategoriích:

  -------------------------------------------------------------------------
  **Kategorie**   **Příklady cviků**
  --------------- ---------------------------------------------------------
  Hruď            Bench Press, Incline Bench Press, Kabelové překřížení,
                  Dips, Push-up

  Záda            Mrtvý tah, Přítahy na hrazdě, Veslování, Lat Pulldown,
                  Seated Row

  Nohy            Dřep, Leg Press, Rumunský mrtvý tah, Leg Curl, Leg
                  Extension, Výpady

  Ramena          Overhead Press, Boční zdvihy, Přední zdvihy, Face Pull,
                  Arnold Press

  Biceps          Bicepsový zdvih s činkou, Kladivový zdvih, Koncentrický
                  zdvih, Preacher Curl

  Triceps         Tricepsový pushdown, Skull Crusher, Tricepsové kliky,
                  French Press

  Břicho          Plank, Crunch, Leg Raise, Ab Wheel, Russian Twist

  Kardio          Běh, Veslařský trenažér, Spinning, Eliptický trenažér
  -------------------------------------------------------------------------

**10. Doporučený technický stack**

  -----------------------------------------------------------------------
  **Vrstva**            **Technologie**
  --------------------- -------------------------------------------------
  Frontend framework    React nebo Vue.js

  Styling               Tailwind CSS

  Grafy                 Recharts nebo Chart.js

  Ukládání dat (MVP)    localStorage / IndexedDB

  Hosting               Vercel nebo Netlify (zdarma)

  Budoucí backend       Supabase (PostgreSQL + auth)
  -----------------------------------------------------------------------

**11. Kritéria přijetí (Definition of Done)**

Produkt je připraven k vydání, když:

-   Uživatel dokáže zaznamenat kompletní trénink bez chyby

-   Uživatel dokáže upravit a smazat sérii, cvik i celý trénink

-   Neplatné vstupy (záporná váha, prázdné povinné pole) jsou zamítnuty
    s jasnou chybovou hláškou

-   Prázdný dashboard zobrazuje uvítací výzvu, ne prázdnou stránku

-   Data přetrvají po zavření a znovuotevření prohlížeče

-   Graf a tabulka správně zobrazují historická data

-   Týdenní statistika se správně resetuje každé pondělí

-   Aplikace je plně funkční na mobilním prohlížeči Chrome a Safari

-   Načítání hlavní stránky trvá méně než 2 sekundy

*GymLog PRD · verze 1.1 · Březen 2026*
