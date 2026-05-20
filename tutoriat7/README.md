# 🔀 Operatorul DIVISION și Vizualizări (VIEW)

---

## 1. Introducere în Operatorul DIVISION

### 1.1. Ce este DIVISION?

**DIVISION** este o operație din algebra relațională care răspunde la întrebări de tipul **"toți"**, **"toate"**, **"fiecare"**.

#### 🎯 Exemple intuitive:

| Întrebare | Traducere în algebra relațională |
|-----------|----------------------------------|
| "Găsește angajații care au lucrat pe **toate** proiectele cu buget 10000" | DIVISION |
| "Care studenți au promovat **toate** examenele din sesiune?" | DIVISION |
| "Care clienți au cumpărat **fiecare** produs din categoria X?" | DIVISION |

### 1.2. De ce este DIVISION dificil în SQL?

SQL nu are un operator direct `DIVISION`. Trebuie să **simulăm** cuantificatorul universal (`∀`) folosind cuantificatorul existențial (`∃`) și negația.

> 💡 **Regula de aur:**
> ```
> ∀x P(x)  ≡  ¬∃x ¬P(x)
> ```
> 
> **Traducere:** "Pentru toți x, P(x) este adevărat" este echivalent cu "Nu există niciun x pentru care P(x) să fie fals."

### 1.3. Aplicații practice

- **Resurse umane:** Angajați care au competențele necesare pentru toate taskurile unui proiect
- **E-commerce:** Clienți care au evaluat toate produsele dintr-o categorie
- **Educație:** Studenți care au participat la toate cursurile obligatorii
- **Logistică:** Furnizori care livrează toate tipurile de produse necesare

---

## 2. Fundamente teoretice - Cuantificatori

### 2.1. Cuantificatorul universal (∀)

**∀** = "pentru toți", "pentru fiecare"

**Exemplu:** ∀ angajat, angajatul are salariu > 0

### 2.2. Cuantificatorul existențial (∃)

**∃** = "există cel puțin un", "există"

**Exemplu:** ∃ angajat cu salariu > 20000

### 2.3. Transformarea logică

```
∀x P(x)  ≡  ¬∃x ¬P(x)
```

**Demonstrație cu exemplu:**

- **Enunț original:** "Toți angajații au lucrat pe proiectul X"
- **Transformare:** "Nu există niciun angajat care să NU fi lucrat pe proiectul X"

### 2.4. Traducerea în SQL

| Cuantificator | SQL |
|---------------|-----|
| `∀` (pentru toți) | `NOT EXISTS (... NOT EXISTS ...)` |
| `∃` (există) | `EXISTS` |
| `¬∃` (nu există) | `NOT EXISTS` |

---

## 3. Schema HR extinsă

### 3.1. Structura tabelelor

```sql
-- Tabelul EMPLOYEES (angajați)
CREATE TABLE EMPLOYEES (
    EMPLOYEE_ID    NUMBER(6)    PRIMARY KEY,
    FIRST_NAME     VARCHAR2(20),
    LAST_NAME      VARCHAR2(25) NOT NULL,
    EMAIL          VARCHAR2(25) NOT NULL,
    PHONE_NUMBER   VARCHAR2(20),
    HIRE_DATE      DATE         NOT NULL,
    JOB_ID         VARCHAR2(10) NOT NULL,
    SALARY         NUMBER(8,2),
    COMMISSION_PCT NUMBER(2,2),
    MANAGER_ID     NUMBER(6),
    DEPARTMENT_ID  NUMBER(4)
);

-- Tabelul PROJECT (proiecte)
CREATE TABLE PROJECT (
    PROJECT_ID     NUMBER(6)    PRIMARY KEY,
    PROJECT_NAME   VARCHAR2(50) NOT NULL,
    BUDGET         NUMBER(10,2),
    START_DATE     DATE,
    END_DATE       DATE,
    MANAGER_ID     NUMBER(6)    -- FK către EMPLOYEES
);

-- Tabelul WORKS_ON (relație M:N între angajați și proiecte)
CREATE TABLE WORKS_ON (
    EMPLOYEE_ID    NUMBER(6)    NOT NULL,
    PROJECT_ID     NUMBER(6)    NOT NULL,
    HOURS_WORKED   NUMBER(5,2),
    ROLE           VARCHAR2(30),
    PRIMARY KEY (EMPLOYEE_ID, PROJECT_ID),
    FOREIGN KEY (EMPLOYEE_ID) REFERENCES EMPLOYEES(EMPLOYEE_ID),
    FOREIGN KEY (PROJECT_ID) REFERENCES PROJECT(PROJECT_ID)
);
```

### 3.2. Diagrama ER

```
         EMPLOYEES
             |
             | 1
             |
             | 
             |
             | N
         WORKS_ON
             |
             | N
             |
             | 
             |
             | 1
         PROJECT
```

### 3.3. Date de exemplu

```sql
-- Inserări în EMPLOYEES
INSERT INTO EMPLOYEES (EMPLOYEE_ID, FIRST_NAME, LAST_NAME, SALARY, DEPARTMENT_ID)
VALUES (100, 'Steven', 'King', 24000, 90);

INSERT INTO EMPLOYEES (EMPLOYEE_ID, FIRST_NAME, LAST_NAME, SALARY, DEPARTMENT_ID)
VALUES (101, 'Neena', 'Kochhar', 17000, 90);

INSERT INTO EMPLOYEES (EMPLOYEE_ID, FIRST_NAME, LAST_NAME, SALARY, DEPARTMENT_ID)
VALUES (200, 'Jennifer', 'Whalen', 4400, 20);

-- Inserări în PROJECT
INSERT INTO PROJECT (PROJECT_ID, PROJECT_NAME, BUDGET)
VALUES (1, 'Website Redesign', 10000);

INSERT INTO PROJECT (PROJECT_ID, PROJECT_NAME, BUDGET)
VALUES (2, 'Mobile App', 10000);

INSERT INTO PROJECT (PROJECT_ID, PROJECT_NAME, BUDGET)
VALUES (3, 'Database Migration', 15000);

INSERT INTO PROJECT (PROJECT_ID, PROJECT_NAME, BUDGET)
VALUES (4, 'Cloud Infrastructure', 10000);

-- Inserări în WORKS_ON
-- Angajatul 100 lucrează pe TOATE proiectele cu buget 10000
INSERT INTO WORKS_ON (EMPLOYEE_ID, PROJECT_ID, HOURS_WORKED, ROLE)
VALUES (100, 1, 40, 'Team Lead');

INSERT INTO WORKS_ON (EMPLOYEE_ID, PROJECT_ID, HOURS_WORKED, ROLE)
VALUES (100, 2, 35, 'Developer');

INSERT INTO WORKS_ON (EMPLOYEE_ID, PROJECT_ID, HOURS_WORKED, ROLE)
VALUES (100, 4, 30, 'Architect');

-- Angajatul 101 lucrează pe UNELE proiecte cu buget 10000
INSERT INTO WORKS_ON (EMPLOYEE_ID, PROJECT_ID, HOURS_WORKED, ROLE)
VALUES (101, 1, 25, 'Developer');

INSERT INTO WORKS_ON (EMPLOYEE_ID, PROJECT_ID, HOURS_WORKED, ROLE)
VALUES (101, 3, 20, 'Tester');

-- Angajatul 200 lucrează doar pe proiectul 3
INSERT INTO WORKS_ON (EMPLOYEE_ID, PROJECT_ID, HOURS_WORKED, ROLE)
VALUES (200, 3, 15, 'Analyst');
```

### 3.4. Verificarea datelor

