# 🌳 Cereri Ierarhice, Clauza WITH și Operatorul EXISTS

---

## 1. Cereri Ierarhice — `START WITH` / `CONNECT BY`

Cererile ierarhice permit traversarea datelor organizate **în structuri de tip arbore** (ex: relația angajat–manager din tabelul `EMPLOYEES`).

### Clauzele principale:

| Clauză | Rol |
| :--- | :--- |
| `START WITH` | Identifică **rădăcinile** arborelui (punctul de start) |
| `CONNECT BY PRIOR` | Definește **relația părinte–copil** între linii |
| `LEVEL` | Pseudocoloană — indică **adâncimea** nodului față de rădăcină |

---

### 1.1. Sintaxă generală

```sql
SELECT [LEVEL,] coloane
FROM tabel
[WHERE conditie]
START WITH conditie_radacina
CONNECT BY PRIOR conditie_legatura;
```

---

### 1.2. Operatorul `PRIOR`

`PRIOR` face referință la linia **„părinte"** din iterația curentă. Plasarea lui determină **direcția traversării**:

```
Top-down  (rădăcină → frunze): CONNECT BY PRIOR cheie_parinte = cheie_copil
Bottom-up (frunze → rădăcină): CONNECT BY PRIOR cheie_copil   = cheie_parinte
```

---

### 1.3. Traversare Top-Down (de la manager la subordonați)

```sql
-- Pornind de la angajatul cu EMPLOYEE_ID = 100 (rădăcina),
-- coboară prin ierarhie spre subordonați
SELECT LEVEL, EMPLOYEE_ID, FIRST_NAME, MANAGER_ID
FROM EMPLOYEES
START WITH EMPLOYEE_ID = 100
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID;
--                ↑ PRIOR pe EMPLOYEE_ID → valoarea părintelui
--                                       = MANAGER_ID al copilului
```

```
Rezultat (exemplu):
LEVEL | EMPLOYEE_ID | FIRST_NAME | MANAGER_ID
  1   |    100      |   Steven   |   NULL      ← rădăcina
  2   |    101      |   Neena    |   100
  2   |    102      |   Lex      |   100
  3   |    103      |   Alexander|   102
  3   |    104      |   Bruce    |   102
```

---

### 1.4. Traversare Bottom-Up (de la subordonat la manager)

```sql
-- Pornind de la angajatul cu EMPLOYEE_ID = 107,
-- urcă prin ierarhie spre rădăcină (top management)
SELECT LEVEL, EMPLOYEE_ID, FIRST_NAME, MANAGER_ID
FROM EMPLOYEES
START WITH EMPLOYEE_ID = 107
CONNECT BY PRIOR MANAGER_ID = EMPLOYEE_ID;
--                ↑ PRIOR pe MANAGER_ID → valoarea părintelui (managerul copilului)
--                                      = EMPLOYEE_ID al rândului curent
```

---

### 1.5. Pseudocoloana `LEVEL`

`LEVEL` returnează **adâncimea** nodului curent față de rădăcină:
- `LEVEL = 1` → rădăcina (linia de start)
- `LEVEL = 2` → copiii direcți ai rădăcinii
- `LEVEL = 3` → nepoții rădăcinii
- etc.

```sql
-- Formatare vizuală a ierarhiei cu LPAD
SELECT LEVEL,
       LPAD(' ', (LEVEL - 1) * 4) || FIRST_NAME AS IERARHIE,
       EMPLOYEE_ID,
       MANAGER_ID
FROM EMPLOYEES
START WITH MANAGER_ID IS NULL       -- rădăcina: angajatul fără manager
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID;
```

```
LEVEL | IERARHIE
  1   | Steven
  2   |     Neena
  2   |     Lex
  3   |         Alexander
  3   |         Bruce
```

---

### 1.6. Filtrarea cu `WHERE` în cereri ierarhice

```sql
-- WHERE filtrează DUPĂ construirea arborelui (nu afectează structura)
SELECT LEVEL, EMPLOYEE_ID, FIRST_NAME
FROM EMPLOYEES
WHERE LEVEL <= 2                          -- afișează doar primele 2 niveluri
START WITH MANAGER_ID IS NULL
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID;
```

> ⚠️ **`WHERE`** filtrează liniile din rezultat **după** construirea arborelui — nu elimină noduri intermediare din ierarhie.  
> ⚠️ **`CONNECT BY`** nu poate conține **subcereri**.

