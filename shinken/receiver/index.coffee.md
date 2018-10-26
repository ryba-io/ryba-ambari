
# Shinken Receiver (optional)

Receives data passively from local or remote protocols. Passive data reception
that is buffered before forwarding to the appropriate Scheduler (or receiver for global commands).
Allows to set up a "farm" of Receivers to handle a high rate of incoming events.
Modules for receivers:

* NSCA - NSCA protocol receiver
* Collectd - Receive performance data from collectd via the network
* CommandPipe - Receive commands, status updates and performance data
* TSCA - Apache Thrift interface to send check results using a high rate buffered TCP connection directly from programs
* Web Service - A web service that accepts http posts of check results (beta)

This module is only needed when enabling passive checks

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        iptables: module: 'masson/core/iptables', local: true
        commons: module: 'ryba-ambari-takeover/shinken/commons', local: true
        receiver: module: 'ryba-ambari-takeover/shinken/receiver'
      configure:
        'ryba-ambari-takeover/shinken/receiver/configure'
      commands:
        'check':
          'ryba-ambari-takeover/shinken/receiver/check'
        'install': [
          'ryba-ambari-takeover/shinken/receiver/install'
          'ryba-ambari-takeover/shinken/receiver/start'
          'ryba-ambari-takeover/shinken/receiver/check'
        ]
        'prepare':
          'ryba-ambari-takeover/shinken/receiver/prepare'
        'start':
          'ryba-ambari-takeover/shinken/receiver/start'
        'status':
          'ryba-ambari-takeover/shinken/receiver/status'
        'stop':
          'ryba-ambari-takeover/shinken/receiver/stop'