```sql
-- Proiectele cu buget 10000
SELECT PROJECT_ID, PROJECT_NAME, BUDGET
FROM PROJECT
WHERE BUDGET = 10000;

-- Rezultat: 3 proiecte (1, 2, 4)

-- Verificare: pe câte proiecte cu buget 10000 lucrează fiecare angajat
SELECT 
    e.EMPLOYEE_ID,
    e.FIRST_NAME || ' ' || e.LAST_NAME AS NUME_COMPLET,
    COUNT(w.PROJECT_ID) AS NR_PROIECTE_BUGET_10000
FROM EMPLOYEES e
LEFT JOIN WORKS_ON w ON e.EMPLOYEE_ID = w.EMPLOYEE_ID
LEFT JOIN PROJECT p ON w.PROJECT_ID = p.PROJECT_ID AND p.BUDGET = 10000
GROUP BY e.EMPLOYEE_ID, e.FIRST_NAME, e.LAST_NAME;

-- Rezultat așteptat:
-- 100 | Steven King    | 3  ← lucrează pe TOATE cele 3 proiecte
-- 101 | Neena Kochhar  | 1  ← lucrează doar pe 1 proiect
-- 200 | Jennifer Whalen| 0  ← nu lucrează pe niciun proiect cu buget 10000
```

---

## 4. Metode de implementare DIVISION

### 📌 Problema de referință

> **Să se obțină codurile angajaților atașați TUTUROR proiectelor cu buget = 10000.**

---

### Metoda 1 — Dublu `NOT EXISTS` (Simulare directă ∀)

#### 🔍 Logica pas cu pas

1. **Cererea externă:** pentru fiecare angajat din `WORKS_ON`
2. **Primul `NOT EXISTS`:** nu există niciun proiect cu budget=10000...
3. **Al doilea `NOT EXISTS`:** ...pe care angajatul respectiv să NU lucreze

#### 💻 Implementare SQL

```sql
SELECT DISTINCT employee_id
FROM works_on a
WHERE NOT EXISTS (
    -- Selectează toate proiectele cu budget = 10000
    SELECT 1
    FROM project p
    WHERE p.budget = 10000
    AND NOT EXISTS (
        -- Verifică dacă angajatul curent lucrează pe acest proiect
        SELECT 1
        FROM works_on b
        WHERE p.project_id = b.project_id
          AND b.employee_id = a.employee_id
    )
);
```

#### 📊 Explicație detaliată cu exemplu

**Pentru angajatul 100:**

1. Oracle verifică: există vreun proiect cu budget=10000 pe care angajatul 100 NU lucrează?
   - Proiectul 1: angajatul 100 lucrează? DA ✓
   - Proiectul 2: angajatul 100 lucrează? DA ✓
   - Proiectul 4: angajatul 100 lucrează? DA ✓
2. Răspuns: NU există niciun proiect pe care să nu lucreze
3. Concluzie: angajatul 100 **este inclus** în rezultat

**Pentru angajatul 101:**

1. Oracle verifică: există vreun proiect cu budget=10000 pe care angajatul 101 NU lucrează?
   - Proiectul 1: angajatul 101 lucrează? DA ✓
   - Proiectul 2: angajatul 101 lucrează? NU ✗
   - **STOP** — am găsit un proiect pe care nu lucrează
2. Răspuns: DA, există proiecte pe care nu lucrează
3. Concluzie: angajatul 101 **NU este inclus** în rezultat

#### ⚙️ Variante de optimizare

```sql
-- Variantă cu alias mai explicite
SELECT DISTINCT angajat.employee_id
FROM works_on angajat
WHERE NOT EXISTS (
    SELECT 1
    FROM project proiect_tinta
    WHERE proiect_tinta.budget = 10000
    AND NOT EXISTS (
        SELECT 1
        FROM works_on asociere
        WHERE asociere.project_id = proiect_tinta.project_id
          AND asociere.employee_id = angajat.employee_id
    )
);
```

---

### Metoda 2 — Simulare cu `COUNT`

#### 🔍 Logica pas cu pas

1. Filtrează din `WORKS_ON` doar înregistrările pentru proiectele cu budget=10000
2. Grupează pe angajat și numără câte astfel de proiecte are fiecare
3. Păstrează angajații care au numărul egal cu **totalul** proiectelor cu budget=10000

#### 💻 Implementare SQL

```sql
SELECT employee_id
FROM works_on
WHERE project_id IN (
    SELECT project_id
    FROM project
    WHERE budget = 10000
)
GROUP BY employee_id
HAVING COUNT(project_id) = (
    SELECT COUNT(*)
    FROM project
    WHERE budget = 10000
);
```

#### 📊 Explicație detaliată

**Pasul 1 - Filtrare:**

```sql
-- Rezultatul clauzei WHERE
SELECT employee_id, project_id
FROM works_on
WHERE project_id IN (SELECT project_id FROM project WHERE budget = 10000);

-- Rezultat:
-- 100 | 1
-- 100 | 2
-- 100 | 4
-- 101 | 1
```

**Pasul 2 - Grupare și numărare:**

```sql
-- După GROUP BY și COUNT
-- 100 | 3 proiecte
-- 101 | 1 proiect
```

**Pasul 3 - Filtrare cu HAVING:**

```sql
-- Total proiecte cu budget 10000: 3
-- Păstrăm doar angajații cu COUNT = 3
-- Rezultat final: 100
```

#### ⚠️ Atenție la capcane!

```sql
-- ❌ GREȘIT - numără TOATE proiectele angajatului, nu doar cele cu buget 10000
SELECT employee_id
FROM works_on
GROUP BY employee_id
HAVING COUNT(project_id) = (
    SELECT COUNT(*) FROM project WHERE budget = 10000
);

-- Problemă: dacă angajatul are 3 proiecte în total, dar doar 1 cu buget 10000,
-- va fi totuși inclus în rezultat (pentru că 3 = 3)!

-- ✅ CORECT - filtrare înainte de grupare
SELECT employee_id
FROM works_on
WHERE project_id IN (SELECT project_id FROM project WHERE budget = 10000)
GROUP BY employee_id
HAVING COUNT(project_id) = (SELECT COUNT(*) FROM project WHERE budget = 10000);
```

#### 🎯 Variantă cu JOIN explicit

```sql
SELECT w.employee_id
FROM works_on w
JOIN project p ON w.project_id = p.project_id
WHERE p.budget = 10000
GROUP BY w.employee_id
HAVING COUNT(DISTINCT w.project_id) = (
    SELECT COUNT(*)
    FROM project
    WHERE budget = 10000
);
```

---

### Metoda 3 — Operatorul `MINUS` (Diferență de mulțimi)

#### 🔍 Logica pas cu pas

1. Generează **toate combinațiile posibile** (produs cartezian) între angajați și proiecte cu budget=10000
2. Scade din ele combinațiile care **există deja** în `WORKS_ON`
3. Rezultatul = combinații care **lipsesc** (angajați care NU acoperă toate proiectele)
4. Scade acești angajați din toți angajații → rămân cei care acoperă **toate** proiectele

#### 💻 Implementare SQL

```sql
-- Toți angajații care apar în works_on
SELECT DISTINCT employee_id
FROM works_on

MINUS

-- Angajații care NU acoperă toate proiectele (au combinații lipsă)
SELECT employee_id
FROM (
    -- Toate combinațiile posibile angajat × proiect_cu_buget_10000
    SELECT e.employee_id, p.project_id
    FROM (SELECT DISTINCT employee_id FROM works_on) e
    CROSS JOIN (SELECT project_id FROM project WHERE budget = 10000) p
    
    MINUS
    
    -- Combinațiile care există deja în works_on
    SELECT employee_id, project_id
    FROM works_on
) combinatii_lipsa;
```

