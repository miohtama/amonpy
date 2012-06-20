==================
amonpy
==================

Amonpy is the Python client for Amon

===============
 Installation
===============


1. Install the package with ``pip install amonpy`` or alternatively you can
download the tarball and run ``python setup.py install``


===============
 Configuration
===============


::

    import amonpy
    amonpy.config.address = 'http://amonhost:port'

    amonpy.config.protocol = 'http|zeromq'


=========
 Usage
=========

You can use amopy module to send log messages or exceptions to the Amon server.

::

    import amonpy

    amonpy.log(message)
    amonpy.log(message, ['list', 'of', 'tags'])


    data = {
        'exception_class': '',
        'url': '',
        'backtrace': '',
        'enviroment': '',
        'data': ''

    }

    amonpy.exception(data)

========================================
 More complex usage example (Amon Plus)
========================================

Here is an example script how to log custom errors to Amon+ service.
The script monitors a certain file and if this file is gone or not have been
refreshed for a while then an exception is logged.

* All communication is done over HTTP. Amon and Amon+ expose a JSON based loggin API

* **Amon+ requires application key** for all log messages. Create application
  key in Amon+ web interface and put it into the script

* To use this script you can simply drop it to ``/etc/cron.daily`` folder
  on your server where the log file must be monitored.

* In Amon+ you can connect this log message to an email alert, based on tags, and thus have incoming
  message when things go bad

Example code::

    #!/usr/bin/python
    """

        A simple Amon script to monitor whether a file has been updated recently.

    """

    import os
    import amonpy
    import time
    import socket
    import sys

    # The script identifies itself as server name + script name pair in log messages
    MY_NAME = socket.gethostname() + ":" + os.path.abspath(os.path.join(os.getcwd(), sys.argv[0]))

    # Which tags we set on our log entry
    # Later can be used to configurae alerts in amon+
    TAGS = ["error"]

    FILENAME = "/var/log/serious-log-file.log"

    #: One hour
    THRESHOLD = 3600

    # amonpy connects to the server via http/zeromg and application key secret
    amonpy.config.address = 'http://your.amonplus.server.com:2464'
    amonpy.config.application_key = "xxxxx"


    def log_error(message):
        """
        Send message to amon logs. stamped with info where it is coming from
        """
        amonpy.log("%s: %s" % (MY_NAME, message), TAGS)
        sys.exit(message)  # Terminate the script with output to terminal


    def main():
        """
        Check if file last modification date is too old and log an error in Amon.
        """
        if not os.path.exists(FILENAME):
            log_error("File is gone: %s" % FILENAME)

        modification_time = os.path.getmtime(FILENAME)

        age = time.time() - modification_time
        if age > THRESHOLD * 60:
            log_error("File is not up-to-date: %s" % FILENAME)


    main()


================
 Django
================

Using amonpy in Django is exactly the same as in every other python library. You can customize the config options
by adding them somewhere in `settings.py`

::

    # in settings.py
    import amonpy
    amon.config.address = 'http://amonhost:port'


To capture and log exceptions

::

   MIDDLEWARE_CLASSES = (
    .....
    'amonpy.adapters.DjangoExceptionMiddleware'
    )

===============
 Requirements
===============


Python 2.6+

requests

