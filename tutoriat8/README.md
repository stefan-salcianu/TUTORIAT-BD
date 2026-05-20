## 🗺️ Schema bazei de date — Librărie Online

O librărie cu autori, cărți, clienți, comenzi și recenzii.

```sql
CREATE TABLE autori (
    id_autor       INT PRIMARY KEY,
    nume           VARCHAR(100),
    nationalitate  VARCHAR(50),
    an_debut       INT
);

CREATE TABLE carti (
    id_carte       INT PRIMARY KEY,
    titlu          VARCHAR(200),
    id_autor       INT,
    gen_literar    VARCHAR(50),
    pret           DECIMAL(8,2),
    an_publicare   INT,
    stoc           INT,
    FOREIGN KEY (id_autor) REFERENCES autori(id_autor)
);

CREATE TABLE clienti (
    id_client      INT PRIMARY KEY,
    nume           VARCHAR(100),
    email          VARCHAR(100),
    oras           VARCHAR(50),
    data_inregistrare DATE
);

CREATE TABLE comenzi (
    id_comanda     INT PRIMARY KEY,
    id_client      INT,
    data_comanda   DATE,
    status         VARCHAR(20),   -- 'livrata', 'in_procesare', 'anulata'
    total          DECIMAL(10,2),
    FOREIGN KEY (id_client) REFERENCES clienti(id_client)
);

CREATE TABLE detalii_comanda (
    id_comanda     INT,
    id_carte       INT,
    cantitate      INT,
    pret_unitar    DECIMAL(8,2),
    PRIMARY KEY (id_comanda, id_carte),
    FOREIGN KEY (id_comanda) REFERENCES comenzi(id_comanda),
    FOREIGN KEY (id_carte)   REFERENCES carti(id_carte)
);

CREATE TABLE recenzii (
    id_recenzie    INT PRIMARY KEY,
    id_client      INT,
    id_carte       INT,
    nota           INT,           -- 1–5
    comentariu     VARCHAR2(4000),
    data_recenzie  DATE,
    FOREIGN KEY (id_client) REFERENCES clienti(id_client),
    FOREIGN KEY (id_carte)  REFERENCES carti(id_carte)
);
```

---

### 📥 Date de test

<details>
<summary>Click pentru scriptul de populare</summary>

