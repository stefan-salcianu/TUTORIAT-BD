# 👨🏽‍🏫 Pregătire COLOCVIU

---

## 📋 Set 1

<img width="679" height="510" alt="image" src="https://github.com/user-attachments/assets/22da4472-9aba-4124-90b5-03a544959713" />

### Exercițiul 1


```sql
SELECT DISTINCT 
    T.nume, 
    T.prenume,
    F.denumire AS denumire_facilitate
FROM TURIST T
JOIN TURIST_CAZARE TC ON T.id = TC.id_turist
JOIN CAZARE CZ        ON TC.id_cazare = CZ.id
JOIN CAMERA C         ON CZ.id_camera = C.id
LEFT JOIN FACILITATI_HOTEL FH ON C.id_hotel = FH.id_hotel
LEFT JOIN FACILITATE F        ON FH.id_facilitate = F.id;
```

**Explicație:**
- Folosim `LEFT JOIN` pentru facilitățile hotelului pentru a include și turiștii cazați la hoteluri fără facilități
- `DISTINCT` elimină duplicatele (un turist poate avea acces la mai multe facilități)

---

### Exercițiul 2

```sql
SELECT 
    H.denumire AS denumire_hotel,
    COUNT(DISTINCT CASE WHEN T.localitate_domiciliu = 'Bucuresti' THEN T.id END) AS turisti_Bucuresti,
    COUNT(DISTINCT CASE WHEN T.localitate_domiciliu = 'Iasi' THEN T.id END) AS turisti_Iasi,
    COUNT(DISTINCT CASE WHEN T.localitate_domiciliu = 'Timisoara' THEN T.id END) AS turisti_Timisoara
FROM HOTEL H
JOIN CAMERA C         ON H.id = C.id_hotel
JOIN CAZARE CZ        ON C.id = CZ.id_camera
JOIN TURIST_CAZARE TC ON CZ.id = TC.id_cazare
JOIN TURIST T         ON TC.id_turist = T.id
WHERE T.localitate_domiciliu IN ('Bucuresti', 'Iasi', 'Timisoara')
GROUP BY H.id, H.denumire;
```

**Explicație:**
- Folosim `CASE WHEN` pentru a crea coloane separate pentru fiecare oraș
- `COUNT(DISTINCT ...)` asigură că turiștii sunt numărați o singură dată
- `WHERE IN` filtrează doar orașele dorite

---

### Exercițiul 3

```sql
WITH CapacitateHotel AS (
    -- Pasul 1: Calculăm capacitatea totală pentru fiecare hotel
    SELECT id_hotel, SUM(capacitate) AS capacitate_totala
    FROM CAMERA
    GROUP BY id_hotel
),
HoteluriCapacitateMaxima AS (
    -- Pasul 2: Selectăm doar hotelurile care au capacitatea maximă absolută
    SELECT id_hotel
    FROM CapacitateHotel
    WHERE capacitate_totala = (SELECT MAX(capacitate_totala) FROM CapacitateHotel)
),
TarifCurentHotel AS (
    -- Pasul 3: Calculăm cel mai mic tarif curent (activ azi) pentru hotelurile selectate anterior
    SELECT C.id_hotel, MIN(TF.tarif) AS tarif_minim_curent
    FROM CAMERA C
    JOIN TARIF_CAMERA TF ON C.id = TF.id_camera
    WHERE C.id_hotel IN (SELECT id_hotel FROM HoteluriCapacitateMaxima)
      AND SYSDATE BETWEEN TF.data_start AND TF.data_end
    GROUP BY C.id_hotel
)
-- Rezultat final: Extragem denumirea hotelului care are tariful cel mai mic dintre cele cu capacitate maximă
SELECT H.denumire
FROM HOTEL H
JOIN TarifCurentHotel TCH ON H.id = TCH.id_hotel
WHERE TCH.tarif_minim_curent = (SELECT MIN(tarif_minim_curent) FROM TarifCurentHotel);
```

**Explicație:**
- Folosim **CTE (Common Table Expressions)** pentru a structura logica în pași
- **Pasul 1:** Agregăm capacitatea totală pe hotel
- **Pasul 2:** Identificăm hotelurile cu capacitatea maximă
- **Pasul 3:** Găsim tariful minim curent pentru aceste hoteluri
- **Final:** Selectăm hotelul cu tariful cel mai mic

---

## 📋 Set 2

<img width="512" height="493" alt="image" src="https://github.com/user-attachments/assets/b25ceab8-0c08-4a90-9fb3-905d800e9850" />

### Exercițiul 1


