# Bandit 17 → 18 — porównywanie plików przez `diff`

## 🎯 Cel

Odnaleźć jedyną linię zmienioną między starą i nową wersją pliku.

## 🧠 Nowe pojęcie

`diff` porównuje pliki linia po linii i pokazuje, co usunięto, dodano lub zmieniono.

## 🧰 Poznane polecenie

```bash
diff <stary-plik> <nowy-plik>
```

W klasycznym wyniku:

```text
< linia ze starego pliku
> linia z nowego pliku
```

## 🔎 Mój sposób myślenia

Nie analizuję setek linii ręcznie. Używam narzędzia, które pokazuje wyłącznie różnice.

## ⚠️ Pułapka

Kierunek porównania ma znaczenie. Znak `<` odnosi się do pierwszego pliku, a `>` do drugiego.

## 🛡️ Znaczenie w cyberbezpieczeństwie

`diff` pomaga wykrywać zmiany konfiguracji, reguł, skryptów i plików systemowych.

---

[← Powrót do README](../README.md)
