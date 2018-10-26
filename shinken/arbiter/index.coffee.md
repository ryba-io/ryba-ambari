
# Shinken Arbiter

Loads the configuration files and dispatches the host and service objects to the
scheduler(s). Watchdog for all other processes and responsible for initiating
failovers if an error is detected. Can route check result events from a Receiver
to its associated Scheduler.

    module.exports =
      deps:
        ssl : module: 'masson/core/ssl', local: true
        iptables: module: 'masson/core/iptables', local: true
        commons:  module: 'ryba-ambari-takeover/shinken/commons', local: true, required: true
        monitoring: module: 'ryba-ambari-takeover/commons/monitoring', local: true, required: true
        arbiter: module: 'ryba-ambari-takeover/shinken/arbiter'
        reactionner: module: 'ryba-ambari-takeover/shinken/reactionner'
        receiver: module: 'ryba-ambari-takeover/shinken/receiver'
        scheduler: module: 'ryba-ambari-takeover/shinken/scheduler'
        broker: module: 'ryba-ambari-takeover/shinken/broker'
        poller: module: 'ryba-ambari-takeover/shinken/poller'
      configure:
        'ryba-ambari-takeover/shinken/arbiter/configure'
      commands:
        'check':
          'ryba-ambari-takeover/shinken/arbiter/check'
        'install': [
          'ryba-ambari-takeover/shinken/arbiter/install'
          'ryba-ambari-takeover/shinken/arbiter/start'
          'ryba-ambari-takeover/shinken/arbiter/check'
        ]
        'prepare':
          'ryba-ambari-takeover/shinken/arbiter/prepare'
        'start':
          'ryba-ambari-takeover/shinken/arbiter/start'
        'status':
          'ryba-ambari-takeover/shinken/arbiter/status'
        'stop':
          'ryba-ambari-takeover/shinken/arbiter/stop'
