Note: this file is open to members of the standards community 

# Minimum Interoperable Profile for Authentication Context in Federated Protocols
##### P. Dingle, Microsoft, D. Olds, Broadcom, A. Parecki, Okta
To be submitted to: OpenID Foundation Connect WG

September 2023

Status:  Draft.

## Abstract
OpenID Connect 1.0 and SAML 2.0 are federated identity specifications with built-in mechanisms for negotiating the authentication-related security controls that are required before access can be granted, but the default treatment of requests and responses in each of those protocols differ significantly.  While SAML 2.0 requires strict processing of authentication context by default, OpenID Connect by default presumes “voluntary” processing. This specification profiles OpenID Connect to accomplish equivalent processing behavior to the default SAML 2.0 specification, adding examples and test criteria for both specifications to give implementers high confidence that any given set of controls can be reliably enforce without worries about protocol divergence. 

## Introduction
SAML 2.0 and OpenID Connect evolved decades apart from each other, and yet operate effectively side-by-side in many identity architectures today as mechanisms for secure introduction of end users across federated parties. Each protocol contains a mechanism for specifying the authentication security controls that must be in place during this process, and both protocols use the term ‘authentication context’ to describe the security controls each party agrees to. Authentication contexts are referred to by identifiers and these identifiers are referred to as ACRs, or authentication context class references.   

This document profiles OpenID Connect such that request parameters, response attributes and validation steps match the default operation of SAML 2.0. To meet this goal, the protocol defines a concept of an ACR Request and an ACR response as a protocol-agnostic concept, then points to the relevant specification text that implements each step.  The profile also defines the test criteria, error conditions, and additional validations that federated parties may choose to use to additionally identify risk and ensure in every possible way that the profile is adhered to.

The minimum interoperability profile is created with the following principles in mind:
* The profile must be compatible with the linked ratified core specifications. 
* The profile may add additional attribute definitions, scopes, or metadata parameters where those additional elements are extensible by design in each specification.
* The profile assumes breach and recommends pro-active additional validation to detect neglectful misconfiguration and/or abuse.  Absence of a claim or metadata element is never interpreted as intentional.
* Authentication Method and Authentication Method References (AMRs) are out of scope of this document.
* All error conditions are defined and are expected to be tested for accuracy by implementers.

Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].

In the .txt version of this document, values are quoted to indicate that they are to be taken literally. When using these values in protocol messages, the quotes MUST NOT be used as part of the value. In the HTML version of this document, values to be taken literally are indicated by the use of this fixed-width font.

## Terminology

 * __Analogous Term__: A single defined term used to refer to conceptually equivalent concepts in either SAML or OIDC. Analogous relationships are defined in the terminology section.
 * __Assertion__: A structured document that binds a set of claims to a subject identifier. Analogous to a SAML assertion, OIDC id_token. See [SAML§2], [OIDC§2]. 
 * __Authentication Context__:  A security control applying requirements to the manner in which a given subject is authenticated prior to issuance of a federated assertion.  In this specifications, authentication contexts are referred to by their class reference (also known as an ACR).
 * __ACR (Authentication Context Class Reference)__:   A unique identifier associated to a given authentication context, defined using the syntax and namespace matching the federated protocol in use. In this specification, an ACR and the authentication context control that it refers to are used interchangeably. 
 * __ACR Request__: the portion of a federated authentication request defining ACR-related requirements. 
 * __ACR Response__:  the portion of a returned federated assertion that are set as a direct result of an ACR Request.
 * __Claim__: A unit of descriptive information with a name and value. Analogous to attribute.
 * __End-User__: The human that the subject represents.
 * __Essential Claim__: "A claim specified by the Client as being necessary to ensure a smooth authorization experience for the specific task requested by the End-User". 
 * __Evidence__: Information used to corroborate or explain how a security control was executed.
 * __IDP (Identity Provider)__: An entity that determines the end user's authentication context and issues an assertion. Analogous to an OpenID Provider (OP). See [SAML §3.4] and [OIDC §1.2].
 * __RP (Relying Party)__: An entity that receives and validates an assertion. Used interchangeably with SP (Service Provider) and Client. See [SAML §3.4] and [OIDC §1.2]
 * __Security Control__:  A named and defined safeguard used to mitigate risk of misuse, fraud or attack.
 * __Subject, sub__: Identifier shared between IDP and RP, expected to be unique for the RP with respect to the IDP.
 * __Unsolicited Assertion__: an assertion arrives at an RP that is not the result of an authentication request.  This type of assertion occurs in [SAML2] and is the result of IDP-initiated federation.
 * __Voluntary Claim__: "Claim specified by the Client as being useful but not Essential for the specific task requested by the End-User". See [OIDC §1.2]

 ### Use of Analogous Terms
