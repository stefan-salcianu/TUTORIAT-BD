<img width="1302" height="1275" alt="image" src="https://github.com/user-attachments/assets/06327d60-dc28-42d9-9bca-89744586729e" />






### Problema 1: Contorizare condiționată (CASE în COUNT)

**Cerință:** Afișați pentru fiecare livrator numărul de comenzi efectuate pentru clienții înregistrați în anii 2024 și 2025. Includeți și livratorii care nu au comenzi (vor apărea cu 0).

**Indicii de rezolvare:**
* Se folosește `LEFT JOIN` pentru a asigura apariția tuturor livratorilor, inclusiv a celor fără comenzi.
* Se utilizează `CASE WHEN ... THEN C.ID_COMANDA END` în interiorul funcției de `COUNT`. Acest lucru permite filtrarea condiționată; dacă s-ar fi folosit `WHERE`, livratorii fără comenzi în acei ani ar fi fost excluși din rezultat.
* Intervalul de filtrare: `>= '2024-01-01' AND < '2026-01-01'`.

**Soluție SQL:**
```sql
SELECT
    L.ID_LIVRATOR,
    L.NUME,
    COUNT(CASE WHEN 
          CL.DATA_INREGISTRARE >= DATE '2024-01-01' AND CL.DATA_INREGISTRARE < DATE '2026-01-01' 
          THEN C.ID_COMANDA END) 
    AS NUMAR_COMENZI
FROM
    LIVRATOR_restaurant L
LEFT JOIN
    COMANDA_RESTAURANT C ON L.ID_LIVRATOR = C.ID_LIVRATOR
LEFT JOIN
    CLIENT_RESTAURANT CL ON C.ID_CLIENT = CL.ID_CLIENT
GROUP BY
    L.ID_LIVRATOR, L.NUME;
```
### Problema 2: Top-N dinamic bazat pe o subcerere
Cerință: Afișați primii N clienți ordonați descrescător după totalul de kcal consumate (din produsele comandate), unde N = numărul de livratori angajați începând cu 1 ianuarie 2022.

Indicii de rezolvare:

Subquery-ul SELECT COUNT(*) FROM LIVRATOR_RESTAURANT WHERE DATA_ANGAJARE >= '2022-01-01' calculează valoarea pentru N.

Se aplică tiparul clasic Oracle Top-N: ROWNUM <= N aplicat peste o subcerere care ordonează datele (ORDER BY DESC).

Totalul caloriilor este calculat prin gruparea per client: SUM(M.KCAL).

Soluție SQL:

```
SQL
SELECT *
FROM (
    SELECT 
        CL.ID_CLIENT, CL.NUME, SUM(M.KCAL) AS TOTAL_KCAL
    FROM 
        CLIENT_RESTAURANT CL
    JOIN
        COMANDA_RESTAURANT C ON CL.ID_CLIENT = C.ID_CLIENT
    JOIN
        LISTA_COMANDA L ON L.ID_COMANDA = C.ID_COMANDA
    JOIN
        MENIU M ON M.ID_PRODUS = L.ID_PRODUS
    GROUP BY 
        CL.ID_CLIENT, CL.NUME
    ORDER BY 
        TOTAL_KCAL DESC
)
WHERE ROWNUM <= (
    SELECT COUNT(*)
    FROM LIVRATOR_RESTAURANT
    WHERE DATA_ANGAJARE >= DATE '2022-01-01'
);
```
### Problema 3: Condiții universale (FOR ALL) emulate prin HAVING
Cerință: Afișați clienții care au comandat doar produse cu cel mult 100 kcal (toate produsele din comenzile lor au ≤ 100 kcal).

Indicii de rezolvare:

Este necesară o grupare la nivel de client.

Trick util pentru colocviu: În loc să se folosească corelări greoaie cu NOT EXISTS, se folosește funcția agregată MAX: HAVING MAX(M.KCAL) <= 100. Dacă cel mai caloric produs comandat de un client este sub limita de 100 kcal, atunci logic, toate produsele sale au cel mult 100 kcal.

Soluție SQL:

```
SQL
SELECT CL.ID_CLIENT, CL.NUME
FROM
    CLIENT_RESTAURANT CL
JOIN
    COMANDA_RESTAURANT C ON CL.ID_CLIENT = C.ID_CLIENT
JOIN
    LISTA_COMANDA L ON L.ID_COMANDA = C.ID_COMANDA
JOIN
    MENIU M ON M.ID_PRODUS = L.ID_PRODUS
GROUP BY
    CL.ID_CLIENT, CL.NUME
HAVING
    MAX(M.KCAL) <= 100;
```
### Problema 4: Pivotare manuală (Agregare Condiționată)
Cerință: Afișați numele și prenumele fiecărui client, împreună cu suma totală cheltuită strict pe comenzi livrate, împărțită pe trei coloane distincte: suma cheltuită pe 'PIZZA', suma pe 'BAUTURA', și suma pe 'ALTELE' (restul categoriilor).

