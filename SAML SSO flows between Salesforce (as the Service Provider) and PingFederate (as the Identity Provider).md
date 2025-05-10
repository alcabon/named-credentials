**Important Notes Before We Start:**

* **Placeholders:** URLs will use placeholders like **https://\<your-salesforce-domain\>.my.salesforce.com**, **https://\<pf-idp-hostname.example.com\>**, and **PartnerSpId=\<Salesforce\_Entity\_ID\>**. You'll need to replace these with your actual configured values.  
* **HTTPS/TLS Certificates:** All communication should be over HTTPS. This means both Salesforce and PingFederate will have their own HTTPS/TLS certificates to secure the browser-to-server connections. These are standard web server certificates and are distinct from the SAML-specific signing and encryption certificates.  
  * **Location:** Server-side (Salesforce or PingFederate). Private keys are on the server; public keys (and their chains) are presented to the browser for validation.  
* **SAML Certificates:**  
  * **Signing Certificates:** Used to create digital signatures on SAML messages (AuthnRequest, Assertion) to ensure integrity and authenticity.  
    * **Private Key**: Held by the signing entity (e.g., **Salesforce for AuthnRequest**, **PingFederate for Assertion**). Stored securely on the respective platform.  
    * **Public Key**: Shared with the verifying entity. Stored in the verifying entity's trust configuration (e.g., PingFederate has Salesforce's public key; Salesforce has PingFederate's public key).  
  * **Encryption Certificates:** Used to encrypt SAML Assertions (or parts of them) to ensure confidentiality.  
    * **Public Key**: Held by the encrypting entity (e.g., PingFederate uses Salesforce's public encryption key).  
    * **Private Key**: Held by the decrypting entity (e.g., Salesforce uses its private decryption key). Stored securely on the platform.  
* **Metadata Exchange:** The most common and recommended way to exchange public certificates, entity IDs, and endpoint URLs is by exchanging SAML 2.0 metadata files between Salesforce and PingFederate.

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
    SF->>"User/Browser": 2. Redirects to PingFederate (IdP) with SAML AuthnRequest <br>(SF signs AuthnRequest with its Request Signing Certificate)
    "User/Browser"->>PF: 3. Sends SAML AuthnRequest to PingFederate SSO URL
    PF-->>PF: 4. Verifies AuthnRequest signature <br>(using SF's public Request Signing Certificate)
    PF-->>"User/Browser": 5. Prompts for authentication (if no active session)
    "User/Browser"->>PF: 6. Submits credentials
    PF-->>PF: 7. Authenticates user
    PF-->>PF: 8. Generates SAML Assertion
    note right of PF: Assertion is signed with PF's IdP Signing Certificate. <br> Optionally encrypted with SF's Assertion Decryption Certificate.
    PF->>"User/Browser": 9. Sends SAML Assertion (via HTTP POST to SF ACS URL)
    "User/Browser"->>SF: 10. Submits SAML Assertion to Salesforce ACS URL
    SF-->>SF: 11. Verifies Assertion signature (using PF's public IdP Signing Certificate)
    SF-->>SF: 12. Decrypts Assertion if encrypted <br>(using SF's private Assertion Decryption Certificate)
    SF-->>SF: 13. Processes Assertion, identifies user, creates session
    SF->>"User/Browser": 14. Grants access to Salesforce resource
```

### **IdP-Initiated SSO Flow**

This flow starts when the user initiates login from PingFederate or a PingFederate-linked portal.

```mermaid
sequenceDiagram
    actor "User/Browser"
    participant PF as PingFederate (IdP)
    participant SF as Salesforce (SP)

    "User/Browser"->>PF: 1. Accesses PingFederate IdP-initiated SSO URL for Salesforce <br>(e.g., /idp/startSSO.ping?PartnerSpId=Salesforce_Entity_ID)
    PF-->>"User/Browser": 2. Prompts for authentication (if no active session)
    "User/Browser"->>PF: 3. Submits credentials (if prompted)
    PF-->>PF: 4. Authenticates user (if credentials submitted or session exists)
    PF-->>PF: 5. Generates SAML Assertion
    note right of PF: (signed with PF's IdP Signing Certificate. <br>Optionally encrypted with SF's Assertion Decryption Certificate)
    PF->>"User/Browser": 6. Sends SAML Assertion (via HTTP POST to SF ACS URL)
    "User/Browser"->>SF: 7. Submits SAML Assertion to Salesforce ACS URL
    SF-->>SF: 8. Verifies Assertion signature <br>(using PF's public IdP Signing Certificate)
    SF-->>SF: 9. Decrypts Assertion if encrypted <br>(using SF's private Assertion Decryption Certificate)
    SF-->>SF: 10. Processes Assertion, identifies user, creates session
    SF->>"User/Browser": 11. Grants access to Salesforce
```

These diagrams should give you a clear visual representation of the SAML SSO processes.

### SP-Initiated SSO Flow (Salesforce to PingFederate)

Here's a breakdown of all steps, with new or clarified details for those not fully covered previously:

* **Step 1: User/Browser \-\>\> SF: Attempts to access Salesforce resource**  
  * The user tries to open a Salesforce page or resource directly in their web browser.  
* **Step 2: SF \-\>\> User/Browser: Redirects to PingFederate (IdP) with SAML AuthnRequest**  
  *  Salesforce, realizing the user isn't logged in, creates a SAML Authentication Request (AuthnRequest). It then sends a redirect instruction to the user's browser, telling it to go to PingFederate's SSO URL with this AuthnRequest. The AuthnRequest may be signed by Salesforce.  
* **Step 3: User/Browser \-\>\> PF: Sends SAML AuthnRequest to PingFederate SSO URL**  
  * The user's browser follows the redirect from Salesforce and sends the SAML AuthnRequest to the specified PingFederate Single Sign-On (SSO) endpoint.  
* **Step 4: PF \--\>\> PF: Verifies AuthnRequest signature**  
  * PingFederate receives the AuthnRequest. If it's signed, PingFederate verifies the signature using Salesforce's public request signing certificate to ensure it came from a trusted Service Provider.  
* **Step 5: PF \--\>\> User/Browser: Prompts for authentication (if no active session)**  
  * If the user doesn't already have an active session with PingFederate, PingFederate presents a login page (or another authentication challenge) to the user's browser.  
* **Step 6: User/Browser \-\>\> PF: Submits credentials**  
  * The user enters their credentials (e.g., username and password) into the PingFederate login page.  
* **Step 7: PF \--\>\> PF: Authenticates user**  
  * PingFederate validates the credentials submitted by the user against its configured identity store (e.g., Active Directory, LDAP).  
* **Step 8: PF \--\>\> PF: Generates SAML Assertion**  
  * Upon successful authentication, PingFederate creates a SAML Assertion. This assertion contains information about the user's identity and authentication status. It is digitally signed by PingFederate's IdP signing certificate. It may also be encrypted for Salesforce.  
* **Step 9: PF \-\>\> User/Browser: Sends SAML Assertion (via HTTP POST to SF ACS URL)**  
  * PingFederate sends the SAML Assertion back to the user's browser. This is typically done via an HTTP POST method, instructing the browser to automatically submit the assertion to Salesforce's Assertion Consumer Service (ACS) URL.  
* **Step 10: User/Browser \-\>\> SF: Submits SAML Assertion to Salesforce ACS URL**  
  * The user's browser automatically POSTs the SAML Assertion received from PingFederate to the Salesforce ACS endpoint.  
* **Step 11: SF \--\>\> SF: Verifies Assertion signature**  
  * Salesforce receives the SAML Assertion and verifies its digital signature using PingFederate's public IdP signing certificate. This confirms the assertion's integrity and authenticity.  
* **Step 12: SF \--\>\> SF: Decrypts Assertion if encrypted**  
  * If the SAML Assertion was encrypted by PingFederate (using Salesforce's public assertion decryption certificate), Salesforce decrypts it using its corresponding private key.  
* **Step 13: SF \--\>\> SF: Processes Assertion, identifies user, creates session**  
  * Salesforce parses the SAML Assertion, extracts the user's identity (e.g., username or Federation ID), maps it to a Salesforce user, and establishes a session for that user.  
* **Step 14: SF \-\>\> User/Browser: Grants access to Salesforce resource**  
  * With a valid session established, Salesforce grants the user access to the initially requested resource or their Salesforce home page.

### IdP-Initiated SSO Flow (PingFederate to Salesforce)

Here's a breakdown of all steps, with new details for those not fully covered previously:

* **Step 1: User/Browser \-\>\> PF: Accesses PingFederate IdP-initiated SSO URL for Salesforce**  
  * The user clicks a link or bookmark that directly initiates login via PingFederate for the Salesforce application (e.g., from an internal portal).  
* **Step 2: PF \--\>\> User/Browser: Prompts for authentication (if no active session)**  
  * If the user doesn't have an active session with PingFederate, PingFederate presents a login page or another authentication challenge.  
* **Step 3: User/Browser \-\>\> PF: Submits credentials (if prompted)**  
  * If prompted, the user enters their credentials into PingFederate's login form.  
* **Step 4: PF \--\>\> PF: Authenticates user (if credentials submitted or session exists)**  
  * PingFederate validates the submitted credentials or confirms an existing session.  
* **Step 5: PF \--\>\> PF: Generates SAML Assertion**  
  * Upon successful authentication (or if a session already exists), PingFederate creates a SAML Assertion for Salesforce, signed with its IdP signing certificate and optionally encrypted.  
* **Step 6: PF \-\>\> User/Browser: Sends SAML Assertion (via HTTP POST to SF ACS URL)**  
  * PingFederate sends the SAML Assertion to the user's browser, typically via an auto-submitting HTTP POST form directed at Salesforce's ACS URL.  
* **Step 7: User/Browser \-\>\> SF: Submits SAML Assertion to Salesforce ACS URL**  
  * The user's browser automatically POSTs the SAML Assertion received from PingFederate to the Salesforce ACS endpoint.  
* **Step 8: SF \--\>\> SF: Verifies Assertion signature**  
  * Salesforce receives the SAML Assertion and verifies its digital signature using PingFederate's public IdP signing certificate.  
* **Step 9: SF \--\>\> SF: Decrypts Assertion if encrypted**  
  * If the SAML Assertion was encrypted, Salesforce decrypts it using its private assertion decryption key.  
* **Step 10: SF \--\>\> SF: Processes Assertion, identifies user, creates session**  
  * Salesforce parses the SAML Assertion, identifies the user, maps them to a Salesforce user account, and establishes a new session.  
* **Step 11: SF \-\>\> User/Browser: Grants access to Salesforce**  
  * Salesforce grants the user access, typically redirecting them to their default landing page within Salesforce.
 
Okay, this is a great request for adding more practical detail\! Here's a breakdown of each step for both SAML SSO flows, including examples of URL formats and the typical locations and uses of the certificates involved.

**Important Notes Before We Start:**

* **Placeholders:** URLs will use placeholders like https://\<your-salesforce-domain\>.my.salesforce.com, https://\<pf-idp-hostname.example.com\>, and PartnerSpId=\<Salesforce\_Entity\_ID\>. You'll need to replace these with your actual configured values.  
* **HTTPS/TLS Certificates:** All communication should be over HTTPS. This means both Salesforce and PingFederate will have their own HTTPS/TLS certificates to secure the browser-to-server connections. These are standard web server certificates and are distinct from the SAML-specific signing and encryption certificates.  
  * **Location:** Server-side (Salesforce or PingFederate). Private keys are on the server; public keys (and their chains) are presented to the browser for validation.  
* **SAML Certificates:**  
  * **Signing Certificates:** Used to create digital signatures on SAML messages (AuthnRequest, Assertion) to ensure integrity and authenticity.  
    * Private Key: Held by the signing entity (e.g., Salesforce for AuthnRequest, PingFederate for Assertion). Stored securely on the respective platform.  
    * Public Key: Shared with the verifying entity. Stored in the verifying entity's trust configuration (e.g., PingFederate has Salesforce's public key; Salesforce has PingFederate's public key).  
  * **Encryption Certificates:** Used to encrypt SAML Assertions (or parts of them) to ensure confidentiality.  
    * Public Key: Held by the encrypting entity (e.g., PingFederate uses Salesforce's public encryption key).  
    * Private Key: Held by the decrypting entity (e.g., Salesforce uses its private decryption key). Stored securely on the platform.  
* **Metadata Exchange:** The most common and recommended way to exchange public certificates, entity IDs, and endpoint URLs is by exchanging SAML 2.0 metadata files between Salesforce and PingFederate.

### ---

# SP-Initiated SSO Flow (Salesforce to PingFederate)

This flow starts when the user tries to access Salesforce directly.

**Step 1: User/Browser \-\>\> SF: Attempts to access Salesforce resource**

* **URL Example:**  
  * https://\<your-salesforce-domain\>.my.salesforce.com/ (Salesforce org)  
  * https://\<your-salesforce-domain\>.lightning.force.com/lightning/page/home (Lightning Experience)  
  * https://\<your-community-domain\>.com/s/ (Experience Cloud site)  
* **Certificates:**  
  * **Salesforce HTTPS/TLS Certificate:** Secures the connection between the browser and Salesforce.

**Step 2: SF \-\>\> User/Browser: Redirects to PingFederate (IdP) with SAML AuthnRequest**

* **URL Format (Browser is redirected to PingFederate's SSO endpoint using HTTP-Redirect Binding):** https://\<pf-idp-hostname.example.com\>/idp/SSO.saml2 ?SAMLRequest=\<Base64URLEncoded\_Deflated\_AuthnRequest\> \&RelayState=\<URLEncoded\_Opaque\_Value\_or\_TargetURL\> \&SigAlg=\<URLEncoded\_SignatureAlgorithmURI\> (e.g., http%3A%2F%2Fwww.w3.org%2F2001%2F04%2Fxmldsig-more%23rsa-sha256) \&Signature=\<Base64URLEncoded\_Signature\>  
* **Certificates:**  
  * **Salesforce Request Signing Certificate (Private Key):** Used by Salesforce to sign the AuthnRequest.  
    * **Location:** Managed in Salesforce under "Certificate and Key Management." The specific certificate is selected in the Salesforce "Single Sign-On Settings" for the PingFederate configuration.  
  * **Salesforce HTTPS/TLS Certificate:** Secures the redirect response to the browser.

**Step 3: User/Browser \-\>\> PF: Sends SAML AuthnRequest to PingFederate SSO URL**

* **URL Example (Browser navigates to the URL from Step 2):** https://\<pf-idp-hostname.example.com\>/idp/SSO.saml2?SAMLRequest=...\&RelayState=...\&SigAlg=...\&Signature=...  
* **Certificates:**  
  * **PingFederate HTTPS/TLS Certificate:** Secures the connection between the browser and PingFederate.

**Step 4: PF \--\>\> PF: Verifies AuthnRequest signature**

* **URL Example:** N/A (internal PingFederate process).  
* **Certificates:**  
  * **Salesforce Request Signing Certificate (Public Key):** Used by PingFederate to verify the signature of the AuthnRequest.  
    * **Location:** Stored in PingFederate's configuration for the Salesforce SP Connection. Typically uploaded from Salesforce metadata or as a separate certificate file (.crt, .cer).

**Step 5: PF \--\>\> User/Browser: Prompts for authentication (if no active session)**

* **URL Example (PingFederate's login page, path can vary):** https://\<pf-idp-hostname.example.com\>/pf/adapter/UsernamePasswordLogin.form https://\<pf-idp-hostname.example.com\>/idp/resumeSAML2Flow.ping (intermediate PingFederate flow URL)  
* **Certificates:**  
  * **PingFederate HTTPS/TLS Certificate:** Secures the login page served to the browser.

**Step 6: User/Browser \-\>\> PF: Submits credentials**

* **URL Example (Form POST to PingFederate's authentication handler):** https://\<pf-idp-hostname.example.com\>/pf/adapter/UsernamePasswordLogin.form (or the action URL of the login form)  
* **Certificates:**  
  * **PingFederate HTTPS/TLS Certificate:** Secures the submission of credentials from the browser.

**Step 7: PF \--\>\> PF: Authenticates user**

* **URL Example:** N/A (internal PingFederate process). May involve internal calls to identity stores like:  
  * LDAP: ldap://\<ldap-server\>:\<port\> or ldaps://\<ldap-server\>:\<port\>  
  * Database: JDBC connection strings.  
* **Certificates:**  
  * If PingFederate connects to backend stores via secure protocols (e.g., LDAPS), PingFederate's truststore might need the CA certificate for the LDAP server's certificate, or client certificates if mutual authentication is used.

**Step 8: PF \--\>\> PF: Generates SAML Assertion**

* **URL Example:** N/A (internal PingFederate process).  
* **Certificates:**  
  * **PingFederate IdP Signing Certificate (Private Key):** Used by PingFederate to sign the SAML Assertion.  
    * **Location:** Managed within PingFederate's certificate management (e.g., "Signing & Decryption Keys"). Selected in the IdP Adapter or SP Connection settings.  
  * **(Optional) Salesforce Assertion Encryption Certificate (Public Key):** If assertion encryption is enabled, PingFederate uses this to encrypt the assertion.  
    * **Location:** Stored in PingFederate's configuration for the Salesforce SP Connection. Obtained from Salesforce metadata (usually labeled as "encryption" usage).

**Step 9: PF \-\>\> User/Browser: Sends SAML Assertion (via HTTP POST to SF ACS URL)**

* The browser is on a PingFederate page (e.g., https://\<pf-idp-hostname.example.com\>/idp/SAML2AssertionConsumer) that contains an auto-submitting HTML form.  
* **Form Action URL (Salesforce ACS URL):** https://\<your-salesforce-domain\>.my.salesforce.com/saml/SSO (This is the Salesforce default. It can vary for communities or custom configurations).  
* **Form Parameters:** SAMLResponse (Base64 encoded SAML Assertion), RelayState.  
* **Certificates:**  
  * **PingFederate HTTPS/TLS Certificate:** Secures the PingFederate page serving the auto-POST form.  
  * The SAML Assertion within the SAMLResponse is already signed (and possibly encrypted) using certificates from Step 8\.

**Step 10: User/Browser \-\>\> SF: Submits SAML Assertion to Salesforce ACS URL**

* **URL Example (Salesforce Assertion Consumer Service URL):**  
  * Default: **https://\<your-salesforce-domain\>.my.salesforce.com/saml/SSO**  
  * Communities/Portals: **https://\<your-community-domain\>/saml/SSO** or **ttps://\<your-salesforce-domain\>.my.salesforce.com/\<community\_url\_path\>/saml/SSO**  
  * This URL is specified in Salesforce's SAML configuration and its metadata.  
* **Certificates:**  
  * **Salesforce HTTPS/TLS Certificate:** Secures the connection for the browser POSTing the assertion to Salesforce.

**Step 11: SF \--\>\> SF: Verifies Assertion signature**

* **URL Example:** N/A (internal Salesforce process).  
* **Certificates:**  
  * **PingFederate IdP Signing Certificate (Public Key):** Used by Salesforce to verify the signature of the SAML Assertion.  
    * **Location:** Uploaded to Salesforce in the "Single Sign-On Settings" for the PingFederate IdP configuration. Obtained from PingFederate metadata or as a certificate file.

**Step 12: SF \--\>\> SF: Decrypts Assertion if encrypted**

* **URL Example:** N/A (internal Salesforce process).  
* **Certificates:**  
  * **Salesforce Assertion Encryption Certificate (Private Key):** Used by Salesforce to decrypt the encrypted assertion.  
    * **Location:** Managed in Salesforce under "Certificate and Key Management." This is the private key corresponding to the public encryption certificate PingFederate used in Step 8\.

**Step 13: SF \--\>\> SF: Processes Assertion, identifies user, creates session**

* **URL Example:** N/A (internal Salesforce process).  
* **Certificates:** None directly involved in this processing logic, but rely on successful prior certificate operations.

**Step 14: SF \-\>\> User/Browser: Grants access to Salesforce resource**

* **URL Example (Target resource or default landing page):**  
  * RelayState URL from Step 2 if provided.  
  * https://\<your-salesforce-domain\>.lightning.force.com/  
* **Certificates:**  
  * **Salesforce HTTPS/TLS Certificate:** Secures the Salesforce page served to the browser.

### ---

# IdP-Initiated SSO Flow (PingFederate to Salesforce)

This flow starts when the user initiates login from PingFederate or a PingFederate-linked portal.

**Step 1: User/Browser \-\>\> PF: Accesses PingFederate IdP-initiated SSO URL for Salesforce**

* **URL Example:** https://\<pf-idp-hostname.example.com\>/idp/startSSO.ping?PartnerSpId=\<Salesforce\_Entity\_ID\_for\_PF\>  
  * \<Salesforce\_Entity\_ID\_for\_PF\> is the identifier for the Salesforce SP connection configured in PingFederate (e.g., https://\<your-salesforce-domain\>.my.salesforce.com or a custom URN).  
* **Certificates:**  
  * **PingFederate HTTPS/TLS Certificate:** Secures the connection.

**Step 2: PF \--\>\> User/Browser: Prompts for authentication (if no active session)**

* **URL Example (PingFederate's login page):** https://\<pf-idp-hostname.example.com\>/pf/adapter/UsernamePasswordLogin.form  
* **Certificates:**  
  * **PingFederate HTTPS/TLS Certificate:** Secures the login page.

**Step 3: User/Browser \-\>\> PF: Submits credentials (if prompted)**

* **URL Example (POST to PingFederate's authentication handler):** https://\<pf-idp-hostname.example.com\>/pf/adapter/UsernamePasswordLogin.form  
* **Certificates:**  
  * **PingFederate HTTPS/TLS Certificate:** Secures credential submission.

**Step 4: PF \--\>\> PF: Authenticates user (if credentials submitted or session exists)**

* **URL Example:** N/A (internal PingFederate process).  
* **Certificates:** (Same as SP-Initiated Step 7\) Potential use of certificates for secure backend connections.

**Step 5: PF \--\>\> PF: Generates SAML Assertion**

* **URL Example:** N/A (internal PingFederate process).  
* **Certificates:**  
  * **PingFederate IdP Signing Certificate (Private Key):** (Same as SP-Initiated Step 8\) To sign the assertion. Location: PingFederate.  
  * **(Optional) Salesforce Assertion Encryption Certificate (Public Key):** (Same as SP-Initiated Step 8\) To encrypt the assertion. Location: PingFederate (obtained from Salesforce).

**Step 6: PF \-\>\> User/Browser: Sends SAML Assertion (via HTTP POST to SF ACS URL)**

* Similar to SP-Initiated Step 9\. Browser is on a PingFederate page with an auto-submitting form.  
* **Form Action URL (Salesforce ACS URL):** https://\<your-salesforce-domain\>.my.salesforce.com/saml/SSO (or the ACS URL configured in PingFederate for this Salesforce SP connection).  
* **Form Parameters:** SAMLResponse, RelayState (optional, can be configured in PingFederate for IdP-initiated to specify a landing page in Salesforce).  
* **Certificates:**  
  * **PingFederate HTTPS/TLS Certificate.** Assertion itself is already processed with SAML certs.

**Step 7: User/Browser \-\>\> SF: Submits SAML Assertion to Salesforce ACS URL**

* **URL Example (Salesforce ACS URL):** https://\<your-salesforce-domain\>.my.salesforce.com/saml/SSO  
* **Certificates:**  
  * **Salesforce HTTPS/TLS Certificate.**

**Step 8: SF \--\>\> SF: Verifies Assertion signature**

* **URL Example:** N/A (internal Salesforce process).  
* **Certificates:**  
  * **PingFederate IdP Signing Certificate (Public Key):** (Same as SP-Initiated Step 11\) To verify assertion signature. Location: Salesforce.

**Step 9: SF \--\>\> SF: Decrypts Assertion if encrypted**

* **URL Example:** N/A (internal Salesforce process).  
* **Certificates:**  
  * **Salesforce Assertion Encryption Certificate (Private Key):** (Same as SP-Initiated Step 12\) To decrypt the assertion. Location: Salesforce.

**Step 10: SF \--\>\> SF: Processes Assertion, identifies user, creates session**

* **URL Example:** N/A (internal Salesforce process).  
* **Certificates:** None for this specific sub-step.

**Step 11: SF \-\>\> User/Browser: Grants access to Salesforce**

* **URL Example (Default Salesforce landing page or IdP-initiated start URL specified by RelayState or IdP config):** https://\<your-salesforce-domain\>.lightning.force.com/  
* **Certificates:**  
  * **Salesforce HTTPS/TLS Certificate.**

This detailed breakdown should give you a much clearer picture of the URLs and certificate usage at each stage of the SAML SSO flows between Salesforce and PingFederate.

---

ou're asking a very good and precise question about where the HTTPS/TLS certificates for Salesforce itself are managed or viewable. Here's how it works:

**1\. For Standard Salesforce Domains (e.g., your-domain.my.salesforce.com, your-domain.lightning.force.com):**

* **Salesforce Manages These:** For the core domains that Salesforce provides and hosts for your organization (like your My Domain, Lightning URLs, etc.), Salesforce automatically provisions and manages the HTTPS/TLS certificates. These certificates ensure that the connection between your users' browsers and the Salesforce servers is encrypted.  
* **Not Directly Viewable/Manageable in Setup:** You generally do **not** see or manage these specific HTTPS/TLS certificates directly within the Salesforce Setup UI in the same way you manage SAML signing/encryption certificates or certificates for custom domains (see next point). Salesforce handles their lifecycle (issuance, renewal, etc.) for these standard domains.  
* **How to "See" Them:** You can always view the details of the active HTTPS/TLS certificate for any Salesforce page (or any website) by using your web browser's security features. Typically, you click the padlock icon in the address bar next to the URL, which will allow you to inspect the certificate details (issuer, validity period, subject, etc.).

**2\. For Custom Domains Pointing to Salesforce (e.g., www.yourcompany.com serving an Experience Cloud site or your org):**

* **Customer Managed or Salesforce-Managed Option:** If you have configured a custom domain to serve your Salesforce org or Experience Cloud sites, then you *are* involved in the HTTPS/TLS certificate management for that specific custom domain.  
* **Viewable/Manageable in Setup:** For these custom domains, you would typically manage the HTTPS/TLS certificate options within Salesforce Setup:  
  * Go to **Setup \> Domain Management \> Domains**.  
  * When you add or edit a custom domain, Salesforce provides options for how HTTPS is served.1 This usually includes:  
    * **Salesforce serves the domain over HTTPS using a Salesforce-managed certificate:** Salesforce provisions and renews a certificate for your custom domain.  
    * Using your own CA-signed certificate: You can upload your own HTTPS/TLS certificate (and its private key and any intermediates) that you've obtained from a Certificate Authority (CA).2 These uploaded certificates would be managed under **Setup \> Security \> Certificate and Key Management**.  
* So, in the context of custom domains you bring to Salesforce, you *can* see and manage the associated HTTPS/TLS certificate (or the option for Salesforce to manage it) within the Setup interface.

**In Summary:**

* **Standard Salesforce Domains:** Salesforce handles the HTTPS/TLS certificates. You view them via your browser, not directly in Setup as a manageable item.  
* **Custom Domains for Salesforce:** You have options to manage the HTTPS/TLS certificates for these within Salesforce Setup, either by uploading your own or using a Salesforce-managed one.

You are correct in your initial understanding that these HTTPS/TLS certificates are distinct from the SAML-specific signing and encryption certificates, which are explicitly managed in "Certificate and Key Management" (for certificates Salesforce uses as an SP or IdP) and "Single Sign-On Settings" (for uploading an external IdP's public SAML signing certificate).

Yes, that's a correct understanding.

When PingFederate interacts with Salesforce endpoints (like the Assertion Consumer Service \- ACS URL) over HTTPS, the security of that HTTPS connection itself is handled by TLS/SSL. Salesforce's servers will present their HTTPS/TLS certificate to establish this secure connection.

**Here's why those HTTPS/TLS certificates are configured and trusted independently of the SAML-specific settings within PingFederate for the Salesforce Service Provider (SP) connection:**

1. **Transport Layer vs. Application Layer Security:**  
   * HTTPS/TLS Certificates: These secure the transport layer (the pipe through which data flows).1 They ensure that the communication between the client (which could be the user's browser, or in some less common SAML scenarios, PingFederate itself making a direct back-channel call) and the Salesforce server is encrypted and that the client is talking to the authentic Salesforce server.  
   * **SAML Certificates:** These secure the SAML messages themselves (the content flowing through the pipe). They are used for digital signatures (to verify the origin and integrity of SAML AuthnRequests or Assertions) and encryption (to ensure the confidentiality of the SAML Assertion content).2  
2. **Trust Establishment for HTTPS/TLS:**  
   * When PingFederate (or a browser directed by PingFederate) connects to a Salesforce HTTPS URL (e.g., https://\<your-salesforce-domain\>.my.salesforce.com/saml/SSO), Salesforce presents its server's HTTPS/TLS certificate.  
   * The client (browser or PingFederate's Java runtime environment \- JRE) verifies this certificate against its list of trusted Certificate Authorities (CAs). Salesforce uses publicly trusted CAs for its servers.  
   * This trust is generally pre-established in the browser's or JRE's default truststore (e.g., cacerts file in the JRE). There's usually no specific action needed *within PingFederate's SAML configuration for Salesforce* to trust Salesforce's standard HTTPS/TLS certificate because it's issued by a well-known CA.  
   * If Salesforce were using an HTTPS/TLS certificate from a private/internal CA (which is not the case for standard public Salesforce services), then PingFederate's JRE truststore would need to be updated to trust that private CA, but this is a general JRE/server configuration, not a SAML SP connection-specific setting.  
3. **SAML Configuration in PingFederate:**  
   * When you configure Salesforce as an SP in PingFederate, you are concerned with SAML protocol details:  
     * Salesforce's Entity ID.  
     * Salesforce's Assertion Consumer Service (ACS) URLs (these are HTTPS URLs, but PingFederate cares about the *endpoint address*, not managing the TLS cert for that address).  
     * The public certificate from Salesforce used for signing SAML AuthnRequests (if Salesforce is configured to sign them).  
     * The public certificate from Salesforce used for encrypting SAML Assertions sent by PingFederate (if encryption is configured).  
     * Attribute contract, authentication context, etc.

**In essence:**

PingFederate's SAML configuration for Salesforce deals with *what to put inside the SAML messages* and *where to send them at the application level*. The underlying trust for the *HTTPS connection to Salesforce's domain* relies on standard TLS/SSL procedures and trust anchors, which are generally independent of the specific SAML partnership settings in PingFederate.

So, while PingFederate needs to know Salesforce's ACS *URL* (which is an HTTPS URL), it doesn't typically require you to upload Salesforce's *HTTPS/TLS server certificate* into the SAML SP connection settings. It expects the JRE (and browsers) to handle that part.
