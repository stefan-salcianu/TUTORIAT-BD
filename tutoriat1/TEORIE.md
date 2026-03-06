# 🗄️ Gestiunea Bazelor de Date - Concepte și SQL


## 1. Introducere
O **bază de date** este un ansamblu structurat de date coerente, organizat fără redundanță inutilă. Aceasta este concepută pentru a fi accesată în mod concurent de către mai mulți utilizatori, asigurând integritatea informației.

## 2. Ce este un SGBD?
Un **Sistem de Gestiune a Bazelor de Date (SGBD)** este produsul software care permite interacțiunea cu datele. 
**Funcții principale:**
* **Definirea** structurii datelor.
* **Consultarea** (extragerea) informațiilor.
* **Actualizarea** și întreținerea consistenței datelor.

---

## 3. Limbajul SQL
**SQL (Structured Query Language)** este limbajul standard neprocedural utilizat pentru comunicarea cu bazele de date relaționale.

### Categorii de instrucțiuni:
Instrucțiunile sunt grupate în sub-limbaje specifice:

| Limbaj | Acronim | Comenzi | Descriere |
| :--- | :--- | :--- | :--- |
| **Definire (LDD)** | DDL | `CREATE`, `ALTER`, `DROP` | Modifică structura tabelelor. |
| **Prelucrare (LMD)** | DML | `INSERT`, `UPDATE`, `DELETE`, `SELECT` | Gestionează datele din tabele. |
| **Control (LCD)** | TCL | `COMMIT`, `ROLLBACK` | Gestionează tranzacțiile și salvarea. |

> **Notă:** SQL include și instrucțiuni pentru controlul sesiunii, al sistemului și instrucțiuni încapsulate.

---

## 4. Analiza Sintaxei SELECT

Sintaxa generală a comenzii `SELECT` urmează această structură:

```sql
SELECT { [ {DISTINCT | UNIQUE} | ALL] lista_campuri | *} 
FROM [nume_schemă.]nume_obiect 
[, [nume_schemă.]nume_obiect ...] 
[WHERE condiție_clauza_where] 
[START WITH condiție_clauza_start_with
 CONNECT BY condiție_clauza_connect_by] 
[GROUP BY expresie [, expresie ...] 
[HAVING condiție_clauza_having] 
[ORDER BY {expresie | poziţie} [, {expresie | poziţie} ...] ];
```

## 4. Instrucțiunea SELECT: Reguli și Alias-uri

`SELECT` este o clauză **obligatorie** într-o interogare SQL.  
Câmpurile din `SELECT` se separă prin **virgulă** și pot fi:

---

### 1. Coloane simple

```sql
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES;
```

Selectează direct coloanele `FIRST_NAME` și `SALARY` din tabel.

---

### 2. Operații matematice

```sql
SELECT FIRST_NAME, SALARY * 12 AS SALARIU_ANUAL
FROM EMPLOYEES;
```

Se realizează o **operație matematică** pentru a calcula salariul anual.

---

### 3. Funcții aplicate pe coloane

```sql
SELECT UPPER(FIRST_NAME) AS NUME_MAJUSCULE
FROM EMPLOYEES;
```

Funcția `UPPER()` transformă valorile din coloană în **majuscule**.

---

### 4. Subcereri (Subqueries)

Subcererile sunt comenzi `SELECT` incluse în alte interogări.

```sql
SELECT FIRST_NAME, SALARY
FROM EMPLOYEES
WHERE SALARY > (
    SELECT AVG(SALARY)
    FROM EMPLOYEES
);
```

Această interogare returnează angajații care au **salariul mai mare decât media salariilor**.

### 💡 Reguli de Sintaxă:
* **Alias-uri**: Se pot pune cu sau fără `AS`. Dacă alias-ul conține spații (blank-uri), se folosesc **ghilimelele** (`" "`).
* **String-uri (Valori)**: Se definesc întotdeauna cu **apostrof** (`' '`). Ghilimelele sunt doar pentru alias-uri!
* **Wildcard (`*`)**: Afișează toate coloanele. 
    * ⚠️ **Eroare**: Nu poți folosi `SELECT *, column_name...`. Dacă apare `*`, nu mai poți adăuga câmpuri individuale decât dacă prefixezi tabelul (`SELECT E.*, E.col`).
