Note: this file is open to members of the standards community 

# Minimum Interoperable Profile for Authentication Context in Federated Protocols
##### P. Dingle, Microsoft
To be submitted to: OpenID Foundation Connect WG
September 2023
Status:  Text is pseudo normative, some sections still in bullet/paragraph form. Error conditions not yet discussed.

## Abstract
OpenID Connect 1.0 and SAML 2.0 are federated identity specifications with built-in mechanisms for negotiating the authentication-related security controls that are required before access can be granted, but the default treatment of requests and responses in each of those protocols differ significantly.  While SAML 2.0 requires strict processing of authentication context by default, OpenID Connect by default presumes “voluntary” processing. This specification profiles OpenID Connect to accomplish equivalent processing behavior to the default SAML 2.0 specification, adding examples and test criteria for both specifications to give implementers high confidence that any given set of controls can be reliably enforce without worries about protocol divergence. 

## Introduction
SAML 2.0 and OpenID Connect evolved decades apart from each other, and yet operate effectively side-by-side in many identity architectures today as mechanisms for secure introduction of end users across federated parties. Each protocol contains a mechanism for specifying the authentication security controls that must be in place during this process, and both protocols use the term ‘authentication context’ to describe the security controls each party agrees to. Authentication contexts are referred to by identifiers and these identifiers are referred to as ACRs, or authentication context class references.   

This document profiles OpenID Connect such that request parameters, response attributes and validation steps match the default operation of SAML 2.0. To meet this goal, the protocol defines a concept of an ACR Request and an ACR response as a protocol-agnostic concept, then points to the relevant specification text that implements each step.  The profile also defines the test criteria, error conditions, and additional validations that federated parties may choose to use to additionally identify risk and ensure in every possible way that the profile is adhered to.
The minimum interoperability profile is created with the following principles in mind:
 * The profile must be compatible with the linked ratified core specifications. 
 * The profile may add additional attribute definitions, scopes, or metadata parameters where those additional elements are extensible by design in each specification.
  * The profile assumes breach and recommends pro-active additional validation to detect neglectful misconfiguration and/or abuse.  Absence of a claim or metadata element is never interpreted as intentional.
 * All error conditions are defined and are expected to be tested for accuracy by implementers.
Requirements Notation and Conventions
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].
In the .txt version of this document, values are quoted to indicate that they are to be taken literally. When using these values in protocol messages, the quotes MUST NOT be used as part of the value. In the HTML version of this document, values to be taken literally are indicated by the use of this fixed-width font.
## New Terminology
 * __Security Control__:  A named and defined safeguard used to mitigate risk of misuse, fraud or attack.
Authentication Context:  A security control applying requirements to the manner in which a given subject is authenticated prior to issuance of a federated assertion.  In this specifications, authentication contexts are referred to by their class reference (also known as an ACR).  
 * __Authentication Context Class Reference (ACR) or ACR Value__:   A unique identifier associated to a given context, defined using the syntax and namespace matching the federated protocol in use. In this specification, an ACR and the authentication context control that it refers to are used interchangeably.
* __Evidence__: Information used to corroborate or explain how a security control was executed.
 * __ACR Request__: the portion of a federated authentication request defining ACR-related requirements. 
 * __ACR Response__:  the portion of a returned federated assertion that are set as a direct result of an ACR Request
 * __Unsolicited Assertion__: an assertion arrives at an RP that is not the result of an authentication request.  This type of assertion occurs in [SAML2] and is the result of IDP-initiated federation.

 ## Common Protocol Concepts
Where differing terms are defined in the federated protocols referenced in this profile, the chart below can be used to translate.  For the purposes of this document, a Relying Party (RP) is a federated party that consumes identity information, and an Identity Provider (IdP) is the federated party that issues the identity information.  The container that wraps identity data into a verifiable bundle associated to a uniquely identifiable subject is an assertion, and the assertion contains name/value information pairs called claims.
 