```sql
INSERT ALL 
  INTO autori VALUES (1, 'Mircea Eliade', 'Română', 1930)
  INTO autori VALUES (2, 'Mihail Sadoveanu', 'Română', 1904)
  INTO autori VALUES (3, 'George Orwell', 'Britanică', 1933)
  INTO autori VALUES (4, 'Gabriel García Márquez', 'Columbiană', 1955)
  INTO autori VALUES (5, 'Haruki Murakami', 'Japoneză', 1979)
  INTO autori VALUES (6, 'Agatha Christie', 'Britanică', 1920)
  INTO autori VALUES (7, 'Stephen King', 'Americană', 1974)
  INTO autori VALUES (8, 'Elena Ferrante', 'Italiană', 1992)
SELECT * FROM dual;

INSERT ALL
  INTO carti VALUES (1, 'Maitreyi', 1, 'Roman', 45.00, 1933, 30)
  INTO carti VALUES (2, 'Baltagul', 2, 'Roman', 38.00, 1930, 25)
  INTO carti VALUES (3, '1984', 3, 'Distopie', 55.00, 1949, 50)
  INTO carti VALUES (4, 'Ferma animalelor', 3, 'Satiră', 40.00, 1945, 40)
  INTO carti VALUES (5, 'Un veac de singurătate', 4, 'Magic Realism', 75.00, 1967, 20)
  INTO carti VALUES (6, 'Dragoste în vremea holerei', 4, 'Roman', 65.00, 1985, 15)
  INTO carti VALUES (7, 'Norwegian Wood', 5, 'Roman', 60.00, 1987, 35)
  INTO carti VALUES (8, 'Kafka pe malul mării', 5, 'Roman', 62.00, 2002, 28)
  INTO carti VALUES (9, 'Crima din Orient Express', 6, 'Thriller', 42.00, 1934, 45)
  INTO carti VALUES (10, 'Zece negri mititei', 6, 'Thriller', 44.00, 1939, 38)
  INTO carti VALUES (11, 'It', 7, 'Horror', 90.00, 1986, 22)
  INTO carti VALUES (12, 'Strălucirea', 7, 'Horror', 80.00, 1977, 18)
  INTO carti VALUES (13, 'Prietena mea genială', 8, 'Roman', 55.00, 2011, 32)
  INTO carti VALUES (14, 'Povestea noului nume', 8, 'Roman', 57.00, 2012, 27)
SELECT * FROM dual;


INSERT ALL
  INTO clienti VALUES (1, 'Andreea Popescu', 'andreea@email.ro', 'București', DATE '2022-03-10')
  INTO clienti VALUES (2, 'Radu Ionescu', 'radu@email.ro', 'Cluj-Napoca', DATE '2022-07-22')
  INTO clienti VALUES (3, 'Maria Constantin', 'maria@email.ro', 'Iași', DATE '2023-01-05')
  INTO clienti VALUES (4, 'Vlad Dumitrescu', 'vlad@email.ro', 'București', DATE '2023-04-18')
  INTO clienti VALUES (5, 'Ioana Gheorghe', 'ioana@email.ro', 'Timișoara', DATE '2023-09-30')
  INTO clienti VALUES (6, 'Mihai Stan', 'mihai@email.ro', 'Cluj-Napoca', DATE '2024-01-15')
  INTO clienti VALUES (7, 'Elena Marin', 'elena@email.ro', 'București', DATE '2024-02-28')
  INTO clienti VALUES (8, 'Cristian Popa', 'cristian@email.ro', 'Brașov', DATE '2024-03-12')
SELECT * FROM dual;


INSERT ALL
  INTO comenzi VALUES (1, 1, DATE '2024-01-10', 'livrata', 95.00)
  INTO comenzi VALUES (2, 2, DATE '2024-01-18', 'livrata', 115.00)
  INTO comenzi VALUES (3, 3, DATE '2024-02-02', 'livrata', 75.00)
  INTO comenzi VALUES (4, 1, DATE '2024-02-15', 'livrata', 140.00)
  INTO comenzi VALUES (5, 4, DATE '2024-02-20', 'anulata', 55.00)
  INTO comenzi VALUES (6, 5, DATE '2024-03-05', 'livrata', 122.00)
  INTO comenzi VALUES (7, 2, DATE '2024-03-14', 'livrata', 90.00)
  INTO comenzi VALUES (8, 6, DATE '2024-03-22', 'in_procesare', 62.00)
  INTO comenzi VALUES (9, 7, DATE '2024-04-01', 'livrata', 80.00)
  INTO comenzi VALUES (10, 3, DATE '2024-04-10', 'livrata', 65.00)
  INTO comenzi VALUES (11, 8, DATE '2024-04-18', 'in_procesare', 44.00)
  INTO comenzi VALUES (12, 1, DATE '2024-05-01', 'livrata', 60.00)
SELECT * FROM dual;

INSERT ALL
  INTO detalii_comanda VALUES (1, 3, 1, 55.00)  INTO detalii_comanda VALUES (1, 9, 1, 42.00)
  INTO detalii_comanda VALUES (2, 7, 1, 60.00)  INTO detalii_comanda VALUES (2, 4, 1, 40.00)
  INTO detalii_comanda VALUES (2, 9, 1, 42.00)  INTO detalii_comanda VALUES (3, 2, 1, 38.00)
  INTO detalii_comanda VALUES (3, 1, 1, 45.00)  INTO detalii_comanda VALUES (4, 5, 1, 75.00)
  INTO detalii_comanda VALUES (4, 8, 1, 62.00)  INTO detalii_comanda VALUES (5, 3, 1, 55.00)
  INTO detalii_comanda VALUES (6, 6, 1, 65.00)  INTO detalii_comanda VALUES (6, 13, 1, 55.00)
  INTO detalii_comanda VALUES (7, 11, 1, 90.00) INTO detalii_comanda VALUES (8, 8, 1, 62.00)
  INTO detalii_comanda VALUES (9, 12, 1, 80.00) INTO detalii_comanda VALUES (10, 14, 1, 57.00)
  INTO detalii_comanda VALUES (10, 1, 1, 45.00) INTO detalii_comanda VALUES (11, 10, 1, 44.00)
  INTO detalii_comanda VALUES (12, 7, 1, 60.00)
SELECT * FROM dual;

INSERT ALL
  INTO recenzii VALUES (1, 1, 3, 5, 'Absolut magistral!', DATE '2024-01-25')
  INTO recenzii VALUES (2, 2, 7, 4, 'Atmosferă incredibilă.', DATE '2024-02-01')
  INTO recenzii VALUES (3, 3, 2, 5, 'O capodoperă.', DATE '2024-02-15')
  INTO recenzii VALUES (4, 1, 9, 4, 'Suspans de nota 10.', DATE '2024-03-01')
  INTO recenzii VALUES (5, 4, 3, 3, 'Bun, dar așteptam mai mult.', DATE '2024-03-10')
  INTO recenzii VALUES (6, 5, 6, 5, 'M-a emoționat profund.', DATE '2024-03-20')
  INTO recenzii VALUES (7, 2, 4, 5, 'Satiră strălucitoare.', DATE '2024-03-28')
  INTO recenzii VALUES (8, 6, 8, 4, 'Murakami la cel mai bun.', DATE '2024-04-05')
  INTO recenzii VALUES (9, 7, 12, 3, 'Prea lung pentru gustul meu.', DATE '2024-04-12')
  INTO recenzii VALUES (10, 3, 14, 5, 'Nu m-am putut opri din citit.', DATE '2024-04-22')
  INTO recenzii VALUES (11, 8, 10, 4, 'Clasic bine construit.', DATE '2024-05-02')
  INTO recenzii VALUES (12, 1, 5, 5, 'Schimbă percepția asupra vieții.', DATE '2024-05-10')
SELECT * FROM dual;
```

