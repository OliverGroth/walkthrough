# Block 5: Köra & testa lokalt
**Tid:** 30 minuter

---

## Slide 5.1: Lokala utvecklingsverktyg

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              LOKALA UTVECKLINGSVERKTYG                     │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  scripts/                                                  │
│  ├── train_standalone.py     Träna utan Docker            │
│  └── predict_and_visualize.py Prediktion + plots          │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Varför lokala scripts?                                    │
│                                                            │
│  • Snabbare iteration (ingen Docker-build)                │
│  • Enklare debugging                                      │
│  • Fungerar utan databasanslutning                        │
│  • Progress bars och snygg output                         │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Skillnad mot produktion:                                  │
│                                                            │
│  Lokalt:     CSV-fil, lokal MLflow, .pkl-filer            │
│  Produktion: SQL Server, MLflow-server, MinIO             │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 5.1: Introduktion (5 min)

### Vad jag säger:

> "Välkomna tillbaka! Nu ska vi titta på hur man kör och testar lokalt. Det här är viktigt för er utvecklingsprocess.
>
> I `scripts/`-mappen finns två huvudscript:
> - `train_standalone.py` – för att träna modeller lokalt
> - `predict_and_visualize.py` – för att köra prediktion och skapa plots
>
> Varför lokala scripts istället för att köra flows direkt? Jo, de är optimerade för utveckling:
> - Ingen Docker-build behövs
> - Snyggare output med progress bars
> - Fungerar utan databasanslutning (läser från CSV)
> - Enklare att debugga
>
> I produktion körs `flows/training_flow.py` via Docker. Men logiken är densamma – scripts importerar samma funktioner från `src/`."

---

## Slide 5.2: train_standalone.py

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              train_standalone.py                           │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Användning:                                               │
│                                                            │
│  # Snabbtest (5 trials, default horisonter)               │
│  uv run python scripts/train_standalone.py                │
│                                                            │
│  # Specifika horisonter och trials                        │
│  uv run python scripts/train_standalone.py \              │
│      --horizons 1 2 3 --trials 50                         │
│                                                            │
│  # Quick mode (50 trials, alla horisonter)                │
│  uv run python scripts/train_standalone.py --quick        │
│                                                            │
│  # Full träning (300 trials, alla horisonter)             │
│  uv run python scripts/train_standalone.py --full         │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Flaggor:                                                  │
│                                                            │
│  --horizons 1 2 3    Vilka horisonter (default: alla)     │
│  --trials 50         Antal Optuna trials                  │
│  --quick             50 trials, alla horisonter           │
│  --full              300 trials, alla horisonter          │
│  --tscv              TimeSeriesSplit CV (default: på)     │
│  --no-tscv           Stäng av TSCV                        │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 5.2: Träna lokalt (10 min)

### Vad jag säger:

> "Låt mig visa hur `train_standalone.py` fungerar."

**[Öppna terminal]**

> "Det enklaste sättet att köra är utan argument – då får man default-inställningar."

```bash
uv run python scripts/train_standalone.py
```

> "Men oftast vill man specificera vad man tränar. Här är de vanligaste användningsfallen:"

**Snabbtest (för att verifiera att koden fungerar):**

```bash
uv run python scripts/train_standalone.py --horizons 1 --trials 10
```

> "Det här tar ungefär 30 sekunder. Bra för att testa att en kodändring inte kraschar."

**Utveckling (för att utvärdera en ändring):**

```bash
uv run python scripts/train_standalone.py --horizons 1 2 --trials 50
```

> "50 trials ger ett någorlunda tillförlitligt MAE-värde. Tar några minuter."

**Full träning (innan deploy):**

```bash
uv run python scripts/train_standalone.py --full
```

> "300 trials per horisont, alla 7 horisonter. Tar ungefär en timme. Kör det här innan ni pushar en större ändring."

### Vad jag visar (live demo):

```bash
# Kör en snabb träning
uv run python scripts/train_standalone.py --horizons 1 --trials 20
```

