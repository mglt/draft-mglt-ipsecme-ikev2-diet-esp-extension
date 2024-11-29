---
title: Internet Key Exchange version 2 (IKEv2) extension for Header Compression Profile (HCP) 
abbrev: EHC extension
docname: draft-ietf-ipsecme-ikev2-diet-esp-extension-02
ipr: trust200902
area: Security
wg: IPsecme
kw: Internet-Draft
cat: std
stream: IETF

pi:
  toc: yes
  sortrefs: yes
  symrefs: yes

author:
  -
    ins: D. Migault
    name: Daniel Migault
    org: Ericsson
    email: daniel.migault@ericsson.com
  -
    ins: T. Guggemos
    name: Tobias Guggemos
    org: LMU
    email: guggemos@nm.ifi.lmu.de
  -
    ins:  D. Schinazi
    name:  David Schinazi
    org: Google LLC
    email: dschinazi.ietf@gmail.com
  -
    ins: W. Atwood
    name: J. William Atwood
    org: Concordia University
    email: william.atwood@concordia.ca
  -
    ins: D. Liu
    name: Daiying Liu
    org: Ericsson
    email: harold.liu@ericsson.com
  -
    ins: S. Preda
    name: Stere Preda
    org: Ericsson
    email: stere.preda@ericsson.com
  -
    ins: M. Hatami
    name: Maryam Hatami
    org: Concordia University
    email: maryam.hatami@mail.concordia.ca
  -
    ins: S. Céspedes
    name: Sandra Céspedes
    org: Concordia University
    email: sandra.cespedes@concordia.ca



---