</details>

---

## 🧠 Înainte să începi — Regula de aur

Înainte să scrii orice query, pune-ți această întrebare:

```
"Vreau UN SINGUR rând per grup de date   →  GROUP BY
 sau vreau RÂNDURI INDIVIDUALE?"          →  fără GROUP BY
```

**Semnale de GROUP BY:** *câți, câte, total, suma, media, maxim per X, minim per X*
**Semnale fără GROUP BY:** *afișează, listează, găsește, detaliile, toți cei care...*

> ⚠️ **Atenție la capcanele de limbaj:** cuvintele *„total"*, *„medie"* sau *„cel mai"* nu înseamnă **automat** GROUP BY. Uneori sunt doar subquery-uri de referință.

---

## 🎯 Exerciții

---

### #01 — Fără GROUP BY
**Cerință:** Afișează titlul, genul literar și prețul tuturor cărților de tip „Roman".

<details>
<summary>💡 Gândire</summary>
Vreau să văd fiecare carte în parte. Filtrez după gen, nu agreghez nimic.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT titlu, gen_literar, pret
FROM carti
WHERE gen_literar = 'Roman'
ORDER BY pret DESC;
```
</details>

---

### #02 — Cu GROUP BY
**Cerință:** Câte cărți există în fiecare gen literar?

<details>
<summary>💡 Gândire</summary>
„Câte... per gen" = grupare pe gen + COUNT.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT gen_literar, COUNT(*) AS nr_carti
FROM carti
GROUP BY gen_literar
ORDER BY nr_carti DESC;
```
</details>

---

### #03 — Fără GROUP BY
**Cerință:** Găsește cartea cu cel mai mic preț din librărie.

<details>
<summary>💡 Gândire</summary>
Vreau UN singur rând — cartea ieftinistă. Minimul e un subquery de referință, nu o grupare.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT titlu, pret
FROM carti
WHERE pret = (SELECT MIN(pret) FROM carti);
```
</details>

---

### #04 — Cu GROUP BY
**Cerință:** Care este prețul mediu al cărților per gen literar?

<details>
<summary>💡 Gândire</summary>
„Preț mediu... per gen" = grupare + AVG.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT gen_literar,
       ROUND(AVG(pret), 2) AS pret_mediu,
       COUNT(*)            AS nr_carti
FROM carti
GROUP BY gen_literar
ORDER BY pret_mediu DESC;
```
</details>

---

### #05 — ⚠️ Capcană — Fără GROUP BY
**Cerință:** Afișează toate comenzile unui client (Andreea Popescu), cu data și totalul fiecăreia.

<details>
<summary>💡 Gândire</summary>
Mulți vor pune GROUP BY pentru că „e vorba de comenzi per client". Dar vreau lista detaliată a comenzilor, nu un total — deci rânduri individuale.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT c.id_comanda, c.data_comanda, c.status, c.total
FROM comenzi c
JOIN clienti cl ON c.id_client = cl.id_client
WHERE cl.nume = 'Andreea Popescu'
ORDER BY c.data_comanda;
```
</details>

---

### #06 — Cu GROUP BY
**Cerință:** Care este valoarea totală a comenzilor per client?

<details>
<summary>💡 Gândire</summary>
„Total... per client" = grupare + SUM.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT cl.nume, COUNT(c.id_comanda) AS nr_comenzi, SUM(c.total) AS total_cheltuit
FROM clienti cl
LEFT JOIN comenzi c ON cl.id_client = c.id_client
GROUP BY cl.id_client, cl.nume
ORDER BY total_cheltuit DESC;
```
</details>

