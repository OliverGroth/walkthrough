# Block 3: Dataflödet steg för steg
**Tid:** 45 minuter

---

## Slide 3.1: Dataflödet – Översikt

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              DATAFLÖDET                                    │
│              Från rådata till prognos                      │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  ┌─────────────┐                                          │
│  │ SQL Server  │ ← Stored procedure: [getVPB_Data]        │
│  │ Patientdata │   Surrogate_Key, Contact_Start           │
│  └──────┬──────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  ┌─────────────┐                                          │
│  │Preprocessing│ ← Rensa, aggregera, hantera outliers     │
│  └──────┬──────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  ┌─────────────┐     ┌─────────────┐                      │
│  │   Merge     │◄────│ Open-Meteo  │ ← Väderdata          │
│  └──────┬──────┘     └─────────────┘                      │
│         │                                                  │
│         ▼                                                  │
│  ┌─────────────┐                                          │
│  │  Feature    │ ← ~50 features skapas                    │
│  │ Engineering │                                          │
│  └──────┬──────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  ┌─────────────┐                                          │
│  │   Optuna    │ ← 300 trials, väljer bästa params        │
│  │  LightGBM   │                                          │
│  └──────┬──────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  ┌─────────────┐                                          │
│  │   MLflow    │ ← Modeller registreras                   │
│  └─────────────┘                                          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 3.1: Översikt (5 min)

### Vad jag säger:

> "Välkomna tillbaka! Nu ska vi följa data genom hela systemet – från rådata i SQL Server till färdiga prognoser.
>
> Jag har ritat upp flödet här. Låt mig gå igenom det övergripande först, sedan dyker vi ner i varje steg.
>
> **Steg 1:** Data kommer från SQL Server via en stored procedure. Den returnerar två kolumner: ett ID och en tidsstämpel för varje patientbesök.
>
> **Steg 2:** Preprocessing – vi rensar data, tar bort dubletter, aggregerar till daglig nivå, hanterar outliers.
>
> **Steg 3:** Vi hämtar väderdata från Open-Meteo API och mergar med patientdata.
>
> **Steg 4:** Feature engineering – här skapas ~50 features som modellen använder.
>
> **Steg 5:** Träning med Optuna och LightGBM.
>
> **Steg 6:** Modeller registreras i MLflow.
>
> Låt oss gå igenom varje steg i detalj."

---

## Slide 3.2: Steg 1 – Ladda rådata

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              STEG 1: LADDA RÅDATA                          │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Källa: SQL Server                                         │
│  Stored procedure: [getVPB_Data]                           │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Input:                                                    │
│  ┌─────────────────┬─────────────────────────┐            │
│  │ Surrogate_Key   │ Contact_Start           │            │
│  ├─────────────────┼─────────────────────────┤            │
│  │ 12345           │ 2024-01-15 08:23:45     │            │
│  │ 12346           │ 2024-01-15 09:01:12     │            │
│  │ 12347           │ 2024-01-15 09:15:33     │            │
│  │ ...             │ ...                     │            │
│  └─────────────────┴─────────────────────────┘            │
│                                                            │
│  • Surrogate_Key = unikt ID per besök                     │
│  • Contact_Start = tidpunkt för besöket                   │
│  • ~100-200 rader per dag                                 │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Lokal utveckling: Läser från CSV istället                │
│  data/raw/emergency_visits.csv                            │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 3.2: Ladda rådata (10 min)

### Vad jag säger:

> "Första steget är att ladda rådata. I produktion kommer data från SQL Server via en stored procedure som heter `[getVPB_Data]`."

**[Öppna src/data/preprocessing.py]**

> "Låt mig visa hur det fungerar i koden."

**[Visa load_data_from_database()]**

```python
def load_data_from_database(connection_string, stored_procedure='[getVPB_Data]'):
    """Ladda data från SQL Server."""
    
    conn = pyodbc.connect(connection_string)
    cursor = conn.cursor()
    
    query = f"EXEC {stored_procedure}"
    cursor.execute(query)
    
    rows = cursor.fetchall()
    # ... konvertera till DataFrame
```

> "Funktionen tar en connection string och kör stored procedure. Resultatet är en lista med rader som vi konverterar till en pandas DataFrame.
>
> Datan har två kolumner:
> - **Surrogate_Key** – ett unikt ID för varje besök
> - **Contact_Start** – tidpunkten när besöket startade
>
> Det är allt vi behöver. Resten räknar vi ut."

**[Visa fallback till CSV]**

> "För lokal utveckling finns en fallback. Om `DB_CONNECTION_URL` inte är satt läser koden från en CSV-fil istället."

