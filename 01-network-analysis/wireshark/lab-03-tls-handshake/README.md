# Lab 03 - tls handshake analysis

## Cel

Celem jest zrozumienie jak klient i serwer ustanawiają `szyfrowane połączenie TLS` oraz jak rozpoznać jego najważniejsze etapy w Wiresharku

## Scenariusz

Po rozwiązaniu nazwy domeny przez DNS i ustanowieniu połączenia TCP klient rozpoczyna negocjację TLS z serwerem.
Przykładowa kolejność:
`DNS -> TCP handshake -> TLS handshake -> zaszyfrowane dane w aplikacji`

## Zastosowane filtry

Podstawowy filtr:

`tls`

Pakiety negocjacji TLS:

`tls.handshake`

Pakiety Client Hello:

`tls.handshake.type == 1`

Pakiety Server Hello:

`tls.handshake.type == 2`

## Przebieg komunikacji
1. Client Hello

Klient wysyła wiadomość Client Hello do serwera.

Przykładowy kierunek:

`192.168.1.29:52341 → adres_serwera:443`

Client Hello może zawierać:

obsługiwane wersje TLS,
proponowane zestawy szyfrów,
rozszerzenia protokołu,
nazwę domeny w polu SNI,
parametry potrzebne do uzgodnienia kluczy sesji.
SNI - Server Name Indication

Pole SNI informuje serwer, z jaką domeną klient chce się połączyć.

Jest to potrzebne, ponieważ pod jednym adresem IP może działać wiele różnych domen.

Przykład:

`server_name: portal.example.com`

W klasycznym TLS pole SNI jest widoczne podczas analizy ruchu, mimo że późniejsze dane aplikacji są szyfrowane.

2. Server Hello

Serwer odpowiada wiadomością Server Hello.

Przykładowy kierunek:

`adres_serwera:443 → 192.168.1.29:52341`

Serwer wybiera spośród opcji obsługiwanych przez obie strony:

wersję TLS,
zestaw szyfrów,
parametry kryptograficzne sesji.

Przykład:

`TLS 1.3`

`TLS_AES_128_GCM_SHA256`

Serwer nie wybiera TLS 1.3 wyłącznie dlatego, że jest nowszy. Wybiera wersję, którą obsługują zarówno klient, jak i serwer, zgodnie z ich konfiguracją i zasadami bezpieczeństwa.

Znaczenie zestawu szyfrów

Przykładowy zestaw:

`TLS_AES_128_GCM_SHA256`

Oznacza:

`AES_128` — algorytm szyfrowania z kluczem 128-bitowym,
`GCM` — tryb szyfrowania zapewniający również kontrolę integralności danych,
`SHA256` — funkcja skrótu używana w mechanizmach TLS.

SHA-256 nie służy tutaj po prostu do „sprawdzenia certyfikatu”. Jest elementem szerszych mechanizmów integralności i uwierzytelniania używanych przez TLS.

Certyfikat serwera

Serwer przedstawia klientowi certyfikat cyfrowy.

Klient sprawdza między innymi:

czy certyfikat nie wygasł,
czy dotyczy właściwej domeny,
czy został wystawiony przez zaufany urząd certyfikacji,
czy łańcuch certyfikatów jest poprawny,
czy podpis cyfrowy certyfikatu jest prawidłowy.

Certyfikat pomaga potwierdzić, że klient komunikuje się z właściwym serwerem.

Certificate Verify i Finished

Serwer może wysłać wiadomość Certificate Verify, która potwierdza, że posiada klucz prywatny odpowiadający przedstawionemu certyfikatowi.

Wiadomości Finished potwierdzają, że negocjacja TLS przebiegła prawidłowo i obie strony wyliczyły odpowiednie klucze sesji.

W TLS 1.3 część komunikatów handshake jest już szyfrowana, dlatego nie wszystkie informacje muszą być czytelnie widoczne w przechwyconym ruchu.

## Application Data

Po zakończeniu handshake Wireshark pokazuje pakiety opisane jako:

Application Data

Oznacza to, że przesyłane są dane aplikacji, na przykład treść komunikacji HTTPS.

Można nadal zobaczyć między innymi:

adres źródłowy i docelowy,
porty,
kierunek komunikacji,
długość pakietów,
czas transmisji,
używany protokół transportowy.

Bez kluczy szyfrujących nie można jednak odczytać właściwej treści danych.


## Najważniejsze obserwacje

TLS jest ustanawiany po nawiązaniu połączenia TCP.
Client Hello przedstawia możliwości klienta.
Server Hello zawiera wybór parametrów połączenia.
SNI może ujawniać nazwę domeny.
Certyfikat służy do uwierzytelnienia serwera.
TLS 1.3 szyfruje część komunikatów handshake.
Application Data oznacza przesyłanie zaszyfrowanych danych aplikacji.
Szyfrowanie ukrywa treść, ale nie wszystkie metadane komunikacji.

## Czego się nauczyłem

rozpoznawania Client Hello i Server Hello,
identyfikowania wersji TLS,
odczytywania zestawu szyfrów,
odnajdywania pola SNI,
rozumienia roli certyfikatu,
rozpoznawania zaszyfrowanych danych aplikacji,
analizowania metadanych zaszyfrowanego ruchu.
Znaczenie w cyberbezpieczeństwie

Analiza TLS może pomóc wykrywać:

połączenia z podejrzanymi domenami,
nieważne lub nietypowe certyfikaty,
używanie przestarzałych wersji TLS,
nietypowe zestawy szyfrów,
komunikację z infrastrukturą atakującego,
anomalie w częstotliwości i kierunku połączeń.

   
