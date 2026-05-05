# Išmoktos pamokos — VMI IB ataskaitų parseris

## IB HTM ataskaitos struktūra
- HTML sekcijos naudoja porines ID: `sec<Name>_<AccountId>Heading` ir `tbl<Name>_<AccountId>Body`
- Vidiniai sekcijų pavadinimai skiriasi nuo rodomų:
  - `Transactions` = Trades (ne `Trades`)
  - `CombDiv` = Dividends
  - `CombInt` = Interest
  - `CombFees` = Fees
  - `CombDepWith` = Deposits & Withdrawals
  - `FxPositions` = Forex Balances
  - `ConversionRates` = Base Currency Exchange Rate (tik dienos ataskaitose)
- Lentelėse maišosi header eilutės (th), duomenų eilutės (td) ir suvestinės/total eilutės
- Kiekvieno simbolio suvestinė prasideda `Total<Symbol>` (be tarpo)
- Valiutų grupių antraštė — vienos celės eilutė su valiutos kodu (pvz. „EUR", „USD")
- „Total" eilutė = valiutos lygio suma, „Total inUSD" = USD konversija

## Valiutų konvertavimo subtilybės
- IB pateikia USD sumas, bet VMI reikia EUR
- EUR/USD uždarymo kursas yra Forex Balances sekcijoje
- USD konvertuojamas į EUR: USD suma / EUR_USD kursas
- **Kritiškai svarbu**: Forex Balances rodo tik valiutas su grynų pinigų likučiu!
  Valiutos, esančios tik akcijų pozicijose (pvz. SEK), NEPASIRODO forex likučiuose.
  FX kursus reikia išvesti iš „Total" / „Total inUSD" eilučių porų Open Positions lentelėje.
- Metinė ataskaita NETURI „Base Currency Exchange Rate" sekcijos (ConversionRates) —
  ta sekcija egzistuoja tik dienos ataskaitose
- Periodo pabaigos kursas visoms konversijoms sukuria ~1–3% nuokrypį nuo IB spot kursų
  kiekvienai tranzakcijai. EUR nominuotiems įrašams konversija tiksli (kursas = 1.0).

## IB sandorių kodai
- O = Atidarymo sandoris (Opening)
- C = Uždarymo sandoris (Closing)
- A = Priskyrimas (Assignment) — opcionai
- Ex = Įvykdymas (Exercise) — opcionai
- Ep = Pasibaigęs (Expired) — opcionai
- P = Dalinis įvykdymas (Partial)
- MLL = MaxLossLimit (rizikos valdymas)
- IM = IB sistemos sandoriai
- FPA = Dalinės akcijos (Fractional shares)
- RI = Reinvestavimas

## Išlaikymo mokesčio (Withholding Tax) struktūra
- Dividendų išlaikymas: skirtingi tarifai pagal šalį (US 15%, BE 30%, DE ~26%, FR 25%, CH 35%, FI 35%, JP ~15%)
- Palūkanų išlaikymas: „Withholding @ 20% on Credit Interest" — tai LT GPM nuo EUR/USD palūkanų
- Kai kurie WHT įrašai gali turėti ankstesnių metų datas (korekcijos)

## VMI investicinės sąskaitos taisyklės
- Investicinės sąskaitos režimas: mokestis tik nuo išėmimų, viršijančių įnašus
- Deklaruojama GPM311 formoje, H dalyje
- Reikia sekti: įnašus, išėmimus, pradinį likutį
- **Einamųjų metų tranzakcijos** (II, PP) — kiekviena atskira eilutė
- **Iki deklaravimo pradžios** (IA, IS) — po vieną konsoliduotą eilutę
- IA = grynų pinigų likutis, IS = instrumentų **cost basis** (įsigijimo kaina), NE rinkos vertė
- Visos sumos EUR, visada teigiamos (kodas nustato kryptį)
- CSV: UTF-8, maks. 5 000 eilučių, kablelis arba tab skirtukas
- Pirmos deklaracijos terminas 2025 metams: 2026 m. birželio 1 d.
- Šaltinis: https://www.vmi.lt/evmi/investicine-saskaita

## Kas laikoma įnašu (II)
- Piniginiai pervedimai į sąskaitą (Electronic Fund Transfer)
- **Dividendai** — kiekvienas dividendas yra atskiras II įrašas (dividendai = įnašai)
- Kiti piniginiai kreditai

## Opcionų apdorojimas
- Opcionai yra finansinės priemonės, prekiaujamos investicinėje sąskaitoje
- Pasibaigę opcionai (Ep kodas) turi realizuotą P/L
- Priskirti opcionai (A kodas) virsta akcijų pozicijomis
- Opcionų P/L yra bendro sąskaitos rezultato dalis

## NAV ir pradinis likutis
- NAV (grynoji turto vertė) = oficiali sąskaitos vertė iš IB
- NAV apima: pozicijas, grynuosius, kaupimus, VP skolinimo korekcijas
- Pradiniam likučiui naudojamas **cost basis**, ne NAV (rinkos vertė)
- Skirtumas tarp cost basis ir NAV = nerealizuotas pelnas/nuostoliai