#### 📊 Explicație cu date concrete

**Date inițiale:**

- Angajați în works_on: {100, 101, 200}
- Proiecte cu budget 10000: {1, 2, 4}

**Pasul 1 - Produs cartezian (toate combinațiile posibile):**

```
100 × 1
100 × 2
100 × 4
101 × 1
101 × 2
101 × 4
200 × 1
200 × 2
200 × 4
```

**Pasul 2 - Combinații existente în works_on:**

```
100 × 1 ✓
100 × 2 ✓
100 × 4 ✓
101 × 1 ✓
```

**Pasul 3 - Combinații lipsă (MINUS):**

```
101 × 2  ← lipsă
101 × 4  ← lipsă
200 × 1  ← lipsă
200 × 2  ← lipsă
200 × 4  ← lipsă
```

**Pasul 4 - Angajați cu combinații lipsă:**

```
{101, 200}
```

**Pasul 5 - Toți angajații MINUS cei cu combinații lipsă:**

```
{100, 101, 200} MINUS {101, 200} = {100}
```

#### 🎯 Variantă simplificată

```sql
-- Versiune mai compactă
SELECT DISTINCT employee_id FROM works_on
MINUS
SELECT employee_id
FROM (
    SELECT employee_id, project_id
    FROM (SELECT DISTINCT employee_id FROM works_on),
         (SELECT project_id FROM project WHERE budget = 10000)
    MINUS
    SELECT employee_id, project_id FROM works_on
);
```

---

### Metoda 4 — `NOT EXISTS` cu `MINUS` (Includere de mulțimi)

#### 🔍 Logica matematică

Verificăm relația de **includere**: `A ⊇ B`

Dacă mulțimea proiectelor angajatului **include** mulțimea proiectelor cu budget 10000, atunci:

```
B ⊆ A  ⟺  B \ A = ∅
```

**Traducere:** "Dacă diferența (proiecte_budget_10000 − proiecte_angajat) este vidă, atunci angajatul acoperă toate proiectele."

#### 💻 Implementare SQL

```sql
SELECT DISTINCT employee_id
FROM works_on a
WHERE NOT EXISTS (
    -- Proiectele cu budget = 10000
    (SELECT project_id FROM project WHERE budget = 10000)
    
    MINUS
    
    -- Proiectele pe care lucrează angajatul curent
    (SELECT project_id
     FROM works_on b
     WHERE b.employee_id = a.employee_id)
);
```

#### 📊 Explicație detaliată

**Pentru angajatul 100:**

```sql
-- Proiecte cu budget 10000: {1, 2, 4}
-- Proiecte pe care lucrează angajatul 100: {1, 2, 4, ...}
-- Diferența: {1, 2, 4} MINUS {1, 2, 4, ...} = ∅ (mulțime vidă)
-- NOT EXISTS(∅) = TRUE → angajatul 100 este inclus
```

**Pentru angajatul 101:**

```sql
-- Proiecte cu budget 10000: {1, 2, 4}
-- Proiecte pe care lucrează angajatul 101: {1, 3}
-- Diferența: {1, 2, 4} MINUS {1, 3} = {2, 4} ≠ ∅
-- NOT EXISTS({2, 4}) = FALSE → angajatul 101 NU este inclus
```

#### 🎯 Variantă cu JOIN

```sql
SELECT DISTINCT a.employee_id
FROM works_on a
WHERE NOT EXISTS (
    SELECT p.project_id
    FROM project p
    WHERE p.budget = 10000
    MINUS
    SELECT w.project_id
    FROM works_on w
    WHERE w.employee_id = a.employee_id
);
```

---

### 📊 Comparație metode — Performanță și claritate

| Metodă | Complexitate citire | Performanță | Când să folosești |
|--------|-------------------|-------------|-------------------|
| **1. Dublu NOT EXISTS** | Medie | Bună | Când vrei logică clară și directă |
| **2. COUNT** | Ușoară | Foarte bună | Când ai nevoie de claritate maximă |
| **3. MINUS dublu** | Grea | Medie | Când vrei să înveți algebra relațională |
| **4. NOT EXISTS + MINUS** | Medie-Grea | Bună | Când gândești în termeni de mulțimi |

#### 💡 Recomandări practice

- **Pentru începători:** Metoda 2 (COUNT) — cea mai intuitivă
- **Pentru interviuri:** Metoda 1 (dublu NOT EXISTS) — cea mai clasică
- **Pentru optimizare:** Testează cu `EXPLAIN PLAN` — performanța variază cu distribuția datelor
- **Pentru mentenanță:** Metoda 2 (COUNT) — cea mai ușor de înțeles de alți developeri

---

## 5. Exerciții ghidate DIVISION

### 5.1. Compararea mulțimilor — Reguli fundamentale

```
A ⊇ B  (A include B / cel puțin)     ≡  B \ A = ∅
A ⊆ B  (A inclus în B / cel mult)    ≡  A \ B = ∅
A = B  (egalitate / exact)           ≡  A \ B = ∅  ȘI  B \ A = ∅
A ∩ B ≠ ∅  (au elemente comune)      ≡  EXISTS (SELECT ... INTERSECT ...)
A ∩ B = ∅  (disjuncte)               ≡  NOT EXISTS (SELECT ... INTERSECT ...)
```

---

### 5.2. Exercițiul 1: Cel puțin (⊇)

> **Găsiți angajații care au lucrat CEL PUȚIN pe aceleași proiecte ca angajatul 200.**
>
> *(mulțimea_angajat ⊇ mulțimea_angajat_200)*

#### Soluția 1 — NOT EXISTS cu MINUS

```sql
SELECT DISTINCT a.employee_id
FROM works_on a
WHERE NOT EXISTS (
    -- Proiectele angajatului 200
    (SELECT project_id FROM works_on WHERE employee_id = 200)
    
    MINUS
    
    -- Proiectele angajatului curent
    (SELECT project_id FROM works_on WHERE employee_id = a.employee_id)
);
```

#### Explicație:

- Angajatul 200 lucrează pe proiectele {3}
- Pentru ca un angajat X să fie inclus, mulțimea lui trebuie să includă {3}
- Verificăm: {3} MINUS {proiecte_X} = ∅?

#### Soluția 2 — Dublu NOT EXISTS

```sql
SELECT DISTINCT a.employee_id
FROM works_on a
WHERE NOT EXISTS (
    SELECT 1
    FROM works_on ref
    WHERE ref.employee_id = 200
    AND NOT EXISTS (
        SELECT 1
        FROM works_on b
        WHERE b.employee_id = a.employee_id
          AND b.project_id = ref.project_id
    )
);
```

#### Soluția 3 — COUNT

```sql
SELECT w.employee_id
FROM works_on w
WHERE w.project_id IN (
    SELECT project_id FROM works_on WHERE employee_id = 200
)
GROUP BY w.employee_id
HAVING COUNT(DISTINCT w.project_id) = (
    SELECT COUNT(DISTINCT project_id)
    FROM works_on
    WHERE employee_id = 200
);
```

#### 📊 Rezultat așteptat:

Cu datele noastre:
- Angajatul 200: {3}
- Angajatul 100: {1, 2, 4} — include {3}? NU
- Angajatul 101: {1, 3} — include {3}? DA ✓

**Rezultat:** {101, 200}

---

### 5.3. Exercițiul 2: Cel mult (⊆)

> **Găsiți angajații care au lucrat CEL MULT pe aceleași proiecte ca angajatul 200.**
>
> *(mulțimea_angajat ⊆ mulțimea_angajat_200)*

