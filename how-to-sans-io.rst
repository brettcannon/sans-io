Writing I/O-Free (Sans-I/O) Protocol Implementations
====================================================

This informational document is an attempt to make the case for implementing
network protocols without writing I/O, and to provide concrete assistance and
instructions for doing so in Python.

What Is An I/O-Free Protocol Implementation?
--------------------------------------------

An I/O-free protocol implementation (colloquially referred to as a "sans-IO"
implementation) is an implementation of a network protocol that contains no
code that does any form of network I/O or any form of asynchronous flow
control. Put another way, a sans-IO protocol implementation is one that is
defined entirely in terms of synchronous functions returning synchronous
results, and that does not block or wait for any form of I/O. Examples of this
kind of application can be found on :doc:`the landing page <index>`.

Such a protocol implementation is, at the surface level, not very useful. After
all, it's not very helpful to write a network protocol implementation that
doesn't speak to the network! However, it turns out that writing protocol
implementations in this manner provides a number of extremely useful benefits
to the wider software ecosystem, as well as to the quality of your own code.

In the rest of this document we'll outline what those benefits are, and also
describe some of the techniques that are commonly employed to write protocol
stacks in this manner.

.. note:: It should be noted that the sans-IO implementation style is a
          specific facet of several broader software design best practices. In
          particular, good comparisons can be made to Bob Martin's `Clean
          Architecture`_, to well-realised Model-View-Controller applications,
          and to the broader software design principle of separation of
          concerns. While all of these are good patterns worth being understood
          in their own right, it is often helpful to discuss these patterns as
          they apply to a specific problem domain: this is exactly what
          sans-IO is all about.


.. _why-bother:

Why Write I/O-Free Protocol Implementations?
--------------------------------------------

Writing sans-IO protocol implementations provides a number of useful benefits,
both to the implemenation itself and to the wider software ecosystem. We'll
discuss these in turn.


.. _simple-testable-correct:

Simplicity, Testability, and Correctness
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Simplicity
^^^^^^^^^^

On a self-serving level, writing a protocol implementation containing no I/O
makes writing a high-quality implementation substantially easier. It is no
secret that network I/O is particularly prone to a wide variety of unexpected
failure modes that can occur at almost any time, even in the simplest cases.
When the protocol implementation no longer drives its own I/O, but instead has
data passed to and from it using in-memory buffers of bytes, the space of
possible failures is *subtantially* decreased.

Given that writing to and reading from memory never fails, the implementation
has a much simpler time managing its data. The only concern the implementation
now has is around buffering incomplete data (that is, data that cannot yet be
fully parsed). Buffering data is generally a fairly simple concern compared to
dealing with sockets, and is usually required *anyway* (as anyone who has
encountered a short ``recv`` response can testify). All of this means that the
implementation has a much simpler time managing its input and output data.

This simplicity also ends up stretching to flow control within the
implementation as well. Given that the implementation now spends all its time
reading from and writing to byte buffers, it is never possible for the
implementation to block or need to stop, except when it runs out of room in its
buffers. There is never any requirement to pause computation in order to wait
for more data to arrive or for data to be sent, and it is relatively easy to
structure your implementation so it does not have to be safe to re-entrancy.
All of this has the effect of vastly limiting the number of possible flows of
control through the implementation, which drastically helps your ability to
understand the implementation.

Testability
^^^^^^^^^^^

From simplicity flows testability. Because there are far fewer possible flows
of control through the program, it is much, *much* easier to hit 100% branch
coverage from your tests, because there are far fewer possible entry points and
locations in the state space.

Additionally, because the code under test deals only with buffers of bytes for
both its input and output, the test code no longer needs to pretend to manage
sockets. It becomes extremely simple to write tests that validate the
correctness of the implementation, because those tests simply shove sequences
of bytes in and validate the sequences of bytes that come out. As there is no
I/O involved in the implementation (even mocked-out I/O), there is no risk of
non-determinism in the tests. Either the test passes or it does not: it will
never be a "flaky" test that requires a difficult-to-reproduce test
environment.

It also becomes possible to make certain assertions about the code under test
that are generally impossible when the implementation is making I/O calls. For
example, it is not unreasonable to make the assertion that the tests should be
able to attain 100% code and branch coverage using *only* the public APIs: that
is, only by passing in byte sequences or calling public API functions. If a
branch of code is not reachable by making those calls, it is quite literally
impossible to reach without the user monkeypatching your implementation. Such
code is entirely unnecessary, and can be safely removed entirely.

Finally, because you have no mocking and no actual I/O, your tests become
extremely fast and safe to run in parallel. It is also pretty easy to provide
test fixtures and combinatorial expansion of tests, which makes it possible to
provide thousands of test cases. It is also very easy to write new test cases
to reproduce bugs and to prevent regressions.

Achieving all of this when the implementation uses asynchronous flow control
primitives or actual I/O is much harder. To validate the correctness of the
implementation in the face of all possible I/O errors is extremely difficult,
and requires essentially triggering all of these possible I/O errors at all
locations they could possibly occur in your code, and then validating that the
implementation handles that error as you'd expect. A similar notion is true for
asynchronous flow control primitives: these need to be fired in all possible
orders to make the same guarantees about correctness.

Correctness
^^^^^^^^^^^

From simplicity and testability flows correctness. It is possible to develop an
extremely high degree of confidence about the correctness of the protocol
implementation due to the relative simplicity of the implementation and due to
the extremely high quantity of test coverage that you can achieve.

While this level of correctness may not be achievable by the application that
uses the implementation, it is nonetheless extremely helpful to know that the
protocol implementation is highly likely to behave in a reproducible,
consistent way, and to generate well-formed output in all cases.

This is particularly valuable with network protocols, as it allows you to
drastically increase the probability that your implementation will be able to
interoperate with other implementations in the wild. And in the event that you
have an interoperability problem, it will be easy to reproduce that problem in
test conditions and confirm whether it is your implementation or the other one
that has misunderstood the protocol.

Finally, from a truly selfish perspective, the more correct your implementation
is, the fewer bug reports you'll have to deal with from your users!


Reusability
~~~~~~~~~~~

The less selfish improvement that is obtained from writing sans-IO protocol
implementations is that they become *dramatically* more re-useable. The Python
ecosystem as it stands in 2016 contains a number of implementations of almost
every common network protocol, and to within a rounding error exactly none of
them share non-trivial protocol code.

This is an enormous amount of duplicated effort. Writing a protocol stack for
a relatively simple protocol is a decent amount of work, and writing one for
a complex protocol is an *extremely* substantial effort that can take hundreds
of person-hours. Duplicating this effort is a poor allocation of resources that
the open source and free software communities can `increasingly ill-afford`_.

While the duplication of effort is bad enough, we are also repeatedly writing
the same bugs. This is somewhat inevitable given the difficulty of producing a
correct I/O-based protocol implementation (see :ref:`simple-testable-correct`),
but it is also caused because these various implementations often have no
overlap in their development teams. This causes us to repeatedly stumble into
the same subtle issues without being able to share knowledge about them, let
alone share code to fix the problem. This leads to further multiplicative
inefficiencies.

There is obviously plenty of great reasons to write a competing implementation
for a network protocol: you may want to learn how the protocol works, or you
may believe that the current implementations have poor APIs or poor
correctness. However, many reimplementations do not occur for these reasons:
instead, they occur because all current implementations either bake their I/O
in or they bake their expected flow control mechanisms. For example, aiohttp
was not able to use httplib's parser, because httplib bakes its socket calls
into that parser, making it unsuitable for an asyncio environment.

By keeping async flow control and I/O out of your protocol implementation, it
provides the ability to use that implementation across all forms of flow
control. This means that the core of the protocol implementation is divorced
entirely from the way I/O is done or the way the API is designed. This provides
the Python community with huge advantages:

- people who want to experiment with simpler or better API designs can do so
  without needing to write a protocol implementation or being constrained by
  the pre-existing API designs.
- those who want to pursue unusual asynchronous flow control approaches (e.g.
  `curio`_) can obtain new implementations that are compatible with those new
  approaches with minimal effort and without needing to be an expert in all
  protocols.
- people with unusual or high-performance I/O requirements can take control of
  their own I/O code without needing to rewrite the entire protocol stack. For
  example, people wanting to write high-performance HTTP/2 implementations will
  want to architect their I/O around the `TCP_NOTSENT_LOWAT`_ socket option,
  which is not easily possible with most I/O-included implementations.

This also allows us to centralize our work. If all, or even most, Python
libraries center around the same small number of implementations of popular
protocols, that makes it possible for the best protocol experts in the Python
community to focus their efforts on fixing bugs and adding features to the core
protocol implementations, leading to a "rising tide lifts all boats" effect on
the community.


How To Write I/O-Free Protocol Implementations
----------------------------------------------

Assuming that :doc:`why-bother` has convinced you, the logical next question
is: how do you write a protocol implementation that does no I/O?

While each protocol is unique, there are several core design principles that
can be used to help provide the scaffolding for your sans-IO implementation.

.. _inputs-and-outputs:

Inputs and Outputs
~~~~~~~~~~~~~~~~~~

When it comes to network protocols, at a fundamental level they all consume and
produce byte sequences. For protocols implemented over TCP (or any
``SOCK_STREAM``-type socket), they use a byte stream. For protocols implemented
over UDP, or over any lower-level protocol than that (e.g. directly over IP),
they communicate in terms of datagrams, rather than byte streams.

For byte-stream based protocols, the protocol implementation can use a single
input buffer and a single output buffer. For input (that is, receiving data
from the network), the calling code is responsible for delivering code to the
implementation via a single input (often via a method called ``receive_bytes``,
or something similar). The implementation will then append these bytes to its
internal byte buffer. At this point, it can choose to either eagerly process
those bytes, or do so lazily at the behest of the calling code.

When it comes to generating output, a byte-stream based protocol has two
options. It can either write its bytes to an internal buffer and provide an API
for extracting bytes from that buffer, as done by `hyper-h2`_, or it can return
bytes directly when the calling code triggers events (more on this later), as
done by `h11`_. The distinction between these two choices is not enormously
important, as one can easily be transformed into the other, but using an
internal byte buffer is recommended if it is possible that the act of receiving
input bytes can cause output bytes to be produced: that is, if the protocol
implementation sometimes automatically responds to the peer without user input.

For datagram based protocols, it is usually important to preserve the datagram
boundaries. For this reason, while the general structure of the above points
remains the same, the inputs and outputs should be changed to consume and
return iterables of bytestrings. Each element in the iterable will correspond
to a single datagram.

Events
~~~~~~

The major abstraction used by most of the sans-IO protocol stacks is to
translate the bytes received from the network into "events". Essentially, this
abstraction defines a network protocol as a serialization mechanism for a
sequence of semantic "events" that can occur on that protocol.

In this abstraction model, both peers in a protocol emit and receive events.
In terms of receiving events, events can either be returned to the calling code
immediately whenever bytes are provided, or they can be lazily produced in
response to the calling code's request. Both approaches have their advantages
and disadvantages, and it doesn't matter enormously which is chosen.

When it comes to emitting events, there are several possible approaches, but
two are in active use. The first, and comfortably the most common, is to emit
events using function calls. For example, a HTTP implementation may have a
function call entitled ``send_headers`` which emits a bytestream that, if
received by the same implementation, would cause a ``RequestReceived`` event to
be emitted. This is the approach used by `hyper-h2`_.

However, an alternative approach is to have a single method that accepts
*events*, the same events that the implementation emits. This is the approach
used by `h11`_. This approach has the substantial advantage of symmetry of
input and output to the implementation, but the moderate disadvantage of being
a slightly uncomfortable programming approach for many developers.

Either approach works well.

Integrating With I/O
--------------------

At some point, of course, your sans-IO protocol implementation needs to be
joined to some actual I/O. There are two obvious possible goals when doing
this. The first is to provide a complete native-feeling API for the given I/O
model. The second is provide an implementation that can easily be swapped to
run in multiple I/O models. Each has different design requirements.

If you are designing for a full native-feeling API for a given I/O model (e.g.
Twisted or asyncio), you will want to buy entirely into that platform's
standard design patterns. You can liberally use flow control primitives and
the appropriate interfaces and I/O mechanisms to transfer data. This allows you
to build a module like, for example, `aiohttp`_ without having to reimplement
HTTP from the ground up. It also allows you to optimise for common use-cases,
and generally provide a no-friction interface.

Another possibility is to try as much as possible to push your I/O and flow
control primitives to the *edges* of the program or library, providing
integration points for multiple backends. This requires substantial care and
discipline, as it requires that your entire codebase be predicated around
sans-IO primitives except for a very tiny nucleus of code that uses the I/O and
flow control primitives of the given platform. This allows you to have a single
codebase that drops neatly into multiple I/O and flow control paradigms with
very little change, though at the cost of quite possibly not feeling very
native in some or all of them.

Acknowledgements
----------------

This document would not exist without the hard work of the excellent people
involved in the Python Async Special Interest Group, who have worked tirelessly
to build and extend the asynchronous programming paradigm in Python, as well as
the programming communities behind all of the asynchronous programming
frameworks that are used and loved by the Python community.



.. _Clean Architecture: https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html
.. _increasingly ill-afford: http://www.fordfoundation.org/library/reports-and-studies/roads-and-bridges-the-unseen-labor-behind-our-digital-infrastructure/
.. _curio: http://curio.readthedocs.io/en/latest/
.. _TCP_NOTSENT_LOWAT: https://lwn.net/Articles/560082/
.. _hyper-h2: https://github.com/python-hyper/hyper-h2
.. _h11: https://github.com/njsmith/h11
.. _aiohttp: http://aiohttp.readthedocs.io/en/stable/
