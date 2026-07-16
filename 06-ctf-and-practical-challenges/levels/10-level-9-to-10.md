# Bandit 9 → 10 — czytelne ciągi w pliku binarnym

## 🎯 Cel

Wyciągnąć nieliczne tekstowe fragmenty z pliku zawierającego głównie dane binarne.

## 🧠 Nowe pojęcie

`strings` wyszukuje sekwencje drukowalnych znaków w danych binarnych. Wynik można dalej filtrować przez `grep`.

## 🧰 Poznany potok

```bash
strings <plik> | grep "<charakterystyczny-wzorzec>"
```

## 🔎 Mój sposób myślenia

1. Rozpoznaję, że zwykłe `cat` nie jest właściwym narzędziem.
2. Wydobywam tylko fragmenty tekstowe.
3. Zawężam wynik po znanym markerze.

## ⚠️ Pułapka

`strings` nie interpretuje struktury pliku — jedynie wydobywa drukowalne ciągi. Wynik zawsze wymaga oceny kontekstu.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Polecenie pomaga szybko znaleźć domeny, ścieżki, komunikaty i inne artefakty wewnątrz plików wykonywalnych.

---

[← Powrót do README](../README.md)
