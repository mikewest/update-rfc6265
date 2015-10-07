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

# IANA Considerations

The permanent message header field registry (see {{RFC3864}}) has been updated
with the following registrations.

## Cookie

: Header field name
:: Cookie
: Applicable protocol
:: http
: Status
:: standard
: Author/Change controller
:: IETF
: Specification document
:: this specification (Section 5.4)

## Set-Cookie

: Header field name
:: Set-Cookie
: Applicable protocol
:: http
: Status
:: standard
: Author/Change controller
:: IETF
: Specification document
:: this specification (Section 5.2)

## Cookie2

: Header field name
:: Cookie2
: Applicable protocol
:: http
: Status
:: obsoleted
: Author/Change controller
:: IETF
: Specification document
:: {{RFC2965}}

## Set-Cookie2

: Header field name
:: Set-Cookie2
: Applicable protocol
:: http
: Status
:: obsoleted
: Author/Change controller
:: IETF
: Specification document
:: {{RFC2965}}

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
