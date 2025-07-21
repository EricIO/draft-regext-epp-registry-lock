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

A registry lock secures a domain from any unauthorized changes to it on the registry level
as opposed to the registrar level.

This document describes an EPP extension to the domain object mapping ({{!RFC5731}}) that allows a
sponsoring client to manage registry locked domains automatically.

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

## Registry Lock

Many registries provide registrants for a way to add protection to their
domains by locking them at the registry level.  By locking a domain a
registrant ensures that any changes (except for renewing the domain) are
verified by the registry before going into effect.

In this document, we define an EPP extension that enables registries and
registrars to automate the registry lock process.

### Registry Lock Contact

A registry lock contact is a contact connected with a domain object that
is able to authorize changes to the object.

### Status Values for Locked Domains

Once a registry lock has been applied on a domain object the object
MUST have the serverDeleteProhibited status value set on it.

### Handling changes to a locked domain

If a transform command is processed successfully, a server MUST respond
with a result code of 1001 as defined in section 3 of {{!RFC5730}}, to
indicate that the transform is awaiting authorization to be completed.
If a transform command is authorized within the specified deadline a
server MUST send a poll message indicating the successfull transform.

If a transform is not authorized within the deadline a server MUST send
a poll message indicating the failure of the transform.

## Extension Elements

This extension adds additional elements to the EPP domain mapping.

### The <regLock:policy> element

The <regLock:policy> element contains the following elements:

- One OPTIONAL <regLock:timeout> element that contains the timeout for which
  after any <domain:update> operation will fail.
- One OPTIONAL <regLock:quorom> element that contains a positive non-zero integer
  that sets the number of registy lock contacts that MUST authorize the domain
  changes for it to be approved.

### The <regLock:contact> element

The <regLock:contact> element contains the following elements:

- <regLock:contactID> the contact identifier
- <regLock:method> the method used by the registry lock contact to authorize changes

## EPP Command Mapping

### EPP Query Commands

#### EPP <check> Command

This extension does not add any elements to the EPP <check> command
or <check> response described in the EPP domain mapping {{!RFC5731}}.

#### EPP <info> Command

This extension does not add any elements to the EPP <info> command
described in the EPP domain mapping {{!RFC5731}}.  However, additional
elements are defined for the <info> response.

An example <info> response for a domain object with no pending updates:

S:<?xml version="1.0" encoding="UTF-8" standalone="no"?>
S:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0"
S:     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
S:  <response>
S:    <result code="1000">
S:      <msg>Command completed successfully</msg>
S:    </result>
S:    <resData>
S:      <domain:infData
S:       xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
S:        <domain:name>example.com</domain:name>
S:        <domain:roid>EXAMPLE1-REP</domain:roid>
S:        <domain:status s="ok"/>
S:        <domain:registrant>jd1234</domain:registrant>
S:        <domain:contact type="admin">sh8013</domain:contact>
S:        <domain:contact type="tech">sh8013</domain:contact>
S:        <domain:ns>
S:          <domain:hostObj>ns1.example.com</domain:hostObj>
S:          <domain:hostObj>ns2.example.com</domain:hostObj>
S:        </domain:ns>
S:        <domain:host>ns1.example.com</domain:host>
S:        <domain:host>ns2.example.com</domain:host>
S:        <domain:clID>ClientX</domain:clID>
S:        <domain:crID>ClientY</domain:crID>
S:        <domain:crDate>1999-04-03T22:00:00.0Z</domain:crDate>
S:        <domain:upID>ClientX</domain:upID>
S:        <domain:upDate>1999-12-03T09:00:00.0Z</domain:upDate>
S:        <domain:exDate>2005-04-03T22:00:00.0Z</domain:exDate>
S:        <domain:trDate>2000-04-08T09:00:00.0Z</domain:trDate>
S:        <domain:authInfo>
S:          <domain:pw>2fooBAR</domain:pw>
S:        </domain:authInfo>
S:      </domain:infData>
S:    </resData>
S:    <extension>
S:      <regLock:infData
S:       xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
S:        <regLock:policyData>
S:          <regLock:timeout>1h</regLock:timeout>
S:          <regLock:quorom>2</reglock:quorom>
S:        </regLock:policyData>
S:        <regLock:contactData>
S:          <regLock:contact>
S:            <regLock:id>rl1001</regLock:id>
S:            <regLock:method>email</regLock:method>
S:          </regLock:contact>
S:          <regLock:contact>
S:            <regLock:id>rl1002</regLock:id>
S:            <regLock:method>email</regLock:method>
S:          </regLock:contact>
S:        </regLock:contactData>
S:      </secDNS:infData>
S:    </extension>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54322-XYZ</svTRID>
S:    </trID>
S:  </response>
S:</epp>

An example <info> response for a domain with pending updates:

