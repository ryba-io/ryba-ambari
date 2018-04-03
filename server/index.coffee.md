
# Ambari Server

[Ambari-server][Ambari-server] is the master host for ambari software.
Once logged into the ambari server host, the administrator can  provision, 
manage and monitor a Hadoop cluster.

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        ambari_server_local: module: 'ryba/ambari/server', local: true, required: true
        ambari_server: module: 'ryba/ambari/server', required: true
        
      configure:
        'ryba-ambari-takeover/server/configure'
      commands:
        'check':
          'ryba-ambari-takeover/server/check'
        'install': [
          'ryba-ambari-takeover/server/install'
        ]

[Ambari-server]: http://ambari.apache.org
