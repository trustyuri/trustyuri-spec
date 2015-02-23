Trusty URI Specification - Version 1
====================================

_Tobias Kuhn, 23 February 2015, http://www.trustyuri.net_

This document contains version 1 of the specification of the _trusty URI_
approach.

Permanent URL for this version:
http://trustyuri.net/spec/v1.FADQoZWcYugekAb4jW-Zm3_5Cd9tmkkYEV0bxK2fLSKao.md

The previous version of this document was version 0, which was preliminary and
not stable:
http://trustyuri.net/spec/v0.FA4BwXfTl2X-ABWKUF2k0T044yS2-KmO_R0zBftSsc96k.md

Version 1 described here is _not_ backward compatible with version 0 for module
`RA` (the handling of plain literals according to RDF 1.1 breaks compatibility).
Future versions, however, are supposed to be 100% backward compatible with this
version. This means that when the extension of modules cannot be done in a
backward compatible manner in the future, new modules will have to be defined
and old ones will have to be deprecated.


Basics
------

Hash values of trusty URIs are encoded in Base64 notation with some common
modifications for making it safe to use them in URIs and file names:

> **Definition 1.**
> Every character that is a standard ASCII letter (`A-Z` or `a-z`), a digit
> (`0-9`), a hyphen (`-`), or an underscore (`_`) is called a _Base64
> character_, representing in this order the numbers from 0 to 63. There are no
> other Base64 characters.

Trusty URIs have the following structure:

> **Definition 2.**
> Every trusty URI ends with at least 25 Base64 characters. The sequence of
> characters following the last non-Base64 character is called the _artifact
> code_. The first two characters of the artifact code are called the _module
> identifier_. The sequence of characters following the module identifier is
> called _data part_, which is identical to or contains a _hash part_.

The current modules only generate URIs with exactly 45 trailing Base64
characters, but the definition is kept open for future modules.

The first character of the module identifier specifies the type of the content
and therefore the type of the module; the second character is a version
identifier of the module. The main content of the data part is the hash value,
but it can also contain other information such as parameters and sub-types. Its
concrete structure depends on the module.

As everybody who has access to the respective domain is free to define and use
URIs at will, we can only be sure that a certain URI is a trusty URI once we
have found and verified a content that matches the hash. For that reason, the
concept of a _potential trusty URI_ needs to be introduced:

> **Definition 3.**
> Every URI that could be a trusty URI according to the restrictions of
> Definition 2 with a module identifier matching a defined module and with a
> data part that is consistent with the structural restrictions of the given
> module (in particular with respect to its length) is called a _potential
> trusty URI_.

With these ingredients, trusty URIs can be verified:

> **Definition 4.**
> Given a potential trusty URI and a digital artifact, if the identifier part
> refers to an module that returns a hash value for the digital artifact that is
> identical to the one encoded in the hash part, then the potential trusty URI
> is a _verified trusty URI_ and the digital artifact is its _verified content_.

For convenience reasons, we can append a file extension like `.txt` or `.nq` to
trusty URIs. The resulting URIs are technically no trusty URIs anymore, but it
is easy to strip the extension and get the respective trusty URIs.

As the hash values are located in the final part of the URIs, it is
straightforward to also use them in file names and to deal with them in a local
file system without worrying about the first part of the URI. For example, the
name of such a file could therefore be:

    r1.RAcbjcRIQozo2wBMq4WcCYkFAjRz0AX-Ux3PquZZrC68s.nq

Such files are called _trusty files_.


Modules
-------

There are currently three modules available: `FA`, `RA`, and `RB`.


### Module FA

Version A of module type F, i.e. module `FA`, works on the byte content of
files. A hash value is calculated using SHA-256 [SHA-256] on the content of the
file in byte representation. The file name and other metadata are not
considered. Two zero-bits are appended to the resulting hash value, and then
transformed to Base64 notation as defined above. The resulting 43 characters
make up the data part of the trusty URI.

Empty files, for example, get the following URI suffix:

    FA47DEQpj8HBSa-\_TImW-5JCeuQeRkm5NMpJWZG3hSuFU

