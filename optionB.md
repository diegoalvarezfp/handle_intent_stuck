sequenceDiagram
    participant Frontend
    participant Pablo
    participant Alfred
    

    Frontend->>Pablo: Get Payment Methods
    Pablo->>+Alfred: Get Payment Methods

    opt Saved Cards With No Metadata
        alt Card Metadata Lookup via Bin Table
            activate Alfred
            Alfred-->>+Alfred: Lookup Card Metadata from Bin Table
            deactivate Alfred
        else Card Metadata Loookup via PSP       
        end
        Alfred-->>+Alfred: Save Card Metadata
            Note right of Alfred: Persist Card Metadata to avoid <br />lookup in future transactions
    end

    Alfred-->>-Pablo: Available Payment Methods (Payment Methods, Saved Tokens with Metadata)
    activate Pablo
    Pablo-->>Pablo: Cache Metadata to Token-keyed Dictionary
    Pablo-->>Pablo: Filter Supported Payment Methods and Tokens
        Note right of Pablo: In case of non-UEN vendors and non-KYC users, remove unsupported payment targets and foreign issued saved tokens/cards
    Pablo-->>Frontend: Available Payment Methods (Payment Methods, Saved Tokens)
    deactivate Pablo

    alt Select Saved Card or Token
        Frontend->>Pablo: Authorize Payment (Token)        
    else New Card Entry
        Frontend->>Pablo: Authorize Payment (Encrypted Card)
        Pablo->>Alfred: Tokenize Card (Encrypted Card)
        activate Alfred
        opt No Metadata from PSP
            Alfred-->>+Alfred: Lookup Card Metadata from Bin Table        
        end
        Alfred-->>Alfred: Save New Card Token with Metadata
        deactivate Alfred
        Alfred-->>Pablo: Tokenized Card (Token, Metadata)
    end

    activate Pablo
    Pablo-->>Pablo: Get Cached Metadata from Token-keyed Dictionary as needed
    Pablo-->>Pablo: Validate Card Token is Allowed
        Note right of Pablo: In case of non-UEN vendors and non-KYC users, Pablo enforces that unsupported payment methods and/or foreign issued cards are not allowed
    deactivate Pablo

    alt Payment Method/Token is Supported
        Pablo->>Alfred: Authorize Payment (Token)
        Alfred-->>Pablo: Payment Success
        activate Pablo
        Pablo-->>Pablo: Confirm Order
        deactivate Pablo
        Pablo-->>Frontend: Payment Completed and Order Confrimed
    else Payment Method/Token is NOT Supported
        Pablo-->>Frontend: Payment Declined
    end
