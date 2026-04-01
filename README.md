# 🫀 Medical Data Visualizer

A Python data analysis project that visualizes relationships between cardiac disease, body measurements, blood markers, and lifestyle choices using **pandas**, **seaborn**, and **matplotlib**.

> Part of the **freeCodeCamp – Data Analysis with Python** certification.

***

## 📁 Project Files

| File | Purpose |
|------|---------|
| `medical_data_visualizer.py` | Main script — all data processing and chart generation logic |
| `medical_examination.csv` | Dataset — 70,000 patient records with 12 features |
| `main.py` | Development runner — generates charts and runs unit tests |
| `test_module.py` | Unit tests provided by freeCodeCamp |
| `catplot.png` | Output — categorical bar chart |
| `heatmap.png` | Output — correlation heatmap |
| `requirements.txt` | Python dependencies |

***

## 📊 Dataset

Each row = one patient. Columns are grouped into three types:

| Type | Features |
|------|---------|
| **Objective** (measured) | `age` (days), `height` (cm), `weight` (kg), `gender` |
| **Examination** (test results) | `ap_hi` systolic BP, `ap_lo` diastolic BP, `cholesterol` (1/2/3), `gluc` (1/2/3) |
| **Subjective** (self-reported) | `smoke`, `alco`, `active` (all binary 0/1) |
| **Target** | `cardio` — 0 = no disease, 1 = has disease |

***

## 🔧 How It Works

### Step 1 — Load Data
```python
df = pd.read_csv('medical_examination.csv')
```

### Step 2 — Add `overweight` Column
BMI = weight(kg) / height(m)²
If BMI > 25 → overweight = 1, else overweight = 0

```python
df['overweight'] = (df['weight'] / ((df['height'] / 100) ** 2) > 25).astype(int)
```

### Step 3 — Normalize `cholesterol` and `gluc`
Make 0 = good, 1 = bad (original scale: 1=normal, 2=above normal, 3=well above normal)

```python
df['cholesterol'] = (df['cholesterol'] > 1).astype(int)
df['gluc']        = (df['gluc'] > 1).astype(int)
```

***

## 📈 Chart 1 — Categorical Plot (`catplot.png`)

**What it shows:** Count of good (0) vs bad (1) outcomes for `cholesterol`, `gluc`, `smoke`, `alco`, `active`, and `overweight` — split into two panels by `cardio`.

**Key functions used:**

| Function | Why it's used |
|----------|-------------|
| `pd.melt()` | Converts wide format (6 feature columns) → long format (variable + value columns) so seaborn can plot all features in one chart |
| `.groupby().size()` | Counts patients in each group (cardio × variable × value) |
| `.reset_index(name='total')` | Converts grouped index back into a normal DataFrame column named `total` |
| `sns.catplot(col='cardio', hue='value', kind='bar')` | Creates two side-by-side bar panels, one per cardio value |

```python
df_cat = pd.melt(df, id_vars=['cardio'],
                 value_vars=['cholesterol','gluc','smoke','alco','active','overweight'])

df_cat = df_cat.groupby(['cardio','variable','value']).size().reset_index(name='total')

sns.catplot(data=df_cat, x='variable', y='total',
            hue='value', col='cardio', kind='bar')
```

***

## 🌡️ Chart 2 — Correlation Heatmap (`heatmap.png`)

**What it shows:** How strongly every feature correlates with every other feature, using only clean patient data.

### Data Cleaning (removes bad records)

| Filter | Reason |
|--------|--------|
| `ap_lo <= ap_hi` | Diastolic can't be higher than systolic — physically impossible |
| Height within 2.5th–97.5th percentile | Remove extreme outliers |
| Weight within 2.5th–97.5th percentile | Remove extreme outliers |

**Key functions used:**

| Function | Why it's used |
|----------|-------------|
| `df.corr()` | Calculates correlation coefficient between every pair of numeric columns |
| `np.triu()` | Creates upper-triangle mask — hides duplicate half of symmetric matrix |
| `sns.heatmap(annot=True, mask=mask)` | Draws color-coded correlation grid with values, hiding upper triangle |

```python
df_heat = df[
    (df['ap_lo'] <= df['ap_hi']) &
    (df['height'] >= df['height'].quantile(0.025)) &
    (df['height'] <= df['height'].quantile(0.975)) &
    (df['weight'] >= df['weight'].quantile(0.025)) &
    (df['weight'] <= df['weight'].quantile(0.975))
]

corr = df_heat.corr()
mask = np.triu(np.ones_like(corr, dtype=bool))

sns.heatmap(corr, mask=mask, annot=True, fmt='.1f',
            square=True, linewidths=0.5)
```

***

## ▶️ How to Run

```bash
# Install dependencies
pip install -r requirements.txt

# Run charts + tests
python main.py
```

Expected output: `Ran 4 tests in ~2.6s — OK`

***

## 🧠 Key Concepts (Quick Revision)

| Concept | One-line explanation |
|---------|---------------------|
| `pd.melt()` | Unpivots columns → rows. Wide → long format for easier plotting |
| `groupby().size()` | Splits data into groups, counts rows in each group |
| `reset_index()` | Converts group labels from index back into regular columns |
| `quantile(0.025)` | Value below which 2.5% of data lies — used for outlier removal |
| `np.triu()` | Returns upper triangle of a matrix as a boolean mask |
| `sns.catplot()` | High-level seaborn function for categorical bar/point plots with facets |
| `sns.heatmap()` | Visualizes a matrix (like correlation) as a color grid |
| `corr()` | Pearson correlation between all numeric columns, range -1 to 1 |

***

## ✅ Tests

All 4 unit tests pass (`test_module.py` provided by freeCodeCamp).

***

## 🏅 Certification

[freeCodeCamp – Data Analysis with Python](https://www.freecodecamp.org/learn/data-analysis-with-python/)