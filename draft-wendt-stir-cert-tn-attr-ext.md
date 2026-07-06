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
  RFC9475:
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
This document specifies a non-critical X.509 v3 certificate extension that conveys a set of self-asserted attributes describing the telephone numbers identified in the certificate's TNAuthList. The attributes are declared by the holder of the certificate about its own telephone numbers and require no separate authority token, because they describe or constrain only those numbers and grant no authority to any other party. The extension defines an extensible framework with an IANA registry of attribute types and seeds that registry with four types: a PASSporT Placement Service (PPS) URI, a do-not-originate indication, a do-not-originate-messaging indication, and a set of authorized originating providers. Relying parties use these self-declarations as policy signals, treating communications that do not conform to them as candidates for blocking. The mechanism is backward compatible with existing STIR certificates.
--- middle

# Introduction

The STIR (Secure Telephone Identity Revisited) framework provides a means of cryptographically asserting authority over the telephone numbers used in a communication, using PASSporTs carried in SIP requests as defined in {{RFC8224}} and {{RFC8225}}, signed with certificates that carry a TNAuthList extension {{RFC8226}}. A certificate, including a delegate certificate {{RFC9060}}, attests that its holder has been validated as authorized for the telephone numbers in its TNAuthList.

The holder of such a certificate is therefore in a unique position to make verifiable statements about those same telephone numbers. Because the certificate already establishes the holder's right to use the numbers, statements the holder makes about how those numbers are used, or about which providers may originate communications from them, are trustworthy self-assertions: they describe or constrain the holder's own numbers and grant no authority to any other party. This document defines a certificate extension that carries such self-asserted attributes, so that relying parties can obtain the number holder's own declarations about a number and apply them as policy signals.

These attributes are not authoritative in the sense of enabling a service or conferring a capability. They are transparent declarations of the number holder's intent. A relying party that obtains them can recognize communications that do not conform to the holder's declared intent, for example a call that purports to originate from a number the holder has declared never originates calls, and treat such communications as candidates for blocking. The defending value of an attribute therefore comes from a relying party being able to retrieve it, keyed by telephone number, and act on it.

These attributes express the number holder's intent regarding outbound origination from the covered telephone numbers, and they are evaluated within the STIR trust framework. STIR identity information, carried as PASSporTs {{RFC8225}} signed with STIR certificates and conveyed for example in a SIP Identity header field {{RFC8224}} or, for messaging, as described in {{RFC9475}}, is the means by which conformance to that intent is attested for outbound voice and messaging from a telephone number. The scope of these attributes therefore follows the scope of STIR itself, rather than any particular service or transport: a service that adopts STIR to attest origination from a telephone number participates in this framework and inherits these attributes, whether it is carried over interconnected networks or is otherwise proprietary. A service that does not use STIR is not evaluated under it.

Because each attribute describes only the holder's own telephone numbers and grants nothing to anyone else, no authority token is required to assert it. This distinguishes the attributes defined here from the authority recorded in the TNAuthList itself, which is validated at issuance and represents delegated scope. The self-asserted attributes are carried alongside that authority but are semantically separate from it: a relying party MUST NOT interpret any attribute defined here as extending or modifying the certificate's scope of telephone number authority.

This document defines the extension and an extensible IANA registry of attribute types, and seeds the registry with four types: a PASSporT Placement Service (PPS) URI, a do-not-originate indication, a do-not-originate-messaging indication, and a set of authorized originating providers. Future specifications may register additional types that meet the self-assertion criterion defined in the IANA Considerations.

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

The extension is intended for use in end-entity STI certificates {{RFC8226}} and delegate certificates {{RFC9060}} whose TNAuthList authorizes specific telephone numbers. It carries one or more typed attributes, each a self-assertion by the certificate holder about those numbers. In this document, the telephone numbers identified in the certificate's TNAuthList are referred to as the covered telephone numbers.

