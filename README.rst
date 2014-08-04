python-logstash
===============

Python logging handler for Logstash.
http://logstash.net/

Changelog
=========
0.4.1
  - Added TCP handler.
0.3.1
  - Added support for Python 3
0.2.2
  - Split Handler into Handler and Formatter classes
0.2.1
  - Added support for the new JSON schema in Logstash 1.2.x. See details in
    http://tobrunet.ch/2013/09/logstash-1-2-0-upgrade-notes-included/ and
    https://logstash.jira.com/browse/LOGSTASH-675

    - Added ``version`` parameter. Available values: 1 (Logstash 1.2.x version format), 0 - default (previous version).


Installation
============

Using pip::

  pip install python-logstash

Usage
=====

``LogstashHandler`` is a custom logging handler which sends Logstash messages using UDP.

For example::

  import logging
  import logstash

  test_logger = logging.getLogger('test_logger')
  test_logger.setLevel(logging.INFO)
  test_logger.addHandler(logstash.LogstashHandler('localhost', 5959, version=1))

  test_logger.info('Test logstash message.')
  test_logger.warning('Test logstash message.')
  test_logger.error('Test logstash message.')

  # Add tags, which can later be used in elastic-search + kibana (or other tools)
  d = {'tags': ['Testing', 'Documentation']}
  test_logger.info('Test logstash message and tags', extra=d)

  # Add custom fields for easier searching (with or without other fields - for example 'tags')
  extra = {'user_id': 1}
  test_logger.info('Test logstash fields', extra=extra)
  fields = {'test_field': True, 'tags': ['Testing']}
  test_logger.info('Test logstash fields', extra=fields)

When using ``extra`` field make sure you don't use reserved names. From `Python documentation <https://docs.python.org/2/library/logging.html>`_.
     | "The keys in the dictionary passed in extra should not clash with the keys used by the logging system. (See the `Formatter <https://docs.python.org/2/library/logging.html#logging.Formatter>`_ documentation for more information on which keys are used by the logging system.)"

Set up logstash so that it listens on port 5959 and parses the input as json.

For example::

    bin/logstash -e 'input { udp { port => 5959 codec => json } } output { stdout { codec => rubydebug } }'


Using with Django
=================

Modify your ``settings.py`` to integrate ``python-logstash`` with Django's logging::

  LOGGING = {
    ...
    'handlers': {
        'logstash': {
            'level': 'DEBUG',
            'class': 'logstash.LogstashHandler',
            'host': 'localhost',
            'port': 5959,
            'version': 1,
            'message_type': 'django',  # optional
        },
    },
    'loggers': {
        'django.request': {
            'handlers': ['logstash'],
            'level': 'DEBUG',
            'propagate': True,
        },
    },
    ...
  }
