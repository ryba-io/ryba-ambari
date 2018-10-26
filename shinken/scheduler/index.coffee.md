
# Shinken Scheduler

Plans the next run of host and service checks
Dispatches checks to the poller(s)
Calculates state and dependencies
Applies KPI triggers
Raises Notifications and dispatches them to the reactionner(s)
Updates the retention file (or other retention backends)
Sends broks (internal events of any kind) to the broker(s)

    module.exports =
      deps:
        ssl : module: 'masson/core/ssl', local: true
        iptables: module: 'masson/core/iptables', local: true
        commons: implicit: true, module: 'ryba-ambari-takeover/shinken/commons', local: true
        scheduler: module: 'ryba-ambari-takeover/shinken/scheduler'
      configure:
        'ryba-ambari-takeover/shinken/scheduler/configure'
      commands:
        'check':
          'ryba-ambari-takeover/shinken/scheduler/check'
        'install': [
          'ryba-ambari-takeover/shinken/scheduler/install'
          'ryba-ambari-takeover/shinken/scheduler/start'
          'ryba-ambari-takeover/shinken/scheduler/check'
        ]
        'prepare':
          'ryba-ambari-takeover/shinken/scheduler/prepare'
        'start':
          'ryba-ambari-takeover/shinken/scheduler/start'
        'status':
          'ryba-ambari-takeover/shinken/scheduler/status'
        'stop':
          'ryba-ambari-takeover/shinken/scheduler/stop'
