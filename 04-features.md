# Block 4: Praktiskt – Lägga till features
**Tid:** 30 minuter

---

## Slide 4.1: Varför nya features?

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              LÄGGA TILL NYA FEATURES                       │
│              Det vanligaste sättet att förbättra modellen  │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Varför?                                                   │
│                                                            │
│  • Modellen kan bara lära sig från features den får       │
│  • Nya features = nya "ledtrådar"                         │
│  • Ofta mer effektivt än att ändra modellparametrar       │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Processen:                                                │
│                                                            │
│  1. Hypotes – varför skulle detta hjälpa?                 │
│  2. Implementera funktionen                               │
│  3. Anropa i engineer_features()                          │
│  4. Skriv test                                            │
│  5. Träna och jämför MAE                                  │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  💡 Börja alltid med en hypotes!                          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 4.1: Introduktion (5 min)

### Vad jag säger:

> "Nu ska vi göra något praktiskt – vi ska lägga till en ny feature tillsammans. Det här är det vanligaste sättet att förbättra modellen.
>
> Varför features? Jo, modellen kan bara lära sig från den information den får. Om vi inte berättar att det är julafton kan den inte lära sig att julafton är speciell. Nya features ger nya 'ledtrådar'.
>
> Processen är alltid densamma:
> 1. **Hypotes** – varför skulle detta hjälpa? Det är viktigt att tänka igenom innan man kodar.
> 2. **Implementera** – skriv funktionen
> 3. **Integrera** – anropa i `engineer_features()`
> 4. **Testa** – skriv ett enhetstest
> 5. **Utvärdera** – träna och se om MAE förbättras
>
> Låt mig visa ett konkret exempel."

---

## Slide 4.2: Exempel – Dagar till nästa helgdag

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              EXEMPEL: DAGAR TILL NÄSTA HELGDAG             │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Hypotes:                                                  │
│                                                            │
│  "Dagarna precis före en helgdag kan ha fler patienter    │
│   – folk vill bli klara innan helgen"                     │
│                                                            │
│  "Dagarna precis efter en helgdag kan ha fler patienter   │
│   – uppskjutna besök under helgen"                        │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Feature: days_to_next_holiday                             │
│                                                            │
│  ┌────────────┬─────────────────────┐                     │
│  │ Date       │ days_to_next_holiday│                     │
│  ├────────────┼─────────────────────┤                     │
│  │ 2024-12-20 │ 4 (till julafton)   │                     │
│  │ 2024-12-21 │ 3                   │                     │
│  │ 2024-12-22 │ 2                   │                     │
│  │ 2024-12-23 │ 1                   │                     │
│  │ 2024-12-24 │ 0 (julafton)        │                     │
│  └────────────┴─────────────────────┘                     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 4.2: Hypotes och design (5 min)

### Vad jag säger:

> "Låt oss ta ett konkret exempel. Vi har redan `is_holiday` som säger om det är helgdag eller inte. Men vi har ingen feature som säger hur **nära** en helgdag vi är.
>
> **Hypotes:** Dagarna precis före en helgdag kan ha fler patienter – folk vill bli klara innan helgen. Och dagarna precis efter kan också ha fler – uppskjutna besök.
>
> **Feature:** `days_to_next_holiday` – antal dagar till nästa helgdag.
>
> Värdet 0 betyder att det är helgdag idag. Värdet 1 betyder att det är helgdag imorgon. Och så vidare.
>
> Låt oss implementera det."

---

## Slide 4.3: Implementering

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              IMPLEMENTERING                                │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Steg 1: Skapa funktionen i feature_engineering.py        │
│                                                            │
│  def add_days_to_next_holiday(df, horizon=0):             │
│      """Antal dagar till nästa helgdag."""                │
│      import holidays                                       │
│                                                            │
│      target_dates = df['Date'] + pd.Timedelta(days=horizon)│
│      se_holidays = holidays.Sweden(years=...)             │
│                                                            │
│      def days_to_next(date):                              │
│          for i in range(0, 31):                           │
│              if (date + timedelta(days=i)) in se_holidays:│
│                  return i                                 │
│          return 30                                        │
│                                                            │
│      df['days_to_next_holiday'] = target_dates.apply(     │
│          days_to_next                                     │
│      )                                                     │
│      return df                                             │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Steg 2: Anropa i engineer_features()                     │
│                                                            │
│  df = add_holiday_features(df, horizon)                   │
│  df = add_days_to_next_holiday(df, horizon)  # NY RAD     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 4.3: Live-kodning (15 min)

