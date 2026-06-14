---
###
# PASSporT Placement Service (PPS) URI Certificate Extension for STIR/SHAKEN Certificates
###
title: "PASSporT Placement Service (PPS) URI Certificate Extension for STI Certificates"
abbrev: "PPS URI Certificate Extension"
category: std

docname: draft-sliwa-stir-pps-ext-00
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
venue:
  group: "Secure Telephone Identity Revisited"
  type: "Working Group"
  mail: "stir@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/stir/"
  github: "appliedbits/draft-sliwa-stir-pps-ext"
  latest: "https://github.com/appliedbits/draft-sliwa-stir-pps-ext"

author:
 -
    fullname: Rob &#x015A;liwa
    organization: Somos Inc.
    email: robjsliwa@gmail.com
 -
    fullname: Chris Wendt
    organization: Somos Inc.
    email: chris@appliedbits.com

normative:
  RFC3986:
  RFC5280:
  RFC8224:
  RFC8225:
  RFC8226:
  RFC8816:
  RFC9060:
  I-D.ietf-stir-certificate-transparency:
  I-D.ietf-stir-servprovider-oob:
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

--- abstract
This document specifies a non-critical X.509 v3 certificate extension that conveys the HTTPS URI of a PASSporT Placement Service (PPS) associated with the telephone numbers authorized in a STIR Certificate. The extension enables originators and verifiers of STIR PASSporTs to discover, with a single certificate lookup, where Out-of-Band (OOB) PASSporTs can be retrieved. The mechanism only provides a new way to discover the URI of a PPS endpoint and is fully backward compatible with existing STIR certificates and OOB APIs.
--- middle

# Introduction

The STIR (Secure Telephone Identity Revisited) framework provides a means of cryptographically asserting the identity of the calling party in a telephone call by using PASSporTs carried in SIP requests, as defined in {{RFC8224}} and {{RFC8225}}. To support deployment in environments where SIP Identity headers may be removed or are not end-to-end transmittable, such as in non-IP or hybrid telephony networks, the STIR Out-of-Band (OOB) mechanism was introduced in {{RFC8816}}. In OOB scenarios, PASSporTs are published to a PASSporT Placement Service (PPS) where they may be retrieved independently of the SIP signaling path.

Note: The service defined in {{RFC8816}} is named the Call Placement Service (CPS). This document refers to it as the PASSporT Placement Service (PPS). The two terms denote the same architectural role. The PPS name is used to avoid acronym collision with Certification Practice Statement (CPS) as used in {{RFC8226}} and certificate policy practice, and because the service places and serves PASSporTs and is not limited to telephone calls.

To enable discovery of the appropriate PPS for a given telephone number or SPC, this document defines a certificate extension that binds a PPS URI to the identity resources listed in the TNAuthList of the STI certificate. This PPS URI extension provides a verifiable association between a number resource and its corresponding PPS, enabling relying parties to discover PPS endpoints by observing STI Certificate Transparency (STI-CT) logs defined in {{I-D.ietf-stir-certificate-transparency}}.

This specification defines the syntax and semantics of the PPS URI certificate extension, describes how it is encoded in {{X.509}} certificates also defined in {{RFC5280}}, and outlines validation procedures for Certification Authorities and relying parties. This extension is intended to be used in conjunction with existing STIR certificates defined in {{RFC8226}} and delegate certificates defined in {{RFC9060}} infrastructure, and supports enhanced transparency and automation in OOB PASSporT routing.

## Relationship to Other Specifications

This document defines the certificate extension data format for embedding PPS URIs in STIR certificates. It is designed to work within the broader STIR Out of Band ecosystem as follows:

- {{RFC8816}} defines the OOB architecture and the PASSporT Placement Service concept, which it names the Call Placement Service (CPS).
- {{I-D.ietf-stir-servprovider-oob}} describes a service-provider-specific OOB deployment model and identifies, in its Section 4, the possibility of embedding placement service information directly in STIR certificates. This document also defines a discovery mechanism built on CT log monitoring that consumes this extension.
- {{I-D.ietf-stir-certificate-transparency}} defines STI Certificate Transparency logs that can be used to publish and discover certificates containing this extension.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The id-pe-ppsURI Certificate Extension

This {{X.509}} extension is non-critical, applicable only to end-entity certificates, and defined with ASN.1 {{X.680}} {{X.681}} {{X.682}} {{X.683}} later in this section.

