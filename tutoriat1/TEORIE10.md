# 📐 Modelare Baze de Date — Diagrame, Tabele, Relații

---

## 1. Modelul Relațional — Concepte de Bază

O **bază de date relațională** organizează datele în **tabele (relații)**, formate din rânduri și coloane.

### Terminologie:

| Termen Matematic | Termen Practic | Descriere |
| :--- | :--- | :--- |
| **Relație** | Tabel | Structura care stochează datele |
| **Tuplu** | Rând / Înregistrare | O instanță a datelor |
| **Atribut** | Coloană / Câmp | O proprietate a entității |
| **Domeniu** | Tip de date | Mulțimea valorilor posibile ale unui atribut |
| **Grad** | Număr de coloane | Câte atribute are relația |
| **Cardinalitate** | Număr de rânduri | Câte tupluri are relația la un moment dat |

---

## 2. Chei în Modelul Relațional

Cheile identifică în mod unic tuplurile și stabilesc legăturile dintre tabele.

---

### 2.1. Cheie Candidată (Candidate Key)

Un set **minimal** de atribute care identifică unic fiecare tuplu.

> Exemplu: În `EMPLOYEES`, atât `EMPLOYEE_ID` cât și `EMAIL` sunt chei candidate.

---

### 2.2. Cheie Primară (Primary Key — PK)

Cheia candidată **aleasă** ca identificator principal.

- ❌ Nu poate conține `NULL`
- ❌ Nu poate conține duplicate
- ✅ Un singur tabel → o singură cheie primară

```
EMPLOYEES(EMPLOYEE_ID*, FIRST_NAME, LAST_NAME, EMAIL, SALARY)
              ^
           PRIMARY KEY
```

---

### 2.3. Cheie Externă (Foreign Key — FK)

Atribut care **referențiează cheia primară** a altui tabel (sau a aceluiași tabel — auto-relație).

- ✅ Poate conține `NULL` (relație opțională)
- ✅ Poate repeta valori
- ⚠️ Valoarea trebuie să **existe** în tabelul referențiat sau să fie `NULL`

```
EMPLOYEES.DEPARTMENT_ID  →  DEPARTMENTS.DEPARTMENT_ID
         FK                          PK
```

---

### 2.4. Cheie Supercheie (Superkey)

Orice mulțime de atribute care identifică unic tuplurile — **nu neapărat minimală**.

> Orice cheie candidată este o supercheie, dar nu invers.

---

### 2.5. Cheie Artificială (Surrogate Key)

Cheie primară **generată artificial** (ID auto-increment), fără semnificație în lumea reală.

> Folosită când nu există un identificator natural bun.

---

## 3. Integritatea Datelor

| Tip integritate | Regulă |
| :--- | :--- |
| **Entității** | Cheia primară nu poate fi NULL |
| **Referențială** | FK trebuie să referențieze o valoare existentă în tabelul părinte |
| **Domeniului** | Valorile trebuie să respecte tipul și restricțiile coloanei |
| **Definită de utilizator** | Reguli de business specifice (ex: salariu > 0) |

---

## 4. Diagrama Entitate-Relație (ERD)

**ERD (Entity-Relationship Diagram)** reprezintă vizual structura bazei de date: entități, atribute și relațiile dintre ele.

---

### 4.1. Entitate

Obiect din lumea reală despre care stocăm informații.

```
┌─────────────┐
│  EMPLOYEES  │
└─────────────┘
```

| Tip entitate | Descriere | Exemplu |
| :--- | :--- | :--- |
| **Tare** | Există independent | `EMPLOYEES`, `DEPARTMENTS` |
| **Slabă** | Depinde de altă entitate | `DEPENDENTS` față de `EMPLOYEES` |

---

### 4.2. Atribute

| Tip Atribut | Descriere | Exemplu |
| :--- | :--- | :--- |
| **Simplu** | Valoare atomică, indivizibilă | `SALARY`, `AGE` |
| **Compus** | Format din mai multe sub-atribute | `ADDRESS` (strada, oraș, cod) |
| **Derivat** | Calculat din alte atribute | `AGE` calculat din `BIRTH_DATE` |
| **Multivaluat** | Poate avea mai multe valori | `PHONE_NUMBERS` |
| **Cheie** | Identifică unic entitatea | `EMPLOYEE_ID` |

