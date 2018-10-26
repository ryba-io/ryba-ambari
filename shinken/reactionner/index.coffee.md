
# Shinken Reactionner

Gets notifications and eventhandlers from the scheduler, executes plugins/scripts
and sends the results to the scheduler.

    module.exports =
      deps:
        ssl : module: 'masson/core/ssl', local: true
        iptables: module: 'masson/core/iptables', local: true
        commons: implicit: true, module: 'ryba-ambari-takeover/shinken/commons', local: true
        reactionner: module: 'ryba-ambari-takeover/shinken/reactionner'
      configure:
        'ryba-ambari-takeover/shinken/reactionner/configure'
      commands:
        'check':
          'ryba-ambari-takeover/shinken/reactionner/check'
        'install': [
          'ryba-ambari-takeover/shinken/reactionner/install'
          'ryba-ambari-takeover/shinken/reactionner/start'
          'ryba-ambari-takeover/shinken/reactionner/check'
        ]
        'prepare':
          'ryba-ambari-takeover/shinken/reactionner/prepare'
        'start':
          'ryba-ambari-takeover/shinken/reactionner/start'
        'status':
          'ryba-ambari-takeover/shinken/reactionner/status'
        'stop':
          'ryba-ambari-takeover/shinken/reactionner/stop'
