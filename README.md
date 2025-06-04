# 📦 Inwentaryzacja (Vue 3 + Vite)

Prosta aplikacja do inwentaryzacji oparta na Vue 3 + Vite. Umożliwia skanowanie lub ręczne wprowadzanie kodów, kojarzenie ich z nazwami produktów i zapisywanie wyników lokalnie z możliwością eksportu do CSV.

---

## 🚀 Funkcje

- 📷 Skanowanie kodów kreskowych (EAN13 i inne)
- ✍️ Wprowadzanie danych: indeks, ilość, lokalizacja
- 🧠 Kojarzenie kodów z nazwami produktów z załadowanych plików CSV
- 💾 Zapisywanie danych lokalnie (`localStorage`)
- 🔃 Sortowanie po kolumnach (Lp, Indeks, Nazwa, Czas, Lokalizacja)
- ✅ Dodawanie wiersza jednym kliknięciem
- 🗑 Usuwanie pojedynczych wpisów lub całej tabeli
- 📤 Eksport danych do pliku CSV
- 📱 Przyjazny interfejs mobilny

---

## 🛠️ Uruchomienie lokalne

```bash
# 1. Sklonuj repozytorium
git clone https://github.com/TwojeKonto/inwentaryzacja-app.git

# 2. Wejdź do katalogu
cd inwentaryzacja-app

# 3. Zainstaluj zależności
npm install

# 4. Uruchom serwer developerski
npm run dev
```

Aplikacja będzie dostępna pod adresem: `http://localhost:5173`

---

## 🌍 Wdrożenie online

### ✅ Vercel (zalecane)
1. Wejdź na [https://vercel.com](https://vercel.com) i połącz konto GitHub
2. Wybierz to repozytorium i kliknij **Deploy**
3. Vercel automatycznie rozpozna Vite i skonfiguruje środowisko

### ✅ Alternatywnie: Netlify
1. Uruchom build:
   ```bash
   npm run build
   ```
2. Wejdź na [https://app.netlify.com/drop](https://app.netlify.com/drop)
3. Przeciągnij folder `dist/`

---

## 📂 Struktura plików

```
src/
├── components/
│   └── InwentaryTableRow.vue
├── App.vue
├── main.js
├── assets/
│   └── style.css
public/
  └── favicon.svg
```

---

## 📄 Pliki danych (CSV)

**Plik z nazwami produktów:**
```
Indeks,Nazwa
0001,Sansevieria
0002,Ficus lyrata
...
```

**Plik z kodami kreskowymi:**
```
Indeks,KodKres
0001,5903931000017
0001,0001
0002,5903931000024
...
```

Użytkownik ładuje je lokalnie na początku działania aplikacji.

---

## 📦 Build produkcyjny

```bash
npm run build
```
Wynik znajdziesz w folderze `dist/`.

---

## 👤 Autor

Mikołaj Śnieć  
🛠 Tomaszewski – produkcja i dystrybucja roślin doniczkowych

---

## 📝 Licencja
MIT License