A TNAuthList {{RFC8226}} may syntactically contain telephone number and telephone number range entries, Service Provider Code (SPC) entries, or a combination of both. The attributes defined in this document describe telephone numbers, and they apply only to the telephone number entries in the TNAuthList; they make no assertion about SPC entries. This extension is therefore intended for certificates whose TNAuthList authorizes telephone numbers. It has no effect in a certificate whose TNAuthList contains only SPC entries, and a producer SHOULD NOT include it in such a certificate.

## The Self-Assertion Principle

Every attribute carried in this extension is a self-assertion by the holder of the certificate about the telephone numbers in the certificate's TNAuthList. An attribute does not require an authority token, and is trustworthy without one, precisely because it describes or constrains only the holder's own telephone numbers and grants no authority, capability, or service to any other party.

The worst outcome an incorrect or malicious self-assertion can produce is degradation of service to the asserting holder's own numbers. An attribute cannot expand the holder's authority, cannot confer authority on a third party, and cannot make a statement about a resource the holder does not control. Where an attribute references another party, as the authorized originating providers attribute references SPCs, that reference is a statement of the holder's own acceptance policy and confers nothing on the referenced party; the referenced party's authority, if any, derives solely from its own credentials.

This principle is the inclusion criterion for the IANA registry defined in {{iana}}. An attribute type qualifies for registration only if it satisfies it. Attributes that would grant authority, enable a capability, or assert facts about resources outside the holder's control do not qualify and belong in a token-gated mechanism instead.

The container guarantees only provenance: every attribute in it was asserted by the validated holder of the telephone numbers in the certificate. Each attribute type defines its own semantics, given in {{attribute-types}} for the types defined here and in the registering specification for types added later.

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
  tnAttr-doNotOriginateMessaging   |
  tnAttr-authorizedOriginators,
  ... }

tnAttr-ppsURIs TN-ATTRIBUTE ::=
  { TYPE PPSURIs IDENTIFIED BY 1 }
tnAttr-doNotOriginate TN-ATTRIBUTE ::=
  { TYPE DoNotOriginate IDENTIFIED BY 2 }
tnAttr-doNotOriginateMessaging TN-ATTRIBUTE ::=
  { TYPE DoNotOriginateMessaging IDENTIFIED BY 3 }
tnAttr-authorizedOriginators TN-ATTRIBUTE ::=
  { TYPE AuthorizedOriginators IDENTIFIED BY 4 }

-- Value syntaxes for the defined types

PPSURIs ::= SEQUENCE SIZE (1..MAX) OF IA5String

DoNotOriginate ::= NULL

DoNotOriginateMessaging ::= NULL

AuthorizedOriginators ::= SEQUENCE SIZE (1..MAX) OF SPC

SPC ::= IA5String

END
~~~~~~~~~~~~~

Note: The numeric module and OID assignments marked TBD are temporary. IANA will allocate the permanent values during RFC publication.

## Criticality

The extension MUST be marked non-critical, following the certificate extension processing rules of {{RFC5280}}, so that a relying party that does not recognize it can still validate the certificate. An attribute that a relying party does not understand, or whose type is not in its supported set, is ignored.

# Attribute Types {#attribute-types}

Each attribute type defines its value syntax and its semantics. A relying party that does not support a given type ignores it. The absence of an attribute is not an assertion: a relying party MUST NOT infer a value, restrictive or permissive, for an attribute that is not present. The restrictive attributes defined here constrain behavior only when present.

Some attributes defined here are presence-only: they carry no value of their own and make their assertion simply by being present in the certificate. In the ASN.1 module such an attribute has the value syntax NULL, which conveys nothing beyond the attribute's presence.

A given type SHOULD appear at most once in the extension. If a type that is defined as single-valued appears more than once, a relying party MUST treat the certificate's attribute set as ambiguous for that type and ignore that type.

## PPS URIs (type 1)

The value is a sequence of one or more absolute URIs {{RFC3986}}, each of which:

- Uses the "https" scheme.
- Identifies a PASSporT Placement Service (PPS) for the covered telephone numbers (for example, "https://pps.example.net").

