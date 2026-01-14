# Block 6: Deploy-processen
**Tid:** 20 minuter

---

## Slide 6.1: Från lokal kod till produktion

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              DEPLOY-PROCESSEN                              │
│              Från lokal kod till produktion                │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  ┌──────────────┐                                         │
│  │ Lokal dator  │                                         │
│  │              │                                         │
│  │ 1. Ändra kod │                                         │
│  │ 2. Testa     │                                         │
│  │ 3. Commit    │                                         │
│  │ 4. Push      │                                         │
│  └──────┬───────┘                                         │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────┐                                         │
│  │ Azure DevOps │                                         │
│  │  (Git repo)  │                                         │
│  └──────┬───────┘                                         │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────┐                                         │
│  │ GPU-server   │                                         │
│  │              │                                         │
│  │ 5. git pull  │                                         │
│  │ 6. Docker    │                                         │
│  │    byggs om  │                                         │
│  │ 7. Ny kod    │                                         │
│  │    körs      │                                         │
│  └──────────────┘                                         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 6.1: Översikt (5 min)

### Vad jag säger:

> "Nu ska vi titta på hur kod går från er dator till produktion. Processen är relativt enkel.
>
> **Steg 1-4** sker på er lokala dator:
> - Ni gör en kodändring
> - Testar lokalt (som vi just gick igenom)
> - Committar till Git
> - Pushar till Azure DevOps
>
> **Steg 5-7** sker på GPU-servern:
> - Någon (eller ett script) kör `git pull` för att hämta senaste koden
> - Docker bygger om imagen automatiskt vid nästa körning
> - Ny kod används
>
> Det finns ingen automatisk CI/CD-pipeline just nu – det är en manuell process. Men det är enkelt att lägga till om ni vill."

---

## Slide 6.2: Git-workflow

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              GIT-WORKFLOW                                  │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  # Se status                                              │
│  git status                                               │
│                                                            │
│  # Lägg till ändringar                                    │
│  git add .                                                │
│  # eller specifika filer:                                 │
│  git add src/data/feature_engineering.py                  │
│                                                            │
│  # Committa med beskrivande meddelande                    │
│  git commit -m "feat: add days_to_next_holiday feature"   │
│                                                            │
│  # Pusha till remote                                      │
│  git push                                                 │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Commit-meddelanden:                                       │
│                                                            │
│  feat:  Ny funktionalitet                                 │
│  fix:   Buggfix                                           │
│  docs:  Dokumentation                                     │
│  test:  Tester                                            │
│  chore: Underhåll                                         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 6.2: Git-workflow (5 min)

### Vad jag säger:

> "Låt mig visa Git-workflowet."

**[Öppna terminal]**

```bash
# Se vad som ändrats
git status

# Se diff
git diff src/data/feature_engineering.py
```

> "Innan ni committar, kolla alltid `git status` och `git diff` för att se vad som ändrats."

```bash
# Lägg till ändringar
git add src/data/feature_engineering.py
git add tests/unit/test_feature_engineering.py

# Eller lägg till allt
git add .
```

> "Ni kan lägga till specifika filer eller allt med `.`"

```bash
# Committa
git commit -m "feat: add days_to_next_holiday feature"
```

> "Commit-meddelandet ska vara beskrivande. Vi använder en konvention:
> - `feat:` för ny funktionalitet
> - `fix:` för buggfixar
> - `docs:` för dokumentation
> - `test:` för tester
>
> Det gör det lättare att förstå historiken."

```bash
# Pusha
git push
```

> "Nu ligger koden på Azure DevOps."

---

## Slide 6.3: Uppdatera servern

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              UPPDATERA SERVERN                             │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  # SSH till servern                                       │
│  ssh user@10.30.51.137                                    │
│                                                            │
│  # Gå till projektmappen                                  │
│  cd /path/to/ER-forecast                                  │
│                                                            │
│  # Hämta senaste koden                                    │
│  git pull                                                 │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Docker bygger om automatiskt:                             │
│                                                            │
│  # Vid nästa träning                                      │
│  docker compose -f docker-compose.jobs.yml \              │
│      run --build training                                 │
│                                                            │
│  # --build flaggan bygger om imagen                       │
│  # Ny kod plockas upp automatiskt                         │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Eller via API:n:                                          │
│  curl http://localhost:5000/train                         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 6.3: Uppdatera servern (5 min)

### Vad jag säger:

> "När koden är pushad behöver servern uppdateras."

```bash
# SSH till servern
ssh user@10.30.51.137

# Gå till projektmappen
cd /path/to/ER-forecast

# Hämta senaste koden
git pull
```

> "Det är allt som behövs. Docker bygger om imagen automatiskt vid nästa körning tack vare `--build`-flaggan."

**[Visa docker-compose.jobs.yml]**

```yaml
# docker-compose.jobs.yml
training:
  build:
    context: .
    dockerfile: Dockerfile.training
  command: python -m flows.training_flow
```

