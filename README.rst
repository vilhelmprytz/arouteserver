.. DO NOT EDIT: this file is automatically created by /utils/build_doc

ARouteServer
============

|Documentation| |Build Status| |Unique test cases| |PYPI Version| |Python Versions| |Requirements|


A Python tool to automatically build (and test) feature-rich configurations for BGP route servers.

How it works
------------

#. Two YAML files provide *general policies* and *clients configurations* options:

   .. code:: yaml

      cfg:
        rs_as: 64496
        router_id: "192.0.2.2"
        filtering:
          irrdb:
            enforce_origin_in_as_set: True
            enforce_prefix_in_as_set: True
          rpki_bgp_origin_validation:
            enabled: True
            reject_invalid: True
            ...

   .. code:: yaml

      clients:
        - asn: 64511
          ip:
          - "192.0.2.11"
          - "2001:db8:1:1::11"
          irrdb:
            as_sets:
              - "RIPE::AS-FOO"
        ...

#. ARouteServer acquires external information to enrich them: i.e. `bgpq4`_/`bgpq3`_ for IRR data, `PeeringDB`_ for max-prefix limit and AS-SETs, RPKI ROAs, ...

#. `Jinja2`_ built-in templates are used to render the final route server's configuration file.

   Currently, **BIRD** (>= 1.6.3 up to 1.6.8), **BIRD v2** (starting from 2.0.7) and **OpenBGPD** (OpenBSD 6.1 up to 7.1 and also OpenBGPD Portable 6.5p1 up to 7.1p0) are supported, with almost `feature parity <https://arouteserver.readthedocs.io/en/latest/SUPPORTED_SPEAKERS.html#supported-features>`__ between them.

**Validation** and testing of the configurations generated with this tool are performed using the built-in **live tests** framework: `Docker`_ instances are used to simulate several scenarios and to validate the behaviour of the route server after configuring it with ARouteServer. More details on the `Live tests <https://arouteserver.readthedocs.io/en/latest/LIVETESTS.html>`__ section.

A Docker-based `playground <https://github.com/pierky/arouteserver/tree/master/tools/playground>`__ is available to experiment with the tool in a virtual IXP environment.

Also, a `Docker image <https://hub.docker.com/r/pierky/arouteserver>`__ is provided to start building rich and secure configurations in a couple of minutes.

.. _bgpq3: https://github.com/snar/bgpq3
.. _bgpq4: https://github.com/bgp/bgpq4
.. _PeeringDB: https://www.peeringdb.com/
.. _Jinja2: http://jinja.pocoo.org/
.. _Docker: https://www.docker.com/

Features
--------

- **Path hiding** mitigation techniques (`RFC7947`_ `section 2.3.1 <https://tools.ietf.org/html/rfc7947#section-2.3.1>`__).

- Basic filters (mostly enabled by default):

  - **NEXT_HOP** enforcement (strict / same AS - `RFC7948`_ `section 4.8 <https://tools.ietf.org/html/rfc7948#section-4.8>`__);
  - minimum and maximum IPv4/IPv6 **prefix length**;
  - maximum **AS_PATH length**;
  - reject **invalid AS_PATHs** (containing `private/invalid ASNs <http://mailman.nanog.org/pipermail/nanog/2016-June/086078.html>`_);
  - reject AS_PATHs containing **transit-free** or **never via route-servers** ASNs (using `PeeringDB info_never_via_route_servers attribute <https://github.com/peeringdb/peeringdb/issues/394>`__);
  - reject **bogons**;
  - **max-prefix limit** based on global or client-specific values or on **PeeringDB** data.

- Prefixes and origin ASNs validation (also in *tag-only* mode):

  - **IRR-based filters** (`RFC7948`_ `section 4.6.2 <https://tools.ietf.org/html/rfc7948#section-4.6.2>`__);
  - AS-SETs configured manually or fetched from PeeringDB;
  - support for **IRR sources** (RIPE::AS-FOO, RADB::AS-BAR);
  - **white lists** support;
  - extended dataset for filters generation:

    - RPKI **ROAs used as route objects**;
    - `Origin AS <https://mailman.nanog.org/pipermail/nanog/2017-December/093525.html>`__ from **ARIN Whois** database dump;
    - `NIC.BR Whois data <https://ripe76.ripe.net/presentations/43-RIPE76_IRR101_Job_Snijders.pdf>`_ (slide n. 26) from Registro.br;

  - **RPKI**-based filtering (BGP Prefix Origin Validation);

    - ROAs can be retrieved from publicly available JSON files or from a local validating cache.

