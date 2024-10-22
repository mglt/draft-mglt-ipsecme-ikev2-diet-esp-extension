---
title: Internet Key Exchange version 2 (IKEv2) extension for Header Compression Profile (HCP) 
abbrev: EHC extension
docname: draft-ietf-ipsecme-ikev2-diet-esp-extension-01
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



--- abstract

This document describes an IKEv2 extension for Header Compression to agree on Attributes for Rules Generation. 
This extension define sthe necessary registries for the ESP Header Compression Profile (EHCP) Diet-ESP. 

--- middle


#  Requirements notation

{::boilerplate bcp14}

# Introduction

ESP Header Compression Profile (EHCP) {{!I-D.mglt-ipsecme-diet-esp}} reduces ESP overhead by compressing the ESP and other fields of the protected packet.
EHCP takes Attributes for Rules Generation (AfRG) defined for each Security Association (SA).
Some of these AfRG have already been agreed during the negotiation of the SA via IKEv2.
This extension enable the remaining AfRG to be agreed via IKEv2. 

# Protocol Overview

As depicted in {{fig-overview}}, an initiator willing to apply Header Compression Profile (HCP) notifies its peer with a HCP_SUPPORTED Notify Payload in its IKE_AUTH and CREATE_CHILD_SA exchange. 
The HCP_SUPPORTED contains a list of Proposals payloads, each of which contains an EHCP Name and a list of Attributes for Rules Generation (AfRG){{!I-D.mglt-ipsecme-diet-esp}}. AfRG for which the initiator has no restrictions SHOULD be omitted. 

Any Proposal for which HCP Name or one of the mentionned AfRG is unknown to the initator MUST be ignored. 
If none of the received Proposal is acceptable, the responder MAY ignore the HCP_SUPPORTED Notify Payload. However, it is expected that it sends an indication on the reason for refusing all HCP Proposals. If the reason is an AfRG that has not an acceptable value, it SHOULD respond with HCP_UNSUPPORTED Notify Payload. This Notify Payload SHOULD contain one or multiple acceptable Proposal Payloads as an hint for the initiator.
Otherwise, the receiver selects one of the Proposal and responds with a HCP_SUPPORTED Notify Payload which contains the selected Proposal. When the AfRG was not explicitly provided, the responder explicitly provides the AfRG unless it takes a default value.

Upon receiving an HCP_UNSUPPORTED Notify Payload, the initiator may restart the CREATE_CHILD_SA exchange. 