```python
# I training_flow.py
if use_database:
    df = load_data_from_database(DB_CONNECTION_URL)
else:
    df = load_raw_data('data/raw/emergency_visits.csv')
```

### Vad jag visar (live demo):

```python
# I Python/notebook
from src.data.preprocessing import load_raw_data

df = load_raw_data('data/raw/emergency_visits.csv')
print(df.head())
print(f"Antal rader: {len(df)}")
print(f"Kolumner: {df.columns.tolist()}")
```

---

## Slide 3.3: Steg 2 – Preprocessing

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              STEG 2: PREPROCESSING                         │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  1. remove_duplicates()                                    │
│     └── Ta bort dubbletter (samma ID + tidpunkt)          │
│                                                            │
│  2. aggregate_to_daily()                                   │
│     └── Räkna antal besök per dag                         │
│     └── Surrogate_Key, Contact_Start → Date, Patients     │
│                                                            │
│  3. filter_incomplete_current_day()                        │
│     └── Ta bort dagens data (ofullständig)                │
│                                                            │
│  4. handle_missing_dates()                                 │
│     └── Fyll i saknade datum med veckodagsmedian          │
│                                                            │
│  5. detect_and_handle_outliers()                           │
│     └── Cappa extremvärden vid 2× IQR                     │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Output:                                                   │
│  ┌────────────┬─────────────────┐                         │
│  │ Date       │ Patients_per_day│                         │
│  ├────────────┼─────────────────┤                         │
│  │ 2024-01-15 │ 127             │                         │
│  │ 2024-01-16 │ 134             │                         │
│  │ 2024-01-17 │ 118             │                         │
│  └────────────┴─────────────────┘                         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 3.3: Preprocessing (10 min)

### Vad jag säger:

> "Nästa steg är preprocessing – vi rensar och transformerar rådata till ett format modellen kan använda."

**[Öppna src/data/preprocessing.py]**

> "Det finns fem funktioner som körs i sekvens:"

**1. remove_duplicates()**

> "Först tar vi bort dubletter. Om samma besök finns flera gånger (samma ID och tidpunkt) behåller vi bara första."

```python
def remove_duplicates(df):
    duplicates = df.duplicated(subset=['Surrogate_Key', 'Contact_Start'])
    df = df[~duplicates]
    return df
```

**2. aggregate_to_daily()**

> "Sedan aggregerar vi till daglig nivå. Vi räknar helt enkelt antal unika besök per dag."

```python
def aggregate_to_daily(df):
    df['Date'] = df['Contact_Start'].dt.date
    df_agg = df.groupby('Date').size().reset_index(name='Patients_per_day')
    return df_agg
```

> "Efter det här steget har vi två kolumner: `Date` och `Patients_per_day`. Det är vår tidsserie."

**3. filter_incomplete_current_day()**

> "Om vi kör på morgonen är dagens data ofullständig – vi har kanske bara 3 timmars besök. Den raden tas bort för att inte förstöra lag-features."

**4. handle_missing_dates()**

> "Om det saknas datum (t.ex. systemavbrott) fyller vi i med medianen för den veckodagen. Måndag fylls med måndag-median, tisdag med tisdag-median, osv."

**5. detect_and_handle_outliers()**

> "Till sist hanterar vi outliers. Om ett värde är extremt högt eller lågt (mer än 2× IQR från medianen) cappas det. Vi tar inte bort raden – det skulle skapa hål i tidsserien."

### Vad jag visar (live demo):

```python
from src.data.preprocessing import (
    load_raw_data, remove_duplicates, aggregate_to_daily,
    handle_missing_dates, detect_and_handle_outliers
)

# Steg för steg
df = load_raw_data('data/raw/emergency_visits.csv')
print(f"Rådata: {len(df)} rader")

df = remove_duplicates(df)
print(f"Efter dubletter: {len(df)} rader")

df = aggregate_to_daily(df)
print(f"Efter aggregering: {len(df)} dagar")
print(df.head())

print(f"Medel: {df['Patients_per_day'].mean():.1f}")
print(f"Min/Max: {df['Patients_per_day'].min()} - {df['Patients_per_day'].max()}")
```

---

## Slide 3.4: Steg 3 – Väderdata

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              STEG 3: VÄDERDATA                             │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Källa: Open-Meteo API (gratis, ingen API-nyckel)         │
│  https://open-meteo.com/                                   │
│                                                            │
│  Koordinater: Västerås (59.6099, 16.5448)                 │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Parametrar som hämtas:                                    │
│                                                            │
│  ┌──────────────┬─────────────────────────────┐           │
│  │ Temp_Max     │ Högsta temperatur (°C)      │           │
│  │ Temp_Min     │ Lägsta temperatur (°C)      │           │
│  │ Temp_Mean    │ Medeltemperatur (°C)        │           │
│  │ Precipitation│ Nederbörd (mm)              │           │
│  │ Snowfall     │ Snöfall (cm)                │           │
│  └──────────────┴─────────────────────────────┘           │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Träning: Historiskt väder (archive API)                  │
│  Prediktion: Väderprognos (forecast API)                  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 3.4: Väderdata (5 min)

