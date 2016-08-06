Network protocols, sans I/O
===========================

This page is to provide a single location for people to reference when
looking for network protocol implementations written in Python that
perform no I/O.


Why?
----

In a word: reusability. By implementing network protocols without any
I/O it allows the implementing library to be reused by any other
library or framework regardless of the type of I/O they do. I.e. by
leaving out I/O, a network protocol library can be used by either
synchronous or asynchronous I/O code. This can also extend to other
I/O-related areas such as files, but historically network protocol
implementations have (unnecessarily) been tightly bound to a specific
form of I/O.

`Cory Benfield's PyCon US 2016 talk <https://www.youtube.com/watch?v=7cC3_jGwl_U>`_
provides a nice overview as to why designing protocol implementations
this way is important and the best way to do so.


Implementations
---------------

=========== =======
Protocol    Project
=========== =======
`HTTP/2`_   `hyper-h2`_
`HTTP/1.1`_ h11_
=========== =======

.. _HTTP/2: https://http2.github.io/
.. _hyper-h2: https://github.com/python-hyper/hyper-h2
.. _HTTP/1.1: https://tools.ietf.org/html/rfc7230
.. _h11: https://github.com/njsmith/h11