#### Soluție — NOT EXISTS cu MINUS (inversare)

```sql
SELECT DISTINCT a.employee_id
FROM works_on a
WHERE NOT EXISTS (
    -- Proiectele angajatului curent
    (SELECT project_id FROM works_on WHERE employee_id = a.employee_id)
    
    MINUS
    
    -- Proiectele angajatului 200
    (SELECT project_id FROM works_on WHERE employee_id = 200)
);
```

#### Explicație:

- Verificăm: {proiecte_X} MINUS {3} = ∅?
- Dacă DA → angajatul X lucrează doar pe subset-ul {3}

#### 📊 Rezultat așteptat:

- Angajatul 100: {1, 2, 4} MINUS {3} = {1, 2, 4} ≠ ∅ → NU
- Angajatul 101: {1, 3} MINUS {3} = {1} ≠ ∅ → NU
- Angajatul 200: {3} MINUS {3} = ∅ → DA ✓

**Dacă am avea un angajat care lucrează DOAR pe proiectul 3 sau pe o submulțime a {3}, acela ar fi inclus.**

---

### 5.4. Exercițiul 3: Exact (=)

> **Găsiți angajații care au lucrat pe EXACT aceleași proiecte ca angajatul 200.**
>
> *(A = B ≡ A ⊇ B ȘI A ⊆ B)*

#### Soluție — Combinare ⊇ și ⊆

```sql
SELECT DISTINCT a.employee_id
FROM works_on a
WHERE NOT EXISTS (
    -- A ⊇ B: mulțimea_200 \ mulțimea_a = ∅
    (SELECT project_id FROM works_on WHERE employee_id = 200)
    MINUS
    (SELECT project_id FROM works_on WHERE employee_id = a.employee_id)
)
AND NOT EXISTS (
    -- A ⊆ B: mulțimea_a \ mulțimea_200 = ∅
    (SELECT project_id FROM works_on WHERE employee_id = a.employee_id)
    MINUS
    (SELECT project_id FROM works_on WHERE employee_id = 200)
)
AND a.employee_id != 200;  -- excludem angajatul de referință
```

#### Soluție alternativă — Folosind INTERSECT

```sql
-- Variantă cu INTERSECT pentru verificare de egalitate
WITH proiecte_200 AS (
    SELECT project_id FROM works_on WHERE employee_id = 200
)
SELECT DISTINCT w.employee_id
FROM works_on w
GROUP BY w.employee_id
HAVING 
    -- Același număr de proiecte
    COUNT(DISTINCT w.project_id) = (SELECT COUNT(*) FROM proiecte_200)
    AND
    -- Toate proiectele sunt în mulțimea de referință
    COUNT(DISTINCT w.project_id) = (
        SELECT COUNT(*)
        FROM works_on w2
        WHERE w2.employee_id = w.employee_id
          AND w2.project_id IN (SELECT project_id FROM proiecte_200)
    )
    AND w.employee_id != 200;
```

---

### 5.5. Exercițiul 4: DIVISION pe alte criterii

> **Găsiți angajații care au lucrat pe toate proiectele pe care a lucrat managerul lor.**

#### Date suplimentare necesare:

```sql
-- Presupunem că avem și relația manager-angajat în EMPLOYEES
ALTER TABLE EMPLOYEES ADD CONSTRAINT emp_manager_fk
    FOREIGN KEY (MANAGER_ID) REFERENCES EMPLOYEES(EMPLOYEE_ID);
```

#### Soluție:

```sql
SELECT DISTINCT w1.employee_id
FROM works_on w1
JOIN employees e ON w1.employee_id = e.employee_id
WHERE e.manager_id IS NOT NULL
AND NOT EXISTS (
    -- Proiectele managerului
    (SELECT project_id 
     FROM works_on 
     WHERE employee_id = e.manager_id)
    
    MINUS
    
    -- Proiectele angajatului curent
    (SELECT project_id 
     FROM works_on 
     WHERE employee_id = w1.employee_id)
);
```

---

### 5.6. Exercițiul 5: DIVISION cu JOIN

> **Găsiți departamentele în care toți angajații au salariu > 5000.**

#### Soluție cu dublu NOT EXISTS:

```sql
SELECT DISTINCT d.department_id, d.department_name
FROM departments d
WHERE EXISTS (
    -- Departamentul are cel puțin un angajat
    SELECT 1 FROM employees WHERE department_id = d.department_id
)
AND NOT EXISTS (
    -- Nu există angajat în departament cu salariu <= 5000
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
      AND e.salary <= 5000
);
```

#### Soluție cu agregare:

```sql
SELECT department_id, department_name
FROM departments d
WHERE department_id IN (
    SELECT department_id
    FROM employees
    GROUP BY department_id
    HAVING MIN(salary) > 5000
);
```

---

## 6. Vizualizări (VIEW) - Concepte fundamentale

### 6.1. Ce este o vizualizare?

O **vizualizare (VIEW)** este un **tabel virtual** definit printr-o interogare `SELECT`. Nu stochează date proprii — datele sunt extrase dinamic din tabelele de bază la fiecare accesare.

#### 🎯 Analogie

Gândește-te la VIEW ca la un **"filter Snapchat"** peste o fotografie:
- Fotografia originală = tabelul de bază
- Filtrul = definiția VIEW-ului
- Rezultatul = datele vizibile prin VIEW

### 6.2. De ce folosim vizualizări?

| Beneficiu | Exemplu |
|-----------|---------|
| **Securitate** | Restricționează accesul la coloane sensibile (ex: salary) |
| **Simplificare** | Ascunde JOIN-uri complexe — utilizatorul vede un tabel simplu |
| **Independență date** | Modifici structura tabelului, VIEW rămâne neschimbat |
| **Prezentări multiple** | Aceleași date, perspective diferite pentru departamente |
| **Reutilizare** | Interogări frecvente salvate ca VIEW-uri |

### 6.3. Tipuri de vizualizări

```
VIZUALIZĂRI
    │
    ├── SIMPLE
    │   ├── Bazate pe 1 tabel
    │   ├── Fără GROUP BY, DISTINCT, funcții agregat
    │   └── LMD (INSERT/UPDATE/DELETE) permis ✓
    │
    └── COMPLEXE
        ├── Bazate pe multiple tabele (JOIN)
        ├── Conțin GROUP BY, HAVING, DISTINCT
        ├── Conțin funcții agregat (SUM, AVG, COUNT etc.)
        └── LMD restricționat sau interzis ✗
```

---

## 7. Crearea și gestionarea vizualizărilor

### 7.1. Sintaxa CREATE VIEW

```sql
CREATE [OR REPLACE] [FORCE | NOFORCE] VIEW nume_vizualizare 
    [(alias_coloana1, alias_coloana2, ...)]
AS 
    subcerere
[WITH CHECK OPTION [CONSTRAINT nume_constrangere]]
[WITH READ ONLY [CONSTRAINT nume_constrangere]];
```

### 7.2. Opțiuni explicate

| Opțiune | Descriere | Exemplu de utilizare |
|---------|-----------|---------------------|
| `OR REPLACE` | Recrează VIEW-ul fără să piardă privilegiile acordate | Modificări la definiție |
| `FORCE` | Creează VIEW chiar dacă tabelele de bază nu există | Scripturi de deploy |
| `NOFORCE` | (implicit) Crearea eșuează dacă tabelele lipsesc | Dezvoltare normală |
| `WITH CHECK OPTION` | INSERT/UPDATE acceptate doar pentru linii vizibile în VIEW | Integritate date |
| `WITH READ ONLY` | Interzice orice operație LMD (INSERT/UPDATE/DELETE) | VIEW-uri de raportare |

