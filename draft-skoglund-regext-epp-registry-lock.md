---
title: Registry Lock Extension for the Extensible Provisioning Protocol (EPP)
abbrev: EPP Registry Lock
docname: draft-ietf-regext-registry-lock-latest
cat: std
ipr: trust200902
date: {DATE}

author:
-
    ins: E. Skoglund
    name: Eric Skoglund
    organization: The Swedish Internet Foundation
    email: eric.skoglund@internetstiftelsen.se
-
    ins: S. Kämpf
    name: Sascha Kämpf
    organization: DENIC
    email: kaempf@denic.de

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
verified by the registry before going into effect.  As of yet there have
been no standardized process for for a registry lock and registries that
provide it have come up with different processes, most including some
level of manual intervention.

While a domain that has registry lock enabled on it needs authorization
for any changes made to it a registry MAY allow changes bypassing the
authorization via automated DNSSEC provisioning, for example using
a CDSS/CSYNC scanner.

The removal of a registry lock is a manual, operational procedure and
is not coveredd in this specification.

In this document, we define an EPP extension that enables registries and
registrars to further automate the registry lock process.

### Registry Lock Contact

A registry lock contact is a contact connected to a domain object that
is able to authorize changes to the object. A domain MAY have multiple
registry lock contacts, in which case changes MAY need to be authorized
by multiple registry lock contacts. A server MAY restrict the maximum
number of registry lock contacts connected to a domain.

### Status Values for Locked Domains

Once a registry lock has been applied on a domain object the object
MUST have the serverDeleteProhibited status value set on it. Additionally
a new status value of domainLocked MUST be set on the domain object.

### Adding a registry lock to a domain

When a registry lock is set on a domain a server MUST respond with a result
code oc 1001 as defined in section 3 of {{!RFC5730}}. A server must then
await for authorization as set out by the registry lock contact parameters
provided. If the command is authorized within the specified deadline a server
MUST send a poll message indicating the successfull transform.

Example poll message indicating success:

~~~~~~~~~~
S:<?xml version="1.0" encoding="UTF-8"?>
S:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
S:   <response>
S:      <result code="1301">
S:         <msg lang="en-US">
S:             Command completed successfully; ack to dequeue
S:         </msg>
S:      </result>
S:      <msgQ id="201" count="1">
S:         <qDate>2013-10-22T14:25:57.0Z</qDate>
S:         <msg>Setting registry lock on domain succeeded.</msg>
S:      </msgQ>
S:    <resData>
S:      <regLock:infData xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
S:        <regLock:domain>example.com</regLock:domain>
S:        <regLock:operation success="true">update</regLock:operation>
S:        <regLock:svTRID>12345-XYZ</regLock:svTRID>
S:        <regLock:approvedBy>
S:          <regLock:contact>
S:             <regLock:id>rl01</regLock:id>
S:          </regLock:contact>
S:        </regLock:approvedBy>
S:      </regLock:pollInfo>
S:    </resData>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54321-XYZ</svTRID>
S:    </trID>
S:   </response>
S:</epp>
~~~~~~~~~~

If a transform is not authorized within the deadline a server MUST send
a poll message indicating the failure of the transform.

Example poll message indicating failure:

~~~~~~~~~~
S:<?xml version="1.0" encoding="UTF-8"?>
S:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
S:   <response>
S:      <result code="1301">
S:         <msg lang="en-US">
S:             Command completed successfully; ack to dequeue
S:         </msg>
S:      </result>
S:      <msgQ id="201" count="1">
S:         <qDate>2013-10-22T14:25:57.0Z</qDate>
S:         <msg>Update of locked domain failed.</msg>
S:      </msgQ>
S:    <resData>
S:      <regLock:infData xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
S:        <regLock:domain>example.com</regLock:domain>
S:        <regLock:operation success="false">update</regLock:operation>
S:        <regLock:svTRID>12345-XYZ</regLock:svTRID>
S:        </regLock:approvedBy>
S:      </regLock:pollInfo>
S:    </resData>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54321-XYZ</svTRID>
S:    </trID>
S:   </response>
S:</epp>
~~~~~~~~~~

### Handling changes to an associated contact object