### Vad jag säger:

> "Steg tre är att hämta väderdata. Vi använder Open-Meteo API – det är gratis och kräver ingen API-nyckel."

**[Öppna src/data/weather_integration.py]**

> "Koordinaterna är hårdkodade till Västerås, men kan konfigureras via miljövariabler.
>
> Vi hämtar fem parametrar: max-, min- och medeltemperatur, nederbörd och snöfall. Hypotesen är att väder påverkar hur många som kommer till akuten – extremt kallt väder kan ge fler fallskador, värmeböljor kan ge värmerelaterade problem."

**[Visa fetch_weather_data()]**

```python
def fetch_weather_data(start_date, end_date, lat, lon):
    url = "https://archive-api.open-meteo.com/v1/archive"
    params = {
        "latitude": lat,
        "longitude": lon,
        "start_date": start_date,
        "end_date": end_date,
        "daily": ["temperature_2m_max", "temperature_2m_min", 
                  "temperature_2m_mean", "precipitation_sum", "snowfall_sum"]
    }
    response = requests.get(url, params=params)
    # ... konvertera till DataFrame
```

> "Det finns två API:er: **archive** för historiskt väder (används vid träning) och **forecast** för väderprognos (används vid daglig prediktion)."

---

## Slide 3.5: Steg 4 – Feature Engineering

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              STEG 4: FEATURE ENGINEERING                   │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Input: Date, Patients_per_day + väderdata                │
│  Output: ~50 features                                      │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Datumfeatures (för MÅLDAGEN):                            │
│  • Day_of_week_sin/cos    Veckodag (cyklisk)              │
│  • Month_sin/cos          Månad (cyklisk)                 │
│  • Weekend                Helg-indikator                  │
│  • is_holiday             Röd dag                         │
│                                                            │
│  Lag-features (från PREDIKTIONSDAGEN):                    │
│  • patients_lag_1..7      Senaste veckan                  │
│  • patients_lag_14,21,28  Samma veckodag tidigare         │
│  • patients_lag_35,42     5-6 veckor tillbaka             │
│                                                            │
│  Rolling-features:                                         │
│  • patients_3d_avg/std    Kortsiktig trend                │
│  • patients_14d_avg/std   Medelsiktig trend               │
│  • patients_30d_avg/std   Långsiktig baslinje             │
│                                                            │
│  Väderfeatures (för MÅLDAGEN):                            │
│  • Temp_Max/Min/Mean, Precipitation, Snowfall             │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 3.5: Feature Engineering (10 min)

### Vad jag säger:

> "Nu kommer feature engineering – det här är där vi skapar alla 'ledtrådar' som modellen använder för att göra prognoser."

**[Öppna src/data/feature_engineering.py]**

> "Vi skapar ungefär 50 features, uppdelade i kategorier."

**Datumfeatures:**

> "Först datumfeatures. Veckodag och månad kodas **cykliskt** med sin/cos. Varför? Jo, veckodag 0 (måndag) och veckodag 6 (söndag) ligger nära varandra i verkligheten – båda gränsar till helgen. Med vanlig numerisk kodning skulle modellen tro att de är långt ifrån varandra."

```python
df['Day_of_week_sin'] = np.sin(2 * np.pi * dayofweek / 7)
df['Day_of_week_cos'] = np.cos(2 * np.pi * dayofweek / 7)
```

> "Vi har också helgdagar via `holidays`-biblioteket. Svenska röda dagar plus klämdagar som julafton."

**Lag-features:**

> "Lag-features är historiska värden. `lag_1` är antal patienter igår, `lag_7` är samma veckodag förra veckan. Vi har lags upp till 42 dagar.
>
> Varför så många? Mönster kan vara veckovisa (måndag vs fredag), men också längre (säsongsvariationer)."

**Rolling-features:**

> "Rolling-features fångar trender. 3-dagars medelvärde visar kortsiktig trend, 30-dagars medelvärde visar långsiktig baslinje. Standardavvikelsen visar volatilitet."

**Viktigt koncept – Approach B:**

> "En viktig detalj: datumfeatures och väder beräknas för **måldagen** (den dag vi prognostiserar), medan lag-features beräknas från **prediktionsdagen** (den dag vi gör prognosen).
>
> Exempel: Om vi står på måndag och prognostiserar onsdag (horisont 2):
> - Veckodag-feature = onsdag
> - lag_1 = söndagens patientantal (det vi visste på måndag)
> - Väder = onsdagens väderprognos"

