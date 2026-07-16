# Bandit 12 → 13 — hexdump i wielowarstwowa kompresja

## 🎯 Cel

Odtworzyć plik z hexdumpu, a następnie zdejmować kolejne warstwy kompresji i archiwizacji.

## 🧠 Nowe pojęcie

Plik może być wielokrotnie pakowany różnymi formatami. Nie należy zgadywać kolejnego kroku — po każdej operacji trzeba ponownie sprawdzić typ danych.

## 🧰 Poznane narzędzia

```bash
mktemp -d
cp <plik> <katalog-roboczy>
xxd -r <hexdump> <plik-wynikowy>
file <plik>
gzip -d <plik>
bzip2 -d <plik>
tar -xf <archiwum>
```

## 🔎 Mój sposób myślenia

```text
hexdump → dane binarne → file → właściwe narzędzie → file → kolejna warstwa
```

Najważniejsza zasada:

> Nie zgaduję rozszerzenia. Patrzę na wynik `file`.

## ⚠️ Pułapka

Narzędzia dekompresujące często oczekują właściwego rozszerzenia. Czasem trzeba najpierw zmienić nazwę pliku, ale dopiero po rozpoznaniu formatu.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Tak samo analizuje się wielowarstwowe załączniki, archiwa, droppery i dane poddane obfuskacji.

---

[← Powrót do README](../README.md)
