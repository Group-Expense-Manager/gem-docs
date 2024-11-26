# Diagrams

### System C4 Diagram
```mermaid

C4Component


Boundary(Person, "","") {
    Person(User, "Użytkownik")

}

Boundary(system_boundary, "System","") {
    
  Component(MobileApp, "Aplikacja mobilna")
  Boundary(cluster_boundary, "Klaster", "") {
    Boundary("public_boundary","Sieć publiczna","") {
        Component(ApiGateway, "Api Gateway")
    }
    Boundary("private_boundary","Sieć prywatna","") {
        Component(ServiceA, "Serwis A")
        Component(ServiceB, "Serwis B")
        Component(ServiceC, "Serwis C")
    }


  }

Boundary(db_boundary, "Bazy danych", "") {
    ComponentDb(DBA, "Baza danych A")
    ComponentDb(DBB, "Baza danych B")
    ComponentDb(DBC, "Baza danych C")
  }

}



Rel(User, MobileApp, "Używa")
UpdateRelStyle(User, MobileApp, $offsetY="-10", $offsetX="-17")
Rel(MobileApp, ApiGateway, "Wysyła żądania")
UpdateRelStyle(MobileApp, ApiGateway, $offsetY="-50", $offsetX="-65")
Rel(ApiGateway, ServiceA, "Przekazuje żądania")
UpdateRelStyle(ApiGateway, ServiceA, $offsetY="-10", $offsetX="-50")
Rel(ApiGateway, ServiceB, "Przekazuje żądania")
UpdateRelStyle(ApiGateway, ServiceB, $offsetY="10", $offsetX="-50")
Rel(ApiGateway, ServiceC, "Przekazuje żądania")
UpdateRelStyle(ApiGateway, ServiceC, $offsetY="20", $offsetX="-80")

Rel(ServiceA,ServiceB, "Wysyła żądania")
UpdateRelStyle(ServiceA, ServiceB, $offsetY="-15", $offsetX="-42")
Rel(ServiceB,ServiceC, "Wysyła żądania")
UpdateRelStyle(ServiceB, ServiceC, $offsetY="-15", $offsetX="-42")


Rel(ServiceA,DBA, "Odczytuje i zapisuje dane")
UpdateRelStyle(ServiceA, DBA, $offsetY="-15", $offsetX="-70")
Rel(ServiceB,DBB, "Odczytuje i zapisuje dane")
UpdateRelStyle(ServiceB, DBB, $offsetY="-15", $offsetX="-70")
Rel(ServiceC,DBC, "Odczytuje i zapisuje dane")
UpdateRelStyle(ServiceC, DBC, $offsetY="-15", $offsetX="-70")

```

### System C4 Diagram v2
```mermaid
C4Component

Person(User, "Użytkownik")

Boundary(system_boundary, "System","") {
      Boundary(user_boundary, "Urządzenie Mobilne", "") {

  Component(MobileApp, "Aplikacja Mobilna")
  }
  Boundary(cluster_boundary, "Klaster Kubernetesa", "") {

    Component(Microservices, "Mikroserwisy")
  }

Boundary(db_boundary, "MongoDB Cloud", "") {
    ComponentDb(DB, "Bazy Danych")

  }

}

Rel(User, MobileApp, "używa")
UpdateRelStyle(User, MobileApp, $offsetY="-45", $offsetX="-5")
Rel(MobileApp, Microservices, "Wysyła żądania")
UpdateRelStyle(MobileApp, Microservices, $offsetY="-13", $offsetX="-50")

Rel(Microservices,DB, "Dostęp do danych")
UpdateRelStyle(Microservices, DB, $offsetY="-15", $offsetX="-50")

UpdateLayoutConfig($c4ShapeInRow="1", $c4BoundaryInRow="3")
```