> "När ni kör `docker compose run --build training` händer följande:
> 1. Docker läser Dockerfile.training
> 2. Bygger en ny image med senaste koden
> 3. Startar containern och kör träning
>
> Ni behöver inte göra något speciellt – kodändringar plockas upp automatiskt."

### Alternativ: Trigga via API

> "Ni kan också trigga träning via API:n:"

```bash
curl http://localhost:5000/train
```

> "Det här är vad Windows Task Scheduler gör på schemat."

---

## Slide 6.4: Docker-struktur

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              DOCKER-STRUKTUR                               │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  docker-compose.services.yml (alltid igång)               │
│  ├── MLflow      :5050    Experiment tracking             │
│  ├── PostgreSQL  :5432    Databas                         │
│  ├── MinIO       :9000    Modellartefakter                │
│  ├── Grafana     :3000    Dashboards                      │
│  └── Prometheus  :9090    Metrics                         │
│                                                            │
│  docker-compose.jobs.yml (on-demand)                      │
│  ├── training              Träningsjobb                   │
│  └── prediction            Prediktionsjobb                │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Kommandon:                                                │
│                                                            │
│  # Starta tjänster                                        │
│  docker compose -f docker-compose.services.yml up -d      │
│                                                            │
│  # Kör träning                                            │
│  docker compose -f docker-compose.jobs.yml \              │
│      run --build training                                 │
│                                                            │
│  # Kör prediktion                                         │
│  docker compose -f docker-compose.jobs.yml \              │
│      run --build prediction                               │
│                                                            │
│  # Se loggar                                              │
│  docker compose -f docker-compose.services.yml logs -f    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 6.4: Docker-struktur (5 min)

### Vad jag säger:

> "Låt mig förklara Docker-strukturen. Det finns två compose-filer:
>
> **docker-compose.services.yml** – tjänster som alltid körs:
> - MLflow för experiment tracking
> - PostgreSQL som databas
> - MinIO för modellartefakter
> - Grafana för dashboards
> - Prometheus för metrics
>
> Dessa startar automatiskt när servern bootar (tack vare `restart: always`).
>
> **docker-compose.jobs.yml** – jobb som körs on-demand:
> - training – träningsjobbet
> - prediction – prediktionsjobbet
>
> Dessa triggas antingen manuellt eller via schemaläggaren."

**[Visa några kommandon]**

```bash
# Se vilka containrar som körs
docker compose -f docker-compose.services.yml ps

# Se loggar från MLflow
docker compose -f docker-compose.services.yml logs mlflow

# Starta om en tjänst
docker compose -f docker-compose.services.yml restart grafana
```

> "Om något inte fungerar, börja med att kolla `docker compose ps` och `docker compose logs`."

---

## Slide 6.5: Felsökning

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│              FELSÖKNING                                    │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Problem: Träning/prediktion kraschar                     │
│                                                            │
│  1. Kolla loggar                                          │
│     cat logs/training.log | tail -100                     │
│     cat logs/prediction.log | tail -100                   │
│                                                            │
│  2. Kolla Docker-loggar                                   │
│     docker compose -f docker-compose.jobs.yml logs        │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Problem: MLflow/Grafana inte tillgänglig                 │
│                                                            │
│  1. Kolla att tjänsterna körs                             │
│     docker compose -f docker-compose.services.yml ps      │
│                                                            │
│  2. Starta om                                             │
│     docker compose -f docker-compose.services.yml \       │
│         restart                                           │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│                                                            │
│  Problem: "Data is too stale"                             │
│                                                            │
│  → Patientdata är för gammal                              │
│  → Kontrollera SQL Server-anslutning                      │
│  → Kör stored procedure manuellt för att testa            │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Manus 6.5: Felsökning (5 min)

### Vad jag säger:

> "Låt mig gå igenom några vanliga felsökningsscenarier.
>
> **Problem: Träning eller prediktion kraschar**
>
> Börja med att kolla loggarna:"

```bash
# Träningsloggar
cat logs/training.log | tail -100

# Prediktionsloggar
cat logs/prediction.log | tail -100
```

> "Loggarna visar ofta exakt vad som gick fel. Leta efter 'ERROR' eller 'Exception'.
>
> **Problem: MLflow eller Grafana inte tillgänglig**
>
> Kolla att Docker-tjänsterna körs:"

```bash
docker compose -f docker-compose.services.yml ps
```

> "Om en tjänst är 'Exited', starta om den:"

```bash
docker compose -f docker-compose.services.yml restart mlflow
```

> "**Problem: 'Data is too stale'**
>
> Det här betyder att patientdata är för gammal. Prediktionsflödet kräver data som är max 1 dag gammal.
>
> Orsaker kan vara:
> - SQL Server-anslutningen fungerar inte
> - Stored procedure returnerar gammal data
> - Nätverksproblem
>
> Testa genom att köra stored procedure manuellt i SQL Server Management Studio."

---

## Övergång till Block 7:

> "Nu har vi gått igenom deploy-processen. Har ni några frågor om hur kod går till produktion?
>
> [Pausa för frågor]
>
> Bra, då avslutar vi med en sammanfattning och öppnar för fler frågor."
