# Bandit 8 → 9 — sortowanie i unikalna linia

## 🎯 Cel

Wskazać jedyną linię, która występuje w pliku tylko raz.

## 🧠 Nowe pojęcie

`uniq` porównuje **sąsiadujące** linie. Dlatego dane należy wcześniej uporządkować przez `sort`.

## 🧰 Poznany potok

```bash
sort <plik> | uniq -u
```

- `|` przekazuje wynik pierwszego polecenia do drugiego,
- `uniq -u` pokazuje wyłącznie linie występujące jeden raz.

## 🔎 Mój sposób myślenia

Najpierw grupuję identyczne wartości obok siebie, a potem proszę o pokazanie wyjątków.

## ⚠️ Pułapka

Samo `uniq -u <plik>` może dać błędny obraz, jeżeli duplikaty nie znajdują się obok siebie.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Sortowanie, agregowanie i wyszukiwanie odstających wartości to podstawowy sposób redukcji szumu.

---

[← Powrót do README](../README.md)