---

### #07 — Fără GROUP BY
**Cerință:** Lista tuturor recenziilor cu nota 5, împreună cu titlul cărții și numele clientului.

<details>
<summary>💡 Gândire</summary>
Vreau să văd fiecare recenzie individuală. Filtrez pe notă, nu grupez.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT r.data_recenzie, cl.nume AS client, ca.titlu, r.nota, r.comentariu
FROM recenzii r
JOIN clienti cl ON r.id_client = cl.id_client
JOIN carti   ca ON r.id_carte  = ca.id_carte
WHERE r.nota = 5
ORDER BY r.data_recenzie;
```
</details>

---

### #08 — Cu GROUP BY
**Cerință:** Care este nota medie primită de fiecare carte?

<details>
<summary>💡 Gândire</summary>
„Notă medie... per carte" = grupare + AVG.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT ca.titlu,
       ROUND(AVG(r.nota), 2) AS nota_medie,
       COUNT(r.id_recenzie)  AS nr_recenzii
FROM carti ca
LEFT JOIN recenzii r ON ca.id_carte = r.id_carte
GROUP BY ca.id_carte, ca.titlu
ORDER BY nota_medie DESC;
```
</details>

---

### #09 — Fără GROUP BY
**Cerință:** Afișează cărțile cu stoc sub 25 de exemplare.

<details>
<summary>💡 Gândire</summary>
Filtrare simplă pe o coloană numerică. Fiecare carte e un rând, nu calculez nimic per grup.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT titlu, stoc, pret
FROM carti
WHERE stoc < 25
ORDER BY stoc;
```
</details>

---

### #10 — Cu GROUP BY + HAVING
**Cerință:** Care sunt autorii cu mai mult de o carte în librărie?

<details>
<summary>💡 Gândire</summary>
„Câte cărți per autor" + filtrare pe acel număr = GROUP BY + HAVING.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT a.nume, COUNT(ca.id_carte) AS nr_carti
FROM autori a
JOIN carti ca ON a.id_autor = ca.id_autor
GROUP BY a.id_autor, a.nume
HAVING COUNT(ca.id_carte) > 1
ORDER BY nr_carti DESC;
```
</details>

---

### #11 — ⚠️ Capcană — Fără GROUP BY
**Cerință:** Afișează clienții care au plasat cel puțin o comandă cu statusul „anulata".

<details>
<summary>💡 Gândire</summary>
Verificare de existență, nu numărare. EXISTS sau IN e suficient — nu avem nevoie de GROUP BY.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT DISTINCT cl.nume, cl.email, cl.oras
FROM clienti cl
JOIN comenzi c ON cl.id_client = c.id_client
WHERE c.status = 'anulata';
```
</details>

---

### #12 — Cu GROUP BY
**Cerință:** Câți clienți s-au înregistrat în fiecare an?

<details>
<summary>💡 Gândire</summary>
„Câți... per an" = grupare pe an + COUNT.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT TO_CHAR(data_inregistrare, 'YYYY') AS an, 
       COUNT(*) AS nr_clienti_noi
FROM clienti
GROUP BY TO_CHAR(data_inregistrare, 'YYYY')
ORDER BY an;
```
</details>

---

### #13 — Fără GROUP BY
**Cerință:** Găsește toate cărțile scrise de autori britanici.

<details>
<summary>💡 Gândire</summary>
JOIN simplu + filtrare pe naționalitate. Rânduri individuale de cărți.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT ca.titlu, a.nume AS autor, ca.an_publicare, ca.pret
FROM carti ca
JOIN autori a ON ca.id_autor = a.id_autor
WHERE a.nationalitate = 'Britanică';
```
</details>

---

### #14 — Cu GROUP BY
**Cerință:** Care este venitul total generat de fiecare autor (prin vânzările din comenzi)?

<details>
<summary>💡 Gândire</summary>
„Total venit... per autor" implică SUM prin mai multe join-uri — tot GROUP BY.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT a.nume AS autor,
       SUM(dc.cantitate * dc.pret_unitar) AS venit_total
FROM autori a
JOIN carti          ca ON a.id_autor    = ca.id_autor
JOIN detalii_comanda dc ON ca.id_carte  = dc.id_carte
JOIN comenzi         c  ON dc.id_comanda = c.id_comanda
WHERE c.status != 'anulata'
GROUP BY a.id_autor, a.nume
ORDER BY venit_total DESC;
```
</details>

---

