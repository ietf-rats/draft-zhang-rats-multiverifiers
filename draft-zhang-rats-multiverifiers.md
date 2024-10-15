---
v: 3

title: 'Handling Multiple Verifiers in the RATS Architecture'
abbrev: Multiple RATS Verifiers
docname: draft-zhang-rats-multiverifiers-latest
area: Security
wg: COSE
kw: Internet-Draft
cat: std
stream: IETF

author:
- name: Jun Zhang
  org: Huawei Technologies France S.A.S.U.
  email: junzhang1@huawei.com
  street: 18, Quai du Point du Jour
  code: '92100'
  city: Boulogne-Billancourt
  country: France
- name: Houda Labiod
  org: Huawei Technologies France S.A.S.U.
  email: houda.labiod@huawei.com
  street: 18, Quai du Point du Jour
  code: '92100'
  city: Boulogne-Billancourt
  country: France
- name: Tieyan Li
  org: Shield Lab, Singapore Research Center, Huawei Technologies
  email: Li.Tieyan@huawei.com
  street: Science Park II., 20 Science Park Road, #03-31
  code: 'Teletech Park'
  country: Singapore
- name: Thanassis Giannetsos
  org: UBITECH Ltd.
  street: Thessalias 8 and Etolias 10
  code: 'GR-15231'
  city: Chalandri,
  country: Greece
- name: Henk Birkholz
  org: Fraunhofer SIT
  abbrev: Fraunhofer SIT
  email: henk.birkholz@sit.fraunhofer.de
  street: Rheinstrasse 75
  code: '64295'
  city: Darmstadt
  country: Germany

normative:
  STD70:
    =: RFC5652
    -: CMS
  RFC3161: TSA
  STD96:
    =: RFC9052
    -: COSE

informative:
  RFC9334: RATS-ARCH

entity:
  SELF: "RFCthis"

--- abstract

In the IETF Remote Attestation Procedures (RATS) architecture, a Verifier accepts Evidence and generates Attestation Results needed by Relying Parties.
This document provides a solution to inconsistent behaviors of the Verifier in the RATS architecture by introducing a mechanism to aggregate Attestation Results collected from multiple Verifiers at the Relying Party while simplifying its policy and operation.

--- middle

# Introduction

The Remote Attestation procedures (RATS) Architecture illustrates an overview of the roles and data-flows between them in {{Section 3 of -RATS-ARCH}}.
{{Section 5 of -RATS-ARCH}} refines the data-flow diagram
by describing two reference models: Passport Model and Background-
check Model.
As discussed in that document, a Verifier accepts
Evidence from Attesters, appraises it using Appraisal Policy, and
generates Attestation Results needed by Relying Parties.

As a single Verifier can introduce a single point of failure, either
as the target of a denial of service attack, due to compromization,
service congestion, or broken Internet connectivity to the Verifier,
relying on a single trusted entity can introduce significant risk.

The architectural pattern of using multiple Verifiers are one
approach to counter such risks.
Nevertheless, it is not guaranteed that different Verifiers generate the same Attestation Results.
Some exemplary reasons include: a) RATS conceptual messages, such as
Reference Values, Endorsements, Appraisal Policy for Evidence for
different Verifiers, are not necessarily aligned, b) certain Verifiers
can be compromised, or c) some Verifiers follow different Appraisal
Policy for Evidence.
This lack of alignment can result in significant issues in both Passport Model and Background-check Model, which is detailed as follows.
The Solution to address the problem of the lack of alignment is detailed in {{sec-three}}.

## Passport Model Cases

Under the Passport Model, an Attester sends Evidence to a Verifier.
The Verifier generates the Attestation Results and sends these back to the Attester.
The Attester conveys the Attestation Results to the Relying Party to proof its trustworthiness.
Fig. 1 and 2 show scenarios that multiple
heterogeneous Verifiers can introduce issues in a Passport
Model based system.

In Fig. 1, if Verifier A is not trusted by the Relying Party,
Attestation Results sent by the Attester can always be rejected
by the Relying Party, which means that the Attester may end up
in a loop of producing and conveying Attestation Evidence and
wait for Attestation Results in vain, repeatedly.