---

### 7.3. Exemplu 1: Vizualizare simplă

```sql
-- Creăm un VIEW cu angajații din departamentul 20
CREATE VIEW emp_dept20 AS
    SELECT employee_id, first_name, last_name, salary, email
    FROM employees
    WHERE department_id = 20;

-- Interogare
SELECT * FROM emp_dept20;

-- Rezultat: doar angajații cu department_id = 20
```

#### Testare operații LMD:

```sql
-- ✅ INSERT permis (VIEW simplu)
INSERT INTO emp_dept20 (employee_id, first_name, last_name, salary, email, department_id)
VALUES (999, 'Test', 'User', 5000, 'test@example.com', 20);

-- ✅ UPDATE permis
UPDATE emp_dept20 
SET salary = salary * 1.1 
WHERE employee_id = 999;

-- ✅ DELETE permis
DELETE FROM emp_dept20 WHERE employee_id = 999;
```

---

### 7.4. Exemplu 2: VIEW cu alias pe coloane

```sql
-- Metoda 1: Alias în SELECT
CREATE VIEW emp_salarii AS
    SELECT 
        employee_id AS cod,
        first_name || ' ' || last_name AS nume_complet,
        salary * 12 AS salariu_anual,
        salary * 12 * 1.19 AS salariu_anual_brut
    FROM employees;

-- Metoda 2: Alias în definiția VIEW
CREATE VIEW emp_salarii_v2 (cod, nume_complet, salariu_anual, salariu_anual_brut) AS
    SELECT 
        employee_id,
        first_name || ' ' || last_name,
        salary * 12,
        salary * 12 * 1.19
    FROM employees;

-- Interogare
SELECT * FROM emp_salarii WHERE salariu_anual > 100000;
```

#### ⚠️ Atenție:

```sql
-- ❌ Nu poți actualiza coloane calculate
UPDATE emp_salarii SET salariu_anual = 120000 WHERE cod = 100;
-- ORA-01733: virtual column not allowed here

-- ✅ Trebuie să actualizezi tabelul de bază
UPDATE employees SET salary = 120000/12 WHERE employee_id = 100;
```

---

### 7.5. Exemplu 3: VIEW cu JOIN (vizualizare complexă)

```sql
CREATE VIEW emp_dept_view AS
    SELECT 
        e.employee_id,
        e.first_name,
        e.last_name,
        e.salary,
        e.department_id,
        d.department_name,
        d.location_id,
        l.city,
        l.country_id
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
    JOIN locations l ON d.location_id = l.location_id;

-- Interogare simplă pentru utilizatori
SELECT employee_id, first_name, department_name, city
FROM emp_dept_view
WHERE city = 'Seattle';
```

#### Key-preserved tables:

- `EMPLOYEES` este **key-preserved** (employee_id unic în VIEW)
- `DEPARTMENTS` **NU** este key-preserved (department_id se repetă)
- `LOCATIONS` **NU** este key-preserved

```sql
-- ✅ UPDATE pe tabel key-preserved (EMPLOYEES)
UPDATE emp_dept_view 
SET salary = 9000 
WHERE employee_id = 100;

-- ❌ UPDATE pe tabel non-key-preserved (DEPARTMENTS)
UPDATE emp_dept_view 
SET department_name = 'New Name' 
WHERE employee_id = 100;
-- ORA-01779: cannot modify a column which maps to a non key-preserved table
```

---

### 7.6. Exemplu 4: VIEW cu funcții agregat

```sql
CREATE VIEW dept_statistics AS
    SELECT 
        d.department_id,
        d.department_name,
        COUNT(e.employee_id) AS nr_angajati,
        AVG(e.salary) AS salariu_mediu,
        MIN(e.salary) AS salariu_minim,
        MAX(e.salary) AS salariu_maxim,
        SUM(e.salary) AS total_salarii
    FROM departments d
    LEFT JOIN employees e ON d.department_id = e.department_id
    GROUP BY d.department_id, d.department_name;

-- Interogare
SELECT * 
FROM dept_statistics 
WHERE nr_angajati > 5
ORDER BY salariu_mediu DESC;
```

#### ⚠️ Limitări:

```sql
-- ❌ Nu poți face INSERT/UPDATE/DELETE pe VIEW cu GROUP BY
INSERT INTO dept_statistics (department_id, department_name, nr_angajati)
VALUES (999, 'Test Dept', 0);
-- ORA-01732: data manipulation operation not legal on this view
```

---

### 7.7. Exemplu 5: WITH CHECK OPTION

```sql
-- VIEW pentru departamentul 20 cu WITH CHECK OPTION
CREATE VIEW emp_dept20_checked AS
    SELECT employee_id, first_name, last_name, email, salary, department_id
    FROM employees
    WHERE department_id = 20
WITH CHECK OPTION CONSTRAINT emp_dept20_ck;

-- ✅ UPDATE care păstrează department_id = 20
UPDATE emp_dept20_checked 
SET salary = 6000 
WHERE employee_id = 202;
-- Succes

-- ❌ UPDATE care schimbă department_id (scoate linia din VIEW)
UPDATE emp_dept20_checked 
SET department_id = 30 
WHERE employee_id = 202;
-- ORA-01402: view WITH CHECK OPTION where-clause violation

-- ❌ INSERT cu alt department_id
INSERT INTO emp_dept20_checked 
    (employee_id, first_name, last_name, email, department_id)
VALUES (998, 'John', 'Doe', 'jdoe@test.com', 50);
-- ORA-01402: view WITH CHECK OPTION where-clause violation

-- ✅ INSERT cu department_id = 20
INSERT INTO emp_dept20_checked 
    (employee_id, first_name, last_name, email, department_id)
VALUES (997, 'Jane', 'Smith', 'jsmith@test.com', 20);
-- Succes
```

#### Explicație WITH CHECK OPTION:

Această opțiune **forțează** ca orice INSERT sau UPDATE prin VIEW să respecte condiția din clauza WHERE a VIEW-ului. Este utilă pentru:
- **Integritatea datelor** — nu poți modifica accidental înregistrări să iasă din VIEW
- **Securitate** — utilizatorii nu pot "scăpa" de restricții prin UPDATE

---

### 7.8. Exemplu 6: WITH READ ONLY

```sql
-- VIEW doar pentru citire (raportare)
CREATE VIEW emp_salary_report AS
    SELECT 
        e.employee_id,
        e.first_name || ' ' || e.last_name AS full_name,
        e.salary,
        e.salary * 12 AS annual_salary,
        d.department_name
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
WITH READ ONLY;

-- ✅ SELECT permis
SELECT * FROM emp_salary_report WHERE annual_salary > 50000;

-- ❌ INSERT interzis
INSERT INTO emp_salary_report (employee_id, full_name, salary)
VALUES (996, 'Test User', 5000);
-- ORA-42399: cannot perform a DML operation on a read-only view

-- ❌ UPDATE interzis
UPDATE emp_salary_report SET salary = 10000 WHERE employee_id = 100;
-- ORA-42399: cannot perform a DML operation on a read-only view

-- ❌ DELETE interzis
DELETE FROM emp_salary_report WHERE employee_id = 100;
-- ORA-42399: cannot perform a DML operation on a read-only view
```

---

### 7.9. Modificarea vizualizărilor

```sql
-- Recreare cu OR REPLACE (păstrează privilegiile)
CREATE OR REPLACE VIEW emp_dept20 AS
    SELECT employee_id, first_name, last_name, salary, email, hire_date, department_id
    FROM employees
    WHERE department_id = 20;

-- Fără OR REPLACE ar trebui:
-- 1. DROP VIEW emp_dept20;
-- 2. CREATE VIEW emp_dept20 ...
-- 3. GRANT din nou privilegiile
```

