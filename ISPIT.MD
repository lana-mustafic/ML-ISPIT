# Zadatak: Predviđanje lojalnosti putnika (Customer Type)

## 1. Odabir target kolone

Za target kolonu sam odabrala 'Customer Type' iz sljedećih razloga:
- Profesorica je navela da 'satisfaction' ne smijemo koristiti jer je previše očigledan
- 'Customer Type' predstavlja binarni klasifikacioni problem (Loyal customer vs Disloyal customer)
- Ova kolona nema nedostajućih vrijednosti
- Postoji logična veza između ostalih kolona (ocjene usluga, kašnjenja, klasa putovanja) 
  i toga da li će putnik ostati lojalan
- Sa poslovne perspektive, aviokompanijama je važno da predvide ko će im biti lojalni kupci

Napomena: Prvobitno sam htjela koristiti 'satisfaction', ali mi je to zabranjeno, 
pa sam odabrala sljedeći najlogičniji target.

---

## 2. Odabir ulaznih kolona

| Kolona | Zadržana? | Obrazloženje |
|---------|----------|--------------|
| Unnamed: 0 | ❌ NE | Predstavlja identifikator i nema prediktivnu vrijednost |
| id | ❌ NE | Jedinstveni identifikator putnika |
| satisfaction | ❌ NE | Može uzrokovati data leakage jer je veoma povezano sa lojalnošću korisnika |
| Gender | ✅ DA | Spol može uticati na korisničke preference |
| Age | ✅ DA | Starost može uticati na obrasce ponašanja putnika |
| Type of Travel | ✅ DA | Poslovni putnici često pokazuju veću lojalnost |
| Class | ✅ DA | Putnici u poslovnoj klasi su često lojalniji |
| Flight Distance | ✅ DA | Dužina leta može uticati na ukupno iskustvo putovanja |
| Inflight wifi service | ✅ DA | Kvalitet usluge utiče na zadovoljstvo i lojalnost |
| Inflight entertainment | ✅ DA | Kvalitet usluge utiče na zadovoljstvo i lojalnost |
| Online support | ✅ DA | Kvalitet korisničke podrške utiče na lojalnost |
| Ease of Online booking | ✅ DA | Jednostavnost rezervacije može povećati lojalnost korisnika |
| On-board service | ✅ DA | Kvalitet usluge tokom leta utiče na korisničko iskustvo |
| Leg room service | ✅ DA | Udobnost tokom leta može uticati na lojalnost |
| Baggage handling | ✅ DA | Kvalitet rukovanja prtljagom utiče na zadovoljstvo korisnika |
| Checkin service | ✅ DA | Efikasnost prijave može uticati na iskustvo putnika |
| Cleanliness | ✅ DA | Čistoća direktno utiče na percepciju kvaliteta usluge |
| Departure Delay in Minutes | ✅ DA | Kašnjenja mogu negativno uticati na lojalnost |
| Arrival Delay in Minutes | ✅ DA | Kašnjenja mogu negativno uticati na lojalnost |



Nakon odabira target kolone 'Customer Type', pristupila sam odabiru ulaznih kolona:

Izbačene kolone i obrazloženje:

1. 'Unnamed: 0' i 'id' – identifikatori koji ne nose informaciju o lojalnosti putnika. 
   Njihovim uklanjanjem izbjegavam data leakage.

2. 'satisfaction' – iako na prvi pogled izgleda korisno, ova kolona je visoko korelirana 
   sa targetom. Zadovoljni putnici su gotovo uvijek lojalni, a nezadovoljni često nelojalni. 
   Zadržavanje ove kolone dovelo bi do lažno visoke tačnosti (preko 95%) i model ne bi 
   naučio prave veze među ostalim karakteristikama.

Zadržane kolone:
- Sve demografske: Gender, Age
- Karakteristike putovanja: Type of Travel, Class, Flight Distance
- Sve ocjene usluga (0-5): Inflight wifi service, Online boarding, Seat comfort, itd.
- Vremena kašnjenja: Departure Delay in Minutes, Arrival Delay in Minutes

Napomena: Kolone sa ocjenama zadržavam jer pretpostavljam da lojalni putnici generalno 
daju više ocjene, ali to model treba da dokaže.

Izbacujemo kolone iz tri glavna razloga:

Razlog	Objašnjenje	Primjer u tvom datasetu
1. Identifikatori	Kolone koje jedinstveno označavaju svaki red, ne nose informaciju za predviđanje	Unnamed: 0, id
2. Data leakage (curenje informacija)	Kolone koje "otkrivaju" target vrijednost prije nego što model uči – daju lažno visoku tačnost	satisfaction (jer znaš da je zadovoljan = lojalan)

4. Izbacivanje nepotrebnih kolona

Prije obrade podataka, izbacila sam sljedeće kolone:

1. 'Unnamed: 0' i 'id' – Ove kolone su identifikatori koji jedinstveno 
   označavaju svakog putnika. One ne sadrže informaciju koja bi mogla 
   predvidjeti da li je putnik lojalan ili ne. Njihovim uklanjanjem 
   sprječavamo model da uči na nasumičnim brojevima.

2. 'satisfaction' – Ovu kolonu izbacujem zbog data leakage. 
   Zadovoljstvo putnika je direktno povezano sa lojalnošću – 
   zadovoljni putnici su gotovo uvijek lojalni. Da je zadržim, 
   model bi postigao vještački visoku tačnost (preko 95%) bez 
   učenja pravih veza među ostalim karakteristikama.