---

### 4.3. Relații între entități

O **relație** descrie asocierea dintre două sau mai multe entități.

```
EMPLOYEES ──── lucrează_în ──── DEPARTMENTS
```

---

## 5. Cardinalitatea Relațiilor

Cardinalitatea specifică **câte instanțe** ale unei entități pot fi asociate cu instanțele altei entități.

---

### 5.1. Unu-la-Unu (1:1)

O înregistrare din A → **cel mult una** din B, și viceversa.

```
EMPLOYEES (1) ──────── are ──────── (1) PASSPORT
```

**Implementare:** FK în oricare din cele două tabele.

```
EMPLOYEES(EMPLOYEE_ID*, ...)
PASSPORT(PASSPORT_ID*, EMPLOYEE_ID#, ...)
```

---

### 5.2. Unu-la-Mulți (1:N)

O înregistrare din A → **mai multe** din B. Fiecare B → un singur A.

```
DEPARTMENTS (1) ──────── conține ──────── (N) EMPLOYEES
```

**Implementare:** FK în tabelul pe partea „N".

```
DEPARTMENTS(DEPARTMENT_ID*, DEPARTMENT_NAME, ...)
EMPLOYEES(EMPLOYEE_ID*, ..., DEPARTMENT_ID#)
                                    ↑
                              FK spre DEPARTMENTS
```

---

### 5.3. Mulți-la-Mulți (M:N)

O înregistrare din A → **mai multe** din B, și invers.

```
EMPLOYEES (M) ──────── participă_la ──────── (N) PROJECTS
```

**Implementare:** Tabel de legătură (junction table / tabel asociativ).

```
EMPLOYEES ──── EMPLOYEE_PROJECTS ──── PROJECTS
(PK)         (FK_emp | FK_proj)        (PK)
```

```
EMPLOYEES(EMPLOYEE_ID*, ...)
PROJECTS(PROJECT_ID*, ...)
WORKS_ON(EMPLOYEE_ID#, PROJECT_ID#, START_DATE, ROLE)
              ↑               ↑
         FK→EMPLOYEES   FK→PROJECTS
```

> ⚠️ O relație M:N **nu poate fi implementată direct** în modelul relațional — necesită obligatoriu un tabel de legătură.

---

## 6. Notații ERD

### 6.1. Notația Chen (clasică)

```
[EMPLOYEES] ──<lucrează_în>── [DEPARTMENTS]
```

| Simbol | Semnificație |
| :--- | :--- |
| Dreptunghi | Entitate |
| Elipsă | Atribut |
| Romb | Relație |
| Linie | Conexiune |

---

### 6.2. Notația Crow's Foot (modernă — folosită în DataGrip, Lucidchart etc.)

```
DEPARTMENTS ||──o{ EMPLOYEES
```

| Simbol | Semnificație |
| :---: | :--- |
| `\|\|` | Exact unul (obligatoriu) |
| `\|o` | Zero sau unul (opțional) |
| `{` / `}` | Mai mulți (many) |
| `o{` | Zero sau mai mulți |
| `\|{` | Unul sau mai mulți |

**Exemple complete:**

```
DEPARTMENTS  ||──o{  EMPLOYEES        (1 dept → 0 sau mulți angajați)
EMPLOYEES    }o──o{  PROJECTS         (M:N cu tabel de legătură)
EMPLOYEES    ||──o|  PASSPORT         (1:1 opțional)
```

---

## 7. Schema Logică — Notație Text

Convenție standard pentru reprezentarea tabelelor și relațiilor:

```
TABEL(atribut1*, atribut2, atribut3#)
         ^                    ^
     Primary Key          Foreign Key
```

---

### Schema HR completă:

