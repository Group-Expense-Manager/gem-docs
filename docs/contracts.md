# Api Contracts

## Authenticator

```mermaid
classDiagram
class NotVerifiedUser {
    id: String
    email: String
    password: String
    createdAt: Date
    code: String
    codeUpdatedAt: Date
}

class VerifiedUser {
    id: String
    email: String
    password: String
    passwordRecoveryCode: String
    passwordRecoveryEmailSentAt: Date?
}

class Verification {
    email: String
    code: String
}
```

```mermaid
sequenceDiagram

    participant Client
    participant Authenticator

    Client->>Authenticator: Register POST /open/register
    Note over Client,Authenticator:Body: {<br>email: String <br> password: String<br>}
    Authenticator->>Authenticator:Validate body
    alt Body is Valid
        Authenticator->>Authenticator:Check if email is not taken
        alt Email is not taken
            Authenticator->>Authenticator:Register user
            Authenticator->>Client: 201 Created
        else Email is taken
            Authenticator->>Client: 409 Conflict
        end
    else Body is not Valid
        Authenticator->>Client: 400 Bad Request
    end

    Client->>Authenticator: Login POST /open/login
    Note over Client,Authenticator: Body: {<br>email: String <br> password: String<br>}
    Authenticator->>Authenticator:Validate body
    alt Body is Valid
        Authenticator->>Authenticator:Authenticate
        alt Successfully authenticated
        Authenticator->>Authenticator:Create token
        Authenticator->>Client: 200 OK
        Note over Client,Authenticator: Body: token: String

        else User is not verified
            Authenticator->>Client: 403 Forbidden
        else Bad credentials
            Authenticator->>Client: 400 Bad Request
        end
    else Body is not Valid
        Authenticator->>Client: 400 Bad Request
    end
    
    Client->>Authenticator: Register POST /open/verify
    Note over Client,Authenticator:Body: {<br>email: String <br> code: String<br>}
    Authenticator->>Authenticator:Validate body
    alt Body is Valid
        Authenticator->>Authenticator: Check if not verified user exists
        alt User exists
            Authenticator->>Authenticator: Check code
            alt Code is correct
                Authenticator->>Authenticator: Verify user
                Authenticator->>Client: 200 OK
                Note over Client,Authenticator: Body: token: String
            else Code is not correct
                Authenticator->>Client: 400 Bad Request
            end
        else User does not exist
        Authenticator->>Client: 404 Not found
        end

    else Body is not Valid
        Authenticator->>Client: 400 Bad Request
    end

    Client->>Authenticator: Send verification email POST /open/send-verification-email
    Note over Client,Authenticator:Body: {<br>email: String}
    Authenticator->>Authenticator: Validate body
    alt Body is Valid
        Authenticator->>Authenticator: Check if not verified user exists
        alt User exists
            Authenticator->>Authenticator: Check if email was recently sent
            alt Email wasn't recently sent
                Authenticator->>Authenticator: Generate code & send verification email
                Authenticator->>Client: 200 OK
            else Email was recently sent
                Authenticator->>Client: 429 Too many requests
            end
        else User does not exist
            Authenticator->>Client: 404 Not found
        end
    else Body is not Valid
        Authenticator->>Client: 400 Bad Request
    end

    Client->>Authenticator: Send password-recovery email POST /open/recover-password
    Note over Client,Authenticator:Body: {<br>email: String}
    Authenticator->>Authenticator: Validate body
    alt Body is Valid
        Authenticator->>Authenticator: Check if not verified user exists
        alt User exists
            Authenticator->>Authenticator: Check if email was recently sent
            alt Email wasn't recently sent
                Authenticator->>Authenticator: send password-recovery email
                Authenticator->>Client: 200 OK
            else Email was recently sent
                Authenticator->>Client: 429 Too many requests
            end
        else User does not exist
        Authenticator->>Client: 404 Not found
        end
    else Body is not Valid
        Authenticator->>Client: 400 Bad Request
    end

    Client->>Authenticator: Send new password by email POST /open/send-password
    Note over Client,Authenticator:RequestParams: {<br>email: String<br>code: String<br>}
    Authenticator->>Authenticator: Check if not verified user exists
    alt User exists
        Authenticator->>Authenticator: Check code
        alt Code is correct
            Authenticator->>Authenticator: Generate & sende new password by email
            Authenticator->>Client: 200 OK
        else Code is not correct
            Authenticator->>Client: 400 Bad Request
        end
    else User does not exist
        Authenticator->>Client: 404 Not found
    end

    Client->>Authenticator: Change password POST /external/change-password
    Note over Client,Authenticator:token-validated: TOKEN<br>Body: {<br>oldPassword: String<br>newPassword: String<br>}
    Authenticator->>Authenticator: Validate body
    alt Body is Valid
        Authenticator->>Authenticator: Check if user exists
        alt User exists
            Authenticator->>Authenticator: Check if old password is correct
            alt Password is correct
                Authenticator->>Authenticator: Update password
                Authenticator->>Client: 200 OK
            else Password is not correct
                Authenticator->>Client: 400 Bad Request
            end

        else User does not exist
            Authenticator->>Client: 404 Not found
        end
    else Body is not Valid
        Authenticator->>Client: 400 Bad Request
    end
```


