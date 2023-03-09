---
v: 3

title: "Updates to the CDDL grammar of RFC 8610"
abbrev: "CDDL grammar updates"

docname: draft-bormann-update-8610-grammar-latest
stream: IETF
updates: 8610
# date:
cat: std
consensus: true
area: ART
workgroup: CBOR
keyword:
 - Concise Data Definition Language
venue:
  group: CBOR
  mail: cbor@ietf.org
  github: cbor-wg/update-8610-grammar

author:
  -
    name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org


normative:
  RFC8610: cddl
  RFC9165: control1

informative:
  RFC5234: abnf
  RFC7405: abnf-case
  I-D.bormann-cbor-cddl-more-control: more-control
  I-D.draft-bormann-cbor-cddl-freezer: freezer
  I-D.bormann-cbor-cddl-2-draft: cddl-2-draft
#  useful:
#    target: https://github.com/cbor-wg/cddl/wiki/Useful-CDDL
#    title: Useful CDDL
  Err6526:
    target: "https://www.rfc-editor.org/errata/eid6526"
    title: Errata Report 6526
    seriesinfo:
      RFC: 8610
    date: false
  Err6527:
    target: "https://www.rfc-editor.org/errata/eid6527"
    title: Errata Report 6527
    seriesinfo:
      RFC: 8610
    date: false
  Err6543:
    target: "https://www.rfc-editor.org/errata/eid6543"
    title: Errata Report 6543
    seriesinfo:
      RFC: 8610
    date: false


--- abstract

[^abs1-]

[^abs1-]:
    At the time of writing,
    the Concise Data Definition Language (CDDL) is defined by
    RFC 8610 and RFC 9165.
    The latter has used the extension point provided in RFC 8610,
    the _control operator_.

    As CDDL is being used in larger projects, the need for corrections
    and additional features has become known that cannot be easily
    mapped into this single extension point.
    Hence, there is a need for evolution of the base CDDL
    specification itself.

    The present document updates errata and makes other small fixes for
    the ABNF grammar defined for CDDL in RFC 8610.

[^status]

[^status]: Previous versions of the changes in this document were part
    of draft-bormann-cbor-cddl-2-draft and previously
    draft-bormann-cbor-cddl-freezer.
    This submission extracts out those grammar changes that are ready
    for WG adoption and publication.

--- middle

# Introduction

[^abs1-]

[^status]

[^seealso]

[^seealso]: Proposals for grammar and other changes that need more
    work can be found in {{-cddl-2-draft}}.  Proposals for other
    additions to the CDDL specification base are in {{-freezer}}.

Note that the existing extension point "control operator" ({{Section
3.8 of -cddl}}) can be exercised for new features in parallel to the
work described here; one set of such proposals is in {{-more-control}}.

The present document, in conjunction with {{-cddl}} as well as
{{-control1}} and {{-more-control}}, is intended to be the specification
base of what has colloquially been called CDDL 1.1.
Additional documents describe further work on CDDL.

## Conventions and Definitions

The Terminology from {{-cddl}} applies.
The grammar in {{-cddl}} is based on ABNF, which is defined in {{-abnf}}
and {{-abnf-case}}.

{::boilerplate bcp14-tagged}

# Clarifications and Changes based on Errata Reports {#clari}


*Compatibility*:
: errata fix (targets 1.0 and 2.0)

A number of errata reports have been made around some details of text
string and byte string literal syntax: {{Err6527}} and {{Err6543}}.
These are being addressed in this section, updating details of the
ABNF for these literal syntaxes.
Also, {{Err6526}} needs to be applied (backslashes have been lost during
RFC processing in some text explaining backslash escaping).

## Err6527 (text string literals) {#e6527}

The ABNF used in {{RFC8610}} for the content of text string literals
is rather permissive:

~~~ abnf
; RFC 8610 ABNF:
text = %x22 *SCHAR %x22
SCHAR = %x20-21 / %x23-5B / %x5D-7E / %x80-10FFFD / SESC
SESC = "\" (%x20-7E / %x80-10FFFD)
~~~