* **Duplicate**: `DISTINCT` sau `UNIQUE` elimină rândurile duplicate din rezultat.
* **Inspecție**: `DESC nume_tabel` (afișează structura). *Funcționează în Oracle SQL Developer, nu neapărat în DataGrip.*

---

## 5. Referențierea Tabelelor și Ambiguitatea

Clauza **`FROM`** este obligatorie. Există mai multe moduri de a referenția coloanele:

---

### 1. Simplu

```sql
SELECT EMPLOYEE_ID, FIRST_NAME
FROM EMPLOYEES;
```

Se folosesc direct numele coloanelor.

---

### 2. Prefixat cu numele tabelului

```sql
SELECT EMPLOYEES.EMPLOYEE_ID, EMPLOYEES.FIRST_NAME
FROM EMPLOYEES;
```

Coloanele sunt prefixate cu numele tabelului.

---

### 3. Prefixat cu alias de tabel (**recomandat**)

```sql
SELECT E.EMPLOYEE_ID, E.FIRST_NAME
FROM EMPLOYEES E;
```

`E` este un **alias** pentru tabelul `EMPLOYEES`, ceea ce face query-ul mai scurt și mai ușor de citit.

---

### 4. Alias de tabel cu spații

```sql
SELECT "TABEL ANGAJATI".EMPLOYEE_ID, "TABEL ANGAJATI".FIRST_NAME
FROM EMPLOYEES "TABEL ANGAJATI";
```

Dacă aliasul conține **spații**, trebuie pus între ghilimele `" "`.

---

> **De ce prefixăm?** Pentru a evita ambiguitatea când:
> * Lucrăm cu mai multe instanțe ale aceluiași tabel.
> * Lucrăm cu tabele diferite care au coloane cu același nume (ex: `MANAGER_ID` în `Employees` și `Departments`).

---

## 6. Filtrarea Datelor (WHERE) și Operatori

Dacă un query are mai multe condiții, se folosește o singură clauză `WHERE`, condițiile fiind legate prin operatori logici.

### Operatori:
* **Comparație**: `=`, `<>`, `!=`, `<`, `<=`, `>`, `>=`.
* **Apartenență la interval**: `BETWEEN val1 AND val2` (interval închis).
* **Apartenență la mulțime**: `IN (val1, val2, ... )`.
* **Logici**: `AND`, `OR`, `NOT`.

---

## 7. Logica Valorilor NULL și UNKNOWN

`NULL` reprezintă absența unei valori. 
* Pentru verificare, se folosește **`IS NULL`** sau **`IS NOT NULL`**.
* Comparațiile directe (`= NULL`) returnează starea **UNKNOWN**.

### Tabel de Adevăr (Logica UNKNOWN):
| Operație | Rezultat |
| :--- | :--- |
| **FALSE AND UNKNOWN** | FALSE |
| **FALSE OR UNKNOWN** | UNKNOWN |
| **TRUE OR UNKNOWN** | TRUE |
| **TRUE AND UNKNOWN** | UNKNOWN |

---

## 8. Sortarea Datelor (ORDER BY)

* **Default**: Sortarea este `ASC` (crescătoare).
* **Specific**: Pentru fiecare coloană se poate pune `ASC` sau `DESC`.
* **Tratarea NULL la sortare**:
    * Sortare **DESC**: `NULL` apare primul.
    * Sortare **ASC**: `NULL` apare ultimul.

---

## 9. Pattern Matching (LIKE)

Folosit pentru a căuta șabloane în șiruri de caractere:
* `_` (underscore): reprezintă un singur caracter.
* `%` (procent): reprezintă zero, unul sau mai multe caractere.

**Exemple:**
* `LIKE '__A%'` -> Șir de minim 3 caractere, unde al 3-lea este 'A'.
* `LIKE '_B_%'` -> Șir de minim 3 caractere, unde al 2-lea este 'B'.

---

### 🛠️ Exemplu de Depanare (Exercițiu)

```sql
SELECT employee_id, last_name 
salary * 12 ANNUAL SALARY 
FROM employees;
```