Unpon receiving the HCP_SUPPORTED Notify Payload, the initiator takes, the Proposal, checks it matches the initial proposal and its policies before implementing the HCP itself. 


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
{: #fig-overview artwork-align="center" title="Diet-ESP parameters agreed via the HCP_SUPPORTED Notify exchange. In this example, the responder has selected the second Proposal in which Attributes for Rules Generation (AfRG) are specified. The missing AfRG are considered taking their default value."}



# HCP_SUPPORTED and HCP_UNSUPPORTED Notify Payloads

{{fig-notify}} describes the HCP_SUPPORTED and HCP_UNACCEPTABLE_PARAMETER Notify Payload. 

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

When sent by the Initiator, the HCP_SUPPORTED Notify Payload contains a list of Proposal payloads described by {{fig-proposal}}. When sent by the responder the HCP_SUPPORTED Notify Payload contains a single Payload described by {{fig-proposal}}. 

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
: The length in octet  of the Proposal Data

Proposal Data:
:A Proposal contains a set of parameters that are represented via Transform Attribute format {{!RFC7296, Section 3.3.5}} and detailed further as described in {{sec-parameters}}.

# Attributes for Rules Generation {#sec-parameters}


Attributes for Rules Generation (AfRG) follow the same format as the Transform Attribute {{!RFC7296, Section 3.3.5}} reminded for convenience by  

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



# Registrating a Header Compression Profile

An HCP needs to register a HCP Name in {{tab:hcp-name}}, the specification that describes the operations associated to the EHCP, as well as the different AfRG. For each AfRG, the corresponding Attribute Type, AF value and Attribute Data MUST be specified.


# Registration of EHCP Diet ESP

This section defines the code points that are needed to agree on AfRG between two IKEv2 peers.

* HCP Name: "Diet-ESP" as specified in {{tab:hcp-name}}. 
* Specification : {{!I-D.mglt-ipsecme-diet-esp}}

The following Attributes for Rules Generation are defined: 

DSCP Compression/Decompression Action (CDA)
* Designation: dscp_cda
* Has Associated Data: YES (AF=1)
* Attribute Data: DSCP CDA takes discret values coded over one byte as described in DSCP CDA Value Registry  {{tab:dscp_cda}}
* Default Value: the default value is set to "uncompress" 

ECN Compression/Decompression Action (CDA)
* Designation: ecn_cda
* Has Associated Data: YES (AF=1)
* Attribute Data: ECN CDA takes discret values coded over one byte as described in the ECN CDA Value Registry {{tab:ecn_cda}}
* Default Value: the default value is set to "uncompress" 

Flow Label  Compression/Decompression Action (CDA)
* Designation: flow_label_cda
* Has Associated Data: YES (AF=1)
* Attribute Data: Flow Label CDA takes discret values coded over one byte as described in the Flow Label CDA Value Registry {{tab:fl_cda}}
* Default Value: the default value is set to "uncompress" 

OS or Network Bit Alignment
* Designation: alignment
* Has Associated Data: YES (AF=1)
* Attribute Data: Byte Alignment takes discret values coded over one byte as described in the Bit Alignment Value Registry {{tab:align}}
* Default Value: the default value is set to "32 bit" which correspond to the standard IPv6 bit alignment

Security Policy Index (SPI) Least Significant Bits (LSB)
* Designation: esp_spi_lsb
* Has Associated Data: YES (AF=1)
* Attribute Data: SPI LSB designates the number of bits that are provided to infer the SPI. This number is between 0 and 32. 
* Default Value: the default value is 32 which is the size of teh standard ESP

Sequence Number (SN) Least Significant Bits (LSB)
* Designation: esp_sn_lsb
* Has Associated Data: YES (AF=1)
* Attribute Data: SN LSB designates the number of bits that are provided to infer the SPI. This number is between 0 and 32. 
* Default Value: the default value is 32 which is the size of teh standard ESP


# IANA section


IANA has allocated two values in the "IKEv2 Notify Message Types - Status Types" registry:

~~~
  Value    Notify Messages - Status Types
-----------------------------------------
  TBA1    HCP_SUPPORTED
  TBA2    HCP_UNSUPPORTED
~~~

This specification requests the IANA to create an IKEv2 Header Compression registry (see {{tab:hcp-name}}), as well as the necessary registries for the ESP Header Compression Profile Diet-ESP ( see ). 

All registries are "Specification Required".  
 
## Registry for IKEv2 Header Compression Profile {#tab:hcp-name}


| Value (1 Byte) | Designation | Reference | 
|----------------|-------------|-----------|
|  0             | Diet-ESP    | ThisRFC   |
|  1-255         | unallocated |  -        |

<!-- {: #tab-hcp-name artwork-align="center" title="Registry for IKEv2 Header Compression Profile"} -->


## Registries associated to the ESP Header Compression Profile Diet-ESP

## Registry for Diet-ESP Attributes for Rules Generation {#tab-afrg} 

Registry for Attributes for Rules Generation for the ESP Header Compression Profile Diet-ESP. When Associated Data is set to YES, the AF bit of the corresponding Transform Attribute Payload is set to 1 and 0 otherwise.

|  AfRG Code Point | Full Name      |  Designation     | Has Associated Data | Reference
|------------------|----------------|------------------|---------------------|----------
|  0               | DSCP CDA       | dscp_cda         | YES                 | ThisRFC
|  1               | ECN CDA        | ecn_cda          | YES                 | ThisRFC
|  2               | Flow Label CDA | flow_label_cda   | YES                 | ThisRFC
|  3               | Alignment      | alignment        | YES                 | ThisRFC 
|  4               | SPI LSB        | esp_spi_lsb      | YES                 | ThisRFC 
|  5               | SN  LSB        | esp_spi_sn       | YES                 | ThisRFC 
|  6 - 2^16-1      | unallocated    |     -            |        -            |    -    

<!-- {: #tab-afrg artwork-align="center" title="Registry for Attributes for Rules Generation for the ESP Header Compression Profile Diet-ESP. When Associated Data is set to YES, the AF bit of the corresponding Transform Attribute Payload is set to 1 and 0 otherwise."} -->


## Registries for the Values of Diet-ESP Attributes for Rules Generation  


### DSCP CDA Value Registry  {#tab:dscp_cda}

Value      | Designation | Reference | 
-----------|-------------|-----------|
  0        | uncompress  | ThisRFC   |
  1        | lower       | ThisRFC   |
  2        | sa          | ThisRFC   | 
  3-255    | unallocated |    -      |

<!-- {: #tab:dscp_cda artwork-align="center" title="DSCP CDA Value Registry"} -->

### ECDN CDA Value Registry {#tab:ecn_cda}

Value      | Designation | Reference | 
-----------|-------------|-----------|
  0        | uncompress  | ThisRFC   |
  1        | lower       | ThisRFC   |
  2-255    | unallocated |    -      |

<!-- {: #tab:ecn_cda artwork-align="center" title="ECDN CDA Value Registry"} -->

### Flow Label CDA Value Registry {#tab:fl_cda}
 
Value      | Designation | Reference | 
-----------|-------------|-----------|
  0        | uncompress  | ThisRFC   |
  1        | lower       | ThisRFC   |
  2        | generated   | ThiesRFC  |
  3        | zero        | ThisRFC   |
  4-255    | unallocated |    -      |

<!-- {: #tab:fl_cda artwork-align="center" title="Flow Label CDA Value Registry"} -->

### OS or Network Byte Alignment {#tab:align}

Value      | Designation | Reference | 
-----------|-------------|-----------|
  0        | 8 bit       | ThisRFC   |
  1        | 16 bit      | ThisRFC   |
  2        | 32 bit      | ThiesRFC  |
  3        | 64 bit      | ThisRFC   |
  4-255    | unallocated |    -      |

<!-- {: #tab:align artwork-align="center" title="OS or Network Byte Alignment"} -->

  
# Security Considerations

The protocol defined in this document does not modify IKEv2. 





