Backend configuration file
##########################

This section describes parameters in the backend configuration file.


About the backend configuration file
====================================

Backend configuration file defines configuration for a |platform| node.


Location
========

This file is located in the directory with the backend binary and is named ``config.toml``.

.. todo::

    How this works with workDir?

    This file is created on the first run?


Sections
========

.. todo::

    Add [Autoupdate]

    Running mode parameter?

The configuration file has several sections:

.. describe:: default section

    This section defines general parameters.

.. describe:: [TCPServer]

    This section defines parameters for TCPServer.

    TCPServer supports the network interaction between nodes.

.. describe:: [HTTP]

    This section defines parameters for HTTPServer.

    HTTPServer provides REST API.

.. describe:: [DB]

    This section defines parameters for the node's database engine, PostgreSQL.

.. describe:: [StatsD]

    This section defines parameters for the node operation metrics collector, StatsD.

    .. todo::

        Explain this better.

.. describe:: [Centrifugo]

    This section defines parameters for the notifications service, Centrifugo.


Configuration file example
==========================

.. code-block:: js

    LogLevel = "ERROR"
    LogFileName = ""
    InstallType = "PRIVATE_NET"
    NodeStateID = "*"
    TestMode = false
    StartDaemons = ""
    KeyID = -3785392309674179665
    EcosystemID = 0
    BadBlocks = ""
    FirstLoadBlockchainURL = ""
    FirstLoadBlockchain = ""
    MaxPageGenerationTime = 0
    WorkDir = "files"
    PrivateDir = "files"
    RunningMode = "privateBlockchain"

    [TCPServer]
      Host = "127.0.0.1"
      Port = 7078

    [HTTP]
      Host = "127.0.0.1"
      Port = 7079

    [DB]
      Name = "egaas"
      Host = "localhost"
      Port = 5432
      User = "egaas"
      Password = "egaas"

    [StatsD]
      Name = "apla"
      Host = "127.0.0.1"
      Port = 8125

    [Centrifugo]
      Secret = ""
      URL = ""

    [Autoupdate]
      ServerAddress = "http://127.0.0.1:12345"
      PublicKeyPath = "update.pub"

