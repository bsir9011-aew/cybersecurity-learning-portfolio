# Bandit 6 → 7 — właściciel, grupa i błędy dostępu

## 🎯 Cel

Znaleźć plik w całym systemie na podstawie właściciela, grupy i dokładnego rozmiaru.

## 🧠 Nowe pojęcie

Każdy plik ma właściciela i grupę. Podczas przeszukiwania `/` pojawia się dużo komunikatów `Permission denied`, ponieważ zwykły użytkownik nie ma dostępu do wszystkich katalogów.

## 🧰 Poznany wzorzec

```bash
find / -type f -user <właściciel> -group <grupa> -size <liczba>c 2>/dev/null
```

`2>/dev/null` przekierowuje komunikaty błędów, pozostawiając wyniki standardowe.

## 🔎 Mój sposób myślenia

1. Zamieniam każdą cechę pliku w filtr.
2. Szukam od katalogu głównego.
3. Oddzielam wartościowe wyniki od szumu błędów.

## ⚠️ Pułapka

Ukrycie błędów jest wygodne, ale w diagnostyce produkcyjnej nie powinno się robić tego bezmyślnie — błąd może być ważną wskazówką.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Właściciel i grupa są kluczowe podczas analizy uprawnień, eskalacji uprawnień i nieprawidłowych konfiguracji.

---

[← Powrót do README](../README.md)
