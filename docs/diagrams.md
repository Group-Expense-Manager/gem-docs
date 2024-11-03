# Diagrams

### System C4 Diagram
``` mermaid

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
``` mermaid
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
``` mermaid
sequenceDiagram

Client->>+ApiGateway: POST open/register (user data)
ApiGateway->>+Authenticator: POST open/register (user data)
Authenticator->>-ApiGateway: CREATED
ApiGateway->>-Client: CREATED

Client->>+ApiGateway: POST /open/login (credentials)
ApiGateway->>+Authenticator: POST /open/login (credentials)
Authenticator->>-ApiGateway: OK (token)
ApiGateway->>-Client: OK (token)

Client->>+ApiGateway: POST /open/verify (email address, code)
ApiGateway->>+Authenticator: POST /open/verify (email address, code)
Authenticator->>+ UserDetailsManager: POST /internal/user-details (id + email as username)
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
EmailSender->>EmailClient: EMAIL
EmailSender->>-Authenticator: OK
Authenticator->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: POST /open/send-password?email=val&code=val 
ApiGateway->>+Authenticator: POST /open/send-password?email=val&code=val
Authenticator->>+EmailSender: POST /internal/password (email address + code)
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

``` mermaid
sequenceDiagram

Client->>+ApiGateway: PUT /external/user-details (token + username,firstname,lastname,pfp)
ApiGateway->>+UserDetailsManager: PUT /external/user-details (id + username,firstname,lastname,pfp)
UserDetailsManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/user-details (token)
ApiGateway->>+UserDetailsManager: GET /external/user-details (id)
UserDetailsManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/user-details/{group_id} (token)
ApiGateway->>+UserDetailsManager: GET /external/user-details/{group_id} (id)
UserDetailsManager->>+ GroupManager: GET /internal/members/{group_id}
GroupManager->>- UserDetailsManager: OK (group members ids)
UserDetailsManager->>+ GroupManager: /internal/groups/{group_id}/ids
GroupManager->>- UserDetailsManager: OK
UserDetailsManager->>-ApiGateway: OK
ApiGateway->>-Client: OK
```

## Group Manager

``` mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/groups (token + group name,currency,options,colour)
ApiGateway->>+GroupManager:  POST /external/groups (id + group name,currency,options,colour)
GroupManager->>+CurrencyManager: GET /internal/currencies
CurrencyManager->>-GroupManager: OK (available currency)
GroupManager->>-ApiGateway: CREATED
ApiGateway->>-Client: CREATED

Client->>+ApiGateway: DELETE /external/groups/{group_id} (token)
ApiGateway->>+GroupManager:  POST /external/groups/{group_id} (id)
GroupManager->>+FinanceAdapter: GET /internal/balances/groups/{group_id} (id)
FinanceAdapter->>-GroupManager: OK (balances of users)
GroupManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: PUT /external/groups/{group_id} (token + group name,colour)
ApiGateway->>+GroupManager:  PUT /external/groups/{group_id} (id + group name,colour)
GroupManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/groups/{group_id} (token)
ApiGateway->>+GroupManager:  GET /external/groups/{group_id} (id)
GroupManager->>-ApiGateway: OK (group data)
ApiGateway->>-Client: OK (group data)

Client->>+ApiGateway: POST /external/groups/join (token + join code)
ApiGateway->>+GroupManager:  POST /external/groups/join (id + join code)
GroupManager->>-ApiGateway: OK (group data)
ApiGateway->>-Client: OK (group data)
```

## Expense Manager

``` mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/expenses/{group_id} (token + expense data)
ApiGateway->>+ExpenseManager: POST /external/expenses/{group_id} (id + expense data)
ExpenseManager->>+GroupManager: GET /internal/groups/{group_id}
GroupManager->>-ExpenseManager: OK (group data)
ExpenseManager->>+CurrencyManager: GET /internal/currencies
CurrencyManager->>-ExpenseManager: OK (available currency)
ExpenseManager->>+CurrencyManager: GET /internal/exchange-rate?from=val1&to=val2
CurrencyManager->>-ExpenseManager: OK (exchange rate)
ExpenseManager->>-ApiGateway: CREATED
ApiGateway->>-Client: CREATED


Client->>+ApiGateway: PUT /external/expenses/{group_id}/{expense_id} (token + updated expense data)
ApiGateway->>+ExpenseManager: PUT /external/expenses/{group_id}/{expense_id} (id + updated expense data)
ExpenseManager->>+GroupManager: GET /internal/groups/{group_id}
GroupManager->>-ExpenseManager: OK (group data)
ExpenseManager->>+CurrencyManager: GET /internal/currencies
CurrencyManager->>-ExpenseManager: OK (available currency)
ExpenseManager->>+CurrencyManager: GET /internal/exchange-rate?from=val1&to=val2
CurrencyManager->>-ExpenseManager: OK (exchange rate)
ExpenseManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: DELETE /external/expenses/{group_id}/{expense_id} (token)
ApiGateway->>+ExpenseManager: DELETE /external/expenses/{group_id}/{expense_id} (id)
ExpenseManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/expenses/{group_id} (token + filters)
ApiGateway->>+ExpenseManager: GET /external/expenses/{group_id} (id + filters)
ExpenseManager->>+GroupManager: GET /internal/members/{group_id}
GroupManager->>-ExpenseManager: OK (group members ids)
ExpenseManager->>-ApiGateway: OK + ids + names  + date + status
ApiGateway->>-Client: OK + ids + names  + date + status


