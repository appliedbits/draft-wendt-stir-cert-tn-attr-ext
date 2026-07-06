---
###
# TN Attribute Certificate Extension for STIR/SHAKEN Certificates
###
title: "TN Attribute Certificate Extension for STI Certificates"
abbrev: "TN Attribute Certificate Extension"
category: std

docname: draft-wendt-stir-cert-tn-attr-ext-00
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Secure Telephone Identity Revisited"
keyword:
- stir
- certificates
- delegate certificates
- tn attributes
- do not originate
venue:
  group: "Secure Telephone Identity Revisited"
  type: "Working Group"
  mail: "stir@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/stir/"
  github: "appliedbits/draft-wendt-stir-cert-tn-attr-ext"
  latest: "https://github.com/appliedbits/draft-wendt-stir-cert-tn-attr-ext"

author:
 -
    fullname: Chris Wendt
    organization: Somos Inc.
    email: chris@appliedbits.com
 -
    fullname: Rob &#x015A;liwa
    organization: Somos Inc.
    email: robjsliwa@gmail.com


normative:
  RFC3986:
  RFC5280:
  RFC5912:
  RFC8224:
  RFC8225:
  RFC8226:
  RFC8816:
  RFC8126:
  RFC9060:
  X.509:
    title: "Information technology - Open Systems Interconnection - The Directory: Public-key and attribute certificate frameworks"
    author:
      - org: International Telecommunication Union
    date: October 2016
    target: https://www.itu.int/rec/T-REC-X.509
    seriesinfo:
      ITU-T: Recommendation X.509, ISO/IEC 9594-8
  X.680:
    title: "Information Technology - Abstract Syntax Notation One (ASN.1): Specification of basic notation"
    author:
      - org: International Telecommunication Union
    date: August 2015
    target: https://www.itu.int/rec/T-REC-X.680
    seriesinfo:
      ITU-T: Recommendation X.680, ISO/IEC 8824-1
  X.681:
     title: "Information Technology - Abstract Syntax Notation One (ASN.1): Information object specification"
     author:
      - org: International Telecommunication Union
     date: August 2015
     target: https://www.itu.int/rec/T-REC-X.681
     seriesinfo:
      ITU-T: Recommendation X.681, ISO/IEC 8824-2
  X.682:
    title: "Information Technology - Abstract Syntax Notation One (ASN.1): Constraint specification"
    author:
      - org: International Telecommunication Union
    date: August 2015
    target: https://www.itu.int/rec/T-REC-X.682
    seriesinfo:
      ITU-T: Recommendation X.682, ISO/IEC 8824-3
  X.683:
    title: "Information Technology - Abstract Syntax Notation One (ASN.1): Parameterization of ASN.1 specifications"
    author:
      - org: International Telecommunication Union
    date: August 2015
    target: https://www.itu.int/rec/T-REC-X.683
    seriesinfo:
      ITU-T: Recommendation X.683, ISO/IEC 8824-4

informative:
  I-D.ietf-stir-certificate-transparency:
  I-D.ietf-stir-servprovider-oob:

--- abstract
This document specifies a non-critical X.509 v3 certificate extension that conveys a set of self-asserted attributes describing the telephone numbers or Service Provider Codes (SPCs) identified in the certificate's TNAuthList. The attributes are declared by the holder of the certificate about its own numbering resources and require no separate authority token, because they describe or constrain only those resources and grant no authority to any other party. The extension defines an extensible framework with an IANA registry of attribute types and seeds that registry with four types: a PASSporT Placement Service (PPS) URI, a do-not-originate indication, a set of authorized originating providers, and a text-enabled indication. Relying parties use these self-declarations as policy signals, treating communications that do not conform to them as candidates for blocking. The mechanism is backward compatible with existing STIR certificates.
--- middle

# Introduction

The STIR (Secure Telephone Identity Revisited) framework provides a means of cryptographically asserting authority over the telephone numbers or Service Provider Codes (SPCs) used in a communication, using PASSporTs carried in SIP requests as defined in {{RFC8224}} and {{RFC8225}}, signed with certificates that carry a TNAuthList extension {{RFC8226}}. A certificate, including a delegate certificate {{RFC9060}}, attests that its holder has been validated as authorized for the numbering resources in its TNAuthList.

