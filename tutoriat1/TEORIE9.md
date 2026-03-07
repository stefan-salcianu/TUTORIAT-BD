# 🔀 Operatorul DIVISION și Vizualizări (VIEW)

---

## 1. Operatorul DIVISION

### 1.1. Ce este DIVISION?

**DIVISION** este o operație relațională care returnează valorile atributelor dintr-o relație care apar în **toate** valorile atributelor din cealaltă relație.

> 💡 **Exemplu intuitiv:** „Găsește angajații care au lucrat pe **toate** proiectele cu buget 10000."

Operatorul DIVISION este legat de **cuantificatorul universal** (`∀`) care **nu există direct în SQL**. Se simulează folosind cuantificatorul existențial (`∃`) prin relația:

```
∀x P(x)  ≡  ¬∃x ¬P(x)
```

> 💡 „Pentru toți x, P(x) este adevărat" = „Nu există niciun x pentru care P(x) să fie fals."

---

### 1.2. Schema extinsă HR — tabele folosite

```
EMPLOYEES (EMPLOYEE_ID*, ...)
PROJECT   (PROJECT_ID*, BUDGET, ...)
WORKS_ON  (EMPLOYEE_ID#, PROJECT_ID#)   ← tabel asociativ M:N
```

- Relația **`angajat lucrează în proiect`** → M:N → tabel `WORKS_ON`
- Relația **`angajat conduce proiect`** → 1:N

---

### 1.3. Problema de referință

> **Să se obțină codurile angajaților atașați tuturor proiectelor cu buget = 10000.**

---

### Metoda 1 — Dublu `NOT EXISTS` (implementarea directă a cuantificatorului universal)

```sql
-- ∀ proiect cu budget=10000, angajatul lucrează pe acel proiect
-- ≡ NU EXISTĂ proiect cu budget=10000 pe care angajatul NU lucrează
SELECT DISTINCT employee_id
FROM works_on a
WHERE NOT EXISTS (
    SELECT 1
    FROM project p
    WHERE p.budget = 10000
    AND NOT EXISTS (
        SELECT 1
        FROM works_on b
        WHERE p.project_id = b.project_id
          AND b.employee_id = a.employee_id
    )
);
```

**Logică:**
- Cererea externă: pentru fiecare angajat din `works_on`
- Primul `NOT EXISTS`: nu există niciun proiect cu budget=10000...
- Al doilea `NOT EXISTS`: ...pe care angajatul respectiv să NU lucreze

---

### Metoda 2 — Simulare cu `COUNT`

```sql
-- Angajatul trebuie să lucreze pe EXACT ATÂTEA proiecte cu budget=10000
-- câte proiecte cu budget=10000 există în total
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

**Logică:**
1. Filtrează din `works_on` doar înregistrările pentru proiectele cu budget=10000.
2. Grupează pe angajat și numără câte proiecte are fiecare.
3. Păstrează angajații care au numărul egal cu **totalul** proiectelor cu budget=10000.

---

### Metoda 3 — Operatorul `MINUS`

```sql
-- Toți angajații din works_on
-- MINUS
-- Angajații care NU acoperă toate proiectele (există combinație angajat-proiect lipsă)
SELECT DISTINCT employee_id
FROM works_on
MINUS
SELECT employee_id
FROM (
    -- Toate combinațiile posibile angajat × proiect_budget10000
    SELECT employee_id, project_id
    FROM (SELECT employee_id FROM works_on) t1,
         (SELECT project_id FROM project WHERE budget = 10000) t2
    MINUS
    -- Combinațiile care există deja în works_on
    SELECT employee_id, project_id
    FROM works_on
) t3;
```

**Logică:**
- Generează toate combinațiile posibile (produs cartezian) angajat × proiect_cu_budget_10000.
- Scade din ele combinațiile care există deja în `works_on`.
- Dacă rămâne ceva → angajatul NU acoperă toate proiectele → îl excludem cu `MINUS` extern.

---

### Metoda 4 — `NOT EXISTS` cu `MINUS` (A include B → B\A = ∅)

```sql
-- Ideea: mulțimea_proiectelor_angajat include mulțimea_proiectelor_budget10000
-- ≡ (proiecte_budget10000 MINUS proiecte_angajat) = ∅
SELECT DISTINCT employee_id
FROM works_on a
WHERE NOT EXISTS (
    -- Proiectele cu budget=10000
    (SELECT project_id FROM project WHERE budget = 10000)
    MINUS
    -- Proiectele pe care lucrează angajatul curent
    (SELECT project_id
     FROM project p, works_on b
     WHERE p.project_id = b.project_id
       AND b.employee_id = a.employee_id)
);
```

**Logică:**
- `NOT EXISTS` verifică că diferența (proiecte_budget10000 − proiecte_angajat) este **vidă**.
- Dacă diferența e vidă → angajatul acoperă **toate** proiectele cu budget=10000.

---

### 1.4. Comparație metode

| Metodă | Mecanism | Complexitate |
| :--- | :--- | :--- |
| **1** — Dublu NOT EXISTS | Simulare directă `∀` → `¬∃¬` | Clasică, ușor de înțeles logic |
| **2** — COUNT | Numărare și comparare | Simplă, intuitivă |
| **3** — MINUS dublu | Diferență de mulțimi, produs cartezian | Mai complexă ca structură |
| **4** — NOT EXISTS + MINUS | `A ⊇ B ≡ B \ A = ∅` | Elegantă, combină operatori |

---

### 1.5. Variante de exerciții Division

```sql
-- a) Angajații care au lucrat CEL PUȚIN pe aceleași proiecte ca angajatul 200
--    (mulțimea angajat ⊇ mulțimea angajat_200)
SELECT DISTINCT a.employee_id
FROM works_on a
WHERE NOT EXISTS (
    (SELECT project_id FROM works_on WHERE employee_id = 200)
    MINUS
    (SELECT project_id FROM works_on WHERE employee_id = a.employee_id)
);

