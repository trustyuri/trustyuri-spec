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
hyphen (-), or an underscore (_) is called a _Base64 character_, representing
in this order the numbers from 0 to 63. There are no other Base64 characters.

...