### #15 — ⚠️ Capcană — Fără GROUP BY
**Cerință:** Afișează cărțile cu prețul mai mare decât media generală a prețurilor din librărie.

<details>
<summary>💡 Gândire</summary>
„Media" apare, dar e doar un subquery de comparație. Vreau rânduri individuale de cărți, nu o statistică per grup.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT titlu, gen_literar, pret
FROM carti
WHERE pret > (SELECT AVG(pret) FROM carti)
ORDER BY pret DESC;
```
</details>

---

### #16 — Cu GROUP BY
**Cerință:** Care este prețul minim, maxim și mediu al cărților per autor?

<details>
<summary>💡 Gândire</summary>
Trei agregate per autor = GROUP BY cu MIN, MAX, AVG.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT a.nume,
       MIN(ca.pret)              AS pret_minim,
       MAX(ca.pret)              AS pret_maxim,
       ROUND(AVG(ca.pret), 2)   AS pret_mediu
FROM autori a
JOIN carti ca ON a.id_autor = ca.id_autor
GROUP BY a.id_autor, a.nume;
```
</details>

---

### #17 — Fără GROUP BY
**Cerință:** Afișează toate comenzile livrate din luna martie 2024, cu numele clientului.

<details>
<summary>💡 Gândire</summary>
Filtrare pe dată și status. Fiecare comandă e un rând separat.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT c.id_comanda, cl.nume, c.data_comanda, c.total
FROM comenzi c
JOIN clienti cl ON c.id_client = cl.id_client
WHERE c.status = 'livrata'
  AND c.data_comanda BETWEEN DATE '2024-03-01' AND DATE '2024-03-31';
```
</details>

---

### #18 — Cu GROUP BY
**Cerință:** Câte comenzi livrate a plasat fiecare client din București?

<details>
<summary>💡 Gândire</summary>
„Câte... per client" + filtrare = WHERE (oraș) + GROUP BY + COUNT.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT cl.nume, COUNT(c.id_comanda) AS nr_comenzi_livrate
FROM clienti cl
JOIN comenzi c ON cl.id_client = c.id_client
WHERE cl.oras = 'București'
  AND c.status = 'livrata'
GROUP BY cl.id_client, cl.nume
ORDER BY nr_comenzi_livrate DESC;
```
</details>

---

### #19 — ⚠️ Capcană — Fără GROUP BY
**Cerință:** Afișează detaliile fiecărei comenzi: ce cărți conține, cantitatea și prețul.

<details>
<summary>💡 Gândire</summary>
Vreau linii detaliate din `detalii_comanda` — un rând per carte per comandă. Nu agreg nimic.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT c.id_comanda, c.data_comanda, cl.nume AS client,
       ca.titlu, dc.cantitate, dc.pret_unitar,
       dc.cantitate * dc.pret_unitar AS subtotal
FROM comenzi c
JOIN clienti          cl ON c.id_client   = cl.id_client
JOIN detalii_comanda  dc ON c.id_comanda  = dc.id_comanda
JOIN carti            ca ON dc.id_carte   = ca.id_carte
ORDER BY c.id_comanda, ca.titlu;
```
</details>

---

### #20 — Cu GROUP BY + HAVING
**Cerință:** Care sunt cărțile cu o notă medie mai mare de 4?

<details>
<summary>💡 Gândire</summary>
„Notă medie per carte" + filtrare pe medie = GROUP BY + HAVING (nu WHERE, fiindcă filtrăm un rezultat agregat).
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT ca.titlu, ROUND(AVG(r.nota), 2) AS nota_medie, COUNT(*) AS nr_recenzii
FROM carti ca
JOIN recenzii r ON ca.id_carte = r.id_carte
GROUP BY ca.id_carte, ca.titlu
HAVING AVG(r.nota) > 4
ORDER BY nota_medie DESC;
```
</details>

---

### #21 — Fără GROUP BY
**Cerință:** Găsește clienții care NU au plasat nicio comandă.

<details>
<summary>💡 Gândire</summary>
Verificare de absență cu LEFT JOIN sau NOT EXISTS. Nu numărăm nimic per grup.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT cl.nume, cl.email, cl.oras
FROM clienti cl
LEFT JOIN comenzi c ON cl.id_client = c.id_client
WHERE c.id_comanda IS NULL;
```
</details>

---

### #22 — Cu GROUP BY
**Cerință:** Care este stocul total disponibil per gen literar?

<details>
<summary>💡 Gândire</summary>
„Total stoc... per gen" = grupare + SUM.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT gen_literar, SUM(stoc) AS stoc_total, COUNT(*) AS nr_titluri
FROM carti
GROUP BY gen_literar
ORDER BY stoc_total DESC;
```
</details>

