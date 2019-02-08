
# Ambari Server Install

See the Ambari documentation relative to [Software Requirements][sr] before
executing this module.

    module.exports = header: 'Ambari Server Install', handler: ({options}) ->
      @call 'ryba-ambari-takeover/ambari/agent/wait', options.wait_ambari_agent

# ## Registry
# 
# | Service    | Port  | Proto | Parameter       |
# |------------|-------|-------|-----------------|
# | HST SERVER | 9000  |  tcp  |  HTTP Port      |
# | HST COM    | 9440  |  tcp  |  HTTPS Port     |
# | Analyzer   | 9060  |  tcp  |  HTTPS Port     |
# 
# IPTables rules are only inserted if the parameter "iptables.action" is set to
# "start" (default value).
# 
#       @tools.iptables
#         header: 'Iptables'
#         rules: [
#           { chain: 'INPUT', jump: 'ACCEPT', dport: 9000, protocol: 'tcp', state: 'NEW', comment: "SMARTSENSE SERVER" }
#           { chain: 'INPUT', jump: 'ACCEPT', dport: 9440, protocol: 'tcp', state: 'NEW', comment: "SMARTSENSE AGENT" }
#           { chain: 'INPUT', jump: 'ACCEPT', dport: 9060, protocol: 'tcp', state: 'NEW', comment: "Acitivty Analyzer" }
#         ]
#         if: options.iptables
# 

      @registry.register ['ambari','cluster','deploy'], "ryba-ambari-actions/lib/cluster/deploy"
      @registry.register ['ambari','cluster','start'], "ryba-ambari-actions/lib/cluster/start"

      @krb5.addprinc options.krb5.admin,
        header: 'hdfs principal'
        principal: options.hdfs.krb5_user.principal
        password: options.hdfs.krb5_user.password

      @krb5.addprinc options.krb5.admin,
        header: 'HBase principal'
        principal: options.hbase_admin.principal
        password: options.hbase_admin.password
      
      @call header: 'Hive DB Setup', if: options.hive_db? , ->
        switch options.hive_db.engine
          when 'mariadb', 'mysql'
            # mysql_exec = "mysql -u#{options.db.admin_username} -p#{options.db.admin_password} -h#{options.db.host} -P#{options.db.port} "
            @system.execute
              cmd: db.cmd (merge {}, options.hive_db, database: null) , """
              create database #{options.hive_db.database};
              grant all privileges on #{options.hive_db.database}.* to #{options.hive_db.username}@'localhost' identified by '#{options.hive_db.password}';
              grant all privileges on #{options.hive_db.database}.* to #{options.hive_db.username}@'%' identified by '#{options.hive_db.password}';
              flush privileges;
              """
              unless_exec: db.cmd options.hive_db, "use #{options.hive_db.database}"

      @call header: 'Oozie DB Setup', if: options.oozie_db?, ->
          throw Error 'Database engine not supported' unless options.oozie_db.engine in ['mysql', 'mariadb', 'postgresql']
          escape = (text) -> text.replace(/[\\"]/g, "\\$&")
          version_local = db.cmd(options.oozie_db, "select data from OOZIE_SYS where name='oozie.version'") + "| tail -1"
          version_remote = "ls /usr/hdp/current/oozie-server/lib/oozie-client-*.jar | sed 's/.*client\\-\\(.*\\).jar/\\1/'"
          @db.user options.oozie_db, database: null,
            header: 'User'
          @db.database options.oozie_db,
            header: 'Database'
            user: options.oozie_db.username
          @db.schema options.oozie_db,
            header: 'Schema'
            if: options.oozie_db.engine is 'postgresql'
            schema: options.oozie_db.schema or options.oozie_db.database
            database: options.oozie_db.database
            owner: options.oozie_db.username

      @call header: 'Ranger Admin DB Setup', if: options.ranger_db?, ->
        switch options.ranger_db.engine
          when 'mariadb', 'mysql'
            # mysql_exec = "mysql -u#{options.ranger_db.admin_username} -p#{options.ranger_db.admin_password} -h#{options.ranger_db.host} -P#{options.ranger_db.port} "
            @system.execute
              cmd: db.cmd options.ranger_db, """
              SET GLOBAL log_bin_trust_function_creators = 1;
              create database  #{options.configurations['ranger-admin-site']['ranger.jpa.jdbc.database']} CHARACTER SET=latin1; 
              grant all privileges on #{options.configurations['ranger-admin-site']['ranger.jpa.jdbc.database']}.* to #{options.configurations['ranger-admin-site']['ranger.jpa.jdbc.user']}@'localhost' identified by '#{options.configurations['ranger-admin-site']['ranger.jpa.jdbc.password']}';
              grant all privileges on #{options.configurations['ranger-admin-site']['ranger.jpa.jdbc.database']}.* to #{options.configurations['ranger-admin-site']['ranger.jpa.jdbc.user']}@'%' identified by '#{options.configurations['ranger-admin-site']['ranger.jpa.jdbc.password']}';
              flush privileges;
              """
              unless_exec: db.cmd options.ranger_db, "use #{options.configurations['ranger-admin-site']['ranger.jpa.jdbc.database']}"
            @system.execute
              cmd: db.cmd options.ranger_db, """
              use #{options.configurations['ranger-admin-site']['ranger.jpa.jdbc.database']};
              insert into x_db_version_h SET version='DEFAULT_ADMIN_UPDATE', active='Y', \ 
              updated_at='2017-12-21 13:58:22', updated_by='#{options.fqdn}', \
              inst_by='#{options.fqdn}';
              """
              unless_exec: db.cmd options.ranger_db, "select * from #{options.configurations['ranger-admin-site']['ranger.jpa.jdbc.database']}.x_db_version_h where version='DEFAULT_ADMIN_UPDATE'  | grep '1 rows'"
              code_skipped: 1
      
      # @ambari.cluster.deploy
      #   header: 'Install Cluster'
      #   debug: true
      #   url: options.ambari_url
      #   if: options.post_component and options.takeover
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   name: options.cluster_name



      @call header: "Ranger Post Install", (_, cb) ->
        ssh = merge {}, options.ssh
        ssh['root'].host = options.fqdn if options.root?
        node = nikita()
        node.ssh.open header: "Delegate to: #{options.services['RANGER']['RANGER_ADMIN']['hosts'][0]}", host: options.services['RANGER']['RANGER_ADMIN']['hosts'][0]
        node.java.keystore_add
          header: 'SSL'
          keystore: options.configurations['ranger-admin-site']['ranger.service.https.attrib.keystore.file']
          storepass: options.configurations['ranger-admin-site']['ranger.service.https.attrib.keystore.pass']
          key: "#{options.ranger_ssl.key.source}"
          cert: "#{options.ranger_ssl.cert.source}"
          keypass: options.configurations['ranger-admin-site']['ranger.service.https.attrib.keystore.pass']
          name: options.configurations['ranger-admin-site']['ranger.service.https.attrib.keystore.keyalias']
          local: "#{options.ranger_ssl.cert.local}"
        node.java.keystore_add
          keystore: options.configurations['ranger-admin-site']['ranger.truststore.file']
          storepass: options.configurations['ranger-admin-site']['ranger.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ranger_ssl.cacert.source}"
          local: "#{options.ranger_ssl.cacert.local}"
        node.java.keystore_add
          keystore: '/usr/java/latest/jre/lib/security/cacerts'
          storepass: 'changeit'
          caname: "hadoop_root_ca"
          cacert: "#{options.ranger_ssl.cacert.source}"
          local: "#{options.ranger_ssl.cacert.local}"
        node.system.execute
          header: 'Fix credential Store'
          cmd: """
            cd /usr/hdp/current/ranger-admin
            java -cp "cred/lib/*" org.apache.ranger.credentialapi.buildks create rangeradmin.keystore -value '#{options.configurations['ranger-admin-site']['ranger.service.https.attrib.keystore.pass']}' -provider jceks://file#{options.configurations['ranger-admin-site']['ranger.credential.provider.path']}
          """
        node.system.link
          header: 'DB Driver'
          source: '/usr/share/java/mysql-connector-java.jar'
          target: '/usr/hdp/current/ranger-admin/lib/mysql-connector-java.jar'
        node.ssh.close()
        node.next cb
      
      @call header: 'Knox Gateway Post Install', if: options.services['KNOX']?, handler: ->
        {ssh, fqdn, services, configurations, knox_opts, ssl} = options
        @each options.services['KNOX']['KNOX_GATEWAY']['hosts'], ({options}, cb) ->
          delegate_to = options.key
          opts_ssh = merge {}, ssh
          opts_ssh['root'].host = fqdn if options.root?
          node = nikita()
          node.ssh.open header: "Delegate to: #{delegate_to}", host: delegate_to
          node.each  [
            '/usr/hdp/current/knox-server/data/security/master'
            '/usr/hdp/current/knox-server/data/security/keystores'
          ] , ({options}) ->
              node.system.remove  target: options.key
          node.file.render
            header: 'Ambari Knox Ldap Caching'
            target: "/etc/knox/conf/ehcache.xml"
            source: "#{__dirname}/../knox/resources/ehcache.j2"
            local: true
            context: options: knox_opts
          node.call header: 'Topologies', ->
            for nameservice, topology of knox_opts.topologies
              doc = builder.create 'topology', version: '1.0', encoding: 'UTF-8'
              gateway = doc.ele 'gateway' if topology.providers?
              for role, p of topology.providers
                provider = gateway.ele 'provider'
                provider.ele 'role', role
                provider.ele 'name', p.name
                provider.ele 'enabled', if p.enabled? then "#{p.enabled}" else 'true'
                if typeof p.config is 'object'
                  for name in Object.keys(p.config).sort()
                    if p.config[name]
                      param = provider.ele 'param'
                      param.ele 'name', name
                      param.ele 'value', p.config[name]
              for role, url_params of topology.services
                unless url_params is false
                  service = doc.ele 'service'
                  service.ele 'role', role.toUpperCase()
                  if Array.isArray url_params then for u in url_params
                    service.ele 'url', u
                  else if typeof url_params is 'object'
                    service.ele 'url',url_params.url
                    if url_params.params? then for param,value of url_params.params
                      service.ele 'param', param
                      service.ele 'value', value
                  else if url_params not in [null, ''] then service.ele 'url', url_params
              node.file
                target: "/etc/knox/conf/topologies/#{nameservice}.xml"
                content: doc.end pretty: true
                backup: true
                eof: true
              node.file.render
                target: "/etc/knox/conf/#{nameservice}-ehcache.xml"
                source: "#{__dirname}/../knox/resources/ehcache.j2"
                local: true
                context: nameservice: nameservice
          node.call
            header: 'Create Keystore'
            unless_exists: '/usr/hdp/current/knox-server/data/security/master'
          , (_, callback) ->
            sshexec = node.ssh opts_ssh
            sshexec.shell (err, stream) =>
              stream.write "su -l #{knox_opts.user.name} -c '/usr/hdp/current/knox-server/bin/knoxcli.sh create-master'\n"
              stream.on 'data', (data, extended) ->
                if /Enter master secret/.test data then stream.write "#{knox_opts.ssl.keystore.password}\n"
                if /Master secret is already present on disk/.test data then callback null, false
                else if /Master secret has been persisted to disk/.test data then callback null, true
              stream.on 'exit', -> callback Error 'Exit before end'
          node.call header: 'Store Password', ->
            # Create alias to store password used in topology
            for alias,password of knox_opts.realm_passwords then do (alias,password) =>
              nameservice=alias.split("-")[0]
              node.system.execute
                cmd: "/usr/hdp/current/knox-server/bin/knoxcli.sh create-alias #{alias} --cluster #{nameservice} --value #{password}"
          node.java.keystore_add
            keystore: knox_opts.ssl.keystore.target
            storepass: knox_opts.ssl.keystore.password
            key: ssl.key.source
            cert: ssl.cert.source
            keypass: knox_opts.ssl.keystore.keypass
            name: ssl.key.name
            local:  ssl.key.local
          node.java.keystore_add
            keystore: knox_opts.ssl.keystore.target
            storepass: knox_opts.ssl.keystore.password
            caname: knox_opts.ssl.truststore.caname
            cacert: ssl.cacert.source
            local: ssl.cacert.local
          node.java.keystore_add
            keystore: "#{knox_opts.jre_home or knox_opts.java_home}/lib/security/cacerts"
            storepass: 'changeit'
            caname: ssl.truststore.caname
            cacert: ssl.cacert.source
            local: ssl.cacert.local
          node.system.execute
            if: -> node.status -1
            cmd: "/usr/hdp/current/knox-server/bin/knoxcli.sh create-alias gateway-identity-passphrase --value #{knox_opts.ssl.keystore.keypass}"
          node.ssh.close()
          node.next cb

## Schedule Purge Transaction Logs

A ZooKeeper server will not remove old snapshots and log files when using the
default configuration (see autopurge below), this is the responsibility of the
operator.

The PurgeTxnLog utility implements a simple retention policy that administrators
can use. Its expected arguments are "dataLogDir [snapDir] -n count".

Note, Automatic purging of the snapshots and corresponding transaction logs was
introduced in version 3.4.0 and can be enabled via the following configuration
parameters autopurge.snapRetainCount and autopurge.purgeInterval.

```
/usr/bin/java \
  -cp /usr/hdp/current/zookeeper-server/zookeeper.jar:/usr/hdp/current/zookeeper-server/lib/*:/usr/hdp/current/zookeeper-server/conf \
  org.apache.zookeeper.server.PurgeTxnLog  /var/zookeeper/data/ -n 3
```


      @call header: 'Zookeeper Server Post Install', handler: ->
        {ssh, fqdn, services, configurations, knox_opts, ssl, zookeeper, zookeeper_user } = options
        @each options.services['ZOOKEEPER']['ZOOKEEPER_SERVER']['hosts'], ({options}, cb) ->
          delegate_to = options.key
          opts_ssh = merge {}, ssh
          opts_ssh['root'].host = fqdn if options.root?
          node = nikita()
          node.ssh.open header: "Delegate to: #{delegate_to}", host: delegate_to
          node.cron.add
            header: 'Schedule Purge'
            cmd: """
            /usr/bin/java -cp /usr/hdp/current/zookeeper-server/zookeeper.jar:/usr/hdp/current/zookeeper-server/lib/*:/usr/hdp/current/zookeeper-server/conf \
              org.apache.zookeeper.server.PurgeTxnLog \
              #{configurations['zoo.cfg'].dataLogDir or ''} #{configurations['zoo.cfg'].dataDir} -n #{zookeeper.retention}
            """
            user: zookeeper_user.name
            when: zookeeper.purge
          node.ssh.close()
          node.next cb

      @call header: 'YARN NM Server Post Install', handler: ->
        {ssh, fqdn, services, configurations, ssl} = options
        @each options.services['YARN']['NODEMANAGER']['hosts'], ({options}, cb) ->
          delegate_to = options.key
          opts_ssh = merge {}, ssh
          opts_ssh['root'].host = fqdn if options.root?
          node = nikita()
          node.ssh.open header: "Delegate to: #{delegate_to}", host: delegate_to
          node.service name: 'snappy'
          node.service name: 'snappy-devel'
          node.system.link
            source: '/usr/lib64/libsnappy.so'
            target: '/usr/hdp/current/hadoop-client/lib/native/.'
          node.call (_, callback) ->
            node.service
              name: 'lzo-devel'
              relax: true
            , (err) ->
              node.service.remove
                if: !!err
                name: 'lzo-devel'
              node.next callback
          node.service
            name: 'hadooplzo'
          node.service
            name: 'hadooplzo-native'
          node.ssh.close()
          node.next cb

      @call header: 'Hadoop CORE Post Install Topology', handler: ->
        {ssh, fqdn, services, configurations, ssl, topology, hdfs_user, hadoop_group} = options
        @each options.core_hosts, ({options}, cb) ->
          delegate_to = options.key
          opts_ssh = merge {}, ssh
          opts_ssh['root'].host = fqdn if options.root?
          node = nikita()
          node.ssh.open header: "Delegate to: #{delegate_to}", host: delegate_to
          node.file
            target: "/etc/hadoop/conf/rack_topology.sh"
            source: "#{__dirname}/../hadoop/resources/rack_topology.sh"
            local: true
            uid: hdfs_user.name
            gid: hadoop_group.name
            mode: 0o755
            backup: true
          node.file
            target: "/etc/hadoop/conf/rack_topology.data"
            content: topology
              .map (node) ->
                "#{node.ip}  #{node.rack or ''}"
              .join '\n'
            uid: hdfs_user.name
            gid: hadoop_group.name
            mode: 0o755
            backup: true
            eof: true
          node.ssh.close()
          node.next cb

      # todo Version Ambari Check
      if options.ambari_infra
        @file.download
          header: 'Ambari Infra params.py'
          source: "#{__dirname}/../ambari_infra/resources/params.py"
          target: "/var/lib/ambari-server/resources/common-services/AMBARI_INFRA/0.1.0/package/scripts/params.py"
          local: true
          mode: 0o755
        @file.download
          header: 'Ambari Infra setup_infra_solr.py'
          source: "#{__dirname}/../ambari_infra/resources/setup_infra_solr.py"
          target: "/var/lib/ambari-server/resources/common-services/AMBARI_INFRA/0.1.0/package/scripts/setup_infra_solr.py"
          local: true
          mode: 0o755
        @file.download
          header: 'Logsearch Server params.py'
          source: "#{__dirname}/../logsearch/resources/params.py"
          target: "/var/lib/ambari-server/resources/common-services/LOGSEARCH/0.5.0/package/scripts/params.py"
          local: true
          mode: 0o755
        @file.download
          header: 'Logsearch Server params.py'
          source: "#{__dirname}/../resources/solr_cloud_util.py"
          target: "/usr/lib/ambari-server/lib/resource_management/libraries/functions/solr_cloud_util.py"
          local: true
          mode: 0o755

      @call -> process.exit 0

## Compression

      @ambari.cluster.start
        header: 'Start Cluster'
        debug: true
        url: options.ambari_url
        if: options.post_component and options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        name: options.cluster_name
          

        # @system.link
        #     header: 'MySQL Connector'
        #     source: '/usr/share/java/mysql-connector-java.jar'
        #     target: '/usr/hdp/current/sqoop-client/lib/mysql-connector-java.jar'


## Dependencies

    fs = require 'fs'
    glob = require 'glob'
    path = require 'path'
    quote = require 'regexp-quote'
    db = require 'nikita/lib/misc/db'
    ssh2fs = require 'ssh2-fs'
    nikita = require 'nikita'
    {merge} = require 'nikita/lib/misc'
    builder = require 'xmlbuilder'

[sr]: http://docs.hortonworks.com/HDPDocuments/Ambari-2.2.2.0/bk_Installing_HDP_AMB/content/_meet_minimum_system_requirements.html
