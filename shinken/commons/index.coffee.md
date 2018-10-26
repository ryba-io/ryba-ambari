
# Shinken Commons

This module contains configuration, dependencies, and installation steps commons
to all shinken submodules

    module.exports =
      deps:
        ssl:  module: 'masson/core/ssl', local: true
        commons: module: 'ryba-ambari-takeover/shinken/commons'
      configure:
        'ryba-ambari-takeover/shinken/commons/configure'
      commands:
        'install':
          'ryba-ambari-takeover/shinken/commons/install'
        'prepare':
          'ryba-ambari-takeover/shinken/commons/prepare'