If an update command for a contact object associated with a locked domain
is processed successfully, a server MUST respond with a result code of 1001
as defined in section 3  of {{!RFC5730}}, to indicate that the transform is
awaiting authorization to be completed. If a transform command is authorized
within the specified deadline a server MUST send a poll message indicating
the successfull transform.

Example poll message indicating success:

~~~~~~~~~~
S:<?xml version="1.0" encoding="UTF-8"?>
S:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
S:   <response>
S:      <result code="1301">
S:         <msg lang="en-US">
S:             Command completed successfully; ack to dequeue
S:         </msg>
S:      </result>
S:      <msgQ id="201" count="1">
S:         <qDate>2013-10-22T14:25:57.0Z</qDate>
S:         <msg>Update of contact succeeded.</msg>
S:      </msgQ>
S:    <resData>
S:      <regLock:infData xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
S:        <regLock:domain>example.com</regLock:domain>
S:        <regLock:operation success="true">update</regLock:operation>
S:        <regLock:svTRID>12345-XYZ</regLock:svTRID>
S:        <regLock:approvedBy>
S:          <regLock:contact>
S:             <regLock:id>rl01</regLock:id>
S:          </regLock:contact>
S:        </regLock:approvedBy>
S:      </regLock:pollInfo>
S:    </resData>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54321-XYZ</svTRID>
S:    </trID>
S:   </response>
S:</epp>
~~~~~~~~~~

If setting the lock is not authorized within the deadline a server MUST send
a poll message indicating the failure of the transform.

Example poll message indicating failure:

~~~~~~~~~~
S:<?xml version="1.0" encoding="UTF-8"?>
S:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
S:   <response>
S:      <result code="1301">
S:         <msg lang="en-US">
S:             Command completed successfully; ack to dequeue
S:         </msg>
S:      </result>
S:      <msgQ id="201" count="1">
S:         <qDate>2013-10-22T14:25:57.0Z</qDate>
S:         <msg>Setting registry lock failed.</msg>
S:      </msgQ>
S:    <resData>
S:      <regLock:infData xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
S:        <regLock:domain>example.com</regLock:domain>
S:        <regLock:operation success="false">update</regLock:operation>
S:        <regLock:svTRID>12345-XYZ</regLock:svTRID>
S:        </regLock:approvedBy>
S:      </regLock:pollInfo>
S:    </resData>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54321-XYZ</svTRID>
S:    </trID>
S:   </response>
S:</epp>
~~~~~~~~~~

### Handling changes to a locked domain

If an update command for a locked domain is processed successfully,
a server MUST respond with a result code of 1001 as defined in section 3
of {{!RFC5730}}, to indicate that the transform is awaiting authorization
to be completed. If a transform command is authorized within the specified
deadline a server MUST send a poll message indicating the successfull transform.

Example poll message indicating success:

~~~~~~~~~~
S:<?xml version="1.0" encoding="UTF-8"?>
S:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
S:   <response>
S:      <result code="1301">
S:         <msg lang="en-US">
S:             Command completed successfully; ack to dequeue
S:         </msg>
S:      </result>
S:      <msgQ id="201" count="1">
S:         <qDate>2013-10-22T14:25:57.0Z</qDate>
S:         <msg>Update of locked domain succeeded.</msg>
S:      </msgQ>
S:    <resData>
S:      <regLock:infData xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
S:        <regLock:domain>example.com</regLock:domain>
S:        <regLock:operation success="true">update</regLock:operation>
S:        <regLock:svTRID>12345-XYZ</regLock:svTRID>
S:        <regLock:approvedBy>
S:          <regLock:contact>
S:             <regLock:id>rl01</regLock:id>
S:          </regLock:contact>
S:        </regLock:approvedBy>
S:      </regLock:pollInfo>
S:    </resData>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54321-XYZ</svTRID>
S:    </trID>
S:   </response>
S:</epp>
~~~~~~~~~~

If a transform is not authorized within the deadline a server MUST send
a poll message indicating the failure of the transform.

Example poll message indicating failure:

