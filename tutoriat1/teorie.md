# 📑 Ședința 01 (Deep Dive): Arhitectura și Proiectarea Reluțională

În această secțiune explorăm mecanismele interne care asigură că datele rămân corecte, sigure și ușor de accesat.

---

## 🏗️ 1. Cele 3 Niveluri de Abstracție (ANSI/SPARC)
De ce separăm datele în 3 straturi? Pentru **Independența Datelor**.

1.  **Nivelul Extern (Vederi de utilizator):**
    * Ce vede user-ul. Un student vede notele lui, un administrator vede salariile profesorilor.
    * *Beneficiu:* Putem schimba structura bazei de date fără ca utilizatorul să observe vreo diferență în interfață.

2.  **Nivelul Conceptual (Schema Logică):**
    * Aici definim **ce** date stocăm și ce **relații** există între ele.
    * Folosim diagrame **Entity-Relationship (ERD)**.
    

3.  **Nivelul Intern (Schema Fizică):**
    * Cum sunt organizate fișierele pe disc (B-Trees, Hash-uri, Indexi).
    * De obicei, SGBD-ul (Oracle/MySQL) se ocupă de asta, dar performanța interogărilor tale depinde de acest nivel.

---

## 📐 2. Modelul Entitate-Relație (E-R) pe larg
Înainte de orice tabel, trebuie să identificăm:

### A. Entitățile (Substantivele)
Obiecte din lumea reală despre care salvăm date.
* *Exemplu:* `STUDENT`, `CURS`, `FACULTATE`.

### B. Atributele (Adjectivele)
Proprietățile entităților.
* **Simple:** `Nume`, `Prenume`.
* **Compuse:** `Adresa` (Formată din Stradă, Oraș, Cod Poștal).
* **Multivaluate:** `Numar_Telefon` (Un student poate avea mai multe).
* **Derivate:** `Varsta` (Calculată din `Data_Nasterii` - *Sfat: Nu stocăm vârsta în BD, o calculăm la nevoie!*).

### C. Relațiile (Verbele)
Cum interacționează entitățile.
* Un Student **se înscrie** la un Curs.

---

## 🔐 3. Integritatea Datelor (Regulile Jocului)
Fără integritate, baza de date e doar un fișier text dezordonat. Există 3 tipuri majore:

| Tip Integritate | Regula de Aur | Explicație |
| :--- | :--- | :--- |
| **Integritatea Entității** | PK ≠ NULL | Fiecare rând trebuie să poată fi identificat. Nu poți avea un student "fantomă" fără ID. |
| **Integritatea Referențială** | FK trebuie să existe | Dacă în tabelul `NOTE` pui `id_student = 500`, atunci studentul cu ID-ul 500 **trebuie** să existe în tabelul `STUDENTI`. |
| **Integritatea Domeniului** | Tip de date corect | Dacă coloana `Nota` e definită ca `NUMBER`, nu poți insera textul "Foarte Bine". |

---

## 🔄 4. Transformarea M:N (Many-to-Many)
Aceasta este cea mai frecventă problemă de design. 
* **Problemă:** Un Student are mai multe Cursuri. Un Curs are mai mulți Studenți.
* **Soluție:** Creăm un **Tabel Asociativ** (sau tabel de legătură).



**Exemplu:**
1. Tabel `STUDENTI` (PK: `id_student`)
2. Tabel `CURSURI` (PK: `id_curs`)
3. Tabel `INSCRIERI` (PK compus din: `id_student` + `id_curs`) -> Acesta este tabelul de legătură.

---

## ⚡ 5. Introducere în Obiectele SQL
Pe lângă tabele, într-o bază de date vom mai lucra cu:
* **Constraint-uri (Constrângeri):** `UNIQUE` (valori unice, dar pot fi null), `CHECK` (ex: `CHECK (nota >= 1 AND nota <= 10)`).
* **Indexi:** "Semne de carte" care accelerează căutarea (fără ei, SGBD-ul citește tot tabelul rând cu rând).
* **Vederi (Views):** Interogări salvate care se comportă ca niște tabele virtuale.

---

> 🧠 **Exercițiu de gândire:** > Dacă ștergem un Student din baza de date, ce se întâmplă cu notele lui? 
> * Opțiunea A: Se șterg automat (**ON DELETE CASCADE**).
> * Opțiunea B: Rămân acolo, dar fără "proprietar" (Eroare de integritate referențială!).
> * Opțiunea C: Nu avem voie să ștergem studentul până nu îi ștergem manual notele (**RESTRICT**).
