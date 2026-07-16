# OverTheWire Bandit — dziennik nauki 0–19 🏴‍☠️

Publiczna, **spoiler-light** dokumentacja mojej nauki podstaw Linuksa, SSH i pracy z usługami sieciowymi w grze [OverTheWire Bandit](https://overthewire.org/wargames/bandit/).

> Repozytorium nie zawiera haseł, kluczy prywatnych ani wyników będących gotowymi odpowiedziami. Pokazuje poznane mechanizmy, narzędzia i sposób rozumowania.

## 🎯 Cel repozytorium

- utrwalić praktyczne komendy Linuksa,
- rozumieć, **dlaczego** dane rozwiązanie działa,
- dokumentować rozwój pod pracę w obszarze SOC i cyberbezpieczeństwa,
- pokazać rekruterowi regularną, samodzielną pracę laboratoryjną.

## ✅ Postęp

| Etap | Główna umiejętność | Status |
|---|---|---:|
| Start | Logowanie SSH | ✅ |
| 0 → 1 | Nawigacja i odczyt pliku | ✅ |
| 1 → 2 | Plik o nazwie `-` | ✅ |
| 2 → 3 | Spacje i myślniki w nazwie | ✅ |
| 3 → 4 | Pliki ukryte | ✅ |
| 4 → 5 | Rozpoznawanie typu pliku | ✅ |
| 5 → 6 | `find` i właściwości pliku | ✅ |
| 6 → 7 | Właściciel, grupa i rozmiar | ✅ |
| 7 → 8 | Wyszukiwanie tekstu przez `grep` | ✅ |
| 8 → 9 | Sortowanie i unikalne linie | ✅ |
| 9 → 10 | Czytelne ciągi w danych binarnych | ✅ |
| 10 → 11 | Dekodowanie Base64 | ✅ |
| 11 → 12 | ROT13 i `tr` | ✅ |
| 12 → 13 | Hexdump i wielowarstwowa kompresja | ✅ |
| 13 → 14 | Klucz prywatny SSH i `scp` | ✅ |
| 14 → 15 | Netcat, localhost i porty | ✅ |
| 15 → 16 | Połączenie TLS przez OpenSSL | ✅ |
| 16 → 17 | Skanowanie portów i identyfikacja usług | ✅ |
| 17 → 18 | Porównywanie plików przez `diff` | ✅ |
| 18 → 19 | `.bashrc` i zdalne wykonanie polecenia | ✅ |
| 19 → 20 | SUID i efektywne uprawnienia | ✅ |

## 🧭 Struktura

```text
bandit-github-notes/
├── README.md
├── CHEATSHEET.md
├── .gitignore
└── levels/
    ├── 00-start-ssh.md
    ├── 01-level-0-to-1.md
    └── ...
```

## 🧠 Najważniejsza zasada

Nie zgaduję rozwiązania. Najpierw zbieram informacje:

```text
zobacz → rozpoznaj → zawęź → wykonaj → sprawdź wynik
```

## 🔐 Bezpieczeństwo

Nigdy nie publikuję:

- haseł do kolejnych poziomów,
- kluczy prywatnych,
- plików `*.key` i `*.private`,
- zrzutów ekranu zawierających dane dostępowe.