Indicii de rezolvare:

Se folosește o tehnică de pivotare clasică utilizând SUM(CASE WHEN ... THEN ... ELSE 0 END).

Funcția NVL(..., 0) asigură afișarea cifrei 0 în loc de NULL acolo unde clientul nu are nicio comandă din respectiva categorie.

Soluție SQL:

```
SQL
SELECT C.NUME, 
       C.PRENUME,
       NVL(SUM(CASE WHEN M.CATEGORIE = 'PIZZA' THEN LC.CANTITATE * LC.PRET_UNITAR ELSE 0 END), 0) AS TOTAL_PIZZA,
       NVL(SUM(CASE WHEN M.CATEGORIE = 'BAUTURA' THEN LC.CANTITATE * LC.PRET_UNITAR ELSE 0 END), 0) AS TOTAL_BAUTURI,
       NVL(SUM(CASE WHEN M.CATEGORIE NOT IN ('PIZZA', 'BAUTURA') THEN LC.CANTITATE * LC.PRET_UNITAR ELSE 0 END), 0) AS TOTAL_ALTELE
FROM CLIENT_RESTAURANT C
JOIN COMANDA_RESTAURANT CR ON C.ID_CLIENT = CR.ID_CLIENT
JOIN LISTA_COMANDA LC ON CR.ID_COMANDA = LC.ID_COMANDA
JOIN MENIU M ON LC.ID_PRODUS = M.ID_PRODUS
WHERE CR.STATUS = 'LIVRATA'
GROUP BY C.ID_CLIENT, C.NUME, C.PRENUME
ORDER BY TOTAL_PIZZA DESC;

```
### Problema 5: Clasificare dinamică a entităților (CASE în SELECT)
Cerință: Afișați numele, prenumele, mijlocul de transport și o coloană nouă numită EVALUARE pentru livratori. Evaluarea se face astfel:

Inactiv: dacă nu are nicio comandă preluată.

Performant: dacă are cel puțin 3 comenzi finalizate (LIVRATA).

Eco-Junior: dacă folosește BICICLETA sau TROTINETA și are minim o comandă livrată (dar mai puțin de 3, ca să nu suprascrie regula anterioară).

Standard: pentru orice alt caz.

Indicii de rezolvare:

Se utilizează instrucțiunea CASE direct în clauza SELECT.

Condițiile din instrucțiunea CASE sunt evaluate secvențial (de sus în jos).

Se folosesc funcții de agregare (COUNT și SUM) direct în ramurile de evaluare WHEN.

Soluție SQL:
```
SQL
SELECT L.NUME, 
       L.PRENUME, 
       L.MIJLOC_TRANSPORT,
       CASE 
           WHEN COUNT(C.ID_COMANDA) = 0 THEN 'Inactiv'
           WHEN SUM(CASE WHEN C.STATUS = 'LIVRATA' THEN 1 ELSE 0 END) >= 3 THEN 'Performant'
           WHEN L.MIJLOC_TRANSPORT IN ('BICICLETA', 'TROTINETA') 
                AND SUM(CASE WHEN C.STATUS = 'LIVRATA' THEN 1 ELSE 0 END) > 0 THEN 'Eco-Junior'
           ELSE 'Standard'
       END AS EVALUARE
FROM LIVRATOR_RESTAURANT L
LEFT JOIN COMANDA_RESTAURANT C ON L.ID_LIVRATOR = C.ID_LIVRATOR
GROUP BY L.ID_LIVRATOR, L.NUME, L.PRENUME, L.MIJLOC_TRANSPORT
ORDER BY EVALUARE;
```
### Problema 6: Sortare Customizată (CASE în ORDER BY)
Cerință: Afișați tot meniul (denumire, categorie, preț, disponibilitate) respectând următoarea regulă de sortare:

Produsele disponibile (Y) apar primele, ordonate crescător după preț.

Produsele indisponibile (N) sunt "împinse" la final, și între ele sunt ordonate alfabetic după denumire.

Indicii de rezolvare:

Se introduce instrucțiunea CASE direct în directiva ORDER BY.

Primul bloc de CASE setează prioritatea de grup (produsele Y primesc rangul 1, cele N rangul 2).

Următoarele blocuri de CASE tratează criteriile de sortare specifice per categorie (preț pentru disponibile, nume pentru indisponibile).

Soluție SQL:

```
SQL
SELECT DENUMIRE, CATEGORIE, PRET, DISPONIBIL
FROM MENIU
ORDER BY 
    CASE WHEN DISPONIBIL = 'Y' THEN 1 ELSE 2 END ASC,   -- Prioritizare grup
    CASE WHEN DISPONIBIL = 'Y' THEN PRET END ASC,       -- Sortare secundara pentru Y
    CASE WHEN DISPONIBIL = 'N' THEN DENUMIRE END ASC;   -- Sortare secundara pentru N

```