This allows almost any non-C0 character to be escaped by a backslash,
but critically misses out on the `\uXXXX` and `\uHHHH\uLLLL` forms
that JSON allows to specify characters in hex (which should be
applying here according to Bullet 6 of {{Section 3.1 of -cddl}}).
Both can be solved by updating the SESC production to:

~~~ abnf
; new rules collectively defining SESC:
SESC = "\" ( %x22 / "/" / "\" /                 ; \" \/ \\
             %x62 / %x66 / %x6E / %x72 / %x74 / ; \b \f \n \r \t
             (%x75 hexchar) )                   ; \uXXXX
hexchar = non-surrogate / (high-surrogate "\" %x75 low-surrogate)
non-surrogate = ((DIGIT / "A"/"B"/"C" / "E"/"F") 3HEXDIG) /
                ("D" %x30-37 2HEXDIG )
high-surrogate = "D" ("8"/"9"/"A"/"B") 2HEXDIG
low-surrogate = "D" ("C"/"D"/"E"/"F") 2HEXDIG
~~~
{: #e6527-new1 title="Updated string parsing to allow hex escapes"}

(Notes:
In ABNF, strings such as `"A"`, `"B"` etc. are case-insensitive, as is
intended here.
We could have written `%x62` as `%s"b"`, but didn't, in order to
maximize ABNF tool compatibility.)

Now that SESC is more restrictively formulated, this also requires an
update to the BCHAR production used in the ABNF syntax for byte string
literals:

~~~ abnf
; RFC 8610 ABNF:
bytes = [bsqual] %x27 *BCHAR %x27
BCHAR = %x20-26 / %x28-5B / %x5D-10FFFD / SESC / CRLF
bsqual = "h" / "b64"
~~~

In BCHAR, the updated version explicitly allows `\'`, which is no
longer allowed in the updated SESC:

~~~ abnf
; new rule for BCHAR:
BCHAR = %x20-26 / %x28-5B / %x5D-10FFFD / SESC / "\'" / CRLF
~~~
{: #e6527-new2 title="Updated rule for BCHAR"}

## Err6543 (byte string literals)

The ABNF used in {{RFC8610}} for the content of byte string literals
lumps together byte strings notated as text with byte strings notated
in base16 (hex) or base64 (but see also updated BCHAR production above):

~~~ abnf
; RFC 8610 ABNF:
bytes = [bsqual] %x27 *BCHAR %x27
BCHAR = %x20-26 / %x28-5B / %x5D-10FFFD / SESC / CRLF
~~~

### Change proposed by Errata Report 6543
{:unnumbered}

Errata report 6543 proposes to handle the two cases in separate
productions (where, with an updated SESC, BCHAR obviously needs to be
updated as above):

~~~ abnf
; Err6543 proposal:
bytes = %x27 *BCHAR %x27
      / bsqual %x27 *QCHAR %x27
BCHAR = %x20-26 / %x28-5B / %x5D-10FFFD / SESC / CRLF
QCHAR = DIGIT / ALPHA / "+" / "/" / "-" / "_" / "=" / WS
~~~~
{: #e6543-1 title="Errata Report 8653 Proposal to Split the Byte String Rules"}

This potentially causes a subtle change, which is hidden in the WS production:

~~~ abnf
; RFC 8610 ABNF:
WS = SP / NL
SP = %x20
NL = COMMENT / CRLF
COMMENT = ";" *PCHAR CRLF
PCHAR = %x20-7E / %x80-10FFFD
CRLF = %x0A / %x0D.0A
~~~
{: #e6543-2 title="Definition of WS from RFC 8610"}

This allows any non-C0 character in a comment, so this fragment
becomes possible:

~~~ cddl
foo = h'
   43424F52 ; 'CBOR'
   0A       ; LF, but don't use CR!
'
~~~

The current text is not unambiguously saying whether the three apostrophes
need to be escaped with a `\` or not, as in:

~~~ cddl
foo = h'
   43424F52 ; \'CBOR\'
   0A       ; LF, but don\'t use CR!
'
~~~

... which would be supported by the existing ABNF in {{-cddl}}.

### No change needed after {{e6527}}
{:unnumbered}

This document takes the simpler approach of leaving the processing of
the content of the byte string literal to a semantic step after
processing the syntax of the `bytes`/`BCHAR` rules as updated by
{{e6527-new1}} and {{e6527-new2}}.

The rules in {{e6543-2}} are therefore applied to the result of this
processing where `bsqual` is given as `h` or `b64`.

Note that this approach also works well with the use of byte strings
in {{Section 3 of -control1}}.
It does require some care when copy-pasting into CDDL models from ABNF
that contains single quotes (which may also hide as apostrophes
in comments); these need to be escaped or possibly replaced by `%x27`.

Finally, our approach may leave the door open wider to extending
`bsqual` as proposed in {{Appendix A.1 of -cddl-2-draft}}.


# Small Enabling Grammar Changes

The two subsections in this section specify two small changes to the
grammar that are intended to enable certain kinds of specifications.

Empty data models {#empty}
-----------------

{:compact}
*Compatibility*:
: backward (not forward)

{{-cddl}} requires a CDDL file to have at least one rule.


~~~ abnf
; RFC 8610 ABNF:
cddl = S 1*(rule S)
~~~


This makes sense when the file has to stand alone, as a CDDL data
model needs to have at least one rule to provide an entry point (start
rule).

With CDDL 2.0, CDDL files can also include directives<!-- (see
{{directives}}) -->, and these might be the source of all the rules that
ultimately make up the module created by the file.
Any other rule content in the file has to be available for directive
processing, making the requirement for at least one rule cumbersome.

Therefore, we extend the grammar as follows:

~~~ abnf
; new top-level rule:
cddl = S *(rule S)
~~~

and make the existence of at least one rule a semantic constraint, to
be fulfilled after processing of all directives.


Non-literal Tag Numbers {#tagnum}
-----------------------

{:compact}
*Compatibility*:
: backward (not forward)

The CDDL 1.0 syntax for expressing tags in CDDL is (ABNF as in {{-abnf}}):

~~~ abnf
; extracted from RFC 8610 ABNF:
type2 /= "#" "6" ["." uint] "(" S type S ")"
~~~

This means tag numbers can only be given as literal numbers (uints).
Some specifications operate on ranges of tag numbers, e.g., {{?RFC9277}}
has a range of tag numbers 1668546817 (0x63740101) to 1668612095
(0x6374FFFF) to tag specific content formats.
This can currently not be expressed in CDDL.

CDDL 2.0 extends this to:

~~~ abnf
; new rules collectively defining the tagged case:
type2 /= "#" "6" ["." tag-number] "(" S type S ")"
tag-number = uint / ("<" type ">")
~~~

So the above range can be expressed in a CDDL fragment such as:

~~~ cddl
ct-tag<content> = #6.<ct-tag-number>(content)
ct-tag-number = 1668546817..1668612095
; or use 0x63740101..0x6374FFFF
~~~

Note that this syntax reuses the angle bracket syntax for generics;
this reuse is innocuous as a generic parameter/argument only ever
occurs after a rule name (`id`), while it occurs after `.` here.
(Whether there is potential for human confusion can be debated; the
above example deliberately uses generics as well.)

# Security Considerations

The grammar fixes and updates in this document are not believed to
create additional security considerations.
The security considerations in {{Section 5 of -cddl}} do apply, and
specifically the potential for confusion is increased in an
environment that uses a combination of CDDL tools some of which have
been updated and some of which have not been, in particular based on
{{clari}}.

# IANA Considerations

This document has no IANA actions.


--- back

# Updated Collected ABNF for CDDL

This appendix provides the full ABNF from {{-cddl}} with the updates
applied in the present document.

~~~ cddl
{::include cddl-1-1-update.abnf}
~~~
{: #collected-abnf title="ABNF for CDDL as updated"}

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
