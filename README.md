Hash-URI Specification
======================

This document is **work in progress**. It contains the specification of the
hash-URI approach.


Implementations
---------------

There are currently three (partial) implementations:

- hashuri-java: https://github.com/tkuhn/hashuri-java
- hashuri-perl: https://github.com/tkuhn/hashuri-perl
- hashuri-python: https://github.com/tkuhn/hashuri-python


Basics
------

Generally, hash-URIs are URIs that contain a certain kind of hash value. This
hash value is encoded in Base64 notation with some common modifications for
making it safe to use in URIs and filenames:

**Definition.**
Every character that is a standard ASCII letter (A-Z or a-z), a digit (0-9), a
hyphen (-), or an underscore ( _ ) is called a _Base64 character_, representing
in this order the numbers from 0 to 63. There are no other Base64 characters.

As everybody who has access to the respective domain is free to define and use
URIs at will, we can only be sure that a certain URI is a hash-URI once we
have found and verified a content that matches the hash. For that reason, we
need to introduce the concept of a _potential hash-URI_:

**Definition.**
If the last 25 characters of a URI are all Base64 characters, then this URI
is a _potential hash-URI_.

Our concrete proposals only generate URIs with exactly 45 trailing Base64
characters, but we keep the definition open for future additions and
extensions. Such character sequences consist of two parts:

**Definition.**
The two characters immediately following the last non-Base64 character of a
potential hash-URI are called _identifier part_. The sequence of characters
following the identifier part is called _data part_, which is identical to or
contains a _hash part_.

The first character of the _identifier part_ specifies the type of content
and therefore the type of algorithm; the second character is a version number
of the algorithm. The main content of the _data part_ is the hash value, but
it can also contain other information such as parameters and sub-types. Its
concrete structure depends on the algorithm. With these ingredients,
hash-URIs can be _verified_:

**Definition.**
Given a potential hash-URI and a digital artifact, if the identifier part
refers to an algorithm that returns a hash value for the digital artifact
that is identical to the one encoded in the hash part, then the potential
hash-URI is a _verified hash-URI_ and the digital artifact is its _verified
resource_.

We can append a file extension like `.txt` or `.trig` to a hash-URI. The
resulting URIs are technically no hash-URIs anymore, but it is easy to strip
the extension and get the respective hash-URI from it.

...