Where differing terms exist in each protocol, an analogous relation is defined in the terminology section above. In any case discussing over-the-wire protocol interactions, protocol-correct terms will be used. For the purposes of this document, the terms IDP, RP, assertion, and claim will be used throughout the document.

### SAML 2.0 Authentication Context Excluded Features
This minimum interoperabilty profile explicitly does not attempt to alter SAML 2.0 functionality, however there is a small set of SAML 2.0 functionality that is from the interoperability profile and should be avoided by implementers:
 * Comparison operators (`minimum`, `maximum`, `exist`)
 * Authentication Context Declaration References

## OpenID Connect Minimum Profile
Requirements in this section are normative and augment the existing normative requirements in [OIDC]. This profile defines five phases where ACR requirements are modified:

                                             0. End user
                                             │  access attempt
                                             ▼
      ┌─────────────────┐  1. ACR Metadata  ┌─────────────────┐
      │                 │ ◄─ ─ ─ ─ ─ ─ ─ ─► │                 │
      │    Identity     │                   │     Relying     │
      │    Provider     │                   │      Party      │
      │                 │  2. ACR Request   │                 │
      │ ┌─────────────┐ │ ◄──────────────── │                 │
      │ │3. ACR       │ │                   │                 │
      │ │   Evaluation│ │                   │                 │
      │ └─────────────┘ │ ────────────────► │ ┌─────────────┐ │
      │                 │  4. ACR Response  │ │5. ACR       │ │
      │                 │                   │ │   Validation│ │
      │                 │                   │ └─────────────┘ │
      └─────────────────┘                   └─────────────────┘

 1. __ACR Metadata__: An RP looks up ACR-related properties published by an IDP relating to profile adherence and claim support in advance of or at time of an individual federated interaction.  
 1. __ACR Request__: An RP generates an authentication request that contains parameters and values specifying one or more authentication contexts as an essential part of federated processing. 
 1. __ACR Evaluation__: An IDP evaluates an ACR request and determines whether an end user meets the criteria specified, performing any necessary interactions needed to bring an end-user into adherence, or determining that adherence is not possible.
 1. __ACR Return__: An IDP responds to a given ACR request with either an error message or a valid assertion containing ACR-compliant claims and claim values that matches the results of the ACR Evaluation.
 1. __ACR Validation__: An RP validates a received assertion to ensure the IDP has met all of the original requirements of the ACR Request and either allows access or returns an error to the user.

### ACR Request
The ACR Request is an [OIDC] authentication request that specifies one or more essential authentication contexts.  This profile defines two different options to specify authentication contexts, one that uses a default [OIDC] mechanism and a second that redefines an OIDC parameter and requires explicit metadata confirmation. 