~~~~~~~~~~
S:<?xml version="1.0" encoding="UTF-8"?>
S:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
S:   <response>
S:      <result code="1301">
S:         <msg lang="en-US">
S:             Command completed successfully; ack to dequeue
S:         </msg>
S:      </result>
S:      <msgQ id="201" count="1">
S:         <qDate>2013-10-22T14:25:57.0Z</qDate>
S:         <msg>Update of locked domain failed.</msg>
S:      </msgQ>
S:    <resData>
S:      <regLock:infData xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
S:        <regLock:domain>example.com</regLock:domain>
S:        <regLock:operation success="false">update</regLock:operation>
S:        <regLock:svTRID>12345-XYZ</regLock:svTRID>
S:        </regLock:approvedBy>
S:      </regLock:pollInfo>
S:    </resData>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54321-XYZ</svTRID>
S:    </trID>
S:   </response>
S:</epp>
~~~~~~~~~~

### Handling changes to an associated contact object

If an update command for a contact object associated with a locked domain
is processed successfully, a server MUST respond with a result code of 1001
as defined in section 3  of {{!RFC5730}}, to indicate that the transform is
awaiting authorization to be completed. If a transform command is authorized
within the specified deadline a server MUST send a poll message indicating
the successfull transform.

Example poll message indicating success:

~~~~~~~~~~
S:<?xml version="1.0" encoding="UTF-8"?>
S:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
S:   <response>
S:      <result code="1301">
S:         <msg lang="en-US">
S:             Command completed successfully; ack to dequeue
S:         </msg>
S:      </result>
S:      <msgQ id="201" count="1">
S:         <qDate>2013-10-22T14:25:57.0Z</qDate>
S:         <msg>Update of contact succeeded.</msg>
S:      </msgQ>
S:    <resData>
S:      <regLock:infData xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
S:        <regLock:domain>example.com</regLock:domain>
S:        <regLock:operation success="true">update</regLock:operation>
S:        <regLock:svTRID>12345-XYZ</regLock:svTRID>
S:        <regLock:approvedBy>
S:          <regLock:contact>
S:             <regLock:id>rl01</regLock:id>
S:          </regLock:contact>
S:        </regLock:approvedBy>
S:      </regLock:pollInfo>
S:    </resData>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54321-XYZ</svTRID>
S:    </trID>
S:   </response>
S:</epp>
~~~~~~~~~~

If a transform is not authorized within the deadline a server MUST send
a poll message indicating the failure of the transform.

Example poll message indicating failure:

~~~~~~~~~~
S:<?xml version="1.0" encoding="UTF-8"?>
S:<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
S:   <response>
S:      <result code="1301">
S:         <msg lang="en-US">
S:             Command completed successfully; ack to dequeue
S:         </msg>
S:      </result>
S:      <msgQ id="201" count="1">
S:         <qDate>2013-10-22T14:25:57.0Z</qDate>
S:         <msg>Update of contact failed.</msg>
S:      </msgQ>
S:    <resData>
S:      <regLock:infData xmlns:regLock="urn:ietf:params:xml:ns:regLock-1.0">
S:        <regLock:domain>example.com</regLock:domain>
S:        <regLock:operation success="false">update</regLock:operation>
S:        <regLock:svTRID>12345-XYZ</regLock:svTRID>
S:        </regLock:approvedBy>
S:      </regLock:pollInfo>
S:    </resData>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54321-XYZ</svTRID>
S:    </trID>
S:   </response>
S:</epp>
~~~~~~~~~~

## Extension Elements

This extension adds additional elements to the EPP domain mapping.

### The `<regLock:policy>` element

The `<regLock:policy>` element contains the following elements:

- One OPTIONAL `<regLock:timeout>` element that contains the timeout for which
  after any `<domain:update>` operation will fail. A server MAY restrict the allowed
  values.
- One OPTIONAL `<regLock:quorom>` element that contains a positive non-zero integer
  that sets the number of registy lock contacts that MUST authorize the domain
  changes for it to be approved. A server MAY restrict the number of registry lock
  contacts a domain object can have.

### The `<regLock:contact>` element

The `<regLock:contact>` element contains the following elements:

- `<regLock:contactID>` the contact identifier.
- An OPTIONAL `<regLock:method>` the method used by the registry lock contact to
  authorize changes. A server MAY restrict the allowed values.

#### Well known `<regLock:method>` values

A server MAY allow any value for the `<regLock:method>` element the following
values have a well known definition:

