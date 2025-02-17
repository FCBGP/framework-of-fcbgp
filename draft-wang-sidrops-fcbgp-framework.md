---
title: "Framework of Forwarding Commitment BGP"
abbrev: "fcbgp-framework"
category: std

docname: draft-wang-sidrops-fcbgp-framework-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "SIDR Operations"
keyword:
 - BGP Security
 - Inter-Domain Forwarding

author:
  -
      fullname: Ke Xu
      org: Tsinghua University
      city: Beijing
      country: China
      email: xuke@tsinghua.edu.cn
  -
      fullname: Xiaoliang Wang
      org: Tsinghua University
      city: Beijing
      country: China
      email: wangxiaoliang0623@foxmail.com
  -
      fullname: Zhuotao liu
      org: Tsinghua University
      city: Beijing
      country: China
      email: zhuotaoliu@tsinghua.edu.cn
  -
      fullname: Li Qi
      org: Tsinghua University
      city: Beijing
      country: China
      email: qli01@tsinghua.edu.cn
  -
      fullname: Jianping Wu
      org: Tsinghua University
      city: Beijing
      country: China
      email: jianping@cernet.edu.cn

normative:
  RFC4271: # TEST
  RFC8205: # TEST
  RFC6480: # TEST
  RFC8209: # TEST


informative:


--- abstract

This document descibes a security framework based on Forwarding Commitment BGP(FC-BGP). FC serves as a versatile and extensible verification primitive, seamlessly integrating with existing hop-by-hop forwarding architectures, using a chain-based verification mechanism to ensure both end-to-end authenticity and operational validity across Autonomous Systems (ASes).By combining topological information and operational actions, FC-based framework provides a foundation for robust inter-domain routing and forwarding security.


--- middle

# Introduction

The Internet's routing and forwarding system relies on hop-by-hop mechanism to achieve global connectivity, providing significant scalability and flexibility. However, this paradigm also introduces security challenges such as route hijacking and forwarding path manipulation. The root cause lies in the mismatch between verification capabilities and information propagation range, which prevents end-to-end security while maintaining the flexibility of hop-by-hop processing. Due to "indirect" nature of hop-by-hop mechanism, the authenticity and integrity of information cannot be verified.

The existing representative mechanism BGPsec {{RFC8205}} aims to provide end-to-end verification by propagating onion-style signatures in BGP update messages. Nevertheless, this design conflicts with Internet's distributed governance and hop-by-hop processing philosophy, resulting in substantial deployment hurdles. Resource Public Key Infrastructure (RPKI) {{RFC6480}} is another key technology that reduces risk of route hijacking through cryptographic authentication of IP addresses and AS numbers, primarily by enabling Route Origin Validation (ROV). However, RPKI mainly focuses on verifying route origins rather than path security. Consequently, there is a lack of a unified and scalable verification mechanism.

This document specifies a security framework based on Forwarding Commitment BGP(FC-BGP). FC is a cryptographic signature code employed to validate an AS's routing intentions at its directly connected hop points. The incrementally deployable framework based on a universal verification primitive FC, is designed to be compatible with existing hop-by-hop forwarding architectures, while overcoming the limitations of current verification capabilities. By utilizing chain-based validation, it establishes an end-to-end verification mechanism across Autonomous Systems (ASes), ensuring authenticity and operational validity of information across ASes. This strategy enables enhanced trust propagation across the global Internet while maintaining the existing hop-by-hop forwarding paradigm, providing a secure and compatible framework to address the security dilemmas caused by mismatch between verification scope and information propagation range.

## Requirements Language

{::boilerplate bcp14-tagged}

# Overview

~~~~~~
+----+---+         +----+---+         +----+---+         +--------+
|  AS A  +-------->+  AS B  +-------->+  AS C  +-------->+  AS D  |
+----+---+  FC(A)  +----+---+  FC(A)  +----+---+  FC(A)  +----+---+
     ^                  ^      FC(B)       ^      FC(B)       ^    
     |                  |                  |      FC(C)       |    
     |                  |                  |                  |    
     |                  |                  |                  |    
   FC(A)              FC(B)              FC(C)              FC(D)                                                    