## Authenticator
```mermaid
sequenceDiagram

Client->>+ApiGateway: POST open/register (user data)
ApiGateway->>+Authenticator: POST open/register (user data)
Authenticator->>+EmailSender: POST /internal/verification (email + code)
EmailSender->>EmailClient: EMAIL
EmailSender->>-Authenticator: OK
Authenticator->>-ApiGateway: CREATED
ApiGateway->>-Client: CREATED

Client->>+ApiGateway: POST /open/login (credentials)
ApiGateway->>+Authenticator: POST /open/login (credentials)
Authenticator->>-ApiGateway: OK (token)
ApiGateway->>-Client: OK (token)

Client->>+ApiGateway: POST /open/verify (email address, code)
ApiGateway->>+Authenticator: POST /open/verify (email address, code)
Authenticator->>+ UserDetailsManager: POST /internal/user-details (id + username)
UserDetailsManager->>+AttachmentStore: POST /internal/users/{userId}/generate
AttachmentStore->>-UserDetailsManager: CREATED (attachmentId)
UserDetailsManager->>-Authenticator: CREATED
Authenticator->>-ApiGateway: OK (token)
ApiGateway->>-Client: OK (token)

Client->>+ApiGateway: POST /open/send-verification-email (email address)
ApiGateway->>+Authenticator: POST open/send-verification-email (email address)
Authenticator->>+EmailSender: POST /internal/verification (email address + code)
EmailSender->>EmailClient: EMAIL
EmailSender->>-Authenticator: OK
Authenticator->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: POST /open/recover-password (email address)
ApiGateway->>+Authenticator: POST /open/recover-password (email address)
Authenticator->>+EmailSender: POST /internal/recover (email address + link)
EmailSender->>+UserDetailsManager: POST /internal/user-details/username/{id}
UserDetailsManager->>-EmailSender: OK (username)
EmailSender->>EmailClient: EMAIL
EmailSender->>-Authenticator: OK
Authenticator->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: POST /open/reset-password?email={email}&code={code} 
ApiGateway->>+Authenticator: POST /open/reset-password?email={email}&code={code}
Authenticator->>+EmailSender: POST /internal/password (email address + code)
EmailSender->>+UserDetailsManager: POST /internal/user-details/username/{id}
UserDetailsManager->>-EmailSender: OK (username)
EmailSender->>EmailClient: EMAIL
EmailSender->>-Authenticator: OK
Authenticator->>-ApiGateway: OK
ApiGateway->>-Client: OK


Client->>+ApiGateway: POST external/change-password (token + old&new password)
ApiGateway->>+Authenticator: POST external/change-password (id + old&new password)
Authenticator->>-ApiGateway: OK
ApiGateway->>-Client: OK
```
## User Details Manager

```mermaid
sequenceDiagram

Client->>+ApiGateway: PUT /external/user-details (token + details)
ApiGateway->>+UserDetailsManager: PUT /external/user-details (id + details)
UserDetailsManager->>-ApiGateway: OK 
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/user-details (token)
ApiGateway->>+UserDetailsManager: GET /external/user-details (id)
UserDetailsManager->>-ApiGateway: OK (user details)
ApiGateway->>-Client: OK (user details)

Client->>+ApiGateway: GET /external/user-details/groups/{groupId} (token)
ApiGateway->>+UserDetailsManager: GET /external/user-details/groups/{groupId} (id)
UserDetailsManager->>+ GroupManager: GET /internal/groups/users/{userId}
GroupManager->>- UserDetailsManager: OK (group ids)
UserDetailsManager->>+ GroupManager: /internal/members/{groupId}
GroupManager->>- UserDetailsManager: OK (member ids)
UserDetailsManager->>-ApiGateway: OK (group members details)
ApiGateway->>-Client: OK (group members details)

Client->>+ApiGateway: GET /external/user-details/groups/{groupId}/members/{memberId} (token)
ApiGateway->>+UserDetailsManager: GET /external/user-details/groups/{groupId}/members/{memberId} (id)
UserDetailsManager->>+ GroupManager: GET /internal/groups/users/{userId}
GroupManager->>- UserDetailsManager: OK (group ids)
UserDetailsManager->>+ GroupManager: /internal/members/{groupId}
GroupManager->>- UserDetailsManager: OK (member ids)
UserDetailsManager->>-ApiGateway: OK (group member details)
ApiGateway->>-Client: OK (group member details)
```