[![hackmd-github-sync-badge](https://hackmd.io/P414llRtRz-0KYOHCh9Qfw/badge)](https://hackmd.io/P414llRtRz-0KYOHCh9Qfw)
 abstract

This document describes an IKEv2 extension for Header Compression to agree on Attributes for Rules Generation. 
This extension defines the necessary registries for the ESP Header Compression Profile (EHCP) Diet-ESP. 

--- middle


#  Requirements notation

{::boilerplate bcp14}

# Introduction

The ESP Header Compression Profile (EHCP) {{!I-D.ietf-ipsecme-diet-esp}} minimizes the overhead associated with ESP by compressing both the ESP header and additional fields within the secured packet. EHCP utilizes Attributes for Rules Generation (AfRG) that are specified for each Security Association (SA). Certain AfRG have already been established during the SA negotiation process through IKEv2. This extension facilitates the agreement on the remaining AfRG through IKEv2.

# Protocol Overview

As illustrated in {{fig-overview}}, an initiator intending to utilize the Header Compression Profile (HCP) informs its peer by sending a HCP_SUPPORTED Notify Payload during the IKE_AUTH and CREATE_CHILD_SA exchanges. The HCP_SUPPORTED includes a list of Proposal payloads, each comprising an EHCP Name along with a set of Attributes for Rules Generation (AfRG){{!I-D.ietf-ipsecme-diet-esp}}. Any AfRG for which the initiator wishes to specify no limitations SHOULD be excluded, i.e., an AfRG is only sent if the sending peer wants the receiving peer to select a subset of the available values. A given AfRG MAY be repeated with different values in order to provide a list of acceptable values. A range of possible AfRG values MAY be indicated as well.

If a Proposal contains an unknown HCP Name, or any AfRG in a Proposal is unknown, then the entire Proposal must be discarded by the responder. If none of the received Proposals are deemed acceptable, the responder MAY choose to discard the HCP_SUPPORTED Notify Payload. Nevertheless, it is anticipated that the responder will provide an explanation for rejecting all HCP Proposals. If the reason pertains to an AfRG with an unacceptable value, the responder SHOULD reply with an HCP_UNSUPPORTED Notify Payload. This Notify Payload SHOULD include one or more acceptable Proposal Payloads to guide the initiator.

Conversely, if the receiver identifies a suitable Proposal, it will respond with an HCP_SUPPORTED Notify Payload that includes the chosen Proposal. In cases where the AfRG was not explicitly stated, the responder will provide the AfRG unless it defaults to a standard value. Each AfRG MUST NOT be mentioned more than one time. When multiple values are provided for a specific AfRG (either multiple values being provided or via a range of acceptable values), the responder MUST NOT provide more than one value. The Proposal MUST NOT contain any range of AfRG.

Upon receipt of an HCP_UNSUPPORTED Notify Payload, the initiator has the option to restart the CREATE_CHILD_SA exchange.

When the initiator receives the HCP_SUPPORTED Notify Payload, it will evaluate the Proposal to ensure that it aligns with the initial proposal and adheres to its policies prior to executing the HCP.

~~~
Initiator                         Responder
-------------------------------------------------------------------
HDR, SA, KEi, Ni -->
                           <-- HDR, SA, KEr, Nr
HDR, SK {IDi, AUTH,
     SA, TSi, TSr,
     N(HCP_SUPPORTED
         Proposal_ID=1, HCP Name="Diet-ESP"
           AfRG_a
           ...
           AfRG_i
         ...
         Proposal_ID=2, HCP Name="Diet-ESP"
           AfRG_a
           ...
           AfRG_j)
                           <-- HDR, SK {IDr, AUTH,
                                    SA, TSi, TSr,
                                    N(HCP_SUPPORTED
                                      Proposal_ID=2, HCP Name="Diet-ESP"
                                        AfRG_a      
                                        ...
                                        AfRG_j, 
                                        AfRG_k, 
                                        ...
                                        AfRG_u)
~~~
{: #fig-overview artwork-align="center" title="The parameters for Diet-ESP have been established through the HCP_SUPPORTED Notify exchange. In this instance, the responder has opted for the second Proposal, which includes the specified Attributes for Rules Generation (AfRG). Any absent AfRG will default to its predetermined values."}



# HCP_SUPPORTED and HCP_UNSUPPORTED Notify Payloads

{{fig-notify}} describes the HCP_SUPPORTED and HCP_UNSUPPORTED Notify Payload. 

~~~
                       1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Next Payload  |C|  RESERVED   |         Payload Length        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Protocol ID  |   SPI Size    |      Notify Message Type      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-notify artwork-align="center" title="Notify Payload"}

The fields Next Payload, Critical Bit, RESERVED, and Payload Length are defined in section 3.10 of {{!RFC7296}}.

Protocol ID (1 octet):
: set to zero.

SPI Size (1 octet):
: set to zero.

Notify Message Type (2 octets):
: Specifies the type of notification message. It is set to TBA1 for HCP_SUPPORTED and TBA2 for HCP_UNSUPPORTED

When sent by the Initiator, the HCP_SUPPORTED Notify Payload contains a list of Proposal payloads described in {{fig-proposal}}. When sent by the responder the HCP_SUPPORTED Notify Payload contains a single Payload described in {{fig-proposal}}. 

~~~
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Proposal ID  |   HCP Name   |      Proposal Length           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
~                          Proposal Data                        ~
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-proposal artwork-align="center" title="Proposal Payload"}
 
Proposal ID (1 octet):
: The number identifying the Proposal. 

EHCP Name (2 octets): 
: The identifier of the EHCP Name. (see {{tab:hcp-name}}) 

Proposal Length (2 octets):
: The length in octets  of the Proposal Data

Proposal Data:
: A Proposal contains a set of parameters that are represented via Transform Attribute format {{!RFC7296, Section 3.3.5}} and detailed further as described in {{sec-parameters}}.

# Attributes for Rules Generation {#sec-parameters}


Attributes for Rules Generation (AfRG) follow the same format as the Transform Attribute {{!RFC7296, Section 3.3.5}} copied for convenience in {{fig-attribute}}.

~~~
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|A|       Attribute Type        |    AF=0  Attribute Length     |
|F|                             |    AF=1  Attribute Value      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                   AF=0  Attribute Data                        |
|                   AF=1  Not Transmitted                       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-attribute artwork-align="center" title="Transform Attribute Payload"}

There are two types of AfRG: 1) AfRG that are specific to a given HCP and 2) generic AfRG. 

