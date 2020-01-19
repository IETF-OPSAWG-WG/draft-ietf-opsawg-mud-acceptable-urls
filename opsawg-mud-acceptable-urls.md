---
title: Authorized update to MUD URLs
abbrev: mud-acceptable-urls
docname: draft-richardson-opsawg-mud-acceptable-urls-00

# stand_alone: true

ipr: trust200902
area: Operations
wg: OPSAWG Working Group
kw: Internet-Draft
cat: bcp

pi:    # can use array (if all yes) or hash here
  toc: yes
  sortrefs:   # defaults to yes
  symrefs: yes

author:


- ins: M. Richardson
  name: Michael Richardson
  org: Sandelman Software Works
  email: mcr+ietf@sandelman.ca

normative:
  RFC8520:
  RFC8499:

informative:
  I-D.richardson-opsawg-securehomegateway-mud: securehomegateway-mud
  boycrieswolf:
    title: "The Boy Who Cried Wolf"
    target: "https://fablesofaesop.com/the-boy-who-cried-wolf.html"
    date: 2020-01-18
  boywolfinfosec:
    title: "Security Alerts - A Case of the Boy Who Cried Wolf?"
    target: "https://www.infosecurity-magazine.com/opinions/security-alerts-boy-cried-wolf/"
    date: 2020-01-18
  falsemalware:
    title: "False malware alerts cost organizations $1.27M annually, report says"
    target: "https://www.scmagazine.com/home/security-news/false-malware-alerts-cost-organizations-1-27m-annually-report-says/ and http://go.cyphort.com/Ponemon-Report-Page.html"
    date: 2020-01-18

--- abstract

This document provides a way for an RFC8520 Manufacturer Usage Description
(MUD) definitions to declare what are acceptable replacement URLs for a device.

--- middle

# Introduction

{{RFC8520}} provides a standardized way to describe how a specific purpose
device makes use of Internet resources.  Access Control Lists (ACLs) can be
defined in an RFC8520 Manufacturer Usage Description (MUD) file that permit
a device to access Internet resources by DNS name.

MUD URLs can come from a number of sources:

* IDevID Extensions
* DHCP
* LLDP

* {{-securehomegateway-mud}} proposes to scan them from QRcodes.

The IDevID mechanism provides a URL that is asserted cryptographically by a manufacturer.
However, it is difficult for manufacturers to update the IDevID of a device which is already in a box.

The DHCP and LLDP mechanisms are not signed, but are asserted by the device.
A firmware update may update what MUD URL is emitted.
Sufficiently well targetted malware could also change the MUD URL.

The QRcode mechanism is usually done via paper/stickers, and is typically not under the control of the device itself at all.

While MUD files may include signatures, it is not mandatory to check them, and there is not a clear way to connect the entity which signed the MUD file to the device itself.
A malicious device does not need to make up a MUD file if there is already an available, and already trusted MUD file which it can use to impersonate the device.

One defense against this is to not trust MUD URLs which are different from the one that was placed in an IDevID.
Or if the initial MUD URL was not taken from an IDevID, it could be trusted on first use.
But, if the MUD controller has locked down the URL, then updates to the URL
are difficult to do.

# Updating MUD URLs vs Updating MUD files

## Updating the MUD file in place

One option is for the manufacturer to never change the MUD URL due to firmware updates.
The published description is updated whenever the behaviour of the firmware changes.

### Adding capabilities

For situations where new capabilities are added to the firmware, the MUD file will detail the new access that the new firmware requires.
This may involve new incoming or outgoing connections that should be authorized.
Devices which have been upgraded to the new firmware will make use of the new features.
Devices which have not been upgraded to the new firmware may have new connections that are authorized, but which the device does not use (outgoing connections), or for which the device is not prepared to respond to (new incoming connections).

It is possible that older versions of the firmware have vulnerabilities which were not easily exploitable due to the MUD file preventing particular kinds of access.
As an example, an older firmware could have a no credentials required (or default credentials) access via telnet on port 23 or HTTP on port 80.
The MUD file protected the device such that it could either not be accessed at all, or access was restricted to connections from a controller only.

Useful and needed upgrades to the firmware could add credentials to that service, permitting it to be opened up for more general access.
The new MUD file would provide for such access, but when combined with the weak security of the old firmware, results in a compromised device.

While there is an argument that old firmware was insecure and should be replaced, it is often the case that the upgrade process involves downtown, or can introduce risks due to needed evaluations not having been completed yet.
As an example: moving vehicles (cars, airplanes, etc.) should not perform upgrades while in motion.
It is probably undesireable to perform any upgrade to an airplane outside of its service facility.
The owner of a vehicle may desire to only perform software upgrades when they are at home, and could make other arrangements for transporation, rather than when parked at a remote cabin.
The situation for medical devices is even more complex.

### Removing capabilities

For situations where existing capabilities prove to be a problem and are to be turned off or removed in subsequent versions of the firmware, the MUD file will be updated to disallow connections that previously were allowed.

In this case, the new MUD file will forbid some connection which the old firmware still expects to do.
As explained in the previous section, upgrades may not always occur immediately upon release of the new firmware.

In this case the old device will be performing unwanted connections, and the MUD controller when be alerting the device owner that the device is mis-behaving.
This causes a {{boycrieswolf}} situation, leading to real security issues being ignored.
This is a serious issue as documented also in {{boywolfinfosec}}, and {{falsemalware}}.


# Privacy Considerations

TBD

# Security Considerations

TBD

--- back

# Appendices