---

### #23 — Fără GROUP BY
**Cerință:** Afișează cărțile publicate înainte de 1950, ordonate cronologic.

<details>
<summary>💡 Gândire</summary>
Filtrare pe an, sortare. Rânduri individuale.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT titlu, an_publicare, pret, gen_literar
FROM carti
WHERE an_publicare < 1950
ORDER BY an_publicare;
```
</details>

---

### #24 — Cu GROUP BY
**Cerință:** Câte recenzii a scris fiecare client și care e nota medie pe care o acordă?

<details>
<summary>💡 Gândire</summary>
„Câte + medie... per client" = grupare + COUNT + AVG.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT cl.nume,
       COUNT(r.id_recenzie)   AS nr_recenzii,
       ROUND(AVG(r.nota), 2) AS nota_medie_acordata
FROM clienti cl
JOIN recenzii r ON cl.id_client = r.id_client
GROUP BY cl.id_client, cl.nume
ORDER BY nr_recenzii DESC;
```
</details>

---

### #25 — ⚠️ Capcană — Fără GROUP BY
**Cerință:** Afișează pentru fiecare carte procentul din stocul total al librăriei.

<details>
<summary>💡 Gândire</summary>
„Procent" și „total" apar, dar fiecare carte rămâne un rând individual — totalul e un subquery de referință, nu o grupare.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT titlu, stoc,
       ROUND(stoc * 100.0 / (SELECT SUM(stoc) FROM carti), 2) AS procent_din_stoc
FROM carti
ORDER BY procent_din_stoc DESC;
```
</details>

---

### #26 — Cu GROUP BY
**Cerință:** Care sunt orașele din care provin clienții și câți clienți are fiecare?

<details>
<summary>💡 Gândire</summary>
„Câți... per oraș" = grupare pe oraș + COUNT.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT oras, COUNT(*) AS nr_clienti
FROM clienti
GROUP BY oras
ORDER BY nr_clienti DESC;
```
</details>

---

### #27 — Fără GROUP BY
**Cerință:** Afișează toate cărțile împreună cu autorul lor și naționalitatea autorului.

<details>
<summary>💡 Gândire</summary>
JOIN simplu pentru a adăuga context. Un rând per carte.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT ca.titlu, a.nume AS autor, a.nationalitate, ca.pret, ca.an_publicare
FROM carti ca
JOIN autori a ON ca.id_autor = a.id_autor
ORDER BY a.nationalitate, ca.titlu;
```
</details>

---

### #28 — Cu GROUP BY
**Cerință:** Câte comenzi are fiecare status și care e valoarea lor totală?

<details>
<summary>💡 Gândire</summary>
„Câte + total... per status" = grupare pe status + COUNT + SUM.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT status,
       COUNT(*)       AS nr_comenzi,
       SUM(total)     AS valoare_totala,
       ROUND(AVG(total), 2) AS valoare_medie
FROM comenzi
GROUP BY status;
```
</details>

---

### #29 — ⚠️ Capcană — Fără GROUP BY
**Cerință:** Afișează cărțile care nu au primit nicio recenzie.

<details>
<summary>💡 Gândire</summary>
Verificare de absență — LEFT JOIN + IS NULL sau NOT EXISTS. Nu numărăm per grup.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT ca.titlu, ca.gen_literar, ca.pret
FROM carti ca
LEFT JOIN recenzii r ON ca.id_carte = r.id_carte
WHERE r.id_recenzie IS NULL;
```
</details>

---

### #30 — Cu GROUP BY + HAVING
**Cerință:** Care sunt autorii ale căror cărți au un preț mediu mai mare de 60 lei?

<details>
<summary>💡 Gândire</summary>
„Preț mediu per autor" + filtrare pe medie = GROUP BY + HAVING.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT a.nume, ROUND(AVG(ca.pret), 2) AS pret_mediu_carti
FROM autori a
JOIN carti ca ON a.id_autor = ca.id_autor
GROUP BY a.id_autor, a.nume
HAVING AVG(ca.pret) > 60
ORDER BY pret_mediu_carti DESC;
```
</details>

---

### #31 — Fără GROUP BY
**Cerință:** Afișează top 3 cele mai scumpe cărți din librărie.

<details>
<summary>💡 Gândire</summary>
Sortare + LIMIT. Nu grupez — aleg rânduri individuale.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT titlu, gen_literar, pret
FROM carti
ORDER BY pret DESC
FETCH FIRST 3 ROWS ONLY;