Client->>+ApiGateway: GET /external/expenses/{group_id}/{expense_id} (token)
ApiGateway->>+ExpenseManager: GET /external/expenses/{expense_id} (id)
ExpenseManager->>-ApiGateway: OK + expense data
ApiGateway->>-Client: OK + expense data

Client->>+ApiGateway: POST /external/accept-expense/{expense_id} (token + resolve)
ApiGateway->>+ExpenseManager: POST /external/accept-expense/{expense_id} (id + resolve)
ExpenseManager->>-ApiGateway: OK
ApiGateway->>-Client: OK
```

## Payment Manager

``` mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/payments/{group_id} (token + payment data)
ApiGateway->>+PaymentManager: POST /external/payments/{group_id} (id + payment data)
PaymentManager->>+GroupManager: GET /internal/groups/{group_id}
GroupManager->>-PaymentManager: OK (group data)
PaymentManager->>+CurrencyManager: GET /internal/currencies
CurrencyManager->>-PaymentManager: OK (available currency)
PaymentManager->>+CurrencyManager: GET /internal/exchange-rate?from=val1&to=val2
CurrencyManager->>-PaymentManager: OK (exchange rate)
PaymentManager->>-ApiGateway: CREATED
ApiGateway->>-Client: CREATED

Client->>+ApiGateway: PUT /external/payments/{group_id}/{payment_id} (token + updated payment data)
ApiGateway->>+PaymentManager: PUT /external/payments/{group_id}/{payment_id} (id + updated payment data)
PaymentManager->>+GroupManager: GET /internal/groups/{group_id}
GroupManager->>-PaymentManager: OK (group data)
PaymentManager->>+CurrencyManager: GET /internal/currencies
CurrencyManager->>-PaymentManager: OK (available currency)
PaymentManager->>+CurrencyManager: GET /internal/exchange-rate?from=val1&to=val2
CurrencyManager->>-PaymentManager: OK (exchange rate)
PaymentManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: DELETE /external/payments/{group_id}/{payment_id} (token)
ApiGateway->>+PaymentManager: DELETE /external/payments/{group_id}/{payment_id} (id)
PaymentManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/payments/{group_id} (token + filters)
ApiGateway->>+PaymentManager: GET /external/payments/{group_id} (id + filters)
PaymentManager->>+GroupManager: GET /internal/members/{group_id}
GroupManager->>-PaymentManager: OK (group members ids)
PaymentManager->>-ApiGateway: OK + ids + names + date + status
ApiGateway->>-Client: OK + ids + names + date + status


Client->>+ApiGateway: GET /external/payments/{payment_id} (token + filters)
ApiGateway->>+PaymentManager: GET /external/payments/{payment_id} (id + filters)
PaymentManager->>-ApiGateway: OK + payment data
ApiGateway->>-Client: OK + payment data

Client->>+ApiGateway: POST /external/accept-payment/{payment_id} (token + resolve)
ApiGateway->>+PaymentManager: POST /external/accept-payment/{payment_id} (id + resolve)
PaymentManager->>-ApiGateway: OK
ApiGateway->>-Client: OK
```


## Currency Manager

``` mermaid
 sequenceDiagram

Client->>+ApiGateway: GET /external/currencies (token)
ApiGateway->>+CurrencyManager:  GET /external/currencies
CurrencyManager->>-ApiGateway: OK (available currencies)
ApiGateway->>-Client: OK (available currencies)

```

## Finance Adapter

```mermaid
sequenceDiagram
Client->>+ApiGateway: GET /external/balances/groups?group_id={group_id} (token)
ApiGateway->>+FinanceAdapter: GET /external/balances/groups?group_id={group_id} (id)
FinanceAdapter->>+GroupManager: GET /internal/members/{group_id}
GroupManager->>-FinanceAdapter: OK (group members ids)
FinanceAdapter->>+ ExpenseManager: GET /internal/expenses?group_id={group_id} (id)
ExpenseManager->>- FinanceAdapter: OK (expense data)
FinanceAdapter->>+ PaymentManager: GET /internal/payments?group_id={group_id} (id)
PaymentManager->>- FinanceAdapter: OK (payment data)
FinanceAdapter->>-ApiGateway: OK (group balance + suggested alignment)
ApiGateway->>-Client: OK (group balance + suggested alignment)

