---
v: 3

title: 'Handling Multiple Verifiers in the RATS Architecture'
abbrev: Multiple RATS Verifiers
docname: draft-zhang-rats-multiverifiers-latest
area: Security
wg: RATS
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
  email: agiannetsos@ubitech.eu
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
  RFC9334: RATS-ARCH
  I-D.voit-rats-trustworthy-path-routing: I-D.voit-rats-trustworthy-path-routing

informative:
  RFC9397: 10.17487/RFC9397
  I-D.liu-nasr-requirements: I-D.liu-nasr-requirements
  TAP:
    author:
      org: Trusted Computing Group
    title: TCG Trusted Attestation Protocol (TAP) Use Cases for TPM Families 1.2 and 2.0 and DICE
    date: 2019-11-05
    target: https://trustedcomputinggroup.org/wp-content/uploads/TCG_TNC_TAP_Use_Cases_v1r0p35_published.pdf
    seriesinfo: 1.0

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

         .-------------.
         |             | Compare Evidence
         |  Verifier A | against appraisal policy
         |             |
         '--------+----'
             ^    |
    Evidence |    | Attestation
             |    | Result
             |    v
         .---+--------.              .-------------. Compare
         |            +------------>X|             | Attestation
         |  Attester  | Attestation  |   Relying   | Result against
         |            | Result       |    Party    | appraisal
         '------------'              '-------------' policy

   Figure 1: Passport Model with Verifier A not trusted by Relying
Party.

         .-------------.
         |             | Compare Evidence
         |  Verifier A | against appraisal policy
         |             |
         '--------+----'
             ^    |
    Evidence |    | Attestation
             |    | Result A (positive)
             |    v
         .---+--------.              .-------------. Compare
         |            +------------->|             | Attestation
         |  Attester  | Attestation  |   Relying   | Result against
         |            | Result A     |    Party    | appraisal
         '---+--------'              '-------------' policy
             |    ^
    Evidence |    | Attestation
             |    | Result B (negative)
             |    |
             V    |
         .--------+----.
         |             | Compare Evidence
         |  Verifier B | against appraisal policy
         |             |
         '-------------'

   Figure 2: Passport Model with cheating Attester

## Background-check Model cases
   Under the Background-check Model, an Attester sends Evidence to a
Verifier via a Relaying Party, and the Verifier generates the
Attestation Results and sends them back to the Relying Party.

   Fig. 3 and 4 show scenarios where multiple heterogeneous Verifiers
introduce potential issues in a Background-check Model.

   In Fig. 3, even if a Verifier is trusted by a Relying Party,
there is no assurance that it is working as intended and only does what
it is supposed to do and nothing else. If multiple Verifiers exist,
neither Evidence might reach all Verifiers nor all Attestation Results
might reach the Relying Party due to failing conveyance mechanisms, or
due to the Verifier itself being compromised or malfunctioning.,
or hardware problems.

  In Fig. 4, a Relying Party is able to alternate between Verifiers.
When these Verifiers are heterogeneous though, a Relying Party might
receive different or conflicting Attestation Results from them, which
means the trustworthy assessment of the Attester can rely (and fail)
on a specific selection of Verifiers made by at the Relying Party side.

                                    .-------------.
                                    |             | Compare Evidence
                                    |   Verifier  | against
                                    |             | appraisal
                                    |      x(2)   | policy
                                    '--------+----'
                                         ^   x(3)
                                Evidence |   | Attestation
                                         x(1)| Result
                                         |   v
       .------------.               .----|--------.
       |            +-------------->|---'         | Compare
       |            |               |             | Attestation
       |  Attester  |   Evidence    |     Relying | Result against
       |            |               |      Party  | appraisal policy
       '------------'               '-------------'

   Figure 3: A Background-Check Model where a Verifier is not available
because of 1) a Relying Party not being reachable by the Verifier, 2)
a malfunction of the Verifier.

                                    .-------------.
                                    |             | Compare Evidence
                                    |   Verifier  | against
                                    |      A      | appraisal
                                    '--------+----' policy
                                         ^   |
                                Evidence |   | Attestation
                                         |   | Result (positive)
                                         |   v
       .------------.               .----|--------. Compare
       |            +-------------->|---'         | Attestation
       |  Attester  |   Evidence    |     Relying | Result against
       |            |               |      Party  | appraisal policy
       '------------'               '----+--------'
                                         |   ^
                                Evidence |   | Attestation
                                         |   | Result (negative)
                                         v   |
                                    .--------+----.
                                    |             | Compare Evidence
                                    |   Verifier  | against
                                    |      B      | appraisal
                                    '-------------' policy

    Figure 4: A Background-Check Model conveying conflicting Attestation
