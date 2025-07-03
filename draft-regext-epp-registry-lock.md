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

## Conventions Used in  This Document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all
capitals, as shown here.

In this document's examples, "C:" represents lines sent by a protocol
client and "S:" represents lines returned by a protocol server.
Indentation and white space in these examples are provided only to
illustrate element relationships and are not required features of
this protocol.

## Extension Elements

This extension adds additional elements to the EPP domain mapping.