## Email Sender

```mermaid
sequenceDiagram

    participant Authenticator
    participant EmailSender

    Authenticator->>EmailSender: Send verification email POST /internal/verification
    Note over Authenticator,EmailSender:Body: {<br>email: String <br> code: String<br>}
    EmailSender->>EmailSender:Send verification email
    EmailSender->>Authenticator: 200 OK

    Authenticator->>EmailSender: Send password-recovery email POST /internal/password-recovery
    Note over Authenticator,EmailSender:Body: {<br>email: String <br> link: String<br>}
    EmailSender->>EmailSender:Send password-recovery email
    EmailSender->>Authenticator: 200 OK


    Authenticator->>EmailSender: Send new password by email POST /internal/password
    Note over Authenticator,EmailSender:Body: {<br>email: String <br> password: String<br>}
    EmailSender->>EmailSender:Send new password
    EmailSender->>Authenticator: 200 OK
    
```

## User Details Manager

```mermaid
classDiagram

class UserDetails {
    id: String
    username: String
    firstname: String?
    lastname: String?
    attachmentId: String?
}

```

```mermaid
sequenceDiagram

    participant Client
    participant Authenticator
    participant UserDetailsManager

    Authenticator->>UserDetailsManager: Create user details POST /internal/user-details
    Note over Authenticator,UserDetailsManager: token-validated: TOKEN<br>Body: {<br>email: String <br>}
    UserDetailsManager->>UserDetailsManager: Create user details
    UserDetailsManager->>Authenticator: 201 Created

    Client->>UserDetailsManager: Change user details PUT /external/user-details
    Note over Client,UserDetailsManager: token-validated: TOKEN<br> Body: {<br>username: String <br>firstname: String? <br>lastname: String? <br>attachmentId: String?<br>}
    UserDetailsManager->>UserDetailsManager: Validate body
    alt Body is valid
        UserDetailsManager->>UserDetailsManager: Change user details
        UserDetailsManager->>Client: 200 OK
    else Body is not valid
        UserDetailsManager->>Client: 400 Bad Request
    end


    Client->>UserDetailsManager: get user details GET /external/user-details
    Note over Client,UserDetailsManager: token-validated: TOKEN
    UserDetailsManager->>UserDetailsManager: Get user details
    UserDetailsManager->>Client: 200 OK
    Note over Client,UserDetailsManager:<br> Body: {<br>username: String <br>firstname: String? <br>lastname: String? <br>attachmentId: String?<br>}

    Client->>UserDetailsManager: get user details of group members GET /external/user-details/{group_id}
    Note over Client,UserDetailsManager: token-validated: TOKEN
    UserDetailsManager->>UserDetailsManager: Check if user is a group member
    alt User is a group member
        UserDetailsManager->>UserDetailsManager: Get user details of group members
        UserDetailsManager->>Client: 200 OK
        Note over Client,UserDetailsManager:<br> Body: List{ {<br>username: String <br>firstname: String? <br>lastname: String? <br>attachmentId: String?<br>}}
    else User is not a group member
        UserDetailsManager->>Client: 403 Forbidden
    end
   
```

## Attachment Store

```mermaid

classDiagram
    class UserAttachment {
    _id: ObjectId
    userId: String
    contentType: String
    size: Number
    data: BSON
    createdAt: Date
    updatedAt: Date
    attachmentHistory: AttachmentHistory[]
    }
  
    class GroupAttachment {
        _id: ObjectId
        groupId: String
        uploadedByUser: String
        contentType: String
        size: Number
        data: BSON
        createdAt: Date
        updatedAt: Date
        attachmentHistory: AttachmentHistory[]
    }
    
    class AttachmentHistory {
        updateBy: String
        updatedAt: Date
        size: Number
    }
```