### Vad jag visar (live demo):

```python
from src.data.feature_engineering import engineer_features

# Visa features för horisont 1
df_features = engineer_features(df, weather_df, horizon=1)
print(f"Antal features: {len(df_features.columns)}")
print(df_features.columns.tolist())

# Visa några värden
print(df_features[['Date', 'Patients_per_day', 'patients_lag_1', 
                   'patients_lag_7', 'Day_of_week_sin']].head(10))
```

---

## Slide 3.6: Steg 5 – Träning

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              STEG 5: TRÄNING                               │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Data split (temporal, ingen shuffle):                     │
│                                                            │
│  ┌──────────────────────────────────────────────────┐     │
│  │ Train (80%)    │ Val (10%) │ Test (10%)         │     │
│  │ 2020-2023      │ 2024 jan  │ 2024 feb-mar       │     │
│  └──────────────────────────────────────────────────┘     │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Optuna hyperparameter-sökning:                            │
│                                                            │
│  • 300 trials per horisont                                │
│  • Testar kombinationer av:                               │
│    - n_estimators (50-500)                                │
│    - max_depth (3-12)                                     │
│    - learning_rate (0.001-0.3)                            │
│    - ... 7 parametrar totalt                              │
│  • Väljer kombination med lägst MAE på validering         │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Slutmodell:                                               │
│  • Tränas på Train + Val med bästa parametrar             │
│  • Utvärderas på Test                                     │
│  • + 2 kvantilmodeller för konfidensintervall             │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 3.6: Träning (5 min)

### Vad jag säger:

> "Steg fem är själva träningen. Först delar vi data i tre delar: träning, validering och test. Viktigt: **ingen shuffle** – vi behåller tidsordningen. Annars skulle modellen kunna 'fuska' genom att se framtida data."

**[Visa split]**

> "80% träning, 10% validering, 10% test. Valideringen används av Optuna för att välja hyperparametrar. Testet används för slutgiltig utvärdering.
>
> Optuna kör 300 trials – det betyder att den testar 300 olika kombinationer av hyperparametrar. Den är smart och lär sig vilka områden som är lovande.
>
> När Optuna är klar tränar vi en slutmodell med de bästa parametrarna på all tränings- och valideringsdata. Sedan utvärderar vi på testdata som modellen aldrig sett.
>
> Till sist tränar vi två kvantilmodeller för konfidensintervall."

---

## Slide 3.7: Steg 6 – MLflow

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              STEG 6: MLFLOW                                │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Varje träning loggar:                                     │
│                                                            │
│  • Parametrar (alla hyperparametrar)                      │
│  • Metrics (MAE, RMSE, Coverage)                          │
│  • Artefakter (modell-fil)                                │
│  • Tags (horizon, datum, etc)                             │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Model Registry:                                           │
│                                                            │
│  ┌─────────────────────────────────────────────────┐      │
│  │ er_forecast_horizon_1                           │      │
│  │ ├── Version 1  [Archived]                       │      │
│  │ ├── Version 2  [Archived]                       │      │
│  │ ├── Version 3  [Production] ← Används nu       │      │
│  │ └── Version 4  [Staging]                        │      │
│  └─────────────────────────────────────────────────┘      │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Automatisk promotion:                                     │
│  • Om test MAE < 15 → promotas till Production            │
│  • Gamla versioner arkiveras (behåller 52 st)             │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 3.7: MLflow (5 min)

### Vad jag säger:

> "Sista steget är att registrera modellen i MLflow. Ni har redan sett MLflow-gränssnittet, men låt mig förklara vad som händer i koden.
>
> Varje träning skapar en 'run' i MLflow. Den loggar:
> - Alla hyperparametrar
> - Metrics som MAE, RMSE, coverage
> - Själva modellfilen
> - Tags som horisont och datum
>
> Modellen registreras också i Model Registry. Där finns alla versioner av varje modell. En modell kan vara i olika 'stages': None, Staging, Production, eller Archived.
>
> Koden promotar automatiskt till Production om test-MAE är under 15 patienter. Det är en säkerhetsgräns – om en modell är sämre än så vill vi inte att den ska användas.
>
> Gamla versioner arkiveras automatiskt. Vi behåller de senaste 52 versionerna – ungefär ett års veckovis träning."

---

## Övergång till Block 4:

> "Nu har vi gått igenom hela dataflödet från rådata till MLflow. Har ni några frågor?
>
> [Pausa för frågor]
>
> Okej, nu ska vi göra det praktiskt – vi ska lägga till en ny feature tillsammans."