## Group Manager

```mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/groups (token + group name, currencies)
ApiGateway->>+GroupManager:  POST /external/groups (id + group name, currencies)
GroupManager->>+CurrencyManager: GET /internal/currencies
CurrencyManager->>-GroupManager: OK (available currency)
GroupManager->>+AttachmentStore: POST /internal/groups/{groupId}/users/{userId}/generate
AttachmentStore->>-GroupManager: OK (attachmentId)
GroupManager->>-ApiGateway: CREATED (group data)
ApiGateway->>-Client: CREATED (group data)

Client->>+ApiGateway: DELETE /external/groups/{groupId} (token)
ApiGateway->>+GroupManager:  DELETE /external/groups/{groupId} (id)
GroupManager->>+FinanceAdapter: GET /internal/balances/groups/{groupId} (id)
FinanceAdapter->>-GroupManager: OK (balances of users)
GroupManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: PUT /external/groups/{groupId} (token + group name, currencies)
ApiGateway->>+GroupManager:  PUT /external/groups/{groupId} (id + group name, currencies)
GroupManager->>+CurrencyManager: GET /internal/currencies
CurrencyManager->>-GroupManager: OK (available currency)
GroupManager->>-ApiGateway: OK (group data)
ApiGateway->>-Client: OK (group data)

Client->>+ApiGateway: GET /external/groups/{groupId} (token)
ApiGateway->>+GroupManager:  GET /external/groups/{groupId} (id)
GroupManager->>-ApiGateway: OK (group data)
ApiGateway->>-Client: OK (group data)

Client->>+ApiGateway: GET /external/groups (token)
ApiGateway->>+GroupManager:  GET /external/groups (id)
GroupManager->>-ApiGateway: OK (groups data)
ApiGateway->>-Client: OK (groups data)

Client->>+ApiGateway: POST /external/groups/join/{joinCode} (token)
ApiGateway->>+GroupManager:  POST /external/groups/join{joinCode}  (id)
GroupManager->>-ApiGateway: OK (group data)
ApiGateway->>-Client: OK (group data)
```

## Expense Manager

```mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/expenses/{groupId} (token + expense data)
ApiGateway->>+ExpenseManager: POST /external/expenses/{groupId} (id + expense data)
ExpenseManager->>+ GroupManager: GET /internal/groups/users/{userId}
GroupManager->>- ExpenseManager: OK (group ids)
ExpenseManager->>+GroupManager: GET /internal/groups/{groupId}
GroupManager->>-ExpenseManager: OK (group data)
ExpenseManager->>+CurrencyManager: GET /internal/currencies
CurrencyManager->>-ExpenseManager: OK (available currency)
ExpenseManager->>+CurrencyManager: GET /internal/currencies/from/{baseCurrency}/to/{targetCurrency}/?date={date}
CurrencyManager->>-ExpenseManager: OK (exchange rate)
ExpenseManager->>-ApiGateway: CREATED
ApiGateway->>-Client: CREATED


Client->>+ApiGateway: PUT /external/expenses/{groupId}/{expense_id} (token + updated expense data)
ApiGateway->>+ExpenseManager: PUT /external/expenses/{groupId}/{expense_id} (id + updated expense data)
ExpenseManager->>+ GroupManager: GET /internal/groups/users/{userId}
GroupManager->>- ExpenseManager: OK (group ids)
ExpenseManager->>+GroupManager: GET /internal/groups/{groupId}
GroupManager->>-ExpenseManager: OK (group data)
ExpenseManager->>+CurrencyManager: GET /internal/currencies
CurrencyManager->>-ExpenseManager: OK (available currency)
ExpenseManager->>+CurrencyManager: GET /internal/currencies/from/{baseCurrency}/to/{targetCurrency}/?date={date}
CurrencyManager->>-ExpenseManager: OK (exchange rate)
ExpenseManager->>+FinanceAdapter: POST /internal/generate/groups/{groupId}
FinanceAdapter->>-ExpenseManager: OK
ExpenseManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: DELETE /external/expenses/{groupId}/{expense_id} (token)
ApiGateway->>+ExpenseManager: DELETE /external/expenses/{groupId}/{expense_id} (id)
ExpenseManager->>+ GroupManager: GET /internal/groups/users/{userId}
GroupManager->>- ExpenseManager: OK (group ids)
ExpenseManager->>+FinanceAdapter: POST /internal/generate/groups/{groupId}
FinanceAdapter->>-ExpenseManager: OK
ExpenseManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/expenses/{expense_id}/groups/{groupId} (token)
ApiGateway->>+ExpenseManager: GET /external/expenses/{expense_id}/groups/{groupId} (id)
ExpenseManager->>+ GroupManager: GET /internal/groups/users/{userId}
GroupManager->>- ExpenseManager: OK (group ids)
ExpenseManager->>-ApiGateway: OK (expense data)
ApiGateway->>-Client: OK (expense data)

Client->>+ApiGateway: POST /external/expenses/decide (token + decision)
ApiGateway->>+ExpenseManager: POST /external/expenses/decide (id + decision)
ExpenseManager->>+ GroupManager: GET /internal/groups/users/{userId}
GroupManager->>- ExpenseManager: OK (group ids)
ExpenseManager->>+FinanceAdapter: POST /internal/generate/groups/{groupId}
FinanceAdapter->>-ExpenseManager: OK
ExpenseManager->>-ApiGateway: OK
ApiGateway->>-Client: OK
```

## Payment Manager

```mermaid
sequenceDiagram

    Client->>+ApiGateway: POST /external/payments/{groupId} (token + payment data)
    ApiGateway->>+PaymentManager: POST /external/payments/{groupId} (id + payment data)
    PaymentManager->>+ GroupManager: GET /internal/groups/users/{userId}
    GroupManager->>- PaymentManager: OK (group ids)
    PaymentManager->>+GroupManager: GET /internal/groups/{groupId}
    GroupManager->>-PaymentManager: OK (group data)
    PaymentManager->>+CurrencyManager: GET /internal/currencies
    CurrencyManager->>-PaymentManager: OK (available currency)
    PaymentManager->>+CurrencyManager: GET /internal/currencies/from/{baseCurrency}/to/{targetCurrency}/?date={date}
    CurrencyManager->>-PaymentManager: OK (exchange rate)
    PaymentManager->>-ApiGateway: CREATED
    ApiGateway->>-Client: CREATED


    Client->>+ApiGateway: PUT /external/payments/{groupId}/{payment_id} (token + updated payment data)
    ApiGateway->>+PaymentManager: PUT /external/payments/{groupId}/{payment_id} (id + updated payment data)
    PaymentManager->>+ GroupManager: GET /internal/groups/users/{userId}
    GroupManager->>- PaymentManager: OK (group ids)
    PaymentManager->>+GroupManager: GET /internal/groups/{groupId}
    GroupManager->>-PaymentManager: OK (group data)
    PaymentManager->>+CurrencyManager: GET /internal/currencies
    CurrencyManager->>-PaymentManager: OK (available currency)
    PaymentManager->>+CurrencyManager: GET /internal/currencies/from/{baseCurrency}/to/{targetCurrency}/?date={date}
    CurrencyManager->>-PaymentManager: OK (exchange rate)
    PaymentManager->>+FinanceAdapter: POST /internal/generate/groups/{groupId}
    FinanceAdapter->>-PaymentManager: OK
    PaymentManager->>-ApiGateway: OK
    ApiGateway->>-Client: OK

    Client->>+ApiGateway: DELETE /external/payments/{groupId}/{payment_id} (token)
    ApiGateway->>+PaymentManager: DELETE /external/payments/{groupId}/{payment_id} (id)
    PaymentManager->>+ GroupManager: GET /internal/groups/users/{userId}
    GroupManager->>- PaymentManager: OK (group ids)
    PaymentManager->>+FinanceAdapter: POST /internal/generate/groups/{groupId}
    FinanceAdapter->>-PaymentManager: OK
    PaymentManager->>-ApiGateway: OK
    ApiGateway->>-Client: OK

    Client->>+ApiGateway: GET /external/payments/{payment_id}/groups/{groupId} (token)
    ApiGateway->>+PaymentManager: GET /external/payments/{payment_id}/groups/{groupId} (id)
    PaymentManager->>+ GroupManager: GET /internal/groups/users/{userId}
    GroupManager->>- PaymentManager: OK (group ids)
    PaymentManager->>-ApiGateway: OK (payment data)
    ApiGateway->>-Client: OK (payment data)

    Client->>+ApiGateway: POST /external/payments/decide (token + decision)
    ApiGateway->>+PaymentManager: POST /external/payments/decide (id + decision)
    PaymentManager->>+ GroupManager: GET /internal/groups/users/{userId}
    GroupManager->>- PaymentManager: OK (group ids)
    PaymentManager->>+FinanceAdapter: POST /internal/generate/groups/{groupId}
    FinanceAdapter->>-PaymentManager: OK
    PaymentManager->>-ApiGateway: OK
    ApiGateway->>-Client: OK
```