### Vad jag säger:

> "Nu ska vi koda det här tillsammans. Jag öppnar `feature_engineering.py`."

**[Öppna src/data/feature_engineering.py]**

> "Jag scrollar ner till där de andra feature-funktionerna ligger... här, efter `add_holiday_features`."

### Vad jag skriver (live):

```python
def add_days_to_next_holiday(df: pd.DataFrame, horizon: int = 0) -> pd.DataFrame:
    """
    Lägg till antal dagar till nästa helgdag.
    
    Hypotes: Dagarna precis före/efter helgdagar kan ha annorlunda
    patientmönster (uppskjutna besök, förberedelser inför helg).
    
    Args:
        df: DataFrame med 'Date' kolumn
        horizon: Prognoshorisont i dagar
    
    Returns:
        DataFrame med 'days_to_next_holiday' feature
    """
    import holidays
    from datetime import timedelta
    
    logger.info(f"Adding days_to_next_holiday feature (horizon={horizon})")
    
    df = df.copy()
    df['Date'] = pd.to_datetime(df['Date'])
    
    # Beräkna för måldagen (inte prediktionsdagen)
    target_dates = df['Date'] + pd.Timedelta(days=horizon)
    
    # Hämta alla svenska helgdagar för relevanta år
    years = target_dates.dt.year.unique().tolist()
    se_holidays = holidays.Sweden(years=years)
    
    def days_to_next(date):
        """Hitta antal dagar till nästa helgdag."""
        date = date.date() if hasattr(date, 'date') else date
        for i in range(0, 31):  # Kolla max 30 dagar framåt
            check_date = date + timedelta(days=i)
            if check_date in se_holidays:
                return i
        return 30  # Default om ingen helgdag inom 30 dagar
    
    df['days_to_next_holiday'] = target_dates.apply(days_to_next)
    
    logger.info(f"Added days_to_next_holiday (range: {df['days_to_next_holiday'].min()}-{df['days_to_next_holiday'].max()})")
    
    return df
```

> "Några saker att notera:
> - Vi använder `horizon` för att beräkna för rätt dag
> - Vi loopar framåt max 30 dagar
> - Vi loggar vad vi gör (bra för debugging)
> - Docstring förklarar hypotesen"

**[Scrolla till engineer_features()]**

> "Nu måste vi anropa funktionen. Jag hittar `engineer_features()` och lägger till ett anrop."

```python
def engineer_features(df, weather_df, horizon):
    # ... befintlig kod ...
    
    df = add_holiday_features(df, horizon)
    df = add_days_to_next_holiday(df, horizon)  # ← NY RAD
    
    # ... resten ...
```

---

## Slide 4.4: Testa

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              TESTA                                         │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Steg 3: Skriv enhetstest                                 │
│                                                            │
│  # tests/unit/test_feature_engineering.py                 │
│                                                            │
│  def test_add_days_to_next_holiday():                     │
│      """Testar days_to_next_holiday feature."""           │
│      df = pd.DataFrame({                                  │
│          'Date': pd.to_datetime([                         │
│              '2024-12-23',  # Dagen före julafton         │
│              '2024-12-24',  # Julafton                    │
│              '2024-12-25',  # Juldagen                    │
│          ]),                                               │
│          'Patients_per_day': [100, 80, 70]                │
│      })                                                    │
│                                                            │
│      result = add_days_to_next_holiday(df, horizon=0)     │
│                                                            │
│      assert 'days_to_next_holiday' in result.columns      │
│      assert result['days_to_next_holiday'].iloc[0] == 1   │
│      assert result['days_to_next_holiday'].iloc[1] == 0   │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Steg 4: Kör testerna                                     │
│                                                            │
│  uv run pytest tests/unit/test_feature_engineering.py -v  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 4.4: Skriva test (5 min)

