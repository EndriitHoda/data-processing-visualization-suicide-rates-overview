# Data Processing and Visualization: Suicide Rates Overview (1985-2016)

Ky repository përdoret për qëllime studimore në fushën e Përgatitjes dhe Vizualizimit të të Dhënave, për një analizë të thellë të normave të vetëvrasjeve nga viti 1985 deri në vitin 2016.

## Përshkrimi i Datasetit

Dataseti që do të përdoret është marrë nga Kaggle dhe përmban një numër të madh të dhënash me madhësi prej 2.71 MB. Ky dataset është i përbërë nga të dhëna të marra nga katër burime të ndryshme dhe është i lidhur sipas kohës dhe vendit. Ai është ndërtuar për të identifikuar sinjale që mund të jenë të lidhura me rritjen e normave të vetëvrasjeve në grupe të ndryshme demografike globalisht, duke përfshirë spektrin socio-ekonomik.

## Objektivat e Projektit

Qëllimi i këtij projekti është përgatitja dhe vizualizimi i të dhënave për të analizuar normat e vetëvrasjeve për grupe të ndryshme demografike dhe faktorë socio-ekonomikë në vite të ndryshme. Më konkretisht, projekti do të fokusohet në:

- **Norma e vetëvrasjeve sipas segmentimeve demografike**: Analiza e normave të vetëvrasjeve për grupe të ndryshme demografike, si mosha, gjinia dhe gjenerata. Kjo na ndihmon të identifikojmë cilat grupe moshe ose gjenerata janë më të ndjeshme ndaj vetëvrasjes në vite të caktuara. Gjithashtu, do të shohim nëse ka një dallim të dukshëm mes meshkujve dhe femrave brenda secilit grup.

- **Korelacioni midis GDP për capita dhe normës së vetëvrasjeve**: Analiza e lidhjes midis faktorëve ekonomikë, si GDP për capita, dhe normës së vetëvrasjeve. Krahasimi i `gdp_per_capita` me `suicides/100k pop` mund të identifikojë trende ku kushtet ekonomike lidhen me norma të ndryshme të vetëvrasjeve në vite dhe segmente të caktuara.

- **Tendencat me kalimin e kohës**: Ndjekja e ndryshimeve në normat e vetëvrasjeve gjatë viteve për të parë modelet ose ndryshimet e mundshme. Ky segmentim sipas shtetit, grupmoshës ose gjeneratës na ndihmon të kuptojmë ndikimin e ndryshimeve shoqërore apo ngjarjeve historike në shëndetin mendor me kalimin e kohës.

## Pjesëmarrësit në Projekt

Studentët që kanë marrë pjesë në këtë projekt janë:
- Endrit Hoda
- Lorik Mustafa
- Meriton Kryeziu

**Profesori**: Mërgim Hoti

Ky projekt synon të sjellë një kuptim më të thellë të faktorëve që ndikojnë në shëndetin mendor globalisht përmes përgatitjes dhe vizualizimit të të dhënave. 

## Ekzekutimi i Kodit Hap pas Hapi

Ja një shpjegim i secilës pjesë të kodit të përdorur në këtë projekt:

### 1. Importoni Libraritë dhe Fshihni Paralajmërimet
```python
import pandas as pd
import warnings

warnings.simplefilter(action='ignore', category=FutureWarning)

### 2. Mbledhja e të dhënave, definimi i tipeve të dhënave, kualiteti i të dhënave
df = pd.read_csv(r'./master.csv')

print(df.dtypes)
print(df.describe())

### 3. Riemërimi i kolonave për përdorim më të lehtë
df=df.rename(columns={'sex':'gender','gdp_per_capita ($)':'gdp_per_capita',' gdp_for_year ($) ':'gdp_for_year', 'HDI for year' : 'hdi_for_year'})

### 4. Modifikimi i tipit të dhënave për kolonën 'gdp_for_year'
df['gdp_for_year'] = df['gdp_for_year'].str.replace(',', '').astype(int)

### 5. Ndryshimi i dimensionalitetit të daraset
df = df[['country','year', 'gender', 'age', 'suicides_no','population','suicides/100k pop','gdp_for_year','gdp_per_capita','hdi_for_year']]

### 5. Menaxhimi i vlerave null
df = df.sort_values(by=['country', 'year'])

df['hdi_for_year'] = df.groupby('country')['hdi_for_year'].transform(lambda x: x.fillna(method='ffill').fillna(method='bfill'))

yearly_mean = df.groupby(['country', 'year'])['hdi_for_year'].mean().reset_index()

yearly_mean['hdi_for_year'] = yearly_mean.groupby('country')['hdi_for_year'].transform(lambda x: x.interpolate(method='linear'))

df = df.merge(yearly_mean, on=['country', 'year'], suffixes=('', '_mean'))

df['hdi_for_year'] = df['hdi_for_year'].combine_first(df['hdi_for_year_mean'])

df = df.drop(columns=['hdi_for_year_mean'])

### 6. Mostrimi i të dhënave (10%)
sampled_data = df.sample(frac=0.1)
print(sampled_data)

### 7. Validimi i vlerave duplikate, Kontrollimi i kolonave specifike
duplicates_check=['country','year','gender','age']
duplicates=df.duplicated(subset=duplicates_check)

if duplicates.any():
    print("Duplicates found. Dropping duplicates.")
    df = df.drop_duplicates()
    print("\nCleaned DataFrame:")
    print(df)
else:
    print("No duplicates found. DataFrame remains unchanged.")

### 7. Kontrollimi i tërë dataframe-it për vlera duplikate
duplicates = df.duplicated()

if duplicates.any():
    print("Duplicates found.")
else:
    print("No duplicates found.")

### 8. Transformimi i kolonave specifike
df['total_suicides'] = df.groupby('year')['suicides_no'].transform('sum')

df['suicides_to_population_ratio'] = df['suicides_no'] / df['population']

### 9. Diskretizimi i përpjestimit të vetëvrasjeve me numrin e popullesisë dhe i gdp në kategori më të përshtatshme
ratio_bins = [-1, 0, 1e-05, 2e-05, 4e-05, 6e-05, 8e-05, 1e-04]  
ratio_labels = ['None','Very Low', 'Low', 'Medium', 'High', 'Very High', 'Extreme']

df['suicides_to_population_ratio_discretize'] = pd.cut(df['suicides_to_population_ratio'], bins=ratio_bins, labels=ratio_labels)

gdp_bins = [0, 1000, 2000, 3000]
gdp_labels = ['Low', 'Medium', 'High']
df['gdp_category'] = pd.cut(df['gdp_per_capita'], bins=gdp_bins, labels=gdp_labels)

### 10. Binarizimi i kolones 'gender'
df['gender_encoded'] = df['gender'].map({'male': 1, 'female': 0})

### 11. Ruajtja e transformimeve ne nje file te ri
df.to_csv('cleaned_data.csv', index=False)