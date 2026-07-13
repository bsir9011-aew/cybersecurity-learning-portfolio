# lab 02 - TCP three-way handshake analysis

## Cel

Celem jest zrozumienie w jaki sposób klient i serwer ustanawiają połączenie TCP oraz jak rozpoznać trzy etapy three way handshake w Wiresharku

## Scenariusz

Komputer użytkownika chce nawiązać połączenie z serwerem strony internetowej działającym na porcie `443`

Przed przesłaniem danych klient i serwer muszą potwierdzić, że obie strony są gotowe do komunikacji

## Zastosowany filtr

`tcp`

Dokładniejszy filtr pokazujący pakiety rozpoczynające połączenie `tcp.flags.syn == 1`

##  Przebieg komunikacji

1. SYN

Klient wysyła pakiet z ustanowioną flagą SYN, przykładowy kierunek: 192.168.1.29:52341 -> adres_serwera:443

Znaczenie:
- klient chce rozpocząć połączenie,
- port 52341 jest tymczasowo portem klienta
- port 443 oznacza usługę HTTPS
- klient przesyła swój początkowy numer sekwencyjny

2. SYN, ACK

Serwer odpowiada pakietem z flagami SYN oraz ACK

Kierunek: adres_serwera:443 -> 192.168.1.29:52341

Znaczenie:
klient potwierdza odpowiedź serwera, połączenie tcp zostaje ustanowione, strony mogą rozpocząć przesyłanie dalszych danych

## Najważniejsze obserwacje:
- TCP three-way handshake składa się z pakietów SYN, SYN-ACK oraz ACK
- Klient rozpoczyna komunikację z tymczasowego portu źródłowego
- Serwer odpowiada ze swojego portu usługi, na przykład `443`
- Adresy IP i porty zmieniają kierunek w odpowiedzi serwera
- Flaga `ACK` potwierdza otrzymanie wcześniejszego pakietu
- Sam handshake nie przesyła jeszcze treści strony internetowej
- Połączenie TCP jest ustanawiane przed rozpoczęciem komunikacji TLS i HTTPS

## Czego się nauczyłem

- rozpoznawanie TCP three-way handshake,
- interpretowanie flag `SYN` i `ACK`
- określanie kierunku komunikacji
- rozróżnianie portu klienta i portu serwera
- identyfikowanie początku połączenia TCP
- stosowanie filtrów TCP w Wiresharku

## Znaczenie w cyber

Analiza pakietów TCP może pomóc wykrywać:

- dużą liczbę nieukończonych połączeń
- skanowanie portów
- próby łączenia sięz nietypowymi usługami
- komunikację z podejrzanymi adresami IP
- ataki wykorzystujące dużą liczbę pakietów `SYN`

## Wnioski

TCP three-way handshake pozwala klientowi i serwerowi potwierdzić gotowość do komunikacji.
Sekwencja: `SYN -> SYN-ACK -> ACK`
oznacza poprawnie ustanowione połączenia TCP. Brak któregoś z etapów może wskazywać na problem sieciowy, odrzucone połączenie oraz podejrzaną aktywność
