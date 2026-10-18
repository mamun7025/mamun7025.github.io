# Card Transaction Flow: 100 AED Visa Payment at Safari Mall

## Parties

| Role | Party |
|---|---|
| Cardholder | Al Mamun |
| Merchant | Safari Mall |
| Acquiring bank | Emirates NBD |
| Acquirer's processor | Network International |
| Card network | Visa |
| Issuer's processor | NEOPAY |
| Issuing bank | Mashreq |

## Flow Diagram (batch of 180 transactions)

```mermaid
flowchart LR
    A[Cardholders - 180 transactions] -->|Swipe/punch Visa cards at Safari Mall| B[Merchant - Safari Mall]
    B -->|Sends all 180 auth requests| C[Acquiring Bank - Emirates NBD]
    C --> D[Acquirer Processor - Network International]
    D --> E[Card Network - Visa]
    E --> F[Issuer Processor - NEOPAY]
    F --> G[Issuing Bank - Mashreq]
    G -->|Approves each of the 180 individually| F
    F --> E --> D --> C
    C -->|180 approvals returned| B

    B -->|End of day: batches all 180 into one clearing file| C
    C --> D --> E --> F --> G
    G -->|Settles net total for all 180 transactions| E
    E -->|Transfers funds| C
    C -->|Credits Safari Mall, minus fees| B
```

## Sequence Diagram (batch of 180 transactions)

```mermaid
sequenceDiagram
    participant Cardholders as 180 Cardholders
    participant Merchant as Safari Mall
    participant Acquirer as Emirates NBD
    participant AcqProc as Network International
    participant Network as Visa
    participant IssProc as NEOPAY
    participant Issuer as Mashreq

    loop For each of the 180 transactions
        Cardholders->>Merchant: Swipe Visa card
        Merchant->>Acquirer: Auth request
        Acquirer->>AcqProc: Forward request
        AcqProc->>Network: Route via Visa
        Network->>IssProc: Forward request
        IssProc->>Issuer: Forward request
        Note over Issuer: Check balance and validity,<br/>post pending hold in ledger
        Issuer-->>IssProc: Approve/Decline
        IssProc-->>Network: Forward response
        Network-->>AcqProc: Route response
        AcqProc-->>Acquirer: Forward response
        Acquirer-->>Merchant: Approved
        Merchant-->>Cardholders: Receipt
    end

    Note over Merchant,Acquirer: End of day - batch all 180 approved transactions
    Merchant->>Acquirer: Clearing file (180 transactions)
    Acquirer->>AcqProc: Forward clearing file
    AcqProc->>Network: Submit for settlement
    Network->>IssProc: Forward settlement request
    IssProc->>Issuer: Forward settlement request
    Note over Issuer: Success: pending hold becomes posted<br/>Failure: reversal entry releases the hold
    Issuer-->>Network: Settle net funds
    Network-->>Acquirer: Transfer funds
    Acquirer-->>Merchant: Credit account (minus fees)
```

## Key Points

- **Authorization is real time.** Each transaction is approved individually, and the issuer immediately posts a pending hold (ledger entry) against the cardholder's available balance.
- **Clearing and settlement are batched.** Usually once a day, all approved transactions are bundled into a clearing file and settled as one net transfer between banks via Visa.
- **Pending becomes posted.** If settlement succeeds, the pending entry is flagged as posted. If it fails, the issuer posts a reversal entry that releases the hold.
- **Reversals are paired by reference ID.** The original debit and its reversal share a transaction or authorization ID, so the app can merge or hide the pair.
- **Running balance is calculated server-side.** Each ledger record stores its own "balance after transaction" value, and the mobile app only displays it.