Client->>+ApiGateway: GET /external/balances/?group_id={group_id} (token)
ApiGateway->>+FinanceAdapter: GET /external/balances/?group_id={group_id} (id)
FinanceAdapter->>+GroupManager: GET /internal/members/{group_id}
GroupManager->>-FinanceAdapter: OK (group members ids)
FinanceAdapter->>+ ExpenseManager: GET /internal/expenses?group_id={group_id} (id)
ExpenseManager->>- FinanceAdapter: OK (expense data)
FinanceAdapter->>+ PaymentManager: GET /internal/payments?group_id={group_id} (id)
PaymentManager->>- FinanceAdapter: OK (payment data)
FinanceAdapter->>-ApiGateway: OK (user balance)
ApiGateway->>-Client: OK (user balance)
    
```


## Report Creator

```mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/generate/{group_id} (token)
ApiGateway->>+ReportCreator: POST /external/generate/{group_id} (id)
ReportCreator->>+GroupManager: GET /internal/members/{group_id}
GroupManager->>-ReportCreator: OK
ReportCreator->>+UserDetailsManager: GET /internal/user-details/{group_id}
UserDetailsManager->>-ReportCreator: OK (group user details)
ReportCreator->>+ FinanceAdapter: GET /internal/report?group_id={group_id} (id)
FinanceAdapter->>- ReportCreator: OK (group balance & suggested alignments & finance data)
ReportCreator->>+AttachmentStore:  POST /internal/attachments/groups/{group_id}?user_id=val (pdf attachment)
AttachmentStore->>-ReportCreator: CREATED (attachment_id)
ReportCreator->>+AttachmentStore:  POST /internal/attachments/groups/{group_id}?user_id=val (csv attachment)
AttachmentStore->>-ReportCreator: CREATED (attachment_id)
ReportCreator->>-ApiGateway: CREATED
ApiGateway->>-Client: CREATED

Client->>+ApiGateway: GET /external/reports/{group_id} (token)
ApiGateway->>+ReportCreator: GET /external/reports/{group_id}(id)
ReportCreator->>+GroupManager: GET /internal/members/{group_id}
GroupManager->>-ReportCreator: OK (group members ids)
ReportCreator->>-ApiGateway: OK (list of report ids, attachment ids, names, date of creation)
ApiGateway->>-Client: OK (list of report ids, attachment ids, names, date of creation)

Client->>+ApiGateway: POST /external/send-report/{group_id}/{report_id} (token + report_type)
ApiGateway->>+ReportCreator: GET /external/send-report/{group_id}/{report_id} (id + report_type)
ReportCreator->>+GroupManager: GET /internal/members/{group_id}
GroupManager->>-ReportCreator: OK (group members ids)
ReportCreator->>+EmailSender: POST /internal/reports (email, report_id)
EmailSender->>-ReportCreator: OK
ReportCreator->>-ApiGateway: OK
ApiGateway->>-Client: OK

```

## Attachment Store

```mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/attachments (token + attachment)
ApiGateway->>+AttachmentStore: POST /external/attachments (id + attachment)
AttachmentStore->>-ApiGateway: CREATED (attachment_id)
ApiGateway->>-Client: CREATED (attachment_id)

Client->>+ApiGateway: POST /external/attachments/groups/{group_id} (token + attachment)
ApiGateway->>+AttachmentStore: POST /external/attachments/groups/{group_id} (id + attachment)
AttachmentStore->>-ApiGateway: CREATED (attachment_id)
ApiGateway->>-Client: CREATED (attachment_id)

Client->>+ApiGateway: GET /external/attachments/{attachment_id} (token)
ApiGateway->>+AttachmentStore: GET /external/attachments/{attachment_id}  (id)
AttachmentStore->>-ApiGateway: OK (attachment)
ApiGateway->>-Client: OK (attachment)

Client->>+ApiGateway: GET /external/attachments/groups/{group_id}/{attachment_id} (token)
ApiGateway->>+AttachmentStore: GET /external/attachments/groups/{group_id}/{attachment_id}  (id)
AttachmentStore->>-ApiGateway: OK (attachment)
ApiGateway->>-Client: OK (attachment)

Client->>+ApiGateway: PUT /external/attachments/{attachmentId} (token + updated attachment)
ApiGateway->>+AttachmentStore: PUT /external/attachments/{attachmentId} (id + updated attachment)
AttachmentStore->>-ApiGateway: OK (attachment_id)
ApiGateway->>-Client: OK (attachment_id)

Client->>+ApiGateway: PUT /external/attachments/groups/{groupId}/{attachmentId} (token + updated attachment)
ApiGateway->>+AttachmentStore: PUT /external/attachments/groups/{groupId}/{attachmentId} (id + updated attachment)
AttachmentStore->>-ApiGateway: OK (attachment_id)
ApiGateway->>-Client: OK (attachment_id)

```