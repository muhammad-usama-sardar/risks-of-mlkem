---
title: "Analysis of Hybrid Key Exchange and Non-hybrid KEMs in TLS 1.3"
category: info

docname: draft-usama-tls-risks-of-mlkem-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Transport Layer Security"
venue:
  group: "Transport Layer Security"
  type: "Working Group"
  mail: "tls@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/tls/"
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
  ID-Crisis:
    title: "Identity Crisis in Confidential Computing: Formal Analysis of Attested TLS"
    date: November 2025,
    target: https://www.researchgate.net/publication/398839141_Identity_Crisis_in_Confidential_Computing_Formal_Analysis_of_Attested_TLS
    author:
      - ins: M. U. Sardar
      - ins: M. Moustafa
      - ins: T. Aura
  ID-Crisis-Repo:
     title: "Identity Crisis in Confidential Computing: Formal Analysis of Attested TLS Protocols"
     date: 2025,
     target: https://github.com/CCC-Attestation/formal-spec-id-crisis
     author:
      - ins: Muhammad Usama Sardar
  reftls: DOI.10.1109/SP.2017.26
  reftls-Repo:
     title: "Verified Models and Reference Implementations for the TLS 1.3 Standard Candidate"
     date: 2026,
     target: https://github.com/Inria-Prosecco/reftls
     author:
      - ins: K. Bhargavan
      - ins: B. Blanchet
      - ins: N. Kobeissi
  I-D.pwouters-crypto-current-practices:
  I-D.barnes-tls-this-could-have-been-an-email:
  rfc3552:
  I-D.ietf-tls-ecdhe-mlkem: hybrid
  I-D.ietf-tls-hybrid-design-09:
  I-D.ietf-tls-hybrid-design:
...

--- abstract

