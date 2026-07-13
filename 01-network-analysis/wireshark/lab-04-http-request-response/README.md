# Lab 04 — HTTP Request and Response Analysis

## Cel

Celem laboratorium jest zrozumienie, jak klient wysyła żądanie HTTP do serwera oraz jak serwer zwraca odpowiedź.

## Scenariusz

Po ustanowieniu połączenia TCP klient wysyła żądanie do serwera [WWW](http://WWW).

W przypadku nieszyfrowanego HTTP treść komunikacji może być widoczna bezpośrednio w Wiresharku.

Typowy schemat:

`TCP handshake → HTTP request → HTTP response`

## Zastosowane filtry

Podstawowy filtr HTTP:

`http`

Tylko żądania:

`http.request`

Tylko odpowiedzi:

`http.response`

Żądania metodą GET:

`http.request.method == "GET"`

Żądania do konkretnej domeny:

`http.host == "example.com"`

## Żądanie HTTP

Klient wysyła żądanie do serwera.

Przykład:

```http
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
```

Najważniejsze elementy:

* `GET` — metoda żądania,
* `/index.html` — żądany zasób,
* `HTTP/1.1` — wersja protokołu,
* `Host` — domena serwera,
* `User-Agent` — informacja o kliencie,
* `Accept` — typ danych, które klient może odebrać.

## Metoda GET

Metoda `GET` służy do pobierania zasobu z serwera.

Przykłady zasobów:

* strona HTML,
* obraz,
* plik CSS,
* skrypt JavaScript,
* dane z API.

Przykład:

`GET /images/logo.png HTTP/1.1`

Oznacza to, że klient prosi serwer o plik `logo.png`.

## Odpowiedź HTTP

Serwer zwraca odpowiedź zawierającą kod statusu, nagłówki oraz opcjonalną treść.

Przykład:

```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1256
Server: nginx
```

Najważniejsze elementy:

* `200 OK` — żądanie zakończyło się powodzeniem,
* `Content-Type` — rodzaj zwróconych danych,
* `Content-Length` — długość odpowiedzi,
* `Server` — oprogramowanie serwera.

## Najważniejsze kody statusu

### 2xx — sukces

* `200 OK` — żądanie wykonane poprawnie,
* `201 Created` — utworzono nowy zasób,
* `204 No Content` — żądanie wykonane, ale bez treści odpowiedzi.

### 3xx — przekierowanie

* `301 Moved Permanently` — trwałe przekierowanie,
* `302 Found` — tymczasowe przekierowanie,
* `304 Not Modified` — zasób nie zmienił się.

### 4xx — błąd klienta

* `400 Bad Request` — nieprawidłowe żądanie,
* `401 Unauthorized` — wymagane uwierzytelnienie,
* `403 Forbidden` — brak uprawnień,
* `404 Not Found` — zasób nie istnieje.

### 5xx — błąd serwera

* `500 Internal Server Error` — błąd aplikacji serwera,
* `502 Bad Gateway` — problem z serwerem pośredniczącym,
* `503 Service Unavailable` — usługa chwilowo niedostępna.

## Kierunek komunikacji

Przykładowe żądanie:

`192.168.1.29:52341 → adres_serwera:80`

Przykładowa odpowiedź:

`adres_serwera:80 → 192.168.1.29:52341`

Port `80` jest standardowym portem nieszyfrowanego HTTP.

Port klienta jest tymczasowym portem dynamicznym przydzielonym przez system operacyjny.

## HTTP a HTTPS

### HTTP

* standardowy port: `80`,
* brak szyfrowania,
* treść żądań i odpowiedzi może być widoczna,
* dane mogą zostać przechwycone.

### HTTPS

* standardowy port: `443`,
* komunikacja jest zabezpieczona przez TLS,
* treść HTTP jest szyfrowana,
* Wireshark pokazuje zazwyczaj `Application Data`.

Schemat:

`HTTPS = HTTP przesyłany wewnątrz szyfrowanego połączenia TLS`


## Najważniejsze obserwacje

* klient inicjuje żądanie HTTP,
* serwer odpowiada kodem statusu,
* metoda `GET` służy do pobierania zasobów,
* nagłówek `Host` wskazuje domenę,
* odpowiedź może zawierać nagłówki i treść,
* HTTP nie zapewnia szyfrowania,
* HTTPS chroni treść komunikacji przy użyciu TLS.

## Czego się nauczyłem

* rozpoznawania żądań i odpowiedzi HTTP,
* identyfikowania metody `GET`,
* odczytywania kodów statusu,
* analizowania nagłówków HTTP,
* określania kierunku komunikacji,
* rozróżniania HTTP i HTTPS,
* stosowania filtrów HTTP w Wiresharku.

## Znaczenie w cyberbezpieczeństwie

Analiza HTTP może pomóc wykrywać:

* pobieranie podejrzanych plików,
* komunikację z podejrzanymi domenami,
* przesyłanie danych bez szyfrowania,
* nietypowe metody HTTP,
* błędy uwierzytelnienia,
* podejrzane nagłówki `User-Agent`,
* kontakt z serwerem dowodzenia atakującego.

Przykładowy podejrzany ruch:

```http
GET /payload.exe HTTP/1.1
Host: suspicious-domain.example
```

Może wskazywać na próbę pobrania złośliwego pliku.

## Wnioski

HTTP umożliwia wymianę żądań i odpowiedzi pomiędzy klientem a serwerem [WWW](http://WWW).

Podstawowy schemat komunikacji:

`żądanie klienta → kod odpowiedzi serwera → przesłanie zasobu`

Nieszyfrowany HTTP pozwala zobaczyć treść komunikacji, dlatego obecnie większość serwisów wykorzystuje HTTPS zabezpieczony protokołem TLS.