---

### 7.10. Ștergerea vizualizărilor

```sql
DROP VIEW emp_dept20;

-- Verificare
SELECT view_name FROM user_views WHERE view_name = 'EMP_DEPT20';
-- 0 rows returned
```

⚠️ **Atenție:** Ștergerea unui VIEW nu afectează tabelele de bază, dar șterge definiția VIEW-ului permanent.

---

## 8. Operații LMD pe vizualizări

### 8.1. Reguli generale

#### ✅ Se pot efectua operații LMD când:

1. VIEW-ul este bazat pe **un singur tabel**
2. Nu conține:
   - Funcții grup (AVG, SUM, COUNT, MIN, MAX)
   - GROUP BY, HAVING
   - DISTINCT
   - ROWNUM
   - Operatori pe mulțimi (UNION, INTERSECT, MINUS)
3. Coloana actualizată **nu este calculată** (expresie, funcție)

#### ❌ Nu se pot efectua operații LMD când:

- VIEW-ul conține orice din restricțiile de mai sus
- VIEW-ul are `WITH READ ONLY`
- Se încearcă actualizarea unei coloane non-key-preserved într-un JOIN

---

### 8.2. Exemplu detaliat — LMD pe VIEW simplu

```sql
-- Creăm VIEW simplu
CREATE VIEW emp_it AS
    SELECT employee_id, first_name, last_name, email, salary, department_id
    FROM employees
    WHERE department_id = 60;  -- IT Department

-- ✅ INSERT
INSERT INTO emp_it (employee_id, first_name, last_name, email, salary, department_id)
VALUES (995, 'Alice', 'Johnson', 'ajohnson@test.com', 7000, 60);

-- Verificare în tabelul de bază
SELECT * FROM employees WHERE employee_id = 995;
-- Linia apare în EMPLOYEES cu department_id = 60

-- ✅ UPDATE
UPDATE emp_it 
SET salary = salary * 1.15 
WHERE employee_id = 995;

-- ✅ DELETE
DELETE FROM emp_it WHERE employee_id = 995;

-- Verificare
SELECT * FROM employees WHERE employee_id = 995;
-- 0 rows (șters și din tabelul de bază)
```

---

### 8.3. Capcană: INSERT fără WITH CHECK OPTION

```sql
-- VIEW fără WITH CHECK OPTION
CREATE VIEW emp_it_no_check AS
    SELECT employee_id, first_name, last_name, email, salary, department_id
    FROM employees
    WHERE department_id = 60;

-- ⚠️ INSERT cu alt department_id (diferit de 60)
INSERT INTO emp_it_no_check 
    (employee_id, first_name, last_name, email, salary, department_id)
VALUES (994, 'Bob', 'Williams', 'bwilliams@test.com', 6500, 50);
-- Succes! Dar...

-- Verificare în VIEW
SELECT * FROM emp_it_no_check WHERE employee_id = 994;
-- 0 rows! (pentru că department_id = 50, nu 60)

-- Verificare în tabelul de bază
SELECT * FROM employees WHERE employee_id = 994;
-- 1 row (linia există, dar nu e vizibilă în VIEW)
```

#### 💡 Soluție: WITH CHECK OPTION

```sql
CREATE OR REPLACE VIEW emp_it_checked AS
    SELECT employee_id, first_name, last_name, email, salary, department_id
    FROM employees
    WHERE department_id = 60
WITH CHECK OPTION;

-- ❌ INSERT cu department_id != 60
INSERT INTO emp_it_checked 
    (employee_id, first_name, last_name, email, salary, department_id)
VALUES (993, 'Charlie', 'Brown', 'cbrown@test.com', 6000, 50);
-- ORA-01402: view WITH CHECK OPTION where-clause violation
```

---

### 8.4. LMD pe VIEW cu JOIN — Key-preserved

```sql
-- VIEW cu JOIN
CREATE VIEW emp_dept_loc AS
    SELECT 
        e.employee_id,      -- din EMPLOYEES (key-preserved)
        e.first_name,       -- din EMPLOYEES
        e.last_name,        -- din EMPLOYEES
        e.salary,           -- din EMPLOYEES
        d.department_id,    -- din DEPARTMENTS (non-key-preserved)
        d.department_name,  -- din DEPARTMENTS
        l.city              -- din LOCATIONS (non-key-preserved)
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
    JOIN locations l ON d.location_id = l.location_id;

-- ✅ UPDATE pe coloană din tabel key-preserved (EMPLOYEES)
UPDATE emp_dept_loc 
SET salary = 10000 
WHERE employee_id = 100;
-- Succes

-- ❌ UPDATE pe coloană din tabel non-key-preserved (DEPARTMENTS)
UPDATE emp_dept_loc 
SET department_name = 'New IT' 
WHERE employee_id = 100;
-- ORA-01779: cannot modify a column which maps to a non key-preserved table

-- Explicație: department_id poate apărea de multe ori în VIEW
-- (mai mulți angajați în același departament)
-- Oracle nu știe ce înregistrare din DEPARTMENTS să actualizeze
```

#### Soluție: UPDATE direct pe tabelul de bază

```sql
-- ✅ UPDATE direct
UPDATE departments 
SET department_name = 'New IT' 
WHERE department_id = (
    SELECT department_id 
    FROM employees 
    WHERE employee_id = 100
);
```

---

### 8.5. Verificarea coloanelor actualizabile

```sql
-- Dicționar: USER_UPDATABLE_COLUMNS
SELECT column_name, updatable, insertable, deletable
FROM user_updatable_columns
WHERE table_name = 'EMP_DEPT_LOC';

-- Rezultat exemplu:
-- EMPLOYEE_ID   | YES | YES | YES
-- FIRST_NAME    | YES | YES | YES
-- SALARY        | YES | YES | YES
-- DEPARTMENT_ID | NO  | NO  | NO    ← non-key-preserved
-- DEPARTMENT_NAME| NO | NO  | NO    ← non-key-preserved
-- CITY          | NO  | NO  | NO    ← non-key-preserved
```

---

## 9. Vizualizări avansate

### 9.1. VIEW cu subcereri

```sql
-- Angajații cu salariu peste media departamentului lor
CREATE VIEW emp_above_avg AS
    SELECT 
        e.employee_id,
        e.first_name,
        e.last_name,
        e.salary,
        e.department_id,
        (SELECT AVG(salary) 
         FROM employees 
         WHERE department_id = e.department_id) AS dept_avg_salary,
        e.salary - (SELECT AVG(salary) 
                    FROM employees 
                    WHERE department_id = e.department_id) AS diff_from_avg
    FROM employees e
    WHERE e.salary > (
        SELECT AVG(salary)
        FROM employees
        WHERE department_id = e.department_id
    );

-- Interogare
SELECT * FROM emp_above_avg ORDER BY diff_from_avg DESC;
```

---

### 9.2. VIEW recursiv (bazat pe alt VIEW)

```sql
-- VIEW 1: Statistici departamente
CREATE VIEW dept_stats AS
    SELECT 
        department_id,
        COUNT(*) AS nr_emp,
        AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id;

-- VIEW 2: bazat pe VIEW 1
CREATE VIEW large_depts AS
    SELECT *
    FROM dept_stats
    WHERE nr_emp >= 5;

-- Interogare
SELECT * FROM large_depts ORDER BY avg_sal DESC;
```

⚠️ **Atenție:** Dacă ștergi `dept_stats`, `large_depts` devine **invalid**.

