A clear roadmap keeps your NYC Yellow Taxi analysis organized from raw data → SQLite → SQL analysis → evidence for or against your hypothesis. The structure below is the full, end‑to‑end plan you and your friend can follow inside your Git repo, with each phase building toward a defensible analytical report.

---

## 1. Define the analytical goal and hypothesis  
Your central question is whether **ridership dipped in late 2019 before the first confirmed NYC COVID‑19 case on March 1, 2020**. That requires:

- A **baseline** (2019)
- A **transition period** (Jan–Mar 2020)
- A **pandemic period** (2020–2023)
- A consistent way to measure ridership (trip counts, distances, pickup density)

This framing guides every downstream decision: what data to load, how to structure tables, and which SQL queries matter.

---

## 2. Set up the project structure in Git  
A clean repo separates code from data and keeps collaboration smooth.

```
nyc-taxi-analysis/
│
├── data/
│   └── raw/                # CSVs (ignored by Git)
│
├── database/
│   ├── taxi.db             # optional to commit if small
│   ├── schema.sql
│   ├── import_2019.sql
│   ├── import_2020.sql
│   └── import_2021_2023.sql
│
├── analysis/
│   ├── ridership_2019.sql
│   ├── covid_transition.sql
│   ├── recovery_2021_2023.sql
│   └── report.md
│
└── README.md
```

Your `.gitignore` ensures CSVs and large DB files never get committed.

---

## 3. Download the NYC Yellow Taxi data  
You’ll pull monthly CSVs from Kaggle or the TLC website and place them in:

```
data/raw/
```

This folder stays local and is shared with your friend through Drive/Dropbox.

---

## 4. Build the SQLite database schema  
SQLite needs a table structure that matches the taxi CSV columns. A typical schema includes:

```sql
CREATE TABLE yellow_trips (
    tpep_pickup_datetime TEXT,
    tpep_dropoff_datetime TEXT,
    passenger_count INTEGER,
    trip_distance REAL,
    fare_amount REAL,
    total_amount REAL,
    payment_type INTEGER,
    PULocationID INTEGER,
    DOLocationID INTEGER,
    year INTEGER,
    month INTEGER
);
```

Adding `year` and `month` makes time‑based queries much faster and easier.

Indexes help with performance:

```sql
CREATE INDEX idx_pickup ON yellow_trips(tpep_pickup_datetime);
CREATE INDEX idx_year_month ON yellow_trips(year, month);
```

---

## 5. Import the CSVs into SQLite  
You’ll use `.mode csv` and `.import` for each file. A reusable import script keeps things consistent:

```sql
.mode csv
.import data/raw/yellow_tripdata_2019-01.csv yellow_trips
UPDATE yellow_trips SET year = 2019, month = 1 WHERE year IS NULL;
```

Repeat for each month and year. Your friend runs the same scripts to build an identical database.

---

## 6. Clean and validate the data  
Before analysis, check:

- Missing timestamps  
- Zero‑distance trips  
- Negative fares  
- Outliers in trip distance  
- Duplicate rows  

These checks ensure your conclusions aren’t skewed by bad data.

---

## 7. Build the analytical queries  
Your analysis moves from broad → narrow → causal inference.

### A. Establish the 2019 baseline  
- Monthly ridership  
- Weekly ridership  
- Seasonal patterns  
- Pickup density by zone  
- Payment type distribution  

Example:

```sql
SELECT year, month, COUNT(*) AS trips
FROM yellow_trips
WHERE year = 2019
GROUP BY year, month
ORDER BY year, month;
```

### B. Zoom in on late 2019 (Oct–Dec)  
This is where your hypothesis lives.

- Daily trip counts  
- Week‑over‑week percent change  
- Airport traffic trends  
- Manhattan core vs outer boroughs  

Example:

```sql
SELECT date(tpep_pickup_datetime) AS day, COUNT(*) AS trips
FROM yellow_trips
WHERE year = 2019 AND month BETWEEN 10 AND 12
GROUP BY day
ORDER BY day;
```

### C. Compare late 2019 → early 2020  
You’re looking for early signals before March 1, 2020.

- January/February 2020 vs same months in 2019  
- Daily ridership leading up to March 1  
- Any anomalies before official case confirmation  

### D. Extend through 2023  
This contextualizes recovery:

- Did ridership return to 2019 levels?  
- Which zones recovered fastest?  
- How did trip distance and passenger count change?  

---

## 8. Interpret the results  
Your report should evaluate:

- Whether late‑2019 dips exceed normal seasonal variation  
- Whether dips appear across all boroughs or only specific zones  
- Whether early‑2020 shows pre‑announcement declines  
- How the pandemic reshaped ridership through 2023  

This is where you confirm or reject your hypothesis.

---

## 9. Write the final report  
A strong structure:

1. **Introduction**  
2. **Data and Methods**  
3. **Results**  
4. **Discussion**  
5. **Conclusion**  
6. **Appendix (SQL queries)**  

This makes your analysis reproducible and academically defensible.

---

## 10. Next step  
You’re ready to start building the database and writing the import scripts. Do you want to set up the **schema.sql** and **import_2019.sql** files next so you and your friend can begin loading data?

Press Crtl-Shift-V to preview in Markdown format.