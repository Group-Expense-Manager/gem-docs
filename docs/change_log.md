# Change Log

## Styczeń 2024
### 08.01.2024 - 15.01.2024
* Prace koncepcyjne - zakres funkcjonalności, ustelenie stosu technologicznego
* Zatwierdzenie pomysłu przez opiekuna

## Luty 2024
### 05.02.2024 - 11.02.2024
* Planowanie architektury aplikacji - podział na serwisy, schemat bazy danych
* Przygotowanie środowiska pracy - GitHub, Jira, Miro, Figma
* Stworzenie wiremocków
### 12.02.2024 - 18.02.2024
* Przygotowanie środowiska pracy - Mongo Atlas
### 19.02.2024 - 25.02.2024
* Przetestowanie oferty Microsoft Azure jako dostawcy usług chmurowych
* Początek prac implementacyjnych - service-template, front
### 26.02.2024 - 03.03.2024
* Przetestowanie oferty Microsoft Azure jako dostawcy usług chmurowych
* Prace nad pipeline'em

## Marzec 2024
### 04.03.2024 - 10.03.2024
* Przetestowanie oferty Amazon AWS jako dostawcy usług chmurowych
### 11.03.2024 - 17.03.2024
* Implementacja rejestracji w serwisie autentykacyjnym
* Przetestowanie oferty Amazon AWS jako dostawcy usług chmurowych
* Stworzenie koncepcyjnych makiet dotyczących autentykacji
* Spotkanie z opiekunem pracy w celu ustalenia szczegołów współpracy
* Stworzenie organizacji na postmanie
### 18.03.2024 - 24.03.2024
* Stworzenie dokumentacji na github pages
* Pisanie pierwszego rozdziału pracy inżynierskiej
* Ulepszenia schematu serwisu o testy integracyjnie i jednostoke
* Dostosowanie platformy Jira
### 25.03.2024 - 31.03.2024
* Początek implementacji widoku autentykacji na froncie
* Ulepszenia schematu serwisu o testy integracyjnie i jednostowe
* Pisanie pierwszego rozdziału pracy inżynierskiej
* Implementacja serwisu autentykacyjnego

## Kwiecień 2024
### 01.04.2024 - 07.04.2024
* Implementacja serwisu autentykacyjnego
* Tworzenie diagramów uml dla każdego serwisu
* Stworzenie biblioteki do wszystkich serwisów
* Dodanie analizatora kodu do schematu serwisu
### 08.04.2024 - 14.04.2024
* Tworzenie diagramów uml dla każdego serwisu
* Konsultacje z opiekunem z przedstawieniem pierwszego rozdziału pracy inżynierskiej
* Konsultacje z pracowni projektowej 
### 15.04.2024 - 21.04.2024
* Aktualizacja wireframów
* Tworzenie tasków w jirze
* Diagram komunikacji między-mikroserwisowej
### 22.04.2024 - 28.04.2024
* Tworzenie kontraktu dla każdego mikroserwisu
### 29.04.2024 - 05.05.2024
* Tworzenie kontraktu dla każdego mikroserwisu
* Implementacja gem-lib i service-template (ostatnie poprawki)
* Implementacja authenticatora, email-sendera, attachment-store'a
* Testowanie Kubernetesa jako orchestratora

## Maj 2024
### 06.05.2024 - 12.05.2024
* Tworzenie Api Gateway'a
* Tworzenie mocków frontendu - komponenty + pierwsze widoki
* Poprawki do authenticatora, email-sendera
### 13.05.2024 - 19.05.2024
* Tworzenie Api Gateway'a
* Tworzenie Expense Managera
* Tworzenie Group Managera
* Tworzenie mocków frontendu
### 20.05.2024 - 26.05.2024
* Tworzenie Expense Managera
* Tworzenie Group Managera
* Tworzenie mockow frontendu
* Tworzenie Currency Managera
### 27.05.2024 - 02.06.2024
* Tworzenie mocków

