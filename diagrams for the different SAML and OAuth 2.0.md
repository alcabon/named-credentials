Here are the sequence diagrams for the different SAML and OAuth 2.0 \+ OpenID Connect scenarios:

**1\. SAML: SP-Initiated Flow (Salesforce as SP, External IdP)**

This flow starts when the user tries to access Salesforce directly.

```mermaid
sequenceDiagram
    participant UB as User's Browser
    participant SF as Salesforce (SP)
    participant ExtIdP as External IdP

    UB->>+SF: Access Salesforce URL/Resource
    SF-->>-UB: Redirect to External IdP (with SAML Request)
    UB->>+ExtIdP: Send SAML Request
    Note right of ExtIdP: If no active IdP session, prompt user
    ExtIdP->>UB: Prompt for Credentials
    UB->>ExtIdP: Submit Credentials
    ExtIdP->>ExtIdP: Authenticate User
    ExtIdP-->>-UB: Redirect to Salesforce ACS (with SAML Response/Assertion)
    UB->>+SF: Send SAML Response/Assertion to ACS URL
    SF->>SF: Validate Assertion Signature & Conditions
    SF->>SF: Identify User (e.g., Federation ID)
    SF->>SF: Create Salesforce Session
    SF-->>-UB: Grant Access / Serve Resource
```
**2\. SAML: IdP-Initiated Flow (Salesforce as SP, External IdP)**

This flow starts when the user is already logged into the external IdP's portal and clicks a link to access Salesforce.

```mermaid
sequenceDiagram
    participant UB as User's Browser
    participant ExtIdP as External IdP
    participant SF as Salesforce (SP)

    UB->>+ExtIdP: Access IdP Portal / Login
    Note right of ExtIdP: User authenticates if no active session
    ExtIdP->>ExtIdP: User is Authenticated
    UB->>ExtIdP: Click Salesforce Application Link/Tile
    ExtIdP-->>-UB: Redirect to Salesforce ACS (with SAML Response/Assertion)
    UB->>+SF: Send SAML Response/Assertion to ACS URL
    SF->>SF: Validate Assertion Signature & Conditions
    SF->>SF: Identify User (e.g., Federation ID)
    SF->>SF: Create Salesforce Session
    SF-->>-UB: Grant Access / Serve Resource
```

**3\. SAML: SP-Initiated Flow (External SP, Salesforce as IdP)**

This flow starts when the user tries to access an external application that uses Salesforce for authentication.

```mermaid
sequenceDiagram
    participant UB as User's Browser
    participant ExtSP as External SP
    participant SF as Salesforce (IdP)

    UB->>+ExtSP: Access External Application URL/Resource
    ExtSP-->>-UB: Redirect to Salesforce IdP (with SAML Request)
    UB->>+SF: Send SAML Request
    Note right of SF: If no active Salesforce session, prompt user
    SF->>UB: Prompt for Salesforce Credentials
    UB->>SF: Submit Credentials
    SF->>SF: Authenticate User
    SF-->>-UB: Redirect to External SP ACS (with SAML Response/Assertion)
    UB->>+ExtSP: Send SAML Response/Assertion to ACS URL
    ExtSP->>ExtSP: Validate Assertion Signature & Conditions
    ExtSP->>ExtSP: Identify User
    ExtSP->>ExtSP: Create Application Session
    ExtSP-->>-UB: Grant Access / Serve Resource
```

**4\. SAML: IdP-Initiated Flow (External SP, Salesforce as IdP)**

This flow starts when the user is logged into Salesforce and clicks a link (e.g., in the App Launcher) to access an integrated external application.

```mermaid
sequenceDiagram
    participant UB as User's Browser
    participant SF as Salesforce (IdP)
    participant ExtSP as External SP

    UB->>+SF: Access Salesforce / Login
    Note right of SF: User authenticates if no active session
    SF->>SF: User is Authenticated
    UB->>SF: Click External Application Link/Tile
    SF-->>-UB: Redirect to External SP ACS (with SAML Response/Assertion)
    UB->>+ExtSP: Send SAML Response/Assertion to ACS URL
    ExtSP->>ExtSP: Validate Assertion Signature & Conditions
    ExtSP->>ExtSP: Identify User
    ExtSP->>ExtSP: Create Application Session
    ExtSP-->>-UB: Grant Access / Serve Resource
```

**5\. OAuth 2.0 \+ OpenID Connect (Authorization Code Flow)**

This is the standard flow for web applications using OIDC for authentication and OAuth 2.0 for authorization, often initiated when a user clicks "Log in with Salesforce".

```mermaid
sequenceDiagram
    participant UB as User's Browser
    participant ClientApp as Client Application (RP)
    participant SF as Salesforce (Auth Server/OP/Resource Server)

    UB->>+ClientApp: Initiate Login/Action
    ClientApp-->>-UB: Redirect to Salesforce Auth Endpoint (requesting 'openid', 'api', 'refresh_token' scopes)
    UB->>+SF: Request Authorization & Authentication
    Note right of SF: If no active session or consent needed
    SF->>UB: Prompt for Login & Consent
    UB->>SF: Submit Credentials & Grant Consent
    SF->>SF: Authenticate User, Validate Consent
    SF-->>-UB: Redirect to Client App Callback URL (with Authorization Code)
    UB->>+ClientApp: Send Authorization Code via Callback
    ClientApp->>+SF: Exchange Auth Code for Tokens (Server-to-Server POST to Token Endpoint)
    Note right of ClientApp: Includes code, client_id, client_secret
    SF-->>-ClientApp: Return Access Token, Refresh Token, ID Token
    ClientApp->>ClientApp: Validate ID Token, Store Tokens Securely
    ClientApp->>+SF: Access Protected API Resource (using Access Token)
    SF-->>-ClientApp: Return Requested Data/Resource
```
