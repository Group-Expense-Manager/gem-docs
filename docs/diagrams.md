# Diagrams

### System C4 Diagram
``` mermaid
C4Component

Person(User, "User")

Boundary(system_boundary, "System","") {
    
  Component(MobileApp, "Mobile App")
  Boundary(cluster_boundary, "Cluster", "") {
    Boundary("public_boundary","Public","") {
        Component(ApiGateway, "Api Gateway")
    }
    Boundary("private_boundary","Private","") {
        Component(ServiceA, "Service A")
        Component(ServiceB, "Service B")
        Component(ServiceC, "Service C")
    }


  }

Boundary(db_boundary, "DB", "") {
    ComponentDb(DBA, "Data Base A")
    ComponentDb(DBB, "Data Base B")
    ComponentDb(DBC, "Data Base C")
  }

}

Rel(User, MobileApp, "Uses")
UpdateRelStyle(User, MobileApp, $offsetY="15", $offsetX="10")
Rel(MobileApp, ApiGateway, "Makes api calls to")
UpdateRelStyle(MobileApp, ApiGateway, $offsetY="-0", $offsetX="-65")
Rel(ApiGateway, ServiceA, "Makes api calls to")
UpdateRelStyle(ApiGateway, ServiceA, $offsetY="-10", $offsetX="-50")
Rel(ApiGateway, ServiceB, "Makes api calls to")
UpdateRelStyle(ApiGateway, ServiceB, $offsetY="10", $offsetX="-50")
Rel(ApiGateway, ServiceC, "Makes api calls to")
UpdateRelStyle(ApiGateway, ServiceC, $offsetY="20", $offsetX="-80")
Rel(ServiceA,DBA, "Reads from & writes to")
UpdateRelStyle(ServiceA, DBA, $offsetY="-15", $offsetX="-70")
Rel(ServiceB,DBB, "Reads from & writes to")
UpdateRelStyle(ServiceB, DBB, $offsetY="-15", $offsetX="-70")
Rel(ServiceC,DBC, "Reads from & writes to")
UpdateRelStyle(ServiceC, DBC, $offsetY="-15", $offsetX="-70")
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
UserDetailsManager->>+ GroupManager: /internal/group?user_id=val
GroupManager->>- UserDetailsManager: OK
UserDetailsManager->>+ GroupManager: /internal/group/{group_id}/ids
GroupManager->>- UserDetailsManager: OK
UserDetailsManager->>-ApiGateway: OK
ApiGateway->>-Client: OK
```

## Group Manager

``` mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/group (token + group name,currencies,options,colour)
ApiGateway->>+GroupManager:  POST /external/group (id + group name,currencies,options,colour)
GroupManager->>+CurrencyManager: GET /internal/currency
CurrencyManager->>-GroupManager: OK (available currencies)
GroupManager->>-ApiGateway: CREATED
ApiGateway->>-Client: CREATED

Client->>+ApiGateway: DELETE /external/group/{group_id} (token)
ApiGateway->>+GroupManager:  POST /external/group/{group_id} (id)
GroupManager->>+AligmentConnector: GET /internal/balance/{group_id} (id)
AligmentConnector->>-GroupManager: OK (balance of users)
GroupManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: PUT /external/group/{group_id} (token + group name,colour)
ApiGateway->>+GroupManager:  PUT /external/group/{group_id} (id + group name,colour)
GroupManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/group/{group_id} (token)
ApiGateway->>+GroupManager:  GET /external/group/{group_id} (id)
GroupManager->>-ApiGateway: OK (group data)
ApiGateway->>-Client: OK (group data)

Client->>+ApiGateway: POST /external/group/join (token + join code)
ApiGateway->>+GroupManager:  POST /external/group/join (id + join code)
GroupManager->>-ApiGateway: OK (group data)
ApiGateway->>-Client: OK (group data)
```

## Expense Manager

``` mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/expense/{group_id} (token + expense data)
ApiGateway->>+ExpenseManager: POST /external/expense/{group_id} (id + expense data)
ExpenseManager->>+GroupManager: GET /internal/group/{group_id}
GroupManager->>-ExpenseManager: OK (group data)
ExpenseManager->>-ApiGateway: CREATED
ApiGateway->>-Client: CREATED


Client->>+ApiGateway: PUT /external/expense/{group_id}/{expense_id} (token + updated expense data)
ApiGateway->>+ExpenseManager: PUT /external/expense/{group_id}/{expense_id} (id + updated expense data)
ExpenseManager->>+GroupManager: GET /internal/group/{group_id}
GroupManager->>-ExpenseManager: OK (group data)
ExpenseManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: DELETE /external/expense/{group_id}/{expense_id} (token)
ApiGateway->>+ExpenseManager: DELETE /external/expense/{group_id}/{expense_id} (id)
ExpenseManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/expense/{group_id} (token + filters)
ApiGateway->>+ExpenseManager: GET /external/expense/{group_id} (id + filters)
ExpenseManager->>+GroupManager: GET /internal/group?user_id=val
GroupManager->>-ExpenseManager: OK
ExpenseManager->>-ApiGateway: OK + ids + names  + date + status
ApiGateway->>-Client: OK + ids + names  + date + status


Client->>+ApiGateway: GET /external/expense/{expense_id} (token)
ApiGateway->>+ExpenseManager: GET /external/expense/{expense_id} (id)
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

Client->>+ApiGateway: POST /external/payment/{group_id} (token + payment data)
ApiGateway->>+PaymentManager: POST /external/payment/{group_id} (id + payment data)
PaymentManager->>+GroupManager: GET /internal/group/{group_id}
GroupManager->>-PaymentManager: OK (group data)
PaymentManager->>-ApiGateway: CREATED
ApiGateway->>-Client: CREATED

Client->>+ApiGateway: PUT /external/payment/{group_id}/{payment_id} (token + updated payment data)
ApiGateway->>+PaymentManager: PUT /external/payment/{group_id}/{payment_id} (id + updated payment data)
PaymentManager->>+GroupManager: GET /internal/group/{group_id}
GroupManager->>-PaymentManager: OK (group data)
PaymentManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: DELETE /external/payment/{group_id}/{payment_id} (token)
ApiGateway->>+PaymentManager: DELETE /external/payment/{group_id}/{payment_id} (id)
PaymentManager->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/payment/{group_id} (token + filters)
ApiGateway->>+PaymentManager: GET /external/payment/{group_id} (id + filters)
PaymentManager->>+GroupManager: GET /internal/group?user_id=val
GroupManager->>-PaymentManager: OK
PaymentManager->>-ApiGateway: OK + ids + names + date + status
ApiGateway->>-Client: OK + ids + names + date + status


Client->>+ApiGateway: GET /external/payment/{payment_id} (token + filters)
ApiGateway->>+PaymentManager: GET /external/payment/{payment_id} (id + filters)
PaymentManager->>-ApiGateway: OK + payment data
ApiGateway->>-Client: OK + payment data

Client->>+ApiGateway: POST /external/accept-payment/{payment_id} (token + resolve)
ApiGateway->>+PaymentManager: POST /external/accept-payment/{payment_id} (id + resolve)
PaymentManager->>-ApiGateway: OK
ApiGateway->>-Client: OK
```

## Alignment Connector

``` mermaid
sequenceDiagram

Client->>+ApiGateway: GET /external/balance/{group_id}/{user_id} (token)
ApiGateway->>+AlignmentConnector: GET /external/balance/{group_id}/{user_id} (id)
AlignmentConnector->>+ ExpenseManager: GET /internal/balance/{group_id}/{user_id} (id)
ExpenseManager->>- AlignmentConnector: OK (balance of user)
AlignmentConnector->>+ PaymentManager: GET /internal/balance/{group_id}/{user_id} (id)
PaymentManager->>- AlignmentConnector: OK (balance of user)
AlignmentConnector->>- ApiGateway: OK (balance of user)
ApiGateway->>-Client: OK (balance of user)

Client->>+ApiGateway: GET /external/alignment/{group_id} (token)
ApiGateway->>+AlignmentConnector: GET /external/alignment/{group_id} (id)
AlignmentConnector->>+ ExpenseManager: GET /internal/balance/{group_id}/ (id)
ExpenseManager->>- AlignmentConnector: OK (balance of group members)
AlignmentConnector->>+ PaymentManager: GET /internal/balance/{group_id} (id)
PaymentManager->>- AlignmentConnector: OK (balance of group members)
AlignmentConnector->>- ApiGateway: OK (balance of group members & alignment)
ApiGateway->>-Client: OK (balance of group members & alignment)
```

## Currency Manager