## Currency Manager

```mermaid
 sequenceDiagram

Client->>+ApiGateway: GET /external/currencies (token)
ApiGateway->>+CurrencyManager:  GET /external/currencies
CurrencyManager->>-ApiGateway: OK (available currencies)
ApiGateway->>-Client: OK (available currencies)

```

## Finance Adapter

```mermaid
sequenceDiagram
Client->>+ApiGateway: GET /external/activities/groups/{groupId} (token + filters)
ApiGateway->>+FinanceAdapter: GET /external/activities/groups/{groupId}  (id + filters)
FinanceAdapter->>+GroupManager: GET /internal/groups/users/{userId}
GroupManager->>-FinanceAdapter: OK (group ids)
FinanceAdapter->>+ ExpenseManager: GET /internal/expenses/activities/groups/{groupId} (filters)
ExpenseManager->>- FinanceAdapter: OK (expense data)
FinanceAdapter->>+ PaymentManager: GET /internal/payments/activities/groups/{groupId} (filters)
PaymentManager->>- FinanceAdapter: OK (payment data)
FinanceAdapter->>-ApiGateway: OK (activities)
ApiGateway->>-Client: OK (activities)

Client->>+ApiGateway: GET /external/balances/groups/{groupId} (token)
ApiGateway->>+FinanceAdapter: GET /external/balances/groups/{groupId} (id)
FinanceAdapter->>+GroupManager: GET /internal/groups/users/{userId}
GroupManager->>-FinanceAdapter: OK (group ids)
FinanceAdapter->>+GroupManager: GET /internal/groups/{groupId}
GroupManager->>-FinanceAdapter: OK (group data)
FinanceAdapter->>+ ExpenseManager: GET /internal/expenses/accepted/groups/{groupId}?currency={currency} (filters)
ExpenseManager->>- FinanceAdapter: OK (expense data)
FinanceAdapter->>+ PaymentManager: GET /internal/payments/accepted/groups/{groupId}?currency={currency}
PaymentManager->>- FinanceAdapter: OK (payment data)
FinanceAdapter->>-ApiGateway: OK (balances)
ApiGateway->>-Client: OK (balances)

Client->>+ApiGateway: GET /external/settlements/groups/{groupId} (token)
ApiGateway->>+FinanceAdapter: GET /external/settlements/groups/{groupId} (id)
FinanceAdapter->>+GroupManager: GET /internal/groups/users/{userId}
GroupManager->>-FinanceAdapter: OK (group ids)
FinanceAdapter->>+GroupManager: GET /internal/groups/{groupId}
GroupManager->>-FinanceAdapter: OK (group data)
FinanceAdapter->>-ApiGateway: OK (balances)
ApiGateway->>-Client: OK (balances)
```