The holder of such a certificate is therefore in a unique position to make verifiable statements about those same numbering resources. Because the certificate already establishes the holder's right to use the numbers, statements the holder makes about how those numbers are used, or about which providers may originate communications from them, are trustworthy self-assertions: they describe or constrain the holder's own resources and grant no authority to any other party. This document defines a certificate extension that carries such self-asserted attributes, so that relying parties can obtain the number holder's own declarations about a number and apply them as policy signals.

These attributes are not authoritative in the sense of enabling a service or conferring a capability. They are transparent declarations of the number holder's intent. A relying party that obtains them can recognize communications that do not conform to the holder's declared intent, for example a call that purports to originate from a number the holder has declared never originates calls, and treat such communications as candidates for blocking. The defending value of an attribute therefore comes from a relying party being able to retrieve it, keyed by telephone number, and act on it.

Because each attribute describes only the holder's own numbering resources and grants nothing to anyone else, no authority token is required to assert it. This distinguishes the attributes defined here from the authority recorded in the TNAuthList itself, which is validated at issuance and represents delegated scope. The self-asserted attributes are carried alongside that authority but are semantically separate from it: a relying party MUST NOT interpret any attribute defined here as extending or modifying the certificate's scope of telephone number authority.

This document defines the extension and an extensible IANA registry of attribute types, and seeds the registry with four types: a PASSporT Placement Service (PPS) URI, a do-not-originate indication, a set of authorized originating providers, and a text-enabled indication. Future specifications may register additional types that meet the self-assertion criterion defined in the IANA Considerations.

Note: One of the attribute types defined here carries the URI of a PASSporT Placement Service (PPS). The service defined in {{RFC8816}} is named the Call Placement Service (CPS). This document refers to it as the PASSporT Placement Service (PPS). The two terms denote the same architectural role. The PPS name is used to avoid acronym collision with Certification Practice Statement (CPS) as used in {{RFC8226}} and certificate policy practice, and because the service places and serves PASSporTs and is not limited to telephone calls.

## Relationship to Other Specifications

This document defines a certificate extension data format. It is designed to compose with the broader STIR ecosystem:

- {{RFC8226}} defines the TNAuthList extension that records the authority over which these attributes are asserted, and {{RFC9060}} defines delegate certificates that carry such authority.
- {{RFC8816}} defines the Out-of-Band (OOB) architecture and the PASSporT Placement Service concept, which it names the Call Placement Service (CPS). The PPS URI attribute defined here points to such a service.
- {{I-D.ietf-stir-servprovider-oob}} describes a service-provider OOB deployment model and the possibility of embedding placement service information in STIR certificates.
- {{I-D.ietf-stir-certificate-transparency}} defines STI Certificate Transparency logs, which are one mechanism by which certificates carrying this extension may be observed.

