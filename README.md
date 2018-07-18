# Goldilocks library
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fotrv4%2Flibgoldilocks.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fotrv4%2Flibgoldilocks?ref=badge_shield)


The Goldilocks library is a production library, providing the ed448 elliptic
curve. It is based on Mike Hamburg's libdecaf library:
[ed448goldilocks](https://sourceforge.net/p/ed448goldilocks/code/ci/master/tree/).

This library is intended for developers who have experience with cryptography.
It doesn't (yet?) include documentation on how to use digital signatures or key
exchange securely. Consult your local cryptographer for advice.

## Building the library

Before you try to build libgoldilocks, verify you have installed:

* [autoconf](https://www.gnu.org/software/autoconf/autoconf.html)
* [valgrind](http://valgrind.org/). At least, version 3.12.0

To generate project configuration:

```
$ autoreconf --install
```

To configure the project:

```
$ ./configure
```

To build and install:

```
$ make && make install
```

To run the tests:

```
$ make test
```

## Using the library

* To run the python wrapper: `python setup.py install --user`
* To generate the code based on the curve data, per field and the decaf tables:
  `make gen-code`
* To generate the code based on the curve data and per field:
  `make gen-code-static`
* To test the internal test programs: `make test`
* To generate the main decaf library and its symlinks: `make lib`

## Mailing lists

Because this is new software, please expect it to have bugs, perhaps even
critical security bugs.  If you are using it, please sign up for updates on
Mike Hamburg's library (note here that any update that we consider useful for
this library from libdecaf will be incorporated):

* Security-critical announcements (very low volume):
    decaf-security@googlegroups.com, join at https://groups.google.com/forum/#!forum/decaf-security
* New version announcements (low volume):
    decaf-announce@googlegroups.com, join at https://groups.google.com/forum/#!forum/decaf-annonuce
* Library discussion (potentially more volume):
    decaf-discuss@googlegroups.com, join at https://groups.google.com/forum/#!forum/decaf-discuss

## General elliptic curve operations.

This is a multi-purpose elliptic curve library. There is a C library, and a set
of C++ wrapper headers. The C++ code consists entirely of inline calls, and has
no compiled component.

The library implements a fairly complete suite of operations on
ed448-Goldilocks:

* Point and scalar serialization and deserialization.
* Point addition, subtraction, doubling, and equality.
* Point multiplication by scalars.  Accelerated double- and dual-scalar multiply.
* Scalar addition, subtraction, multiplication, division, and equality.
* Construction of precomputed tables from points. Precomputed scalarmul.
* Hashing to the curve with an Elligator variant. Inverse of elligator for
  steganography. These are useful for advanced protocols such as
  password-authenticated key exchange (PAKE) and verifiable random functions
  (VRFs).

Internally, the library uses twisted Edwards curves with the "decaf" and
"ristretto" technique to remove the curve's cofactor of 4. The upshot is that
systems using the "decaf" interface will be using a prime-order group, which
mitigates one of the few disadvantages of Edwards curves. However, this means
that it is not able to implement systems which care about cofactor information.

The goal of this library is not only to follow best practices, but to make it
easier for clients of the library to follow best practices. With a few
well-marked exceptions, the functions in this library should be strongly
constant-time: they do not allow secret data to flow to array indices, nor to
control decisions except for a final failure check.  Furthermore, the C++
wrapping uses RAII to automatically clear sensitive data, and has interfaces
designed to prevent certain mistakes.

## CFRG cryptosystems.

The library additionally supports the two cryptosystem defined by the Crypto
Forum Research Group (CFRG): the X448 Diffie-Hellman functions (RFC 7748), and
the EdDSA signature scheme (RFC 8032). Future versions might support additional
operations on these curves, such as precomputed signature verification.

## Symmetric crypto and hashing

The Goldilocks library doesn't implement much symmetric crypto, but it does
contain the hash functions required by the CFRG cryptosystems: SHA-3 and SHAKE.

## Internals

The "decaf" technique is described in https://eprint.iacr.org/2015/673 While the
title of that paper is "removing cofactors through point compression", it might
be more accurate to say "through quotients and isogenies". The internal
representation of points is as "even" elements of a twisted Edwards curve with
a=-1.  Using this subgroup removes a factor of 2 from the cofactor. The
remaining factor of 2 or 4 is removed with a quotient group: any two points
which differ by an element of the 2- or 4-torsion subgroup are considered equal
to each other.

When a point is written out to wire format, it is converted (by isogeny) to a
Jacobi quartic curve, which is halfway between an Edwards curve and a Montgomery
curve. One of the 4 or 8 equivalent points on the Jacobi quartic is chosen (it
is "distinguished" according to certain criteria, such as having a positive
x-coordinate). The x-coordinate of this point is written out. The y-coordinate
is not written out, but the decoder knows which of the two possible
y-coordinates is correct because of the distinguishing rules.
See the paper for more details.

## Licensing

See LICENSE.txt.


[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fotrv4%2Flibgoldilocks.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fotrv4%2Flibgoldilocks?ref=badge_large)

## Caveats

As mentioned in the license, there is absolutely NO WARRANTY on any of this
code. This code might well have security-critical bugs despite best efforts.

It has been attempted to protect against timing attacks and invalid point
attacks, but as of yet there is no attempt to protect against power analysis.