Results originating from multiple Verifiers.

# Terminology

   The following terms are imported from [RFC9334]: Attester, Evidence,
Endorsement, Reference value, Appraisal Policy, Relying Party, and
Verifier. Also imported are the time definitions time(VG), time(NS),
time(EG), time(ER), time(RG),time(RX), and time(OP) from that
document's Appendix A.

   New relevant Events over Time:
   time(AG): the time at the event that the Attestation Results for
the same attester is aggregated.

# Handing Multiple Verifiers {#sec-three}


In this section, we follow the attestation data-flow based on the
Background-Check Model, to support robust aggregation of the
Attestation Results in an environment with many Verifiers that may
be heterogeneous. 
Here heterogeneous means that the Verifiers may
generate different Attestation Results. 
In terms of resilience, these Verifiers are not enabled to be replaced
by a conceptual single (proxy) Verifier as this single Verifier may
still has the availability issue.
The Verifier Manager is introduced into the attestation system 
to help the Relying Party choosing Verifiers that are expected 
to follow the same standard, to avoid the aggregation of
Attestation Results that following different standards. 


## Aggregation of Attestation Results from Multiple Verifiers

Fig. 5 below is a sequence diagram which updates Fig. 14 in
[RFC9334] to support the aggregation of Attestation Results from
multiple Verifiers in a Background-check Model. The nonce is
generated by the Relying Party, in place of each Verifier, so as to
reduce the amount of Evidence generated. The aggregation method
implemented by the Relying Party is out of scope of this draft. For
example, the majority vote could be viewed as a possible solution.
The Attestation Results can be aggregated to help the Relying Party
making the decision, or be aggregated as a new Attestation Result,
which is decided by the Relying Party itself. 

    .---------.    .--------. .--------.    .--------.          .---.
    | Attester|    |Verifier| |Verifier|    |Verifier|          | RP|
    |         |    |    1   | |   2    |    |    k   |          |   |
    '---------'    '--------' '--------'    '--------'          '---'
         |              |         |            |                  |
      Time(VG_a)        ~         ~            ~                  ~
         |              |         |            |                  |
         |<----Nonce---------------------------------------time(NS_r)
      Time(EG_a)        |         |            |                  |
         |              |         |            |                  |
         |-----Evidence{Nonce}----------------------------------->|
         |             |                                time(ER_r_1)
         |             |<-----Evidence{Nonce}---------------------|
         |             |         |            |         time(ER_r_2)
         |      time(RG_v_1)     |<-Evidence{Nonce}---------------|
         |             |  time(RG_v_2)        |         time(ER_r_k)
         |             |         |            |<-Evidence{Nonce}--|   
         |             |         |       time(RG_v_k)             |
         |             |--Attestation Result--------------------->|
         |             | {time(RX_v_1)-time(RG_v_1)}              |
         |             |         |----Attestation Result--------->|
         |             |         |  {time(RX_v_2)-time(RG_v_2)}   |
         |             |         |             |--------AR------->|
         |             |         | {time(RX_v_k)-time(RG_v_k)}    |
         |             |         |             |         time(AG_r)
         |             |         |             |         time(OP_r)

 Figure 5: Background-Check Model with the support of the aggregation
of Attestation Results from multiple Verifiers.

## Verifier Manager