This attribute declares where PASSporTs for the covered telephone numbers may be published and retrieved out of band {{RFC8816}}. The URI locates the service; the interface a PPS exposes at that location is not specified by this document and may differ between deployments. A value that is not an absolute HTTPS URI MUST cause a relying party to ignore that entry. Producers MAY include multiple URIs to express redundancy or locality; selection among them is a matter of local policy.

This attribute is enabling rather than restrictive: it points a relying party to a service. It satisfies the self-assertion principle because it declares only where the holder publishes PASSporTs for its own numbers and grants nothing to any other party.

## Do Not Originate (type 2)

This attribute is presence-only. When present, it asserts that the covered telephone numbers do not originate calls. A call here is the STIR-attested session {{RFC8224}} established for a telephone number, independent of whether it carries voice or video media. A relying party that obtains this attribute for a number, and then sees a call purporting to originate from that number, has a strong signal that the call is illegitimate and SHOULD treat it as a candidate for blocking according to local policy. Absence of the attribute is not an assertion that the number does originate calls.

Note for Working Group consideration: this attribute treats a call as a single channel regardless of media. Should number-based video calling become established on the telephone network, the working group may wish to consider whether to distinguish voice from video origination through a separately registered attribute, for example a do-not-originate-video declaration, so that a holder could indicate that a number does not originate video calls specifically. This document does not make that distinction; the extensible registry defined in the IANA Considerations accommodates such a future type if a need arises.

## Do Not Originate Messaging (type 3)

This attribute is presence-only. When present, it asserts that the covered telephone numbers do not originate messages. A relying party that obtains this attribute for a number, and then sees a message purporting to originate from that number, has a strong signal that the message is illegitimate and SHOULD treat it as a candidate for blocking according to local policy. Absence of the attribute is not an assertion that the number does originate messages.

This attribute is the messaging counterpart of the do-not-originate attribute (type 2), applying the same presence-only semantics to the messaging channel. The two are parallel per-channel signals: do-not-originate declares that the number does not originate calls, and do-not-originate-messaging declares that it does not originate messages. A number that originates neither carries both.

## Authorized Originators (type 4)

The value is a sequence of one or more SPCs, each identifying a provider that the holder of the telephone numbers authorizes to originate and attest communications from those numbers. A relying party that obtains this attribute, and then sees a communication from the covered number attested by a provider whose SPC is not in the set, has a signal that the origination is unauthorized and SHOULD treat it as a candidate for blocking according to local policy. The providers named apply to origination across the channels the holder's numbers support; this document does not distinguish per-channel authorized originators, and a single set applies to both calls and messaging.

This attribute is a statement of the holder's acceptance policy for its own numbers. It confers no authority on the listed providers: a listed provider's ability to sign for the number derives solely from its own credentials, and a relying party MUST NOT interpret presence in this set as STIR authority of any kind. Likewise, a relying party MUST NOT interpret an SPC in this attribute as extending the certificate's own telephone number authority. Absence of this attribute imposes no origination restriction: the holder has declared no authorized-originator set, and communications may be attested by any provider as far as this attribute is concerned.

The do-not-originate and do-not-originate-messaging attributes each take precedence, on their own channel, over the authorized-originators attribute. Where do-not-originate is present, the covered telephone numbers originate no calls regardless of any authorized-originators value; where do-not-originate-messaging is present, they originate no messages regardless of any authorized-originators value. The authorized-originators attribute therefore governs only origination on channels not closed by a do-not-originate signal.

# Processing Rules

A STIR Authentication Service {{RFC8224}} that holds a certificate carrying a PPS URIs attribute SHOULD publish out-of-band PASSporTs for the covered telephone numbers to one of the indicated PPS endpoints.

A relying party that has obtained a certificate carrying this extension, whether from a signed communication or by retrieval keyed to a telephone number, validates the certificate chain to a trusted STIR trust anchor before relying on any attribute. Having done so, it applies each supported attribute according to that attribute's semantics in {{attribute-types}}. The restrictive attributes, do-not-originate, do-not-originate-messaging, and authorized-originators, are policy signals: a relying party SHOULD treat a communication that does not conform to a present restrictive attribute as a candidate for blocking, subject to local policy.