-- b) Angajații care au lucrat CEL MULT pe aceleași proiecte ca angajatul 200
--    (mulțimea angajat ⊆ mulțimea angajat_200)
SELECT DISTINCT a.employee_id
FROM works_on a
WHERE NOT EXISTS (
    (SELECT project_id FROM works_on WHERE employee_id = a.employee_id)
    MINUS
    (SELECT project_id FROM works_on WHERE employee_id = 200)
);

-- c) Angajații care au lucrat pe EXACT aceleași proiecte ca angajatul 200
--    (A = B ≡ A-B = ∅ și B-A = ∅)
SELECT DISTINCT a.employee_id
FROM works_on a
WHERE NOT EXISTS (
    (SELECT project_id FROM works_on WHERE employee_id = 200)
    MINUS
    (SELECT project_id FROM works_on WHERE employee_id = a.employee_id)
)
AND NOT EXISTS (
    (SELECT project_id FROM works_on WHERE employee_id = a.employee_id)
    MINUS
    (SELECT project_id FROM works_on WHERE employee_id = 200)
);
```

> 💡 **Reguli pentru compararea mulțimilor:**
> ```
> A ⊇ B  (A include B)  ≡  B \ A = ∅
> A ⊆ B  (A inclus în B) ≡  A \ B = ∅
> A = B  (egalitate)     ≡  A \ B = ∅ și B \ A = ∅
> ```

---

## 2. Vizualizări (VIEW)

### 2.1. Ce este o vizualizare?

O **vizualizare** este un **tabel virtual** construit pe baza unor tabele sau a altor vizualizări (numite **tabele de bază**). Vizualizarea nu conține date proprii — **reflectă** datele din tabelele de bază.

> 💡 Vizualizările mai sunt numite și **cereri stocate** — sunt definite de o instrucțiune `SELECT` salvată.

### Avantaje:
- **Restricționarea accesului** la date — utilizatorul vede doar coloanele permise
- **Simplificarea** cererilor complexe — ascunde join-uri și logică complicată
- **Independența datelor** față de programele de aplicații
- **Prezentarea** de imagini diferite asupra acelorași date

---

### 2.2. Crearea vizualizărilor — `CREATE VIEW`

```sql
CREATE [OR REPLACE] [FORCE | NOFORCE] VIEW nume_vizualizare [(alias1, alias2, ...)]
AS subcerere
[WITH CHECK OPTION [CONSTRAINT nume_constrangere]]
[WITH READ ONLY [CONSTRAINT nume_constrangere]];
```

| Opțiune | Descriere |
| :--- | :--- |
| `OR REPLACE` | Modifică definiția fără a mai reacorda privilegiile |
| `FORCE` | Creează vizualizarea chiar dacă tabelele de bază nu există (ignoră erorile) |
| `NOFORCE` | Implicit — crearea eșuează dacă tabelele de bază nu există |
| `WITH CHECK OPTION` | Permite INSERT/UPDATE prin vizualizare **doar** pentru linii accesibile vizualizării |
| `WITH READ ONLY` | Interzice **orice operație LMD** prin vizualizare |

---

### 2.3. Exemple de creare

```sql
-- Vizualizare simplă (un singur tabel, fără funcții agregat)
CREATE VIEW EMP_DEPT20 AS
    SELECT EMPLOYEE_ID, FIRST_NAME, LAST_NAME, SALARY
    FROM EMPLOYEES
    WHERE DEPARTMENT_ID = 20;

