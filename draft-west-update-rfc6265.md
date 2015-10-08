---
title: Cookies

abbrev: update-rfc6265
docname: draft-west-update-rfc6265-latest
date: 2015
category: std
updates: 6265

ipr: trust200902
area: General
workgroup: HTTPbis
keyword: Internet-Draft

pi: [toc, sortrefs, symrefs, strict, compact, comments, inline]

author:
-
  ins: M. West
  name: Mike West
  organization: Google, Inc
  email: mkwst@google.com
  uri: https://mikewest.org/

normative:
  RFC1034:
  RFC1123:
  RFC2119:
  RFC2616:
  RFC3490:
  RFC4790:
  RFC5234:
  RFC5890:
  USASCII:
    target: ANSI X3.4
    title: "Coded Character Set -- 7-bit American Standard Code for Information Interchange"
    date: 1986
    author:
      name: American National Standards Institute

informative:
  RFC2109:
  RFC2965:
  RFC2818:
  Netscape:
    target: http://web.archive.org/web/20020803110822/http://wp.netscape.com/newsref/std/cookie_spec.html
    title: "Persistent Client State -- HTTP Cookies"
    date: 1999
    author:
      name: Netscape Communications Corp.
  Kri2001:
    target: http://arxiv.org/abs/cs.SE/0105018
    title: "HTTP Cookies: Standards, Privacy, and Politics"
    seriesinfo:
      ACM: "ACM Transactions on Internet Technology Vol. 1, #2"
    date: November 2001
    author:
      name: David M. Kristol
      ins: D. Kristol
  RFC3629:
  RFC4648:
  RFC3864:
  RFC5895:
  UTS46:
    target: http://unicode.org/reports/tr46/
    title: "Unicode IDNA Compatibility Processing"
    seriesinfo:
      UNICODE: "Unicode Technical Standards # 46"
    date: 2010
    author:
    -
      ins: M. Davis
      name: Mark Davis
    -
      ins: M. Suignard
      name: Michel Suignard
  CSRF:
    target: http://portal.acm.org/citation.cfm?id=1455770.1455782
    title: Robust Defenses for Cross-Site Request Forgery
    date: 2008
    author:
    -
      ins: A. Barth
      name: Adam Barth
    -
      ins: C. Jackson
    -
      ins: J. Mitchell
  Aggarwal2010:
    author:
    -
      ins: G. Aggarwal
    -
      ins: E. Burzstein
    -
      ins: C. Jackson
    -
      ins: D. Boneh
    title: "An Analysis of Private Browsing Modes in Modern Browsers"
    date: 2010
    target: http://www.usenix.org/events/sec10/tech/full_papers/Aggarwal.pdf

--- abstract

This document defines the HTTP `Cookie` and `Set-Cookie` header fields. These
header fields can be used by HTTP servers to store state (called cookies) at
HTTP user agents, letting the servers maintain a stateful session over the
mostly stateless HTTP protocol.  Although cookies have many historical
infelicities that degrade their security and privacy, the Cookie and Set-Cookie
header fields are widely used on the Internet.  This document obsoletes RFC
2965.

--- middle

# Introduction

This document defines the HTTP `Cookie` and `Set-Cookie` header fields. Using
the `Set-Cookie` header field, an HTTP server can pass name/value pairs and
associated metadata (called cookies) to a user agent. When the user agent makes
subsequent requests to the server, the user agent uses the metadata and other
information to determine whether to return the name/value pairs in the `Cookie`
header.

Although simple on their surface, cookies have a number of complexities. For
example, the server indicates a scope for each cookie when sending it to the
user agent.  The scope indicates the maximum amount of time in which the user
agent should return the cookie, the servers to which the user agent should
return the cookie, and the URI schemes for which the cookie is applicable.

For historical reasons, cookies contain a number of security and privacy
infelicities.  For example, a server can indicate that a given cookie is
intended for "secure" connections, but the `Secure` attribute does not provide
integrity in the presence of an active network attacker.  Similarly, cookies
for a given host are shared across all the ports on that host, even though the
usual "same-origin policy" used by web browsers isolates content retrieved via
different ports.

There are two audiences for this specification: developers of cookie-generating
servers and developers of cookie-consuming user agents.

To maximize interoperability with user agents, servers SHOULD limit themselves
to the well-behaved profile defined in Section 4 when generating cookies.

User agents MUST implement the more liberal processing rules defined in Section
5, in order to maximize interoperability with existing servers that do not
conform to the well-behaved profile defined in Section 4.

This document specifies the syntax and semantics of these headers as they are
actually used on the Internet.  In particular, this document does not create
new syntax or semantics beyond those in use today. The recommendations for
cookie generation provided in Section 4 represent a preferred subset of current
server behavior, and even the more liberal cookie processing algorithm provided
in Section 5 does not recommend all of the syntactic and semantic variations in
use today.  Where some existing software differs from the recommended protocol
in significant ways, the document contains a note explaining the difference.