![image](https://github.com/pamelatech/ACRminprofile/assets/2591320/5398d172-5a54-46bf-b79b-3f168c53abb5)

![image](https://github.com/pamelatech/ACRminprofile/assets/2591320/ac3f99c7-65a2-46be-afa6-9f6bdf69976f)

## Concepts without a Direct Cross-protocol Equivalent
There are three in-scope areas where the protocols do not exactly match: 
### Voluntary ACR Claims
[OIDC] formalized the concept of  a “voluntary” authentication context claim; there is no matching normative concept existing in [SAML2], but any IDP that includes an ACR claim in an assertion that was not asked for is treating the ACR as voluntary.  Any claim can be explicitly marked as voluntary or essential in [OIDC], but the acr claim is voluntary by default. There are three different circumstances where an ACR claim is considered voluntary:
 1. When no authentication request precedes an assertion sent to an RP.   This can only happen in [SAML2], because only [SAML2] allows IDP-initiated assertions.
 2. When an [OIDC] authentication request categorizes the acr claim as voluntary explicitly or when the request defaults to voluntary by not specifying either way.  
 3. When no ACR requirements were part of the authentication request, but the IDP returns an ACR claim any way.
RP’s that enforce the rules in the ACR Validation section of this profile can safely process assertions where ACR is either voluntary or essential, but the RP may reject more assertions in the former case, leading to less positive user experience.
### Multiple Authentication Contexts
Neither [SAML2] nor [OIDC] explicitly supported the concept of an IDP returning more authentication contexts than the one that the IDP matched due to an RP request, however many modern implementations of both SAML and OpenID Connect have overloaded the ACR claim because there are strong business needs to communicate various contexts that the user may be operating in.  This profile defines an optional additional claim that can be configured for each of [SAML2] and [OIDC] to equivalently solve the problem. NOTE that this custom claim can only be trusted if the IDP advertises that the claim is protected – accepting a claim of the same name without verifying that the IDP implementation is managing that claim as a security claim would create a security vulnerability, since other actors could populate those claim values with non-authoritative values.
### Authentication Methods (AMR)
Authentication methods (AMR) were created in [OIDC] as a mechanism for providing evidence around how an authentication ceremony was completed. There is no concept of AMR in [SAML2] - this profile defines an optional additional claim that can be implemented by [SAML2] providers to equivalently support this feature. NOTE that this custom SAML attribute can only be trusted if the IDP advertises in metadata that the claim is protected – accepting a claim of the same name without verifying that the IDP implementation is managing that claim as a security claim would create a vulnerability, since other actors could populate those claim values with non-authoritative values.
### Comparison Matching 
Comparisons where a listed authentication context is matched against comparative operators such as “minimum”, “maximum” exist in SAML2 but not in OpenID Connect.  The issue with comparison matching is that both parties must understand the entire set involved in a request including unlisted authentication contexts that may be set members.   
### Authentication Context Declaration References
This is a SAML concept that doesn’t exist in OpenID Connect and is therefore ignored in this specification.
## Authentication Context in Protocol Agnostic Terms
Each of SAML 2.0 [SAML2] and OpenID Connect 1.0 [OIDC] have concepts of authentication context, and of agreement to evaluate security context matching a specific authentication context class reference (ACR).   ACR-related functionality falls into five distinct phases.
 

#### Discovery of ACR Metadata
A mechanism for automated lookup of standardized configuration properties for a given participant in a federated process.  
#### ACR Request 
A set of run-time controls embedded within a given authentication request representing the requirements an RP has for IDP processing of authentication context.
#### ACR Evaluation
The process by which an IDP determines whether an end user meets the criteria outlined in the ACR request and any additional interactions an IDP may initiate to bring an end-user into adherence to a given authentication context.
#### ACR Return 
Data included in the assertion returned to the RP that contains the exact authentication context that the IDP successfully evaluated for the current authentication instant.
#### ACR Validation
The process that an RP goes through to decide whether the IDP has acceptably processed the original ACR request.
## OpenID Connect Minimum Profile
Requirements in this section are normative and augment the existing normative text in [OIDC]. 
### ACR Request
Authentication Request Parameters
The claims authentication request parameter defined in [OIDC] §5.5 is REQUIRED.   The acr_values authentication parameter defined in [OIDC] §3.1.2.1 MUST NOT be included in the authentication request.  In addition to the guidance in  [OIDC] §5.5.1.1.
Claims Parameter Configuration
Within the JSON object contained by the claims parameter (see[OIDC] §5.5):
•	The id_token JSON object is REQUIRED.
•	The acr element of the id_token object is REQUIRED.
•	The essential element of the acr object MUST be set to true.
•	The values element of the acr object MUST be present.
Note: the order of the values listed in the request are significant
An example of a valid ACR request in the form of a claims request parameter follows: 
 
### ACR Evaluation
IDPs MUST test the authentication request for the simultaneous presence of the acr_values request parameter and an essential acr claim in the claims parameter. If both exist, the IDP MUST return an error.  
IDPs performing anomaly detection SHOULD be checking to see that federated connections consistently use this profile.   Authentication requests that usually contain an essential acr claim in the claims parameter but suddenly arrive with no acr requirements or voluntary acr requirements should be treated as suspicious. 
If an IDP chooses to populate multiple authentication contexts for consumption by the RP, the following requirements apply:
•	The IDP MUST publish support for the acrs_supported metadata element as part of IDP discovery.
•	The IDP MUST prevent the  acrs attribute from being directly populated by any other entity or for any other purpose at any time.   
As stated in  [OIDC] §5.5.1.1:
 If the acr Claim is requested as an Essential Claim for the ID Token with a values parameter requesting specific Authentication Context Class Reference values and the implementation supports the claims parameter, the Authorization Server MUST return an acr Claim Value that matches one of the requested values. The Authorization Server MAY ask the End-User to re-authenticate with additional factors to meet this requirement. If this is an Essential Claim and the requirement cannot be met, then the Authorization Server MUST treat that outcome as a failed authentication attempt.

### ACR Return
In the case that an essential acr claim was included in the authentication request, the IDP MUST ensure the returned assertion conforms with [OIDC] and additionally has the following properties:
•	The acr attribute MUST contain a single value that represents the first matching acr value listed in the acr element of the claims parameter.
•	The acrs attribute MAY contain a multi-valued array of strings that represents all matching acr values from the claims parameter, plus other unsolicited acr values.
•	If the acrs attribute is supported, the IDP MUST do the following:
o	Prevent any other party from populating an attribute called “acrs”
o	Always populate the acrs attribute when an essential acr claim is requested. 
An example of a valid ACR return is listed below:
 
### ACR Validation
The RP MUST discard the assertion and refuse to grant access if the following is true:
•	The value in the acr attribute contains a string that does not match an originally requested value.
•	The returned assertion does not contain an acr attribute.
•	The IDP advertises support for acrs via the acrs_supported metadata attribute, but the acrs attribute is not present.
### ACR Metadata and Discovery
This profile defines one new [OIDC] metadata element; this element enables a payload claim to be trusted as a protected security attribute.  Exact mechanisms for discovery of the metadata elements are not specified by this profile.

acrs_supported. OPTIONAL.   A Boolean attribute stating whether the multi-valued acrs claim is available as a trusted security attribute.  Default value is FALSE.

Profile Operation: Strict Authentication Context
To guarantee that both participants in a federated connection are operating by strict processing rules, the RP MUST receive a guarantee that the IDP knows that strict processing is required.  This guarantee may happen either in the assertion or in the IDP’s metadata.  Each case is described in the appropriate section.
If an RP cannot receive a guarantee that the IDP has followed strict processing rules, the RP SHOULD reject the assertion.

## SAML2 Minimum Interoperability Profile
Requirements in this section are crafted not to presume that both parties in the federation have implemented this profile.   These requirements standardize the use of additional security attributes, but those attributes can only be trusted when the IDP makes an explicit guarantee of support. 

AuthnContexts.  OPTIONAL.  Multi-valued list of 
AuthnMethod. OPTIONAL. Multi-valued list of

### ACR Request
As per [SAML2] §3.3.2.2.1, RPs populate the RequestedAuthnContext element of the AuthenticationRequest object with a prioritized list of valid ACR values. 
Additional normative requirements for RPs include the following:
 * The RequestedAuthnContext element SHOULD NOT contain a saml:AuthnContextDeclRef element
 * The AuthenticationRequest element SHOULD NOT contain a comparison parameter set to a value other than exact.
 * Instead of using comparison operators, the RP SHOULD explicitly list every member of a set and order that set such that the preferences listed deliver the same results.    
#### Examples of Valid ACR Requests (within a SAML authentication request)
Require telekinesis if available, but finger snap if telekinesis is not available.
 
Require a minimum Assurance Level of Three (out of Five).
 
Require Better than Assurance Level Four (out of Five).
 
Require a Maximum of Assurance Level Two (out of Five).
 
### ACR Evaluation
The IDP is under the following obligations if they receive an authentication request containing a RequestedAuthnContext element:
 * To only return an assertion if at least one of the listed authentication contexts can be matched (see [SAML2] §3.3.2.2 which says “If the <RequestedAuthnContext> element is present in the query, at least one element in the set of returned assertions MUST contain an element that satisfies the element in the query (see Section 3.3.2.2.1). It is OPTIONAL for the complete set of all such matching assertions to be returned in the response.”
 * To evaluate the set of requested ACRs in priority order (see [SAML2] §3.3.2.2.1 which says: “The set of supplied references MUST be evaluated as an ordered set, where the first element is the most preferred authentication context class or declaration. If none of the specified classes or declarations can be satisfied in accordance with the rules below, then the responder MUST return a message with a second-level of urn:oasis:names:tc:SAML:2.0:status:NoAuthnContext.”
 * To populate the \<AuthnContext> claim. 
 
 This means that the IDP:
   * Cannot return an assertion unless an authentication context exists or is established that matches the requirements identified by at least one authentication context  :
   * MUST Establish a sufficient authentication context if none exists
   * MUST populate the Authentication Context 
-	“The responder MUST return a message with a second-level of urn:oasis:names:tc:SAML:2.0:status:NoAuthnContext” – §3.3.2.2.1 [SAML2]
### ACR Return
Single-Valued Authentication Context
If the ACR Evaluation is successful, the IDP MUST:
•	Include an AuthnStatement element in any returned assertions that contains an AuthnContext element containing the AuthnContextClassRef that matched the end user’s current authentication experience.  See [SAML2] §2.7.2.2.
•	The AuthnContext attribute is defined to be a single value.  If exact comparison matching is used, This single value SHOULD match one and only one of the ACRs in the requested ACR list.
 
Optional: Multi-Valued Authentication Context
An IDP may optionally choose to return an extension element in the AttributeStatement of the assertion called AuthnContexts.   The AuthnContexts attribute MUST be a multi-valued attribute and MUST NOT be included if there is only one value listed. <TODO: define exact schema>. If an IDP includes an AuthnContexts attribute in the AttributeStatement, it MUST also:
•	Ensure that the value in the AuthnContext statement within the AuthnStatement is also in AuthnContexts in the AttributeStatement.
•	Ensure that no entity can set the AuthnContexts attribute except the IDP infrastructure itself
•	Include a metadata statement that can be independently verified by an RP that states the AuthnContexts attribute is a protected attribute.
If a Relying Party receives an assertion containing an AuthnContexts attribute in the AttributeStatement when accompanying metadata statement is made by the IDP, the Relying Party MUST assume the attribute is malicious and reject the assertion.
ACR Validation
Once an RP receives an assertion, the RP MUST:
•	Check for presence of an AuthnContext element within the AuthnStatement object.  
o	If the AuthnContext  claim is not present but the RP sent an ACR Request, the RP MUST reject the assertion.
o	If the assertion had no preceding ACR Request, the RP MAY accept the assertion only if all validation rules in [SAML2] and this profile are met.
•	Check that the value set in the AuthnContext claim was one of the values originally provided in the ACR Request.  If the value set in the AuthnContext attribute is not one of the values originally given in the ACR Request the RP MUST reject the assertion.
•	Check for the presence of an AuthnContexts attribute within the AttributeStatement. If the AuthnContexts attribute is present but does not contain one of the originally requested ACRs, or if the RP cannot guarantee that the IDP considers the AuthnContexts attribute to be a protected attribute, the RP MUST reject the assertion.
ACR Metadata and Discovery
This profile defines two new [SAML2] metadata elements; these elements enable payload claims to be trusted as protected security attributes.  Exact mechanisms for discovery of the metadata elements are not specified by this profile.

SupportedAuthnContexts. OPTIONAL.   A Boolean attribute stating whether the multi-valued acrs claim is available as a trusted security attribute.  Default value is FALSE. Implementers are warned that if SupportedAuthenticationMethods is FALSE or absent for any given IDP, any AuthenticationMethod claims found in assertions should be considered as malicious.


SupportedAuthenticationMethods. OPTIONAL.   A Boolean attribute stating whether the multi-valued attribute within the AttributeStatement element is a protected security attribute.  Default value is FALSE.  Implementers are warned that if SupportedAuthenticationMethods is FALSE or absent, any AuthenticationMethod claims found in assertions should be considered as malicious.

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
Specification	Type of Change	Examples of extension
SAML 2.0 Core	None	
OpenID  Connect core	None	
OpenID Connect Discovery 1.0	Definition of new supported attributes	
SAML 2.0 Attribute Extension
Definition of new supported attributes	Example from SimpleSAMLphp

 
# References

[SAML2]  S. Cantor et al. Assertions and Protocols for the OASIS Security Assertion Markup Language (SAML) V2.0. OASIS SSTC, March 2005. https://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf 
[SAMLAuthnCxt] 	J. Kemp et al. Authentication Context for the OASIS Security Assertion Markup Language (SAML) V2.0. OASIS SSTC, March 2005. https://docs.oasis-open.org/security/saml/v2.0/saml-authn-context-2.0-os.pdf 
[SAMLMetadata] 
[RFC2119] RFC 2119


# Notes from IIW
Step-up Auth spec depends on OIDC acr being a voluntary claim