```mermaid

sequenceDiagram
    
    participant Client
    participant AttachmentStore

    Client->>AttachmentStore: Upload Attachment POST /external/attachments
    Note over Client,AttachmentStore: token-validated: TOKEN<br>Body: ByteArray
    activate AttachmentStore
    AttachmentStore->>AttachmentStore: Check media type
    alt Media type is allowed
        AttachmentStore->>AttachmentStore: Check attachment size
        alt Attachment size within limit
            AttachmentStore->>AttachmentStore: Save attachment
            AttachmentStore->>Client: 201 Created
            deactivate AttachmentStore
            Note over Client,AttachmentStore: content-type: MEDIA_TYPE<br>Body: {<br>attachmentId: "AttachmentId",<br>}
        else Attachment size exceeds limit
            AttachmentStore->>Client: 413 Payload Too Large
        end
    else Media type is not allowed
        AttachmentStore->>Client: 415 Unsupported Media Type
    end

    Client->>AttachmentStore: Upload Attachment POST external/attachments/groups/{groupId}
    Note over Client,AttachmentStore: token-validated:TOKEN<br>Body:ByteArray
    activate AttachmentStore
    AttachmentStore->>AttachmentStore: Check access to group for that user
    alt User has access to group
        AttachmentStore->>AttachmentStore: Check media type
        alt Media type is allowed
            AttachmentStore->>AttachmentStore: Check attachment size
            alt Attachment size within limit
                AttachmentStore->>AttachmentStore: Save attachment
                AttachmentStore->>Client: 201 Created
                deactivate AttachmentStore
                Note over Client,AttachmentStore: content-type:MEDIA_TYPE<br>Body: {<br>attachmentId:"AttachmentId",<br>}
            else Attachment size exceeds limit
                AttachmentStore->>Client: 413 Payload Too Large
            end
        else Media type is not allowed
            AttachmentStore->>Client: 415 Unsupported Media Type
        end
    else User does not have access to group
        AttachmentStore->>Client: 403 Forbidden
    end

    Client->>AttachmentStore: Get Attachment GET /external/attachments/{attachmentId}
    Note over Client,AttachmentStore: token-validated: TOKEN
    activate AttachmentStore
    AttachmentStore->>AttachmentStore: Check user access to attachment
    alt User has access to attachment
        AttachmentStore->>AttachmentStore: Check attachment exists
        alt Attachment is found
            AttachmentStore->>AttachmentStore: Get attachment
            AttachmentStore->>Client: 200 OK
            deactivate AttachmentStore
            Note over Client,AttachmentStore: content-type: MEDIA_TYPE<br>Body: ByteArray
        else Attachment is not found
            AttachmentStore->>Client: 404 Not Found
        end
    else User does not have access to attachment
        AttachmentStore->>Client: 403 Forbidden
    end

    Client->>AttachmentStore: Get Attachment GET /external/attachments/groups/{groupId}/{attachmentId}
    Note over Client,AttachmentStore: token-validated: TOKEN
    activate AttachmentStore
    AttachmentStore->>AttachmentStore: Check user access to attachment
    alt User has access to attachment
        AttachmentStore->>AttachmentStore: Check attachment exists
        alt Attachment is found
            AttachmentStore->>AttachmentStore: Get attachment
            AttachmentStore->>Client: 200 OK
            deactivate AttachmentStore
            Note over Client,AttachmentStore: content-type: MEDIA_TYPE<br>Body: ByteArray
        else Attachment is not found
            AttachmentStore->>Client: 404 Not Found
        end
    else User does not have access to attachment
        AttachmentStore->>Client: 403 Forbidden
    end

    Client->>AttachmentStore: Update Attachment PUT /external/attachments/{attachmentId}
    Note over Client,AttachmentStore: token-validated: TOKEN<br>Body: Updated Attachment Data
    activate AttachmentStore
    AttachmentStore->>AttachmentStore: Check if user has access to attachment
    alt User has access to attachment
        AttachmentStore->>AttachmentStore: Check if attachment exists
        alt Attachment exists
            AttachmentStore->>AttachmentStore: Check if updated media type is the same
            alt Media type is the same
                AttachmentStore->>AttachmentStore: Update attachment data
                AttachmentStore->>Client: 200 OK
                Note over Client,AttachmentStore: content-type:MEDIA_TYPE<br>Body: {<br>attachmentId:"AttachmentId",<br>}
                deactivate AttachmentStore
            else Media type is not the same
                AttachmentStore->>Client: 415 Unsupported Media Type
            end
        else Attachment does not exist
            AttachmentStore->>Client: 404 Not Found
        end
    else User does not have access to attachment
        AttachmentStore->>Client: 403 Forbidden
    end

    Client->>AttachmentStore: Update Attachment PUT /external/attachments/groups/{groupId}/{attachmentId}
    Note over Client,AttachmentStore: token-validated: TOKEN<br>Body: Updated Attachment Data
    activate AttachmentStore
    AttachmentStore->>AttachmentStore: Check if user has access to attachment in the group
    alt User has access to attachment in the group
        AttachmentStore->>AttachmentStore: Check if attachment exists in the group
        alt Attachment exists in the group
            AttachmentStore->>AttachmentStore: Check if updated media type is the same
            alt Media type is the same
                AttachmentStore->>AttachmentStore: Update attachment data
                AttachmentStore->>Client: 200 OK
                Note over Client,AttachmentStore: content-type:MEDIA_TYPE<br>Body: {<br>attachmentId:"AttachmentId",<br>}
                deactivate AttachmentStore
            else Media type is not the same
                AttachmentStore->>Client: 415 Unsupported Media Type
            end
        else Attachment does not exist in the group
            AttachmentStore->>Client: 404 Not Found
        end
    else User does not have access to attachment in the group
        AttachmentStore->>Client: 403 Forbidden
    end
    
```