-- Vizualizare cu alias pe coloane
CREATE VIEW EMP_SAL (ID, NUME, SALARIU_ANUAL) AS
    SELECT EMPLOYEE_ID, FIRST_NAME, SALARY * 12
    FROM EMPLOYEES;

-- Vizualizare complexă (join + funcție agregat)
CREATE VIEW DEPT_STATS AS
    SELECT D.DEPARTMENT_NAME, COUNT(*) NR_ANGAJATI, AVG(E.SALARY) MEDIE_SAL
    FROM EMPLOYEES E
    JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID
    GROUP BY D.DEPARTMENT_NAME;

-- Cu WITH CHECK OPTION (INSERT/UPDATE nu pot scoate linia din vizualizare)
CREATE VIEW EMP_DEPT20_CHECK AS
    SELECT EMPLOYEE_ID, FIRST_NAME, DEPARTMENT_ID
    FROM EMPLOYEES
    WHERE DEPARTMENT_ID = 20
    WITH CHECK OPTION CONSTRAINT check_dept20;

-- Cu WITH READ ONLY (nicio operație LMD permisă)
CREATE OR REPLACE VIEW EMP_READONLY AS
    SELECT EMPLOYEE_ID, FIRST_NAME, SALARY
    FROM EMPLOYEES
    WITH READ ONLY;
```

> ⚠️ Subcererea din `CREATE VIEW` **nu poate conține** clauza `ORDER BY`.  
> ✅ Dacă se dorește ordonare, se folosește `ORDER BY` la **interogarea vizualizării**.

---

### 2.4. Vizualizări simple vs. complexe

| | Vizualizare simplă | Vizualizare complexă |
| :--- | :--- | :--- |
| **Bazată pe** | Un singur tabel | Mai multe tabele sau funcții agregat |
| **Conține** | Fără `GROUP BY`, fără funcții agregat | `GROUP BY`, `HAVING`, `JOIN`, funcții agregat |
| **Operații LMD** | ✅ De obicei permise | ⚠️ Nu întotdeauna permise |

---

### 2.5. Când NU se pot efectua operații LMD

❌ **Nu se pot efectua operații LMD** pe vizualizări care conțin:

- Funcții grup (`AVG`, `SUM`, `COUNT` etc.)
- Clauzele `GROUP BY`, `HAVING`, `START WITH`, `CONNECT BY`
- Cuvântul cheie `DISTINCT`
- Pseudocoloana `ROWNUM`
- Operatori pe mulțimi (`UNION`, `INTERSECT`, `MINUS`)

❌ **Nu se pot actualiza** coloanele:
- Ale căror valori rezultă din **calcule** sau funcția `DECODE`
- Care **nu respectă constrângerile** din tabelele de bază

---

### 2.6. Vizualizări bazate pe JOIN — tabel key-preserved

Pentru vizualizările bazate pe mai multe tabele, operațiile `INSERT`, `UPDATE`, `DELETE` pot modifica datele **doar dintr-un singur tabel de bază** — cel **key-preserved**.

> 💡 Un tabel de bază este **key-preserved** dacă fiecare valoare a cheii sale primare (sau a coloanei `UNIQUE`) este **unică și în vizualizare**.

```sql
-- În această vizualizare, EMPLOYEES este key-preserved (EMPLOYEE_ID unic în vizualizare)
-- DEPARTMENTS NU este key-preserved (DEPARTMENT_ID poate apărea de mai multe ori)
CREATE VIEW EMP_DEPT_VIEW AS
    SELECT E.EMPLOYEE_ID, E.FIRST_NAME, E.SALARY,
           D.DEPARTMENT_ID, D.DEPARTMENT_NAME
    FROM EMPLOYEES E
    JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID;

-- ✅ UPDATE pe coloana din EMPLOYEES (key-preserved)
UPDATE EMP_DEPT_VIEW SET SALARY = 9000 WHERE EMPLOYEE_ID = 100;