```sql
-- Verificare VIEW-uri invalide
SELECT view_name, status FROM user_views;
-- DEPT_STATS  | VALID
-- LARGE_DEPTS | VALID

DROP VIEW dept_stats;

SELECT view_name, status FROM user_views;
-- LARGE_DEPTS | INVALID

-- Recompilare (dacă recreăm dept_stats)
ALTER VIEW large_depts COMPILE;
```

---

### 9.3. VIEW cu CASE

```sql
-- Clasificare angajați după salariu
CREATE VIEW emp_salary_class AS
    SELECT 
        employee_id,
        first_name || ' ' || last_name AS full_name,
        salary,
        CASE
            WHEN salary < 5000 THEN 'Entry Level'
            WHEN salary BETWEEN 5000 AND 10000 THEN 'Mid Level'
            WHEN salary BETWEEN 10001 AND 15000 THEN 'Senior'
            ELSE 'Executive'
        END AS salary_class
    FROM employees;

-- Interogare
SELECT salary_class, COUNT(*) AS count
FROM emp_salary_class
GROUP BY salary_class
ORDER BY 
    CASE salary_class
        WHEN 'Entry Level' THEN 1
        WHEN 'Mid Level' THEN 2
        WHEN 'Senior' THEN 3
        WHEN 'Executive' THEN 4
    END;
```

---

## 10. Probleme practice

### 🎯 Secțiunea DIVISION

#### Problema 1 (★☆☆ - Ușoară)

**Enunț:** Găsiți angajații care au lucrat pe toate proiectele cu buget mai mare de 8000.

<details>
<summary>💡 Indicație</summary>

Adaptați Metoda 2 (COUNT) — schimbați doar condiția pentru buget.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
-- Soluție cu COUNT
SELECT employee_id
FROM works_on
WHERE project_id IN (
    SELECT project_id
    FROM project
    WHERE budget > 8000
)
GROUP BY employee_id
HAVING COUNT(DISTINCT project_id) = (
    SELECT COUNT(*)
    FROM project
    WHERE budget > 8000
);

-- SAU cu dublu NOT EXISTS
SELECT DISTINCT employee_id
FROM works_on a
WHERE NOT EXISTS (
    SELECT 1
    FROM project p
    WHERE p.budget > 8000
    AND NOT EXISTS (
        SELECT 1
        FROM works_on b
        WHERE b.project_id = p.project_id
          AND b.employee_id = a.employee_id
    )
);
```
</details>

---

#### Problema 2 (★★☆ - Medie)

**Enunț:** Găsiți angajații care au lucrat pe exact aceleași proiecte ca și angajatul cu ID-ul 101.

<details>
<summary>💡 Indicație</summary>

Folosiți relația: A = B ⟺ (A \ B = ∅) ȘI (B \ A = ∅)
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT DISTINCT a.employee_id
FROM works_on a
WHERE a.employee_id != 101
AND NOT EXISTS (
    -- Proiectele lui 101 MINUS proiectele lui a
    (SELECT project_id FROM works_on WHERE employee_id = 101)
    MINUS
    (SELECT project_id FROM works_on WHERE employee_id = a.employee_id)
)
AND NOT EXISTS (
    -- Proiectele lui a MINUS proiectele lui 101
    (SELECT project_id FROM works_on WHERE employee_id = a.employee_id)
    MINUS
    (SELECT project_id FROM works_on WHERE employee_id = 101)
);
```
</details>

---

#### Problema 3 (★★☆ - Medie)

**Enunț:** Găsiți angajații care au lucrat pe cel puțin 3 proiecte diferite.

<details>
<summary>💡 Indicație</summary>

Aceasta NU este o problemă de DIVISION — este o problemă de agregare simplă.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT employee_id
FROM works_on
GROUP BY employee_id
HAVING COUNT(DISTINCT project_id) >= 3;
```
</details>

---

#### Problema 4 (★★★ - Dificilă)

**Enunț:** Găsiți departamentele în care toți angajații au lucrat pe cel puțin un proiect.

<details>
<summary>💡 Indicație</summary>

Combinați DIVISION cu verificare de existență în WORKS_ON.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
-- Varianta 1: Dublu NOT EXISTS
SELECT DISTINCT d.department_id, d.department_name
FROM departments d
WHERE EXISTS (
    -- Departamentul are angajați
    SELECT 1 FROM employees WHERE department_id = d.department_id
)
AND NOT EXISTS (
    -- Nu există angajat în departament fără proiecte
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
    AND NOT EXISTS (
        SELECT 1
        FROM works_on w
        WHERE w.employee_id = e.employee_id
    )
);

-- Varianta 2: Cu COUNT
SELECT department_id
FROM employees
GROUP BY department_id
HAVING COUNT(employee_id) = COUNT(
    DISTINCT CASE 
        WHEN employee_id IN (SELECT employee_id FROM works_on)
        THEN employee_id
    END
);
```
</details>

---

#### Problema 5 (★★★ - Dificilă)

**Enunț:** Găsiți angajații care au lucrat pe mai multe proiecte decât managerul lor.

<details>
<summary>💡 Indicație</summary>

Folosiți subcereri corelate și COUNT. Aveți nevoie de EMPLOYEES.MANAGER_ID.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    (SELECT COUNT(*) FROM works_on WHERE employee_id = e.employee_id) AS nr_proiecte_angajat,
    (SELECT COUNT(*) FROM works_on WHERE employee_id = e.manager_id) AS nr_proiecte_manager
FROM employees e
WHERE e.manager_id IS NOT NULL
AND (
    SELECT COUNT(*)
    FROM works_on
    WHERE employee_id = e.employee_id
) > (
    SELECT COUNT(*)
    FROM works_on
    WHERE employee_id = e.manager_id
);
```
</details>

---

### 🎯 Secțiunea VIEW

#### Problema 6 (★☆☆ - Ușoară)

**Enunț:** Creați un VIEW care afișează doar angajații cu salariu peste 10000, incluzând coloanele: employee_id, full_name (concat first + last), salary. VIEW-ul trebuie să fie read-only.

<details>
<summary>✅ Soluție</summary>

```sql
CREATE VIEW high_earners AS
    SELECT 
        employee_id,
        first_name || ' ' || last_name AS full_name,
        salary
    FROM employees
    WHERE salary > 10000
WITH READ ONLY;

-- Test
SELECT * FROM high_earners ORDER BY salary DESC;
```
</details>

---

#### Problema 7 (★★☆ - Medie)

**Enunț:** Creați un VIEW care afișează pentru fiecare departament: department_name, numărul de angajați, salariul mediu, salariul minim și maxim. Includeți doar departamentele cu cel puțin 3 angajați.

<details>
<summary>✅ Soluție</summary>

```sql
CREATE VIEW dept_salary_stats AS
    SELECT 
        d.department_name,
        COUNT(e.employee_id) AS nr_angajati,
        ROUND(AVG(e.salary), 2) AS salariu_mediu,
        MIN(e.salary) AS salariu_minim,
        MAX(e.salary) AS salariu_maxim
    FROM departments d
    JOIN employees e ON d.department_id = e.department_id
    GROUP BY d.department_name
    HAVING COUNT(e.employee_id) >= 3;

-- Test
SELECT * FROM dept_salary_stats ORDER BY salariu_mediu DESC;
```
</details>

---

#### Problema 8 (★★☆ - Medie)

**Enunț:** Creați un VIEW care să afișeze angajații din departamentul 50, asigurându-vă că nu pot fi mutați în alt departament prin UPDATE pe VIEW (folosiți WITH CHECK OPTION).

Testați VIEW-ul încercând:
1. UPDATE valid (schimbarea salariului)
2. UPDATE invalid (schimbarea department_id)

<details>
<summary>✅ Soluție</summary>

```sql
-- Creare VIEW
CREATE VIEW emp_dept50 AS
    SELECT employee_id, first_name, last_name, email, salary, department_id
    FROM employees
    WHERE department_id = 50
