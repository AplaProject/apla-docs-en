######################################################################
Synchronization Monitoring Tool
######################################################################

A special tool that allows to verify that the databases on specified nodes are synchronized.
The tool can work as a daemon process or can be launched to perform a one-time check. The tool's operation principle is based on the following:
Each block contains the hash of all changes made by all transactions in the block. The specified nodes are requested to provide the ID of their last common block. A block with this ID is then requested from all nodes, and the aforementioned hash is compared. If the hash differs, a synchronization error message is sent to the email address specified in the command.

***********************************************************************
Location
***********************************************************************
tools/desync_monitor/

***********************************************************************
Flags
***********************************************************************
The following flags can be used from the command prompt:

* **confPath** - path to the configuration file. By default – config.toml.
* **nodesList** - a list of nodes to request blocks from, separated by commas. By default – 127.0.0.1:7079.
* **daemonMode** - launch as a daemon process; should be used in case the verification is required every N seconds. This flag is set to false by default.
* **queryingPeriod** - if the tool is launched as a daemon process, this parameter sets the time interval between checks (in seconds). By default – 1 second.
* **alertMessageTo** - an email address to which the synchronization error alerts will be sent. By default – alert@apla.io.
* **alertMessageSubj** - message subject to put in the alert message. By default – problem  with nodes synchronization.
* **alertMessageFrom** - address from which the message will be sent. By default – monitor@apla.io.
* **smtpHost** - SMTP server, which will be used to send the email message. By default – "".
* **smtpPort** - SMTP server port, which will be used to send the email message. By default – 25.
* **smtpUsername** - SMTP server username. By default – "".
* **smtpPassword** - SMTP server password. By default – "".

***********************************************************************
Configuration
***********************************************************************
The tool uses a configuration file in the toml format. By default, it looks for the config.toml file in the folder from which the binary was launched. The file path can be changed using the configPath flag.

Configuration File Example

.. code::

        nodes_list = ["http://127.0.0.1:7079", "http://127.0.0.1:7002"]

        [daemon]
        daemon_mode = false
        querying_period = 1

        [alert_message]
        to = "genesis@genesis.space"
        subject = "problem with genesis nodes"
        from = "monitor@genesis.space"

        [smtp]
        host = ""
        port = 25
        username = ""
        password = ""

* **nodes_list** - list of nodes (hosts) to request information from

==============================================================
[daemon]
==============================================================
Daemon mode configuration.

* **daemon_mode** - tells the tool to work as a daemon process and perform synchronization checks.
* **querying_period** - time interval between synchronization checks.

==============================================================
[alert_message]
==============================================================
Alert message parameters.

* **to** - address to which the synchronization error alert messages will be sent.
* **subject** - message subject.
* **from** - sender.

================================================================
[smtp]
================================================================
Parameters of the SMTP server, which will be used to send synchronization error messages.

* **host** - SMTP server, which will be used to send the email messages.
* **port** - SMTP server port, which will be used to send the email messages.
* **username** - SMTP server username.
* **password** - SMTP server password. 