SELECT *
FROM (
    SELECT titlu, gen_literar, pret
    FROM carti
    ORDER BY pret DESC
)
WHERE ROWNUM <= 3;
```
</details>

---

### #32 — Cu GROUP BY
**Cerință:** Top 3 clienți după valoarea totală cumpărată (doar comenzi livrate).

<details>
<summary>💡 Gândire</summary>
„Total per client" + top = GROUP BY + SUM + ORDER BY + LIMIT.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT cl.nume, SUM(c.total) AS total_achizitionat
FROM clienti cl
JOIN comenzi c ON cl.id_client = c.id_client
WHERE c.status = 'livrata'
GROUP BY cl.id_client, cl.nume
ORDER BY total_achizitionat DESC
FETCH FIRST 3 ROWS ONLY;
```
</details>

---

### #33 — Fără GROUP BY
**Cerință:** Găsește autorul cu cel mai vechi debut literar.

<details>
<summary>💡 Gândire</summary>
Vreau UN autor. Minimul e subquery de referință.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT nume, nationalitate, an_debut
FROM autori
WHERE an_debut = (SELECT MIN(an_debut) FROM autori);
```
</details>

---

### #34 — Cu GROUP BY
**Cerință:** Câte cărți diferite au fost comandate în fiecare lună din 2024?

<details>
<summary>💡 Gândire</summary>
„Câte... per lună" = grupare pe lună + COUNT DISTINCT (ca să nu numărăm aceeași carte de mai multe ori).
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT EXTRACT(MONTH FROM c.data_comanda) AS luna,
       COUNT(DISTINCT dc.id_carte) AS nr_titluri_distincte,
       SUM(dc.cantitate)           AS total_exemplare
FROM comenzi c
JOIN detalii_comanda dc ON c.id_comanda = dc.id_comanda
WHERE EXTRACT(YEAR FROM c.data_comanda) = 2024
GROUP BY EXTRACT(MONTH FROM c.data_comanda)
ORDER BY luna;
```
</details>

---

### #35 — ⚠️ Capcană — Fără GROUP BY
**Cerință:** Afișează clienții care au comandat cartea „1984".

<details>
<summary>💡 Gândire</summary>
Mulți vor face GROUP BY pe client. Dar vrem lista de clienți care au comandat această carte — DISTINCT e suficient, nu agregăm.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT DISTINCT cl.nume, cl.oras
FROM clienti cl
JOIN comenzi          c  ON cl.id_client  = c.id_client
JOIN detalii_comanda  dc ON c.id_comanda  = dc.id_comanda
JOIN carti            ca ON dc.id_carte   = ca.id_carte
WHERE ca.titlu = '1984';
```
</details>

---

### #36 — Cu GROUP BY
**Cerință:** Care e numărul total de exemplare vândute per carte (din comenzi livrate)?

<details>
<summary>💡 Gândire</summary>
„Total exemplare... per carte" = grupare + SUM.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT ca.titlu, SUM(dc.cantitate) AS exemplare_vandute
FROM carti ca
JOIN detalii_comanda dc ON ca.id_carte  = dc.id_carte
JOIN comenzi         c  ON dc.id_comanda = c.id_comanda
WHERE c.status = 'livrata'
GROUP BY ca.id_carte, ca.titlu
ORDER BY exemplare_vandute DESC;
```
</details>

---

### #37 — Fără GROUP BY
**Cerință:** Afișează recenziile scrise în aprilie 2024, cu cartea și clientul aferent.

<details>
<summary>💡 Gândire</summary>
Filtrare pe dată, rânduri individuale de recenzii.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT r.data_recenzie, cl.nume AS client, ca.titlu, r.nota, r.comentariu
FROM recenzii r
JOIN clienti cl ON r.id_client = cl.id_client
JOIN carti   ca ON r.id_carte  = ca.id_carte
WHERE r.data_recenzie BETWEEN DATE '2024-04-01' AND DATE '2024-04-30'
ORDER BY r.data_recenzie;
```
</details>

---

### #38 — Cu GROUP BY
**Cerință:** Câte recenzii a primit fiecare gen literar și care e media notelor?

<details>
<summary>💡 Gândire</summary>
Join-uri pentru a ajunge la gen, apoi „total recenzii + medie... per gen" = GROUP BY.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT ca.gen_literar,
       COUNT(r.id_recenzie)   AS nr_recenzii,
       ROUND(AVG(r.nota), 2) AS nota_medie
FROM carti ca
JOIN recenzii r ON ca.id_carte = r.id_carte
GROUP BY ca.gen_literar
ORDER BY nota_medie DESC;
```
</details>

