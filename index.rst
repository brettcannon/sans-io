Network protocols, sans I/O
===========================

This page is to provide a single location for people to reference when
looking for network protocol implementations written in Python that
perform **no** I/O (this means libraries that operate directly on
text or bytes; this excludes libraries that just abstract out I/O).


Why?
----

In a word: *reusability*.
By implementing network protocols without any I/O and instead
operating on bytes or text alone, libraries allow for reuse by other
code regardless of their I/O decisions.
In other words by leaving I/O out of the picture a network protocol
library allows itself to be used by both synchronous and asynchronous
I/O code.
And by not simply abstracting out the I/O it allows users of the
library to drive the network interactions themselves, not the network
protocol library itself; not forcing I/O code to have to conform to a
certain API provides the greatest flexibility for users of such
low-level details such as network protocols.
Working towards this unbinding of network protocols from I/O is very
important as the Python community migrates from synchronous I/O code
to using ``async``/``await`` for asynchronous I/O.

`Cory Benfield's PyCon US 2016 talk <https://www.youtube.com/watch?v=7cC3_jGwl_U>`_
provides a nice overview as to why designing protocol implementations
this way is important and the best way to do so going forward for the
Python community.

More Detail
-----------

For more detail, see the following documents:

.. toctree::
   :maxdepth: 2

   how-to-sans-io


Implementations
---------------

============================ ======================
Protocol                     Project
============================ ======================
`FastCGI`_                   fcgiproto_
`HTTP/2`_                    `hyper-h2`_
`HTTP/1.1`_                  h11_
`IRC`_                       ircproto_
`OAuth 1.0`_ & `OAuth 2.0`_  oauthlib_
`WebSocket`_                 wsproto_
`SOCKSv5`_                   socks5_
`SOCKSv4`_ & `SOCKSv5`_      siosocks_
`RFC 2217 (Serial over IP)`_ pyserial_
`EPICS Channel Access`_      caproto_
`FIX 4.0 - 5.0`_             simplefix_
`QUIC & HTTP/3`_             aioquic_
`Language Server Protocol`_  lsp_
`SMTP`_                      smtpproto_
`D-Bus`_                     jeepney_
`Thorlabs APT`_              thorlabs-apt-protocol_
============================ ======================

.. _FastCGI: https://htmlpreview.github.io/?https://github.com/FastCGI-Archives/FastCGI.com/blob/master/docs/FastCGI%20Specification.html
.. _fcgiproto: https://github.com/agronholm/fcgiproto
.. _HTTP/2: https://http2.github.io/
.. _hyper-h2: https://github.com/python-hyper/hyper-h2
.. _HTTP/1.1: https://tools.ietf.org/html/rfc7230
.. _h11: https://github.com/python-hyper/h11
.. _IRC: https://tools.ietf.org/html/rfc2812
.. _ircproto: https://github.com/agronholm/ircproto
.. _OAuth 1.0: https://tools.ietf.org/html/rfc5849
.. _OAuth 2.0: https://tools.ietf.org/html/rfc6749
.. _oauthlib: https://github.com/idan/oauthlib
.. _WebSocket: http://tools.ietf.org/html/rfc6455
.. _wsproto: https://github.com/jeamland/wsproto
.. _SOCKSv4: https://ftp.icm.edu.pl/packages/socks/socks4/SOCKS4.protocol
.. _SOCKSv5: https://tools.ietf.org/html/rfc1928
.. _socks5: https://github.com/mike820324/socks5
.. _siosocks: https://github.com/pohmelie/siosocks
.. _RFC 2217 (Serial over IP): https://tools.ietf.org/html/rfc2217
.. _pyserial: https://pythonhosted.org/pyserial/pyserial_api.html#serial.rfc2217.PortManager
.. _EPICS Channel Access: https://epics.anl.gov/base/R3-16/0-docs/CAproto/index.html
.. _caproto: https://nsls-ii.github.io/caproto
.. _FIX 4.0 - 5.0: https://www.fixtrading.org/standards/fix-5-0-sp-2/
.. _simplefix: https://github.com/da4089/simplefix
.. _`QUIC & HTTP/3`: https://quicwg.org/
.. _aioquic: https://github.com/aiortc/aioquic
.. _`Language Server Protocol`: https://microsoft.github.io/language-server-protocol/
.. _lsp: https://github.com/WindSoilder/lsp
.. _SMTP: https://tools.ietf.org/html/rfc5321
.. _smtpproto: https://github.com/agronholm/smtpproto
.. _`D-Bus`: https://www.freedesktop.org/wiki/Software/dbus/
.. _jeepney: https://gitlab.com/takluyver/jeepney/\
.. _`Thorlabs APT`: https://www.thorlabs.com/newgrouppage9.cfm?objectgroup_id=9019
.. _thorlabs-apt-protocol: https://gitlab.com/yaq/thorlabs-apt-protocol

Libraries
---------

There are also some libraries that help to implement network protocols without
performing any I/O:

  * `ohneio`_ (network protocol parsing; `Getting started <https://ohneio.readthedocs.io/en/latest/getting_started.html#getting-started>`_)
  * gidgethub_ (GitHub API)
  * google-music-proto_ (Google Music API)

.. _ohneio: https://github.com/acatton/ohneio
.. _gidgethub: https://gidgethub.readthedocs.io/
.. _google-music-proto: https://google-music-proto.readthedocs.io/