## Czerwiec 2024
### 03.06.2024 - 09.06.2024
* Tworzenie mocków
### 10.06.2024 - 16.06.2024
* Tworzenie mocków
* Tworzenie frontendu - autentykacja
* Drobne poprawki serwisu do zarządzania grupami
* Dodanie funkcjonalości do serwisu zarządzania wydatkami
### 17.06.2024 - 23.06.2024
* Tworzenie frontendu - grupy, wydatki
* Postawienie backendu na klastrze k8s
* Konsultacje z opiekunem, przedstawienie stopnia zaawansowania pracy
### 24.06.2024 - 30.06.2024
* Implementacja UserDetailsManager, GroupManagera, CurrencyManagera
## Lipiec 2024
### 01.07.2024 - 07.07.2024
* Implementacja UserDetailsManagera, Authentykatora, ExpenseManagera
### 08.07.2024 - 14.07.2024
* Implementacja ExpenseManagera, UserDetailsManagera
### 15.07.2024 - 21.07.2024
* Dodanie funkcjonalności defaultowe obrazka dla grupy w AttachmentStore
* Implementacja funkcjonalności wysyłania maili do użytkowników w EmailSender
* Implementacja funcjonalności edycji załaczników w AttachmentStore
### 22.07.2024 - 28.07.2024
* Przygotowanie serwisu zarządzającego strukturą Kubernetes'a do łatwego developmentu (Helm Chart)
* Mniejsze poprawki do serwisów Authenticator, UserDetailsManager, ExpenseManager, EmailSender
### 29.07.2024 - 04.08.2024
* Dodanie funkcjonalności zbierania i analizowania logów w serwisach (Grafana, Loki)
* Dodanie integracji z narodowym bankiem polskim w CurrencyManager
* Rozpoczęcie prac nad PaymentManagerem

## Sierpień 2024
### 05.08.2024 - 11.08.2024
* Dodanie ansychroniczny proces automatycznego zbierania kursów walut z NBP
* Dodanie funkcjonalności edycji wydatków w ExpenseManager
* Dodanie do biblioteki autokonfigurowalnego Execturora jobów asynchronicznych
### 12.08.2024 - 18.08.2024
* Poprawki w Authenticatorze
### 19.08.2024 - 25.08.2024
* Dodanie funkcjonalności tworzenia / edycji / usuwania płatności w PaymentManager
* Dodanie funkcjonalności automatycznego zbierania logów w Serwisach na warstwie kontrolerów i klientów
* Dodanie funkcjonalności analizowanie logow na podstawie trace-Id
### 26.08.2024 - 01.09.2024
* Drobne poprawki w Authenticatorze, EmailSenderze
* Stworzenie komponentów frontedoowych

## Wrzesień 2024
### 02.09.2024 - 08.09.2024
* Stworzenie komponentów frontedoowych
* Stworzenie widoków dla zarządzania profilem i ustawieniami konta
### 09.09.2024 - 15.09.2024
* Stworzenie komponentów frontedoowych
* Poprawki w GroupManagerze, ExepenseManagerze, PaymentManagera
### 16.09.2024 - 22.09.2024
* Poprawki w ExpenseManagerze
* Stworzenie widoków dla zarządzania grupami
* Refactor kodu frontedowego
### 23.09.2024 - 29.09.2024
* Poprawki w ExpenseManagerze
* Stworzenie komponentów frontedoowych
* Stworzenie widoków dla tworzenia grupy
* Stworzenie widoków dla tworzenia wydatków
### 30.09.2024 - 06.10.2024
* Poprawki w komponentach frontedowych
* Stworzenie widoku szczegółów wydatków
* Stworzenie widoku szczegółów płatności
* Stworzenie widoku tworzenia płatności
* Poprawki w AttachmentStore, GroupManager

## Październik 2024
### 07.10.2024 - 13.10.2024
* Spotkanie z opiekunem w celu przedstawienia postępu i planu dalszej pracy
* Stworzenie komponentów frontedoowych
* Stworzenie widoków edycji płatności i wydatków
* Poprawki w AttachmentStore
* Przygotowanie aplikacji do buildu na Androida