~~~~~~
{: #figure1 title="Overview of FC-BGP."}



An overview of FC-BGP is shown in {{figure1}}. The key primitive in FC-BGP is the Forwarding Commitment (FC), which is a publicly verifiable code that certifies an AS's routing intent on one of its directly connected hops, i.e., an FC indicates whether the AS is willing to carry traffic for a specific prefix over the hop.
Upon receiving a BGP announcement, if AS A decides to accept the route and extends the path to its (selected) neighbor AS B, AS A commits its routing intent by generating a cryptographically-signed FC. Therefore, downstream on-path ASes (such as AS B) can validate the correctness of a BGP update by checking the FCs associated with the individual hops on the AS-path. Because the FCs are designed to be hop-specific and path-agnostic, a deployed AS can immediately certify its routing intent regardless of the deployment status of other ASes. This is fundamentally different from existing path-level BGP authentication protocol (e.g., BGPsec) where an on-path AS cannot approve any form of routing intent unless all on-path ASes are upgraded.

FC-BGP is not bound to a specific FC propagation method and can use, but is not limited to, the following mechanisms:

1. Extend BGP Update Message. Assign a new BGP Update Path Attribute to carry FCs.
2. Propose a new propagation protocol that guarantees consistent FC propagation.
3. Use existing network infrastructure, such as extending RPKI to add a new signed object to store FCs.

Meanwhile, the flexibility of FCs further enables efficient forwarding validation on the dataplane. Specifically, because the FCs are self-proving, an AS can conceptually construct a certified AS-path using a list of consecutive per-hop FCs, and then binds its network traffic (identified by < src-AS, dst-AS, prefix >) to the path, such as < AS B, AS A, P(B) >, where P(B) is the prefix owned by AS B destined to AS A. This binding information essentially defines the authorized forwarding path for the traffic < AS B, AS A, P(B) >. Therefore, by advertising the binding information globally, both on-path and off-path ASes are aware of the desired forwarding paths so that they can collaboratively discard unwanted traffic that takes unauthorized paths.

Similar to FC propagation, the propagation of binding messages in FC-BGP is not restricted to specific methods and can be, but is not limited to, the following:

1. Propose a new propagation protocol that guarantees the consistency of binding messages.
2. Use existing network infrastructure, such as extending RPKI to add a new signed object to store binding messages.


# Forwarding Commitment

FC-BGP enhances the security of inter-domain routing and forwarding by building a publicly verifiable view of the forwarding commitments. At a high level, a routing commitment (FC) of an AS is a cryptographically-signed primitive that binds the AS's routing decisions (e.g. willing to forward traffic for a prefix via one of its directly-connected hops). With this view, ASes are able to:

1. Evaluate the authenticity (or security) of the control plane BGP announcements based on committed routing decisions from relevant ASes.
2. Ensure that the dataplane forwarding is consistent with the routing decisions committed in the control plane.

Upon receiving a BGP announcement, an upgraded AS generates a corresponding FC that contains the core information of the announcement, such as prefixes, sending AS, and receiving AS, and will be signed with the sender's private key for security.

# BGP Path Validation

~~~~~~~~~~~~~~~
                                            FClist:F(C,D,P)
                          FClist:F(B,C,P)          F(B,C,P)
       FClist:F(A,B,P)           F(A,B,P)          F(A,B,P)
     | AS Path:A       |  AS Path:A-B     | AS Path:A-B-C    |
     +---------------->+----------------->+----------------->|
     |                 |                  |                  |
     |                 |                  |                  |
     |                 |                  |                  |
+----+---+        +----+---+         +----+---+         +----+---+
|  AS A  +--------+  AS B  +---------+  AS C  +---------+  AS D  |
+----+---+        +----+---+         +----+---+         +----+---+
     |                 |                  |                  |
     |        F(C,D,P)-(src:P(D),dst:P(A))+<-----------------+
     |                 |                  |                  |
     |                 |                  |                  |
     |                 |<-----------------+------------------+
     |                 |      F(B,C,P)-(src:P(D),dst:P(A))   |
     |                 |                  |                  |
     |                 |                  |                  |
     |<----------------+------------------+------------------+
                   F(A,B,P)-(src:P(D),dst:P(A))
~~~~~~~~~~~~~~~
{: #figure2 title="Example of FC-BGP."}


Consider an illustrative example using the four-AS topology shown in {{figure2}}. In this process, FC-BGP generates the corresponding FC and propagates to downstream ASes (e.g., adding it to the Path Attributes of the BGP updates), so that the receiving AS can validate the authenticity of the announcement. Suppose AS C receives a BGP announcement P(A): A->B->C from its neighbor B. If AS C decides to further advertise this path to its neighbor D based on its routing policy, it generates a FC F(C,D,P), propagates it  to AS D, and forwards the BGP update message to D.

When AS D receives the route from C, it can determine the authenticity of the current AS path by verifying the list of FCs correctly reflects the AS path.

# Forwarding Validation

AS shown in {{figure2}}, to enable forwarding validation, ASes need to announce the traffic-FCs binding relationships. Specifically, suppose AS D confirms that the AS-path C->B->A for reaching prefix P(A) is legitimate, it binds the traffic (src:P(D),dst:P(A)) (where P(D) is the prefix owned by AS D) with the FC list F(C,D,P)-F(B,C,P)-F(A,B,P), and then publicly announces the binding relationship.

Upon receiving the relationship, other ASes can build traffic filtering rules based on the relationship to enable forwarding validation on dataplane. For instance, by interpreting the binding relationship produced by AS D, AS C confirms that the traffic (src:P(D), dst:P(A)) shall be forwarded over the link L(CD), and AS B confirms that the traffic shall be forwarded on link L(BC). Network traffic violating these binding rules is considered to take unauthorized paths.

To enable network-wide forwarding verification, these binding rules may be further broadcast globally (instead of just informing the ASes on the AS-path) so that off-path ASes can also discard unauthorized flows.

# Security Considerations

## Security Guarantees

When FC-BGP used in conjunction with origin validation, the following security guarantees can be achieved:

1. The source AS in a route announcement is authorized.
2. FC-BGP speakers on the AS-Path are authorized to propagate the route announcements.
3. The forwarding path of packets is consistent with the routing path announced by the FC-BGP speakers.

FC-BGP is designed to enhance the security of control plane routing and dataplane forwarding in the Internet at the network layer. Specifically, FC-BGP allows an AS to independently prove its BGP routing decisions with publicly verifiable cryptography commitments, based on which any on-path AS can verify the authenticity of a BGP path; meanwhile FC-BGP ensures the consistency between the control plane and dataplane, i.e., the network traffic shall take the forwarding path that is consistent with the control plane routing decisions, or otherwise be discarded. More crucially, the above security guarantees offered by FC-BGP are not binary, i.e., secure or non-secure. Instead, the security benefits are strictly monotonically increasing as the deployment rate of FC-BGP (i.e., the percentage of ASes that are upgraded to support FC-BGP) increases.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
