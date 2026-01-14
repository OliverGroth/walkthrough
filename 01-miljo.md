# Block 1: Miljöuppsättning & Python-ramverk
**Tid:** 30 minuter

---

## Slide 1.1: Verktyg & Teknologier

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              VERKTYG & TEKNOLOGIER                         │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Python 3.11+                                              │
│  └── Språket allt är skrivet i                            │
│                                                            │
│  uv                                                        │
│  └── Modern pakethanterare (ersätter pip/conda)           │
│  └── Hanterar virtuella miljöer automatiskt               │
│  └── Lockfil garanterar samma versioner varje gång        │
│                                                            │
│  Docker                                                    │
│  └── Containrar för produktion                            │
│  └── Isolerar miljön från servern                         │
│                                                            │
│  Git                                                       │
│  └── Versionshantering                                    │
│  └── Azure DevOps som remote                              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 1.1: Verktyg & Teknologier (10 min)

### Vad jag säger:

> "Låt oss börja med verktygen. Projektet är skrivet i **Python 3.11** eller senare – det är ett krav för att vissa bibliotek ska fungera.
>
> För att hantera Python-paket använder vi **uv**. Det är en relativt ny pakethanterare som är mycket snabbare än pip och enklare än conda. Den stora fördelen är att den hanterar virtuella miljöer automatiskt – ni behöver aldrig tänka på att aktivera en miljö.
>
> En annan viktig sak med uv är **lockfilen** – `uv.lock`. Den låser exakt vilka versioner av alla paket som används. Det betyder att om jag kör koden på min dator och ni kör den på servern, så får vi exakt samma versioner. Inga överraskningar.
>
> I produktion kör vi allt i **Docker-containrar**. Det ger oss en isolerad miljö som är identisk varje gång. Men för lokal utveckling behöver ni inte Docker – då räcker uv.
>
> Versionshantering sker via **Git**, och repot ligger på Azure DevOps."

### Vad jag visar:

**Terminal – visa uv i praktiken:**

```bash
# Visa att uv finns
uv --version

# Visa hur man installerar (om de behöver göra det)
# curl -LsSf https://astral.sh/uv/install.sh | sh

# Installera alla dependencies för projektet
cd ER-forecast
uv sync

# Visa att det skapade en .venv-mapp
ls -la .venv/

# Kör ett kommando i projektets miljö
uv run python --version
uv run python -c "import lightgbm; print(lightgbm.__version__)"
```

### Poänger att lyfta:

- `uv sync` läser `pyproject.toml` och `uv.lock`, installerar allt
- `uv run <kommando>` kör kommandot i rätt miljö
- Ni behöver aldrig skriva `source .venv/bin/activate`

---

## Slide 1.2: Dependencies (pyproject.toml)

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              DEPENDENCIES                                  │
│              pyproject.toml                                │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Datahantering                                             │
│  ├── pandas          Tabeller och tidsserier              │
│  └── numpy           Numeriska beräkningar                │
│                                                            │
│  Machine Learning                                          │
│  ├── lightgbm        Gradient boosting (modellen)         │
│  ├── optuna          Hyperparameter-optimering            │
│  └── scikit-learn    Hjälpfunktioner                      │
│                                                            │
│  MLOps                                                     │
│  ├── mlflow          Experiment tracking                  │
│  └── boto3           S3/MinIO-anslutning                  │
│                                                            │
│  Data & API                                                │
│  ├── pyodbc          SQL Server-anslutning                │
│  ├── requests        Väder-API                            │
│  └── holidays        Svenska helgdagar                    │
│                                                            │
│  Testning                                                  │
│  └── pytest          Testramverk                          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 1.2: Dependencies (10 min)

### Vad jag säger:

> "Nu ska vi titta på vilka paket projektet använder. Allt definieras i **pyproject.toml** – det är den moderna standarden för Python-projekt.
>
> Jag delar in dem i kategorier:"

**[Öppna pyproject.toml i editorn]**

> "**Datahantering**: pandas och numpy – standardverktygen för att jobba med data i Python. Pandas för tabeller, numpy för numeriska beräkningar.
>
> **Machine Learning**: Här är **LightGBM** – det är själva modellen vi använder. Det är en typ av gradient boosting, liknande XGBoost men ofta snabbare. **Optuna** används för att hitta bästa hyperparametrarna automatiskt – den testar hundratals kombinationer och väljer den bästa. **scikit-learn** ger oss hjälpfunktioner som train/test-split.
>
> **MLOps**: **MLflow** är vår experiment-tracker och modellregister. Varje träning loggas där med alla parametrar och resultat. **boto3** används för att prata med MinIO, som är vår S3-kompatibla lagring för modellfiler.
>
> **Data & API**: **pyodbc** är drivrutinen för att ansluta till SQL Server där patientdata finns. **requests** används för att hämta väderdata från Open-Meteo API. **holidays** är ett bibliotek som vet alla svenska helgdagar.
>
> **Testning**: **pytest** är testramverket vi använder."