Prior to this document, there were at least three descriptions of cookies: the
so-called "Netscape cookie specification" {{Netscape}}, RFC 2109 {{RFC2109}},
and RFC 2965 {{RFC2965}}.  However, none of these documents describe how the
`Cookie` and `Set-Cookie` headers are actually used on the Internet (see
{{Kri2001}} for historical context).  In relation to previous IETF
specifications of HTTP state management mechanisms, this document requests the
following actions:

1.  Change the status of {{RFC2109}} to Historic (it has already been obsoleted
    by {{RFC2965}}).

2.  Change the status of {{RFC2965}} to Historic.

3.  Indicate that {{RFC2965}} has been obsoleted by this document.

In particular, in moving RFC 2965 to Historic and obsoleting it, this document
deprecates the use of the `Cookie2` and `Set-Cookie2` header fields.

# Conventions

## Conformance Criteria

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in {{RFC2119}}.

Requirements phrased in the imperative as part of algorithms (such as "strip any
leading space characters" or "return false and abort these steps") are to be
interpreted with the meaning of the key word ("MUST", "SHOULD", "MAY", etc.)
used in introducing the algorithm. 

Conformance requirements phrased as algorithms or specific steps can be
implemented in any manner, so long as the end result is equivalent. In
particular, the algorithms defined in this specification are intended to be
easy to understand and are not intended to be performant.

## Syntax Notation

This specification uses the Augmented Backus-Naur Form (ABNF) notation of
{{RFC5234}}.

The following core rules are included by reference, as defined in {{RFC5234}},
Appendix B.1: `ALPHA` (letters), `CR` (carriage return), `CRLF` (CR LF), `CTLs`
(controls), `DIGIT` (decimal 0-9), `DQUOTE` (double quote), `HEXDIG`
(hexadecimal 0-9/A-F/a-f), `LF` (line feed), `NUL` (null octet), `OCTET` (any
8-bit sequence of data except `NUL`), `SP` (space), `HTAB` (horizontal tab),
`CHAR` (any {{USASCII}} character), `VCHAR` (any visible {{USASCII}} character),
and `WSP` (whitespace).

The `OWS` (optional whitespace) rule is used where zero or more linear
whitespace characters MAY appear:

~~~ abnf
OWS            = *( [ obs-fold ] WSP )
                ; "optional" whitespace
obs-fold       = CRLF
~~~

`OWS` SHOULD either not be produced or be produced as a single `SP` character.

## Terminology

The terms "user agent", "client", "server", "proxy", and "origin server" have
the same meaning as in the HTTP/1.1 specification ({{RFC2616}}, Section 1.3).

The `request-host` is the name of the host, as known by the user agent, to which
the user agent is sending an HTTP request or from which it is receiving an HTTP
response (i.e., the name of the host to which it sent the corresponding HTTP
request).

The term `request-uri` is defined in Section 5.1.2 of {{RFC2616}}.

Two sequences of octets are said to case-insensitively match each other if and
only if they are equivalent under the `i;ascii-casemap` collation defined in
{{RFC4790}}.

The term string means a sequence of non-`NUL` octets.

# Overview

This section outlines a way for an origin server to send state information to a
user agent and for the user agent to return the state information to the origin
server.

To store state, the origin server includes a `Set-Cookie` header in an HTTP
response. In subsequent requests, the user agent returns a `Cookie` request
header to the origin server. The `Cookie` header contains cookies the user agent
received in previous `Set-Cookie` headers. The origin server is free to ignore
the `Cookie` header or use its contents for an application-defined purpose.

Origin servers MAY send a `Set-Cookie` response header with any response. User
agents MAY ignore `Set-Cookie` headers contained in responses with 100-level
status codes but MUST process `Set-Cookie` headers contained in other responses
(including responses with 400- and 500-level status codes).  An origin server
can include multiple `Set-Cookie` header fields in a single response. The
presence of a `Cookie` or a `Set-Cookie` header field does not preclude HTTP
caches from storing and reusing a response.

Origin servers SHOULD NOT fold multiple `Set-Cookie` header fields into a single
header field. The usual mechanism for folding HTTP headers fields (i.e., as
defined in {{RFC2616}}) might change the semantics of the `Set-Cookie` header
field because the %x2C (",") character is used by `Set-Cookie` in a way that
conflicts with such folding.

## Examples

Using the `Set-Cookie` header, a server can send the user agent a short string
in an HTTP response that the user agent will return in future HTTP requests that
are within the scope of the cookie. For example, the server can send the user
agent a "session identifier" named SIDwith the value `31d4d96e407aad42`. The
user agent then returns the session identifier in subsequent requests.

== Server -> User Agent ==

Set-Cookie: SID=31d4d96e407aad42

== User Agent -> Server ==

Cookie: SID=31d4d96e407aad42

The server can alter the default scope of the cookie using the `Path` and
`Domain` attributes. For example, the server can instruct the user agent to
return the cookie to every path and every subdomain of example.com.

== Server -> User Agent ==

Set-Cookie: SID=31d4d96e407aad42; Path=/; Domain=example.com

== User Agent -> Server ==

Cookie: SID=31d4d96e407aad42

As shown in the next example, the server can store multiple cookies at the user
agent. For example, the server can store a session identifier as well as the
user's preferred language by returning two `Set-Cookie` header fields. Notice
that the server uses the `Secure` and `HttpOnly` attributes to provide
additional security protections for the more sensitive session identifier (see
Section 4.1.2).

== Server -> User Agent ==

Set-Cookie: SID=31d4d96e407aad42; Path=/; Secure; HttpOnly
Set-Cookie: lang=en-US; Path=/; Domain=example.com

== User Agent -> Server ==

Cookie: SID=31d4d96e407aad42; lang=en-US

Notice that the `Cookie` header above contains two cookies, one named `SID` and
one named `lang`. If the server wishes the user agent to persist the cookie over
multiple "sessions" (e.g., user agent restarts), the server can specify an
expiration date in the `Expires` attribute.  Note that the user agent might
delete the cookie before the expiration date if the user agent's cookie store
exceeds its quota or if the user manually deletes the server's cookie.

== Server -> User Agent ==

Set-Cookie: lang=en-US; Expires=Wed, 09 Jun 2021 10:18:14 GMT

== User Agent -> Server ==

Cookie: SID=31d4d96e407aad42; lang=en-US

Finally, to remove a cookie, the server returns a `Set-Cookie` header with an
expiration date in the past.  The server will be successful in removing the
cookie only if the `Path` and the `Domain` attribute in the Set-Cookie header
match the values used when the cookie was created.

== Server -> User Agent ==

Set-Cookie: lang=; Expires=Sun, 06 Nov 1994 08:49:37 GMT

== User Agent -> Server ==

Cookie: SID=31d4d96e407aad42

# Server Requirements

This section describes the syntax and semantics of a well-behaved profile of the
`Cookie` and `Set-Cookie` headers.

## Set-Cookie

The `Set-Cookie` HTTP response header is used to send cookies from the server to
the user agent.

### Syntax

Informally, the `Set-Cookie` response header contains the header name
"Set-Cookie" followed by a ":" and a cookie.  Each cookie begins with a
`name-value-pair`, followed by zero or more `attribute-value` pairs. Servers
SHOULD NOT send `Set-Cookie` headers that fail to conform to the following
grammar:

~~~ abnf
set-cookie-header = "Set-Cookie:" SP set-cookie-string
set-cookie-string = cookie-pair *( ";" SP cookie-av )
cookie-pair       = cookie-name "=" cookie-value
cookie-name       = token
cookie-value      = *cookie-octet / ( DQUOTE *cookie-octet DQUOTE )
cookie-octet      = %x21 / %x23-2B / %x2D-3A / %x3C-5B / %x5D-7E
                      ; US-ASCII characters excluding CTLs,
                      ; whitespace DQUOTE, comma, semicolon,
                      ; and backslash
token             = <token, defined in [RFC2616], Section 2.2>

cookie-av         = expires-av / max-age-av / domain-av /
                   path-av / secure-av / httponly-av /
                   extension-av
expires-av        = "Expires=" sane-cookie-date
sane-cookie-date  = <rfc1123-date, defined in [RFC2616], Section 3.3.1>
max-age-av        = "Max-Age=" non-zero-digit *DIGIT
                      ; In practice, both expires-av and max-age-av
                      ; are limited to dates representable by the
                      ; user agent.
non-zero-digit    = %x31-39
                      ; digits 1 through 9
domain-av         = "Domain=" domain-value
domain-value      = <subdomain>
                      ; defined in {{RFC1034}}, Section 3.5, as
                      ; enhanced by {{RFC1123}}, Section 2.1
path-av           = "Path=" path-value
path-value        = <any CHAR except CTLs or ";">
secure-av         = "Secure"
httponly-av       = "HttpOnly"
extension-av      = <any CHAR except CTLs or ";">
~~~

Note that some of the grammatical terms above reference documents that use
different grammatical notations than this document (which uses ABNF from
{{RFC5234}}).

The semantics of the `cookie-value` are not defined by this document.

To maximize compatibility with user agents, servers that wish to store arbitrary
data in a cookie-value SHOULD encode that data, for example, using Base64
{{RFC4648}}.

The portions of the `set-cookie-string` produced by the `cookie-av` term` are
known as attributes. To maximize compatibility with user agents, servers SHOULD
NOT produce two attributes with the same name in the same `set-cookie-string`.
(See Section 5.3 for how user agents handle this case.)

Servers SHOULD NOT include more than one `Set-Cookie` header field in the same
response with the same `cookie-name`.  (See Section 5.2 for how user agents
handle this case.)

If a server sends multiple responses containing `Set-Cookie` headers
concurrently to the user agent (e.g., when communicating with the user agent
over multiple sockets), these responses create a "race condition" that can lead
to unpredictable behavior.

NOTE: Some existing user agents differ in their interpretation of two-digit
years.  To avoid compatibility issues, servers SHOULD use the `rfc1123-date`
format, which requires a four-digit year.

NOTE: Some user agents store and process dates in cookies as 32-bit UNIX time_t
values. Implementation bugs in the libraries supporting time_t processing on
some systems might cause such user agents to process dates after the year 2038
incorrectly.

### Semantics (Non-Normative)

This section describes simplified semantics of the `Set-Cookie` header. These
semantics are detailed enough to be useful for understanding the most common
uses of cookies by servers.  The full semantics are described in Section 5.

When the user agent receives a `Set-Cookie` header, the user agent stores the
cookie together with its attributes.  Subsequently, when the user agent makes
an HTTP request, the user agent includes the applicable, non-expired cookies
in the `Cookie` header.

If the user agent receives a new cookie with the same `cookie-name`,
`domain-value`, and `path-value` as a cookie that it has already stored, the
existing cookie is evicted and replaced with the new cookie. Notice that
servers can delete cookies by sending the user agent a new cookie with an
`Expires` attribute with a value in the past.

Unless the cookie's attributes indicate otherwise, the cookie is returned only
to the origin server (and not, for example, to any subdomains), and it expires
at the end of the current session (as defined by the user agent). User agents
ignore unrecognized cookie attributes (but not the entire cookie).

#### The Expires Attribute

The `Expires` attribute indicates the maximum lifetime of the cookie,
represented as the date and time at which the cookie expires. The user agent is
not required to retain the cookie until the specified date has passed. In fact,
user agents often evict cookies due to memory pressure or privacy concerns.

#### The Max-Age Attribute

The `Max-Age` attribute indicates the maximum lifetime of the cookie,
represented as the number of seconds until the cookie expires. The user agent is
not required to retain the cookie for the specified duration. In fact, user
agents often evict cookies due to memory pressure or privacy concerns.

NOTE: Some existing user agents do not support the `Max-Age` attribute. User
agents that do not support the `Max-Age` attribute ignore the attribute.

If a cookie has both the `Max-Age` and the `Expires` attribute, the `Max-Age`
attribute has precedence and controls the expiration date of the cookie. If a
cookie has neither the `Max-Age` nor the `Expires` attribute, the user agent
will retain the cookie until "the current session is over" (as defined by the
user agent).

#### The Domain Attribute

The `Domain` attribute specifies those hosts to which the cookie will be sent.
For example, if the value of the `Domain` attribute is "example.com", the user
agent will include the cookie in the Cookie header when making HTTP requests to
`example.com`, `www.example.com`, and `www.corp.example.com`.  (Note that a
leading %x2E ("."), if present, is ignored even though that character is not
permitted, but a trailing %x2E ("."), if present, will cause the user agent to
ignore the attribute.)  If the server omits the `Domain` attribute, the user
agent will return the cookie only to the origin server.

WARNING: Some existing user agents treat an absent `Domain` attribute as if the
`Domain` attribute were present and contained the current host name.  For
example, if example.com returns a `Set-Cookie` header without a `Domain`
attribute, these user agents will erroneously send the cookie to
`www.example.com` as well.

The user agent will reject cookies unless the `Domain` attribute specifies a
scope for the cookie that would include the origin server. For example, the
user agent will accept a cookie with a `Domain` attribute of "example.com" or
of "foo.example.com" from foo.example.com, but the user agent will not accept
a cookie with a `Domain` attribute of "bar.example.com" or of
"baz.foo.example.com".

NOTE: For security reasons, many user agents are configured to reject `Domain`
attributes that correspond to "public suffixes".  For example, some user
agents will reject `Domain` attributes of "com" or "co.uk". (See Section 5.3 for
more information.)

#### The Path Attribute

The scope of each cookie is limited to a set of paths, controlled by the
`Path` attribute. If the server omits the `Path` attribute, the user agent will
use the "directory" of the `request-uri`'s path component as the default value.
(See Section 5.1.4 for more details.)

The user agent will include the cookie in an HTTP request only if the `path`
portion of the `request-uri` matches (or is a subdirectory of) the cookie's
`Path` attribute, where the %x2F ("/") character is interpreted as a directory
separator.

Although seemingly useful for isolating cookies between different paths within
a given host, the `Path` attribute cannot be relied upon for security (see
Section 8).

#### The Secure Attribute

The `Secure` attribute limits the scope of the cookie to "secure" channels
(where "secure" is defined by the user agent).  When a cookie has the `Secure`
attribute, the user agent will include the cookie in an HTTP request only if
the request is transmitted over a secure channel (typically HTTP over Transport
Layer Security (TLS) {{RFC2818}}).

Although seemingly useful for protecting cookies from active network attackers,
the `Secure` attribute protects only the cookie's confidentiality. An active
network attacker can overwrite `Secure` cookies from an insecure channel,
disrupting their integrity (see Section 8.6 for more details).

#### The HttpOnly Attribute

The `HttpOnly` attribute limits the scope of the cookie to HTTP requests. In
particular, the attribute instructs the user agent to omit the cookie when
providing access to cookies via "non-HTTP" APIs (such as a web browser API that
exposes cookies to scripts).

Note that the `HttpOnly` attribute is independent of the `Secure` attribute: a
cookie can have both the `HttpOnly` and the `Secure` attribute.

## Cookie

### Syntax

The user agent sends stored cookies to the origin server in the `Cookie` header.
If the server conforms to the requirements in Section 4.1 (and the user agent
conforms to the requirements in Section 5), the user agent will send a `Cookie`
header that conforms to the following grammar:

~~~ abnf
cookie-header = "Cookie:" OWS cookie-string OWS
cookie-string = cookie-pair *( ";" SP cookie-pair )
~~~

### Semantics

Each `cookie-pair` represents a cookie stored by the user agent. The
`cookie-pair` contains the `cookie-name` and `cookie-value` the user agent
received in the `Set-Cookie` header.

Notice that the cookie attributes are not returned.  In particular, the server
cannot determine from the `Cookie` header alone when a cookie will expire, for
which hosts the cookie is valid, for which paths the cookie is valid, or
whether the cookie was set with the `Secure` or `HttpOnly` attributes.

The semantics of individual cookies in the `Cookie` header are not defined by
this document. Servers are expected to imbue these cookies with
application-specific semantics.

Although cookies are serialized linearly in the `Cookie` header, servers SHOULD
NOT rely upon the serialization order. In particular, if the `Cookie` header
contains two cookies with the same name (e.g., that were set with different
`Path` or `Domain` attributes), servers SHOULD NOT rely upon the order in which
these cookies appear in the header.

# User Agent Requirements

This section specifies the `Cookie` and `Set-Cookie` headers in sufficient
detail that a user agent implementing these requirements precisely can
interoperate with existing servers (even those that do not conform to the
well-behaved profile described in Section 4).

A user agent could enforce more restrictions than those specified herein (e.g.,
for the sake of improved security); however, experiments have shown that such
strictness reduces the likelihood that a user agent will be able to interoperate
with existing servers.

## Subcomponent Algorithms

This section defines some algorithms used by user agents to process specific
subcomponents of the `Cookie` and `Set-Cookie` headers.

### Dates

The user agent MUST use an algorithm equivalent to the following algorithm to
parse a `cookie-date`.  Note that the various boolean flags defined as a part
of the algorithm (i.e., `found-time`, `found-day-of-month`, `found-month`,
`found-year`) are initially "not set".

1.  Using the grammar below, divide the `cookie-date` into `date-tokens`.

    ~~~
    cookie-date     = *delimiter date-token-list *delimiter
    date-token-list = date-token *( 1*delimiter date-token )
    date-token      = 1*non-delimiter

    delimiter       = %x09 / %x20-2F / %x3B-40 / %x5B-60 / %x7B-7E
    non-delimiter   = %x00-08 / %x0A-1F / DIGIT / ":" / ALPHA / %x7F-FF
    non-digit       = %x00-2F / %x3A-FF

    day-of-month    = 1*2DIGIT ( non-digit *OCTET )
    month           = ( "jan" / "feb" / "mar" / "apr" /
                        "may" / "jun" / "jul" / "aug" /
                        "sep" / "oct" / "nov" / "dec" ) *OCTET
    year            = 2*4DIGIT ( non-digit *OCTET )
    time            = hms-time ( non-digit *OCTET )
    hms-time        = time-field ":" time-field ":" time-field
    time-field      = 1*2DIGIT
    ~~~

2.  Process each `date-token` sequentially in the order the `date-token`s
    appear in the `cookie-date`:

    1.  If the `found-time` flag is not set and the token matches the
        `time` production, set the `found-time` flag and set the `hour-value`,
        `minute-value`, and `second-value` to the numbers denoted by the digits
        in the `date-token`, respectively. Skip the remaining sub-steps and
        continue to the next `date-token`.

    2.  If the `found-day-of-month` flag is not set and the `date-token` matches
        the `day-of-month` production, set the `found-day-of-month` flag and set
        the `day-of-month-value` to the number denoted by the `date-token`. Skip
        the remaining sub-steps and continue to the next `date-token`.

    3.  If the `found-month` flag is not set and the `date-token` matches the
        `month` production, set the `found-month` flag and set the `month-value`
        to the month denoted by the `date-token`. Skip the remaining sub-steps
        and continue to the next `date-token`.

    4.  If the `found-year` flag is not set and the `date-token` matches the
        `year` production, set the `found-year` flag and set the `year-value` to
        the number denoted by the `date-token`.  Skip the remaining sub-steps
        and continue to the next `date-token`.

3.  If the `year-value` is greater than or equal to 70 and less than or equal to
    99, increment the `year-value` by 1900.

4.  If the `year-value` is greater than or equal to 0 and less than or equal to
    69, increment the `year-value` by 2000.

    1.  NOTE: Some existing user agents interpret two-digit years differently.

5.  Abort these steps and fail to parse the `cookie-date` if:

    *  at least one of the `found-day-of-month`, `found-month`, `found-year`, or
       `found-time` flags is not set,

    *  the `day-of-month-value` is less than 1 or greater than 31,

    *  the `year-value` is less than 1601,

    *  the `hour-value` is greater than 23,

    *  the `minute-value` is greater than 59, or

    *  the `second-value` is greater than 59.

    (Note that leap seconds cannot be represented in this syntax.)

6.  Let the `parsed-cookie-date` be the date whose `day-of-month`, `month`,
    `year`, `hour`, `minute`, and `second` (in UTC) are the
    `day-of-month-value`, the `month-value`, the `year-value`, the `hour-value`,
    the `minute-value`, and the `second-value`, respectively.  If no such date
    exists, abort these steps and fail to parse the `cookie-date`.

7.  Return the `parsed-cookie-date` as the result of this algorithm.

### Canonicalized Host Names

A canonicalized host name is the string generated by the following algorithm:

1.  Convert the host name to a sequence of individual domain name labels.

2.  Convert each label that is not a Non-Reserved LDH (NR-LDH) label, to an
    A-label (see Section 2.3.2.1 of {{RFC5890}} for the former and latter), or
    to a "punycode label" (a label resulting from the "ToASCII" conversion in
    Section 4 of {{RFC3490}}), as appropriate (see Section 6.3 of this
    specification).

3.  Concatenate the resulting labels, separated by a %x2E (".") character.

### Domain Matching

A string domain-matches a given domain string if at least one of the following
conditions hold:

* The domain string and the string are identical. (Note that both the domain
  string and the string will have been canonicalized to lower case at this
  point.)

* All of the following conditions hold:

  * The domain string is a suffix of the string.

  * The last character of the string that is not included in the domain string
    is a %x2E (".") character.

  * The string is a host name (i.e., not an IP address).

### Paths and Path-Match

TODO: Copy.

## The Set-Cookie Header

TODO: Copy.

# Implementation Considerations

## Limits

Practical user agent implementations have limits on the number and size of
cookies that they can store. General-use user agents SHOULD provide each of the
following minimum capabilities:

* At least 4096 bytes per cookie (as measured by the sum of the length of the
  cookie's name, value, and attributes).

* At least 50 cookies per domain.

* At least 3000 cookies total.

Servers SHOULD use as few and as small cookies as possible to avoid reaching
these implementation limits and to minimize network bandwidth due to the
`Cookie` header being included in every request.

Servers SHOULD gracefully degrade if the user agent fails to return one or more
cookies in the `Cookie` header because the user agent might evict any cookie at
any time on orders from the user.

## Application Programming Interfaces

One reason the `Cookie` and `Set-Cookie` headers use such esoteric syntax is
that many platforms (both in servers and user agents) provide a string-based
application programming interface (API) to cookies, requiring
application-layer programmers to generate and parse the syntax used by the
`Cookie` and `Set-Cookie` headers, which many programmers have done incorrectly,
resulting in interoperability problems.

Instead of providing string-based APIs to cookies, platforms would be
well-served by providing more semantic APIs. It is beyond the scope of this
document to recommend specific API designs, but there are clear benefits to
accepting an abstract "`Date`" object instead of a serialized date string.

## IDNA Dependency and Migration

IDNA2008 {{RFC5890}} supersedes IDNA2003 {{RFC3490}}. However, there are
differences between the two specifications, and thus there can be differences
in processing (e.g., converting) domain name labels that have been registered
under one from those registered under the other. There will be a transition
period of some time during which IDNA2003-based domain name labels will exist
in the wild. User agents SHOULD implement IDNA2008 {{RFC5890}} and MAY
implement {{UTS46}} or {{RFC5895}} in order to facilitate their IDNA transition.
If a user agent does not implement IDNA2008, the user agent MUST implement
IDNA2003 {{RFC3490}}.

# Privacy Considerations

Cookies are often criticized for letting servers track users. For example, a
number of "web analytics" companies use cookies to recognize when a user returns
to a web site or visits another web site.  Although cookies are not the only
mechanism servers can use to track users across HTTP requests, cookies
facilitate tracking because they are persistent across user agent sessions and
can be shared between hosts.

## Third-Party Cookies

Particularly worrisome are so-called "third-party" cookies. In rendering an HTML
document, a user agent often requests resources from other servers (such as
advertising networks). These third-party servers can use cookies to track the
user even if the user never visits the server directly. For example, if a user
visits a site that contains content from a third party and then later visits
another site that contains content from the same third party, the third party
can track the user between the two sites.

Some user agents restrict how third-party cookies behave. For example, some of
these user agents refuse to send the `Cookie` header in third-party requests.
Others refuse to process the `Set-Cookie` header in responses to third-party
requests. User agents vary widely in their third-party cookie policies. This
document grants user agents wide latitude to experiment with third-party cookie
policies that balance the privacy and compatibility needs of their users.
However, this document does not endorse any particular third-party cookie
policy.

Third-party cookie blocking policies are often ineffective at achieving their
privacy goals if servers attempt to work around their restrictions to track
users. In particular, two collaborating servers can often track users without
using cookies at all by injecting identifying information into dynamic URLs.

## User Controls

User agents SHOULD provide users with a mechanism for managing the cookies
stored in the cookie store. For example, a user agent might let users delete
all cookies received during a specified time period or all the cookies related
to a particular domain. In addition, many user agents include a user interface
element that lets users examine the cookies stored in their cookie store.

User agents SHOULD provide users with a mechanism for disabling cookies. When
cookies are disabled, the user agent MUST NOT include a `Cookie` header in
outbound HTTP requests and the user agent MUST NOT process `Set-Cookie` headers
in inbound HTTP responses.

Some user agents provide users the option of preventing persistent storage of
cookies across sessions. When configured thusly, user agents MUST treat all
received cookies as if the `persistent-flag` were set to `false`. Some popular
user agents expose this functionality via "private browsing" mode
{{Aggarwal2010}}.

Some user agents provide users with the ability to approve individual writes to
the cookie store. In many common usage scenarios, these controls generate a
large number of prompts. However, some privacy-conscious users find these
controls useful nonetheless.

## Expiration Dates

Although servers can set the expiration date for cookies to the distant future,
most user agents do not actually retain cookies for multiple decades. Rather
than choosing gratuitously long expiration periods, servers SHOULD promote user
privacy by selecting reasonable cookie expiration periods based on the purpose
of the cookie. For example, a typical session identifier might reasonably be
set to expire in two weeks.

# Security Considerations

## Overview

Cookies have a number of security pitfalls.  This section overviews a few of the
more salient issues.

In particular, cookies encourage developers to rely on ambient authority for
authentication, often becoming vulnerable to attacks such as cross-site request
forgery {{CSRF}}.  Also, when storing session identifiers in cookies, developers
often create session fixation vulnerabilities.

Transport-layer encryption, such as that employed in HTTPS, is insufficient to
prevent a network attacker from obtaining or altering a victim's cookies because
the cookie protocol itself has various vulnerabilities (see "Weak Confidentiality"
and "Weak Integrity", below).  In addition, by default, cookies do not provide
confidentiality or integrity from network attackers, even when used in conjunction
with HTTPS.

## Ambient Authority

A server that uses cookies to authenticate users can suffer security
vulnerabilities because some user agents let remote parties issue HTTP requests
from the user agent (e.g., via HTTP redirects or HTML forms).  When issuing
those requests, user agents attach cookies even if the remote party does not
know the contents of the cookies, potentially letting the remote party exercise
authority at an unwary server.

Although this security concern goes by a number of names (e.g., cross-site
request forgery, confused deputy), the issue stems from cookies being a form of
ambient authority.  Cookies encourage server operators to separate designation
(in the form of URLs) from authorization (in the form of cookies).
Consequently, the user agent might supply the authorization for a resource
designated by the attacker, possibly causing the server or its clients to
undertake actions designated by the attacker as though they were authorized by
the user.

Instead of using cookies for authorization, server operators might wish to
consider entangling designation and authorization by treating URLs as
capabilities.  Instead of storing secrets in cookies, this approach stores
secrets in URLs, requiring the remote entity to supply the secret itself.
Although this approach is not a panacea, judicious application of these
principles can lead to more robust security.

## Clear Text

Unless sent over a secure channel (such as TLS), the information in the `Cookie`
and `Set-Cookie` headers is transmitted in the clear.

1.  All sensitive information conveyed in these headers is exposed to an
    eavesdropper.

2.  A malicious intermediary could alter the headers as they travel in either
    direction, with unpredictable results.

3.  A malicious client could alter the `Cookie` header before transmission,
    with unpredictable results.

Servers SHOULD encrypt and sign the contents of cookies (using whatever format
the server desires) when transmitting them to the user agent (even when sending
the cookies over a secure channel). However, encrypting and signing cookie
contents does not prevent an attacker from transplanting a cookie from one user
agent to another or from replaying the cookie at a later time.

In addition to encrypting and signing the contents of every cookie, servers that
require a higher level of security SHOULD use the `Cookie` and `Set-Cookie`
headers only over a secure channel.  When using cookies over a secure channel,
servers SHOULD set the `Secure` attribute (see Section 4.1.2.5) for every
cookie.  If a server does not set the `Secure` attribute, the protection
provided by the secure channel will be largely moot.

For example, consider a webmail server that stores a session identifier in a
cookie and is typically accessed over HTTPS.  If the server does not set the
`Secure` attribute on its cookies, an active network attacker can intercept any
outbound HTTP request from the user agent and redirect that request to the
webmail server over HTTP. Even if the webmail server is not listening for HTTP
connections, the user agent will still include cookies in the request.  The
active network attacker can intercept these cookies, replay them against the
server, and learn the contents of the user's email.  If, instead, the server had
set the `Secure` attribute on its cookies, the user agent would not have
included the cookies in the clear-text request.

## Session Identifiers

Instead of storing session information directly in a cookie (where it might be
exposed to or replayed by an attacker), servers commonly store a nonce (or
"session identifier") in a cookie.  When the server receives an HTTP request
with a nonce, the server can look up state information associated with the
cookie using the nonce as a key.

Using session identifier cookies limits the damage an attacker can cause if the
attacker learns the contents of a cookie because the nonce is useful only for
interacting with the server (unlike non-nonce cookie content, which might itself
be sensitive).  Furthermore, using a single nonce prevents an attacker from
"splicing" together cookie content from two interactions with the server, which
could cause the server to behave unexpectedly.

Using session identifiers is not without risk.  For example, the server SHOULD
take care to avoid "session fixation" vulnerabilities. A session fixation attack
proceeds in three steps.  First, the attacker transplants a session identifier
from his or her user agent to the victim's user agent.  Second, the victim uses
that session identifier to interact with the server, possibly imbuing the
session identifier with the user's credentials or confidential information.
Third, the attacker uses the session identifier to interact with server
directly, possibly obtaining the user's authority or confidential information.

## Weak Confidentiality

   Cookies do not provide isolation by port.  If a cookie is readable by
   a service running on one port, the cookie is also readable by a
   service running on another port of the same server.  If a cookie is
   writable by a service on one port, the cookie is also writable by a
   service running on another port of the same server.  For this reason,
   servers SHOULD NOT both run mutually distrusting services on
   different ports of the same host and use cookies to store security-
   sensitive information.

   Cookies do not provide isolation by scheme.  Although most commonly
   used with the http and https schemes, the cookies for a given host
   might also be available to other schemes, such as ftp and gopher.
   Although this lack of isolation by scheme is most apparent in non-
   HTTP APIs that permit access to cookies (e.g., HTML's document.cookie
   API), the lack of isolation by scheme is actually present in
   requirements for processing cookies themselves (e.g., consider
   retrieving a URI with the gopher scheme via HTTP).

   Cookies do not always provide isolation by path.  Although the
   network-level protocol does not send cookies stored for one path to
   another, some user agents expose cookies via non-HTTP APIs, such as
   HTML's document.cookie API.  Because some of these user agents (e.g.,
   web browsers) do not isolate resources received from different paths,
   a resource retrieved from one path might be able to access cookies
   stored for another path.

## Weak Integrity

Cookies do not provide integrity guarantees for sibling domains (and their
subdomains). For example, consider `foo.example.com` and `bar.example.com`. The
`foo.example.com` server can set a cookie with a `Domain` attribute of
"`example.com`" (possibly overwriting an existing "`example.com`" cookie set by
`bar.example.com`), and the user agent will include that cookie in HTTP requests
to `bar.example.com`.  In the worst case, `bar.example.com` will be unable to
distinguish this cookie from a cookie it set itself. The `foo.example.com`
server might be able to leverage this ability to mount an attack against
`bar.example.com`.

Even though the `Set-Cookie` header supports the `Path` attribute, the `Path`
attribute does not provide any integrity protection because the user agent
will accept an arbitrary `Path` attribute in a `Set-Cookie` header.  For
example, an HTTP response to a request for `http://example.com/foo/bar` can set
a cookie with a `Path` attribute of "/qux".  Consequently, servers SHOULD NOT
both run mutually distrusting services on different paths of the same host and
use cookies to store security-sensitive information.

An active network attacker can also inject cookies into the `Cookie` header
sent to `https://example.com/` by impersonating a response from
`http://example.com/` and injecting a `Set-Cookie` header.  The HTTPS server
at `example.com` will be unable to distinguish these cookies from cookies that
it set itself in an HTTPS response.  An active network attacker might be able
to leverage this ability to mount an attack against `example.com` even if
`example.com` uses HTTPS exclusively.

Servers can partially mitigate these attacks by encrypting and signing the
contents of their cookies.  However, using cryptography does not mitigate the
issue completely because an attacker can replay a cookie he or she received from
the authentic `example.com` server in the user's session, with unpredictable
results.

Finally, an attacker might be able to force the user agent to delete cookies by
storing a large number of cookies.  Once the user agent reaches its storage
limit, the user agent will be forced to evict some cookies.  Servers SHOULD NOT
rely upon user agents retaining cookies.

## Reliance on DNS

Cookies rely upon the Domain Name System (DNS) for security.  If the DNS is
partially or fully compromised, the cookie protocol might fail to provide the
security properties required by applications.

# IANA Considerations

The permanent message header field registry (see {{RFC3864}}) has been updated
with the following registrations.

## Cookie

Header field name:
: Cookie

Applicable protocol:
: http

Status:
: standard

Author/Change controller:
: IETF

Specification document:
: this specification (Section 5.4)

## Set-Cookie

Header field name:
: Set-Cookie

Applicable protocol:
: http

Status:
: standard

Author/Change controller:
: IETF

Specification document:
: this specification (Section 5.2)

## Cookie2

Header field name:
: Cookie2

Applicable protocol:
: http

Status:
: obsoleted

Author/Change controller:
: IETF

Specification document:
: {{RFC2965}}

## Set-Cookie2

Header field name:
: Set-Cookie2

Applicable protocol:
: http

Status:
: obsoleted

Author/Change controller:
: IETF

Specification document:
: {{RFC2965}}

--- back

# Acknowledgements

This document borrows heavily from RFC 2109 {{RFC2109}}.  We are indebted to
David M. Kristol and Lou Montulli for their efforts to specify cookies.  David
M. Kristol, in particular, provided invaluable advice on navigating the IETF
process.  We would also like to thank Thomas Broyer, Tyler Close, Alissa Cooper,
Bil Corry, corvid, Lisa Dusseault, Roy T. Fielding, Blake Frantz, Anne van
Kesteren, Eran Hammer-Lahav, Jeff Hodges, Bjoern Hoehrmann, Achim Hoffmann,
Georg Koppen, Dean McNamee, Alexey Melnikov, Mark Miller, Mark Pauley, Yngve N.
Pettersen, Julian Reschke, Peter Saint-Andre, Mark Seaborn, Maciej Stachowiak,
Daniel Stenberg, Tatsuhiro Tsujikawa, David Wagner, Dan Winship, and Dan Witte
for their valuable feedback on this document.
