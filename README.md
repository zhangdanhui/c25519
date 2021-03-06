
Wei25519, Curve25519 and Ed25519 for low-memory systems
=============================================

> Original by Daniel Beer <dlbeer@gmail.com> 25 Apr 2014, updated 03 Oct 2017
> Modified by Nikolas Rösener <nroesener@uni-bremen.de> 01 Aug 2018

This package contains portable public-domain implementations of Daniel
J. Bernstein's Curve25519[^1] Diffie-Hellman function, and of the
Ed25519 signature system[^2]. Furthermore it includes transformations
between the birationally equivalent curves Wei25519, Curve25519 and Ed25519.
These transformations are used by an implementation on ECDSA that performs
operations on the Ed25519 curve.

The memory consumption is low enough that
they could be reasonably considered for most microcontroller
applications. In particular, Curve25519 scalar multiplication uses less
than half a kB of peak stack usage.

All functions are implemented in a way which yields constant execution
time with respect to secret data.

The package is divided into the following separable subsystems:

``f25519``

  ~ Constant-time field arithmetic on integers modulo 2^255-19. Elements
    are represented as 32-byte little-endian integers.

``c25519``

  ~ The Curve25519 Diffie-Hellman function. The secret and public key
    formats are compatible with NaCl.

``ed25519``

  ~ Arithmetic of points of the Edwards-curve equivalent of Curve25519.

``morph25519``

  ~ Conversion between short Weierstrass (wei25519), Montgomery (c25519)
    and Edwards (ed25519) coordinates. This can be used, for example,
    to compute the Curve25519 Diffie-Hellman exchange in Edwards
    coordinates or to perform ECDSA with Ed25519.

``fprime``

  ~ Constant-time field arithmetic on integers modulo arbitrary primes
    (up to a fixed, but configurable, size). This is much slower than
    f25519, which is optimized to take advantage of the sparse form of
    2^255-19.

``ecdsa``

  ~ An implementation of ECDSA_Wei25519 that performs scalar multiplications
    using the ed25519 back end.

``sha512``

  ~ A simple implementation of the SHA-512 hash function.

``edsign``

  ~ The Ed25519 signature system. The key and signature formats are
    compatible with the SUPERCOP reference implementation, and it produces
    identical signatures.

To build and test the package, type:

    make test

You can find usage examples for each module in the form of a test.
The API for each routine is documented in its .h file.

The ed25519_sign_test and ed25519_verify_test routines, that demonstrate
interoperability with test vectors published on ed25519.cr.yp.to, were
contributed by Larry Doolittle.

No functions are provided for generating secret keys. To do this,
generate a random 32-byte string using a secure random number generator.
Then, call ``c25519_prepare`` (or ``ed25519_prepare``) to clamp the
lower and upper bits as required by the specification.

To generate a public key, scalar-multiply the base point of the curve by
the secret key. To complete a Diffie-Hellman exchange, scalar-multiply
the other party's public key by your own secret key. The resulting point
should then be hashed to produce a shared secret. The hashing is
important, because the set of X-coordinates produced by scalar
multiplication is a strict subset of possible 32-byte strings. In
particular, the MSB will always be zero. Hashing removes the individual
bits' bias. In NaCl, the hash function used for this purpose is
HSalsa20, with an all-zero input field, and the curve point used as a
key.

These functions use a compact representation for field elements, and as
a result, they use far less stack space for computation than other
implementations. You should check the usage for your target device, but
when compiled for an ATmega128 with AVR-GCC 4.7.2 (-O1), stack usage
figures for key functions are:

    Func                                 Cost    Frame   Height
    ------------------------------------------------------------------------
    > edsign_verify                      1078      364        6
    > edsign_sign                        1050      336        6
    > edsign_sec_to_pub                   786       72        6
    > c25519_smult                        462      280        3

These figures may be improved further by CPU and compiler specific
optimizations.

License
-------

This modified version of C25519 is licensed under the MIT license.

The [original C25519](https://www.dlbeer.co.nz/oss/c25519.html) is in the public domain.


References
----------

[^1]: Daniel J. Bernstein (2006) "Curve25519: New Diffie-Hellman speed
records". Document ID: 4230efdfa673480fc079449d90f322c0. URL:
[http://cr.yp.to/ecdh/curve25519-20060209.pdf]. Date: 2006.02.09.

[^2]: Daniel J. Bernstein, Niels Duif, Tanja Lange, Peter Schwabe,
Bo-Yin Yang. (2012) "High-speed high-security signatures." Journal of
Cryptographic Engineering 2 (2012), 77-89. Document ID:
a1a62a2f76d23f65d622484ddd09caf8. URL:
[http://cr.yp.to/papers.html#ed25519]. Date: 2011.09.26.