In Fig. 2, Verifier A generates positive Attestation Results
for an Attester, while Verifier B generates negative Attestation
Results for the same Attester.
To trick a Relying Party into putting unjustified trust in the Attester, an Attester can act maliciously by selectively forwarding only Attestation Results from Verifier A and not Verifier B. Such malicious behavior would render a trustworthiness assessment of Attesters by the Relying Party biased or unreliable.

# Two

# Three  {#sec-three}

RFC 3161 {{-TSA}} provides a method to timestamp a message digest to prove that it was created before a given time.

This document defines two new CBOR Object Signing and Encryption (COSE) {{-COSE}} header parameters that carry the TimestampToken (TST) output of RFC 3161, thus allowing existing and widely deployed trust infrastructure to be used with COSE structures used for signing (`COSE_Sign` and `COSE_Sign1`).

## Use Cases

This section discusses two use cases, each representing one of the two modes of use defined in {{modes}}.

A first use case is a digital document signed alongside a trustworthy timestamp.
This is a common case in legal contracts.
In such scenario, the document signer wants to reinforce the claim that the document existed on a specific date.
To achieve this, the document signer acquires a fresh TST for the document from a TSA, combines it with the document, and then signs the bundle.
Later on, a relying party consuming the signed bundle can be certain that the document existed _at least_ at the time specified by the TSA.
The relying party does not have to trust the signer's clock, which may have been maliciously altered or simply inaccurate.

This usage scenario motivates the "Timestamp then COSE" mode defined in {{sec-timestamp-then-cose}}.

A second use case is the notarization of a signed document by registering it at a Transparency Service.
This is common for accountability and auditability of issued documents.
Once a document is registered at a Transparency Service's append-only log, its log entry cannot be changed.
In certain cases, the registration policy of a Transparency Service may add a trustworthy timestamp to the signed document.
This is done to lock the signature to a specific point in time.
To achieve this, the Transparency Service acquires a TST from a TSA, bundles it alongside the signed document, and then registers it.
A relying party that wants to ascertain the authenticity of the document after the signing key has been compromised, can do so by making sure that no revocation information has been made public before the time asserted in the TST.

This usage scenario motivates the "COSE then Timestamp" mode described in {{sec-cose-then-timestamp}}.

## Requirements Notation

{::boilerplate bcp14-tagged}

# Modes of Use {#modes}

There are two different modes of composing COSE protection and timestamping, motivated by the usage scenarios discussed above.

The diagrams in this section illustrate the processing flow of the specified modes.
For simplicity, only the `COSE_Sign1` processing is shown.
Similar diagrams for `COSE_Sign` can be derived by allowing multiple `private-key` boxes and replacing the label `[signature]` with `[signatures]`.

## Timestamp then COSE (TTC) {#sec-timestamp-then-cose}

shows the case where a datum is first digested and submitted to a TSA to be timestamped.

This mode is utilized when the signature should also be performed over the timestamp to provide an immutable timestamp.

A signed COSE message is then built as follows:

* The obtained timestamp token is added to the protected headers,
* The original datum becomes the payload of the signed COSE message.

The message imprint sent to the TSA ({{Section 2.4 of -TSA}}) MUST be the hash of the payload field of the COSE signed object.

## COSE then Timestamp (CTT) {#sec-cose-then-timestamp}

shows the case where the signature(s) field of the signed COSE object is digested and submitted to a TSA to be timestamped.
The obtained timestamp token is then added back as an unprotected header into the same COSE object.

This mode is utilized when a record of the timing of the signature operation is desired.

In this context, timestamp tokens are similar to a countersignature made by the TSA.

# RFC 3161 Time-Stamp Tokens COSE Header Parameters {#sec-tst-hdr}

The two modes described in {{sec-timestamp-then-cose}} and {{sec-cose-then-timestamp}} use different inputs into the timestamping machinery, and consequently create different kinds of binding between COSE and TST.
To clearly separate their semantics two different COSE header parameters are defined as described in the following subsections.

## `3161-ttc` {#sec-tst-hdr-ttc}

