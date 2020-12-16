# Dokumentacja bibliotek mobilnych Przelewy24

Biblioteki mobilne Przelewy24 umożliwiają integrację płatności Przelewy24 w aplikacjach mobilnych, analogicznie jak w przypadku płatności web. Poniższa dokumentacja bazuje na dokumentacji integracji płatności www:

[Dokumentacja integracji płatności www Przelewy24](https://developers.przelewy24.pl/index.php?pl)


Aby móc dokonać integracji z systemem Przelewy24 niezbędne są dane sprzedawcy. Jeżeli ich nie posiadasz, możesz je uzyskać kontaktując się z działem handlowym:
- Tel: +48 61 642 93 45
- Email: <oferty@przelewy24.pl>


## 1. Opis systemu

Biblioteki mobilne Przelewy24 to natywne biblioteki na platformę Andoid i iOS. Umożliwiające wykonanie płatności wewnątrz aplikacji mobilnej, bez konieczności przełączania użytkownika między aplikacją, a przeglądarką lub inną aplikacją. Cały proces płatności odbywa się w jednym oknie aplikacji. W bibliotekach dostępne są różne metody płatności: przelewy bankowe, karty płatnicze, BLIK, wirtualne portfele (np. PayPal, SkyCash) i inne.

### 1.1 Jak wygląda przebieg transakcji przy użyciu biblioteki mobilnej?

Po wywołaniu płatności na ekranie smartfonu pojawia się okno płatności, zawierające kontrolkę WebView z załadowanym serwisem transakcyjnym przelewy24.pl. Po wybraniu metody płatności w tym samym oknie ładuje się strona wybranego banku/metody. Użytkownik loguje się do swojego konta, albo podaje dane potrzebne do płatności (np. dane karty, kod BLIK). Następnie użytkownik zatwierdza płatność. Okno biblioteki zmayka się ze statusem "płatność zakończona" albo z kodem błędu (np. jeżeli wylogowano się z banku bez potwierdzenia płatności, albo podany błędny kod BLIK).

![](img/diagram1.png)


### 1.2 Inicjowanie płatności

Do zainicjowania płatności konieczne jest zarejestrowanie transakcji za pomocą osobnego zapytania z serwera partnera. Opis rejestracji transakcji znajduje się w dokumentacji płatności www [link](https://developers.przelewy24.pl/index.php?pl#tag/Obsluga-transakcji-API/paths/~1api~1v1~1transaction~1register/post).

Jeżeli transakcja ma być zrealizowana w bibliotece mobilnej, przy rejestracji należy dodać parametry: `mobileLib=1` oraz `sdkVersion=X`, w którym należy podać numer wersji biblioteki mobilnej. Wartość tego parametru można pobrać bezpośrednio z biblioteki z klasy `P24SdkVersion` (iOS - `P24SdkVersion.value`, Android - `P24SdkVersion.value()`).

W wyniku rejestracji transakcji otrzymujemy TOKEN. Aby zainicjować transakcję w bibliotece mobilnej wystarczy przekazać ten token do metody `Transfer`.

**UWAGA!**

 > Rejestrując transakcję, która będzie wykonana w bibliotece mobilnej należy pamiętać o dodatkowych parametrach:
- `channel` – jeżeli nie będzie ustawiony, to domyślnie w bibliotece pojawią się formy płatności „przelew tradycyjny” i „użyj przedpłatę”, które są niepotrzebne przy płatności mobilnej. Aby wyłączyć te opcje należy ustawić w tym parametrze flagi nie uwzględniające tych form (np. wartość 3 – przelewy i karty)
- `method` – jeżeli w bibliotece dla danej transakcji ma być ustawiona domyślnie dana metoda płatności, należy ustawić ją w tym parametrze przy rejestracji
- `urlStatus` - adres, który zostanie wykorzystany do weryfikacji transakcji przez serwer partnera po zakończeniu procesu płatności w bibliotece mobilnej


### 1.3 Weryfikacja poprawności transakcji

Po dokonaniu wpłaty biblioteka kończy pracę i wraca do aplikacji. Nie czeka na zaksięgowanie jej w systemie Przelewy24 – zwraca do aplikacji status `paymentFinished` albo `paymentError` jeżeli coś poszło nie tak.
W momencie zaksięgowania wpłaty system Przelewy24 wysyła asynchronicznie powiadomienie o transakcji na adres `urlStatus` podany przez partnera w konfiguracji. Serwer partnera po odebraniu powiadomienia musi wysłać do Przelewy24 żądanie weryfikacji transakcji. W tym momencie serwer sprzedawcy posiada informację o zaksięgowaniu wpłaty. Aplikacja powinna wtedy odpytać swój serwer o status transakcji.

Parametr `urlStatus` należy ustawić w panelu transakcyjnym (w tym celu należy przesłać adres skryptu na [serwis@przelewy24.pl](serwis@przelewy24.pl) z adresu e-mail, na który jest założone konto).

## 2. Definicje

- **Sprzedawca** - Instytucja lub osoba prywatna korzystająca z usług serwisu PRZELEWY24.

- **Identyfikator sesji** - Unikalny identyfikator służący do weryfikacji danych pojedynczej
transakcji. Identyfikator ten pobierany jest od sprzedawcy.

- **CRC** - Losowy ciąg znaków służący do generowania sumy kontrolnej przesyłanych
parametrów, do pobrania z panelu Przelewy24.
