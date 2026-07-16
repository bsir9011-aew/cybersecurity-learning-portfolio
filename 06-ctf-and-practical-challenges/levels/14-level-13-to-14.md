# Bandit 13 → 14 — klucz prywatny SSH i `scp`

## 🎯 Cel

Zalogować się na kolejne konto za pomocą dostarczonego klucza prywatnego zamiast hasła.

## 🧠 Nowe pojęcie

Klucz prywatny nie jest tekstem wpisywanym w polu hasła. Jest plikiem przekazywanym klientowi SSH przez opcję `-i`.

W aktualnej konfiguracji OverTheWire połączenie między poziomami przez `localhost` może być blokowane. Klucz trzeba wtedy bezpiecznie pobrać na własny komputer i użyć do połączenia z serwerem zewnętrznym.

## 🧰 Poznane wzorce

```bash
scp -P <port> <użytkownik>@<host>:<zdalny-plik> .
ssh -i <klucz-prywatny> <użytkownik>@<host> -p <port>
```

W `scp` port podaje się przez **duże `-P`**, a w `ssh` przez małe `-p`.

## 🔎 Mój sposób myślenia

```text
klucz na serwerze → bezpieczny transfer przez SCP → zapis lokalny → SSH z opcją -i
```

## ⚠️ Pułapka

Na Windowsie plik skopiowany przez Notatnik może zawierać niewidoczny znak BOM lub niewłaściwe końce linii. Objawem może być:

```text
Load key: invalid format
```

Pomogło zapisanie czystej treści jako ASCII/UTF-8 bez BOM. Klucza prywatnego nigdy nie dodaję do repozytorium.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Uwierzytelnianie kluczem jest standardem administracji serwerami. Utrata klucza prywatnego może oznaczać przejęcie dostępu.

---

[← Powrót do README](../README.md)