WITH CHECK OPTION CONSTRAINT emp_dept50_ck;

-- Test 1: UPDATE valid
UPDATE emp_dept50 SET salary = salary * 1.1 WHERE employee_id = 124;
-- Succes

-- Test 2: UPDATE invalid
UPDATE emp_dept50 SET department_id = 60 WHERE employee_id = 124;
-- ORA-01402: view WITH CHECK OPTION where-clause violation

-- Rollback pentru a reveni la starea inițială
ROLLBACK;
```
</details>

---

#### Problema 9 (★★★ - Dificilă)

**Enunț:** Creați un VIEW complex care combină:
- Angajați cu departamentele și locațiile lor
- Numărul de proiecte pe care lucrează fiecare
- Total ore lucrate pe toate proiectele

Includeți doar angajații care au lucrat cel puțin pe un proiect.

<details>
<summary>✅ Soluție</summary>

```sql
CREATE VIEW emp_project_summary AS
    SELECT 
        e.employee_id,
        e.first_name || ' ' || e.last_name AS full_name,
        d.department_name,
        l.city,
        COUNT(DISTINCT w.project_id) AS nr_proiecte,
        NVL(SUM(w.hours_worked), 0) AS total_ore
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
    JOIN locations l ON d.location_id = l.location_id
    JOIN works_on w ON e.employee_id = w.employee_id
    GROUP BY 
        e.employee_id,
        e.first_name,
        e.last_name,
        d.department_name,
        l.city;

-- Test
SELECT * FROM emp_project_summary WHERE nr_proiecte >= 2;

-- Verificare actualizabilitate
SELECT column_name, updatable
FROM user_updatable_columns
WHERE table_name = 'EMP_PROJECT_SUMMARY';
```
</details>

---

#### Problema 10 (★★★ - Dificilă)

**Enunț:** Creați o ierarhie de VIEW-uri:
1. VIEW de bază: angajații cu departamentele lor
2. VIEW intermediar: adaugă statistici de salariu pe departament
3. VIEW final: include doar angajații cu salariu peste media departamentului

Testați interogând VIEW-ul final.

<details>
<summary>✅ Soluție</summary>

```sql
-- VIEW 1: Bază
CREATE VIEW v1_emp_dept AS
    SELECT 
        e.employee_id,
        e.first_name,
        e.last_name,
        e.salary,
        e.department_id,
        d.department_name
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id;

-- VIEW 2: Cu statistici
CREATE VIEW v2_emp_dept_stats AS
    SELECT 
        v1.*,
        (SELECT AVG(salary) 
         FROM employees 
         WHERE department_id = v1.department_id) AS dept_avg_salary,
        (SELECT MAX(salary) 
         FROM employees 
         WHERE department_id = v1.department_id) AS dept_max_salary
    FROM v1_emp_dept v1;

-- VIEW 3: Peste medie
CREATE VIEW v3_above_average AS
    SELECT *
    FROM v2_emp_dept_stats
    WHERE salary > dept_avg_salary;

-- Test
SELECT 
    employee_id,
    first_name,
    last_name,
    department_name,
    salary,
    dept_avg_salary,
    ROUND(salary - dept_avg_salary, 2) AS diff_from_avg
FROM v3_above_average
ORDER BY diff_from_avg DESC;

-- Verificare ierarhie
SELECT view_name, text 
FROM user_views 
WHERE view_name LIKE 'V__%'
ORDER BY view_name;
```
</details>

---

### 🎯 Probleme combinate DIVISION + VIEW

#### Problema 11 (★★★ - Dificilă)

**Enunț:** Creați un VIEW care afișează angajații care au lucrat pe toate proiectele din departamentul lor. VIEW-ul trebuie să includă: employee_id, full_name, department_name, nr_proiecte_departament, nr_proiecte_angajat.

<details>
<summary>💡 Indicație</summary>

Combinați DIVISION cu JOIN-uri pentru a obține informațiile cerute.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
CREATE VIEW emp_all_dept_projects AS
WITH dept_projects AS (
    -- Proiecte conduse/asociate cu fiecare departament
    SELECT DISTINCT 
        e.department_id,
        p.project_id
    FROM employees e
    JOIN works_on w ON e.employee_id = w.employee_id
    JOIN project p ON w.project_id = p.project_id
),
emp_complete AS (
    -- Angajații care au lucrat pe toate proiectele departamentului lor
    SELECT DISTINCT e.employee_id
    FROM employees e
    WHERE NOT EXISTS (
        SELECT dp.project_id
        FROM dept_projects dp
        WHERE dp.department_id = e.department_id
        MINUS
        SELECT w.project_id
        FROM works_on w
        WHERE w.employee_id = e.employee_id
    )
    AND EXISTS (
        SELECT 1 FROM dept_projects WHERE department_id = e.department_id
    )
)
SELECT 
    e.employee_id,
    e.first_name || ' ' || e.last_name AS full_name,
    d.department_name,
    (SELECT COUNT(DISTINCT project_id) 
     FROM dept_projects 
     WHERE department_id = e.department_id) AS nr_proiecte_departament,
    (SELECT COUNT(DISTINCT project_id) 
     FROM works_on 
     WHERE employee_id = e.employee_id) AS nr_proiecte_angajat
FROM emp_complete ec
JOIN employees e ON ec.employee_id = e.employee_id
JOIN departments d ON e.department_id = d.department_id;

-- Test
SELECT * FROM emp_all_dept_projects;
```
</details>

---

#### Problema 12 (★★★ - Foarte dificilă)

**Enunț:** Creați un VIEW care implementează o "clasificare" a angajaților după gradul de acoperire a proiectelor:
- "Complete" — au lucrat pe toate proiectele disponibile
- "Majority" — au lucrat pe > 50% din proiecte
- "Minority" — au lucrat pe <= 50% din proiecte

VIEW-ul trebuie să fie updatable pe coloana salary (dacă e posibil).

<details>
<summary>✅ Soluție</summary>

```sql
CREATE VIEW emp_project_coverage AS
    SELECT 
        e.employee_id,
        e.first_name,
        e.last_name,
        e.salary,
        e.department_id,
        COUNT(DISTINCT w.project_id) AS nr_proiecte_lucrate,
        (SELECT COUNT(*) FROM project) AS total_proiecte,
        ROUND(
            COUNT(DISTINCT w.project_id) * 100.0 / (SELECT COUNT(*) FROM project),
            2
        ) AS procent_acoperire,
        CASE
            WHEN COUNT(DISTINCT w.project_id) = (SELECT COUNT(*) FROM project)
                THEN 'Complete'
            WHEN COUNT(DISTINCT w.project_id) > (SELECT COUNT(*) FROM project) / 2
                THEN 'Majority'
            ELSE 'Minority'
        END AS coverage_category
    FROM employees e
    LEFT JOIN works_on w ON e.employee_id = w.employee_id
    GROUP BY e.employee_id, e.first_name, e.last_name, e.salary, e.department_id;

-- Test citire
SELECT * FROM emp_project_coverage ORDER BY procent_acoperire DESC;

-- Test UPDATE (salary e din tabelul de bază, deci updatable)
UPDATE emp_project_coverage 
SET salary = salary * 1.05 
WHERE coverage_category = 'Complete';

-- Verificare
SELECT column_name, updatable, insertable, deletable
FROM user_updatable_columns
WHERE table_name = 'EMP_PROJECT_COVERAGE';
```
</details>

---
