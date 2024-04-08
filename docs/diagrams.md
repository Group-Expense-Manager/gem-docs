# Diagrams

## Authenticator
``` mermaid
Client->>+ApiGateway: POST open/register (user data)
ApiGateway->>+Authenticator: POST open/register (user data)
Authenticator-->>-ApiGateway: CREATED
ApiGateway-->>-Client: CREATED

Client->>+ApiGateway: POST /open/login (credentials)
ApiGateway->>+Authenticator: POST /open/login (credentials)
Authenticator-->>-ApiGateway: OK (token)
ApiGateway-->>-Client: OK (token)

Client->>+ApiGateway: POST /open/verify (email address, code)
ApiGateway->>+Authenticator: POST /open/verify (email address, code)
Authenticator->>+ UserDetailsManager: POST /internal/user-details (id + email as username)
UserDetailsManager->>-Authenticator: CREATED
Authenticator-->>-ApiGateway: OK (token)
ApiGateway-->>-Client: OK (token)

Client->>+ApiGateway: POST /open/send-verification-email (email address)
ApiGateway->>+Authenticator: POST open/send-verification-email (email address)
Authenticator->>+EmailSender: POST /internal/verification (email address + code)
EmailSender->>EmailClient: EMAIL
EmailSender-->>-Authenticator: OK
Authenticator-->>-ApiGateway: OK
ApiGateway-->>-Client: OK

Client->>+ApiGateway: POST /open/recover-password (email address)
ApiGateway->>+Authenticator: POST /open/recover-password (email address)
Authenticator->>+EmailSender: POST /internal/verification (email address + link)
EmailSender->>EmailClient: EMAIL
EmailSender-->>-Authenticator: OK
Authenticator-->>-ApiGateway: OK
ApiGateway-->>-Client: OK

Client->>+ApiGateway: POST /open/send-password?email=val&code=val 
ApiGateway->>+Authenticator: POST /open/send-password?email=val&code=val
Authenticator->>+EmailSender: POST /internal/password (email address + code)
EmailSender->>EmailClient: EMAIL
EmailSender-->>-Authenticator: OK
Authenticator-->>-ApiGateway: OK
ApiGateway-->>-Client: OK


Client->>+ApiGateway: POST external/change-password (token + old&new password)
ApiGateway->>+Authenticator: POST external/change-password (id + old&new password)
Authenticator-->>-ApiGateway: OK
ApiGateway-->>-Client: OK
```
## User Details Manager

``` mermaid
sequenceDiagram
title UserDetailsManager

Client->>+ApiGateway: PUT /external/user-details (token + username,firstname,lastname,pfp?)
ApiGateway->>+Authenticator: PUT /external/user-details (id + username,firstname,lastname,pfp?)
Authenticator->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/user-details (token)
ApiGateway->>+Authenticator: GET /external/user-details (id)
Authenticator->>-ApiGateway: OK
ApiGateway->>-Client: OK

Client->>+ApiGateway: GET /external/user-details/{group_id} (token)
ApiGateway->>+Authenticator: GET /external/user-details/{group_id} (id)
Authenticator->>+ GroupManager: /internal/group/{group_id}/ids
GroupManager->>- Authenticator: OK
Authenticator->>+ GroupManager: /internal/group?user_id=val
GroupManager->>- Authenticator: OK
Authenticator->>-ApiGateway: OK
ApiGateway->>-Client: OK
```