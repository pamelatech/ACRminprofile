# Minimum Interoperable Profile for Authentication Context in Federated Protocols
##### P. Dingle, Microsoft, D. Olds, Broadcom, A. Parecki, Okta
To be submitted to: OpenID Foundation Connect WG

February 2024 (draft)
## Abstract
OpenID Connect 1.0 offers a default interpretation of authentication context as a voluntary processing element in a federated transaction. An OpenID Provider(OP) may choose whether or not to enforce security controls requested by a Relying Party prior to issuing an id_token. This profile codifies the patterns and mechanisms by which an OpenID Provider may declare default treatment of any authentication context in a Relying Party request as essential. Processing rules for both OpenID Providers and Relying Parties intentionally closely mirror the processing logic of SAML 2.0, with an explicit goal of ensuring that federated interactions may be nested and transformed between the two protocols with no change to authentication context processing. 

## Introduction
SAML[SAML] and OpenID Connect[OIDC] evolved decades apart from each other, and yet operate effectively side-by-side in many identity architectures as mechanisms for secure introduction of end users across federated parties. Each protocol adopts the concept of ‘authentication context’ to describe the security controls negotiated between parties, yet differences in default processing logic create difficulty for federated entities with high assurance processing requirements, specifically relying parties that expect OPs not to issue assertions in the case where essential security controls cannot be met. While it is in theory possible for a relying party to structure an [OIDC] authentication request to redefine the acr claim as an essential rather than voluntary claim, in practice few OPs support the parameter by which the redefinition would take place (the claims request parameter). 

This profile:
 * Defines a standardized mechanism for an OP to declare that any requested authentication context will be treated as an essential claim.
 * Requires the use of the unmet_authentication_requirements error code in cases where no authentication context can be met and adds additional validation recommendations for OPs and RPs.
 * Recommends acr_values parameter construction formats for equivalency with SAML comparative operator constructs.
 * Gives non-normative examples of nested SAML and OIDC requests and responses 



# References
[OIDC] OpenID Connect Core 1.0 incorporating errata set 2. https://openid.net/specs/openid-connect-core-1_0.html.

[OIDCUAR]  Lodderstedt, T., "OpenID Connect Core Error Code unmet_authentication_requirements", 8 May 2019, <https://openid.net/specs/openid-connect-unmet-authentication-requirements-1_0.html>.

[OIDCDISC] Sakimura, N., Bradley, J., Jones, M., and E. Jay, "OpenID Connect Discovery 1.0 incorporating errata set 1", 8 November 2014, <https://openid.net/specs/openid-connect-discovery-1_0.html>.

[SAML2]  S. Cantor et al. Assertions and Protocols for the OASIS Security Assertion Markup Language (SAML) V2.0. OASIS SSTC, March 2005. https://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf. 

[SAMLAuthnCxt]  J. Kemp et al. Authentication Context for the OASIS Security Assertion Markup Language (SAML) V2.0. OASIS SSTC, March 2005. https://docs.oasis-open.org/security/saml/v2.0/saml-authn-context-2.0-os.pdf 

[SAMLMetadata] 

[RFC2119] RFC 2119
