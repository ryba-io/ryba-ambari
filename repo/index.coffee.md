
# Ambari Repo

    module.exports =
      deps: {}
      configure:
        'ryba-ambari-takeover/ambari/repo/configure'
      commands:
        'install':  [
            'ryba/ambari/repo/install'
          ]
