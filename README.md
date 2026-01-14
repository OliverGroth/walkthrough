# Kodgenomgång – Manusmaterial

Detta är manusmaterialet för kodgenomgången med Region Västmanland.

## Översikt

| Fil | Block | Tid | Innehåll |
|-----|-------|-----|----------|
| [00-intro.md](00-intro.md) | Introduktion | 5 min | Välkomna, agenda |
| [01-miljo.md](01-miljo.md) | Block 1 | 30 min | Miljöuppsättning & Python-ramverk |
| [02-projektstruktur.md](02-projektstruktur.md) | Block 2 | 45 min | Projektstruktur & viktiga filer |
| [03-dataflode.md](03-dataflode.md) | Block 3 | 45 min | Dataflödet steg för steg |
| [04-features.md](04-features.md) | Block 4 | 30 min | Praktiskt: Lägga till features |
| [05-kora-lokalt.md](05-kora-lokalt.md) | Block 5 | 30 min | Köra & testa lokalt |
| [06-deploy.md](06-deploy.md) | Block 6 | 20 min | Deploy-processen |
| [07-avslutning.md](07-avslutning.md) | Block 7 | 10 min | Avslutning & frågor |

**Total tid:** ~4 timmar (inklusive 2 pauser à 15 min)

## Tidsplan

```
0:00  - 0:05   Introduktion
0:05  - 0:35   Block 1: Miljöuppsättning
0:35  - 1:20   Block 2: Projektstruktur & viktiga filer
1:20  - 1:35   ☕ PAUS
1:35  - 2:20   Block 3: Dataflödet
2:20  - 2:50   Block 4: Lägga till features
2:50  - 3:05   ☕ PAUS
3:05  - 3:35   Block 5: Köra & testa lokalt
3:35  - 3:55   Block 6: Deploy-processen
3:55  - 4:00   Block 7: Avslutning
```

## Hur materialet är strukturerat

Varje fil innehåller:

1. **Slides** – ASCII-art slides som kan kopieras till presentationsverktyg eller visas direkt
2. **Manus** – Vad du ska säga under varje del
3. **Vad jag visar** – Konkreta kommandon och kod att visa live
4. **Övergångar** – Hur du leder över till nästa block

## Förberedelser

Innan genomgången:

- [ ] Ha koden öppen i VS Code/Cursor
- [ ] Ha en terminal redo i projektmappen
- [ ] Kör `uv sync` så miljön är redo
- [ ] Ha testdata i `data/raw/`
- [ ] Kör igenom `train_standalone.py` en gång så det finns resultat att visa
- [ ] Ha webbläsare redo med MLflow/Grafana (om ni ska visa det)

## Tips

- **Pausa för frågor** – Uppmuntra frågor under hela genomgången
- **Anpassa tempot** – Om något tar längre tid, skippa detaljer i senare block
- **Visa live** – Kör kommandon live istället för att bara visa slides
- **Var konkret** – Använd faktiska filer och data från projektet

## Anpassningar

Materialet kan anpassas efter målgruppen:

- **Mer teknisk publik:** Fördjupa i Block 3-4, visa mer kod
- **Mindre teknisk publik:** Fokusera på Block 1-2 och 5-6, skippa detaljer
- **Kortare tid:** Kombinera Block 3+4, skippa Block 6

## Filer att ha öppna

Under genomgången kommer du visa dessa filer:

```
pyproject.toml
env.template
flows/training_flow.py
flows/prediction_flow.py
src/data/feature_engineering.py
src/models/train.py
src/models/lightgbm_model.py
scripts/train_standalone.py
tests/unit/test_feature_engineering.py
```

Ha dem redo i VS Code så du snabbt kan växla mellan dem.
