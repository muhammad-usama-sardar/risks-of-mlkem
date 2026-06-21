---
title: "Analysis of Hybrid Key Establishment and Standalone ML-KEM in TLS 1.3"
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

This memo maps out the relevant technical facets surrounding quantum-resistant key establishment in TLS 1.3 and provides some preliminary discussion to help developers and policymakers make informed choices.
In particular, it presents hybrid key establishment and standalone ML-KEM in TLS 1.3.
Moreover, it offers minimal implementation guidance for hybrid key establishment.
The memo finally presents technical insights into hybrid key establishment and standalone ML-KEM in TLS 1.3.
Our observation is that hybrid key establishment is preferable over standalone ML-KEM until a powerful CRQC exists which breaks most bits of pre-quantum.
This memo is not a standard nor has it been shown to have consensus of the IETF community.

--- middle

# Introduction

Readers are assumed to be familiar with {{NistFips203}}, {{I-D.ietf-tls-rfc8446bis}}, and {{I-D.ietf-tls-mlkem}}. Please note that the memo has currently several hyperlinks.

## Objectives

The memo serves three objectives:

* Identifying the underlying technical facets and common complexities of the deployment debate to provide clarity
* Minimal implementation guidance for hybrids
* Summary of formal methods works for hybrid key establishment and standalone ML-KEM in an accessible and intuitive way

### Presenting Different Technical Facets

The memo identifies technical facets that affect deployment reasoning.
Facets are categorized as **primitive-level** and **protocol-level**.
Examples include break timing, residual pre-quantum security, deployment cost, patents, and implementation behavior.

### Minimal Implementation Guidance for Hybrids
The minimal implementation consideration for hybrid key establishment is not only whether ML-KEM is secure as a
primitive, but also whether a TLS deployment can show that both hybrid
components were validated, transcript-bound, and fail-closed under the
negotiated group.

### Technical Insights

