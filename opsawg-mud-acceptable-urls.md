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
  I-D.reddy-opsawg-mud-tls:
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
This is a serious issue as documented also in {{boywolfinfosec}}, and
{{falsemalware}}.

### Significant changes to protocols

{{I-D.reddy-opsawg-mud-tls}} suggests MUD definitions to allow examination of TLS protocol details.
Such a profile may be very specific to the TLS library which is shipped.
Changes to the library (including bug fixes) may cause significant changes to
the profile, requiring changes to the profile described in the MUD file.
Such changes are likely neither forward nor backward compatible with other
versions, and in place updates to MUD files is not indicated.

## Motivation for updating MUD URLs

While many small tweaks to a MUD file can be done in place, the situation
described above, particularly when it comes to removing capabilities will
require updates to the MUD URL.
A strategy is do this securely is needed, and the rest of this document
provides a mechanism to do this securely.

# Threat model for MUD URLs

Only the DHCP and LLDP MUD URL mechanisms are sufficiently close to the firmware
version that they can be easily updated when the firmware is updated.
Because of that sensitivity, they are also easily changed by malware.

There are mitigating mechanisms which may be enough.
The MUD files are signed by the manufacturer.  
{{RFC8520}} has not established a trust model for MUD controllers to
determine whether a signature from a specific entity is legitimate as a
signature for a particular device.
{{RFC8520}} leaves this to the industry to work out.

## leveraging the manufacturer signature

Many MUD controllers currently use a Trust on First Use mechanism where the
first time a signature from a device is verified, the signatory is recorded.
Subsequent updates to that MUD file MUST be signed by the same entity to be
accepted.

Based upon this process, an update to the MUD URL would be valid if the new
MUD file was signed by the same entity that signed the previous entry.
This mechanism permits a replacement URL to be any URL that the same
manufacturer can provide.

## Concerns about same-signer mechanism

There is still a potential threat: a manufacturer which has many products may
have a MUD definition for another product that has the privileges that the
malware desires.

# Proposed possible Extension

The first MUD file which is defined for a device can come from an IDevID, or
via Trust on First Use with DHCP.  
This initial, initially trusted, MUD file will be called the "root" file.

MUD files start with a self-referential MUD-URL attribute.

## Semantic changes only

Proposal 1 involves no changes to the definition.
It is instead a semantic change: updates to the MUD URL for a device must
come from a URL that is identical to the mud-URL in the root file, but may
have a different basename component.

That is, if the mud-url is http://example.com/hello/there/file.json then any
URL that starts with http://example.com/hello/there/ would be acceptable.
Note: the new URLs are not relative.

## Add an extension

Proposal 2 involves an extension to the MUD definition.
A new extension, _mud-base-url_ would be created that would contain the value
http://example.com/hello/there/ rather than being implied from the mud-url.

# Privacy Considerations

TBD

# Security Considerations

TBD

--- back

# Appendices