- **Blackhole filtering** support:

  - optional **NEXT_HOP rewriting**;
  - signalling via BGP Communities (`BLACKHOLE <https://tools.ietf.org/html/rfc7999#section-5>`__ and custom communities);
  - client-by-client control over propagation.

- **Graceful shutdown** support:

  - honor the **GRACEFUL_SHUTDOWN** BGP community received from clients (`draft-ietf-grow-bgp-gshut-11 <https://tools.ietf.org/html/draft-ietf-grow-bgp-gshut-11>`_);
  - allow to perform a graceful shutdown of the route server itself.

- Control and informative communities:

  - prefix/origin ASN present/not present in **IRRDBs data**;
  - do (not) announce to any / **peer** / on **RTT basis**;
  - **prepend** to any / **peer** / on **RTT basis**;
  - add **NO_EXPORT** / **NO_ADVERTISE** to any / **peer**;
  - custom informational BGP communities.

- Optional session features on a client-by-client basis:

  - prepend route server ASN (`RFC7947`_ `section 2.2.2.1 <https://tools.ietf.org/html/rfc7947#section-2.2.2.1>`__);
  - active sessions;
  - **GTSM** (Generalized TTL Security Mechanism - `RFC5082`_);
  - **ADD-PATH** capability (`RFC7911`_).

- Automatic building of clients list:

  - `integration <https://arouteserver.readthedocs.io/en/latest/USAGE.html#ixp-manager-integration>`__ with **IXP-Manager**;
  - `fetch lists <https://arouteserver.readthedocs.io/en/latest/USAGE.html#automatic-clients>`__ from **PeeringDB** records and **Euro-IX member list JSON** files.

- **IX-F Member Export** JSON files `creation <https://arouteserver.readthedocs.io/en/latest/USAGE.html#ixf-member-export-command>`__.

- Related tools:

  - The `Playground <https://github.com/pierky/arouteserver/tree/master/tools/playground>`__, to experiment with the tool in a virtual IXP environment.

  - `Invalid routes reporter <https://arouteserver.readthedocs.io/en/latest/TOOLS.html#invalid-routes-reporter>`__, to log or report invalid routes and their reject reason.

A comprehensive list of features can be found within the comments of the distributed configuration file on `GitHub <https://github.com/pierky/arouteserver/blob/master/config.d/general.yml>`__ or on the `documentation web page <https://arouteserver.readthedocs.io/en/latest/GENERAL.html>`__.

More feature are already planned: see the `Future work <https://arouteserver.readthedocs.io/en/latest/FUTUREWORK.html>`__ section for more details.

.. _RFC7947: https://tools.ietf.org/html/rfc7947
.. _RFC7948: https://tools.ietf.org/html/rfc7948
.. _RFC5082: https://tools.ietf.org/html/rfc5082
.. _RFC7911: https://tools.ietf.org/html/rfc7911

Full documentation
------------------

Full documentation can be found on ReadTheDocs: https://arouteserver.readthedocs.org/

Presentations
-------------

- Euro-IX "Learn with us: ARouteServer tutorial", 28 July 2021: `video <https://www.youtube.com/watch?v=aiBeFs6xnYs>`__ (33:13)

- RIPE74, 10 May 2017, Connect Working Group: `video <https://ripe74.ripe.net/archives/video/87/>`__ (9:53), `slides <https://ripe74.ripe.net/presentations/22-RIPE74-ARouteServer.pdf>`__ (PDF)

- Salottino MIX, 30 May 2017: `slides <https://www.slideshare.net/PierCarloChiodi/salottino-mix-2017-arouteserver-ixp-automation-made-easy>`__

Mentions / endorsements:

- Job Snijders, LACNIC29, 3 May 2018: `slides <https://www.lacnic.net/innovaportal/file/2621/1/lacnic29_peering_tutorial.pdf>`__ (PDF)