Nakon izbacivanja, dataset ima 22 kolone (od originalnih 25). 
Sve ostale kolone su zadržane jer potencijalno utiču na Customer Type.
---

## 3. Početna analiza podataka (EDA)

Nakon učitavanja dataseta izvršena je osnovna eksploratorna analiza podataka:

- `df.shape` → **(103904, 25)**, što znači da dataset sadrži 103.904 reda i 25 kolona.
- `df.head()` → prikazuje prvih pet redova i potvrđuje prisustvo numeričkih i kategorijskih atributa.
- `df.info()` → pokazuje da dataset sadrži 5 kategorijskih (`object`) kolona, dok su ostale numeričkog tipa.
- `df.isnull().sum()` → otkriveno je da samo kolona **Arrival Delay in Minutes** sadrži nedostajuće vrijednosti (310).
- `df.describe()` → daje statistički pregled numeričkih karakteristika.

**3. Početno istraživanje podataka (EDA)**

Nakon učitavanja dataseta, izvršila sam sljedeće komande:

- df.head() - prikazala sam prvih 5 redova kako bih vidjela kako izgledaju podaci. 
  Uočila sam da postoje numeričke (Age, Flight Distance) i kategorijske kolone (Gender, Satisfaction).

- df.info() - pokazao je da dataset ima X redova i Y kolona. 
  Primijetila sam da kolone 'Arrival Delay in Minutes' ima 310 NaN vrijednosti, 
  a 'Departure Delay in Minutes' ima 52 NaN vrijednosti.

- df.isnull().sum() - potvrdila sam tačan broj nedostajućih vrijednosti:
  * Arrival Delay in Minutes: 310 nedostajućih
  * Departure Delay in Minutes: 52 nedostajućih
  * Ostale kolone nemaju NaN vrijednosti.

- df.describe() - prikazala sam osnovnu statistiku za numeričke kolone. 
  Uočila sam da se Age kreće od 7 do 85 godina, a Flight Distance od 31 do 4983 milja.

---

## 4. Obrada nedostajućih vrijednosti

Analizom je utvrđeno da samo kolona **Arrival Delay in Minutes** sadrži nedostajuće vrijednosti.

Primijenjena strategija:

- Za numeričke kolone korišten je **medijan**, jer je otporan na ekstremne vrijednosti.
- Za kategorijske kolone korišten je **mod** (najčešća vrijednost).

Primjeri:

- `Arrival Delay in Minutes` popunjena je vrijednošću **0.0** (medijan).
- `Gender` bi u slučaju nedostajućih vrijednosti bio popunjen vrijednošću **Female** (mod).

Nakon obrade:

```python
df.isnull().sum().sum()
```

Rezultat je bio:

```python
0
```

što potvrđuje da više nema nedostajućih vrijednosti.

---

## 5. Encoding (pretvaranje kategorijskih podataka u numeričke)

Za target kolonu **Customer Type** korišten je **LabelEncoder**:

- Disloyal Customer → 0
- Loyal Customer → 1

Za ostale kategorijske kolone korišten je **One-Hot Encoding** pomoću funkcije `pd.get_dummies()`:

- Gender
- Type of Travel
- Class

Parametar `drop_first=True` korišten je radi izbjegavanja multikolinearnosti.

Nakon encodinga svi atributi su postali numerički i spremni za treniranje modela.

---

## 6. Podjela na trening i test skup

Podaci su podijeljeni pomoću funkcije `train_test_split()`:

- `test_size = 0.2`
- `random_state = 42`
- `stratify = y`

Dobijeni skupovi:

| Skup | Dimenzije |
|--------|------------|
| X_train | (83123, 22) |
| X_test | (20781, 22) |
| y_train | (83123,) |
| y_test | (20781,) |

Korištenjem parametra `stratify` očuvana je originalna distribucija klasa u svim skupovima podataka.

---

## 7. Skaliranje podataka

Za model **Logistic Regression** primijenjeno je skaliranje pomoću **StandardScaler-a**, jer je ovaj model osjetljiv na različite raspone vrijednosti atributa.

Korištene metode:

- `fit_transform()` nad trening skupom
- `transform()` nad test skupom

Model **Random Forest** ne zahtijeva skaliranje, pa je treniran na originalnim vrijednostima atributa.

---

## 8. Izbor modela i treniranje

Za rješavanje problema korištena su dva klasifikacijska modela:

### Logistic Regression

Parametri:

- `max_iter = 1000`
- `random_state = 42`

Predstavlja jednostavan i interpretabilan bazni model.

### Random Forest

Parametri:

- `n_estimators = 100`
- `random_state = 42`

Predstavlja napredniji ansambl model sposoban da prepozna složenije obrasce u podacima.

---

## 9. Rezultati

Nakon treniranja i evaluacije modela, **Random Forest** ostvario je bolje rezultate po svim posmatranim metrikama, posebno po vrijednosti:

- **Recall = 99.57%**

To znači da model vrlo uspješno prepoznaje lojalne korisnike i rijetko ih pogrešno klasificira.

### Zaključak

Na osnovu dobijenih rezultata, **Random Forest** se pokazao kao najuspješniji model za predviđanje lojalnosti putnika. Model može biti koristan aviokompanijama za identifikaciju korisnika sa visokim potencijalom za dugoročnu lojalnost i planiranje odgovarajućih marketinških strategija.