The `3161-ttc` COSE _protected_ header parameter MUST be used for the mode described in {{sec-timestamp-then-cose}}.

The `3161-ttc` protected header parameter contains a DER-encoded RFC3161 TimeStampToken wrapped in a CBOR byte string (Major type 2).

To minimize dependencies, the hash algorithm used for signing the COSE message SHOULD be the same as the algorithm used in the RFC3161 MessageImprint.

## `3161-ctt` {#sec-tst-hdr-ctt}

The `3161-ctt` COSE _unprotected_ header parameter MUST be used for the mode described in {{sec-cose-then-timestamp}}.

The message imprint sent in the request to the TSA MUST be either:

* the hash of the signature field of the `COSE_Sign1` message.
* the hash of the signatures field of the `COSE_Sign` message.

In either case, to minimize dependencies, the hash algorithm SHOULD be the same as the algorithm used for signing the COSE message.
This may not be possible if the timestamp token has been obtained outside the processing context in which the COSE object is assembled.

The `3161-ctt` unprotected header parameter contains a DER-encoded RFC3161 TimeStampToken wrapped in a CBOR byte string (Major type 2).

# Timestamp Processing

RFC 3161 timestamp tokens use CMS as signature envelope format.
{{-CMS}} provides the details about signature verification, and {{-TSA}} provides the details specific to timestamp token validation.
The payload of the signed timestamp token is the TSTInfo structure defined in {{-TSA}}, which contains the message imprint that was sent to the TSA.
The hash algorithm is contained in the message imprint structure, together with the hash itself.

As part of the signature verification, the receiver MUST make sure that the message imprint in the embedded timestamp token matches a hash of either the payload, signature, or signature fields, depending on the mode of use and type of COSE structure.

{{Appendix B of -TSA}} provides an example that illustrates how timestamp tokens can be used to verify signatures of a timestamped message when utilizing X.509 certificates.

# Security Considerations

Please review the Security Considerations section in {{-TSA}}; these considerations apply to this document as well.

Also review the Security Considerations section in {{-COSE}}; these considerations apply to this document as well, especially the need for implementations to protect private key material.

The following scenario assumes an attacker can manipulate the clocks on the COSE signer and its relying parties, but not the TSA.
It is also assumed that the TSA is a trusted third party, so the attacker cannot impersonate the TSA and create valid timestamp tokens.
In such a setting, any tampering with the COSE signer's clock does not have an impact because, once the timestamp is obtained from the TSA, it becomes the only reliable source of time.
However, in both CTT and TTC mode, a denial of service can occur if the attacker can adjust the relying party's clock so that the CMS validation fails.
This could disrupt the timestamp validation.

In CTT mode, an attacker could manipulate the unprotected header by removing or replacing the timestamp.
To avoid that, the signed COSE object should be integrity protected during transit and at rest.

In TTC mode, the TSA is given an opaque identifier (a cryptographic hash value) for the payload.
While this means that the content of the payload is not directly revealed, to prevent comparison with known payloads or disclosure of identical payloads being used over time, the payload would need to be armored, e.g., with a nonce that is shared with the recipient of the header parameter but not the TSA.
Such a mechanism can be employed inside the ones described in this specification, but is out of scope for this document.

# IANA Considerations

IANA is requested to add the COSE header parameters defined in {{tbl-new-hdrs}} to the "COSE Header Parameters" registry {{!IANA.cose_header-parameters}}.

| Name | Label | Value Type | Value Registry | Description | Reference |
| `3161-tcc` | TBD1 | bstr | - | RFC 3161 timestamp token | {{&SELF}}, {{sec-tst-hdr-ttc}} |
| `3161-ctt` | TBD2 | bstr | - | RFC 3161 timestamp token | {{&SELF}}, {{sec-tst-hdr-ctt}} |
{: #tbl-new-hdrs align="left" title="New COSE Header Parameters"}

--- back

# Acknowledgments
{:unnumbered}

The editors would like to thank
Carl Wallace,
Leonard Rosenthol,
Michael B. Jones,
Michael Prorock,
Orie Steele,
and
Steve Lasker
for their reviews and comments.
