# SQL — Global Sustainability Analysis 🌍

> Analisi comparativa multi-dimensionale su dati economici, sanitari, ambientali e migratori di 195 paesi (2023) per supportare le decisioni strategiche di un'organizzazione no-profit internazionale.

**IT** | Progetto SQL realizzato nell'ambito del Master in Data Science, Analytics e AI @ Start2Impact University.

---

## Obiettivo / Objective

**IT:** Trasformare dati grezzi eterogenei in insight operativi per guidare l'allocazione delle risorse di un'organizzazione no-profit focalizzata sullo sviluppo sostenibile globale. L'analisi risponde a tre domande strategiche:

1. Quali paesi presentano le maggiori criticità ambientali?
2. Esistono inefficienze tra disponibilità economica e risultati sanitari?
3. La fragilità economica è associata a esiti migratori critici?

**EN:** Convert heterogeneous raw data into actionable insights to guide resource allocation for a sustainability-focused NGO, answering three strategic questions about environmental stress, healthcare inefficiency, and migration vulnerability.

---

## Dataset

| Dataset | Descrizione | Source |
|---|---|---|
| `Global_Country_Information` | Indicatori economici, sanitari, demografici, ambientali — 195 paesi, 2023 | Kaggle |
| `Missing_Migrants` | Migranti morti o dispersi a livello globale, 2023 | IOM Missing Migrants Project |

I due dataset sono stati uniti tramite `JOIN ON Country` per arricchire l'analisi migratoria con dati economici.

---

## Metodologia / Methodology

Analisi condotta in **PostgreSQL**, strutturata in 4 filoni tematici:

### 1. Analisi Ambientale — Stress CO₂ vs Copertura Forestale

Indicatore costruito: `CO2_Emission / Forested_Area` per identificare i paesi con minore capacità di assorbimento naturale.

```sql
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

Paesi con PIL superiore alla media globale ma aspettativa di vita inferiore alla media → inefficienze sistemiche.

```sql
SELECT Country, GDP, Life_Expectancy, Physicians_Per_Thousand
FROM Global_Country_Information
WHERE GDP IS NOT NULL AND Life_Expectancy IS NOT NULL
  AND GDP > (SELECT AVG(GDP) FROM Global_Country_Information)
  AND Life_Expectancy < (SELECT AVG(Life_Expectancy) FROM Global_Country_Information)
ORDER BY GDP DESC;
```

### 3. Fragilità Sociale — Mortalità Infantile & Istruzione

```sql
SELECT Country, Infant_Mortality, Gross_Tertiary_Education_Enrollment
FROM Global_Country_Information
WHERE Infant_Mortality IS NOT NULL
  AND Gross_Tertiary_Education_Enrollment IS NOT NULL
ORDER BY Infant_Mortality DESC
LIMIT 10;
```

### 4. Migrazione — Vittime per Milione di Abitanti (JOIN tra dataset)

```sql
SELECT
    GCI.Country,
    GCI.GDP,
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

### Stress Ambientale (CO₂ / Area Forestale)
| Paese | CO₂ Emissioni | Copertura Forestale |
|---|---|---|
| Egitto | 238.56 | 0.10% |
| Arabia Saudita | 563.45 | 0.50% |
| Algeria | 150.01 | 0.80% |

### Inefficienza Sanitaria (PIL alto, aspettativa di vita bassa)
| Paese | PIL | Aspettativa di Vita |
|---|---|---|
| Nigeria | ~$448 mld | 54.3 anni |
| Sudafrica | ~$351 mld | 63.9 anni |
| India | ~$2.6 tln | 69.4 anni |

### Esiti Migratori (Vittime per milione di abitanti, 2023)
| Paese | Vittime per milione |
|---|---|
| Myanmar | 3.33 |
| Haiti | 2.22 |
| Afghanistan | 1.31 |

---

## Raccomandazioni Strategiche / Strategic Recommendations

- **Ambiente:** concentrare gli interventi di riforestazione in Egitto, Arabia Saudita e Algeria
- **Sanità:** programmi sanitari mirati in Nigeria, Sudafrica e India — alto PIL ma basso impatto sulla salute
- **Migrazione:** interventi strutturali sulle cause economiche in Myanmar, Haiti e Afghanistan

---

## Stack Tecnologico / Tech Stack

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-SQL-blue?logo=postgresql)
![SQL](https://img.shields.io/badge/SQL-Advanced-lightgrey)

- **Database:** PostgreSQL
- **Tecniche SQL:** JOIN, Subquery, Aggregazioni, NULLIF, normalizzazione per popolazione
- **Ambiente:** pgAdmin / SQL editor

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

## Come Riprodurre / How to Run

1. Importare i dataset in PostgreSQL (link nelle query)
2. Eseguire i file `.sql` in ordine numerico
3. I risultati di ogni query corrispondono a un'area tematica dell'analisi

---

## Autore / Author

**Luca Cino** — [LinkedIn](https://www.linkedin.com/in/lucacino) | [GitHub](https://github.com/lucacino)

Master Professionale in Data Science, Analytics e AI @ Start2Impact University