---

### 1.7. Excluderea unui subarbore cu `CONNECT BY` și `WHERE`

```sql
-- Excluderea angajatului 108 și a întregului subarbore al său
SELECT LEVEL, EMPLOYEE_ID, FIRST_NAME
FROM EMPLOYEES
START WITH MANAGER_ID IS NULL
CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID
       AND EMPLOYEE_ID <> 108;   -- exclude nodul 108 și tot subarborele lui
```

---

## 2. Clauza WITH (Common Table Expression — CTE)

**`WITH`** permite definirea unui **bloc de cerere temporar** (numit și CTE — Common Table Expression) înainte de instrucțiunea `SELECT` principală. Blocul poate fi **reutilizat** de mai multe ori în aceeași interogare.

> 💡 Util când o cerere face referință **de mai multe ori** la același bloc care conține `JOIN`-uri și funcții agregat — evită repetarea codului și îmbunătățește lizibilitatea.

### Sintaxă generală:

```sql
WITH nume_bloc AS (
    SELECT ...
    FROM ...
    [WHERE ...]
)
SELECT ...
FROM nume_bloc
[WHERE ...];
```

---

### 2.1. Exemplu simplu

```sql
-- Definiție bloc: totalul salariilor pe departament
WITH TOTAL_SALARII AS (
    SELECT DEPARTMENT_ID, SUM(SALARY) AS TOTAL
    FROM EMPLOYEES
    GROUP BY DEPARTMENT_ID
)
-- Utilizare bloc: departamentele cu totalul mai mare decât media totalurilor
SELECT D.DEPARTMENT_NAME, TS.TOTAL
FROM DEPARTMENTS D
JOIN TOTAL_SALARII TS ON D.DEPARTMENT_ID = TS.DEPARTMENT_ID
WHERE TS.TOTAL > (SELECT AVG(TOTAL) FROM TOTAL_SALARII)
ORDER BY TS.TOTAL DESC;
```

> 💡 `TOTAL_SALARII` este calculat **o singură dată** și folosit de **două ori** — în `JOIN` și în subcererea din `WHERE`.

---

### 2.2. Exemplu cu mai multe blocuri WITH

```sql
WITH
    SAL_DEPT AS (
        SELECT DEPARTMENT_ID, SUM(SALARY) AS TOTAL_SAL
        FROM EMPLOYEES
        GROUP BY DEPARTMENT_ID
    ),
    MEDIA_TOTALE AS (
        SELECT AVG(TOTAL_SAL) AS MEDIE
        FROM SAL_DEPT
    )
SELECT D.DEPARTMENT_NAME, SD.TOTAL_SAL
FROM DEPARTMENTS D
JOIN SAL_DEPT SD    ON D.DEPARTMENT_ID = SD.DEPARTMENT_ID
JOIN MEDIA_TOTALE M ON SD.TOTAL_SAL > M.MEDIE
ORDER BY SD.TOTAL_SAL DESC;
```

---

### 2.3. Exemplu cu subcerere complexă

```sql
-- Angajații care lucrează într-un departament cu cel puțin un angajat
-- al cărui salariu este egal cu salariul maxim din departamentul 30
WITH MAX_DEPT30 AS (
    SELECT MAX(SALARY) AS MAX_SAL
    FROM EMPLOYEES
    WHERE DEPARTMENT_ID = 30
)
SELECT E.FIRST_NAME, E.SALARY, E.DEPARTMENT_ID
FROM EMPLOYEES E
WHERE E.DEPARTMENT_ID IN (
    SELECT DEPARTMENT_ID
    FROM EMPLOYEES
    WHERE SALARY = (SELECT MAX_SAL FROM MAX_DEPT30)
);
```

---

## 3. Operatorul EXISTS

**`EXISTS`** testează dacă o subcerere corelată returnează **cel puțin o linie**. Nu analizează valorile returnate — verifică doar **existența** lor.

| Rezultat subcerere | `EXISTS` returnează |
| :--- | :--- |
| Cel puțin o linie | `TRUE` |
| Nicio linie | `FALSE` |

> 💡 **Eficiență:** `EXISTS` **se oprește** din căutare imediat ce găsește prima linie care satisface condiția — nu continuă scanarea.

---

### 3.1. Sintaxă generală

```sql
SELECT coloane
FROM tabel_extern
WHERE EXISTS (
    SELECT 1            -- ce se selectează nu contează, important e că există
    FROM tabel_intern
    WHERE conditie_corelata
);
```

