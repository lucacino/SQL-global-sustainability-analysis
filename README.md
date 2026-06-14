# SQL — Global Sustainability Analysis 🌍

> Analisi comparativa multi-dimensionale su dati economici, sanitari, ambientali e migratori di 195 paesi (2023) per supportare le decisioni strategiche di un'organizzazione no-profit internazionale.

**IT** | Progetto SQL realizzato nell'ambito del Master in Data Science, Analytics e AI @ Start2Impact University.

---

## Obiettivo / Objective

**IT:** Trasformare dati eterogenei in insight operativi per guidare l'allocazione delle risorse di un'organizzazione no-profit focalizzata sullo sviluppo sostenibile globale. L'analisi risponde a tre domande strategiche:

1. Quali paesi presentano le maggiori criticità ambientali?
2. Esistono inefficienze tra disponibilità economica e risultati sanitari?
3. La fragilità economica è associata a esiti migratori critici?

**EN:** Convert heterogeneous raw data into actionable insights to guide resource allocation for a sustainability-focused NGO, answering three strategic questions about environmental stress, healthcare inefficiency, and migration vulnerability.

---

## Dataset

| Dataset | Descrizione | Source |
|---|---|---|
| `Global_Country_Information` | Indicatori economici, sanitari, demografici, ambientali — 195 paesi, 2023 | [Kaggle](https://www.kaggle.com/datasets/nelgiriyewithana/countries-of-the-world-2023) |
| `Missing_Migrants` | Migranti morti o dispersi a livello globale, 2023 | [Kaggle](https://www.kaggle.com/datasets/nelgiriyewithana/global-missing-migrants-dataset) |

I due dataset sono stati uniti tramite `JOIN ON Country` per arricchire l'analisi migratoria con dati economici.

---

## Metodologia / Methodology

Analisi condotta in **PostgreSQL**, strutturata in 4 filoni tematici. Ogni query è preceduta da un commento che descrive obiettivo e risultato atteso.

### 1. Stress Ambientale — CO₂ vs Copertura Forestale

```sql
-- Obiettivo: Identificare i paesi con elevato livello di emissioni di CO2
-- ma con una bassa copertura forestale, condizione che riduce
-- la capacità di assorbimento delle emissioni.
-- Risultato atteso: Una lista di paesi prioritari per programmi
-- di riforestazione o transizione energetica.

SELECT
    Country,
    Co2_Emission,
    Forested_Area,
    (Co2_Emission / NULLIF(Forested_Area, 0)) AS Emissioni_per_area_forestale
FROM Global_Country_Information
WHERE Co2_Emission IS NOT NULL
  AND Forested_Area IS NOT NULL
ORDER BY Emissioni_per_area_forestale DESC
LIMIT 10;
```

### 2. Efficienza Sanitaria — PIL Elevato vs Aspettativa di Vita

```sql
-- Obiettivo: Individuare paesi con livelli economici medio-alti
-- ma risultati sanitari inferiori alla media, suggerendo
-- inefficienze strutturali dove un intervento sanitario
-- potrebbe avere alto impatto marginale.
-- Risultato atteso: Identificazione di paesi con alto PIL
-- ma bassa aspettativa di vita.

SELECT
    Country,
    GDP,
    Life_Expectancy,
    Physicians_Per_Thousand,
    Gross_Tertiary_Education_Enrollment
FROM Global_Country_Information
WHERE GDP IS NOT NULL AND Life_Expectancy IS NOT NULL
  AND GDP > (SELECT AVG(GDP) FROM Global_Country_Information)
  AND Life_Expectancy < (SELECT AVG(Life_Expectancy) FROM Global_Country_Information)
ORDER BY GDP DESC;
```

### 3. Fragilità Sociale — Mortalità Infantile & Istruzione

```sql
-- Obiettivo: Confrontare mortalità infantile e accesso
-- all'istruzione per identificare paesi dove interventi
-- integrati (sanità + educazione) possono essere più efficaci.
-- Risultato atteso: Evidenziare paesi con alta mortalità
-- infantile e basso livello di istruzione terziaria.

SELECT
    Country,
    Infant_Mortality,
    Gross_Tertiary_Education_Enrollment
FROM Global_Country_Information
WHERE Infant_Mortality IS NOT NULL
  AND Gross_Tertiary_Education_Enrollment IS NOT NULL
ORDER BY Infant_Mortality DESC
LIMIT 10;
```

### 4. Migrazione — Vittime per Milione di Abitanti (JOIN)

```sql
-- Obiettivo: Analizzare la relazione tra condizioni economiche
-- del paese di origine e numero di migranti morti o dispersi,
-- normalizzando i dati per dimensione del paese.
-- Risultato atteso: Identificare paesi dove la fragilità
-- economica è associata a esiti migratori particolarmente critici.

SELECT
    GCI.Country,
    GCI.GDP,
    GCI.Unemployment_Rate,
    GCI.Population,
    SUM(MM.Total_Dead_and_Missing) AS Vittime_totali,
    (SUM(MM.Total_Dead_and_Missing)::NUMERIC / NULLIF(GCI.Population, 0)) * 1000000
        AS Vittime_per_milione_abitanti
FROM Global_Country_Information GCI
JOIN Missing_Migrants MM ON GCI.Country = MM.Country_of_Origin
WHERE MM.Incident_Year = 2023
GROUP BY GCI.Country, GCI.GDP, GCI.Unemployment_Rate, GCI.Population
ORDER BY Vittime_per_milione_abitanti DESC;
```

---

## Risultati Chiave / Key Results

### Stress Ambientale
| Paese | CO₂ Emissioni | Copertura Forestale |
|---|---|---|
| Egitto | 238.56 | 0.10% |
| Arabia Saudita | 563.45 | 0.50% |
| Algeria | 150.01 | 0.80% |

### Inefficienza Sanitaria
| Paese | PIL | Aspettativa di Vita |
|---|---|---|
| Nigeria | ~$448 mld | 54.3 anni |
| Sudafrica | ~$351 mld | 63.9 anni |
| India | ~$2.6 tln | 69.4 anni |

### Fragilità Migratoria (vittime per milione di abitanti, 2023)
| Paese | Vittime per milione |
|---|---|
| Myanmar | 3.33 |
| Haiti | 2.22 |
| Afghanistan | 1.31 |

---

## Raccomandazioni Strategiche / Strategic Recommendations

- **Ambiente:** concentrare interventi di riforestazione e transizione energetica in Egitto, Arabia Saudita e Algeria
- **Sanità:** programmi mirati in Nigeria, Sudafrica e India — alto PIL, basso impatto sulla salute
- **Migrazione:** interventi strutturali sulle cause economiche in Myanmar, Haiti e Afghanistan

---

## Tech Stack

- Database: PostgreSQL
- Tecniche SQL: JOIN, Subquery, Aggregazioni, NULLIF, normalizzazione per popolazione
- Ambiente: pgAdmin

---

## Struttura del Repository

```
sql-global-sustainability-analysis/
├── README.md
├── queries/
│   ├── 01_environmental_stress.sql
│   ├── 02_healthcare_efficiency.sql
│   ├── 03_infant_mortality.sql
│   └── 04_migration_vulnerability.sql
└── presentation/
    └── Progetto_SQL_di_Luca_Cino.pdf
```

---

## Autore / Author

**Luca Cino** — [LinkedIn](https://www.linkedin.com/in/lucacino) | [GitHub](https://github.com/lucacino)

Master Professionale in Data Science, Analytics e AI @ Start2Impact University
