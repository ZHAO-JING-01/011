title: "TODO - Your title"
abbrev: "TODO - Abbreviation"
category: info

docname: draft-ietf-spring-srv6-srh-extended-policy-key<br />
date:7 July 2024 <br />
consensus: true<br />
v: 00<br />
area: AREA<br />
workgroup: SPRING Working Group<br />
venue:
  group: Spring<br />
  type: Working Group<br />
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: J. Zhao
    organization: China Unicom
    email: zhaoj501@chinaunicom.cn

    fullname: W.X. Lve
    organization: China Unicom
    email: lvwx28@chinaunicom.cn




--- abstract

This paper defines a new extension header-SRv6 Policy Key to address the problems of timeliness and accuracy of controller-aware paths in SR-MPLS and SRv6 networks.The scheme enables network nodes to report path information to the controller by adding a path unique identifier to the message header, which ensures that the controller has a real-time and accurate picture of the SR path status, even if the SL tag is lost in transmission or the controller is unable to monitor it directly.This approach aims to optimise the network management efficiency and intelligent decision-making capability of SDN, especially in multipath tunnel configuration and load balancing scenarios, to improve network availability and O&M efficiency.


# Introduction

In SRv6 networks, the software-defined network (SDN) controller, as a core component, is responsible for the centralised management and dynamic configuration of network resources, and is the key to achieving network flexibility and intelligence.

Currently, the controller's perception of SRv6 message paths relies only on theoretical derivation, which has limitations in terms of timeliness and accuracy. Specifically, there is an inherent delay in the controller's update of the network state, and the state acquisition mechanism is occasionally abnormal, requiring a second refresh for confirmation, which constitutes a severe test for scenarios relying on real-time state for path computation and decision-making.Taking the multipath tunnel configuration as an example, ideally it should be able to dynamically adjust the traffic paths according to the master-standby relationship of the three preset tunnels and their priorities to ensure high availability and efficiency of the network. However, in practice, due to the lag in the controller's sensing state, it may not be able to instantly respond to the actual changes in the network links, resulting in inaccurate path inference, which affects the effectiveness of operations such as traffic counting and proper guidance. Further, in tunnel load balancing scenarios, where a single path contains multiple sub-paths running in parallel, the traffic distribution strategy based on hash rules or random allocation of devices improves bandwidth utilisation efficiency, but at the same time exacerbates the complexity of operation and maintenance monitoring, as the existing mechanism is difficult to track and verify the actual forwarding path of messages in real time.

In order to solve the above problems, this paper defines a new SRH extension header called "SRv6 Policy Key", which is used to identify the tunnel. This identification is carried by the message header and reported to the controller via the network node, which facilitates the controller to identify the SR path, thus enhancing the state-awareness capability of the controller in the SR, and ensuring that it can grasp the real-time network status more quickly and accurately, thus effectively assisting the operation and maintenance decision-making.




## Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals, as shown here.

## Terminology
SID: Segment ID. <br />
SRH: Segment Routing Header. <br />
SR-MPLS: Segment Routing with MPLS data plane.


# SRv6 Policy KEY
## Format of an SRv6 Policy KEY
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Next Header   |  Hdr Ext Len  | Routing Type  | Segments Left |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Last Entry   |     Flags     |              Tag              |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |            Segment List[0] (128-bit IPv6 address)             |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                           ...                                 |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |            Segment List[n] (128-bit IPv6 address)             |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                         SRv6 Policy KEY                       |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    //                                                             //
    //         Optional Type Length Value objects (variable)       //
    //                                                             //
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

## SRv6 Policy KEY TLV
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |     Type      |    Length     |   Flags       |  Reserved     |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                         Preference                            |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                        Policy Color                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                          Headend                              |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                          Endpoint                             |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
<center>Figure 2: SRv6 Policy KEY TLV</center>

Type：An 8-bit code point. <br />
Length: The length of the variable-length data field in bytes.6. <br />
Flags：8bit，marks list. <br />
Preference：32bit，marks SRv6 Policy Candidate Path. <br />
Policy Color：32bit，a Color of SRv6 Policy. <br />
Headend，128bit，first node of SRv6 Policy. <br />
Endpoint，128bit，destination address of SRv6 Policy.. 


# Use Cases for SRv6 Policy KEY
## Case 1:The controller cannot sense it in real time
SRv6 Policy Key addresses the challenges faced by controllers in real-time sensing of actual paths by providing unique identifiers for paths:
Actual path speculation is limited by path information sensing delay: facing the problem of network state sensing delay, especially the 1-minute delay in link state update and the need for secondary verification of initial detection errors, as well as the problem of untimely updating of network device configurations, the SRv6 Policy Key strengthens the controller's real-time path identification capability by providing a unique identifier for paths, effectively alleviating the challenge of relying on instantaneous information for path decision-making. path decision-making, ensuring the accuracy and efficiency of network control. <br />
This is reflected in two scenarios:<br />
--In the design of architectures with triple redundant tunnels, seamless switchover between master and standby requires precise path awareness of each path state to maintain service continuity.<br />
--Under the single-tunnel multipath policy, traffic is flexibly allocated based on link status and priority, requiring accurate path awareness for efficient traffic management and optimisation.
## Case 2:The controller cannot sense it in real time
In complex network load sharing scenarios, a single path is divided into three parallel sub-paths to jointly carry traffic, and its allocation is randomly executed by devices based on specific hash rules. Although this mechanism improves bandwidth utilisation, there is the problem of not being able to infer the actual path, which brings challenges to O&M management and fault troubleshooting. The controller can obtain real-time paths through the SRv6 Policy Key, overcoming the unpredictability of paths caused by the original random allocation.


# Functional Description
By accurately identifying the actual message transmission path, the management and control capability of the SDN controller can be effectively improved.

## Function1: Path Consistency Verification
The perception of actual paths ensures that the controller is able to accurately assess the consistency between the actual paths of data transmission and the predefined desired paths. This process involves systematically comparing the forwarding trajectories of network packets with the planned paths with the aim of identifying and resolving potential path deviations, thereby enhancing the reliability and efficiency of the network.
## Function2: Service flow analysis function
A node on the network can record the SRv6 Policy, Candidate Path, and List that pass through the node, and collect statistics based on the service logic at the three levels. When a node is upgraded or relocated, you can determine the services affected. Nodes on the network can collect traffic statistics based on the SRv6 Policy, Candidate Path, and List that pass through the node, and collect statistics based on the service logic of the three levels.
## Function3：Controller path visualization
The controller collects the packet header information of each node and performs statistical analysis to enhance the visual management of network path information.



# Security Considerations

TBD.


# IANA Considerations

This document has no IANA actions.


# Acknowledgments

TODO acknowledge.
