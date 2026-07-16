# Bandit 2 → 3 — spacje i myślniki w nazwie

## 🎯 Cel

Poprawnie odwołać się do pliku zawierającego spacje oraz znaki mogące wyglądać jak opcje.

## 🧠 Nowe pojęcie

Powłoka dzieli polecenie na argumenty według spacji. Cudzysłów sprawia, że cała nazwa jest jednym argumentem.

## 🧰 Poznany wzorzec

```bash
cat -- "<nazwa pliku ze spacjami>"
```

Możliwe jest także poprzedzanie każdej spacji znakiem `\`.

## 🔎 Mój sposób myślenia

Rozdzielam dwa problemy:

1. spacje — rozwiązuję cudzysłowem,
2. początkowe myślniki — rozwiązuję przez `--`.

## ⚠️ Pułapka

Bez cudzysłowu powłoka potraktuje kolejne części nazwy jako oddzielne argumenty.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Poprawne cytowanie argumentów zapobiega błędom skryptów i części podatności typu command injection.

---

[← Powrót do README](../README.md)
