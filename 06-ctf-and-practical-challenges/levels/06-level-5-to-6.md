# Bandit 5 → 6 — wyszukiwanie po cechach pliku

## 🎯 Cel

Odnaleźć plik spełniający kilka właściwości jednocześnie: typ, dokładny rozmiar i brak prawa wykonywania.

## 🧠 Nowe pojęcie

`find` pozwala budować filtr z wielu warunków. Zamiast przeglądać katalogi ręcznie, opisuję poszukiwany obiekt.

## 🧰 Poznany wzorzec

```bash
find <katalog> -type f -size <liczba>c ! -executable
```

- `-type f` — zwykły plik,
- `-size ...c` — dokładny rozmiar w bajtach,
- `! -executable` — plik niewykonywalny.

## 🔎 Mój sposób myślenia

Zmieniam opis zadania w zestaw filtrów i łączę je w jednym poleceniu.

## ⚠️ Pułapka

Jednostka ma znaczenie: `c` oznacza bajty. Bez niej `find` może interpretować rozmiar w innych blokach.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Filtrowanie po metadanych przyspiesza triage dużej liczby plików i artefaktów.

---

[← Powrót do README](../README.md)