This specification defines range_afrg_proposal as a Generic Attribute for Rules Generation to specify that a given AfRG can be selected within a range of values.

* Designation: range_afrg_proposal 
* Has Associated Data: YES (AF=0)
* Attribute Data: Let AfRG_min and AfRG_max be the minimum and maximum values of the proposed range, expressed following the Transform Attribute Payload format. The corresponding Attribute Data is the concatenation of AfRG_min and AfRG_max. 


# Registering a Header Compression Profile {#sec-reg}

An HCP needs to register an HCP Name taken from {{tab:hcp-name}} in {{sec:hcp-name}}, the specification that describes the operations of the EHCP, as well as the different AfRG. For each AfRG, the corresponding Attribute Type, the AF value, the Attribute Data and the Default Value MUST be specified.


# Registration of Diet-ESP EHCP 

This section defines the code points that are needed to agree on the AfRG between two IKEv2 peers as described in {{sec-reg}}.

* HCP Name: "Diet-ESP" as specified in {{tab:hcp-name}}, {{sec:hcp-name}}. 
* Specification : {{!I-D.ietf-ipsecme-diet-esp}}

The following Attributes for Rules Generation are defined: 

DSCP Compression/Decompression Action (CDA)

* Designation: dscp_cda
* Has Associated Data: YES (AF=0)
* Attribute Data: DSCP CDA takes discrete values coded over one byte as described in DSCP CDA Value Registry  ({{tab:dscp_cda}} in {{sec:dscp_cda}})
* Default Value: the default value is set to "uncompress" 

ECN Compression/Decompression Action (CDA)

* Designation: ecn_cda
* Has Associated Data: YES (AF=0)
* Attribute Data: ECN CDA takes discrete values coded over one byte as described in the ECN CDA Value Registry ({{tab:ecn_cda}} in {{sec:ecn_cda}})
* Default Value: the default value is set to "uncompress" 

Flow Label  Compression/Decompression Action (CDA)

* Designation: flow_label_cda
* Has Associated Data: YES (AF=0)
* Attribute Data: Flow Label CDA takes discrete values coded over one byte as described in the Flow Label CDA Value Registry ({{tab:fl_cda}} in {{sec:fl_cda}})
* Default Value: the default value is set to "uncompress" 

OS or Network Bit Alignment

* Designation: alignment
* Has Associated Data: YES (AF=0)
* Attribute Data: Byte Alignment takes discrete values coded over one byte as described in the Bit Alignment Value Registry ({{tab:align}} in {{sec:align}})
* Default Value: the default value is set to "32 bit", which corresponds to the standard IPv6 bit alignment

Security Policy Index (SPI) Least Significant Bits (LSB)

* Designation: esp_spi_lsb
* Has Associated Data: YES (AF=0)
* Attribute Data: SPI LSB designates the number of bits that are provided to infer the SPI. This number is between 0 and 32. 
* Default Value: the default value is 32, which is the size of the standard SPI in the standard ESP

Sequence Number (SN) Least Significant Bits (LSB)

* Designation: esp_sn_lsb
* Has Associated Data: YES (AF=0)
* Attribute Data: SN LSB designates the number of bits that are provided to infer the SPI. This number is between 0 and 32. 
* Default Value: the default value is 32, which is the size of the standard SN in the standard ESP



# IANA Considerations


## Registration of IKEv2 Notify Message Types 

IANA has allocated two values in the "IKEv2 Notify Message Types - Status Types" registry:

~~~
  Value    Notify Messages - Status Types
-----------------------------------------
  TBA1    HCP_SUPPORTED
  TBA2    HCP_UNSUPPORTED
~~~