```
REGIONS
  REGION_ID*        NUMBER
  REGION_NAME       VARCHAR2(25)

COUNTRIES
  COUNTRY_ID*       CHAR(2)
  COUNTRY_NAME      VARCHAR2(40)
  REGION_ID#        NUMBER         → REGIONS.REGION_ID

LOCATIONS
  LOCATION_ID*      NUMBER(4)
  STREET_ADDRESS    VARCHAR2(40)
  CITY              VARCHAR2(30)
  COUNTRY_ID#       CHAR(2)        → COUNTRIES.COUNTRY_ID

DEPARTMENTS
  DEPARTMENT_ID*    NUMBER(4)
  DEPARTMENT_NAME   VARCHAR2(30)
  MANAGER_ID#       NUMBER(6)      → EMPLOYEES.EMPLOYEE_ID (auto-relație)
  LOCATION_ID#      NUMBER(4)      → LOCATIONS.LOCATION_ID

JOBS
  JOB_ID*           VARCHAR2(10)
  JOB_TITLE         VARCHAR2(35)
  MIN_SALARY        NUMBER(6)
  MAX_SALARY        NUMBER(6)

EMPLOYEES
  EMPLOYEE_ID*      NUMBER(6)
  FIRST_NAME        VARCHAR2(20)
  LAST_NAME         VARCHAR2(25)   NOT NULL
  EMAIL             VARCHAR2(25)   UNIQUE
  HIRE_DATE         DATE           NOT NULL
  SALARY            NUMBER(8,2)
  COMMISSION_PCT    NUMBER(2,2)
  JOB_ID#           VARCHAR2(10)   → JOBS.JOB_ID
  DEPARTMENT_ID#    NUMBER(4)      → DEPARTMENTS.DEPARTMENT_ID
  MANAGER_ID#       NUMBER(6)      → EMPLOYEES.EMPLOYEE_ID (auto-relație)

JOB_HISTORY
  EMPLOYEE_ID#*     NUMBER(6)      → EMPLOYEES.EMPLOYEE_ID
  START_DATE*       DATE
  END_DATE          DATE
  JOB_ID#           VARCHAR2(10)   → JOBS.JOB_ID
  DEPARTMENT_ID#    NUMBER(4)      → DEPARTMENTS.DEPARTMENT_ID
  PK compusă: (EMPLOYEE_ID, START_DATE)
```

---

## 8. Diagrama Vizuală — Schema HR

```
REGIONS
  ↑ 1:N
COUNTRIES
  ↑ 1:N
LOCATIONS
  ↑ 1:N
DEPARTMENTS ←─────────────────────────────────────────┐
  ↑ 1:N                                               │ MANAGER_ID (auto-relație)
EMPLOYEES ──────────────→ JOBS                        │
  │  ↑ auto-relație (MANAGER_ID → EMPLOYEE_ID)        │
  │  └──────────────────────────────────────────────→─┘
  ↑ 1:N
JOB_HISTORY
```

---

## 9. Tipuri de Relații Speciale

---

### 9.1. Relație Reflexivă (Auto-relație / Self-join)

O entitate se asociază **cu ea însăși**.

```
EMPLOYEES ──── supervizează ──── EMPLOYEES
(MANAGER_ID referențiază EMPLOYEE_ID din același tabel)
```

```
EMPLOYEES(EMPLOYEE_ID*, FIRST_NAME, ..., MANAGER_ID#)
                                               ↑
                                    FK → EMPLOYEES.EMPLOYEE_ID
```

---

### 9.2. Entitate Asociativă (Tabel de legătură cu atribute proprii)

Tabelul de legătură poate conține **atribute proprii** dincolo de cheile externe.

```
┌──────────┐    ┌──────────────────────────┐    ┌──────────┐
│ EMPLOYEES│────│      WORKS_ON            │────│ PROJECTS │
│          │    │  EMPLOYEE_ID# (PK, FK)   │    │          │
│          │    │  PROJECT_ID#  (PK, FK)   │    │          │
│          │    │  START_DATE              │    │          │
│          │    │  ROLE                    │    │          │
└──────────┘    └──────────────────────────┘    └──────────┘
```

---

### 9.3. Ierarhie (Relație IS-A / Moștenire)

O entitate este **subtip** al alteia.

```
            PERSON
           /       \
      EMPLOYEE   CUSTOMER
      /      \
  MANAGER   CLERK
```

**Implementare variante:**
- Un tabel cu coloană discriminatoare (`TYPE`)
- Tabel pentru superclasă + tabele separate pentru subclase
- Tabele complet separate pentru fiecare subclasă

---

## 10. Normalizarea Bazelor de Date