> "Titta på outputen:
> - Först laddas data och väder
> - Sedan visas data split (vilka datum som är train/val/test)
> - Progress bar för varje horisont
> - 'Best MAE' uppdateras live när Optuna hittar bättre parametrar
> - I slutet visas en sammanfattning"

**[Visa output]**

> "Resultatet sparas också till `logs/training_summary_*.md`. Där kan ni se alla detaljer."

```bash
# Visa senaste sammanfattningen
ls -la logs/training_summary_*.md | tail -3
cat logs/training_summary_*.md | tail -30
```

---

## Slide 5.3: predict_and_visualize.py

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              predict_and_visualize.py                      │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Användning:                                               │
│                                                            │
│  # Kör prediktion och skapa plots för alla horisonter     │
│  uv run python scripts/predict_and_visualize.py           │
│                                                            │
│  # Specifika horisonter                                   │
│  uv run python scripts/predict_and_visualize.py \         │
│      --horizons 1 2 3                                     │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Output:                                                   │
│                                                            │
│  plots/                                                    │
│  ├── horizon_1_predictions.png                            │
│  ├── horizon_2_predictions.png                            │
│  ├── ...                                                  │
│  └── all_horizons_performance.png                         │
│                                                            │
│  data/predictions/                                         │
│  ├── horizon_1_test_predictions.csv                       │
│  └── ...                                                  │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Visar: Faktiskt vs predikterat + konfidensintervall      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 5.3: Visualisera resultat (5 min)

### Vad jag säger:

> "Efter träning vill man ofta visualisera resultatet. Det gör `predict_and_visualize.py`."

```bash
uv run python scripts/predict_and_visualize.py
```

> "Scriptet:
> 1. Laddar de tränade modellerna från `models/`
> 2. Kör prediktion på testdata
> 3. Beräknar MAE, RMSE, coverage
> 4. Skapar plots
>
> Plotterna sparas i `plots/`-mappen."

**[Öppna en plot]**

> "Här ser ni en typisk plot. Blå linje är faktiskt antal patienter, orange är prediktionen, och det skuggade området är 95% konfidensintervall.
>
> Coverage visar hur stor andel av de faktiska värdena som ligger inom konfidensintervallet. Vi siktar på ~95%.
>
> Det finns också en sammanfattande plot `all_horizons_performance.png` som visar MAE och coverage för alla horisonter."

---

## Slide 5.4: Köra tester

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              KÖRA TESTER                                   │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  # Alla enhetstester                                      │
│  uv run pytest tests/unit/ -v                             │
│                                                            │
│  # Specifik testfil                                       │
│  uv run pytest tests/unit/test_feature_engineering.py -v  │
│                                                            │
│  # Specifikt test                                         │
│  uv run pytest tests/unit/test_preprocessing.py::test_remove_duplicates -v │
│                                                            │
│  # Med coverage-rapport                                   │
│  uv run pytest tests/unit/ --cov=src --cov-report=html    │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Teststruktur:                                             │
│                                                            │
│  tests/                                                    │
│  ├── unit/                   Enhetstester                 │
│  │   ├── test_preprocessing.py                            │
│  │   ├── test_feature_engineering.py                      │
│  │   ├── test_train.py                                    │
│  │   └── ...                                              │
│  └── integration/            Integrationstester           │
│      └── ...                                              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 5.4: Tester (10 min)

### Vad jag säger:

> "Tester är viktiga för att säkerställa att koden fungerar som förväntat. Vi använder pytest."

**[Öppna terminal]**

### Köra alla enhetstester:

```bash
uv run pytest tests/unit/ -v
```

> "Flaggan `-v` ger verbose output – ni ser varje test som körs och om det passerar eller failar."

**[Visa output]**

> "Gröna bockar betyder att testet passerade. Om något är rött har något gått fel."

### Köra specifik testfil:

```bash
uv run pytest tests/unit/test_feature_engineering.py -v
```