---

### #39 — ⚠️ Capcană — Fără GROUP BY
**Cerință:** Afișează autorii care au cel puțin o carte cu prețul sub 45 lei.

<details>
<summary>💡 Gândire</summary>
Mulți vor număra cărțile ieftine per autor. Dar vrem doar să știm că **există** cel puțin una — EXISTS e suficient.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT DISTINCT a.nume, a.nationalitate
FROM autori a
WHERE EXISTS (
    SELECT 1 FROM carti ca
    WHERE ca.id_autor = a.id_autor
      AND ca.pret < 45
);
```
</details>

---

### #40 — Cu GROUP BY (Avansat — ROLLUP)
**Cerință:** Creează un raport cu valoarea totală a comenzilor per oraș, plus un total general.

<details>
<summary>💡 Gândire</summary>
„Total per oraș + total general" = GROUP BY cu ROLLUP pentru totaluri parțiale și generale.
</details>

<details>
<summary>✅ Soluție</summary>

```sql
SELECT
    COALESCE(cl.oras, '── TOTAL GENERAL ──') AS oras,
    COUNT(c.id_comanda)  AS nr_comenzi,
    SUM(c.total)         AS valoare_totala
FROM clienti cl
JOIN comenzi c ON cl.id_client = c.id_client
WHERE c.status = 'livrata'
GROUP BY ROLLUP(cl.oras)
ORDER BY GROUPING(cl.oras), cl.oras;
```
</details>

---

## 📊 Recapitulare vizuală

| Semnal din cerință | Folosești | Exemplu |
|---|---|---|
| „afișează toate...", „lista..." | ❌ fără | lista cărților de tip Roman |
| „câți / câte... per X" | ✅ cu | câte cărți per gen |
| „cel mai mare / mic din..." | ❌ fără (subquery) | cartea cu cel mai mic preț |
| „total / sumă... per X" | ✅ cu | venit total per autor |
| „media... per X" | ✅ cu | nota medie per carte |
| „mai mult decât media..." | ❌ fără (subquery) | cărți cu preț > media |
| „grupurile cu mai mult de..." | ✅ cu + HAVING | autori cu > 2 cărți |
| „există cel puțin un..." | ❌ fără (EXISTS/IN) | autori cu cărți sub 45 lei |
| „top N per..." | ✅ cu + LIMIT | top 3 clienți |
| „totaluri parțiale + general" | ✅ cu ROLLUP | vânzări per oraș + total |

---

## 🚨 Cele mai frecvente greșeli

### Greșeala 1 — GROUP BY când vrei detalii
```sql
-- ❌ Greșit: pierzi detaliile individuale
SELECT id_client, SUM(total)
FROM comenzi
GROUP BY id_client;
-- Dacă voiai lista comenzilor, nu totalul, nu pui GROUP BY!

-- ✅ Corect pentru lista detaliată
SELECT id_client, id_comanda, data_comanda, total
FROM comenzi
ORDER BY id_client;
```

### Greșeala 2 — WHERE în loc de HAVING (sau invers)
```sql
-- ❌ Greșit: nu poți filtra un agregat cu WHERE
SELECT gen_literar, AVG(pret) FROM carti
WHERE AVG(pret) > 50        -- eroare!
GROUP BY gen_literar;

-- ✅ Corect: HAVING filtrează după agregare
SELECT gen_literar, AVG(pret) FROM carti
GROUP BY gen_literar
HAVING AVG(pret) > 50;

-- ✅ Corect: WHERE filtrează înainte de agregare (pe coloane normale)
SELECT gen_literar, AVG(pret) FROM carti
WHERE an_publicare > 1950   -- OK, filtrare pe rând, nu pe agregat
GROUP BY gen_literar;
```

### Greșeala 3 — COUNT(*) vs COUNT(DISTINCT ...)
```sql
-- ❌ Poate număra duplicate (dacă un client are mai multe comenzi cu aceeași carte)
SELECT id_client, COUNT(*) AS nr_carti_cumparate
FROM detalii_comanda dc
JOIN comenzi c ON dc.id_comanda = c.id_comanda
GROUP BY id_client;

-- ✅ Numără titluri distincte per client
SELECT id_client, COUNT(DISTINCT dc.id_carte) AS titluri_distincte
FROM detalii_comanda dc
JOIN comenzi c ON dc.id_comanda = c.id_comanda
GROUP BY id_client;
```

---
