Hash-URI Specification
======================

This document is **work in progress**. It contains the specification of the
hash-URI approach.


Example
-------

Generally, hash-URIs are URIs that contain a certain kind of hash value that
can be used to verify the respective resource. This is an example of a
hash-URI:

    http://example.org/np1.RAcbjcRIQozo2wBMq4WcCYkFAjRz0AX-Ux3PquZZrC68s

The last 45 characters of this URI (after the period symobl `.`) are the
hash-URI part. The first two characters of the hash-URI part (`RA` in this
example) define the type and version of the algorithm. Only `FA` for plain
file content and `RA` for sets of RDF graphs are supported at this point.
The remaining 43 characters are the actual hash value that represents the
content of the resource this URI represents.


Implementations
---------------

There are currently three (partial) implementations:

- hashuri-java: https://github.com/tkuhn/hashuri-java
- hashuri-perl: https://github.com/tkuhn/hashuri-perl
- hashuri-python: https://github.com/tkuhn/hashuri-python


Basics
------

Hash values of hash-URIs are encoded in Base64 notation with some common
modifications for making it safe to use them in URIs and filenames:

> **Definition.**
> Every character that is a standard ASCII letter (`A-Z` or `a-z`), a digit
> (`0-9`), a hyphen (`-`), or an underscore (`_`) is called a _Base64
> character_, representing in this order the numbers from 0 to 63. There are no
> other Base64 characters.

As everybody who has access to the respective domain is free to define and use
URIs at will, we can only be sure that a certain URI is a hash-URI once we
have found and verified a content that matches the hash. For that reason, we
need to introduce the concept of a _potential hash-URI_:

> **Definition.**
> If the last 25 characters of a URI are all Base64 characters, then this URI
> is a _potential hash-URI_.

Our concrete proposals only generate URIs with exactly 45 trailing Base64
characters, but we keep the definition open for future additions and
extensions. Such character sequences consist of two parts:

> **Definition.**
> The two characters immediately following the last non-Base64 character of a
> potential hash-URI are called _identifier part_. The sequence of characters
> following the identifier part is called _data part_, which is identical to or
> contains a _hash part_.

The first character of the _identifier part_ specifies the type of content
and therefore the type of algorithm; the second character is a version number
of the algorithm. The main content of the _data part_ is the hash value, but
it can also contain other information such as parameters and sub-types. Its
concrete structure depends on the algorithm. With these ingredients,
hash-URIs can be _verified_:

> **Definition.**
> Given a potential hash-URI and a digital artifact, if the identifier part
> refers to an algorithm that returns a hash value for the digital artifact
> that is identical to the one encoded in the hash part, then the potential
> hash-URI is a _verified hash-URI_ and the digital artifact is its _verified
> resource_.

We can append a file extension like `.txt` or `.nq` to a hash-URI. The
resulting URIs are technically no hash-URIs anymore, but it is easy to strip
the extension and get the respective hash-URI from it.


Algorithms
----------

There are currently two algorithms available: `RA` and `FA`.


### Algorithm FA

Version A of algorithm F (i.e. `FA`) works on the byte content of files.

A hash value is calculated using SHA-256 on the content of the file in byte representation. The file name and other metadata are not considered. Two zero-bits are appended to the resulting hash value, and then transformed to Base64 notation as described above. The resulting 43 characters make up the data part of the hash-URI.

For empty files, for example, we get the following URI suffix:

> FA47DEQpj8HBSa-_TImW-5JCeuQeRkm5NMpJWZG3hSuFU

In general, when adding such a suffix to a URI, it has to be made sure that it is preceded by a non-Base64 character, such as a dot (`.`), a slash (`/`), or a hash sign (`#`).


### Algorithm RA

Version A of algorithm R (i.e. `RA`) works on RDF content, possibly covering multiple named graphs.

This algorithm does not allow for blank nodes, because they make graph normalization much more difficult (future versions might support it though). The algorithm supports, however, that the hash-URI itself appears in the RDF data it represents, including extended URIs such as:

> http://example.org/resource1.RA5AbXdpz5DcaYXCh9l3eI9ruBosiL5XDU3rxBbBaUO70.Part1
> http://example.org/resource2.RA5AbXdpz5DcaYXCh9l3eI9ruBosiL5XDU3rxBbBaUO70.Part2

This can also be used to handle blank nodes. For example, blank nodes can be transformed into URIs of the form:

> http://example.org/resource1.RA5AbXdpz5DcaYXCh9l3eI9ruBosiL5XDU3rxBbBaUO70..1
> http://example.org/resource1.RA5AbXdpz5DcaYXCh9l3eI9ruBosiL5XDU3rxBbBaUO70..2

It is assumed that the data is a set of named RDF graphs. For sets of RDF triples without a named graph, they are all considered to belong to a special named graph that is represented with the empty string.

To check whether a given hash _h_ correctly represents a given set of named graphs, we first have to sort the triples of all named graphs. Because the hash-URI can appear in the RDF data it represents, we first have to replace all occurrences of _h_ in the URIs by a blank character in a preprocessing step. To determine the order of any two triples, the first applicable rule of the following list is applied:

1. If their graph URIs differ, the triple with the lexicographically smaller preprocessed graph URI is first.
2. If their subject URIs differ, the triple with the lexicographically smaller preprocessed subject URI is first.
3. If their predicate URIs differ, the triple with the lexicographically smaller preprocessed predicate URI is first.
4. If one has a literal as object and the other has a non-literal, the triple with the non-literal as object is first.
5. If both have a URI as object, the triple with the lexicographically smaller preprocessed object URI is first.
6. If the literal labels of the objects differ, the triple with the lexicographically smaller literal label is first.
7. If one of the object literals has a datatype identifier and the other does not, the triple without a datatype identifier is first.
8. If one of the object literals has a language identifier and the other does not, the triple without a language identifier is first.
9. The triple with the lexicographically smaller datatype or language identifier is first.

The lexicographic order is defined on strings of Unicode characters. If two strings have different characters at at least one position, the string with the smaller integer value at the first differing position is first. Otherwise, the shorter string is first.

After the triples have been sorted, a sequence of Unicode characters _s_ is built. For each triple, the serialization of its graph, its subject, its predicate, and its object are added to the end of _s_, in this order and with a newline character at the end of each of the four. The serialization of graph, subject, and predicate identifiers is simply their preprocessed URI string. Objects that consist of a URI are treated the same way. Literals with a datatype are serialized as a circumflex character (`^`) followed by the datatype URI, a blank space, and the escaped literal string. Literal strings are escaped by replacing `\` by `\\` and newline characters by `\n`. Literals with a language property are serialized as an at-sign `@` followed by the language string, a blank space, and the escaped literal string. Other literals are serialized as a hash sign `#` followed by the escaped literal string.

The actual computation of the hash data is identical to Algorithm F: a SHA-256 hash is generated for _s_ in UTF-8 encoding, two zero-bits are appended, and the result is transformed to Base64 notation.

