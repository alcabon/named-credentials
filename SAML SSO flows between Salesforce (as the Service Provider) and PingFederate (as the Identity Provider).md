Mermaid sequence diagrams to illustrate the SAML SSO flows between Salesforce (as the Service Provider) and PingFederate (as the Identity Provider).

Here are the diagrams for both SP-Initiated and IdP-Initiated SSO:

### **SP-Initiated SSO Flow**

This flow starts when the user tries to access Salesforce directly.

```mermaid
sequenceDiagram
    actor "User/Browser"
    participant SF as Salesforce (SP)
    participant PF as PingFederate (IdP)

    "User/Browser"->>SF: 1. Attempts to access Salesforce resource
    SF->>"User/Browser": 2. Redirects to PingFederate (IdP) with SAML AuthnRequest \n(SF signs AuthnRequest with its Request Signing Certificate)
    "User/Browser"->>PF: 3. Sends SAML AuthnRequest to PingFederate SSO URL
    PF-->>PF: 4. Verifies AuthnRequest signature \n(using SF's public Request Signing Certificate)
    PF-->>"User/Browser": 5. Prompts for authentication (if no active session)
    "User/Browser"->>PF: 6. Submits credentials
    PF-->>PF: 7. Authenticates user
    PF-->>PF: 8. Generates SAML Assertion
    note right of PF: Assertion is signed with PF's IdP Signing Certificate. \nOptionally encrypted with SF's Assertion Decryption Certificate.
    PF->>"User/Browser": 9. Sends SAML Assertion (via HTTP POST to SF ACS URL)
    "User/Browser"->>SF: 10. Submits SAML Assertion to Salesforce ACS URL
    SF-->>SF: 11. Verifies Assertion signature (using PF's public IdP Signing Certificate)
    SF-->>SF: 12. Decrypts Assertion if encrypted \n(using SF's private Assertion Decryption Certificate)
    SF-->>SF: 13. Processes Assertion, identifies user, creates session
    SF->>"User/Browser": 14. Grants access to Salesforce resource
```

**Details from the flow** 1**:**

* **Step 2 (SF to User/Browser):** Salesforce initiates the SAML request and redirects the user to PingFederate's Single Sign-On Endpoint URL. The SAML AuthnRequest may be signed by Salesforce's Request Signing Certificate.1  
* **Step 4 (PF):** PingFederate verifies the signature of the SAML AuthnRequest using the Salesforce Request Signing Certificate.1  
* **Step 6 (PF to User/Browser):** PingFederate generates a SAML Response (Assertion) signed with its PingFederate Signing Certificate and directs it to Salesforce's Assertion Consumer Service (ACS) URL via POST.1  
* **Step 8 (SF):** Salesforce verifies the signature of the SAML Response using the PingFederate Signing Certificate.1

### **IdP-Initiated SSO Flow**

This flow starts when the user initiates login from PingFederate or a PingFederate-linked portal.

```mermaid
sequenceDiagram
    actor "User/Browser"
    participant PF as PingFederate (IdP)
    participant SF as Salesforce (SP)

    "User/Browser"->>PF: 1. Accesses PingFederate IdP-initiated SSO URL for Salesforce \n(e.g., /idp/startSSO.ping?PartnerSpId=Salesforce_Entity_ID)
    PF-->>"User/Browser": 2. Prompts for authentication (if no active session)
    "User/Browser"->>PF: 3. Submits credentials (if prompted)
    PF-->>PF: 4. Authenticates user (if credentials submitted or session exists)
    PF-->>PF: 5. Generates SAML Assertion
    note right of PF: (signed with PF's IdP Signing Certificate. \nOptionally encrypted with SF's Assertion Decryption Certificate)
    PF->>"User/Browser": 6. Sends SAML Assertion (via HTTP POST to SF ACS URL)
    "User/Browser"->>SF: 7. Submits SAML Assertion to Salesforce ACS URL
    SF-->>SF: 8. Verifies Assertion signature \n(using PF's public IdP Signing Certificate)
    SF-->>SF: 9. Decrypts Assertion if encrypted \n(using SF's private Assertion Decryption Certificate)
    SF-->>SF: 10. Processes Assertion, identifies user, creates session
    SF->>"User/Browser": 11. Grants access to Salesforce
```

**Details from the flow** 1**:**

* **Step 1 (User/Browser to PF):** User accesses the PingFederate SSO application endpoint for the Salesforce SP connection (e.g., https://\<your-pingfederate-domain\>/idp/startSSO.ping?PartnerSpId=\<Salesforce-Entity-ID\>).1  
* **Step 5 (PF):** Upon successful authentication, PingFederate generates a SAML assertion signed with the PingFederate Signing Certificate.1  
* **Step 6 (PF to User/Browser):** PingFederate sends this SAML assertion to the Salesforce ACS URL via HTTP POST.1  
* **Step 8 (SF):** Salesforce receives the SAML assertion and verifies the digital signature using the PingFederate Signing Certificate.1

These diagrams should give you a clear visual representation of the SAML SSO processes.
