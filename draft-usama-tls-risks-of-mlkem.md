---
title: "Analysis of Hybrid Key Exchange and Standalone ML-KEM in TLS 1.3"
category: info

docname: draft-usama-tls-risks-of-mlkem-latest
submissiontype: independent  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: false
v: 3
workgroup: "Independent Submission Stream"
venue:
  github: "muhammad-usama-sardar/risks-of-mlkem"
  latest: "https://muhammad-usama-sardar.github.io/risks-of-mlkem/draft-usama-tls-risks-of-mlkem.html"

author:
 -
    fullname: "Muhammad Usama Sardar"
    organization: TU Dresden, Germany
    email: "muhammad_usama.sardar@tu-dresden.de"

normative:
  NistFips203: DOI.10.6028/NIST.FIPS.203
  I-D.ietf-tls-mlkem:
  I-D.ietf-tls-rfc8446bis:
  TLS-FATT:
     author:
        org: IETF TLS WG
     title: TLS FATT Process
     target: https://github.com/tlswg/tls-fatt
     date: June 2025

informative:
  I-D.usama-tls-fatt-extension:
  I-D.pwouters-crypto-current-practices:
  I-D.barnes-tls-this-could-have-been-an-email:
  rfc3552:
  I-D.ietf-tls-ecdhe-mlkem: hybrid
  I-D.ietf-tls-hybrid-design-09:
  I-D.ietf-tls-hybrid-design:
  FATT-CHANCE:
    target: https://eprint.iacr.org/2026/1147
    title: "FATT Chance: On the Robustness of Standalone and Hybrid ML-KEM Key Exchange in TLS 1.3"
    date: 2026
    seriesinfo: "Cryptology ePrint Archive, Report 2026/1147"
    author:
    -
      ins: N. Kobeissi
      name: Nadim Kobeissi
      org: Symbolic Software

...

--- abstract

The memo presents *symbolic* and *computational* analysis of hybrid key exchange and standalone ML-KEM.
This memo also maps out the relevant technical facets surrounding quantum-resistant key exchange and provides some preliminary discussion to help developers and policymakers make informed choices.
Our observation is that hybrid key exchange is preferable over standalone ML-KEM until a powerful CRQC exists which breaks **all** the bits of pre-quantum.
Finally, it offers minimal implementation guidance for hybrid key exchange.
This memo is not a standard nor has it been shown to have consensus of the IETF community.

--- middle

# Introduction

Readers are assumed to be familiar with {{NistFips203}}, {{I-D.ietf-tls-rfc8446bis}}, and {{I-D.ietf-tls-mlkem}}. Please note that the memo has currently several hyperlinks.

## Objectives

The memo serves three objectives:

* Summary of formal methods (symbolic and computational) works for hybrid key exchange and standalone ML-KEM
* Identifying the underlying technical facets and common complexities of the deployment debate to provide clarity
* Minimal implementation guidance for hybrids

### Summary of Formal Methods Works

