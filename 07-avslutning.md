# Block 7: Avslutning
**Tid:** 10 minuter

---

## Slide 7.1: Sammanfattning – Kom-ihåg-listan

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              KOM-IHÅG-LISTAN                               │
│              Snabbreferens                                 │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Om du vill...              Gå till...                    │
│                                                            │
│  Förstå hela flödet         flows/training_flow.py        │
│                                                            │
│  Lägga till features        src/data/feature_engineering.py│
│                                                            │
│  Ändra modellparametrar     src/models/train.py           │
│                                                            │
│  Köra lokalt                scripts/train_standalone.py   │
│                                                            │
│  Se loggar                  logs/training.log             │
│                             logs/prediction.log           │
│                                                            │
│  Se träningsresultat        logs/training_summary_*.md    │
│                                                            │
│  Felsöka                    Loggar + MLflow UI            │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  💡 Börja alltid med att läsa loggarna!                   │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 7.1: Sammanfattning (3 min)

### Vad jag säger:

> "Låt mig sammanfatta det viktigaste från idag.
>
> **Kom-ihåg-listan** – när ni ska göra något, här är var ni börjar:
>
> - **Förstå hela flödet:** `flows/training_flow.py` – det är entrépunkten
> - **Lägga till features:** `src/data/feature_engineering.py` – det vanligaste sättet att förbättra modellen
> - **Ändra modellparametrar:** `src/models/train.py` – Optuna-sökrymden
> - **Köra lokalt:** `scripts/train_standalone.py` – för utveckling
> - **Se loggar:** `logs/training.log` och `logs/prediction.log`
> - **Se resultat:** `logs/training_summary_*.md`
>
> Och det viktigaste tipset: **börja alltid med att läsa loggarna** när något inte fungerar."

---

## Slide 7.2: Dokumentation

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              DOKUMENTATION                                 │
│              Läs vidare                                    │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  docs/                                                     │
│  │                                                         │
│  ├── arkitekturoversikt.md                                │
│  │   └── Systemdiagram, IP-adresser, Docker               │
│  │   └── Hur klient och server kommunicerar               │
│  │                                                         │
│  ├── guide-for-vidareutveckling.md                        │
│  │   └── Kom igång med lokal utveckling                   │
│  │   └── Steg-för-steg instruktioner                      │
│  │                                                         │
│  ├── data-och-modellbeskrivning.md                        │
│  │   └── Alla features förklarade                         │
│  │   └── Modellarkitektur                                 │
│  │   └── Utvärderingsmetrics                              │
│  │                                                         │
│  └── genomgang/                                           │
│      └── Materialet från denna genomgång                  │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  README.md – Snabbstart och översikt                      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 7.2: Dokumentation (2 min)

### Vad jag säger:

> "Det finns mer dokumentation i `docs/`-mappen:
>
> - **arkitekturoversikt.md** – systemdiagram, IP-adresser, hur Docker är konfigurerat
> - **guide-for-vidareutveckling.md** – steg-för-steg för att komma igång
> - **data-och-modellbeskrivning.md** – alla features förklarade, modellarkitektur
>
> Och nu finns också **genomgang/**-mappen med materialet från idag.
>
> **README.md** i projektets rot har snabbstart-instruktioner."

---

## Slide 7.3: Vanliga frågor

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              VANLIGA FRÅGOR                                │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  F: Hur ofta ska modellen tränas om?                      │
│  S: Veckovis är standard. Kan justeras efter behov.       │
│                                                            │
│  F: Vad händer om MAE blir sämre?                         │
│  S: Modellen promotas inte till Production om MAE > 15.   │
│     Gamla Production-modellen fortsätter användas.        │
│                                                            │
│  F: Kan vi lägga till fler datakällor?                    │
│  S: Ja! Skapa ny funktion i feature_engineering.py.       │
│     T.ex. influensadata, evenemang, etc.                  │
│                                                            │
│  F: Hur lång historik behövs?                             │
│  S: Minst 1 år för att fånga säsongsmönster.              │
│     Mer är bättre (2-3 år idealt).                        │
│                                                            │
│  F: Vad gör vi om servern kraschar?                       │
│  S: Docker-tjänster startar automatiskt vid reboot.       │
│     Kolla logs om något inte fungerar.                    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 7.3: Vanliga frågor (3 min)

### Vad jag säger:

> "Några vanliga frågor som brukar dyka upp:
>
> **Hur ofta ska modellen tränas om?**
> Veckovis är standard. Det fångar upp förändringar i mönster utan att vara för ofta. Kan justeras om ni märker att mönster ändras snabbare.
>
> **Vad händer om MAE blir sämre?**
> Modellen promotas inte till Production om test-MAE är över 15. Den gamla Production-modellen fortsätter användas. Ni ser det i MLflow – nya versionen stannar i 'Staging'.
>
> **Kan vi lägga till fler datakällor?**
> Absolut! Det är bara att skapa en ny funktion i `feature_engineering.py`. Exempel: influensadata från Folkhälsomyndigheten, stora evenemang i staden, etc.
>
> **Hur lång historik behövs?**
> Minst ett år för att fånga säsongsmönster. Två till tre år är idealt. Mer data ger generellt bättre modeller.
>
> **Vad gör vi om servern kraschar?**
> Docker-tjänsterna har `restart: always` och startar automatiskt vid reboot. Om något inte fungerar, kolla loggarna."

---

## Slide 7.4: Kontakt & Support

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              KONTAKT & SUPPORT                             │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Under supportperioden:                                    │
│                                                            │
│  [Kontaktinfo här]                                        │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Tips för framtida frågor:                                │
│                                                            │
│  1. Kolla loggarna först                                  │
│  2. Sök i dokumentationen                                 │
│  3. Testa lokalt om möjligt                               │
│  4. Beskriv problemet tydligt:                            │
│     - Vad försökte ni göra?                               │
│     - Vad hände?                                          │
│     - Vad förväntade ni er?                               │
│     - Relevanta loggrader                                 │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Tack för idag! 🎉                                        │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 7.4: Avslutning (2 min)

### Vad jag säger:

> "[Om det finns supportperiod, nämn kontaktinfo här]
>
> Några tips för framtida frågor:
> 1. Kolla loggarna först – ofta finns svaret där
> 2. Sök i dokumentationen
> 3. Testa lokalt om möjligt
> 4. Om ni behöver hjälp, beskriv problemet tydligt:
>    - Vad försökte ni göra?
>    - Vad hände?
>    - Vad förväntade ni er?
>    - Bifoga relevanta loggrader
>
> Har ni några avslutande frågor?
>
> [Pausa för frågor]
>
> Tack för idag! Lycka till med vidareutvecklingen!"

---

## Checklista efter genomgången

För presentatören – saker att följa upp:

- [ ] Skicka länk till dokumentationen
- [ ] Bekräfta att alla har tillgång till repot
- [ ] Bekräfta att alla kan SSH:a till servern (om relevant)
- [ ] Boka eventuell uppföljning
- [ ] Skicka kontaktinfo för supportperioden
