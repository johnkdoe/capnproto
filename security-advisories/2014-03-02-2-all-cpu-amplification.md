Problem
=======

CPU usage amplification attack.

Discovered by
=============

Ben Laurie <ben@links.org>

Announced
=========

2014-03-02

Impact
======

- Remotely cause a peer to use excessive CPU time and other resoucres to
  process a very small message, possibly enabling a DoS attack.

Fixed in
========

- git commit [b9da55c6b003cca89b25f19d52749461215292d1][0]
- release 0.5.1.1:
  - Unix: https://capnproto.org/capnproto-c++-0.5.1.1.tar.gz
  - Windows: https://capnproto.org/capnproto-c++-win32-0.5.1.1.zip
- release 0.4.1.1:
  - Unix: https://capnproto.org/capnproto-c++-0.4.1.1.tar.gz
- release 0.6 (future)

[0]: https://github.com/sandstorm-io/capnproto/commit/b9da55c6b003cca89b25f19d52749461215292d1

Details
=======

The Cap'n Proto list pointer format allows encoding a list whose elements are
claimed each to have a size of zero. Such a list could claim to have up to
2^29-1 elements while only taking 8 or 16 bytes on the wire. The receiving
application may expect, say, a list of structs. A zero-size struct is a
perfectly legal (and, in fact, canonical) encoding for a struct whose fields
are all set to their default values. Therefore, the application may notice
nothing wrong and proceed to iterate through and handle each element in the
list, potentially taking a lot of time and resources to do so.

Note that this kind of vulnerability is very common in other systems. Any
system which accepts compressed input can allow an attacker to deliver an
arbitrarily large uncompressed message using very little compressed bandwidth.
Applications should do their own validation to ensure that lists and blobs
inside a message have reasonable size. However, Cap'n Proto takes the
philosophy that any security mistake that is likely to be common in
naively-written application code is in fact a bug in Cap'n Proto -- we should
provide defenses so that the application developer doesn't have to.

To fix the problem, this change institutes the policy that, for the purpose of
the "message traversal limit", a list of zero-sized elements will be counted as
if each element were instead one word wide. The message traversal limit is an
existing anti-amplification measure implemented by Cap'n Proto; see:

https://capnproto.org/encoding.html#amplification-attack

Preventative measures
=====================

This problem was discovered through fuzz testing using American Fuzzy Lop,
wich identified the problem as a "hang", although in fact the test case just
took a very long time to complete. We are incorporating testing with AFL into
our release process going forward.