### Vad jag visar:

**Öppna `pyproject.toml` och scrolla igenom:**

```python
# Visa dependencies-sektionen
dependencies = [
    "pandas>=2.1.0",
    "numpy>=1.24.0",
    "lightgbm>=4.1.0",
    "optuna>=3.4.0",
    "mlflow>=2.8.0",
    # ... etc
]
```

**Visa lockfilen kort:**

```bash
# Visa att lockfilen finns och är stor
wc -l uv.lock
head -50 uv.lock
```

### Frågor att förvänta sig:

- "Varför LightGBM och inte XGBoost?" → Snabbare, bra på tabulär data
- "Vad händer om vi vill uppgradera ett paket?" → Ändra i pyproject.toml, kör `uv sync`

---

## Slide 1.3: Miljövariabler (env.template)

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              MILJÖVARIABLER                                │
│              env.template → .env                           │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Lokal utveckling:                                         │
│  └── Kopiera env.template till .env                       │
│  └── .env är gitignored (committas aldrig)                │
│                                                            │
│  Produktion:                                               │
│  └── Sätts i Docker/systemd                               │
│  └── Känsliga värden i säker lagring                      │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Viktiga variabler:                                        │
│                                                            │
│  MLFLOW_*           Anslutning till MLflow-server          │
│  MINIO_*            Modellartefakter (S3-lagring)          │
│  POSTGRES_*         Databas för prognoser                  │
│  DB_CONNECTION_URL  SQL Server (endast produktion)         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 1.3: Miljövariabler (10 min)

### Vad jag säger:

> "Sista delen av miljöuppsättningen handlar om **miljövariabler** – konfiguration som varierar mellan miljöer.
>
> I repot finns en fil som heter **env.template**. Den innehåller alla miljövariabler med exempel-värden. För lokal utveckling kopierar man den till **.env** och anpassar om det behövs.
>
> Viktigt: **.env-filen committas aldrig** – den ligger i .gitignore. Anledningen är att den kan innehålla känsliga saker som lösenord.
>
> I produktion sätts miljövariablerna på annat sätt – antingen i Docker Compose-filen eller via systemd."

**[Öppna env.template]**

> "Låt mig gå igenom de viktigaste:
>
> **MLFLOW_*** – Konfiguration för MLflow-servern. Port, hostname, databasanslutning.
>
> **MINIO_*** – MinIO är vår S3-kompatibla lagring där modellartefakter sparas. Access key och secret key är som användarnamn och lösenord.
>
> **POSTGRES_*** – PostgreSQL-databasen som lagrar prognoser och MLflow-metadata.
>
> **DB_CONNECTION_URL** – Den här är speciell. Den innehåller anslutningssträngen till SQL Server där patientdata finns. Den sätts **endast i produktion**. När den är tom (som vid lokal utveckling) faller koden tillbaka på att läsa från CSV-filer istället."

### Vad jag visar:

**Öppna `env.template`:**

```bash
# Visa filen
cat env.template
```

**Visa hur man kommer igång lokalt:**

```bash
# Kopiera template till .env
cp env.template .env

# Visa att .env är gitignored
cat .gitignore | grep .env
```

**Visa hur koden använder miljövariabler:**

```python
# I flows/training_flow.py
DB_CONNECTION_URL = os.getenv('DB_CONNECTION_URL', '')

# Om tom → använd CSV
if DB_CONNECTION_URL:
    df = load_data_from_database(DB_CONNECTION_URL)
else:
    df = load_raw_data('data/raw/emergency_visits.csv')
```

### Poänger att lyfta:

- Lokalt: kopiera env.template → .env
- Produktion: variablerna sätts i Docker/systemd
- DB_CONNECTION_URL tom = läs från CSV (bra för utveckling)

---

## Övergång till Block 2:

> "Nu har vi gått igenom miljön – Python, uv, dependencies och konfiguration. Har ni några frågor innan vi går vidare till projektstrukturen?
>
> [Pausa för frågor]
>
> Okej, då går vi vidare till projektstrukturen och de viktigaste filerna – det här var något ni specifikt bad om att vi skulle fokusera på."