When adding such a suffix to a URI, it has to be made sure that it is preceded
by a non-Base64 character, such as a dot (`.`), a slash (`/`), or a hash sign
(`#`). This applies to all modules.


### Module RA

Version A of module type R, i.e. module `RA`, works on RDF content, possibly
covering multiple named graphs, relying on RDF version 1.1 [RDF].

This module allows for self-references, i.e. the trusty URI itself may appear in
the RDF data it represents. URIs consisting of the given trusty URI and a suffix
are also supported, such as:

    http://example.org/r2.RA5AbXdpz5DcaYXCh9l3eI9ruBosiL5XDU3rxBbBaUO70#Part1
    http://example.org/r2.RA5AbXdpz5DcaYXCh9l3eI9ruBosiL5XDU3rxBbBaUO70#Part2

Blank nodes are not supported and have to be skolemized in the same way when a
trusty URI is produced. It is furthermore assumed that the data is a set of
named RDF graphs. RDF triples without a named graph are considered to belong to
a special named graph represented with the empty string.

To check whether a given artifact code correctly represents a given set of named
graphs, the triples and graphs have to be sorted first. Because the trusty URI
can appear in the RDF data it represents, all occurrences of the given artifact
code in the URIs have to be replaced by a blank character in a preprocessing
step. To determine the order of any two triples, the first applicable rule of
the following list is applied:

1. If their graph URIs differ, the triple with the lexicographically smaller
   preprocessed graph URI is first.
2. If their subject URIs differ, the triple with the lexicographically smaller
   preprocessed subject URI is first.
3. If their predicate URIs differ, the triple with the lexicographically smaller
   preprocessed predicate URI is first.
4. If one has a literal as object and the other has a non-literal, the triple
   with the non-literal as object is first.
5. If both have a URI as object, the triple with the lexicographically smaller
   preprocessed object URI is first.
6. If the literal labels of the objects differ, the triple with the
   lexicographically smaller literal label is first.
7. If one of the object literals has a datatype identifier and the other does
   not, the triple without a datatype identifier is first.
8. If one of the object literals has a language identifier and the other does
   not, the triple without a language identifier is first.
9. The triple with the lexicographically smaller datatype or language identifier
   is first.

The lexicographic order is defined on strings of Unicode characters. If two
strings have different characters at at least one position, the string with the
smaller integer value at the first differing position is first. Otherwise, the
shorter string is first.

After the triples have been sorted, a sequence of Unicode characters _s_ is
built. For each triple, the serialization of its graph, its subject, its
predicate, and its object are added to the end of _s_, in this order and with a
newline character at the end of each of the four. The serialization of graph,
subject, and predicate identifiers is simply their preprocessed URI string.
Objects that consist of a URI are treated the same way. Literals without a
language tag are serialized as a circumflex character (`^`) followed by the
datatype URI (which, according to RDF 1.1, equals
`http://www.w3.org/2001/XMLSchema#string` if not explicitly specified), a blank
space, and the escaped literal string. Literal strings are escaped by replacing
`\` by `\\` and newline characters by `\n`. Literals with a language tag are
serialized as an at-sign `@` followed by the canonicalized language string, a
blank space, and the escaped literal string.

The actual computation of the hash data is identical to Module F: a SHA-256 hash
is generated for _s_ in UTF-8 encoding, two zero-bits are appended, and the
result is transformed to Base64 notation.


### Module RB

Version B of module type R, i.e. module `RB`, is a slight variation of module
`RA`. While module `RA` can represent any number of RDF graphs, a trusty URI
using module `RB` always represents just one graph. The calculation of the hash
is identical to the procedure described above, with the only restriction being
that all triples have the trusty URI of the given resource as their graph URI.


References
----------

- [SHA-256] National Institute of Standards and Technology (NIST). "Secure Hash
  Standard (SHS)," FIPS PUB 180-4, March 2012.
  http://csrc.nist.gov/publications/fips/fips180-4/fips-180-4.pdf
- [RDF] Richard Cyganiak, David Wood, and Markus Lanthaler. "RDF 1.1 Concepts
  and Abstract Syntax," W3C Recommendation, 25 February 2014.
  http://www.w3.org/TR/rdf11-concepts/