This extension is intended for use in end-entity STI certificates {{RFC8226}} and delegate certificates {{RFC9060}} that include TNAuthList values authorizing the use of specific telephone numbers or Service Provider Codes (SPCs). The PPS URI extension provides a means for the certificate holder to declare the HTTPS endpoint of a PASSporT Placement Service (PPS) defined in {{RFC8816}} that can be used to publish or retrieve PASSporTs for the covered resources.

The presence of this extension allows relying parties to discover the PPS associated with a given telephone number without relying on static configuration or bilateral agreements. This facilitates scalable and verifiable Out-of-Band PASSporT delivery as defined in {{RFC8816}}, using information already published in publicly logged STI certificates.

The extension is encoded as a sequence of IA5Strings containing absolute HTTPS URIs and is identified by an object identifier (OID) assigned in the PKIX id-pe arc. Additional details about the encoding, semantics, and validation rules for the PPS URI list are defined in the sections below.

## ASN.1 Module Syntax

The extension ASN.1 module is defined as follows:

~~~~~~~~~~~~~
PPS-CERT-EXTENSION
  { iso(1) identified-organization(3) dod(6) internet(1)
    security(5) mechanisms(5) pkix(7) id-mod(0)
    id-mod-cmw-collection-extn(TBD0) }

DEFINITIONS EXPLICIT TAGS ::=
BEGIN

IMPORTS
  EXTENSION
  FROM PKIX-CommonTypes-2009  -- RFC 5912
    { iso(1) identified-organization(3) dod(6) internet(1)
      security(5) mechanisms(5) pkix(7) id-mod(0)
      id-mod-pkixCommon-02(57) } ;

-- PPS URIs Certificate Extension

ext-PPSURIs EXTENSION ::= {
  SYNTAX PPSURIs
  IDENTIFIED BY id-pe-ppsURI }

-- PPS URI Extension Syntax

id-pe OBJECT IDENTIFIER ::=
  { iso(1) identified-organization(3) dod(6) internet(1)
    security(5) mechanisms(5) pkix(7) 1 }

id-pe-ppsURI OBJECT IDENTIFIER ::= { id-pe TBD1 }

PPSURIs ::= SEQUENCE SIZE (1..MAX) OF IA5String

END
~~~~~~~~~~~~~

Certificates containing a PPSURI that is not an absolute HTTPS URI as defined in {{RFC3986}} MUST be considered invalid by relying parties.

Note: The numeric assignment TBD is temporary. IANA will allocate a permanent arc under "PKIX SubjectPublicKeyInfo Certificate Extensions" during RFC publication.

## Extension Semantics

Each IA5String value in the sequence MUST be an absolute URI {{RFC3986}} that:

- Uses the "https" scheme.
- Identifies the root of the PPS HTTPS API interface (e.g., "https://pps.example.net/oob/v1").

The sequence MUST contain at least one URI. Producers MAY include multiple URIs to provide resiliency or geographic locality information.

## Criticality

The extension MUST be marked non-critical so that implementations that do not understand it can still validate the certificate.

## Processing Rules

- A STIR Authentication Service (AS), defined in {{RFC8224}}, that holds a Certificate containing id-pe-ppsURI SHOULD publish OOB PASSporTs to the indicated PPS.
- A STIR Verification Service (VS), defined in {{RFC8224}} that receives a PASSporT signed by such a certificate MAY derive the PPS endpoint by reading the extension, or MAY query an external discovery directory that is populated by monitoring the STI-CT logs.
- If the extension and an external directory disagree, the resolution is a matter of local policy.
- A STIR Verification Service (VS) that receives a SIP request without an in-band PASSporT MAY use the calling party's identity (e.g., from the From or P-Asserted-Identity headers) to query a local directory or STI-CT monitor to locate the associated certificate. Once the certificate is located, the VS can extract the PPS URI extension to discover the PASSporT Placement Service and retrieve the PASSporT.

# Use with Out-of-Band

Figure 1 shows the message flow when the extension is present:

~~~~~~~~~~~~~
  +------------+  (1) Request Delegate Cert   +---------+
  | Enterprise |  ---- w/ PPS URI ext ------> | CA / CT |
  | (AS/OOB-AS)|                              +---------+
  +------------+                                   |
       |    |                                      |
       |    | (2a) SIP INVITE                      |
       |    |      (may or may not carry PASSporT) |
       |    v                                      |
       |  [Terminating Network]                    |
       |                                           |
       |  (2b) POST PASSporT to PPS                |
       +------------------------------------------>|
                                              +---------+
                                              |   PPS   |
       +------- (3) GET PASSporT from PPS --->+---------+
       |
  +---------+    (4) Monitor CT logs     +-----------+
  |   VS    |<---------------------------| CT Monitor|
  +---------+                            +-----------+