This specification requests the IANA to create an IKEv2 Header Compression registry (see {{sec:hcp-name}}), as well as the necessary registries for the ESP Header Compression Profile Diet-ESP, that is the Attribute for Rules Generations (see {{sec:afrg}}) as well as, when required, the complementary specific AfRG Values associated with each AfRG (see {{sec:afrg-val}}). 

All registries are "Specification Required".  


## Registry for Generic Attributes for Rules Generation {#sec:gen-afrg} 

Registry for Generic Attributes for Rules Generation. When Associated Data is set to YES, the AF bit of the corresponding Transform Attribute Payload is set to 0; otherwise it is set to 1. The AfRG Code Point mentioned here MUST NOT be reused by any Registries associated with any Profile and is shared by all profiles.


|  AfRG Code Point | Full Name      |  Designation     | Has Associated Data | Reference
|------------------|----------------|------------------|---------------------|----------
|  65535           | RANGE AfRG     | range_afrg       | YES                 | ThisRFC
{: #tab:gen-afrg}

## Registry for IKEv2 Header Compression Profile {#sec:hcp-name}


| Value (1 Byte) | Designation | Reference | 
|----------------|-------------|-----------|
|  0             | Diet-ESP    | ThisRFC   |
|  1-255         | unallocated |  -        |
{: #tab:hcp-name}

## Registry for Diet-ESP Attributes for Rules Generation {#sec:afrg} 

Registry for Attributes for Rules Generation for the ESP Header Compression Profile Diet-ESP. When Associated Data is set to YES, the AF bit of the corresponding Transform Attribute Payload is set to 0; otherwise it is set to 1.

|  AfRG Code Point | Full Name      |  Designation     | Has Associated Data | Reference
|------------------|----------------|------------------|---------------------|----------
|  0               | DSCP CDA       | dscp_cda         | YES                 | ThisRFC
|  1               | ECN CDA        | ecn_cda          | YES                 | ThisRFC
|  2               | Flow Label CDA | flow_label_cda   | YES                 | ThisRFC
|  3               | Alignment      | alignment        | YES                 | ThisRFC 
|  4               | SPI LSB        | esp_spi_lsb      | YES                 | ThisRFC 
|  5               | SN  LSB        | esp_spi_sn       | YES                 | ThisRFC 
|  6 - 2^16-2      | unallocated    |     -            |        -            |    -    
{: #tab:afrg}


## Registries for the Values of Diet-ESP Attributes for Rules Generation {#sec:afrg-val} 


### DSCP CDA Value Registry  {#sec:dscp_cda}

Value      | Designation | Reference | 
-----------|-------------|-----------|
  0        | uncompress  | ThisRFC   |
  1        | lower       | ThisRFC   |
  2        | sa          | ThisRFC   | 
  3-255    | unallocated |    -      |
{: #tab:dscp_cda}

### ECN CDA Value Registry {#sec:ecn_cda}

Value      | Designation | Reference | 
-----------|-------------|-----------|
  0        | uncompress  | ThisRFC   |
  1        | lower       | ThisRFC   |
  2-255    | unallocated |    -      |
{: #tab:ecn_cda}

### Flow Label CDA Value Registry {#sec:fl_cda}
 
Value      | Designation | Reference | 
-----------|-------------|-----------|
  0        | uncompress  | ThisRFC   |
  1        | lower       | ThisRFC   |
  2        | generated   | ThisRFC   |
  3        | zero        | ThisRFC   |
  4-255    | unallocated |    -      |
{: #tab:fl_cda}

### OS or Network Byte Alignment {#sec:align}

Value      | Designation | Reference | 
-----------|-------------|-----------|
  0        | 8 bit       | ThisRFC   |
  1        | 16 bit      | ThisRFC   |
  2        | 32 bit      | ThisRFC   |
  3        | 64 bit      | ThisRFC   |
  4-255    | unallocated |    -      |
{: #tab:align}

# Security Considerations

The protocol defined in this document does not modify IKEv2. 

Proposals may be expressed in various ways and a proposal may be expressed in a specific way so that its treatment overloads the receiver. The receiver needs to consider aborting the exchange when too much resource is required.