Manually configuring the Verifiers in each Relying Party is not well
adapted to the changing of the network environment. As there is no
guarantee of the availability and consolidation of these Verifiers
in the long term. We introduce a new entity in RATS architecture,
which is the Verifier manager, to address these issues. As shown in
Fig. 6, after configuring the anchor seed Verifiers in the Relying
Party, which is typically a small set of trusted Verifiers by the
Relying Party. The Relying Party can communicate with the Verifier
manager with this list of Verifiers, in together with certain
parameters n, for example, the number of Verifiers that it expects
to collect Attestation Results from. The Verifier manager matches
this list with its local database of the groups of Verifiers, find
the groups of Verifiers that behave most close to the majority of
the Verifiers in this list, and picks n Verifiers out of it. Then
the Verifier manager sends these n Verifiers back to the Relying
Party, as its recommended Verifiers. In such a way, each Relying
Party can flexibly configure its policy for the trusted Verifier.
Notice that, the Verifiers in the same group are expected to follow
the same golden measurement, that is, they are expected to generate
the same Attestation Results when they receive the same Evidences. 
The example are the Verifiers that are deployed by the same
company or the alliance following the same standard. Here
the same for Evidences and Attestation Results are
in the sense of semantic, that is, they can be wrapped
in different formats, cwt or jwt for example, but the
content itself are the same. When a Relying Party receives 
certain minority attesation results from certain Verifiers, 
it can inform the Verifier Manager this incidence and the verifier
manager will reduce the reputation of these verifiers, and reduce 
the probability to recommend these Verifiers to relying parties. 
So in the long run, the misbehaved Verifiers will be punished. 

    .---------.   .----------.     .----------.     .--------------.
    | Endorser|  | Reference |     | Verifier |     | Relying Party|
    '+--------'  | Value     |     | Owner    |     | Owner        |
     |           | Provider  |     '----+-----'     '-----+--------'
     |           '------+----'          |                 |
     |                  |               |                 |
     | Endorsements     | Reference     | Appraisal       | Appraisal
     |                  | Values        | Policy for      | Policy for
     |                  |               | Evidence        | Attestation
     '-----------.      |               |                 | Results
                   |    |               |                 |
                   v    v               v                 |
                 .-------------------------.              |
         .------>|         Verifier        +------.       |
        |        '-------------------------'      |       |
        |                                         |       |
        | Evidence                    Attestation |       |
        |                             Results     |       |
        |                                         |       |
        |                                         v       v
    .-----+----.                                .---------------.
    | Attester |                                | Relying Party |
    '----------'                                '---------------'
                                                    |       ^
                             Anchor seed Verifiers, |       | Recommended
                             parameter              |       | Verifiers
                                                    |       |
                                                .------------------.
                                                | Verifier Manager |
                                                '------------------'

                          Figure 6: Revised Data Flow based RFC9334

# Use cases

 This Section illustrates some use cases that can benefit from an
architecture that takes multiple Verifiers into account.

   Use case 1: Intent-driven Attestation Classification for Data Center
Network Solutions

   Need: Establishment of trust in a complex data center environment
comprising multiple VMs instantiated on heterogeneous CPU
architectures \
   Solution: Attestation Verification Service based on a harmonized set
of components to be leveraged by multiple Verifiers. The Evidence
from the attester is only needed to be generated once for each
attestation process, and the Evidence is forwarded between Verifiers to
reach the consensus of the trustworthy judgement of the Attester.\
   Source: TCG Trusted Application Protocol (TAP) Use Cases [TAP]

   Use case 2: Enhancing TEE Device Interface Security Protocol (TDISP)
   
   Need: Enhance Trusted Execution Environment Provisioning (TEEP)
Architecture with TEE-I/O capabilities for the direct verification
assignment of specific system characteristics to targeted (remote)
Verifiers \
   Solution: Harmonized Trusted Computing Base to Achieve Secure
interfaces and Key Management with Multiple Verifiers attesting
different device properties \
   Source: [RFC9397] on TEEP Architecture  	

   Use case 3: Intra- and Inter-Domain Trusted Path Routing
   Need: Trustworthiness Assessment of routing nodes (Attesters)
against multiple Verifiers (Control Plane Orchestrators) residing in
different network administrative domain \
   Solution: Verification of multiple attestation formats supporting
reference integrity manifest with constrained disclosure and resilience
to the failure of certain Verifiers.\
   Source:  Trusted Path Routine  [I-D.voit-rats-trustworthy-path-routing],
network attestation for secure routing [I-D.liu-nasr-requirements]

   Use case 4: network endpoint assessment \
   Need: provide resilience in the attestation service \
   Source: use case from [RFC9334]

   Use case 5: Confidential Data Protection \
   Need: avoid single Verifier corruption, which leads to the leakage
of data privacy. \
   Source: use case from [RFC9334]

# Security Consideration
  [TBD]

# IANA Considerations
  [TBD]




