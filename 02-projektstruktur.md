# Block 2: Projektstruktur & Viktiga Filer
**Tid:** 45 minuter

---

## Slide 2.1: Projektstruktur – Kartan

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              PROJEKTSTRUKTUR                               │
│              "Kartan över kodbasen"                        │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  ER-forecast/                                              │
│  │                                                         │
│  ├── flows/              ← ENTRÉPUNKTER                   │
│  │   ├── training_flow.py     (träning)                   │
│  │   └── prediction_flow.py   (daglig prediktion)         │
│  │                                                         │
│  ├── src/                ← ALL AFFÄRSLOGIK                │
│  │   ├── data/                (datahantering)             │
│  │   ├── models/              (ML-modeller)               │
│  │   ├── monitoring/          (metrics)                   │
│  │   └── utils/               (hjälpfunktioner)           │
│  │                                                         │
│  ├── scripts/            ← UTVECKLINGSVERKTYG             │
│  ├── tests/              ← TESTER                         │
│  ├── data/               ← DATA (gitignored)              │
│  ├── models/             ← SPARADE MODELLER               │
│  ├── logs/               ← LOGGAR                         │
│  └── docs/               ← DOKUMENTATION                  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 2.1: Projektstruktur (15 min)

### Vad jag säger:

> "Nu ska vi titta på hur projektet är organiserat. Tänk på det här som en karta – när ni ska hitta något är det bra att veta var man ska leta.
>
> Jag brukar dela in det i tre huvudkategorier: **det som körs**, **det som importeras**, och **det som stödjer**."

**[Öppna filträdet i VS Code]**