**Normalizarea** elimină redundanța și anomaliile de actualizare prin organizarea datelor în forme normale.

---

### 10.1. Dependența Funcțională

B este **dependent funcțional** de A dacă orice valoare a lui A determină o singură valoare a lui B.

```
A → B
EMPLOYEE_ID → FIRST_NAME   (ID-ul determină numele)
```

---

### 10.2. Forma Normală 1 (1NF)

**Condiții:**
- Toate atributele sunt **atomice** (indivizibile)
- Nu există grupuri repetitive
- Fiecare coloană conține valori de același tip

❌ **Înainte de 1NF (date non-atomice):**

```
| EMP_ID | PHONES                    |
|--------|---------------------------|
| 101    | 0721111111, 0722222222    |
```

✅ **După 1NF:**

```
| EMP_ID | PHONE      |
|--------|------------|
| 101    | 0721111111 |
| 101    | 0722222222 |
```

---

### 10.3. Forma Normală 2 (2NF)

**Condiții:**
- Este în **1NF**
- Fiecare atribut non-cheie depinde **complet** de întreaga cheie primară (nu de o parte din ea)

> ⚠️ Relevant doar când cheia primară este **compusă**.

❌ **Problemă — dependență parțială:**

```
Tabel: ORDER_ITEMS(ORDER_ID*, PRODUCT_ID*, PRODUCT_NAME, QUANTITY)
                                               ↑
                            depinde doar de PRODUCT_ID, nu de toată cheia
```

✅ **Soluție:** Separă `PRODUCT_NAME` în tabelul `PRODUCTS`.

---

### 10.4. Forma Normală 3 (3NF)

**Condiții:**
- Este în **2NF**
- Nu există **dependențe tranzitive** (atribut non-cheie → atribut non-cheie)

❌ **Problemă — dependență tranzitivă:**

```
EMPLOYEES: EMPLOYEE_ID → DEPARTMENT_ID → DEPARTMENT_NAME
                                               ↑
                              tranzitiv față de EMPLOYEE_ID
```

✅ **Soluție:** Mută `DEPARTMENT_NAME` în tabelul `DEPARTMENTS`.

---

### 10.5. Forma Normală Boyce-Codd (BCNF)

**Condiții:**
- Este în **3NF**
- Pentru orice dependență `A → B`, A trebuie să fie o **supercheie**

---

### Rezumat Normalizare:

```
Tabel nestructurat
      ↓
    1NF  →  date atomice, fără repetiții
      ↓
    2NF  →  fără dependențe parțiale (față de PK compusă)
      ↓
    3NF  →  fără dependențe tranzitive
      ↓
   BCNF  →  orice determinant este supercheie
```

---

## 11. Anomalii de Actualizare

Redundanța datelor provoacă **anomalii** la modificarea datelor:

| Anomalie | Descriere | Exemplu |
| :--- | :--- | :--- |
| **Inserare** | Nu poți adăuga date fără date conexe | Nu poți adăuga un departament fără angajați |
| **Ștergere** | Ștergerea unui rând pierde alte informații | Ștergând ultimul angajat, pierzi și departamentul |
| **Actualizare** | Modificarea în mai multe locuri | Schimbarea numelui departamentului în fiecare rând al angajaților |

> ✅ **Normalizarea elimină aceste anomalii.**

---

## 12. Constrângeri în Modelul Relațional

| Constrângere | Descriere | Nivel |
| :--- | :--- | :--- |
| `NOT NULL` | Coloana nu poate fi NULL | Coloană |
| `UNIQUE` | Valorile trebuie să fie unice | Coloană / Tabel |
| `PRIMARY KEY` | NOT NULL + UNIQUE | Coloană / Tabel |
| `FOREIGN KEY` | Referință la PK / UNIQUE din alt tabel | Tabel |
| `CHECK` | Condiție logică definită de utilizator | Coloană / Tabel |
| `DEFAULT` | Valoare implicită când nu se specifică una | Coloană |

---

### Comportament FK la ștergere:

| Opțiune | Efect |
| :--- | :--- |
| *(fără opțiune)* | ❌ Eroare — nu se poate șterge dacă există înregistrări copil |
| `ON DELETE CASCADE` | Șterge automat și înregistrările copil |
| `ON DELETE SET NULL` | Setează FK-ul la NULL în înregistrările copil |

