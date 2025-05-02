# named-credentials

Here is a Mermaid diagram illustrating the modern Named Credential framework, highlighting the relationships between components and detailing the configuration for common OAuth 2.0 flows:

```mermaid
graph TD
    subgraph Callout Initiation & Endpoint
        Apex/Flow -- "Uses callout:NC_Name/path" --> NC
    end

    subgraph Authentication Framework
        NC -- "1. Links to" --> EC
        EC -- "2. Has 1..N" --> P
        P -- "3. Granted Access Via" --> PS
        PS -- "Assigned To" --> User
    end

    subgraph Authentication Details & Flows
        EC -- "Protocol: OAuth 2.0" --> OAuthDetails{"OAuth 2.0 Configuration"}
        EC -- "Other Protocols" --> OtherProtocols

        subgraph OAuth 2.0 Flows [Configured via EC, Principal & Auth Provider]
            OAuthDetails -- "Links to (Optional)" --> AP

            OAuthDetails -- "Flow Type: Browser Flow" --> BF
            BF -- "Stores User Tokens" --> UEC["User External Credential"]
            User -- "Authenticates & Consents" --> BF

            OAuthDetails -- "Flow Type: Client Credentials" --> CC

            OAuthDetails -- "Flow Type: JWT Bearer" --> JWT

            OAuthDetails -- "Flow Type: Password Credentials" --> PW
        end

        subgraph Principal Parameter Storage
            P -- "Named Principal" --> NPParams
            P -- "Per-User Principal" --> PUPParams
        end
    end

    style NC fill:#f9f,stroke:#333,stroke-width:2px
    style EC fill:#ccf,stroke:#333,stroke-width:2px
    style P fill:#9cf,stroke:#333,stroke-width:2px
    style PS fill:#lightgrey,stroke:#333,stroke-width:1px
    style AP fill:#f9d,stroke:#333,stroke-width:1px
    style UEC fill:#cfc,stroke:#333,stroke-width:1px

    classDef params font-size:10px,fill:#fff,stroke:#ccc,stroke-width:1px
    class BF,CC,JWT,PW,NPParams,PUPParams params
    classDef keycomp font-weight:bold
    class NC,EC,P keycomp
```

**Explanation:**

1. **Callout Initiation:** Shows how Apex or Flow initiates a callout using the Named Credential URL.  
2. **Authentication Framework:** Illustrates the core relationship: Named Credential links to an External Credential, which has one or more Principals. Access to use the Principal is granted via Permission Sets/Profiles assigned to Users.  
3. **Authentication Details & Flows:**  
   * Shows that the External Credential defines the main Authentication Protocol.  
   * Zooms into **OAuth 2.0**, showing its link to an optional Auth Provider (often holding endpoints and sometimes credentials).  
   * Details the configuration for common **OAuth 2.0 Flows** (Browser/Auth Code, Client Credentials, JWT Bearer, Password Credentials), listing the key parameters and where they are typically configured (EC, Principal, or AP).  
   * Highlights the role of the **User External Credential** in storing user-specific tokens for Per-User flows like the Browser Flow.  
   * Shows where specific credentials or parameters are typically stored within **Named** vs. **Per-User Principals**.

This diagram provides a visual map of the components, their relationships, and how different OAuth 2.0 flows are configured within the modern framework.