S:<?xml version="1.0" encoding="UTF-8" standalone="no"?>
S:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0"
S:     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
S:  <response>
S:    <result code="1000">
S:      <msg>Command completed successfully</msg>
S:    </result>
S:    <resData>
S:      <domain:infData
S:       xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
S:        <domain:name>example.com</domain:name>
S:        <domain:roid>EXAMPLE1-REP</domain:roid>
S:        <domain:status s="ok"/>
S:        <domain:registrant>jd1234</domain:registrant>
S:        <domain:contact type="admin">sh8013</domain:contact>
S:        <domain:contact type="tech">sh8013</domain:contact>
S:        <domain:ns>
S:          <domain:hostObj>ns1.example.com</domain:hostObj>
S:          <domain:hostObj>ns2.example.com</domain:hostObj>
S:        </domain:ns>
S:        <domain:host>ns1.example.com</domain:host>
S:        <domain:host>ns2.example.com</domain:host>
S:        <domain:clID>ClientX</domain:clID>
S:        <domain:crID>ClientY</domain:crID>
S:        <domain:crDate>1999-04-03T22:00:00.0Z</domain:crDate>
S:        <domain:upID>ClientX</domain:upID>
S:        <domain:upDate>1999-12-03T09:00:00.0Z</domain:upDate>
S:        <domain:exDate>2005-04-03T22:00:00.0Z</domain:exDate>
S:        <domain:trDate>2000-04-08T09:00:00.0Z</domain:trDate>
S:        <domain:authInfo>
S:          <domain:pw>2fooBAR</domain:pw>
S:        </domain:authInfo>
S:      </domain:infData>
S:    </resData>
S:    <extension>
S:      <regLock:infData
S:       xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
S:        <regLock:policyData>
S:          <regLock:timeout>1h</regLock:timeout>
S:          <regLock:quorom>2</reglock:quorom>
S:        </regLock:policyData>
S:        <regLock:contactData>
S:          <regLock:contact>
S:            <regLock:id>rl1001</regLock:id>
S:            <regLock:method>email</regLock:method>
S:          </regLock:contact>
S:          <regLock:contact>
S:            <regLock:id>rl1002</regLock:id>
S:            <regLock:method>email</regLock:method>
S:          </regLock:contact>
S:        </regLock:contactData>
S:        <regLock:updateData>
S:           <regLock:update>
S:             <regLock:trID>foo</regLock:trID>
S:             <regLock:contactID approved=1>rl1001</regLock:contactID>
S:             <regLock:contactID approved=0>rl1002</regLock:contactID>
S:           </regLock:update>
S:        </regLock:updateData>
S:      </secDNS:infData>
S:    </extension>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54322-XYZ</svTRID>
S:    </trID>
S:  </response>
S:</epp>

### EPP <transfer> Command

This extension does not add any elements to the EPP <transfer>
command described in the EPP domain mapping {{!RFC5731}}.  However,
additional elements are defined for the <transfer> response.


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

C:<?xml version="1.0" encoding="UTF-8" standalone="no"?>
C:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
C:  <command>
C:    <create>
C:      <domain:create
C:       xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
C:        <domain:name>allocation.example</domain:name>
C:        <domain:registrant>jd1234</domain:registrant>
C:        <domain:contact type="admin">sh8013</domain:contact>
C:        <domain:contact type="tech">sh8013</domain:contact>
C:        <domain:authInfo>
C:          <domain:pw>2fooBAR</domain:pw>
C:        </domain:authInfo>
C:      </domain:create>
C:    </create>
C:    <extension>
C:      <regLock:create xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
C:        <regLock:policy>
C:          <regLock:timeout>1h</regLock:timeout>
C:        </regLock:policy>
C:        <regLock:contacts>
C:          <regLock:contact>
C:            <regLock:id>rc101</regLock:id>
C:            <regLock:method>email</regLock:method>
C:          </regLock:contact>
C:        </regLock:contacts>
C:      </regLock:create>
C:    </extension>
C:    <clTRID>ABC-12345</clTRID>
C:  </command>
C:</epp>

#### EPP <delete> Command

This extension does not define any additional elements to the
EPP <delete> command or <delete> responses described in the EPP
domain mapping {{!RFC5731}}.  However if a server recieves a
<delete> coomand for a domain object with registry lock set it
MUST be rejected.

#### EPP <renew> Command

This extension does not define any additional elements to the
EPP <renew> command or <renew> responses described in the EPP
domain mapping {{!RFC5731}}.

#### EPP <transfer> Command

This extension does not define any additional elements to the
EPP <transfer> command or <transfer> responses described in
the EPP domain mapping {{!RFC5731}}.

#### EPP <update> Command

This extension defines additional elements for the EPP <update>
command described in the EPP domain mapping {{!RFC5731}}.  No additional
elements are defined for the EPP <update> response.

C:<?xml version="1.0" encoding="UTF-8" standalone="no"?>
C:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0"
C:     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
C:  <command>
C:    <update>
C:      <domain:update
C:       xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
C:        <domain:name>example.com</domain:name>
C:      </domain:update>
C:    </update>
C:    <extension>
C:      <secDNS:update
C:       xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
C:        <regLock:chg>
C:          <regLock:policyData>
C:            <regLock:timeout>3h</regLock:timeout>
C:          </regLock:policyData>
C:        </regLock:chg>
C:      </regLock:update>
C:    </extension>
C:    <clTRID>ABC-12345</clTRID>
C:  </command>
C:</epp>

C:<?xml version="1.0" encoding="UTF-8" standalone="no"?>
C:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0"
C:     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
C:  <command>
C:    <update>
C:      <domain:update
C:       xmlns:domain="urn:ietf:params:xml:ns:domain-1.0">
C:        <domain:name>example.com</domain:name>
C:      </domain:update>
C:    </update>
C:    <extension>
C:      <secDNS:update
C:       xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
C:        <regLock:add>
C:          <regLock:contact>
C:            <regLock:id>rl1003</regLock:id>
C:            <regLock:method>email</regLock:method>
C:          </regLock:contact>
C:          <regLock:contact>
C:            <regLock:id>rl1004</regLock:id>
C:            <regLock:method>email</regLock:method>
C:          </regLock:contact>
C:        </regLock:add>
C:        <regLock:rem>
C:          <regLock:contact>
C:            <regLock:id>rl1001</regLock:id>
C:          </regLock:contact>
C:        </regLock:rem>
C:      </regLock:update>
C:    </extension>
C:    <clTRID>ABC-12345</clTRID>
C:  </command>
C:</epp>

## Formal Syntax
