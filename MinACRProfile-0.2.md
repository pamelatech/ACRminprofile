# Minimum Interoperable Profile for Authentication Context in Federated Protocols
##### P. Dingle, Microsoft, D. Olds, Broadcom, A. Parecki, Okta
To be submitted to: OpenID Foundation Connect WG

February 2024 (draft)
## Abstract
OpenID Connect 1.0 codifies the use of authentication context in a federated transaction as a "voluntary" security control to enforce for an openid provider. For relying parties that require security control enforcement to be considered essential, and for any implementer tasked with enforcing security controls that may involve nested SAML/OpenID Connect requests, this default behavior is painful to compensate for and results in ambiguity that could have security implications. This profile codifies a pattern by which an OpenID Provider may declare default treatment of any authentication context in a Relying Party request as essential, and provides processing rules and recommendations to mirror the processing logic of SAML 2.0, with an explicit goal of ensuring that federated interactions may be nested and transformed between the two protocols with no change to authentication context processing. 

## Introduction
SAML[SAML] and OpenID Connect[OIDC] operate effectively side-by-side in many identity architectures as mechanisms for secure introduction of end users across federated parties. Each protocol adopts the concept of ‘authentication context’ to describe the security controls negotiated between parties, yet differences in default processing logic create difficulty for federated entities with high assurance processing requirements, specifically relying parties that expect OPs not to issue assertions in the case where essential security controls cannot be met. While it is in theory possible for a relying party to structure an [OIDC] authentication request to redefine the acr claim as essential rather than voluntary, to do so requires support for the `claims` request parameter ([OIDC] section 5.5), and very few implementations support this parameter to date.

This profile:
 * Defines required OP metadata and behavior declaring that the OP will treat any requested authentication context as an essential claim.
 * Mandates the use of the unmet_authentication_requirements error code in cases where no authentication context can be met and adds additional validation recommendations for OPs and RPs.
 * Recommends acr_values parameter construction formats for equivalency with SAML comparative operator constructs.
 * Gives non-normative examples of nested SAML and OIDC requests and responses.

## Terminology
 * __Authentication Context__:  A security control applying requirements to the manner in which a given subject is authenticated prior to issuance of a federated assertion.  In this specifications, authentication contexts are referred to by their class reference (also known as an ACR).
 * __Authentication Context Class Reference (ACR)__:   A unique identifier associated to a given authentication context. In this specification, an ACR and the authentication context control that it refers to are used interchangeably.  Example ACRs include mfa (multi-factor authentication) and pwd (password).
 * __`acr` claim__: a standardized label for the id_token/userinfo claim identifying the exact ACR enforced at time of authentication. See [OIDC] section 2.0.
 * __`acr_values` request parameter__:  the name of a parameter that identifies one or more authentication contexts that the OP may evaluate prior to returning an assertion.  See [OIDC] section 3.1.2.1.

### Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].

## ACR Negotiation Steps
Requirements in this section are normative and augment the existing normative requirements in [OIDC]. This profile defines five tasks where authentication context is referenced:

                                             0. End user
                                             │  access attempt
                                             ▼
      ┌─────────────────┐  1. ACR Discovery ┌─────────────────┐
      │                 │ ◄─ ─ ─ ─ ─ ─ ─ ─► │                 │
      │     OpenId      │                   │     Relying     │
      │    Provider     │                   │      Party      │
      │                 │  2. ACR Request   │                 │
      │ ┌─────────────┐ │ ◄──────────────── │                 │
      │ │3. ACR       │ │                   │                 │
      │ │   Evaluation│ │                   │                 │
      │ └─────────────┘ │ ────────────────► │ ┌─────────────┐ │
      │                 │  4. ACR Response  │ │5. ACR       │ │
      │                 │                   │ │   Validation│ │──► 6. End user is 
      │                 │                   │ └─────────────┘ │       granted/denied
      └─────────────────┘                   └─────────────────┘       access 
      
 1. __ACR Discovery__: The RP looks up the ACR-related capabilities of the OP in advance or at the time of an individual federated interaction.  
 1. __ACR Request__: The RP generates an authentication request that lists acceptable ACR values. 
 1. __ACR Evaluation__: The OP evaluates the ACR request and determines whether an end user meets the criteria specified, performing any necessary interactions needed to bring an end-user into adherence, or determining that adherence is not possible.
 1. __ACR Response__: The OP replies to a given ACR request with either a standardized error message or an id_token containing an appropriate `acr` claim. 
 1. __ACR Validation__: The RP validates the received id_token to ensure the OP has met the requirements in the ACR Request.