Figure 1
~~~~~~~~~~~~~

1. The enterprise obtains an STI certificate (either a STIR certificate or delegate certificate) containing the PPS URI. The CA submits the certificate to STI-CT.
2a. The AS sends a SIP INVITE toward the terminating network. The INVITE may or may not carry an in-band PASSporT.
2b. The OOB-AS submits the PASSporT to the PPS indicated by the extension.
3. The terminating VS retrieves the PASSporT from the PPS.
4. The VS (or a monitoring component) discovers the PPS URI by monitoring CT logs for certificates containing the extension.

Note: Although the figure depicts an enterprise scenario, the same mechanism applies when a service provider holds an STI certificate with the PPS URI extension.

# Operational Considerations

- Logging: CAs issuing certificates with id-pe-ppsURI MAY submit the certificate to STI-CT logs.
- Migration overlap: When changing a PPS endpoint, operators SHOULD ensure that both the old and new PPS URIs are operational during the transition. Specifically, the operator SHOULD issue a new certificate containing the updated PPS URI, confirm its presence in CT logs, and only then decommission the old PPS endpoint. The old certificate's PPS URI SHOULD remain functional until the old certificate expires or is revoked.
- Propagation delay: Relying parties that discover PPS URIs through CT log monitoring will experience a delay between certificate issuance and PPS URI availability. This delay depends on CT log inclusion time and monitor polling intervals. Operators SHOULD account for this delay when planning PPS migrations by maintaining the old endpoint for a period beyond the expected propagation time.

# Security Considerations

The PPS URI certificate extension introduces a mechanism for associating telephone number resources with PPS endpoints through STI certificates. The following considerations apply:

- Misuse or Misissuance: A malicious or misconfigured entity may include a PPS URI in a certificate without authorization for the corresponding TNAuthList resources. Certification Authorities (CAs) MUST validate that the entity requesting the certificate has authority over the listed numbers or SPCs before issuing the certificate.
- URI Integrity: The PPS URI is not digitally signed independently of the certificate. Relying parties MUST validate the entire certificate chain before relying on the URI.
- Certificate Expiry and Revocation: PPS URI information may become outdated due to certificate expiration or revocation. Relying parties SHOULD evaluate certificate validity and revocation status when interpreting PPS mappings.
- Log Availability and Monitoring: Relying parties that depend on CT log monitoring for PPS discovery SHOULD monitor multiple trusted logs to ensure timely detection of PPS declarations and prevent omission attacks.
- Information Exposure: The publication of PPS URIs in publicly logged certificates may reveal deployment metadata. This exposure is consistent with existing STIR delegate certificate practices and does not introduce additional privacy risk beyond what is already present in TNAuthList usage.

# IANA Considerations

IANA is requested to assign a new object identifier (OID) for the PPS URI certificate extension in the "PKIX Extension Registry" as follows:

- Name: id-pe-ppsURI
- OID: to be assigned
- Description: Certificate extension for specifying a PASSporT Placement Service (PPS) URI for STIR Out-of-Band PASSporTs
- Reference: [RFC THIS]

--- back

# Changes from draft-sliwa-stir-cert-cps-ext
{:numbered="false"}

This is an editorial section that tracks changes across revisions and is intended to be removed before any eventual publication.

This document was previously published as draft-sliwa-stir-cert-cps-ext and continues that work. The principal change is a rename of the placement service and the extension it identifies:

- The service previously called a Call Placement Service (CPS) is renamed to PASSporT Placement Service (PPS). This avoids the acronym collision with Certification Practice Statement (CPS) as used in {{RFC8226}} and certificate policy practice, and reflects that the service places and serves PASSporTs and is intended to apply to applications beyond telephone calls.
- The certificate extension and its ASN.1 and OID identifiers are renamed accordingly: id-pe-oobURI becomes id-pe-ppsURI, the OOBURIs type becomes PPSURIs, and the OOB-CERT-EXTENSION module becomes PPS-CERT-EXTENSION. The on-the-wire OID value remains to be assigned by IANA and is unaffected by the rename.
- The document title, abbreviation, and filename are updated to reflect the new name.

There are no other technical changes from draft-sliwa-stir-cert-cps-ext.

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