---

## 13. Diferența dintre Modelele de Date

| Model | Descriere | Exemple SGBD |
| :--- | :--- | :--- |
| **Relațional** | Date în tabele cu relații | Oracle, MySQL, PostgreSQL, SQL Server |
| **Ierarhic** | Structură arbore (părinte → copil) | IBM IMS |
| **Rețea** | Extensie ierarhic, mai mulți părinți | IDMS |
| **Obiect-relațional** | OOP + relațional | PostgreSQL, Oracle |
| **NoSQL document** | Documente JSON/BSON fără schemă fixă | MongoDB |
| **NoSQL key-value** | Perechi cheie-valoare | Redis |
| **NoSQL graf** | Noduri și muchii | Neo4j |

---

## 14. Tranzacții și Proprietăți ACID

O **tranzacție** este o unitate logică de lucru formată din una sau mai multe operații.

| Proprietate | Descriere |
| :--- | :--- |
| **Atomicitate** | Totul sau nimic — fie toate operațiile reușesc, fie niciuna |
| **Consistență** | Baza de date trece dintr-o stare validă în alta validă |
| **Izolare** | Tranzacțiile concurente nu interferează una cu alta |
| **Durabilitate** | Odată confirmate (COMMIT), modificările sunt permanente |

```
START TRANSACTION
  → Scade 1000 RON din contul A
  → Adaugă 1000 RON în contul B
COMMIT   ← ambele reușesc
           sau
ROLLBACK ← niciuna nu se aplică
```

---

## 15. Vederi (VIEW) și Indecși

---

### 15.1. Vedere (View)

O **vedere** este o interogare stocată care se comportă ca un tabel virtual.

```
VIEW: EMPLOYEES_PUBLIC
  → Afișează: FIRST_NAME, JOB_ID, DEPARTMENT_ID
  → Ascunde:  SALARY, EMAIL, PHONE_NUMBER
```

- ✅ Simplifică interogările complexe
- ✅ Restricționează accesul la date sensibile
- ✅ Nu stochează date fizic (de obicei)

---

### 15.2. Index

O **structură auxiliară** care accelerează căutările.

- ✅ Îmbunătățește performanța `SELECT` pe coloanele indexate
- ⚠️ Încetinește `INSERT`, `UPDATE`, `DELETE` (indexul trebuie actualizat)
- ✅ Creat automat pe **cheia primară** și coloanele `UNIQUE`

---

## 16. Glosar Rapid

| Termen | Definiție |
| :--- | :--- |
| **Entitate** | Obiect din lumea reală reprezentat în BD |
| **Atribut** | Proprietate a unei entități |
| **Tuplu** | Un rând dintr-un tabel |
| **Relație** | Tabel în modelul relațional |
| **PK (Primary Key)** | Identificator unic al unui rând; NOT NULL + UNIQUE |
| **FK (Foreign Key)** | Referință la PK-ul altui tabel |
| **ERD** | Diagramă Entitate-Relație |
| **Normalizare** | Proces de eliminare a redundanței |
| **1NF / 2NF / 3NF** | Forme normale de organizare a datelor |
| **ACID** | Proprietățile unei tranzacții corecte |
| **Cardinalitate (relație)** | Tipul relației: 1:1, 1:N, M:N |
| **Cardinalitate (tabel)** | Numărul de rânduri dintr-un tabel |
| **Grad** | Numărul de coloane dintr-un tabel |
| **Integritate referențială** | FK trebuie să referențieze o valoare existentă în PK |
| **View** | Tabel virtual bazat pe o interogare |
| **Index** | Structură auxiliară pentru accelerarea căutărilor |
| **Tabel asociativ** | Tabel de legătură pentru relații M:N |
| **Cheie candidată** | Set minimal de atribute care identifică unic un tuplu |
| **Supercheie** | Set (nu neapărat minimal) care identifică unic un tuplu |
| **Cheie artificială** | ID generat artificial, fără semnificație în lumea reală |
| **Dependență funcțională** | A → B: valoarea lui A determină unic valoarea lui B |
| **Dependență tranzitivă** | A → B → C (C depinde tranzitiv de A prin B) |