- Anurag Bhatia, APNIC46, 12 September 2018: `video <https://www.youtube.com/watch?v=XfSNQbiR1cg&t=3140>`__, `slides <https://conference.apnic.net/46/assets/files/APNC402/Automate-your-IX-config.pdf>`__ (PDF)

- Claudio Jeker, RIPE Labs, 28 November 2018: `OpenBGPD - Adding Diversity to the Route Server Landscape <https://labs.ripe.net/Members/claudio_jeker/openbgpd-adding-diversity-to-route-server-landscape>`__.

Who is using ARouteServer?
--------------------------

- `BharatIX <https://www.bharatix.net/>`__, BIRD.

- `CATNIX <http://www.catnix.net/en/>`__, BIRD.

- `CHIX <https://chix.ch/>`__, BIRD and OpenBGPD.

- `CNX <http://cnx.net.kh/>`__, BIRD v2.

- `DO-IX <https://www.do-ix.net/>`__, BIRD.

- `EVIX <https://evix.org/>`__, BIRD.

- `FCIX <https://fcix.net/>`__, BIRD.

- `GAVLIX <https://gavlix.se/>`__.

- `IX Australia <https://www.ix.asn.au/>`__, BIRD v2.

- `IX-Denver <http://ix-denver.org/>`__, BIRD.

- `MBIX <http://www.mbix.ca/>`__, BIRD.

- `MIX <https://www.mix-it.net/>`__, BIRD.

- `Netnod <https://www.netnod.se/>`__, BIRD and GoBGP\ :sup:`1`\.

- `NIXI Mumbai (GPX) <https://nixi.in/>`__, BIRD.

- `NZIX <https://ix.nz/>`__, BIRD v2.

- `PIT-IX <https://pit-ix.net/>`__, BIRD.

- `QCIX <http://www.qcix.net/>`__, BIRD.

- `RO-CIX <https://roix.net/>`__, OpenBGPD.

- `SwissIX <https://www.swissix.ch/>`__, OpenBGPD.

- `Unmetered.Exchange <https://unmetered.exchange/>`__, BIRD.

- `VANIX <https://vanix.ca/>`__.

- `YEGIX <https://yegix.ca>`__, OpenBGPD.

- `YXEIX <http://yxeix.ca/>`__, BIRD.

- `YYCIX <https://yycix.ca>`__, OpenBGPD.

Are you using it? Do you want to be listed here? `Drop me a message <https://pierky.com/#contactme>`__!

\ :sup:`1`\: GoBGP configurations are generated using a fork of the project which is still WIP and that hopefully will be merged upstream in the future.

Bug? Issues? Support requests?
------------------------------

But also suggestions? New ideas?

Please create an `issue on GitHub <https://github.com/pierky/arouteserver/issues>`_ or `drop me a message <https://pierky.com/#contactme>`_.

A Slack channel is also available on the `network.toCode() <https://networktocode.herokuapp.com/>`__ community: **arouteserver**.

Author
------

Pier Carlo Chiodi - https://pierky.com

Blog: https://blog.pierky.com Twitter: `@pierky <https://twitter.com/pierky>`_

.. |Documentation| image:: https://readthedocs.org/projects/arouteserver/badge/?version=latest
    :target: https://arouteserver.readthedocs.org/en/latest/?badge=latest
.. |Build Status| image:: https://github.com/pierky/arouteserver/actions/workflows/cicd.yml/badge.svg?branch=master
    :target: https://github.com/pierky/arouteserver/actions/workflows/cicd.yml
.. |Unique test cases| image:: https://img.shields.io/badge/dynamic/json.svg?uri=https://raw.githubusercontent.com/pierky/arouteserver/master/tests/last.json&label=unique%20test%20cases&query=$.unique_test_cases&colorB=47C327
    :target: https://github.com/pierky/arouteserver/blob/master/tests/last
.. |PYPI Version| image:: https://img.shields.io/pypi/v/arouteserver.svg
    :target: https://pypi.python.org/pypi/arouteserver/
.. |Requirements| image:: https://requires.io/github/pierky/arouteserver/requirements.svg?branch=master
    :target: https://requires.io/github/pierky/arouteserver/requirements/?branch=master
    :alt: Requirements Status
.. |Python Versions| image:: https://img.shields.io/pypi/pyversions/arouteserver.svg
