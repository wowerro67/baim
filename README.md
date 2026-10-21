# Podejrzane pliki kontra serwis do przesyłania danych

Projekt przedstawiający architekturę bezpiecznego przesyłania plików (file upload) w aplikacjach internetowych i mobilnych. Prezentuje zagrożenia wynikające z braku odpowiedniej walidacji oraz demonstruje, jak zaprojektować system odporny na krytyczne podatności poprzez zastosowanie obrony warstwowej (defense in depth).

## Architektura i etapy przetwarzania

Proces bezpiecznego uploadu został podzielony na cztery odizolowane etapy, aby zminimalizować ryzyko ataku na każdym etapie przetwarzania:

1. **Brama / API (Odbiór i wstępna weryfikacja):** 
   System weryfikuje żądanie, sprawdzając limity rozmiaru pliku oraz stosuje rate limiting. Walidacja formatu opiera się na rygorystycznej białej liście (allowlista) oraz sprawdzaniu rzeczywistych sygnatur plików (magic bytes), ignorując nagłówki Content-Type przesyłane przez klienta. Nazwa pliku jest zamieniana na losowy identyfikator (UUID).

2. **Kwarantanna (Tymczasowa izolacja):** 
   Plik trafia do odizolowanego magazynu (storage), który nie posiada publicznego dostępu. Serwer ma w tym miejscu całkowicie wyłączoną możliwość wykonywania jakichkolwiek skryptów, co zapobiega uruchomieniu złośliwego kodu.

3. **Środowisko detekcji (Skanowanie i oczyszczanie):** 
   Plik jest asynchronicznie skanowany przez oprogramowanie antywirusowe oraz analizowany w środowisku sandbox. W przypadku obrazów następuje ich rekompresja w celu usunięcia potencjalnie niebezpiecznych metadanych (np. EXIF, pliki polyglot). Dokumenty mogą być poddane procesowi odbudowy zawartości (CDR).

4. **Czysty magazyn (Bezpieczne serwowanie):** 
   Po pozytywnej weryfikacji plik jest przenoszony do docelowego magazynu obiektowego (Object Storage). Pliki są serwowane z zupełnie osobnej domeny, co zapobiega atakom XSS na główną sesję aplikacji, a odpowiedzi serwera wymuszają bezpieczne nagłówki (m.in. `X-Content-Type-Options: nosniff`).

## Główne założenia i wektory ataków

* **Mitygowane zagrożenia:** Zdalne wykonanie kodu (RCE), Cross-Site Scripting (XSS), Path Traversal, ataki typu DoS (Zip Bomb) oraz Zip Slip.
* **Aspekt mobilny:** Wdrożenie dodatkowych zabezpieczeń dla systemów Android i iOS, takich jak korzystanie z `FileProvider`, blokowanie WebView dla plików lokalnych oraz bezpośredni upload do chmury za pomocą krótkotrwałych, podpisanych adresów URL (Pre-signed URL).
