#Zadatak: Predviđanje lojalnosti putnika (Customer Type)

#1. Odabir target kolone
Odabrala sam kolonu 'Customer Type' kao target varijablu jer:
- Predstavlja binarni klasifikacioni problem (Loyal/Disloyal)
- Nema nedostajućih vrijednosti
- Ima poslovnu vrijednost – aviokompanije žele predvidjeti ko će im ostati vjeran

## 2. Odabir ulaznih kolona

| Kolona | Zadržana? | Obrazloženje |
|---------|----------|--------------|
| Unnamed: 0 | ❌ NE | Identifikator |
| id | ❌ NE | Identifikator |
| satisfaction | ❌ NE | Data leakage (previše povezano sa targetom) |
| Gender | ✅ DA | Može uticati na lojalnost |
| Age | ✅ DA | Starost može uticati na preference |
| Type of Travel | ✅ DA | Poslovni putnici su možda lojalniji |
| Class | ✅ DA | Biznis klasa = vjerniji kupci |
| Flight Distance | ✅ DA | Utiče na iskustvo putovanja |
| Inflight wifi service | ✅ DA | Utiče na zadovoljstvo → lojalnost |
| Inflight entertainment | ✅ DA | Kvalitet usluge utiče na lojalnost |
| Online support | ✅ DA | Kvalitet usluge utiče na lojalnost |
| Ease of Online booking | ✅ DA | Kvalitet usluge utiče na lojalnost |
| On-board service | ✅ DA | Kvalitet usluge utiče na lojalnost |
| Leg room service | ✅ DA | Kvalitet usluge utiče na lojalnost |
| Baggage handling | ✅ DA | Kvalitet usluge utiče na lojalnost |
| Checkin service | ✅ DA | Kvalitet usluge utiče na lojalnost |
| Cleanliness | ✅ DA | Kvalitet usluge utiče na lojalnost |
| Departure Delay in Minutes | ✅ DA | Kašnjenje utiče na lojalnost |
| Arrival Delay in Minutes | ✅ DA | Kašnjenje utiče na lojalnost |


#3. Početno istraživanje podataka (EDA)
Nakon učitavanja dataseta, izvršila sam sljedeće komande:

- df.shape → (103904, 25) - 103904 reda, 25 kolona
- df.head() - prvih 5 redova pokazuju mješavinu numeričkih i kategorijskih podataka
- df.info() - pokazao je da 5 kolona ima tip 'object' (tekst), ostale su numeričke
- df.isnull().sum() - samo 'Arrival Delay in Minutes' ima 310 nedostajućih vrijednosti
- df.describe() - statistički pregled numeričkih kolona

#4.Obrada nedostajućih vrijednosti (NaN)
Samo jedna kolona je imala nedostajuće vrijednosti: 'Arrival Delay in Minutes' (310).

Strategija popunjavanja:
- Numeričke kolone → medijan (otporan na outliere)
- Kategorijske kolone → mod (najčešća vrijednost)

Primjer: 'Arrival Delay in Minutes' popunjena sa 0.0 (medijan), 
'Gender' popunjen sa 'Female' (mod).

Nakon popunjavanja, df.isnull().sum().sum() vratila je 0.

#5.Encoding (pretvaranje teksta u brojeve)
Koristila sam LabelEncoder za target (Customer Type):
- Disloyal customer → 0
- Loyal customer → 1

Koristila sam One-Hot encoding (pd.get_dummies) za ostale kategorijske kolone:
- Gender, Type of Travel, Class (drop_first=True da izbjegnem multikolinearnost)

Nakon encodinga, DataFrame ima 23 numeričke kolone.


#5. Podjela na trening i test skup
Koristila sam train_test_split sa parametrima:
- test_size = 0.2 (20% za testiranje)
- random_state = 42 (reproduktivnost)
- stratify = y (očuvanje distribucije klasa)

Rezultat:
- x_train: (83123, 22)
- x_test: (20781, 22)
- y_train: (83123,)
- y_test: (20781,)

Stratify je uspješno očuvao distribuciju (81.73% lojalnih u svim skupovima).

#7.  Skaliranje (samo za Logistic Regression)
Logistic Regression zahtijeva skaliranje podataka jer je osjetljiva na različite raspone karakteristika. 
Primijenila sam StandardScaler:
- fit_transform() na trening podacima
- transform() na test podacima

Random Forest ne zahtijeva skaliranje.

#8. Izbor modela i treniranje
Odabrala sam dva modela:
1. Logistic Regression (baseline model)
   - max_iter=1000, random_state=42

2. Random Forest (složeniji model)
   - n_estimators=100, random_state=42

Oba modela su istrenirana na odgovarajućim podacima (Logistic Regression na skaliranim).

Random Forest je superiorniji po svim metrikama, posebno po recall-u (99.57%),
što znači da rijetko propušta lojalne kupce.