``` mermaid
 sequenceDiagram

Client->>+ApiGateway: GET /external/currency (token)
ApiGateway->>+CurrencyManager:  GET /external/currency
CurrencyManager->>-ApiGateway: OK (available currencies)
ApiGateway->>-Client: OK (available currencies)

Client->>+ApiGateway: GET /external/exchange-rate (token + currencyFrom,currencyTo)
ApiGateway->>+CurrencyManager:  GET /external/exchange-rate (currencyFrom,currencyTo)
CurrencyManager->>+ ExchangeRateProvider: GET /exchangeRateProvider (currencyFrom,currencyTo)
ExchangeRateProvider->>-CurrencyManager: OK (exchange rate)
CurrencyManager->>-ApiGateway: OK (exchange rate)
ApiGateway->>-Client: OK (exchange rate)
```

## Report Creator

``` mermaid
sequenceDiagram

Client->>+ApiGateway: GET /external/report-pdf/{group_id}?dateFrom=val1&dateTo=val2 (token)
ApiGateway->>+ReportCreator: GET /external/report-pdf/{group_id}?dateFrom=val1&dateTo=val2 (id)
alt report is cached
ReportCreator->>+ AttachmentStore: GET /internal/group/{group_id}(id + report pdf)
AttachmentStore->>- ReportCreator: OK + report
else report is not cached
ReportCreator->>+ ExpenseManager: GET /internal/expense/{group_id} (id)
ExpenseManager->>- ReportCreator: OK (expense data)
ReportCreator->>+ PaymentManager: GET /internal/payment/{group_id}(id)
PaymentManager->>- ReportCreator: OK (payment data)
ReportCreator->>+ AttachmentStore: POST /internal/group/{group_id}(id + report pdf)
AttachmentStore->>- ReportCreator: CREATED
end
ReportCreator->>- ApiGateway: OK (report pdf)
ApiGateway->>-Client: OK (report pdf)

Client->>+ApiGateway: GET /external/report-csv/{group_id}?dateFrom=val1&dateTo=val2 (token)
ApiGateway->>+ReportCreator: GET /external/report-csv/{group_id}?dateFrom=val1&dateTo=val2 (id)
alt report is cached
ReportCreator->>+ AttachmentStore: GET /internal/group/{group_id}(id + report csv)
AttachmentStore->>- ReportCreator: OK + report
else report is not cached
ReportCreator->>+ ExpenseManager: GET /internal/expense/{group_id} (id)
ExpenseManager->>- ReportCreator: OK (expense data)
ReportCreator->>+ PaymentManager: GET /internal/payment/{group_id}(id)
PaymentManager->>- ReportCreator: OK (payment data)
ReportCreator->>+ AttachmentStore: POST /internal/group/{group_id}(id + report csv)
AttachmentStore->>- ReportCreator: CREATED
end
ReportCreator->>- ApiGateway: OK (report csv)
ApiGateway->>-Client: OK (report csv)
```

## Attachment Store

``` mermaid
sequenceDiagram

Client->>+ApiGateway: POST /external/attachments (token + attachment)
ApiGateway->>+ImageStore: POST /external/attachments (id + attachment)
ImageStore->>-ApiGateway: CREATED (attachment_id)
ApiGateway->>-Client: CREATED (attachment_id)

Client->>+ApiGateway: POST /external/attachments/groups/{group_id} (token + attachment)
ApiGateway->>+ImageStore: POST /external/attachments/groups/{group_id} (id + attachment)
ImageStore->>-ApiGateway: CREATED (attachment_id)
ApiGateway->>-Client: CREATED (attachment_id)

Client->>+ApiGateway: GET /external/attachments/{attachment_id} (token)
ApiGateway->>+ImageStore: GET /external/attachments/{attachment_id}  (id)
ImageStore->>-ApiGateway: OK (attachment)
ApiGateway->>-Client: OK (attachment)

Client->>+ApiGateway: GET /external/attachments/groups/{group_id}/{attachment_id} (token)
ApiGateway->>+ImageStore: GET /external/attachments/groups/{group_id}/{attachment_id}  (id)
ImageStore->>-ApiGateway: OK (attachment)
ApiGateway->>-Client: OK (attachment)

Client->>+ApiGateway: PUT /external/attachments/{attachmentId} (token + updated attachment)
ApiGateway->>+ImageStore: PUT /external/attachments/{attachmentId} (id + updated attachment)
ImageStore->>-ApiGateway: OK (attachment_id)
ApiGateway->>-Client: OK (attachment_id)

Client->>+ApiGateway: PUT /external/attachments/groups/{groupId}/{attachmentId} (token + updated attachment)
ApiGateway->>+ImageStore: PUT /external/attachments/groups/{groupId}/{attachmentId} (id + updated attachment)
ImageStore->>-ApiGateway: OK (attachment_id)
ApiGateway->>-Client: OK (attachment_id)

```