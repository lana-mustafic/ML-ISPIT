# Zadatak: Predviđanje lojalnosti putnika (Customer Type)

## 1. Odabir target kolone

Za target kolonu odabrala sam **Customer Type** iz sljedećih razloga:

- Predstavlja binarni klasifikacijski problem (*Loyal Customer* i *Disloyal Customer*).
- Ne sadrži nedostajuće vrijednosti.
- Postoji logična povezanost između karakteristika putovanja, kvaliteta usluge i lojalnosti korisnika.
- Sa poslovnog aspekta, aviokompanijama je važno prepoznati putnike koji će ostati vjerni njihovim uslugama.

Prvobitno sam planirala koristiti kolonu **satisfaction** kao target, međutim profesorica je naglasila da se ona ne smije koristiti jer je previše očigledno povezana sa ostalim karakteristikama i ne predstavlja dovoljno izazovan problem za model.

---

## 2. Odabir ulaznih kolona

Nakon odabira target kolone izvršena je analiza svih atributa kako bi se utvrdilo koje kolone treba zadržati, a koje ukloniti.

### Izbačene kolone

| Kolona | Razlog |
|---------|---------|
| Unnamed: 0 | Predstavlja identifikator i nema prediktivnu vrijednost |
| id | Jedinstveni identifikator putnika |
| satisfaction | Može uzrokovati data leakage jer je veoma povezana sa lojalnošću korisnika |

Kolone **Unnamed: 0** i **id** uklonjene su jer predstavljaju identifikatore. Takve kolone ne sadrže korisne informacije za predviđanje i mogu dovesti do pogrešnog učenja modela.

Kolona **satisfaction** uklonjena je zbog problema poznatog kao **data leakage**. Zadovoljstvo putnika je direktno povezano sa lojalnošću, pa bi model mogao ostvariti veoma visoku tačnost bez stvarnog učenja odnosa među ostalim karakteristikama.

### Zadržane kolone

| Kolona | Zadržana? | Obrazloženje |
|---------|----------|--------------|
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

Nakon uklanjanja nepotrebnih kolona dataset je sadržavao **22 korisne ulazne karakteristike**.

---

## 3. Početna analiza podataka (EDA)

Nakon učitavanja dataseta izvršena je osnovna eksploratorna analiza podataka.

### Analiza strukture dataseta

```python
df.shape
```

Rezultat:

```python
(103904, 25)
```

Dataset sadrži 103.904 reda i 25 kolona.

### Pregled prvih redova

```python
df.head()
```

Prikaz prvih pet redova pokazao je kombinaciju numeričkih i kategorijskih atributa.

### Informacije o kolonama

```python
df.info()
```

Analizom je utvrđeno da dataset sadrži nekoliko kategorijskih kolona tipa `object`, dok su ostale numeričkog tipa.

### Provjera nedostajućih vrijednosti

```python
df.isnull().sum()
```

Uočeno je da kolona **Arrival Delay in Minutes** sadrži 310 nedostajućih vrijednosti.

### Statistički pregled

```python
df.describe()
```

Prikazana je osnovna statistika za numeričke kolone, uključujući minimalne, maksimalne i prosječne vrijednosti.

---

## 4. Obrada nedostajućih vrijednosti

Prilikom analize podataka ustanovljeno je da samo kolona **Arrival Delay in Minutes** sadrži nedostajuće vrijednosti.

Za obradu nedostajućih podataka korištene su sljedeće strategije:

- Numeričke kolone popunjavane su medijanom.
- Kategorijske kolone popunjavane su najčešćom vrijednošću (modom).

Medijan je odabran jer je otporniji na ekstremne vrijednosti od aritmetičke sredine.

Nakon popunjavanja svih nedostajućih vrijednosti izvršena je ponovna provjera:

```python
df.isnull().sum().sum()
```

Rezultat:

```python
0
```

što potvrđuje da dataset više ne sadrži NaN vrijednosti.

---

## 5. Encoding kategorijskih podataka

Kako modeli mašinskog učenja mogu raditi samo sa numeričkim podacima, bilo je potrebno izvršiti pretvaranje kategorijskih vrijednosti u numerički oblik.

### Label Encoding target kolone

```python
Disloyal Customer -> 0
Loyal Customer -> 1
```

### One-Hot Encoding ulaznih atributa

Za ostale kategorijske kolone korišten je One-Hot Encoding:

- Gender
- Type of Travel
- Class

Korišten je parametar `drop_first=True` kako bi se izbjegla multikolinearnost.

Nakon encodinga svi atributi postali su numerički i spremni za treniranje modela.

---

## 6. Podjela na trening i test skup

Podaci su podijeljeni pomoću funkcije `train_test_split()`.

Parametri:

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

Parametar `stratify` korišten je kako bi se očuvala ista distribucija klasa u trening i test skupu.

---

## 7. Skaliranje podataka

Za model **Logistic Regression** primijenjeno je skaliranje pomoću StandardScaler-a.

```python
scaler.fit_transform(X_train)
scaler.transform(X_test)
```

Skaliranje je potrebno jer Logistic Regression koristi udaljenosti i osjetljiv je na različite raspone vrijednosti atributa.

Model **Random Forest** ne zahtijeva skaliranje pa je treniran na originalnim podacima.

---

## 8. Izbor modela i treniranje

### Logistic Regression

```python
LogisticRegression(max_iter=1000, random_state=42)
```

Predstavlja jednostavan i interpretabilan bazni model.

### Random Forest

```python
RandomForestClassifier(n_estimators=100, random_state=42)
```

Predstavlja napredniji ansambl model sposoban da prepozna složenije obrasce u podacima.

---

## 9. Rezultati i zaključak

Nakon treniranja i evaluacije modela, **Random Forest** ostvario je bolje rezultate od Logistic Regression modela.

Najznačajniji rezultat bio je:

- Recall = 99.57%

To znači da model veoma uspješno prepoznaje lojalne korisnike i rijetko ih pogrešno klasificira.

### Zaključak

Na osnovu dobijenih rezultata može se zaključiti da je Random Forest najpogodniji model za predviđanje lojalnosti putnika u ovom datasetu. Model može pomoći aviokompanijama da identifikuju korisnike sa najvećim potencijalom za dugoročnu lojalnost i na osnovu toga planiraju marketinške aktivnosti i programe zadržavanja korisnika.