> 💡 Prin convenție se scrie `SELECT 1` (sau `SELECT *`) în subcererea cu `EXISTS` — valoarea returnată nu contează, doar **existența** înregistrărilor.

---

### 3.2. Exemplu — departamente care au angajați

```sql
-- Departamentele care au cel puțin un angajat
SELECT D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE EXISTS (
    SELECT 1
    FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
);
```

---

### 3.3. `EXISTS` vs. `IN` — diferențe importante

```sql
-- Cu IN
SELECT D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE D.DEPARTMENT_ID IN (
    SELECT E.DEPARTMENT_ID
    FROM EMPLOYEES E
);

-- Echivalent cu EXISTS
SELECT D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE EXISTS (
    SELECT 1
    FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
);
```

| | `IN` | `EXISTS` |
| :--- | :--- | :--- |
| **Tip subcerere** | Nesincronizată (necorelată) | Sincronizată (corelată) |
| **Ce evaluează** | Lista de valori returnată | Existența cel puțin a unei linii |
| **Comportament cu NULL** | ⚠️ `NOT IN` cu NULL → niciun rezultat | ✅ `NOT EXISTS` nu are problema cu NULL |
| **Eficiență** | Evaluează toată lista | Se oprește la prima potrivire |
| **Recomandat când** | Lista este mică | Subcererea returnează multe linii |

---

### 3.4. `NOT EXISTS` — negarea

```sql
-- Departamentele care NU au niciun angajat
SELECT D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM DEPARTMENTS D
WHERE NOT EXISTS (
    SELECT 1
    FROM EMPLOYEES E
    WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID
);
```

> 💡 **`NOT EXISTS`** este mai sigur decât **`NOT IN`** când subcererea poate conține valori `NULL`.  
> Cu `NOT IN`, dacă subcererea returnează un `NULL`, întreaga condiție devine `UNKNOWN` și nu se returnează nicio linie.

---

## 4. Rezumat Rapid

```
CERERI IERARHICE:
  START WITH conditie      → identifică rădăcinile arborelui
  CONNECT BY PRIOR ...     → definește relația părinte-copil
  PRIOR pe stânga          → top-down (rădăcină → frunze)
  PRIOR pe dreapta         → bottom-up (frunze → rădăcină)
  LEVEL                    → pseudocoloană: adâncimea nodului (1 = rădăcină)
  ⚠️ CONNECT BY nu acceptă subcereri

CLAUZA WITH (CTE):
  WITH bloc AS (SELECT ...)
  SELECT ... FROM bloc      → reutilizabil în aceeași instrucțiune
  Util pentru: evitarea repetării, lizibilitate, referințe multiple la
               același bloc cu JOIN-uri și funcții agregat
  Se pot defini mai multe blocuri: WITH b1 AS (...), b2 AS (...) SELECT ...

OPERATORUL EXISTS:
  WHERE EXISTS (SELECT 1 FROM ... WHERE conditie_corelata)
  → TRUE dacă subcererea returnează cel puțin o linie
  → FALSE dacă subcererea nu returnează nicio linie
  → Se oprește la prima potrivire (eficient!)
  → NOT EXISTS mai sigur decât NOT IN (nu are problema cu NULL)
```

---

## 5. Capcane Frecvente ⚠️

| Situație | Problemă | Soluție |
| :--- | :--- | :--- |
| `CONNECT BY` fără `PRIOR` | Eroare Oracle | `PRIOR` este obligatoriu în `CONNECT BY` |
| Subcerere în `CONNECT BY` | Eroare Oracle | `CONNECT BY` nu acceptă subcereri |
| `WHERE LEVEL <= n` vs. filtrare în `CONNECT BY` | `WHERE` filtrează după construirea arborelui | Pune condiția în `CONNECT BY` dacă vrei să taie subarborele |
| `NOT IN` cu `NULL` în subcerere | Nu returnează nicio linie | Folosește `NOT EXISTS` sau adaugă `WHERE col IS NOT NULL` în subcerere |
| `WITH` — bloc nedefinit înainte de utilizare | Eroare Oracle | Definește toate blocurile `WITH` înainte de `SELECT` principal |
| `EXISTS` — ce se pune în `SELECT` din subcerere | Indiferent — nu contează valoarea | Prin convenție se folosește `SELECT 1` sau `SELECT *` |
