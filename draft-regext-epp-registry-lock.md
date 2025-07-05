---
title: Registry Lock Extension for the Extensible Provisioning Protocol (EPP)
abbrev: EPP Registry Lock
docname: draft-ietf-regext-registry-lock-latest
date: {DATE}

normative:

informative:

--- abstract

This document describes an Extensible Provisioning Protocol (EPP) extension
for setting and managing a registry lock on a domain object.

TO BE REMOVED: This document is being collaborated on in Github at:
[https://github.com/EricIO/draft-regext-epp-registry-lock](https://github.com/EricIO/draft-regext-epp-registry-lock).
The most recent working version of the document, open issues, etc. should all be
available there.  The authors (gratefully) accept pull requests.

--- middle

# Introduction

Setting a registry lock on a domain secures it from unauthorized changes to a domain.

This document describes an EPP extension to the domain object mapping ({{!RFC5731}}) that allows a
sponsoring client to manage registry locked domains.

## Conventions Used in  This Document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all
capitals, as shown here.

In this document's examples, "C:" represents lines sent by a protocol
client and "S:" represents lines returned by a protocol server.
Indentation and white space in these examples are provided only to
illustrate element relationships and are not required features of
this protocol.

## Extension Elements

This extension adds additional elements to the EPP domain mapping.

### The <regLock:policy> element

The OPTIONAL <regLock:policy> contains the following elements:

- One OPTIONAL <regLock:timeout> element that contains the timeout for which
  after any <domain:update> operation will fail.
- One OPTIONAL <regLock:quorom> element that contains a positive non-zero integer
  that sets the number of registy lock contacts that MUST authorize the domain
  changes for it to be approved.

### The <regLock:create> element

### The <regLock:update> element

### The <regLock:info> element

## EPP Command Mapping

### EPP Query Commands

#### EPP <check> Command

This extension does not add any elements to the EPP <check> command
or <check> response described in the EPP domain mapping {{!RFC5731}}.

#### EPP <info> Command

This extension does not add any elements to the EPP <info> command
described in the EPP domain mapping {{!RFC5731}}. However, additional
elements are defined for the <info> response.

### EPP <transfer> Command

This extension does not add any elements to the EPP <transfer>
command or <transfer> response described in the EPP domain mapping
{{!RFC5731}}.


### EPP Transform Commands

EPP provides five commands to transform objects: <create> to create
an instance of an object, <delete> to delete an instance of an
object, <renew> to extend the validity period of an object,
<transfer> to manage object sponsorship changes, and <update> to
change information associated with an object.

#### EPP <create> Command

This extension defines additional elements for the EPP <create>
command described in the EPP domain mapping {{!RFC5731}}.  No additional
elements are defined for the EPP <create> response.

#### EPP <delete> Command

This extension does not define any additional elements to the
EPP <delete> command or <delete> responses described in the EPP
domain mapping {{!RFC5731}}.

#### EPP <renew> Command

This extension does not define any additional elements to the
EPP <renew> command or <renew> responses described in the EPP
domain mapping {{!RFC5731}}.

#### EPP <transfer> Command

This extension does not define any additional elements to the
EPP <transfer> command or <transfer> responses described in
the EPP domain mapping {{!RFC5731}}.

#### EPP <update> Command

This extension defines additional elements for the EPP <create>
command described in the EPP domain mapping {{!RFC5731}}.  No additional
elements are defined for the EPP <create> response.

## Formal Syntax