## Report Creator

```mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/generate/groups/{groupId} (token)
ApiGateway->>+ReportCreator: POST /external/generate/{groupId} (id)
ReportCreator->>+GroupManager: GET /internal/groups/users/{userId}
GroupManager->>-ReportCreator: OK (group ids)
ReportCreator->>+GroupManager: GET /internal/groups/{groupId}
GroupManager->>-ReportCreator: OK (group data)
ReportCreator->>+UserDetailsManager: GET /internal/user-details/groups/{groupId}
UserDetailsManager->>-ReportCreator: OK (group members details)
ReportCreator->>+ FinanceAdapter: GET /internal/activities/groups (id)
FinanceAdapter->>- ReportCreator: OK (activities)
ReportCreator->>+ FinanceAdapter: GET /internal/balances/groups (id)
FinanceAdapter->>- ReportCreator: OK (balances)
ReportCreator->>+ FinanceAdapter: GET /internal/settlements/groups (id)
FinanceAdapter->>- ReportCreator: OK (settlements)
ReportCreator->>+AttachmentStore:  POST /internal/groups/{groupId}?userId={userId} ( attachment)
AttachmentStore->>-ReportCreator: CREATED (attachmentId)
ReportCreator->>+EmailSender: /internal/report (report data)
EmailSender->>-ReportCreator: OK
ReportCreator->>-ApiGateway: CREATED
ApiGateway->>-Client: CREATED

```

## Attachment Store

```mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/users (token + attachment)
ApiGateway->>+AttachmentStore: POST /external/users (id + attachment)
AttachmentStore->>-ApiGateway: CREATED (attachmentId)
ApiGateway->>-Client: CREATED (attachmentId)

Client->>+ApiGateway: POST /external/groups/{groupId} (token + attachment)
ApiGateway->>+AttachmentStore: POST /external/groups/{groupId} (id + attachment)
AttachmentStore->>+GroupManager: GET /internal/groups/users/{userId}
GroupManager->>-AttachmentStore: OK (group ids)
AttachmentStore->>-ApiGateway: CREATED (attachmentId)
ApiGateway->>-Client: CREATED (attachmentId)

Client->>+ApiGateway: GET /external/users/{userId}/attachments/{attachmentId} (token)
ApiGateway->>+AttachmentStore: GET /external/users/{userId}/attachments/{attachmentId}  (id)
AttachmentStore->>-ApiGateway: OK (attachment)
ApiGateway->>-Client: OK (attachment)

Client->>+ApiGateway: GET /external/groups/{groupId}/attachments/{attachmentId} (token)
ApiGateway->>+AttachmentStore: GET /external/groups/{groupId}/attachments/{attachmentId}  (id)
AttachmentStore->>+GroupManager: GET /internal/groups/users/{userId}
GroupManager->>-AttachmentStore: OK (group ids)
AttachmentStore->>-ApiGateway: OK (attachment)
ApiGateway->>-Client: OK (attachment)

Client->>+ApiGateway: PUT /external/users/{userId}/attachments/{attachmentId} (token + updated attachment)
ApiGateway->>+AttachmentStore: PUT /external/users/{userId}/attachments/{attachmentId} (id + updated attachment)
AttachmentStore->>-ApiGateway: OK (attachmentId)
ApiGateway->>-Client: OK (attachmentId)

Client->>+ApiGateway: PUT /external/groups/{groupId}/attachment/{attachmentId} (token + updated attachment)
ApiGateway->>+AttachmentStore: PUT /external/groups/{groupId}/attachment/{attachmentId} (id + updated attachment)
AttachmentStore->>+GroupManager: GET /internal/groups/users/{userId}
GroupManager->>-AttachmentStore: OK (group ids)
AttachmentStore->>-ApiGateway: OK (attachmentId)
ApiGateway->>-Client: OK (attachmentId)

```