The draft presents an analysis of hybrid key exchange and standalone ML-KEM.
Analysis covers both symbolic and computational.
Specifically, it covers the [formal analysis](https://github.com/symbolicsoft/reftls/blob/master/paper/tls-hybrid.pdf) by Nadim Kobeissi on the potential issue of asymmetry.
The analysis confirms that asymmetry is not a problem.
Moreover, the analysis concludes that hybrid key exchange is preferable over standalone ML-KEM.
This draft aims to propose this statement to be added in the security considerations of standalone ML-KEM.

This draft offers some preliminary discussion to help the developers and policy makers make informed choices.
Finally, the draft also aims to reduce the endless repitition of arguments from both sides presented on several lists by documenting these arguments so they can simply be referred to.

We acknowledge several IETF participants who have contributed to this draft with their insights.
This draft captures what *we* understand them to be saying.

--- middle

# Introduction

Readers are assumed to be familiar with {{NistFips203}}, {{I-D.ietf-tls-rfc8446bis}}, and {{I-D.ietf-tls-mlkem}}. Please note that the draft has currently several hyperlinks.

We assert that the security considerations of {{I-D.ietf-tls-mlkem}} are insufficient.
We believe that consistent with {{TLS-FATT}} process, *symbolic* and *computational* analysis (to be interpreted as in [SoK](https://eprint.iacr.org/2019/1393.pdf)) of **integration** of standalone ML-KEM in the context of TLS is helpful here.





## Motivation
{: #sec-mot }

{{rfc3552}} requires to document the risks in the security considerations.
To support those requirements for {{I-D.ietf-tls-mlkem}}, this draft aims to formally study the security of standalone ML-KEM in TLS 1.3.
This is because of the following reasons.

In the last WGLC, {{I-D.ietf-tls-mlkem}} had an opposition of several (ca. 25 in our understanding) WG participants -- even more than the supporters (ca. 21 in our understanding). We see 2 possible options:

* Continue tabletop discussions on **subjective** estimation of urgency, risks, costs, tradeoffs, etc., and keep burning WG energy by endless repitition.
* Do some technical analysis using (*symbolic* and *computational*) formal methods to get a confirmation on the security of **integration** of standalone ML-KEM in the context of TLS and offer a statement for security considerations.

We believe the former cannot resolve the dispute. We sincerely **hope** the latter will help.

~~~
We believe the security considerations of {{I-D.ietf-tls-mlkem}} are
insufficient.
~~~

## Intuition
Leaking out the ECDHE key from hybrid key exchange should downgrade the security to the level of a standalone ML-KEM.
Therefore, hybrid key exchange is *in general* more secure, unless:

* ECDHE is fully broken, in which case it still falls equivalent to standalone ML-KEM,
* in the *hypothetical* scenario that there is an implementation bug in the ECDHE part which is triggered only in composition. We have not yet seen any concrete evidence of such a scenario on the list.

We believe that *in general*:

~~~
1. Migration from ECDHE to hybrid key exchange is security improvement.
2. Migration from hybrid key exchange to standalone ML-KEM is security
regression.
~~~


### Expected Learning
We believe formal methods can provide additional value for security considerations of this draft in order to maintain the high cryptographic assurance of TLS.

~~~
Since we have no guarantee on whether ECDHE will break before ML-KEM,
it seems appropriate to do thorough cryptographic analysis.
We believe the Harvest Now, Decrypt Later (HNDL) attack applies
equally well to standalone ML-KEM.
~~~

Adversary can record all traffic and decrypt it when ML-KEM is broken.
The opinions of WG participants here vary from "ML-KEM is secure" to "ML-KEM is probably already secrectly broken."
Formal methods can operate under the assumption that ML-KEM is secure, and focus on the **integration** of ML-KEM in TLS under this assumption.

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

Based on the discussion on list, simply replacing ideal DHKE by ideal ML-KEM in the formal model is not very useful. We ought to focus on the more security-critical questions about **integration** of ML-KEM in TLS.
We present a few high-level observations to consider for security considerations of {{I-D.ietf-tls-mlkem}}:

* The model ought to consider that any agent could have initiated the TLS, rather than assigning the agents with static roles of client and server in the model. When agents are assigned non-static roles, it would be interesting to see whether the asymmetry issue becomes visible in some property. We consider it very critical for security considerations of {{I-D.ietf-tls-mlkem}} and this is the key point of this draft.
* Different failure modes proposed on list can be modeled.
* A large part of the problem is the careful investigation of what to model, under what threat model, under what system model, under what implementation scenarios etc. We believe some of this is important for security considerations of {{I-D.ietf-tls-mlkem}}.
* It will be interesting to see some analysis about any subtle cases where hybrid key exchange in TLS is *not* at least as good as standalone ML-KEM in TLS. Our understanding is that some participants would like to see some statement on the comparison since hybrid key exchange is the de facto industry standard.
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

[Formal analysis](https://github.com/symbolicsoft/reftls/blob/master/paper/tls-hybrid.pdf) by Nadim Kobeissi concludes:

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
* Application traffic secrets are derived before both hybrid components
  have been validated and accepted under the negotiated group.
* Exported state, logs, traces, or implementation APIs make a hybrid
  exchange appear as if only one component was used or accepted.

These cases are not intended to create a new formal proof obligation.
They are implementation-facing checks that help bridge the formal
"both components are bound and accepted" property to concrete failure
modes that implementations can accidentally mishandle.

# Issues That Formal Methods Probably Cannot Solve
{: #sec-gen-issues }

The answers to the following issues are largely dependent on several factors, and the opinions vary largely.

It is necessary to mention that even several respectable cryptographers in the community are not aligned on the issue -- for example see the [long bet](https://github.com/FiloSottile/ecc-vs-lattices-long-bet). Hence, our personal opinion is probably not that important. Probably the best we can do is to capture *our* understanding of the views of WG participants.

~~~
Disclaimer: This is not meant to be an exhaustive list.
This is also not meant to pritoritize any concerns over others.
This is a sincere attempt to slowly capture the opinions
to avoid endless repetitions from both sides.
Many substantive concerns are missing.
We are slowly collecting the concerns, as time allows.
If your substantive concern is missing, it is unintentional.
Please simply submit a *precise* and *concise* PR.
~~~

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

A WG participant [shares](https://mailarchive.ietf.org/arch/msg/tls/NnGrdavTY6KGTVQo46xaPbSHQzw/) that:

~~~
I recently asked one of the members of the CRYSTALS team
whether this is still his view, and the response was:
"Yes, of course."
~~~

## Thorough Review

Please see a very thorough review [here](https://mailarchive.ietf.org/arch/msg/tls/jlsYHENwqMv-4XPRvunqKsAL36k/), which is self-sufficient.

## 'Significantly Harder' Argument

Some participants believe in the 'significantly harder' argument, which assumes independence of breakage of ML-KEM and traditionals:

~~~
If the probablity of one being broken over the next n years is p, and
the probability of the other being broken over the next n years is q,
then the probability of both being broken is pq.
~~~

Please see [this](https://github.com/FiloSottile/ecc-vs-lattices-long-bet#2a-what-counts-as-a-break) for what "broken" may mean here modulo some [exclusions](https://github.com/FiloSottile/ecc-vs-lattices-long-bet#5-exclusions).

Given the very different type of cryptographic constructions involved, independence might be a reasonable assumption.
However, some participants disagree with 'significantly harder' argument with a reasonable counter-argument that in reality, cryptography is much more complicated than that (cf. [this](https://mailarchive.ietf.org/arch/msg/tls/AK7QUiiGX3ynsOhXeUuwn_IY7ik/)):

~~~
Depending on the algorithms and the composition method,
the probability can clearly be q, or smaller than pq.
~~~

In our understanding, most other counter-arguments seem to break the [exclusions](https://github.com/FiloSottile/ecc-vs-lattices-long-bet#5-exclusions).

Please note that this argument is based on the security of *primitives*, rather than the *composition* of primitives in protocols. Hence, formal methods probably have nothing to help here.

## Urgency

It is unclear *whether* and if applicable *when* Cryptographically-Relevant Quantum Computer (CRQC) will eventually become practical.
The opinions vary from never because of complicated physics (see [this](https://eprint.iacr.org/2025/1237)) to be *prepared* for it as early as 2029 (see [Google 2029](https://blog.google/innovation-and-ai/technology/safety-security/cryptography-migration-timeline/) and [Cloudflare 2029](https://blog.cloudflare.com/post-quantum-roadmap/)).
Technically, please note that Google has not even released the **quantum circuit** underlying their recent claims -- apparently the reason for this urgency. So Google's claims may not yet be justified.

Moreover, in our understanding, these deadlines are for PQ-based protection in general regardless of hybrid key exchange or standalone KEMs in TLS. Since hybrid key exchange is wildly in use, these deadlines are mainly for quantum-safe authentication.

In any case, some participants see no reason to create panic for publication of {{I-D.ietf-tls-mlkem}} based on this because many implementations -- such as OpenSSL -- have already implemented standalone ML-KEM, and it is just a matter of enabling it. And frankly, nobody needs permission from the IETF to enable it.

## "Cost"
"Cost" has been presented on the list as the motivation for standalone ML-KEM in TLS but we have not seen any supporting analysis.
Our observation from {{Section 4 of -hybrid}} is that -- for example -- for X25519MLKEM768, the traditional part seems negligible compared to ML-KEM part in `key_exchange`:

| Bytes in field| PQ part (ML-KEM)      | Traditional part (X25519) |
|---------------|--------------------|-------------|
| Client share  | 1184               | 32          |
| Server share  | 1088               | 32          |

We believe other "costs" will depend on several factors -- including but not limited to implementation details and deployment scenario -- and it is quite **subjective**.

There seems to be a need for a thorough study to understand the "cost."
We invite the WG participants to perform cost analysis and share the results with the WG.

## Is Publication Necessary?

Code Points for ML-KEM have already been assigned.
{{I-D.barnes-tls-this-could-have-been-an-email}} provides detailed rationale as to why publication of such documents and the debates around that may be unnecessary. In our understanding, {{I-D.pwouters-crypto-current-practices}} makes similar arguments.

## Shiny New Crypto

ML-KEM is quite new in the IETF and even in the IRTF. Some WG participants have shown concern over premature publication of {{I-D.ietf-tls-mlkem}} until a detailed analysis has been done by CFRG.

CFRG is starting some efforts for analysis. The extended deadline for submission is 22.06. Please see the latest [CFRG chairs email](https://mailarchive.ietf.org/arch/msg/cfrg/6K43Ycr062Ym1G0q4WHxZQ2HW8M/) for further details.

## Formal Mapping of FIPS to IETF BCP14

As discussed on the TLS list, we are not aware of any formal mapping of the FIPS recommendations to the IETF BCP14 terminology, such as SHOULD vs. MUST. In general, we believe re-using FIPS recommendations is ambiguous for IETF readers.

## Outstanding NIST Comments
Some participants believe that NIST has rushed through the process and not addressed all the comments that were submitted during the open review. Please see comments [here](https://csrc.nist.gov/files/pubs/fips/203/ipd/docs/fips-203-initial-public-comments-2023.pdf).

## Too Early
Some participants simply believe that publication of {{I-D.ietf-tls-mlkem}} and related discussions are just too early and unnecessary.

## Patents

Some WG participants have raised some concerns related to patents. See some relevant patents [here](https://datatracker.ietf.org/ipr/search/?submit=draft&id=draft-ietf-tls-mlkem).


# Security Considerations
{: #sec-sec-cons }

The whole document is about improving security considerations.

Like all security proofs, formal analysis is only as strong as its assumptions and model.
The scope is typically limited, and the model does not necessarily capture real-world deployment complexity, implementation details, operational constraints, or misuse scenarios.
Formal methods should be used as complementary and not as subtitute of other analysis methods.



# IANA Considerations

This document has no IANA actions.


--- back

[comment]: <> (# Acknowledgments)

# Acknowledgments
{:numbered="false"}

We would like to thank Yaakov Stein, Ilari Liusvaara, John Preuß Mattsson, Eric Rescorla, Brian E Carpenter, Nadim Kobeissi, and Tibor Jager for their valuable feedback and contributions.

{{sec-gen-issues}} is largely based on the opinions of many IETF participants.

Text in {{sec-sec-cons}} is based on the proposal by John Preuß Mattsson.

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