> "Om ni bara vill testa en specifik fil – t.ex. efter att ha ändrat i `feature_engineering.py`."

### Köra specifikt test:

```bash
uv run pytest tests/unit/test_preprocessing.py::test_remove_duplicates -v
```

> "Ni kan till och med köra ett enskilt test med `::testnamn`."

### Coverage-rapport:

```bash
uv run pytest tests/unit/ --cov=src --cov-report=html
```

> "Coverage visar hur stor del av koden som täcks av tester. Rapporten sparas i `htmlcov/index.html`."

**[Öppna htmlcov/index.html i webbläsare]**

> "Här ser ni vilka filer som har bra täckning och vilka som saknar tester. Röda rader är kod som aldrig körs av något test."

### Visa ett testexempel:

**[Öppna tests/unit/test_preprocessing.py]**

```python
def test_remove_duplicates():
    """Testar att dubletter tas bort korrekt."""
    df = pd.DataFrame({
        'Surrogate_Key': ['A', 'A', 'B', 'C'],
        'Contact_Start': pd.to_datetime([
            '2024-01-01 08:00',
            '2024-01-01 08:00',  # Dublett
            '2024-01-01 09:00',
            '2024-01-01 10:00'
        ])
    })
    
    result = remove_duplicates(df)
    
    assert len(result) == 3  # En dublett borttagen
    assert 'A' in result['Surrogate_Key'].values
```

> "Ett bra test:
> - Har ett beskrivande namn
> - Skapar testdata med kända värden
> - Anropar funktionen
> - Verifierar resultatet med `assert`
>
> Om ni lägger till ny funktionalitet, skriv ett test för den."

---

## Slide 5.5: Typiskt utvecklingsflöde

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              TYPISKT UTVECKLINGSFLÖDE                      │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  1. Gör kodändring                                        │
│     └── t.ex. ny feature i feature_engineering.py         │
│                                                            │
│  2. Kör relevanta tester                                  │
│     └── uv run pytest tests/unit/test_feature_engineering.py -v │
│                                                            │
│  3. Snabbträna för att verifiera                          │
│     └── uv run python scripts/train_standalone.py \       │
│             --horizons 1 --trials 20                      │
│                                                            │
│  4. Om det ser bra ut: träna ordentligt                   │
│     └── uv run python scripts/train_standalone.py --quick │
│                                                            │
│  5. Visualisera och jämför                                │
│     └── uv run python scripts/predict_and_visualize.py    │
│     └── Jämför MAE med tidigare körningar                 │
│                                                            │
│  6. Om förbättring: commit och push                       │
│     └── git add . && git commit -m "Add feature X"        │
│     └── git push                                          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 5.5: Utvecklingsflöde (5 min)

### Vad jag säger:

> "Låt mig sammanfatta ett typiskt utvecklingsflöde:
>
> **Steg 1:** Gör er kodändring. Kanske en ny feature, en buggfix, eller en optimering.
>
> **Steg 2:** Kör relevanta tester. Om ni ändrade i `feature_engineering.py`, kör testerna för den filen.
>
> **Steg 3:** Snabbträna för att verifiera att inget kraschar. 10-20 trials, en horisont. Tar 30 sekunder.
>
> **Steg 4:** Om det fungerar, träna ordentligt. `--quick` ger 50 trials per horisont. Tar några minuter.
>
> **Steg 5:** Visualisera och jämför. Titta på plots, jämför MAE med tidigare körningar i `logs/`.
>
> **Steg 6:** Om det är en förbättring (eller åtminstone inte sämre), commit och push.
>
> Det här är en iterativ process. Ibland fungerar en idé, ibland inte. Det viktiga är att testa och mäta."

---

## Övergång till Block 6:

> "Nu har vi gått igenom hur man kör och testar lokalt. Har ni några frågor?
>
> [Pausa för frågor]
>
> Bra, då går vi vidare till deploy-processen – hur kod går från er dator till produktion."