-- ❌ UPDATE pe coloana din DEPARTMENTS (nu este key-preserved)
UPDATE EMP_DEPT_VIEW SET DEPARTMENT_NAME = 'Test' WHERE EMPLOYEE_ID = 100;
```

> ⚠️ **Reactualizarea tabelelor de bază implică reactualizarea corespunzătoare a vizualizărilor!**

---

### 2.7. `WITH CHECK OPTION` — detalii

```sql
CREATE VIEW EMP_DEPT20 AS
    SELECT * FROM EMPLOYEES
    WHERE DEPARTMENT_ID = 20
    WITH CHECK OPTION;

-- ✅ Permis: modificare care păstrează linia în vizualizare
UPDATE EMP_DEPT20 SET SALARY = 9000 WHERE EMPLOYEE_ID = 201;

-- ❌ Eroare: modificare care ar scoate linia din vizualizare
UPDATE EMP_DEPT20 SET DEPARTMENT_ID = 30 WHERE EMPLOYEE_ID = 201;
-- Ar muta angajatul în departamentul 30, deci nu ar mai fi vizibil în EMP_DEPT20
```

---

### 2.8. Modificarea și ștergerea vizualizărilor

```sql
-- Modificare (recreare cu OR REPLACE — păstrează privilegiile)
CREATE OR REPLACE VIEW EMP_DEPT20 AS
    SELECT EMPLOYEE_ID, FIRST_NAME, SALARY, DEPARTMENT_ID
    FROM EMPLOYEES
    WHERE DEPARTMENT_ID = 20;

-- Ștergere
DROP VIEW EMP_DEPT20;
```

---

### 2.9. Consultarea dicționarului datelor

```sql
-- Vizualizările utilizatorului curent
SELECT VIEW_NAME, TEXT FROM USER_VIEWS;

-- Toate vizualizările accesibile
SELECT * FROM ALL_VIEWS;

-- Coloanele actualizabile dintr-o vizualizare
SELECT * FROM USER_UPDATABLE_COLUMNS
WHERE TABLE_NAME = 'EMP_DEPT20';
```

---

## 3. Rezumat Rapid

```
DIVISION:
  Răspunde la "toți" → simulat prin ¬∃¬ (dublu NOT EXISTS)
  Metoda 1: NOT EXISTS(NOT EXISTS(...))          → simulare directă ∀
  Metoda 2: COUNT(proiecte_angajat) = COUNT(*)   → numărare
  Metoda 3: MINUS dublu + produs cartezian       → diferență mulțimi
  Metoda 4: NOT EXISTS(B MINUS A)                → A ⊇ B ≡ B\A = ∅

  Comparare mulțimi:
    A ⊇ B  ≡  B \ A = ∅       (cel puțin)
    A ⊆ B  ≡  A \ B = ∅       (cel mult)
    A = B  ≡  A\B=∅ și B\A=∅  (exact)

VIEW:
  CREATE [OR REPLACE] VIEW v AS subcerere
  OR REPLACE         → modifică fără să piardă privilegiile
  FORCE              → creează chiar dacă tabelele nu există
  WITH CHECK OPTION  → INSERT/UPDATE menținut în granița vizualizării
  WITH READ ONLY     → interzice orice LMD

  Simplă (1 tabel, fără agregat)  → LMD permis
  Complexă (JOIN, GROUP BY etc.)  → LMD restricționat

  LMD interzis dacă: funcții grup, GROUP BY, DISTINCT, ROWNUM, operatori mulțimi
  LMD permis pe tabelul key-preserved din JOIN

  DROP VIEW v;
  Dicționar: USER_VIEWS, ALL_VIEWS, USER_UPDATABLE_COLUMNS
```

---

## 4. Capcane Frecvente ⚠️

| Situație | Problemă | Soluție |
| :--- | :--- | :--- |
| `ORDER BY` în definiția VIEW | Eroare Oracle | Pune `ORDER BY` la interogarea vizualizării |
| LMD pe vizualizare cu `GROUP BY` / `DISTINCT` | Eroare Oracle | Nu se poate — folosește tabelul de bază direct |
| `UPDATE` pe coloană calculată în VIEW | Eroare Oracle | Coloanele calculate nu sunt actualizabile |
| `UPDATE` pe tabel non-key-preserved în JOIN view | Eroare Oracle | Actualizează tabelul de bază direct |
| `WITH CHECK OPTION` — modificare scoate linia din view | Eroare Oracle | Modificarea trebuie să păstreze linia vizibilă în vizualizare |
| Division cu `COUNT` — angajat pe proiecte cu alte bugete | `COUNT` poate include proiecte nepotrivite | Filtrează cu `WHERE project_id IN (...)` înainte de `GROUP BY` |