The memo covers the formal methods for hybrid key establishment and standalone ML-KEM in TLS.
Formal methods mostly operate at the protocol-level.
Formal methods can provide additional value in order to maintain the high cryptographic assurance of TLS.
This includes *symbolic* and *computational* analysis (to be interpreted as in [SoK](https://eprint.iacr.org/2019/1393.pdf)) of integration of standalone ML-KEM in the context of TLS.
Specifically, it covers the formal analysis {{FATT-CHANCE}} in ProVerif on the potential issue of asymmetry.
The analysis confirms that asymmetry is not a problem.

Formal analysis can address some protocol-composition questions, but it does
not settle every deployment or policy question relevant to standalone ML-KEM
and hybrid key establishment.

## Motivation
{: #sec-mot }

~~~
Since we have no guarantee on whether ECDHE will break before ML-KEM,
it seems appropriate to do thorough cryptographic analysis.
We believe the Harvest Now, Decrypt Later (HNDL) attack applies
equally well to standalone ML-KEM.
~~~

An adversary can record all traffic and decrypt it later if ML-KEM is broken.
The opinions of the community on this matter vary from "ML-KEM is secure" to "ML-KEM is probably already secretly broken."
Formal methods can operate under the assumption that ML-KEM is secure, and focus on the integration of ML-KEM in TLS under this assumption.

* As an example, formal methods can help justify design choices, such as the preference for hybrid key establishments.
It can also help identify all the assumptions under which the properties hold.
* *Computational* analysis -- using tools such as CryptoVerif -- seems like a reasonable approach to ensure security of ML-KEM in TLS, such as binding shared secret `ss` to the TLS transcript hash.

~~~
We believe that the focus of symbolic analysis ought to be on the
*integration* details (transcript binding, key schedule, agreement)
for standalone ML-KEM in the context of TLS, rather than the
*primitive* itself.
~~~

## Intuition
Leaking out the ECDHE key from hybrid key establishment should downgrade the security to the level of a standalone ML-KEM.
Therefore, hybrid key establishment is *in general* more secure, unless:

* ECDHE is fully broken, in which case it still falls equivalent to standalone ML-KEM,
* in the *hypothetical* scenario that there is an implementation bug in the ECDHE part which is triggered only in composition. We are not aware of any concrete evidence of such a scenario.

We believe that *in general*:

~~~
1. Migration from ECDHE to hybrid key establishment is security improvement.
2. Migration from hybrid key establishment to standalone ML-KEM is security
regression, unless CRQC exists to break most ECC keys.
~~~

# Conventions and Definitions

{::boilerplate bcp14-tagged}

* Symbolic analysis: see [SoK](https://eprint.iacr.org/2019/1393.pdf)
* Computational analysis: see [SoK](https://eprint.iacr.org/2019/1393.pdf)
* Standalone ML-KEM refers to {{I-D.ietf-tls-mlkem}}.
* Hybrid key establishment refers to {{-hybrid}} and {{I-D.ietf-tls-hybrid-design}}.

We believe that symbolic and computational models are complementary and not a substitute of each other.

## Key Establishment and Key Encapsulation
{: #sec-proof-break }

In traditional key exchange (DHKE), both endpoints send their public key shares g<sup>x</sup> and g<sup>y</sup> to derive a shared secret g<sup>xy</sup>.

In key encapsulation, there is essentially only one endpoint (say client) which generates the key pair `(dk,ek)` where `dk` represents the *secret decapsulation key* and `ek` represents the *public encapsulation key*.
In a KEM, only one of the endpoints (client in above example) sends the public encapsulation key `ek` and the peer (server) sends a ciphertext `ct`.

# Technical Facets
{: #sec-gen-issues }

This section presents a few such technical facets.

This list is not exhaustive and does not try to prioritize one facet over
another.  If an important facet is missing, the most useful contribution is a
precise and concise PR with proposed text and references.

## Primitive-level

### Which breaks first: ML-KEM-768 or X25519?

A core uncertainty for any hybrid-vs-standalone comparison is:

~~~
Which of the two cryptographic mechanisms breaks first?
How does that relate to the CRQC being developed?
How many bits of pre-quantum cryptography can that CRQC actually
break?
~~~

All three variables may remain uncertain for a long period of time.  This makes
it difficult to reduce the question to one simple ordering of "secure" and
"insecure" choices.

### Does CRQC Break All Bits of Pre-quantum?

The impact of a CRQC on pre-quantum cryptography may not be uniform across all
algorithms, security levels, and attack models.  One concrete position is that
a CRQC may break [only a few bits](https://cr.yp.to/papers/mldsa-20260601.pdf#breakable-keys)
rather than all effective pre-quantum security.  This matters because a hybrid
construction depends on the residual value of the traditional component after a
quantum advance.

### Policy/Regulations
Some countries have a regulatory requirement for hybrid key establishment.  This is
a deployment and compliance constraint.

### Recommendation of Designers
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

### 'Significantly Harder' Argument

One common hybrid-security argument assumes independence between a break of
ML-KEM and a break of the traditional key-exchange component:

~~~
If the probability of one being broken over the next n years is p, and
the probability of the other being broken over the next n years is q,
then the probability of both being broken is pq.
~~~

Please see [this](https://github.com/FiloSottile/ecc-vs-lattices-long-bet#2a-what-counts-as-a-break) for what "broken" may mean here modulo some [exclusions](https://github.com/FiloSottile/ecc-vs-lattices-long-bet#5-exclusions).

Given the very different type of cryptographic constructions involved, independence might be a reasonable assumption.
However, a reasonable counter-argument is that, in reality, cryptography is much more complicated than that and depending on the algorithms and the composition method, the probability can clearly be q, or smaller than pq.

In our understanding, most other counter-arguments seem to break the [exclusions](https://github.com/FiloSottile/ecc-vs-lattices-long-bet#5-exclusions).

Please note that this argument is based on the security of *primitives*, rather than the *composition* of primitives in protocols. Hence, formal methods probably have nothing to help here.

### Shiny New Crypto

ML-KEM is quite new in the IETF and even in the IRTF.
CFRG is starting some efforts for detailed analysis. The extended deadline for submission is 22 June 2026. Please see the latest [CFRG chairs email](https://mailarchive.ietf.org/arch/msg/cfrg/6K43Ycr062Ym1G0q4WHxZQ2HW8M/) for further details.
The confidence question is not only about calendar age.
For deployers, the relevant technical question is how much assurance has
accumulated for the structured-lattice setting, the parameter choices, the
reduction assumptions, and the implementation behavior of the construction.
That is a risk-management input rather than a proof that standalone ML-KEM is
unsafe.

### Outstanding NIST Comments
One concrete position is that not all comments submitted during the open review were fully addressed.  Please see comments [here](https://csrc.nist.gov/files/pubs/fips/203/ipd/docs/fips-203-initial-public-comments-2023.pdf).

### Patents

Some concerns related to patents have been raised. See some relevant patents [here](https://datatracker.ietf.org/ipr/search/?submit=draft&id=draft-ietf-tls-mlkem).

## Protocol-level

### Urgency

It is unclear *whether* and if applicable *when* Cryptographically-Relevant Quantum Computer (CRQC) will eventually become practical.
Public assessments range from skepticism based on the difficulty of the physics
(see [this](https://eprint.iacr.org/2025/1237)) to migration targets that aim to
be *prepared* as early as 2029 (see [Google 2029](https://blog.google/innovation-and-ai/technology/safety-security/cryptography-migration-timeline/)
and [Cloudflare 2029](https://blog.cloudflare.com/post-quantum-roadmap/)).
The technical details behind some public timeline claims are not yet fully public. In particular, Google has not even released the **quantum circuit** underlying their recent claims -- apparently the reason for this urgency.
These claims are therefore best treated as deployment-planning inputs rather than as proof of a specific CRQC arrival date.

Moreover, in our understanding, these deadlines are for PQ-based protection in general regardless of hybrid key establishment or standalone KEMs in TLS. Since hybrid key establishment is widely in use, these deadlines are mainly for quantum-safe authentication.

A separate deployment point is that publication timing does not by itself control
deployment timing.  Implementations such as OpenSSL already include standalone
ML-KEM support, and deployments can enable available code according to their own
policy and risk assessment.

### Cost
Cost may be the motivation for standalone ML-KEM in TLS but we are not aware of any supporting analysis.
Our observation from {{Section 4 of -hybrid}} is that -- for example -- for X25519MLKEM768, the traditional part seems negligible compared to ML-KEM part in `key_exchange`:

| Bytes in field| PQ part (ML-KEM)      | Traditional part (X25519) |
|---------------|--------------------|-------------|
| Client share  | 1184               | 32          |
| Server share  | 1088               | 32          |

This observation does not by itself settle wire-size or middlebox questions.
Those questions need to be measured at the full TLS handshake level, including
ClientHello size, record boundaries, fragmentation behavior, retry paths, and
the deployment's existing extension set.

Other "costs" depend on several factors -- including implementation details,
deployment scenario, latency budget, memory pressure, code complexity, and
operational rollout cost -- and should not be treated as one scalar value.
For broad Internet-facing deployments, ECDHE and hybrid support are likely to
remain necessary for compatibility and policy reasons, so standalone ML-KEM
does not automatically remove the implementation, testing, or operational cost
of the traditional component.
The conclusion may be different for closed, constrained, or endpoint-controlled
deployments, but that is a deployment-class-specific claim and needs evidence
from those environments.

There seems to be a need for a thorough study to understand the "cost."
A useful analysis would separate wire bytes, CPU, memory, latency, implementation
complexity, and operational rollout cost.

### Is Publication Necessary?

Code Points for ML-KEM have already been assigned.
{{I-D.barnes-tls-this-could-have-been-an-email}} provides detailed rationale as to why publication of such documents and the debates around that may be unnecessary. In our understanding, {{I-D.pwouters-crypto-current-practices}} makes similar arguments.

### Formal Mapping of FIPS to IETF BCP14

As discussed on the TLS mailing-list, we are not aware of any formal mapping of the FIPS recommendations to the IETF BCP14 terminology, such as SHOULD vs. MUST. In general, we believe re-using FIPS recommendations is ambiguous for IETF readers.

### Implementation Bugs

Implementation bugs are a separate risk from primitive-level cryptanalysis.
Hybrid key establishment may mitigate some single-component failures, but only if the
implementation actually validates both components, binds them to the same
transcript, derives traffic secrets only after both components are accepted, and
fails closed when either component fails.

### Depth of Hybrids?

The depth of a hybrid is itself a design question.  ML-KEM plus ECC is only one
composition point; other designs could combine module lattices, code-based KEMs,
or hash-based components.  That broader design space is outside the scope of the
formal-methods results discussed above.
If the motivation for standalone ML-KEM is size, constrained deployment, or
operational simplicity, another technical question is whether a different
hybrid design point could address that motivation while retaining a second
cryptographic component.


# Implementation-Facing Negative Cases
{: #sec-impl-negative-cases }

The formal analysis above is not an implementation test suite and does not
replace protocol conformance testing.
However, a short set of negative cases can help implementers check that
the intended hybrid binding property is reflected in their APIs, transcript
handling, and key schedule integration.

In particular, implementations of hybrid key establishment ought to reject, or
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
| Hybrid key establishment retains two components. | Does the implementation make it clear that both the ECDHE and ML-KEM components were present, validated, and bound to the same negotiated group and transcript? | Otherwise a successful handshake may not actually reflect the formal "both components are bound and accepted" property. |
| Standalone ML-KEM has a single KEM shared secret. | Are failures in encapsulation, decapsulation, transcript binding, and key-schedule input handled as fail-closed errors rather than as retry or fallback paths? | A standalone construction has no second key-exchange component to preserve confidentiality if the ML-KEM path is mishandled. |
| Hybrid fallback is useful only when it is explicit. | After a hybrid group is negotiated, can either endpoint silently continue as standalone ECDHE or standalone ML-KEM? | Silent continuation changes the negotiated security property and makes interop failures hard to distinguish from downgrades. |
| Cost claims are deployment-dependent. | Are claimed savings measured separately for wire bytes, CPU, memory, latency, code complexity, and operational rollout? | Treating "cost" as a single value can hide whether a deployment is trading away cryptographic robustness for a negligible or unmeasured gain. |
| Formal models do not cover every implementation interface. | Do APIs, logs, exported state, and test harnesses expose enough detail to show which component failed or succeeded? | Reviewers need observable evidence that the implementation behavior matches the protocol-level claim. |

This matrix is deliberately small.  It is intended to help a reviewer
decide whether a concrete implementation or deployment argument belongs
in the formal-methods discussion, in implementation guidance, or in a
separate operational cost analysis.


# Technical Analysis

## Symbolic Analysis
For brevity, we omit other assumptions in the properties below and focus on the difference.
This assumes the hybrid construction to be secure.

For implementers, the symbolic view can be read as a component-failure exercise.
Instead of asking how hard ML-KEM or ECDHE is to break, the analysis may ask whether security properties still hold under Dolev-Yao attacker if one component secret is already available to the adversary.

### Minimum Viable Modeling
{: #sec-model-analyze }

Based on the discussion on TLS mailing-list, simply replacing ideal DHKE by ideal ML-KEM in the formal model is not very useful. We ought to focus on the more security-critical questions about integration of ML-KEM in TLS.
We present a few high-level observations to consider for security considerations of {{I-D.ietf-tls-mlkem}}:

* The model ought to consider that any agent could have initiated the TLS, rather than assigning the agents with static roles of client and server in the model. When agents are assigned non-static roles, it would be interesting to see whether the asymmetry issue becomes visible in some property. We consider it very critical for security considerations of {{I-D.ietf-tls-mlkem}} and this is the key point of this memo.
* Different failure modes can be modeled.
* A large part of the problem is the careful investigation of what to model, under what threat model, under what system model, under what implementation scenarios etc.
* It will be interesting to see some analysis about any subtle cases where hybrid key establishment in TLS is *not* at least as good as standalone ML-KEM in TLS, since hybrid key establishment is the de facto industry standard.
* We believe brainstorming about some robustness (vs. security) properties would also be useful. Even if the security properties hold, does standalone ML-KEM make side-channel leakage easier? This might be a valuable consideration for the implementers.
* Analysis may be helpful to ensure that the changes -- such as the removal of hash function (cf. Appendix C.1, bullet 3 in {{NistFips203}}) -- from Kyber to ML-KEM preserve the security proofs of Kyber.

Any analysis on these or related security and robustness matters is very welcome.

### Hybrid Key Establishment

Hybrid key establishment still maintains the DHKE part. From formal (symbolic) analysis perspective, g<sup>x</sup> and  g<sup>y</sup> are still sent in hybrid key establishment,  g<sup>xy</sup> is still computed and we believe the commutativity property is applicable for that part as-is. From formal (symbolic) analysis perspective, ML-KEM is complementary to that.

Specifically, from {{Section 4 of -hybrid}}, for the symbolic analysis, X25519MLKEM768 in TLS may be viewed as:

~~~
client's key_exchange value = ek || gx
server's key_exchange value = ct || gy
shared secret = ss || gxy
~~~


Formally, the property hybrid key establishment provides is:

~~~
Security properties of TLS hold unless *both* `gxy` and `ss` are
available to the adversary.
~~~

As presented in {{sec-tech-rat}}, hybrid key establishment preserves ECDHE component `gxy`, and concatenates ML-KEM component `ss` as an additional factor.
So as long as *at least* one of these two secrets is not available to the adversary, all security properties should hold.
In particular, even if ML-KEM is completely broken, i.e., `ss` is available to the adversary, the protocol retains the security level of ECDHE.

### Standalone ML-KEM

At the symbolic level, some analysis -- such as [this](https://eprint.iacr.org/2022/1111.pdf) for KEMTLS in Tamarin -- exists. In our understanding, both client and server encapsulate, which may bring the symmetry.
The formal property standalone ML-KEM provides is:

~~~
Security properties of TLS hold unless `ss` is available to the
adversary.
~~~

### Results
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

Under the stated model assumptions, the results confirm that integrating a KEM into TLS is secure as long as the primitive itself is secure.
In our understanding, the results also imply a clear preference for hybrids under the Dolev-Yao model: if the shared secret from ML-KEM becomes available to the adversary (for example, due to an implementation bug), standalone ML-KEM loses both confidentiality and the modeled key-confirmation / matching-session authentication property.
This should not be read as saying that the signature algorithm or CertificateVerify mechanism is itself broken.
Rather, once the only key-exchange secret is available to the adversary, the Finished-key agreement property captured by the model no longer holds.
Under the same condition, hybrid key establishment still preserves these modeled properties as long as the (EC)DHE secret is not available to the adversary.
We believe this applies until a powerful CRQC exists which breaks **all** the bits of pre-quantum, where the condition of (EC)DHE being available to the adversary is violated.

A practical reading of the result is:

* Standalone ML-KEM has no second key-exchange component if `ss` is exposed or mishandled.
* Hybrid key establishment retains a surviving ECDHE component if `ss` is exposed but `gxy` remains secret.
* Implementation tests should therefore verify not only that the handshake succeeds, but that both hybrid components were present, validated, transcript-bound, and fail-closed before traffic secrets are derived.

The artifacts are available [here](https://github.com/symbolicsoft/reftls) for independent review.


## Computational Analysis

### Hybrids
{: #sec-tech-rat }

Technically, a proof of {{I-D.ietf-tls-hybrid-design-09}} is done in the computational model using CryptoVerif (cf. [ref](https://bblanche.gitlabpages.inria.fr/publications/BlanchetJacommeCSF24.pdf)). As per discussion on TLS mailing-list, it appears that the proof applies to the latest version of the spec {{I-D.ietf-tls-hybrid-design}}, as there seem to be no substantive changes from the perspective of formal proof.

### Standalone ML-KEM

Some existing computational analysis for standalone ML-KEM in TLS include [this](https://eprint.iacr.org/2021/844), [this](https://eprint.iacr.org/2024/1360), and [this](https://www.mdpi.com/1099-4300/27/12/1242).
All of these are based on pen-and-paper (computational) proofs.

For implementers, the practical reading of these analyses is a key-schedule and transcript-binding check.
The question is whether the KEM shared secret is introduced into TLS in a way that preserves the claimed security property.



# Security Considerations
{: #sec-sec-cons }

The whole document is about improving security considerations.

Like all security proofs, formal analysis is only as strong as its assumptions and model.
The scope is typically limited, and the model does not necessarily capture real-world deployment complexity, implementation details, operational constraints, or misuse scenarios.
Technically, formal proof only guarantees anything if all the assumptions hold, which is unlikely in practice.
Formal methods should be used as complementary and not as substitute of other analysis methods.

For implementations, this means that formal-methods results should be paired
with negative testing and review evidence for malformed shares, transcript
mismatches, silent fallback, premature secret derivation, and failure handling.



# IANA Considerations

This memo has no IANA actions.


--- back

[comment]: <> (# Acknowledgments)

# Contributors
{:numbered="false"}

Nadim Kobeissi performed a thorough formal analysis {{sec-results}} at high priority based on our call for analysis in previous versions of the memo to get a confirmation.

Text in {{sec-impl-negative-cases}} was proposed by Songbo Bu. He also proposed revisions in {{sec-gen-issues}}.

Text in {{sec-sec-cons}} is based on the proposal by John Preuß Mattsson.

{{sec-gen-issues}} is based on mailing-list discussion, referenced technical facets, and deployment questions raised during review.

We gratefully thank Yaakov Stein and Ilari Liusvaara for their substantial technical guidance, valuable feedback, and contributions.

# Acknowledgments
{:numbered="false"}

We thank Eric Rescorla, Brian E. Carpenter, Tibor Jager, and Eliot Lear for their valuable feedback.

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
* Comparison with hybrid key establishment

-02

* Added gap analysis
* What to model and analyze? {{sec-model-analyze}}
* Added FATT review is harmless
* Extended comparison with hybrid key establishment
* Opinion of designers {{sec-designers-view}}

-03

* Completely restructured and reframed after confirmation by formal analysis
* Added implementation-facing negative cases and argument matrix
* Some new arguments: implementation bugs, depth of hybrids, policy, all bits, which primitive breaks first?

-04

* Remove links to opinion of IETF participants
* Inform the reader of the technical facets