- email  - The contact will receive an email with further instructions.
- text   - The contact will receive a text message with further instructions.
- letter - The contact will receive a postal letter with further instructions.
- phone  - The contact will receive a phone call with further instructions.
- token  - The contact will be provided means to send a token to the server for
           authorizing changes.

## EPP Command Mapping

### EPP Query Commands

#### EPP `<check>` Command

This extension does not add any elements to the EPP `<check>` command
or `<check>` response described in the EPP domain mapping {{!RFC5731}}.

#### EPP `<info>` Command

This extension does not add any elements to the EPP `<info>` command
described in the EPP domain mapping {{!RFC5731}}.  However, additional
elements are defined for the `<info>` response.

An example `<info>` response for a domain object with no pending updates:

~~~~~~~~~~
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
S:      </regLock:infData>
S:    </extension>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54322-XYZ</svTRID>
S:    </trID>
S:  </response>
S:</epp>
~~~~~~~~~~

An example `<info>` response for a domain with pending updates:

~~~~~~~~~~
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
S:      </regLock:infData>
S:    </extension>
S:    <trID>
S:      <clTRID>ABC-12345</clTRID>
S:      <svTRID>54322-XYZ</svTRID>
S:    </trID>
S:  </response>
S:</epp>
~~~~~~~~~~

### EPP `<transfer>` Command

This extension does not add any elements to the EPP `<transfer>` command
or `<transfer>` responses as described in the EPP domain mapping {{!RFC5731}}.

### EPP Transform Commands

EPP provides five commands to transform objects: `<create>` to create
an instance of an object, `<delete>` to delete an instance of an
object, `<renew>` to extend the validity period of an object,
`<transfer>` to manage object sponsorship changes, and `<update>` to
change information associated with an object.

#### EPP `<create>` Command

This extension defines additional elements for the EPP `<create>`
command described in the EPP domain mapping {{!RFC5731}}.  No additional
elements are defined for the EPP `<create>` response.

~~~~~~~~~~
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
~~~~~~~~~~

#### EPP `<delete>` Command

##### Domain Objects

This extension does not define any additional elements to the
EPP `<delete>` command or `<delete>` responses described in the EPP
domain mapping {{!RFC5731}}.  However if a server recieves a
`<delete>` command for a domain object with registry lock set it
MUST be rejected.

##### Contact Objects

This extension does not define any additional elements to the
EPP `<delete>` command or `<delete>` responses described in the EPP
contact mapping {{!RFC5733}}.  However if a server recieves a
`<delete>` command for a contact object associated as a registry lock
contact with a domain object it MUST be rejected.

#### EPP `<renew>` Command

This extension does not define any additional elements to the
EPP `<renew>` command or `<renew>` responses described in the EPP
domain mapping {{!RFC5731}}.

#### EPP `<transfer>` Command

This extension does not define any additional elements to the
EPP `<transfer>` command or `<transfer>` responses described in
the EPP domain mapping {{!RFC5731}}.

#### EPP `<update>` Command

This extension defines additional elements for the EPP `<update>`
command described in the EPP domain mapping {{!RFC5731}}.  No additional
elements are defined for the EPP `<update>` response.

The EPP `<update>` command provides a transform operation that allows a
client to modify the attributes of a domain object.  In addition to
the EPP command elements described in the EPP domain mapping, the
command MUST contain an `<extension>` element, and the `<extension>`
element MUST contain a child `<regLock:update>` element that identifies
the extension namespace if the client wants to update the domain
object with data defined in this extension.  The `<regLock:update>`
element contains a `<regLock:add>` to add registry lock contacts to
a domain, a `<regLock:rem>` to remove registry lock contacts from a domain,
or a `<regLock:chg>` element to change policy information or a registry lock
contacts authorization method. At least one `<regLock:add>`, `<regLock:rem>`,
or `<regLock:chg>` element MUST be provided.

A `<regLock:add>` or `<regLock:rem>` MUST only contain `<regLock:contact>`
elements as child elements.

An example update command changing policy data:

~~~~~~~~~~
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
C:      <regLock:update
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
~~~~~~~~~~

An example update command adding and removing registry lock contacts:

~~~~~~~~~~
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
C:      <regLock:update
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
~~~~~~~~~~

## Formal Syntax

TBD