> "**flows/** – Det här är entrépunkterna. Det är härifrån allt startar. `training_flow.py` är huvudscriptet för träning, och `prediction_flow.py` är det som körs dagligen för att generera prognoser. Om ni undrar 'var börjar allt?' – det är här.
>
> **src/** – All affärslogik ligger här. Det är uppdelat i underkataloger:
> - `data/` – allt som har med datahantering att göra
> - `models/` – ML-modeller, träning, prediktion
> - `monitoring/` – Prometheus-metrics
> - `utils/` – hjälpfunktioner
>
> Poängen med att ha allt i `src/` är att det går att importera. Flows och scripts importerar från src.
>
> **scripts/** – Utvecklingsverktyg. `train_standalone.py` är det ni använder för att träna lokalt utan Docker. Det är inte produktionskod, men väldigt användbart för utveckling.
>
> **tests/** – Alla tester. Uppdelat i `unit/` (enhetstester) och `integration/` (integrationstester).
>
> **data/** – Här ligger datafiler. Viktigt: det mesta här är gitignored i produktion. `raw/` har rådata, `processed/` har bearbetad data, `predictions/` har genererade prognoser.
>
> **models/** – Sparade modeller som `.pkl`-filer. En fil per horisont.
>
> **logs/** – Loggar från träning och prediktion. Här hittar ni också `training_summary_*.md` som sammanfattar varje träningskörning.
>
> **docs/** – Dokumentation. Det finns redan tre dokument som beskriver arkitektur, data/modell, och vidareutveckling."

### Vad jag visar:

**I terminalen:**

```bash
# Visa strukturen
tree -L 2 -d

# Eller om tree inte finns
ls -la
ls -la src/
ls -la flows/
```

**I VS Code:**

- Expandera mapparna en efter en
- Visa att `src/` har `__init__.py` i varje mapp (det är ett Python-paket)

### Minnesregel:

> "En enkel tumregel: **flows/** är vad som körs, **src/** är hur det görs, **scripts/** är för utveckling."

---

## Slide 2.2: De 5 Viktigaste Filerna

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              DE 5 VIKTIGASTE FILERNA                       │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  1. flows/training_flow.py                                 │
│     └── Huvudscript för träning (orkestrerar allt)        │
│                                                            │
│  2. flows/prediction_flow.py                               │
│     └── Daglig prediktion (körs varje dag)                │
│                                                            │
│  3. src/data/feature_engineering.py  ⭐                    │
│     └── Skapar ~50 features (VIKTIGAST för ändringar)     │
│                                                            │
│  4. src/models/train.py                                    │
│     └── Optuna + LightGBM träningslogik                   │
│                                                            │
│  5. src/models/lightgbm_model.py                           │
│     └── Modell-wrapper (punkt + kvantiler)                │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  💡 Om ni ska ändra något → börja i #3                     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 2.2: Översikt viktiga filer (5 min)

### Vad jag säger:

> "Nu ska vi dyka ner i de fem viktigaste filerna. Det här är de filer ni kommer att titta på mest om ni ska felsöka eller vidareutveckla.
>
> **Nummer 1 och 2** är flows – träning och prediktion. De orkestrerar hela flödet.
>
> **Nummer 3** – `feature_engineering.py` – det här är den viktigaste filen om ni ska göra ändringar. Här skapas alla features som modellen använder. Om ni vill förbättra modellen är det oftast här ni börjar.
>
> **Nummer 4** – `train.py` – träningslogiken med Optuna. Här definieras vilka hyperparametrar som testas.
>
> **Nummer 5** – `lightgbm_model.py` – en wrapper runt LightGBM som hanterar både punktprediktioner och konfidensintervall.
>
> Låt oss gå igenom dem en i taget."

---

## Slide 2.3: training_flow.py

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              flows/training_flow.py                        │
│              "Träningsorkestratorn"                        │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  def training_flow():                                      │
│      │                                                     │
│      ├── STEG 1: Ladda data                               │
│      │   └── SQL Server eller CSV                         │
│      │                                                     │
│      ├── STEG 2: Förbehandla                              │
│      │   └── Dubletter, outliers, saknade datum           │
│      │                                                     │
│      ├── STEG 3: Hämta väderdata                          │
│      │   └── Open-Meteo API                               │
│      │                                                     │
│      ├── STEG 4: Träna 7 modeller                         │
│      │   └── En per horisont (1-7 dagar)                  │
│      │   └── Feature engineering per horisont             │
│      │   └── Optuna hyperparameter-sökning                │
│      │                                                     │
│      ├── STEG 5: Registrera i MLflow                      │
│      │   └── Promota till Production om MAE < 15          │
│      │                                                     │
│      └── STEG 6: Arkivera gamla modeller                  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 2.3: training_flow.py (10 min)

### Vad jag säger:

> "Låt oss börja med `training_flow.py`. Det här är huvudscriptet för träning – det orkestrerar hela processen."

**[Öppna flows/training_flow.py]**

> "Filen är ungefär 500 rader, men strukturen är enkel. Det finns en huvudfunktion `training_flow()` som går igenom sex steg."

**[Scrolla till training_flow-funktionen]**

> "**Steg 1: Ladda data.** Koden kollar först om miljövariabeln `DB_CONNECTION_URL` är satt. Om ja, laddar den från SQL Server via en stored procedure. Om nej, faller den tillbaka på att läsa från en CSV-fil. Det gör att ni kan utveckla lokalt utan databasanslutning."

**[Visa koden]**

```python
# Visa denna del
if use_database:
    df = load_data_from_database(
        connection_string=DB_CONNECTION_URL,
        stored_procedure=DB_STORED_PROCEDURE
    )
else:
    df = load_raw_data(raw_data_path)
```

> "**Steg 2: Förbehandla.** Här anropas flera funktioner för att rensa data – ta bort dubletter, hantera saknade datum, cappa outliers."

**[Visa koden]**

```python
df = remove_duplicates(df)
df = aggregate_to_daily(df)
df = filter_incomplete_current_day(df)
df = handle_missing_dates(df)
df = detect_and_handle_outliers(df)
```

> "**Steg 3: Hämta väderdata.** Anropar Open-Meteo API för att få historisk väderdata för samma period som patientdata."

> "**Steg 4: Träna 7 modeller.** Här är huvudloopen. För varje horisont (1-7 dagar framåt) görs feature engineering och sedan träning med Optuna."

**[Visa loopen]**

```python
for horizon in horizons:
    # Feature engineering för denna horisont
    df = engineer_features(df_base.copy(), weather_df, horizon=horizon)
    df = remove_nan_rows(df, drop_all=True)
    
    # Träna med Optuna
    model, metadata = train_model_for_horizon(
        df=df,
        horizon=horizon,
        n_trials=n_optuna_trials,
        use_tscv=use_tscv
    )
```

> "**Steg 5: Registrera i MLflow.** Varje modell registreras i MLflow Model Registry. Om test-MAE är under tröskeln (15 patienter) promotas den automatiskt till Production.
>
> **Steg 6: Arkivera gamla modeller.** MLflow sparar alla versioner, men vi behåller bara de senaste 52 (ett års veckovis träning). Äldre arkiveras automatiskt."

### Poänger att lyfta:

- Flödet är sekventiellt och lätt att följa
- Varje steg loggas till `logs/training.log`
- Felhantering: om något går fel avbryts flödet och loggas

---

## Slide 2.4: prediction_flow.py

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              flows/prediction_flow.py                      │
│              "Daglig prediktion"                           │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  def prediction_flow():                                    │
│      │                                                     │
│      ├── STEG 1: Ladda senaste patientdata                │
│      │                                                     │
│      ├── STEG 2: Validera datafärskhet                    │
│      │   └── Max 1 dag gammal (annars stopp!)             │
│      │                                                     │
│      ├── STEG 3: Hämta väderprognos                       │
│      │   └── Nästa 7 dagars väder                         │
│      │                                                     │
│      ├── STEG 4: Ladda Production-modeller                │
│      │   └── Från MLflow Model Registry                   │
│      │                                                     │
│      ├── STEG 5: Generera prognoser                       │
│      │   └── Punkt + 95% konfidensintervall               │
│      │                                                     │
│      └── STEG 6: Spara                                    │
│          └── CSV + databas                                │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 2.4: prediction_flow.py (5 min)

### Vad jag säger:

> "Nästa fil är `prediction_flow.py`. Den körs dagligen för att generera prognoser för de kommande 7 dagarna."

**[Öppna flows/prediction_flow.py]**

> "Strukturen liknar träningsflödet, men det finns några viktiga skillnader.
>
> **Steg 2 – Validera datafärskhet.** Det här är viktigt. Koden kontrollerar att patientdata är max 1 dag gammal. Om data är äldre stoppar flödet och kastar ett fel. Varför? Jo, modellen använder lag-features som 'antal patienter igår'. Om data är gammal blir de featuresna felaktiga."

**[Visa koden]**

```python
validate_data_freshness(df, max_staleness_days=1, min_rows_required=60)
```

> "**Steg 3 – Väderprognos.** Till skillnad från träning, där vi använder historiskt väder, hämtar vi här en **prognos** för de kommande dagarna. Varje horisont får vädret för sin specifika måldag."

> "**Steg 4 – Ladda Production-modeller.** Här laddas modellerna från MLflow. Koden letar specifikt efter modeller i 'Production'-stadiet. Om ingen hittas faller den tillbaka på lokala .pkl-filer."

> "**Steg 6 – Spara.** Prognoserna sparas både till CSV (för enkel åtkomst) och till databasen (för Grafana-dashboards)."

### Poänger att lyfta:

- Datafärskhet är kritisk – flödet stoppar om data är för gammal
- Modeller laddas från MLflow (Production stage)
- Output: CSV + databas

---

## Slide 2.5: feature_engineering.py ⭐

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              src/data/feature_engineering.py               │
│              "Hjärtat" ⭐                                   │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  engineer_features(df, weather_df, horizon)                │
│      │                                                     │
│      ├── add_date_features_for_target()                   │
│      │   └── Veckodag, månad, helg (för MÅLDAGEN)         │
│      │                                                     │
│      ├── add_holiday_features()                           │
│      │   └── Svenska röda dagar                           │
│      │                                                     │
│      ├── add_rolling_features()                           │
│      │   └── 3d, 14d, 30d medelvärde och std              │
│      │                                                     │
│      ├── add_lag_features()                               │
│      │   └── lag_1 till lag_42                            │
│      │                                                     │
│      ├── add_yoy_features()                               │
│      │   └── År-över-år jämförelser                       │
│      │                                                     │
│      └── shift_weather_to_target()                        │
│          └── Väder för MÅLDAGEN                           │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  ~50 features totalt                                       │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 2.5: feature_engineering.py (10 min)

### Vad jag säger:

> "Nu kommer vi till den viktigaste filen för vidareutveckling – `feature_engineering.py`. Det här är hjärtat av systemet. Alla features som modellen använder skapas här.
>
> Om ni någonsin vill förbättra modellen är det nästan alltid här ni börjar."

**[Öppna src/data/feature_engineering.py]**

> "Filen är ungefär 800 rader, men den är välorganiserad. Det finns en huvudfunktion `engineer_features()` som anropar alla andra."

**[Visa engineer_features()]**

```python
def engineer_features(df, weather_df, horizon):
    """Skapar ~50 features för modellen."""
    
    # 1. Datumfeatures (för MÅLDAGEN)
    df = add_date_features_for_target(df, horizon)
    df = add_holiday_features(df, horizon)
    
    # 2. Historiska mönster (från PREDIKTIONSDAGEN)
    df = add_rolling_features(df)
    df = add_lag_features(df)
    df = add_yoy_features(df)
    df = add_change_features(df)
    
    # 3. Väder (för MÅLDAGEN)
    df = shift_weather_to_target(df, weather_df, horizon)
    
    return df
```

> "Lägg märke till att `horizon` skickas med till flera funktioner. Det är för att vissa features ska beräknas för **måldagen** (den dag vi prognostiserar) och andra för **prediktionsdagen** (den dag vi gör prognosen).
>
> Till exempel: om vi står på måndag och prognostiserar onsdag (horisont 2), så ska veckodag-featuren vara 'onsdag', inte 'måndag'. Men lag_1 (antal patienter igår) ska vara söndagens värde, inte tisdagens."

**[Visa en specifik funktion]**

```python
def add_lag_features(df, lags=[1,2,3,4,5,6,7,14,21,28,35,42]):
    """Historiska patientantal."""
    for lag in lags:
        df[f'patients_lag_{lag}'] = df['Patients_per_day'].shift(lag)
    return df
```

> "Den här funktionen skapar lag-features. `shift(1)` betyder 'värdet från raden innan', alltså igår. `shift(7)` är samma veckodag förra veckan. Vi har lags upp till 42 dagar (6 veckor)."

**[Visa holiday-funktionen]**

```python
def add_holiday_features(df, horizon):
    """Svenska helgdagar via holidays-biblioteket."""
    import holidays
    
    se_holidays = holidays.Sweden(years=years)
    
    df['is_holiday'] = target_dates.apply(lambda d: d in se_holidays)
    df['is_day_before_holiday'] = df['is_holiday'].shift(-1)
    df['is_day_after_holiday'] = df['is_holiday'].shift(1)
```

> "Här används `holidays`-biblioteket för att identifiera svenska helgdagar. Vi skapar tre features: är det helgdag, är det dagen före en helgdag, är det dagen efter. Dagarna runt helger har ofta annorlunda patientmönster."

### Poänger att lyfta:

- `horizon`-parametern är nyckel – den styr vilken dag features beräknas för
- Varje funktion gör en sak och är lätt att förstå
- Lätt att lägga till nya features (vi gör det praktiskt i Block 4)

---

## Slide 2.6: train.py

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              src/models/train.py                           │
│              "Träningslogiken"                             │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  train_model_for_horizon(df, horizon, n_trials=300)        │
│      │                                                     │
│      ├── 1. Skapa target (shift -horizon)                 │
│      │                                                     │
│      ├── 2. Dela data temporalt                           │
│      │   └── 80% train, 10% val, 10% test                 │
│      │                                                     │
│      ├── 3. Optuna hyperparameter-sökning                 │
│      │   └── 300 trials, väljer bästa MAE                 │
│      │                                                     │
│      ├── 4. Träna slutmodell                              │
│      │   └── Med bästa parametrar på train+val            │
│      │                                                     │
│      └── 5. Träna kvantilmodeller                         │
│          └── För 95% konfidensintervall                   │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Optuna testar kombinationer av:                           │
│  n_estimators, max_depth, learning_rate, ...              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 2.6: train.py (5 min)

### Vad jag säger:

> "Nästa fil är `train.py` som innehåller träningslogiken."

**[Öppna src/models/train.py]**

> "Huvudfunktionen är `train_model_for_horizon()`. Den tar en DataFrame med features och tränar en komplett modell för en specifik horisont."

**[Visa funktionen översiktligt]**

> "Fem steg:
>
> **1. Skapa target.** Målet är antal patienter `horizon` dagar framåt. Det görs med `shift(-horizon)`.
>
> **2. Dela data.** Temporal split – 80% träning, 10% validering, 10% test. Viktigt: ingen shuffle! Tidsserier måste behålla ordningen.
>
> **3. Optuna-sökning.** Här testas 300 olika kombinationer av hyperparametrar. Optuna är smart – den lär sig vilka områden som är lovande och fokuserar där."

**[Visa optuna_objective]**

```python
def optuna_objective(trial, X_train, y_train, X_val, y_val):
    params = {
        'n_estimators': trial.suggest_int('n_estimators', 50, 500),
        'max_depth': trial.suggest_int('max_depth', 3, 12),
        'learning_rate': trial.suggest_float('learning_rate', 0.001, 0.3, log=True),
        # ...
    }
```

> "Här ser ni vilka parametrar som testas och deras sökrymd. `n_estimators` mellan 50 och 500, `max_depth` mellan 3 och 12, och så vidare.
>
> **4. Träna slutmodell.** Med de bästa parametrarna tränas en ny modell på all tränings- och valideringsdata.
>
> **5. Kvantilmodeller.** Till sist tränas två extra modeller för konfidensintervall – en för 2.5%-kvantilen och en för 97.5%-kvantilen."

---

## Slide 2.7: lightgbm_model.py

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              src/models/lightgbm_model.py                  │
│              "Modell-wrapper"                              │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  class LightGBMForecaster:                                 │
│      │                                                     │
│      ├── point_model      Huvudprediktion                 │
│      ├── lower_model      2.5% kvantil                    │
│      └── upper_model      97.5% kvantil                   │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Metoder:                                                  │
│                                                            │
│  train_point_model(X, y, params)                          │
│  train_quantile_models(X, y, params)                      │
│  predict(X, return_intervals=True)                        │
│  save_model(filepath)                                      │
│  load_model(filepath)  [classmethod]                      │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Output från predict():                                    │
│  ┌─────────────────┬─────────────┬─────────────┐          │
│  │ point_prediction│ lower_bound │ upper_bound │          │
│  ├─────────────────┼─────────────┼─────────────┤          │
│  │      127.3      │    108.5    │    146.1    │          │
│  └─────────────────┴─────────────┴─────────────┘          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 2.7: lightgbm_model.py (5 min)

### Vad jag säger:

> "Sista filen är `lightgbm_model.py`. Det här är en wrapper-klass runt LightGBM som gör det enkelt att hantera både punktprediktioner och konfidensintervall."

**[Öppna src/models/lightgbm_model.py]**

> "Klassen `LightGBMForecaster` har tre modeller inuti sig:
> - `point_model` – huvudmodellen som ger punktprediktionen
> - `lower_model` – kvantilmodell för nedre gränsen (2.5%)
> - `upper_model` – kvantilmodell för övre gränsen (97.5%)
>
> Tillsammans ger de ett 95% konfidensintervall."

**[Visa predict-metoden]**

```python
def predict(self, X, return_intervals=True):
    point_preds = self.point_model.predict(X)
    
    if return_intervals:
        lower_preds = self.lower_model.predict(X)
        upper_preds = self.upper_model.predict(X)
        
        return pd.DataFrame({
            'point_prediction': point_preds,
            'lower_bound': lower_preds,
            'upper_bound': upper_preds
        })
```

> "När ni anropar `predict()` med `return_intervals=True` får ni tillbaka en DataFrame med tre kolumner: punktprediktion, nedre gräns, övre gräns.
>
> Det finns också `save_model()` och `load_model()` för att spara och ladda modeller som .pkl-filer."

### Poänger att lyfta:

- En klass = tre modeller (punkt + två kvantiler)
- Enkel att använda: `model.predict(X)`
- Sparas som en .pkl-fil som innehåller alla tre modeller

---

## Övergång till paus:

> "Nu har vi gått igenom projektstrukturen och de fem viktigaste filerna. Har ni några frågor innan vi tar en paus?
>
> [Pausa för frågor]
>
> Okej, låt oss ta 15 minuters paus. När vi kommer tillbaka går vi igenom dataflödet steg för steg."
