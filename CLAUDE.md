# VMI Investicinės sąskaitos deklaravimo įrankis

## Projekto tikslas
Parsinti Interactive Brokers (IB) veiklos ataskaitas (Activity Statement) ir sugeneruoti
duomenis Lietuvos VMI metinei pajamų deklaracijai — investicinės sąskaitos daliai (GPM311, H dalis).

## Duomenų šaltiniai
- `source/` aplanke:
  - IB metinė veiklos ataskaita HTM formatu (pvz., `<AccountId>_2025_2025.htm`)
  - IB dienos/momentinė ataskaita (pvz., `<AccountId>_20241231.htm`) — naudojama pradiniam likučiui
  - VMI CSV formato pavyzdys (`Investicines_pvz_1.csv`)

## Pagrindiniai principai

### Valiutų konvertavimas
- IB bazinė valiuta gali būti USD; visos „Total in USD" reikšmės yra IB konversijos
- VMI reikalauja sumų EUR
- EUR/USD kursas imamas iš IB ataskaitos Forex Balances sekcijos (close price periodo pabaigoje)
- Kiekviena valiutų grupė IB ataskaitoje turi „Total inUSD" suvestinę eilutę
- **Svarbu**: Forex Balances rodo tik valiutas su grynų pinigų likučiu. Valiutoms, kurios yra tik
  akcijų pozicijose (pvz. SEK), kursas išvedamas iš „Total" / „Total inUSD" porų Open Positions lentelėje
- Metinė ataskaita NETURI „Base Currency Exchange Rate" sekcijos — ji yra tik dienos ataskaitose

### VMI CSV formatas
```
saskaita,rusis,data,suma,valstybe
```
- `saskaita` = sąskaitos numeris
- `rusis` = operacijos tipo kodas (žr. žemiau)
- `data` = operacijos data (YYYY-MM-DD)
- `suma` = suma EUR (XXXX.XX, visada teigiama)
- `valstybe` = finansų institucijos šalies kodas (pvz. IE — Interactive Brokers Ireland)

### VMI operacijų kodai (rūšis)
- **IA** = pradinis grynų pinigų likutis deklaravimo pradžios dieną
- **IS** = iki deklaravimo pradžios turėtų finansinių priemonių įsigijimo kaina (cost basis), konsoliduota
- **II** = lėšų įnešimas į investicinę sąskaitą (įnašas)
- **IV** = lėšų įnašas į investicinę sąskaitą gaunant dividendus
- **PP** = lėšų išėmimas iš investicinės sąskaitos (išmoka)
- **IP** = paveldėtos finansinės priemonės
- **ID** = padovanotos finansinės priemonės

Nenaudojami kodai investicinei sąskaitai: KS, KG, KL, IB, PT, PU, PI

### Kas laikoma įnašu (II)
- Piniginiai pervedimai į sąskaitą (Electronic Fund Transfer)
- **Dividendai** — kiekvienas dividendas yra atskiras II įrašas
- Kiti piniginiai kreditai (pvz. palūkanų korektimai)

### Pradinis likutis (IA + IS)
- **IA** = viena konsoliduota eilutė su grynų pinigų likučiu deklaravimo pradžios dieną
- **IS** = viena konsoliduota eilutė su visų instrumentų **cost basis** (įsigijimo kaina), NE rinkos vertė
- Iki deklaravimo pradžios duomenys nėra detalizuojami po atskiras tranzakcijas
- Pradiniam likučiui skaičiuoti naudojama atskira IB ataskaita (dienos ataskaita pradžios datai)

### VMI CSV techniniai reikalavimai
- UTF-8 kodavimas
- Maksimaliai 5 000 įrašų faile
- Skirtukas: kablelis (,) arba tab
- Pirma eilutė = stulpelių pavadinimai
- Forma: GPM311, H dalis
- Šaltinis: https://www.vmi.lt/evmi/investicine-saskaita

### IB ataskaitos HTML struktūra
- Sekcijos identifikuojamos pagal `sectionHeading*` CSS klasės div su ID `sec<Name>_<AccountId>Heading`
- Atitinkami duomenys div su ID `tbl<Name>_<AccountId>Body`
- Pagrindinių sekcijų vidiniai pavadinimai:
  - `Transactions` = Sandoriai (ne `Trades`)
  - `CombDiv` = Dividendai
  - `CombInt` = Palūkanos
  - `CombFees` = Mokesčiai/komisiniai
  - `CombDepWith` = Įnašai ir išėmimai
  - `OpenPositions` = Atviros pozicijos
  - `FxPositions` = Forex likučiai
  - `CorporateActions` = Korporatyviniai veiksmai
  - `NAV` = Grynoji turto vertė

### IB sandorių duomenų struktūra
- Sandoriai grupuojami pagal turto klasę (Stocks, Equity and Index Options, Forex)
- Kiekvienoje turto klasėje — pagal valiutą (CHF, EUR, NOK, USD ir t.t.)
- Suvestinės eilutės: `Total<Symbol>`, `Total`, `Total inUSD`
- Kodai: O=Atidarymas, C=Uždarymas, A=Priskyrimas, Ex=Įvykdymas, Ep=Pasibaigęs, P=Dalinis ir t.t.

## Rezultatų failai
- `output/report_YYYY.txt` — detali skaitoma ataskaita su visomis tranzakcijomis (patikrinimui)
- `output/vmi_YYYY.csv` — CSV įkėlimui į VMI
- `output/vmi_YYYY_annotated.csv` — tas pats CSV su papildomu `aprasymas` stulpeliu patikrinimui
- Visi rezultatai — `output/` aplanke

## Naudojimas
```bash
# Tik einamųjų metų tranzakcijos (be pradinio likučio)
python3 parse_ib.py source/<AccountId>_2025_2025.htm

# Su pradiniu likučiu (pirmi deklaravimo metai)
python3 parse_ib.py source/<AccountId>_2025_2025.htm \
  --declaration-start 2025-01-01 \
  --balance-statement source/<AccountId>_20241231.htm
```

## Technologijos
- Python 3 su BeautifulSoup4 HTML parsinimui
- Išorinių API nereikia (valiutų kursai iš IB ataskaitų)
