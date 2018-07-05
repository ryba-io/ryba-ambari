
# NiFi

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        iptables: module: 'masson/core/iptables', local: true
      configure:
        'ryba-ambari-takeover/nifi/configure'
      commands:
        'prepare':
          'ryba-ambari-takeover/nifi/prepare'