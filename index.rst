Network protocols, sans I/O
===========================

This page is to provide a single location for people to reference when
looking for network protocol implementations written in Python that
perform **no** I/O (this means libraries that abstract out I/O are
also excluded).


Why?
----

In a word: *reusability*.
By implementing network protocols without any I/O and instead
operating on bytes alone, libraries allow for reuse by other code
regardless of their I/O decisions.
In other words by leaving I/O out of the picture a network protocol
library allows itself to be used by both synchronous and asynchronous
I/O code.
Working towards this unbinding of network protocols from I/O is very
important as the Python community migrates from synchronous I/O code
to using ``async``/``await`` for asynchronous I/O.

`Cory Benfield's PyCon US 2016 talk <https://www.youtube.com/watch?v=7cC3_jGwl_U>`_
provides a nice overview as to why designing protocol implementations
this way is important and the best way to do so going forward for the
Python community.


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
