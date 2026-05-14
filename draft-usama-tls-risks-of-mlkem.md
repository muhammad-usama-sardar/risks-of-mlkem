---
title: "Risks of Standalone ML-KEM in TLS 1.3"
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
    organization: TU Dresden
    email: "muhammad_usama.sardar@tu-dresden.de"

normative:
  NistFips203: DOI.10.6028/NIST.FIPS.203
  I-D.ietf-tls-mlkem:

informative:

...

--- abstract

We attest that standalone ML-KEM in TLS breaks the existing proofs of TLS 1.3 in ProVerif.
We also attest that this is a much bigger change than RFC8773bis, which indeed went for FATT review.
We, therefore, kindly ask for FATT review of standalone ML-KEM in TLS.


--- middle

# Introduction

Readers are assumed to be familiar with {{NistFips203}} and {{I-D.ietf-tls-mlkem}}.

We assert that the security considerations of {{I-D.ietf-tls-mlkem}} are insufficient.
We believe that symbolic and computational analysis of ML-KEM in the context of TLS is helpful here.
We request that if the author has done any formal analysis, it would be helpful to present the current state of formal analysis in the next meeting for discussion.


## Motivation

The draft aims to formally study the security of standalone ML-KEM in TLS 1.3 {{I-D.ietf-tls-mlkem}}.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


## Where ProVerif Proofs Break
{: #sec-ml-kem }

While ML-KEM {{I-D.ietf-tls-mlkem}} looks like just a "trivial" addition, it makes changes as deep as the key schedule of TLS. It essentially replaces the *key exchange* by *key encapsulation*. While the former is symmetric, the latter is asymmetric.
This symmetry is in terms of exchange of roles, and that the order does not matter.
The proof in ProVerif is, therefore, based on the commutativity of the components g<sup>x</sup> and g<sup>y</sup>, where g<sup>x</sup> and g<sup>y</sup> represent the public keys of the endpoints.
In ProVerif syntax:
(see details [here](https://github.com/CCC-Attestation/formal-spec-id-crisis/blob/6c3d17a428198aa058f805d16fe6baef7894028f/TLS-a/fix/tls-lib-simple.pvl#L38-L41))

~~~
fun dh_ideal(element,bitstring):element.
equation forall x:bitstring, y:bitstring;
	 dh_ideal(dh_ideal(G,x),y) =
	 dh_ideal(dh_ideal(G,y),x).
~~~

Key encapsulation does not enjoy this property. There is essentially only one endpoint (say client) which generates the key pair `(dk,ek)` where `dk` represents the secret decapsulation key and `ek` represents the public encapsulation key.
As opposed to both endpoints sending their public keys g<sup>x</sup> and g<sup>y</sup> in key exchange, only one of the endpoints (client in above example) sends the public encapsulation key. This asymmetry breaks the existing proofs of TLS 1.3 in ProVerif and requires a new proof.

## Current Status

{{I-D.ietf-tls-mlkem}} had an opposition of several (ca. 25 in our understanding) WG members in the last WGLC. We see 2 possible options:

* Continue tabletop discussions on subjective calculation of risks, costs, tradeoffs, etc., and keep burning WG energy.
* Do some technical analysis using formal methods (such as symbolic and computational) to get a confirmation on the security of ML-KEM in the context of TLS and offer a statement for security considerations, and move on to more critical works like hybrid authentication.

We believe the former cannot resolve the dispute. We believe the latter *may* help.

~~~
We believe the security considerations of {{I-D.ietf-tls-mlkem}} are
insufficient. We also believe FATT review could have significantly
improved it, including but not limited to the preference of hybrids,
and potential issues regarding KEM binding in TLS.
We have provided significant feedback during the two WGLCs. However,
almost none of that is actually reflected in the updated editor's
version.
~~~

### "Cost"
"Cost" has been presented on the list as the motivation for ML-KEM but no reference has yet been presented.
We believe costs will depend on several factors and it is quite subjective.
There seems to be a need for a thorough study to understand the "cost."
We invite the WG participants to perform this analysis and share the results with the WG.

## ML-KEM: FATT Review
{: #sec-sol-ml-kem }

We have formally requested the chairs to initiate the FATT process for {{I-D.ietf-tls-mlkem}}.
See [this](https://mailarchive.ietf.org/arch/msg/tls/rClgrWm2hnhESXHx56U8InbwQQs/) and [this](https://mailarchive.ietf.org/arch/msg/tls/7lj6fYAweMBwNMxFerNl7xhY0pk/).

### Expected Learning
We believe formal methods can provide additional value for security considerations of this draft in order to maintain the high cryptographic assurance of TLS.
Since we have no guarantee on whether ECDHE will break before ML-KEM, it seems appropriate to do thorough cryptographic analysis.
We believe the Harvest Now, Decrypt Later (HNDL) attack applies equally well to standalone ML-KEM.
Adversary can record all traffic and decrypt it when ML-KEM is broken (or probably it is already broken; who knows?)

* As an example, it can help justify design choices, such as the preference for hybrids.
It can help identify ways in which ML-KEM can break.
It can also help identify all the assumptions under which the properties hold.
* As a relevant data point in the context of standardization, LAKE WG has done formal analysis for EDHOC-PSK with KEM ([ref](https://mailarchive.ietf.org/arch/msg/lake/2XGOI9OCwylJUfSCasvvwM2FXmw/)).
* *Computational* analysis (cf. [SoK](https://eprint.iacr.org/2019/1393.pdf))-- using tools such as CryptoVerif -- seems like a reasonable approach to ensure security of ML-KEM in TLS, such as binding.

### Formal Analysis (Work-in-progress)
We have presented observation from our ongoing symbolic security analysis (cf. limitations in {{sec-sec-cons}}) using ProVerif on the mailing list.

We argue that in general:

1. Migration from ECDHE to hybrid is security improvement.
2. Migration from hybrid to standalone ML-KEM is security regression.


#### Hybrid PQ/T

More formally, the property hybrid PQ/T should provide is:

~~~
Hybrid PQ/T is secure unless both ECDHE and ML-KEM are broken.
~~~

Hybrid preserves ECDHE, and adds ML-KEM as an additional factor.
So as long as one of them is not broken, the system is secure.
In particular, even if ML-KEM is completely broken, the system retains the security level of ECDHE.

#### Standalone PQ

On the other hand, the formal property standalone PQ provides is:

~~~
Standalone PQ is secure unless ML-KEM is broken.
~~~

If ML-KEM is broken, the whole system is broken.

#### Comparison
Leak out the ECDHE key from hybrid PQ/T and you get a standalone ML-KEM.
Clearly, hybrid isin general more secure, unless ECDHE is fully broken, in which case it still falls equivalent to standalone ML-KEM, or in the hypothetical scenario that there is an implementation bug in the ECDHE part which is triggered only in composition.

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