### Vad jag säger:

> "Nu ska vi skriva ett test. Det är viktigt av två anledningar: det verifierar att koden fungerar, och det fungerar som dokumentation för framtida utvecklare."

**[Öppna tests/unit/test_feature_engineering.py]**

> "Jag lägger till ett nytt test i slutet av filen."

### Vad jag skriver:

```python
def test_add_days_to_next_holiday():
    """Testar att days_to_next_holiday beräknas korrekt."""
    from src.data.feature_engineering import add_days_to_next_holiday
    
    # Skapa testdata runt jul 2024
    df = pd.DataFrame({
        'Date': pd.to_datetime([
            '2024-12-23',  # Dagen före julafton
            '2024-12-24',  # Julafton (de facto helgdag)
            '2024-12-25',  # Juldagen (röd dag)
            '2024-12-26',  # Annandag jul (röd dag)
            '2024-12-27',  # Vanlig dag
        ]),
        'Patients_per_day': [100, 80, 70, 75, 110]
    })
    
    result = add_days_to_next_holiday(df, horizon=0)
    
    # Verifiera att kolumnen skapades
    assert 'days_to_next_holiday' in result.columns
    
    # 23 dec: 1 dag till julafton (om julafton räknas)
    # eller 2 dagar till juldagen (om bara röda dagar räknas)
    assert result['days_to_next_holiday'].iloc[0] <= 2
    
    # 25 dec (juldagen): 0 dagar (det är helgdag)
    assert result['days_to_next_holiday'].iloc[2] == 0
```

### Vad jag kör:

```bash
# Kör testet
uv run pytest tests/unit/test_feature_engineering.py::test_add_days_to_next_holiday -v
```

> "Grönt! Testet passerar. Nu vet vi att funktionen fungerar."

---

## Slide 4.5: Utvärdera

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              UTVÄRDERA                                     │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Steg 5: Träna och jämför MAE                             │
│                                                            │
│  # Snabbtest med få trials                                │
│  uv run python scripts/train_standalone.py \              │
│      --horizons 1 --trials 20                             │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Jämför resultat:                                          │
│                                                            │
│  Före:  MAE = 6.35                                        │
│  Efter: MAE = 6.28  ✓ Förbättring!                        │
│                                                            │
│  eller                                                     │
│                                                            │
│  Före:  MAE = 6.35                                        │
│  Efter: MAE = 6.41  ✗ Ingen förbättring                   │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  💡 Tips:                                                  │
│  • Kör samma antal trials för rättvis jämförelse          │
│  • Testa på flera horisonter                              │
│  • Dokumentera vad du testade och resultatet              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 4.5: Utvärdera (5 min)

### Vad jag säger:

> "Sista steget är att utvärdera om featuren faktiskt hjälper. Vi tränar modellen och jämför MAE."

### Vad jag kör:

```bash
# Träna med den nya featuren
uv run python scripts/train_standalone.py --horizons 1 --trials 20
```

> "Vi kör med bara 20 trials för att det ska gå snabbt. I verkligheten skulle ni köra med fler för ett mer tillförlitligt resultat.
>
> [Vänta på resultat]
>
> Okej, vi fick MAE = X.XX. Jämför med tidigare körningar i `logs/training_summary_*.md`.
>
> Om MAE gick ner – bra, featuren hjälper! Om MAE gick upp eller var oförändrad – då kanske hypotesen var fel, eller så behöver featuren justeras.
>
> Det är helt okej att en feature inte hjälper. Det viktiga är att testa och lära sig."

### Tips att nämna:

> "Några tips:
> - Kör alltid samma antal trials när ni jämför
> - Testa på flera horisonter, inte bara en
> - Dokumentera vad ni testade och resultatet – det sparar tid senare
> - Om en feature inte hjälper, ta bort den. Fler features är inte alltid bättre."

---

## Övergång till paus:

> "Nu har vi gått igenom hur man lägger till en ny feature – från hypotes till utvärdering. Har ni några frågor?
>
> [Pausa för frågor]
>
> Bra, låt oss ta 15 minuters paus. När vi kommer tillbaka tittar vi på hur man kör och testar lokalt."