# ACR Discovery
## OP Metadata Profile support
The OP MUST publish an [OIDCDISC] compliant discovery endpoint and discovery document. The following new OP metadata claim MUST be present within the discovery document:

acr_essential_supported<br/>
&ensp;&ensp;&ensp;OPTIONAL. Boolean value specifying whether the OP supports conformance to this specification, with `true` indicating support. If omitted, the default value is `false`.  

The following metadata claims defined in [OIDCDISC] section 3 MUST be present in the discovery document:

acr_values_supported<br/>
&ensp;&ensp;&ensp;REQUIRED. JSON array containing a list of the ACRs that this OP will enforce on request. 

claims_parameter_supported<br/>
&ensp;&ensp;&ensp;REQUIRED. Boolean value specifying whether the OP supports use of the claims parameter, with true indicating support. 

The OP MUST list at least one ACR in the `acr_values_supported` JSON array to conform to this profile. 

The following is a truncated non-normative example [OIDCDISC] discovery endpoint response containing profile-conformant metadata:
       HTTP/1.1 200 OK
        Content-Type: application/json
      
        {
         "issuer":  "https://server.example.com",
           ...
         "acr_essential_supported":     true,
         "acr_values_supported":        ["phr","mfa"],
          "claims_parameter_supported": false,
          ...
         }

### Voluntary Processing when ACR is specified
To re-enable the option of voluntary processing for an RP that wants to request preferred security controls but expects to receive an id_token even if none of the controls could be met, the OP MAY advertise the `any` ACR (defined below) as a [OIDCDISC] `acr_values_supported` metadata value.  

any 
&ensp;&ensp;&ensp;Any established authentication context. 

The following is a truncated non-normative example [OIDCDISC] discovery endpoint response containing profile-conformant metadata where voluntary processing has been re-enabled:

       HTTP/1.1 200 OK
        Content-Type: application/json
      
        {
         "issuer":  "https://server.example.com",
           ...
         "acr_essential_supported":     true,
         "acr_values_supported":        ["phr","mfa","pwd","any"],
         "claims_parameter_supported": false,
          ...
         }
         
# ACR Request
 * If RP cannot find metadata, ??
 * RP MUST use either acr_values or claims request parameter in request
 * If RP populates acr claim in request, it must request essential
 * If RP wants voluntary processing from an essential OP, must add "any" to end of ACR values.
 * 
# ACR Evaluation
 * OP MUST ensure that claims parameter & acr_values don't contradict
 * If OP supports minprofile and claims request states voluntary, add "any" to acr list and return any if no other security contexts are found.
 * 
# ACR Response
# ACR Validation


# Security Considerations

## Authentication Request Tampering

# References
[OIDC] OpenID Connect Core 1.0 incorporating errata set 2. https://openid.net/specs/openid-connect-core-1_0.html.

[OIDCUAR]  Lodderstedt, T., "OpenID Connect Core Error Code unmet_authentication_requirements", 8 May 2019, <https://openid.net/specs/openid-connect-unmet-authentication-requirements-1_0.html>.

[OIDCDISC] Sakimura, N., Bradley, J., Jones, M., and E. Jay, "OpenID Connect Discovery 1.0 incorporating errata set 1", 8 November 2014, <https://openid.net/specs/openid-connect-discovery-1_0.html>.

[SAML2]  S. Cantor et al. Assertions and Protocols for the OASIS Security Assertion Markup Language (SAML) V2.0. OASIS SSTC, March 2005. https://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf. 

[SAMLAuthnCxt]  J. Kemp et al. Authentication Context for the OASIS Security Assertion Markup Language (SAML) V2.0. OASIS SSTC, March 2005. https://docs.oasis-open.org/security/saml/v2.0/saml-authn-context-2.0-os.pdf 

[SAMLMetadata] 

[RFC2119] RFC 2119