The defending value of the restrictive attributes is realized only when a relying party can obtain them for a number independently of the communication being evaluated. The cases these attributes are designed to catch, such as an unsigned communication from a do-not-originate number or one attested by an unauthorized provider, are exactly the cases in which the holder's own certificate is not presented on the communication. In those cases the relying party obtains the attributes by retrieving the holder's certificate keyed by telephone number, by a mechanism outside the scope of this document.

# Distribution and Transparency {#conveyance}

Retrieval and distribution of these attributes keyed by telephone number are out of scope for this document and are left to other specifications and to industry practice.

The certificate is the authoritative, signed source of the attribute values. A relying party validates the certificate independently of how it was obtained, so trust in an attribute derives from the certificate rather than from whatever mechanism delivered it.

Transparency logs provide a monitoring mechanism by which distributors can observe the attributes asserted for a telephone number as certificates are issued, and populate the authoritative information that relying parties consult.

A distribution mechanism SHOULD convey the signed certificate together with the distributed information, so that a relying party can verify each attribute directly against the certificate rather than trust the distributor. Where the signed certificate is not conveyed, the relying party depends on the distribution layer for the integrity of the information. This document does not otherwise mandate a distribution mechanism.

# Lifecycle Considerations

The attributes reflect the holder's declarations at the time the certificate was issued. Because they are carried in the certificate, changing an attribute requires issuing a new certificate, and the change takes effect at relying parties only after the new certificate has propagated through the distribution mechanism in use. Attribute currency therefore follows certificate lifetime. Short-lived certificates bound the interval over which a stale declaration remains in use and are RECOMMENDED for this reason. This ties attribute updates to the same validated issuance path as the certificate itself, so that a declaration cannot be forged into the distribution layer.

# Security Considerations

The attributes defined here are self-assertions by the holder of the telephone numbers in the certificate's TNAuthList. Their trust rests on two things: the certificate having been validated at issuance as authorizing the holder for those numbers, and the relying party validating the certificate chain before acting on any attribute. A relying party MUST validate the chain to a trusted STIR trust anchor; the attribute values are not independently signed and derive their integrity from the certificate.

Because each attribute describes or constrains only the holder's own telephone numbers and grants nothing to any other party, the worst outcome of an incorrect or malicious assertion is degradation of service to the asserting holder's own numbers. An attribute cannot expand the holder's authority or confer authority on a third party. In particular, the authorized-originators attribute lists SPCs as a statement of the holder's acceptance policy; it confers no STIR authority on those providers, and a relying party MUST NOT treat presence in that list as authority of any kind, nor as extending the certificate's telephone number scope.

Misissuance is the relevant residual risk: a certificate that carries these attributes without the holder having been properly validated for the listed telephone numbers would carry self-assertions that should not be trusted. The validation that binds a certificate to its telephone numbers is the responsibility of the issuing CA and the certificate profile in use, and is outside the scope of this extension.

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

The registration policy is Specification Required {{RFC8126}}. The designated expert MUST verify that a requested type satisfies the self-assertion criterion: the attribute describes or constrains only the asserting holder's own telephone numbers, grants no authority, capability, or service to any other party, and makes no assertion about a resource the holder does not control. A type that would grant authority, enable a capability, or assert facts about resources outside the holder's control does not qualify and is to be refused, as such a mechanism requires authorization beyond a self-assertion.

The registry is initialized with the following entries, all defined in this document:

| Type | Name | Value Syntax | Reference |
|------|------|--------------|-----------|
| 1 | pps-uris | PPSURIs | (this document) |
| 2 | do-not-originate | DoNotOriginate | (this document) |
| 3 | do-not-originate-messaging | DoNotOriginateMessaging | (this document) |
| 4 | authorized-originators | AuthorizedOriginators | (this document) |

Type value 0 is reserved.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
