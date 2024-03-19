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

acr_values_essential<br/>
&ensp;&ensp;&ensp;OPTIONAL. Boolean value specifying whether the OP supports conformance to this specification, with `true` indicating support. If omitted, the default value is `false`.  

The following metadata claims defined in [OIDCDISC] section 3 MUST be present in the discovery document:

acr_values_supported<br/>
&ensp;&ensp;&ensp;REQUIRED. JSON array containing a list of the ACRs that this OP will enforce on request. 


The OP MUST list at least one ACR in the `acr_values_supported` JSON array to conform to this profile. 

The following is a truncated non-normative example [OIDCDISC] discovery endpoint response containing profile-conformant metadata:
       HTTP/1.1 200 OK
        Content-Type: application/json
      
        {
         "issuer":  "https://server.example.com",
           ...
         "acr_values_essential":     true,
         "acr_values_supported":        ["phr","mfa"],
          ...
         }

### ACR Request
The ACR request is an [OIDC] section 3.2.2.1. compliant authentication request that MUST specify ACR values via one of two methods:
#### Compatibility with OIDC Default Voluntary Processing
Any RP that wants to always get an id_token similar to what can happen in the [OIDC] default voluntary processing can still achieve that goal if the OP lists all the ACRs it supports in the acr_values_supported metadata. The RP can then include every ACR in a request (ordered for preference of enforcement) and get an id_token for any successful authentication.

#### ACR values in claims request parameter
The RP MAY include the claims parameter defined in [OIDC] §5.5 in the authentication request. Within the JSON object contained by the claims parameter (see[OIDC] §5.5):
* The `id_token` JSON object is REQUIRED.
* The `acr` element of the `id_token` object is REQUIRED.
* The `essential` element of the acr object MUST be set to true.
* The values element within the acr object MUST be present and contain at least one value.

A non-normative example of an id_token object in a claims request parameter follows: 

    {
      "id_token": {
        "acr": {
          "essential": true,
          "values": ["inherence","possession"]
        }
      }
    }

#### ACR values in profiled acr_values parameter
If ACR Metadata fetched by the RP contains the metadata `acr_values_essential`, the RP MUST consider the `acr_values` parameter defined in [OIDC] §3.1.2.1 as specifying essential acr values. 

A non-normative example of an acr_values parameter within an ACR request follows: 

      phr mfa pwd

The above parameter values in context of a full ACR Request follows:

      HTTP/1.1 302 Found
      Location: https://server.example.com/authorize?
        response_type=code
        &acr_values=phr%20mfa%20pwd
        &scope=openid%20profile%20email
        &client_id=s6BhdRkqt3
        &state=af0ifjsldkj
        &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb

It is RECOMMENDED that RPs preferentially use the `acr_values` request parameter over the claims request parameter where possible. If the ACR request includes both the acr_values parameter and the claims request parameter and the `acr` claim is a defined value with the claims request object, the IDP MUST return `malformed_acr_parameters` (note this extends important guidance in [OIDC] §5.5.1.1). 

# ACR Evaluation
ACR Evaluation is performed by the OP and acts upon an ACR Request. ACR evaluation adds processing requirements to core specification requirements including but not limited to [OIDC] sections 3.1.2.2, 3.1.2.3, 3.1.2.4 and 5.5.1.1.

The OP MUST test the ACR request for the simultaneous presence of the `acr_values` request parameter and an `acr` element in the claims request parameter JSON object. If simultaneous presence is found, the OP MUST return the `malformed_acr_parameters` error code.  If the essential boolean value included in a present acr element is set to false, this profile cannot be applied to the ACR Request, and the OP SHOULD return the `malformed_acr_parameters` error code.

OPs performing anomaly detection SHOULD detect whether each RP consistently uses this profile.   Authentication requests that usually contain an essential acr claim in the claims parameter but suddenly arrive with no acr requirements or voluntary acr requirements should be treated as suspicious. 

As stated in  [OIDC] §5.5.1.1:
> If the acr Claim is requested as an Essential Claim for the ID Token with a values parameter requesting specific Authentication Context Class Reference values and the implementation supports the claims parameter, the Authorization Server MUST return an acr Claim Value that matches one of the requested values. The Authorization Server MAY ask the End-User to re-authenticate with additional factors to meet this requirement. If this is an Essential Claim and the requirement cannot be met, then the Authorization Server MUST treat that outcome as a failed authentication attempt.

This profile defines any specification of an ACR (by any method) as an essential claim.  This means that section 5.5.1.1 applies to any authentication request that is also an ACR request.  OPs SHOULD advise RPs that have an unacceptable rate of failed authentication attempts to revise their requested ACR values to be more match more of the OP's supported ACR values in order to change the number of situations in which an id_token can be returned.

# ACR Response
The ACR response is an [OIDC] section XXX compliant authentication response that MUST specify an acr claim value within the id_token. 

The OP MUST ensure that every returned id_token conforms with [OIDC] and additionally has the following properties:
* The acr claim MUST contain a single value that represents the first matching acr value listed in the acr element of the claims parameter.
* The acrs attribute MAY contain a multi-valued array of strings that represents all matching acr values from the claims parameter, plus other unsolicited acr values.
* If the acrs claim is supported, the IDP MUST do the following:
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
# ACR Validation

# ACR Error Response
An ACR error response MUST conform to [OIDC] section 3.1.2.5 and 3.1.2.6. 
The following additional error codes are in scope of this profile:

unmet_authentication_requirements - defined in [OIDCUAR]

malformed_acr_parameters
&ensp;&ensp;&ensp;The OpenID Provider is unable to process the authentication request due to ambiguous specification of ACR values in either the acr_values parameter or the claims request parameter (or both).


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
