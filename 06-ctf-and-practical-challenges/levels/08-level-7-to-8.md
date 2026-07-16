# Bandit 7 → 8 — wyszukiwanie tekstu przez `grep`

## 🎯 Cel

Odnaleźć w dużym pliku linię powiązaną z określonym słowem-kluczem.

## 🧠 Nowe pojęcie

`grep` filtruje linie wejścia i pokazuje te, które pasują do wzorca.

## 🧰 Poznane wzorce

```bash
grep "<wzorzec>" <plik>
grep -n "<wzorzec>" <plik>
```

Opcja `-n` dodaje numer znalezionej linii.

## 🔎 Mój sposób myślenia

Nie czytam całego pliku wzrokiem. Używam znanego punktu zaczepienia i zawężam dane do pasujących linii.

## ⚠️ Pułapka

Wielkość liter może mieć znaczenie. Do wyszukiwania bez rozróżniania wielkości służy `grep -i`.

## 🛡️ Znaczenie w cyberbezpieczeństwie

To podstawowy mechanizm filtrowania logów po adresach IP, użytkownikach, błędach i identyfikatorach zdarzeń.

---

[← Powrót do README](../README.md)
