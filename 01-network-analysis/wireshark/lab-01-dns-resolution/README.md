# Lab 01 - DNS Resolution Analysis

##  Cel
Celem jest zrozumienie jak komputer uzyskuje adres IP domeny za pomocą protokołu DNS oraz jak rozpoznać zapytanie i odpowiedź DNS w Wiresharku

## Scenariusz
User wpisuje adres strony w przeglądarce. Komputer nie zna jej adresu IP, dlatego wysyła zapytanie do serwera DNS

Zastosowano filtr `dns`
Filtr wyświetla pakiety rozpoznane przez Wiresharka jako ruch DNS

## Przebieg komunikacji

1. Zapytanie DNS

Komputer użytkownika wysyła zapytanie do serwera DNS
Kierunek: 192.168.1.29 -> 1.1.1.1

port źródłowy: tymczasowy port klienta np 53542
port docelowy: 53
typ zapytania: A
cel: uzyskanie adresu IPv4 domeny

2. Odpowiedź DNS

Serwer DNS odsyła odpowiedź do komputera użytkownika

1.1.1.1 -> 192.168.1.29

z portu 53 na port użyty wcześniej przez klienta. Odpowiedź to adres IP przypisany do domeny.
Rekord ,,A'' mapuje nazwę domenową na adres IPv4, ,,AAAA" działa podobnie tylko na IPv6

przykład example.com -> 93.184.216.34

## Najważniejsze obserwacje

- DNS działa jak książka telefoniczna internetu,
- Komputer pyta o adres IP, ponieważ komunikacja sieciowa odbywa się między adresami IP, a nie samymi nazwami domen
- Klient używa tymczasowo przypisanego portu źródłowego z puli portów dynamicznych
- Serwer DNS nasłuchuje standardowo na porcie 53
- Odpowiedź wraca na ten sam port klienta, z którego wysłano zapytania
- Po otrzymaniu adresu IP komputer może rozpocząć połączenie z serwerem docelowym

## Czego się nauczyłem

- rozpoznanie zapytania i odpowiedzi DNS
- określanie kierunku komunikacji
- identyfikowanie portów źródłowych i docelowych
- rozróżnianie rekordów A i AAAA
- stosowanie podstawowego filtru DNS w Wiresharku
- interpretowanie procesu rozwiązywania nazw domenowych

## Wnioski

Dns jest pierwszym etapem wielu połączeń sieciowych. Zanim komputer połączy się ze stroną internetową, najczęściej musi najpierw ustalić jej adres IP.

Analiza DNS może również pomóc analitykowi bezpieczeństwa wykryć podejrzane domeny, nietypoe zapytania oraz komunikację z infrastrukturą należącą do atakującego
