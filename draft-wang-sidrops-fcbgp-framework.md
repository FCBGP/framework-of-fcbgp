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

# Role of FC in FC-based Framework

The key element of FC-BGP security framework is a transferable universal verification primitive-Forwarding Commitment (FC). The goal of FC is to achieve end-to-end verification within routing and forwarding system while maintaining compatibility with hop-by-hop processing paradigm.

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
{: #figure1 title="Integration of FC with Routing and Forwarding Path."}

The integration of FC with routing and forwarding path is shown in {{figure1}}. During hop-by-hop processing, each AS generates an FC based on its routing intent. The FC provides a verifiable proof through cryptographic signatures, encapsulating routing behavior committed at directly connected hop, including information about previous hop AS, current AS, next hop AS and IP prefix, ensuring the authenticity and trustworthiness of provided routing information. Subsequent AS receiving this information can verify FC from previous AS through a chain-based validation mechanism before appending its own FC information, ensuring authenticity and validity of information. The design of chain-based validation ensures security of information during hop-by-hop propagation, preventing tampering by intermediate nodes or inconsistencies in policies from compromising end-to-end security.

FC based on hop-by-hop verification mechanism, enables chain-based validation across multiple ASes. Each FC generated by an AS can be independently verified without relying on updates from other ASes or cooperation from global network. It also supports end-to-end verification across the entire network. By constructing FC chains composed of multiple FCs between ASes, it can be used not only for individual hop-by-hop verification but also for building path chains that support global path validation, providing a foundation for end-to-end security.

FC provides flexible propagation methods within FC-based framework, which can be achieved by extending existing BGP messages, proposing new propagation protocols, or leveraging existing network infrastructures such as RPKI for propagation. This flexibility allows FC to be compatible with existing network infrastructures and enables gradual deployment and adoption.

# Semantic Structure of FC

~~~~~~~~~~~~~~~
+------------------+ +------------------+ +-----------------+   \
|Route Announcement| | Forwarding Path  | | Routing Policy  |... >Operation
+------------------+ +------------------+ +-----------------+   /
+--------------------------------------------------------------+\
|                          BGP Peering                         | |
+--------------------------------------------------------------+  >Topology
+--------------------------------------------------------------+ |
|                        Link Statement                        | |
+--------------------------------------------------------------+/
~~~~~~~~~~~~~~~
{: #figure2 title="Semantic Structure of FC."}

The uniqueness of FC-based framework lies in FCsemantic design, as shown in {{figure2}}, which encompasses two main types of information: topological information and operational information. This design makes FC not just a supplement to existing security mechanisms, but a fundamental primitive that supports construction of comprehensive security frameworks.

Topological Information: Topological information forms core of FC and is divided into two layers:
1. Link Statement: FC expresses physical connections between ASes through a Link Statement, which records direct physical links such as fiber connections or dedicated lines. This information provides direct evidence of validity of BGP peering relationships. If there is no physical connection between two ASes, they should not establish a BGP peering relationship. The Link Statement ensures objectivity and verifiability of physical connections, eliminating routing announcements based on false connections.

2. BGP Peering: Based on physical connection states, FC further captures logical BGP peering relationships established over these connections. By including previous hop, current AS, and next hop, FC describes inter-AS routing relationships in detail, enabling fine-grained, topology-based chained validation. This helps verify whether an AS's routing announcement aligns with its BGP peering relationship, ensuring that AS has genuinely learned route information from a neighboring AS, preventing route hijacking attacks.

Operational Information: FC encapsulates rich operational semantics that describe specific actions performed by an AS in routing and forwarding processes. It maily includes the following:

1. Route Announcement: An AS announces its routing prefixes and path information to external peers. FC can represent these route announcements within directly connected hop range and provide verifiable commitments to global network, helping prevent route hijacking.

2. Forwarding Path Designation: FC defines transmission path that packets should follow, which is critical for data plane security. By embedding forwarding path information, FC ensures that packets are forwarded as expected, preventing path manipulation or malicious redirection.

3. Routing Policy: FC captures the policies of an AS regarding how to select and propagate paths, ensuring the consistency of routing policies and helping to identify and prevent route leaks and configuration errors.

By integrating topological and operational information, FC expresses behaviors ranging from AS level to global level. Each FC represents routing and forwarding actions that occur within directly connected hop range, and by linking multiple FCs, chained validation can be achieved across scope of multiple ASes. This mechanism ensures comprehensive verification of network operations, guaranteeing consistency and authenticity of information across entire network interaction range.

In chain-based validation, FC is linked into a chain using cryptographic signatures, with each AS verifying the previous FC before appending its own information. This creates a trust chain that spans multiple ASes. This mechanism ensures end-to-end verification while maintaining decentralized nature of Internet.

FC is compatible not only with existing BGP protocols but also supports future routing validation mechanisms, such as RPKI. By leveraging FC as a foundational structure, it strengthens routing information and enhances security and reliability of network communication.

# Security Mechanisms Based on FC

By introducing universal verification primitive FC, FC-based security framework combines independence of hop-by-hop forwarding with end-to-end security verification capability. Below are some security mechanisms and application scenarios enabled by FC:

1. Routing Verification of Incremental Deployment 
FC supports gradual deployment, allowing verification within networks that are not fully deployed. Through hop-by-hop and chain-based verification, FC adapts to partial deployment scenarios, reducing overhead of global verification.

2. Consistency Between Control Plane and Data Plane
By combining BGP peering relationships with physical link information, FC significantly enhances data plane security. For instance, embedding forwarding path information within FC ensures that packets are forwarded along expected routes, preventing path manipulation or malicious detours.

3. Route Leak Prevention and Policy Transparency
FC's operational information allows ASes to define and validate their routing policies, effectively preventing route leaks caused by misconfigurations or malicious policy violations.

4. Compatibility with Future Network Architectures
FC's universal structure remains compatible with both existing and future routing protocols, supporting emerging paradigms such as intent-based networking.

5. Dynamic Defense Mechanisms Against Emerging Attacks
FC's flexibility allows it to adjust to new attack scenarios, dynamically optimizing granularity of FC information to address evolving security challenges.

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