Retrieval of these attributes keyed by telephone number is out of scope for this document. Mechanisms for distributing and querying do-not-originate information by telephone number are already deployed, and the same distribution and query mechanisms are expected to carry the attributes defined here. In all cases the certificate is the authoritative source of the attribute values, and the distribution layer conveys them; see {{conveyance}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The id-pe-tnAttributes Certificate Extension {#extension}

This extension is an X.509 {{X.509}} v3 certificate extension, profiled for the Internet PKI by {{RFC5280}}. It is non-critical, applicable only to end-entity certificates, and defined later in this section with ASN.1 {{X.680}} {{X.681}} {{X.682}} {{X.683}} using the PKIX ASN.1 modules of {{RFC5912}}.

The extension is intended for use in end-entity STI certificates {{RFC8226}} and delegate certificates {{RFC9060}} whose TNAuthList authorizes specific telephone numbers or SPCs. It carries one or more typed attributes, each a self-assertion by the certificate holder about those numbers or SPCs. In this document, the telephone numbers or SPCs identified in the certificate's TNAuthList are referred to as the covered numbering resources.

A TNAuthList {{RFC8226}} may syntactically contain telephone number and telephone number range entries, SPC entries, or a combination of both. This extension is defined only for a certificate whose TNAuthList is homogeneous: one containing only telephone number or telephone number range entries, collectively referred to here as TN entries, or only SPC entries. A producer MUST NOT include this extension in a certificate whose TNAuthList contains both a TN entry and an SPC entry, and a relying party that encounters this extension in such a certificate MUST ignore the extension in its entirety. Because the extension is non-critical, ignoring it does not affect validation of the certificate.

## The Self-Assertion Principle

Every attribute carried in this extension is a self-assertion by the holder of the certificate about the numbering resources in the certificate's TNAuthList. An attribute does not require an authority token, and is trustworthy without one, precisely because it describes or constrains only the holder's own numbering resources and grants no authority, capability, or service to any other party.

The worst outcome an incorrect or malicious self-assertion can produce is degradation of service to the asserting holder's own numbers. An attribute cannot expand the holder's authority, cannot confer authority on a third party, and cannot make a statement about a resource the holder does not control. Where an attribute references another party, as the authorized originating providers attribute references SPCs, that reference is a statement of the holder's own acceptance policy and confers nothing on the referenced party; the referenced party's authority, if any, derives solely from its own credentials.

This principle is the inclusion criterion for the IANA registry defined in {{iana}}. An attribute type qualifies for registration only if it satisfies it. Attributes that would grant authority, enable a capability, or assert facts about resources outside the holder's control do not qualify and belong in a token-gated mechanism instead.

The container guarantees only provenance: every attribute in it was asserted by the validated holder of the numbering resources in the certificate. Each attribute type defines its own semantics, given in {{attribute-types}} for the types defined here and in the registering specification for types added later.

## ASN.1 Module Syntax

The extension is identified by an object identifier (OID) assigned in the PKIX id-pe arc defined in {{RFC5280}}. Its value is a sequence of one or more typed attributes. Each attribute carries an integer type, which keys into the IANA "STIR TN Attribute Types" registry ({{iana}}), and a value whose syntax is defined by the registration for that type. The set of types is extensible: the extension marker in the object set below permits attributes of types not defined in this module to be carried and, if unrecognized, skipped. A specification that defines a new type registers it, with its value syntax, and adds it to the object set; existing implementations that do not recognize the new type are unaffected.

~~~~~~~~~~~~~
TN-ATTRIBUTES-EXTN
  { iso(1) identified-organization(3) dod(6) internet(1)
    security(5) mechanisms(5) pkix(7) id-mod(0)
    id-mod-tn-attributes(TBD0) }

DEFINITIONS EXPLICIT TAGS ::=
BEGIN

IMPORTS
  EXTENSION
  FROM PKIX-CommonTypes-2009  -- RFC 5912
    { iso(1) identified-organization(3) dod(6) internet(1)
      security(5) mechanisms(5) pkix(7) id-mod(0)
      id-mod-pkixCommon-02(57) }

  id-pe
  FROM PKIX1Explicit-2009  -- RFC 5912
    { iso(1) identified-organization(3) dod(6) internet(1)
      security(5) mechanisms(5) pkix(7) id-mod(0)
      id-mod-pkix1-explicit-02(51) } ;

-- TN Attributes Certificate Extension

ext-TNAttributes EXTENSION ::= {
  SYNTAX TNAttributes
  IDENTIFIED BY id-pe-tnAttributes }

id-pe-tnAttributes OBJECT IDENTIFIER ::= { id-pe TBD1 }

-- Information object class binding an attribute value type
-- to its registered integer identifier

TN-ATTRIBUTE ::= CLASS {
  &id    INTEGER UNIQUE,
  &Type
} WITH SYNTAX { TYPE &Type IDENTIFIED BY &id }

-- The extension value: one or more typed attributes

TNAttributes ::= SEQUENCE SIZE (1..MAX) OF TNAttribute

TNAttribute ::= SEQUENCE {
  type   TN-ATTRIBUTE.&id    ({TNAttributeTypes}),
  value  TN-ATTRIBUTE.&Type  ({TNAttributeTypes}{@type}) }

-- Set of attribute types defined in this document.
-- Extensible via the IANA registry; the marker permits
-- types registered by other specifications.

TNAttributeTypes TN-ATTRIBUTE ::= {
  tnAttr-ppsURIs              |
  tnAttr-doNotOriginate       |
  tnAttr-authorizedOriginators |
  tnAttr-textEnabled,
  ... }

tnAttr-ppsURIs TN-ATTRIBUTE ::=
  { TYPE PPSURIs IDENTIFIED BY 1 }
tnAttr-doNotOriginate TN-ATTRIBUTE ::=
  { TYPE DoNotOriginate IDENTIFIED BY 2 }
tnAttr-authorizedOriginators TN-ATTRIBUTE ::=
  { TYPE AuthorizedOriginators IDENTIFIED BY 3 }
tnAttr-textEnabled TN-ATTRIBUTE ::=
  { TYPE TextEnabled IDENTIFIED BY 4 }

-- Value syntaxes for the defined types

PPSURIs ::= SEQUENCE SIZE (1..MAX) OF IA5String

DoNotOriginate ::= NULL

AuthorizedOriginators ::= SEQUENCE SIZE (1..MAX) OF SPC

SPC ::= IA5String

TextEnabled ::= BOOLEAN

END
~~~~~~~~~~~~~

Note: The numeric module and OID assignments marked TBD are temporary. IANA will allocate the permanent values during RFC publication.

## Criticality

The extension MUST be marked non-critical, following the certificate extension processing rules of {{RFC5280}}, so that a relying party that does not recognize it can still validate the certificate. An attribute that a relying party does not understand, or whose type is not in its supported set, is ignored.

# Attribute Types {#attribute-types}

Each attribute type defines its value syntax and its semantics. A relying party that does not support a given type ignores it. The absence of an attribute is not an assertion: a relying party MUST NOT infer a value, restrictive or permissive, for an attribute that is not present. The restrictive attributes defined here constrain behavior only when present.

Attribute types apply according to whether the covered numbering resources are TN entries or SPC entries, as follows:

| Attribute | TN entries | SPC entries |
|-----------|:----------:|:-----------:|
| pps-uris | yes | yes |
| do-not-originate | yes | no |
| authorized-originators | yes | no |
| text-enabled | yes | no |

A producer MUST NOT include an attribute in a certificate whose covered numbering resources are of a type to which that attribute does not apply. A relying party MUST ignore an attribute that does not apply to the type of the covered numbering resources. Recall from {{extension}} that the extension is not applicable at all to a certificate whose TNAuthList mixes TN and SPC entries.

A given type SHOULD appear at most once in the extension. If a type that is defined as single-valued appears more than once, a relying party MUST treat the certificate's attribute set as ambiguous for that type and ignore that type.

## PPS URIs (type 1)

Applies to: TN entries and SPC entries.

The value is a sequence of one or more absolute URIs {{RFC3986}}, each of which:

- Uses the "https" scheme.
- Identifies the root of a PASSporT Placement Service (PPS) HTTPS API interface (for example, "https://pps.example.net/oob/v1").

This attribute declares where PASSporTs for the covered numbering resources may be published and retrieved out of band {{RFC8816}}. A value that is not an absolute HTTPS URI MUST cause a relying party to ignore that entry. Producers MAY include multiple URIs to express redundancy or locality; selection among them is a matter of local policy.

This attribute is enabling rather than restrictive: it points a relying party to a service. It satisfies the self-assertion principle because it declares only where the holder publishes PASSporTs for its own numbers and grants nothing to any other party.

## Do Not Originate (type 2)

Applies to: TN entries only.

The value is NULL; the presence of the attribute is the assertion. When present, this attribute asserts that the covered numbering resources do not originate communications. A relying party that obtains this attribute for a number, and then sees a communication purporting to originate from that number, has a strong signal that the communication is illegitimate and SHOULD treat it as a candidate for blocking according to local policy. Absence of the attribute is not an assertion that the number does originate.

## Authorized Originators (type 3)

Applies to: TN entries only.

The value is a sequence of one or more SPCs, each identifying a provider that the holder of the numbering resources authorizes to originate and attest communications from those resources. A relying party that obtains this attribute, and then sees a communication from the covered number attested by a provider whose SPC is not in the set, has a signal that the origination is unauthorized and SHOULD treat it as a candidate for blocking according to local policy.

This attribute is a statement of the holder's acceptance policy for its own numbers. It confers no authority on the listed providers: a listed provider's ability to sign for the number derives solely from its own credentials, and a relying party MUST NOT interpret presence in this set as STIR authority of any kind. Likewise, a relying party MUST NOT interpret an SPC in this attribute as extending the certificate's own telephone number authority. Absence of this attribute imposes no origination restriction: the holder has declared no authorized-originator set, and communications may be attested by any provider as far as this attribute is concerned.

The do-not-originate and authorized-originators attributes express contradictory intent, since do-not-originate asserts that the numbers originate nothing while authorized-originators names providers permitted to originate. A producer MUST NOT include both in the same certificate. If a relying party nonetheless encounters both, the do-not-originate attribute takes precedence, as the more restrictive of the two, and the authorized-originators attribute is ignored.

## Text Enabled (type 4)

Applies to: TN entries only.

The value is a BOOLEAN, and unlike the presence-only attributes this type is meaningful in both directions. A value of TRUE asserts that the covered numbering resources are text-enabled, that is, provisioned to support text messaging, mirroring the text-enabled status set in number administration. A value of FALSE asserts that they are not text-enabled. Absence of the attribute is indeterminate: it distinguishes a holder that has not asserted a text-enabled status from one that has affirmatively set it either way, and a relying party MUST NOT infer either value from absence.

A relying party MAY use a value of TRUE as a positive signal that messaging associated with the numbers is expected, and MAY treat messaging that purports to originate from numbers asserted FALSE as a candidate for blocking according to local policy. Where the attribute is absent, a relying party draws no conclusion about text capability from this attribute.

# Processing Rules

A STIR Authentication Service {{RFC8224}} that holds a certificate carrying a PPS URIs attribute SHOULD publish out-of-band PASSporTs for the covered numbering resources to one of the indicated PPS endpoints.

A relying party that has obtained a certificate carrying this extension, whether from a signed communication or by retrieval keyed to a telephone number, validates the certificate chain to a trusted STIR trust anchor before relying on any attribute. Having done so, it applies each supported attribute according to that attribute's semantics in {{attribute-types}}. The restrictive attributes, do-not-originate and authorized-originators, are policy signals: a relying party SHOULD treat a communication that does not conform to a present restrictive attribute as a candidate for blocking, subject to local policy. The text-enabled attribute is bidirectional: a relying party MAY use a TRUE value as a positive signal, MAY treat messaging that purports to originate from numbers asserted FALSE as a candidate for blocking, and draws no conclusion from its absence.

The defending value of the restrictive attributes is realized only when a relying party can obtain them for a number independently of the communication being evaluated. The cases these attributes are designed to catch, such as an unsigned communication from a do-not-originate number or one attested by an unauthorized provider, are exactly the cases in which the holder's own certificate is not presented on the communication. In those cases the relying party obtains the attributes by retrieving the holder's certificate keyed by telephone number, by a mechanism outside the scope of this document.

# Distribution and Transparency {#conveyance}

Retrieval and distribution of these attributes keyed by telephone number are out of scope for this document and are left to other specifications and to industry practice.

The certificate is the authoritative, signed source of the attribute values. A relying party validates the certificate independently of how it was obtained, so trust in an attribute derives from the certificate rather than from whatever mechanism delivered it.

Transparency logs provide a monitoring mechanism by which distributors can observe the attributes asserted for a telephone number as certificates are issued, and populate the authoritative information that relying parties consult.

A distribution mechanism SHOULD convey the signed certificate together with the distributed information, so that a relying party can verify each attribute directly against the certificate rather than trust the distributor. Where the signed certificate is not conveyed, the relying party depends on the distribution layer for the integrity of the information. This document does not otherwise mandate a distribution mechanism.

# Lifecycle Considerations

The attributes reflect the holder's declarations at the time the certificate was issued. Because they are carried in the certificate, changing an attribute requires issuing a new certificate, and the change takes effect at relying parties only after the new certificate has propagated through the distribution mechanism in use. Attribute currency therefore follows certificate lifetime. Short-lived certificates bound the interval over which a stale declaration remains in use and are RECOMMENDED for this reason. This ties attribute updates to the same validated issuance path as the certificate itself, so that a declaration cannot be forged into the distribution layer.

# Security Considerations

The attributes defined here are self-assertions by the holder of the numbering resources in the certificate's TNAuthList. Their trust rests on two things: the certificate having been validated at issuance as authorizing the holder for those resources, and the relying party validating the certificate chain before acting on any attribute. A relying party MUST validate the chain to a trusted STIR trust anchor; the attribute values are not independently signed and derive their integrity from the certificate.

Because each attribute describes or constrains only the holder's own numbering resources and grants nothing to any other party, the worst outcome of an incorrect or malicious assertion is degradation of service to the asserting holder's own numbers. An attribute cannot expand the holder's authority or confer authority on a third party. In particular, the authorized-originators attribute lists SPCs as a statement of the holder's acceptance policy; it confers no STIR authority on those providers, and a relying party MUST NOT treat presence in that list as authority of any kind, nor as extending the certificate's telephone number scope.

Misissuance is the relevant residual risk: a certificate that carries these attributes without the holder having been properly validated for the listed numbering resources would carry self-assertions that should not be trusted. The validation that binds a certificate to its numbering resources is the responsibility of the issuing CA and the certificate profile in use, and is outside the scope of this extension.

The restrictive attributes defend against impersonation only when a relying party can retrieve them keyed by telephone number, independently of the communication under evaluation, as discussed in {{conveyance}}. A relying party that cannot perform such retrieval gains no protection from them against the unsigned or wrongly-attested cases. Reliance on a distribution mechanism for retrieval inherits that mechanism's availability and integrity properties; because the certificate is signed, a compromised distribution layer can suppress or withhold attributes but cannot forge them.

Publication of these attributes in retrievable or publicly logged certificates may reveal deployment metadata about a number's usage. This exposure is consistent with existing STIR certificate practice and does not introduce privacy risk beyond what TNAuthList usage already entails.

# IANA Considerations {#iana}

## TN Attributes Certificate Extension OID

IANA is requested to assign an object identifier for the TN Attributes certificate extension in the "SMI Security for PKIX Certificate Extension" registry:

- Description: id-pe-tnAttributes
- Reference: (this document)

## STIR TN Attribute Types Registry

IANA is requested to create a new registry, "STIR TN Attribute Types", to record the integer type identifiers carried in the TN Attributes certificate extension defined in this document.

Each entry records:

- Type: the integer identifier (the "type" field of a TNAttribute).
- Name: a short identifier for the attribute type.
- Value Syntax: the ASN.1 type of the attribute value.
- Reference: the defining specification.

The registration policy is Specification Required {{RFC8126}}. The designated expert MUST verify that a requested type satisfies the self-assertion criterion: the attribute describes or constrains only the asserting holder's own numbering resources, grants no authority, capability, or service to any other party, and makes no assertion about a resource the holder does not control. A type that would grant authority, enable a capability, or assert facts about resources outside the holder's control does not qualify and is to be refused, as such a mechanism requires authorization beyond a self-assertion.

The registry is initialized with the following entries, all defined in this document:

| Type | Name | Value Syntax | Reference |
|------|------|--------------|-----------|
| 1 | pps-uris | PPSURIs | (this document) |
| 2 | do-not-originate | DoNotOriginate | (this document) |
| 3 | authorized-originators | AuthorizedOriginators | (this document) |
| 4 | text-enabled | TextEnabled | (this document) |

Type value 0 is reserved.

--- back

# Changes from Earlier Revisions
{:numbered="false"}

This is an editorial section that tracks changes across revisions and is intended to be removed before any eventual publication.

This document originated as draft-sliwa-stir-cert-cps-ext, which defined a single-purpose certificate extension carrying a Call Placement Service URI, and was subsequently renamed to use the PASSporT Placement Service (PPS) term. This revision generalizes that work:

- The extension is generalized from a single PPS URI value to an extensible set of self-asserted TN attributes. The former PPS URI extension becomes one registered attribute type (type 1) within the new framework, with its value syntax and HTTPS URI semantics carried forward unchanged.
- An IANA "STIR TN Attribute Types" registry is established, with a self-assertion inclusion criterion, and seeded with four types: pps-uris, do-not-originate, authorized-originators, and text-enabled.
- The authorized-originators attribute provides a certificate-carried means for a number holder to declare the providers it authorizes to originate, replacing earlier approaches that overloaded the TNAuthList for this purpose.
- The document is retitled and renamed accordingly.

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
