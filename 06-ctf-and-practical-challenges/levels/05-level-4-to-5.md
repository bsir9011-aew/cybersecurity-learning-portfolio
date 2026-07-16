# Bandit 4 → 5 — rozpoznawanie typu pliku

## 🎯 Cel

Spośród wielu plików wskazać ten, który zawiera dane czytelne dla człowieka.

## 🧠 Nowe pojęcie

Rozszerzenie pliku nie jest wiarygodnym źródłem informacji. Polecenie `file` rozpoznaje format na podstawie zawartości i sygnatur.

## 🧰 Poznane polecenia

```bash
file ./*
cat <plik-tekstowy>
```

## 🔎 Mój sposób myślenia

Nie otwieram wszystkich plików losowo. Najpierw klasyfikuję je przez `file`, a dopiero potem odczytuję właściwy typ.

## ⚠️ Pułapka

Wyświetlenie pliku binarnego przez `cat` może zaśmiecić lub chwilowo rozregulować terminal. Pomaga wtedy polecenie `reset`.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Identyfikacja rzeczywistego formatu jest ważna przy analizie malware, załączników i plików bez poprawnego rozszerzenia.

---

[← Powrót do README](../README.md)