One of the following methods MUST be implemented. If the authentication request includes both the acr_values parameter and a claims parameter and the `acr` claim is a defined value the IDP MUST return an error (note this extends guidance in [OIDC] §5.5.1.1 . 

#### Essential ACR via Claims Request Parameter
The RP MUST include the claims parameter defined in [OIDC] §5.5 in the authentication request.

Within the JSON object contained by the claims parameter (see[OIDC] §5.5):
* The `id_token` JSON object is REQUIRED.
* The `acr` element of the `id_token` object is REQUIRED.
* The `essential` element of the acr object MUST be set to true.
* The values element of the acr object MUST be present and contain at least one value.
Note: the order of the values listed in the request are significant.

An example of a valid ACR request in the form of a claims request parameter follows: 

    {
      "id_token": {
        "acr": {
          "essential": true,
          "values": ["inherence","possession"]
        }
      }
    }

#### Essential ACR via Redefined acr_values Parameter
If ACR Metadata fetched by the RP contains the metadata XX, the RP MUST consider the `acr_values` parameter defined in [OIDC] §3.1.2.1 as specifying essential acr values. 

An example of a valid ACR request in the form of a claims request parameter follows: 

      [phr,mfa,pwd]


 
### ACR Evaluation
IDPs MUST test the authentication request for the simultaneous presence of the `acr_values` request parameter and an essential `acr` claim in the claims parameter. If both exist, the IDP MUST return an error.  

IDPs performing anomaly detection SHOULD be checking to see that federated connections consistently use this profile.   Authentication requests that usually contain an essential acr claim in the claims parameter but suddenly arrive with no acr requirements or voluntary acr requirements should be treated as suspicious. 

If an IDP chooses to populate multiple authentication contexts for consumption by the RP, the following requirements apply:
* The IDP MUST publish support for the `acrs_supported` metadata element as part of IDP discovery.
* The IDP MUST prevent the  acrs attribute from being directly populated by any other entity or for any other purpose at any time.   

As stated in  [OIDC] §5.5.1.1:
> If the acr Claim is requested as an Essential Claim for the ID Token with a values parameter requesting specific Authentication Context Class Reference values and the implementation supports the claims parameter, the Authorization Server MUST return an acr Claim Value that matches one of the requested values. The Authorization Server MAY ask the End-User to re-authenticate with additional factors to meet this requirement. If this is an Essential Claim and the requirement cannot be met, then the Authorization Server MUST treat that outcome as a failed authentication attempt.

### ACR Return
In the case that an essential acr claim was included in the authentication request, the IDP MUST ensure the returned assertion conforms with [OIDC] and additionally has the following properties:
* The acr attribute MUST contain a single value that represents the first matching acr value listed in the acr element of the claims parameter.
* The acrs attribute MAY contain a multi-valued array of strings that represents all matching acr values from the claims parameter, plus other unsolicited acr values.
* If the acrs attribute is supported, the IDP MUST do the following:
  * Prevent any other party from populating an attribute called “acrs”
  * Always populate the acrs attribute when an essential acr claim is requested. 

An example of a valid ACR return is listed below:

    {
      "iss": "https://example.com/tenantb",
      "sub": "Ay782bbtaQ",
      "aud": "6cb045aef3",
      "exp": 1536361411,
      "name": "Joanna Smith",
      "nonce": "12353",
      "acr": "inherence",
      "acrs": ["inherence", "possession", "fips140"],
      "amr": ["phrh", "Yubikey5c"]
    }

 
### ACR Validation
The RP MUST discard the assertion and refuse to grant access if the following is true:
* The value in the acr attribute contains a string that does not match an originally requested value.
* The returned assertion does not contain an acr attribute.
* The IDP advertises support for acrs via the `acrs_supported` metadata attribute, but the acrs attribute is not present.

### ACR Metadata and Discovery
This profile defines one new [OIDC] metadata element; this element enables a payload claim to be trusted as a protected security attribute.  Exact mechanisms for discovery of the metadata elements are not specified by this profile.

`acrs_supported`. OPTIONAL.   A Boolean attribute stating whether the multi-valued acrs claim is available as a trusted security attribute.  Default value is FALSE.

#### Profile Operation: Strict Authentication Context
To guarantee that both participants in a federated connection are operating by strict processing rules, the RP MUST receive a guarantee that the IDP knows that strict processing is required.  This guarantee may happen either in the assertion or in the IDP’s metadata.  Each case is described in the appropriate section.

If an RP cannot receive a guarantee that the IDP has followed strict processing rules, the RP SHOULD reject the assertion.

## SAML2 Minimum Interoperability Profile
Requirements in this section are crafted not to presume that both parties in the federation have implemented this profile.   These requirements standardize the use of additional security attributes, but those attributes can only be trusted when the IDP makes an explicit guarantee of support. 

* AuthnContexts.  OPTIONAL.  Multi-valued list of 
* AuthnMethod. OPTIONAL. Multi-valued list of

### ACR Request
As per [SAML2] §3.3.2.2.1, RPs populate the RequestedAuthnContext element of the AuthenticationRequest object with a prioritized list of valid ACR values. 
Additional normative requirements for RPs include the following:
 * The RequestedAuthnContext element SHOULD NOT contain a saml:AuthnContextDeclRef element
 * The AuthenticationRequest element SHOULD NOT contain a comparison parameter set to a value other than exact.
 * Instead of using comparison operators, the RP SHOULD explicitly list every member of a set and order that set such that the preferences listed deliver the same results.    

#### Examples of Valid ACR Requests (within a SAML authentication request)
Require telekinesis if available, but finger snap if telekinesis is not available.
![image](https://github.com/pamelatech/ACRminprofile/assets/2591320/e2a51126-9b14-4895-b9d0-d46c8a298fd0)

 
Require a minimum Assurance Level of Three (out of Five).
![image](https://github.com/pamelatech/ACRminprofile/assets/2591320/d8265018-94de-4058-ab1c-aa426c9f1892)

 
Require Better than Assurance Level Four (out of Five).
![image](https://github.com/pamelatech/ACRminprofile/assets/2591320/15b769ca-2887-4460-a992-c9d953b3ed61)

 
Require a Maximum of Assurance Level Two (out of Five).
![image](https://github.com/pamelatech/ACRminprofile/assets/2591320/f3c73ef2-56d1-44af-8053-1b4d594b52bb)

 
### ACR Evaluation
The IDP is under the following obligations if they receive an authentication request containing a RequestedAuthnContext element:

* To only return an assertion if at least one of the listed authentication contexts can be matched (see [SAML2] §3.3.2.2 which says “If the <RequestedAuthnContext> element is present in the query, at least one element in the set of returned assertions MUST contain an element that satisfies the element in the query (see Section 3.3.2.2.1). It is OPTIONAL for the complete set of all such matching assertions to be returned in the response.”
* To evaluate the set of requested ACRs in priority order (see [SAML2] §3.3.2.2.1 which says: “The set of supplied references MUST be evaluated as an ordered set, where the first element is the most preferred authentication context class or declaration. If none of the specified classes or declarations can be satisfied in accordance with the rules below, then the responder MUST return a message with a second-level of urn:oasis:names:tc:SAML:2.0:status:NoAuthnContext.”
* To populate the \<AuthnContext> claim. 
 
This means that the IDP:
* Cannot return an assertion unless an authentication context exists or is established that matches the requirements identified by at least one authentication context  :
* MUST Establish a sufficient authentication context if none exists
* MUST populate the Authentication Context 
* “The responder MUST return a message with a second-level of urn:oasis:names:tc:SAML:2.0:status:NoAuthnContext” – §3.3.2.2.1 [SAML2]

### ACR Return

#### Single-Valued Authentication Context

If the ACR Evaluation is successful, the IDP MUST:
* Include an AuthnStatement element in any returned assertions that contains an AuthnContext element containing the AuthnContextClassRef that matched the end user’s current authentication experience.  See [SAML2] §2.7.2.2.
* The AuthnContext attribute is defined to be a single value.  If exact comparison matching is used, This single value SHOULD match one and only one of the ACRs in the requested ACR list.
* ![image](https://github.com/pamelatech/ACRminprofile/assets/2591320/e23755d5-a6df-46cb-85da-406c4e7d2f7f)

 
#### Optional: Multi-Valued Authentication Context

An IDP may optionally choose to return an extension element in the AttributeStatement of the assertion called AuthnContexts.   The AuthnContexts attribute MUST be a multi-valued attribute and MUST NOT be included if there is only one value listed. <TODO: define exact schema>. If an IDP includes an AuthnContexts attribute in the AttributeStatement, it MUST also:

* Ensure that the value in the AuthnContext statement within the AuthnStatement is also in AuthnContexts in the AttributeStatement.
* Ensure that no entity can set the AuthnContexts attribute except the IDP infrastructure itself
* Include a metadata statement that can be independently verified by an RP that states the AuthnContexts attribute is a protected attribute.

If a Relying Party receives an assertion containing an AuthnContexts attribute in the AttributeStatement when accompanying metadata statement is made by the IDP, the Relying Party MUST assume the attribute is malicious and reject the assertion.

#### ACR Validation
Once an RP receives an assertion, the RP MUST:
* Check for presence of an AuthnContext element within the AuthnStatement object.  
    * If the AuthnContext  claim is not present but the RP sent an ACR Request, the RP MUST reject the assertion.
    * If the assertion had no preceding ACR Request, the RP MAY accept the assertion only if all validation rules in [SAML2] and this profile are met.
* Check that the value set in the AuthnContext claim was one of the values originally provided in the ACR Request.  If the value set in the AuthnContext attribute is not one of the values originally given in the ACR Request the RP MUST reject the assertion.
* Check for the presence of an AuthnContexts attribute within the AttributeStatement. If the AuthnContexts attribute is present but does not contain one of the originally requested ACRs, or if the RP cannot guarantee that the IDP considers the AuthnContexts attribute to be a protected attribute, the RP MUST reject the assertion.

#### ACR Metadata and Discovery

This profile defines two new [SAML2] metadata elements; these elements enable payload claims to be trusted as protected security attributes.  Exact mechanisms for discovery of the metadata elements are not specified by this profile.

`SupportedAuthnContexts`. OPTIONAL.   A Boolean attribute stating whether the multi-valued acrs claim is available as a trusted security attribute.  Default value is FALSE. Implementers are warned that if SupportedAuthenticationMethods is FALSE or absent for any given IDP, any AuthenticationMethod claims found in assertions should be considered as malicious.

`SupportedAuthenticationMethods`. OPTIONAL.   A Boolean attribute stating whether the multi-valued attribute within the AttributeStatement element is a protected security attribute.  Default value is FALSE.  Implementers are warned that if SupportedAuthenticationMethods is FALSE or absent, any AuthenticationMethod claims found in assertions should be considered as malicious.

## Cross-Protocol Use Cases & Examples   

## Security Considerations
An example of security issue is: password is used in AAD for first factor, then the same password is used in DUO. If the password gets leaked the user is not secure and this is not classified as a multi factor as you have used the same method and password twice.

### ACR Replacement Attack
An adversary replaces the requested ACR list in the hopes that the RP is not validating responses, with the intent of forcing a user authenticated at a lower level to gain access to resources at the RP that require a higher level.

### Multiple ACR Claim Text Overflow

Attacker adds an acr_values parameter with less stringent controls to the end of an existing request hoping to confuse the IDP and satisfy less stringent requirements (RP also has to not notice the non-matching ACR).

## Error States
Eg:  if the RP specifies 3 ACRs but the IDP doesn’t recognize 2 of them what happens?

### Summary of Changes
| Specification | Type of Change | Examples of extension |
|---------------|----------------|------------------------|
| SAML 2.0      |  Core | None |
| OpenID Connect | core | None |  
| OpenID Connect Discovery 1.0 | Definition of new supported attributes  
| SAML 2.0 Attribute Extension | Definition of new supported attributes | Example from SimpleSAMLphp

 
# References
[OIDC] OpenID Connect Core 1.0 incorporating errata set 2. https://openid.net/specs/openid-connect-core-1_0.html.

[OIDCUAR]  Lodderstedt, T., "OpenID Connect Core Error Code unmet_authentication_requirements", 8 May 2019, <https://openid.net/specs/openid-connect-unmet-authentication-requirements-1_0.html>.

[OIDCDISC] Sakimura, N., Bradley, J., Jones, M., and E. Jay, "OpenID Connect Discovery 1.0 incorporating errata set 1", 8 November 2014, <https://openid.net/specs/openid-connect-discovery-1_0.html>.

[SAML2]  S. Cantor et al. Assertions and Protocols for the OASIS Security Assertion Markup Language (SAML) V2.0. OASIS SSTC, March 2005. https://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf. 

[SAMLAuthnCxt]  J. Kemp et al. Authentication Context for the OASIS Security Assertion Markup Language (SAML) V2.0. OASIS SSTC, March 2005. https://docs.oasis-open.org/security/saml/v2.0/saml-authn-context-2.0-os.pdf 

[SAMLMetadata] 

[RFC2119] RFC 2119


# Notes from IIW
Step-up Auth spec depends on OIDC acr being a voluntary claim

