sequenceDiagram
    participant Frontend
    participant Pablo
    participant Alfred
    
    Frontend->>Pablo: Intent Creation

    activate Pablo
    Pablo-->>Pablo: Check if Vendor has a UEN number and user is non-KYC
        Note right of Pablo: In case of non-UEN vendors and non-KYC users, Pablo will send a list of acceptable payment methods and card attributes as constraints
    deactivate Pablo


    Pablo->>+Alfred: Get Payment Methods (Payment Method and Card Constraints)

    Alfred-->>Alfred: Filter Payment Methods and Payment Instruments <br/>based on Payment Method and Card Constraints
    Alfred-->>-Pablo: Available Payment Methods (Payment Methods, Saved Tokens)
    Pablo-->>Frontend: Available Payment Methods (Payment Methods, Saved Tokens)

    Frontend->>Pablo: Intent Confirmation (Token/Encrypted Card)        

    activate Pablo
    Pablo-->>Pablo: Check if Vendor has a UEN number and user is non-KYC
        Note right of Pablo: In case of non-UEN vendors and non-KYC users, Pablo will send a list of acceptable payment methods and card attributes as constraints
    Pablo->>Alfred: Authorize Payment (Token/Encrypted Card, Payment Method and Card Constraints)        
    deactivate Pablo

    activate Alfred
        Alfred->>Alfred: Check if transaction is supported <br/>based on Payment Method and Card Constraints
    deactivate Alfred

    alt Payment Method/Token is Supported
        Alfred-->>Pablo: Payment Success
        activate Pablo
        Pablo-->>Pablo: Confirm Order
        deactivate Pablo
        Pablo-->>Frontend: Payment Completed and Order Confirmed
    else Payment Method/Token is NOT Supported
        Alfred-->>Pablo: Payment Declined    
        Pablo-->>Frontend: Payment Declined (Pandora exception error)
    end
