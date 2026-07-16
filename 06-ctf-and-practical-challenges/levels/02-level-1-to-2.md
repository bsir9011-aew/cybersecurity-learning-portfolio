# Bandit 1 → 2 — plik o nazwie `-`

## 🎯 Cel

Odczytać plik, którego nazwa składa się z pojedynczego myślnika.

## 🧠 Nowe pojęcie

Wiele programów traktuje `-` jako standardowe wejście, a nazwy zaczynające się od myślnika jako opcje. Trzeba jednoznacznie wskazać, że chodzi o plik.

## 🧰 Poznane wzorce

```bash
cat ./-
cat -- -
```

- `./` wskazuje plik w bieżącym katalogu,
- `--` kończy listę opcji programu.

## 🔎 Mój sposób myślenia

Gdy nazwa pliku koliduje ze składnią polecenia, podaję pełniejszą ścieżkę albo kończę interpretację opcji.

## ⚠️ Pułapka

Samo `cat -` nie musi oznaczać pliku — program może wtedy czekać na dane wpisywane z klawiatury.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Nietypowe nazwy plików bywają używane do ukrywania artefaktów lub utrudniania pracy administratora.

---

[← Powrót do README](../README.md)