The memo covers the formal methods for the security considerations of {{I-D.ietf-tls-mlkem}}.
This includes *symbolic* and *computational* analysis (to be interpreted as in [SoK](https://eprint.iacr.org/2019/1393.pdf)) of integration of standalone ML-KEM in the context of TLS.
Specifically, it covers the formal analysis {{FATT-CHANCE}} in ProVerif on the potential issue of asymmetry.
The analysis confirms that asymmetry is not a problem.

### Presenting Different Technical Facets

The later sections also identify technical facets that affect deployment reasoning but are not fully resolved by the formal analysis alone.
This explains why the memo discusses issues such as break timing, residual pre-quantum security, deployment cost, patents, and implementation behavior after presenting the formal-methods result.

### Minimal Implementation Guidance for Hybrids
The implementation concern is not only whether ML-KEM is secure as a
primitive, but also whether a TLS deployment can show that both hybrid
components were validated, transcript-bound, and fail-closed under the
negotiated group.

## Motivation
{: #sec-mot }

{{rfc3552}} requires to document the risks in the security considerations.
To support those requirements for {{I-D.ietf-tls-mlkem}}, this memo aims to formally study the security of standalone ML-KEM in TLS 1.3.

## Intuition
Leaking out the ECDHE key from hybrid key exchange should downgrade the security to the level of a standalone ML-KEM.
Therefore, hybrid key exchange is *in general* more secure, unless:

* ECDHE is fully broken, in which case it still falls equivalent to standalone ML-KEM,
* in the *hypothetical* scenario that there is an implementation bug in the ECDHE part which is triggered only in composition. We have not yet seen any concrete evidence of such a scenario on the list.

We believe that *in general*:

~~~
1. Migration from ECDHE to hybrid key exchange is security improvement.
2. Migration from hybrid key exchange to standalone ML-KEM is security
regression, unless CRQC exists to break all ECC keys.
~~~


### Expected Learning
We believe formal methods can provide additional value for security considerations of this memo in order to maintain the high cryptographic assurance of TLS.

~~~
Since we have no guarantee on whether ECDHE will break before ML-KEM,
it seems appropriate to do thorough cryptographic analysis.
We believe the Harvest Now, Decrypt Later (HNDL) attack applies
equally well to standalone ML-KEM.
~~~

An adversary can record all traffic and decrypt it later if ML-KEM is broken.
The opinions of the community on this matter vary from "ML-KEM is secure" to "ML-KEM is probably already secrectly broken."
Formal methods can operate under the assumption that ML-KEM is secure, and focus on the integration of ML-KEM in TLS under this assumption.

* As an example, formal methods can help justify design choices, such as the preference for hybrid key exchanges.
It can also help identify all the assumptions under which the properties hold.
* As a relevant data point in the context of standardization, LAKE WG has done formal analysis for EDHOC-PSK with KEM ([ref](https://mailarchive.ietf.org/arch/msg/lake/2XGOI9OCwylJUfSCasvvwM2FXmw/)).
* *Computational* analysis (cf. [SoK](https://eprint.iacr.org/2019/1393.pdf)) -- using tools such as CryptoVerif -- seems like a reasonable approach to ensure security of ML-KEM in TLS, such as binding shared secret `ss` to the TLS transcript hash.

~~~
We believe that the focus of symbolic analysis ought to be on the
*integration* details (transcript binding, key schedule, agreement)
for standalone ML-KEM in the context of TLS, rather than the
*primitive* itself.
~~~

# Conventions and Definitions

{::boilerplate bcp14-tagged}

* Symbolic analysis: see [SoK](https://eprint.iacr.org/2019/1393.pdf)
* Computational analysis: see [SoK](https://eprint.iacr.org/2019/1393.pdf)
* Standalone ML-KEM refers to {{I-D.ietf-tls-mlkem}}.
* Hybrid key exchange refers to {{-hybrid}} and {{I-D.ietf-tls-hybrid-design}}.

We believe that symbolic and computational models are complementary and not a substitute of each other.

## Key Exchange and Key Encapsulation
{: #sec-proof-break }

In traditional key exchange (DHKE), both endpoints send their public key shares g<sup>x</sup> and g<sup>y</sup> to derive a shared secret g<sup>xy</sup>.

In key encapsulation, there is essentially only one endpoint (say client) which generates the key pair `(dk,ek)` where `dk` represents the *secret decapsulation key* and `ek` represents the *public encapsulation key*.
In a KEM, only one of the endpoints (client in above example) sends the public encapsulation key `ek` and the peer (server) sends a ciphertext `ct`.


# Computational Analysis

## Hybrids
{: #sec-tech-rat }

Technically, a proof of {{I-D.ietf-tls-hybrid-design-09}} is done in the computational model using CryptoVerif (cf. [ref](https://bblanche.gitlabpages.inria.fr/publications/BlanchetJacommeCSF24.pdf)). As per list discussion, it appears that the proof applies to the latest version of the spec {{I-D.ietf-tls-hybrid-design}}, as there seem to be no substantive changes from the perspective of formal proof.

## Standalone ML-KEM

Some existing computational analysis for standalone ML-KEM in TLS include [this](https://eprint.iacr.org/2021/844) and [this](https://eprint.iacr.org/2024/1360).
Both are based on pen-and-paper (computational) proofs.
At the symbolic level, some analysis -- such as [this](https://eprint.iacr.org/2022/1111.pdf) for KEMTLS in Tamarin -- exists. In our understanding, both client and server encapsulate, which may bring the symmetry.


# Symbolic Analysis
For brevity, we omit other assumptions in the properties below and focus on the difference.
This assumes hybrid constructor to be secure.

## Minimum Viable Modeling
{: #sec-model-analyze }

Based on the discussion on list, simply replacing ideal DHKE by ideal ML-KEM in the formal model is not very useful. We ought to focus on the more security-critical questions about integration of ML-KEM in TLS.
We present a few high-level observations to consider for security considerations of {{I-D.ietf-tls-mlkem}}:

* The model ought to consider that any agent could have initiated the TLS, rather than assigning the agents with static roles of client and server in the model. When agents are assigned non-static roles, it would be interesting to see whether the asymmetry issue becomes visible in some property. We consider it very critical for security considerations of {{I-D.ietf-tls-mlkem}} and this is the key point of this memo.
* Different failure modes proposed on list can be modeled.
* A large part of the problem is the careful investigation of what to model, under what threat model, under what system model, under what implementation scenarios etc. We believe some of this is important for security considerations of {{I-D.ietf-tls-mlkem}}.
* It will be interesting to see some analysis about any subtle cases where hybrid key exchange in TLS is *not* at least as good as standalone ML-KEM in TLS, since hybrid key exchange is the de facto industry standard.
* We believe brainstorming about some robustness (vs. security) properties would also be useful. Even if the security properties hold, does standalone ML-KEM make side-channel leakage easier? This might be a valuable consideration for the implementers.
* Analysis may be helpful to ensure that the changes -- such as the removal of hash function (cf. Appendix C.1, bullet 3 in {{NistFips203}}) -- from Kyber to ML-KEM preserve the security proofs of Kyber.

Any analysis on these or related security and robustness matters is very welcome.

## Hybrid Key Exchange

Hybrid key exchange still maintains the DHKE part. From formal (symbolic) analysis perspective, g<sup>x</sup> and  g<sup>y</sup> are still sent in hybrid key exchange,  g<sup>xy</sup> is still computed and we believe the commutativity property is applicable for that part as-is. From formal (symbolic) analysis perspective, ML-KEM is complementary to that.

Specifically, from {{Section 4 of -hybrid}}, for the symbolic analysis, X25519MLKEM768 in TLS may be viewed as:

~~~
client's key_exchange value = ek || gx
server's key_exchange value = ct || gy
shared secret = ss || gxy
~~~


Formally, the property hybrid key exchange provides is:

~~~
Security properties of TLS hold unless *both* `gxy` and `ss` are
available to the adversary.
~~~

As presented in {{sec-tech-rat}}, hybrid key exchange preserves ECDHE component `gxy`, and concatenates ML-KEM component `ss` as an additional factor.
So as long as *at least* one of these two secrets is not available to the adversary, all security properties should hold.
In particular, even if ML-KEM is completely broken, i.e., `ss` is available to the adversary, the protocol retains the security level of ECDHE.

## Standalone ML-KEM

On the other hand, the formal property standalone ML-KEM provides is:

~~~
Security properties of TLS hold unless `ss` is available to the
adversary.
~~~

## Results
{: #sec-results }

For the FATT process {{TLS-FATT}}, the symbolic analysis {{FATT-CHANCE}} was done in ProVerif by Nadim Kobeissi, who concludes:

~~~
The hybrid neutralizes every weakness standalone ML-KEM carries,
both the confidentiality single point of failure and the
authentication (key-confirmation) failure that rides along
with it, by demanding that both of its components fail at once;
and moving from today’s classical (EC)DHE to a hybrid never gives
up ground, because the classical guarantee survives intact inside
the combination. That is the precise sense in which the hybrid
strictly dominates standalone.

None of this says standalone ML-KEM is broken. Under its stated
assumption, that ML-KEM holds, it is secure under our models.
The argument against it is one of robustness, not of any
present-day attack: it stakes everything on a single, relatively
young assumption, where the hybrid keeps a mature fallback in
reserve. For a deployer who cannot afford to be wrong about
lattice cryptography for the lifetime of the data being protected,
that distinction is the whole game. The two caveats bound the
picture honestly: reuse is a real and quantifiable cost rather
than a catastrophe, and the role-asymmetry worry, while genuinely
the draft’s headline concern, did not surface as a distinct
vulnerability in the server-authenticated, one-initiator-per-session
setting analyzed here.
~~~

Results confirm integration of KEM in TLS is secure as long as the primitive itself is secure.
In our understanding, results also imply a clear preference for hybrids under the Dolev-Yao model, in the sense that if the shared secret from ML-KEM becomes available to the adversary (for example, due to implementation bug), both confidentiality and authentication are broken in standalone ML-KEM, whereas under same condition, both confidentiality and authentication still hold as long as (EC)DHE is still not available to the adversary.
We believe this applies until a powerful CRQC exists which breaks **all** the bits of pre-quantum, where the condition of (EC)DHE being available to the adversary is violated.

The artifacts are available [here](https://github.com/symbolicsoft/reftls) for independent review.

# Implementation-Facing Negative Cases
{: #sec-impl-negative-cases }

The formal analysis above is not an implementation test suite and does not
replace protocol conformance testing.
However, a short set of negative cases can help implementers check that
the intended hybrid binding property is reflected in their APIs, transcript
handling, and key schedule integration.

In particular, implementations of hybrid key exchange ought to reject, or
fail safely on, cases such as the following:

* **malformed or missing shares**: The negotiated group is a hybrid group, but one component of the peer
  key share is missing, malformed, or associated with a different group.
* **mixed transcript context**: The ECDHE and KEM values are individually well-formed, but assembled
  from different handshakes, transcript contexts, or negotiated groups.
* **fallback after hybrid negotiation**: A peer attempts to continue the handshake as standalone ECDHE or
  standalone ML-KEM after a hybrid group was negotiated.
* **premature secret derivation**: Application traffic secrets are derived before both hybrid components
  have been validated and accepted under the negotiated group.
* **API/logging ambiguity**: Exported state, logs, traces, or implementation APIs make a hybrid
  exchange appear as if only one component was used or accepted.

These cases are not intended to create a new formal proof obligation.
They are implementation-facing checks that help bridge the formal
"both components are bound and accepted" property to concrete failure
modes that implementations can accidentally mishandle.

## Argument Matrix for Implementation Review
{: #sec-argument-matrix }

The discussion above can be read as an argument matrix for implementers
and reviewers.  The point is not to resolve every policy preference in
this memo, but to make each recurring argument map to a concrete
review question.

| Argument | Implementation-facing review question | Why it matters |
|----------|---------------------------------------|----------------|
| Hybrid key exchange retains two components. | Does the implementation make it clear that both the ECDHE and ML-KEM components were present, validated, and bound to the same negotiated group and transcript? | Otherwise a successful handshake may not actually reflect the formal "both components are bound and accepted" property. |
| Standalone ML-KEM has a single KEM shared secret. | Are failures in encapsulation, decapsulation, transcript binding, and key-schedule input handled as fail-closed errors rather than as retry or fallback paths? | A standalone construction has no second key-exchange component to preserve confidentiality if the ML-KEM path is mishandled. |
| Hybrid fallback is useful only when it is explicit. | After a hybrid group is negotiated, can either endpoint silently continue as standalone ECDHE or standalone ML-KEM? | Silent continuation changes the negotiated security property and makes interop failures hard to distinguish from downgrades. |
| Cost claims are deployment-dependent. | Are claimed savings measured separately for wire bytes, CPU, memory, latency, code complexity, and operational rollout? | Treating "cost" as a single value can hide whether a deployment is trading away cryptographic robustness for a negligible or unmeasured gain. |
| Formal models do not cover every implementation interface. | Do APIs, logs, exported state, and test harnesses expose enough detail to show which component failed or succeeded? | Reviewers need observable evidence that the implementation behavior matches the protocol-level claim. |

This matrix is deliberately small.  It is intended to help a reviewer
decide whether a concrete implementation or deployment argument belongs
in the formal-methods discussion, in implementation guidance, or in a
separate operational cost analysis.

# Issues That Formal Methods Probably Cannot Solve
{: #sec-gen-issues }

The answers to the following issues are largely dependent on several factors, and the opinions of the community vary largely.

It is necessary to mention that even several respectable cryptographers in the community are not aligned on the issue -- for example see the [long bet](https://github.com/FiloSottile/ecc-vs-lattices-long-bet). Hence, we present the different schools of thought in this section.

~~~
Disclaimer: This is not meant to be an exhaustive list.
If somthing is missing, please simply submit a
*precise* and *concise* PR, preferably with a reference.
~~~

## Which breaks first: ML-KEM-768 or X25519?
In our understanding, the key open question boils down to:

~~~
Which of the two cryptographic mechanisms breaks first?
How does that relate to the CRQC being developed?
How many bits of pre-quantum cryptography can that CRQC actually
break?
Since all of three can be kept secret for some time,
the opinions of the community vary a lot on the different
possible combinations.
~~~

## Does CRQC Break All Bits of Pre-quantum?
One school of thought believes that CRQC will break all bits of pre-quantum cryptographic, while another believes that it will break [only a few bits](https://cr.yp.to/papers/mldsa-20260601.pdf#breakable-keys).

## Policy/Regulations
Some countries have a regulatory requirement for hybrid key exchange.

## Recommendation of Designers
{: #sec-designers-view }

The authors of Kyber/ML-KEM (see [this](https://pq-crystals.org/kyber/index.shtml)) say:

~~~
For users who are interested in using Kyber, we recommend the
following:

* Use Kyber in a so-called hybrid mode in combination with
established "pre-quantum" security; for example in combination
with elliptic-curve Diffie-Hellman.
[...]
~~~

## Thorough Review

Please see a very thorough review [here](https://mailarchive.ietf.org/arch/msg/tls/jlsYHENwqMv-4XPRvunqKsAL36k/), which is self-sufficient.

## 'Significantly Harder' Argument

One school of thought believes in the 'significantly harder' argument, which assumes independence of breakage of ML-KEM and traditionals:

~~~
If the probablity of one being broken over the next n years is p, and
the probability of the other being broken over the next n years is q,
then the probability of both being broken is pq.
~~~

Please see [this](https://github.com/FiloSottile/ecc-vs-lattices-long-bet#2a-what-counts-as-a-break) for what "broken" may mean here modulo some [exclusions](https://github.com/FiloSottile/ecc-vs-lattices-long-bet#5-exclusions).

Given the very different type of cryptographic constructions involved, independence might be a reasonable assumption.
However, another school of thought disagrees with 'significantly harder' argument with a reasonable counter-argument that in reality, cryptography is much more complicated than that and depending on the algorithms and the composition method, the probability can clearly be q, or smaller than pq.

In our understanding, most other counter-arguments seem to break the [exclusions](https://github.com/FiloSottile/ecc-vs-lattices-long-bet#5-exclusions).

Please note that this argument is based on the security of *primitives*, rather than the *composition* of primitives in protocols. Hence, formal methods probably have nothing to help here.

## Urgency

It is unclear *whether* and if applicable *when* Cryptographically-Relevant Quantum Computer (CRQC) will eventually become practical.
The opinions of the community vary from never because of complicated physics (see [this](https://eprint.iacr.org/2025/1237)) to be *prepared* for it as early as 2029 (see [Google 2029](https://blog.google/innovation-and-ai/technology/safety-security/cryptography-migration-timeline/) and [Cloudflare 2029](https://blog.cloudflare.com/post-quantum-roadmap/)).
Technically, please note that Google has not even released the **quantum circuit** underlying their recent claims -- apparently the reason for this urgency. So Google's claims may not yet be justified.

Moreover, in our understanding, these deadlines are for PQ-based protection in general regardless of hybrid key exchange or standalone KEMs in TLS. Since hybrid key exchange is wildly in use, these deadlines are mainly for quantum-safe authentication.

In any case, one school of thought sees no reason to create panic for publication of {{I-D.ietf-tls-mlkem}} based on this because many implementations -- such as OpenSSL -- have already implemented standalone ML-KEM, and it is just a matter of enabling it. And frankly, nobody needs permission from the IETF to enable it.

## "Cost"
"Cost" has been presented on the list as the motivation for standalone ML-KEM in TLS but we have not seen any supporting analysis.
Our observation from {{Section 4 of -hybrid}} is that -- for example -- for X25519MLKEM768, the traditional part seems negligible compared to ML-KEM part in `key_exchange`:

| Bytes in field| PQ part (ML-KEM)      | Traditional part (X25519) |
|---------------|--------------------|-------------|
| Client share  | 1184               | 32          |
| Server share  | 1088               | 32          |

We believe other "costs" will depend on several factors -- including but not limited to implementation details and deployment scenario -- and it is quite **subjective**.

There seems to be a need for a thorough study to understand the "cost."
We invite the research and standardization community to perform cost analysis and share the results.

## Is Publication Necessary?

Code Points for ML-KEM have already been assigned.
{{I-D.barnes-tls-this-could-have-been-an-email}} provides detailed rationale as to why publication of such documents and the debates around that may be unnecessary. In our understanding, {{I-D.pwouters-crypto-current-practices}} makes similar arguments.

## Shiny New Crypto

ML-KEM is quite new in the IETF and even in the IRTF.
CFRG is starting some efforts for detailed analysis. The extended deadline for submission is 22.06. Please see the latest [CFRG chairs email](https://mailarchive.ietf.org/arch/msg/cfrg/6K43Ycr062Ym1G0q4WHxZQ2HW8M/) for further details.

## Formal Mapping of FIPS to IETF BCP14

As discussed on the TLS list, we are not aware of any formal mapping of the FIPS recommendations to the IETF BCP14 terminology, such as SHOULD vs. MUST. In general, we believe re-using FIPS recommendations is ambiguous for IETF readers.

## Outstanding NIST Comments
One school of thought believes that NIST has rushed through the process and not addressed all the comments that were submitted during the open review. Please see comments [here](https://csrc.nist.gov/files/pubs/fips/203/ipd/docs/fips-203-initial-public-comments-2023.pdf).

## Too Early
One school of thought simply believes that publication of {{I-D.ietf-tls-mlkem}} and related discussions are just too early and unnecessary.

## Patents

Some concerns related to patents have been raised. See some relevant patents [here](https://datatracker.ietf.org/ipr/search/?submit=draft&id=draft-ietf-tls-mlkem).

## Implementation Bugs
One school of thought is worried about the implementations bugs. Some use it as advocacy for the use of hybrids that if one could exploit one of the two primitives, the other one can save.

## Depth of Hybrids?
One school of thought has questioned the ML-KEM + ECC hybrids rather than, say, Module Lattices + McEliece + hash-based three-way composites.


# Security Considerations
{: #sec-sec-cons }

The whole document is about improving security considerations.

Like all security proofs, formal analysis is only as strong as its assumptions and model.
The scope is typically limited, and the model does not necessarily capture real-world deployment complexity, implementation details, operational constraints, or misuse scenarios.
Technically, formal proof only guarantees anything if all the assumptions hold, which is unlikely in practice.
Formal methods should be used as complementary and not as substitute of other analysis methods.



# IANA Considerations

This memo has no IANA actions.


--- back

[comment]: <> (# Acknowledgments)

# Contributors
{:numbered="false"}

Nadim Kobeissi performed a thorough formal analysis {{sec-results}} at high priority based on our call for analysis in previous versions of the memo to get a confirmation.

Text in {{sec-impl-negative-cases}} was proposed by Songbo Bu.

Text in {{sec-sec-cons}} is based on the proposal by John Preuß Mattsson.

{{sec-gen-issues}} is largely based on the opinions of many IETF participants.

We gratefully thank Yaakov Stein and Ilari Liusvaara for their substantial technical guidance, valuable feedback, and contributions.

# Acknowledgments
{:numbered="false"}

We thank Eric Rescorla, Brian E. Carpenter, and Tibor Jager for their valuable feedback.

We acknowledge several IETF participants who have contributed to this memo with their insights.

The research work is funded by German Research Foundation ("Deutsche Forschungsgemeinschaft.")

# History
{:numbered="false"}

-00

* On popular demand, moved from {{I-D.usama-tls-fatt-extension}} to an independent I-D
* Major change: added {{sec-proof-break}}
* Some minor clarifications

-01

* Added justification based on FATT process
* Reorganization, specially in motivation
* Added some common arguments: {{sec-gen-issues}}
* Comparison with hybrid key exchange

-02

* Added gap analysis
* What to model and analyze? {{sec-model-analyze}}
* Added FATT review is harmless
* Extended comparison with hybrid key exchange
* Opinion of designers {{sec-designers-view}}

-03

* Completely restructured and reframed after confirmation by formal analysis
* Added implementation-facing negative cases and argument matrix
* Some new arguments: implementation bugs, depth of hybrids, policy, all bits, which primitive breaks first?

-04

* Remove links to opinion of IETF participants
* Inform the reader of the facets of problem
