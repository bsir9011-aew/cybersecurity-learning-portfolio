# Bandit 11 → 12 — ROT13 i translacja znaków

## 🎯 Cel

Odwrócić prostą transformację alfabetu, w której każdą literę przesunięto o 13 pozycji.

## 🧠 Nowe pojęcie

ROT13 jest szyfrem podstawieniowym, ale nie zapewnia realnego bezpieczeństwa. Dwukrotne zastosowanie ROT13 przywraca tekst pierwotny.

## 🧰 Poznane polecenie

```bash
tr 'A-Za-z' 'N-ZA-Mn-za-m' < <plik>
```

`tr` zamienia znaki ze zbioru pierwszego na odpowiadające im znaki ze zbioru drugiego.

## 🔎 Mój sposób myślenia

Rozbijam alfabet na dwie połowy i definiuję jednoznaczną translację zarówno dla małych, jak i wielkich liter.

## ⚠️ Pułapka

ROT13 dotyczy liter. Cyfry i znaki specjalne pozostają bez zmian.

## 🛡️ Znaczenie w cyberbezpieczeństwie

To prosty przykład różnicy między transformacją danych a prawdziwym szyfrowaniem.

---

[← Powrót do README](../README.md)