```sql
SELECT 
    T.nume, 
    T.prenume, 
    CZ.data_cazare, 
    CZ.nr_zile AS durata_cazare, 
    H.denumire AS nume_hotel
FROM TURIST T
JOIN TURIST_CAZARE TC ON T.id = TC.id_turist
JOIN CAZARE CZ        ON TC.id_cazare = CZ.id
JOIN CAMERA C         ON CZ.id_camera = C.id
JOIN HOTEL H          ON C.id_hotel = H.id
WHERE EXTRACT(YEAR FROM (CZ.data_cazare + CZ.nr_zile)) = EXTRACT(YEAR FROM CZ.data_cazare) + 1;
```

**Explicație:**
- `CZ.data_cazare + CZ.nr_zile` calculează data de checkout
- Comparăm anul de checkout cu anul de checkin
- Dacă diferă cu exact 1, înseamnă că au petrecut Anul Nou la hotel

---

### Exercițiul 2

```sql
SELECT 
    H.localitate,
    COUNT(DISTINCT CASE WHEN EXTRACT(YEAR FROM CZ.data_cazare) = 2017 
                          OR EXTRACT(YEAR FROM (CZ.data_cazare + CZ.nr_zile)) = 2017 
                        THEN T.id END) AS turisti_2017,
    COUNT(DISTINCT CASE WHEN EXTRACT(YEAR FROM CZ.data_cazare) = 2018 
                          OR EXTRACT(YEAR FROM (CZ.data_cazare + CZ.nr_zile)) = 2018 
                        THEN T.id END) AS turisti_2018,
    COUNT(DISTINCT CASE WHEN EXTRACT(YEAR FROM CZ.data_cazare) = 2019 
                          OR EXTRACT(YEAR FROM (CZ.data_cazare + CZ.nr_zile)) = 2019 
                        THEN T.id END) AS turisti_2019
FROM HOTEL H
JOIN CAMERA C         ON H.id = C.id_hotel
JOIN CAZARE CZ        ON C.id = CZ.id_camera
JOIN TURIST_CAZARE TC ON CZ.id = TC.id_cazare
JOIN TURIST T         ON TC.id_turist = T.id
GROUP BY H.localitate;
```

**Explicație:**
- Verificăm atât data de checkin, cât și data de checkout pentru fiecare an
- Un turist este numărat pentru un an dacă a avut cazare activă în acel an (chiar și o zi)
- `DISTINCT` asigură că un turist este numărat o singură dată pe localitate și an

---

### Exercițiul 3

```sql
SELECT 
    H.denumire AS nume_hotel, 
    C.nr_camera, 
    COUNT(CZ.id) AS numar_cazari, 
    COUNT(DISTINCT TC.id_turist) AS numar_turisti_distincti
FROM HOTEL H
JOIN CAMERA C         ON H.id = C.id_hotel
JOIN CAZARE CZ        ON C.id = CZ.id_camera
JOIN TURIST_CAZARE TC ON CZ.id = TC.id_cazare
GROUP BY H.id, H.denumire, C.id, C.nr_camera
HAVING COUNT(CZ.id) >= 3;
```

**Explicație:**
- `COUNT(CZ.id)` numără toate cazările pentru camera respectivă
- `COUNT(DISTINCT TC.id_turist)` numără câți turiști unici au stat în cameră
- `HAVING` filtrează doar camerele cu ≥ 3 cazări

---

### Exercițiul 4

```sql
INSERT INTO FACILITATI_HOTEL (id_hotel, id_facilitate)
VALUES (
    -- Subinterogare pentru ID-ul hotelului cu cele mai multe camere
    (SELECT id_hotel 
     FROM CAMERA 
     GROUP BY id_hotel 
     ORDER BY COUNT(id) DESC 
     FETCH FIRST 1 ROWS ONLY),
    
    -- Subinterogare pentru ID-ul facilității "teren de tenis"
    (SELECT id 
     FROM FACILITATE 
     WHERE LOWER(denumire) = 'teren de tenis')
);
```

**Explicație:**
- Prima subinterogare găsește hotelul cu numărul maxim de camere
- A doua subinterogare găsește ID-ul facilității căutate (case-insensitive)
- `FETCH FIRST 1 ROWS ONLY` este sintaxa Oracle pentru `LIMIT 1`

---

### Exercițiul 5

```sql
-- Pasul 1: Adăugăm coloana
ALTER TABLE CAZARE 
ADD id_platnic NUMBER;

-- Pasul 2: Adăugăm constrangerea de cheie externă
ALTER TABLE CAZARE
ADD CONSTRAINT fk_cazare_platnic
FOREIGN KEY (id_platnic) REFERENCES TURIST(id)
ON DELETE CASCADE;
```

**Explicație:**
- Coloana `id_platnic` va stoca ID-ul turistului care plătește cazarea
- Poate fi diferit de turiștii cazați (unul plătește pentru grup)
- `ON DELETE CASCADE` șterge automat cazările dacă turistul plătitor este șters

---
