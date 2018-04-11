
# Ranger Admin Install

    module.exports =  header: 'Ambari Ranger Admin Install', handler: (options) ->

## Register

      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"

## Identities

      @system.group header: 'Group', options.group
      @system.user header: 'User', options.user

## Package

Install the Ranger Policy Manager package and set it to the latest version. Note, we
select the "kafka-broker" hdp directory. There is no "kafka-consumer"
directories.

      @call header: 'Packages', ->
        @service.install
          name: 'ranger-admin'
        @hdp_select
          name: 'ranger-admin'

## Layout

      @system.mkdir
        target: '/var/run/ranger'
        uid: options.user.name
        gid: options.user.name
        mode: 0o750

## IPTables

| Service              | Port  | Proto       | Parameter          |
|----------------------|-------|-------------|--------------------|
| Ranger policymanager | 6080  | http        | port               |
| Ranger policymanager | 6182  | https       | port               |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      @tools.iptables
        header: 'Ambari Ranger Admin IPTables'
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.site['ranger.service.http.port'], protocol: 'tcp', state: 'NEW', comment: "Ranger Admin HTTP WEBUI" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.site['ranger.service.https.port'], protocol: 'tcp', state: 'NEW', comment: "Ranger Admin HTTPS WEBUI" }
        ]

## Ranger Admin Driver

      @system.link
        header: 'DB Driver'
        source: '/usr/share/java/mysql-connector-java.jar'
        target: options.install['SQL_CONNECTOR_JAR']

## Ranger Databases
Create the rangeradmin and rangerlogger databases.

      @system.execute
        header: 'Fix credential Store'
        cmd: """
          cd /usr/hdp/current/ranger-admin
          java -cp "cred/lib/*" org.apache.ranger.credentialapi.buildks create rangeradmin.keystore -value '#{options.credential_password}' -provider jceks://file#{options.site['ranger.credential.provider.path']}
        """

      @call header: 'DB Setup', ->
        switch options.db.engine
          when 'mariadb', 'mysql'
            # mysql_exec = "mysql -u#{options.db.admin_username} -p#{options.db.admin_password} -h#{options.db.host} -P#{options.db.port} "
            @system.execute
              cmd: db.cmd options.db, """
              SET GLOBAL log_bin_trust_function_creators = 1;
              create database  #{options.install['db_name']} CHARACTER SET=latin1; 
              grant all privileges on #{options.install['db_name']}.* to #{options.install['db_user']}@'localhost' identified by '#{options.install['db_password']}';
              grant all privileges on #{options.install['db_name']}.* to #{options.install['db_user']}@'%' identified by '#{options.install['db_password']}';
              flush privileges;
              """
              unless_exec: db.cmd options.db, "use #{options.install['db_name']}"
            @system.execute
              cmd: db.cmd options.db, """
              create database  #{options.install['audit_db_name']} CHARACTER SET=latin1;
              grant all privileges on #{options.install['audit_db_name']}.* to #{options.install['audit_db_user']}@'localhost' identified by '#{options.install['audit_db_password']}';
              grant all privileges on #{options.install['audit_db_name']}.* to #{options.install['audit_db_user']}@'%' identified by '#{options.install['audit_db_password']}';
              flush privileges;
              """
              unless_exec: db.cmd options.db, "use #{options.install['audit_db_name']}"

      # @db.user options.db, database: null, username: options.install['db_user'], password: options.install['db_password'],
      #   header: 'User'
      #   if: options.db.engine in ['mysql', 'mariadb', 'postgresql']
      # @db.database options.db,
      #   header: 'Database'
      #   user: options.install['db_user']
      #   database: options.install['db_name']
      #   password: options.install['db_password']
      #   if: options.db.engine in ['mysql', 'mariadb', 'postgresql']
      # @db.schema options.db,
      #   header: 'Schema'
      #   if: options.db.engine is 'postgresql'
      #   schema: options.install['db_name']
      #   database: options.install['db_name']
      #   password: options.install['db_password']
      #   owner: options.install['db_user']


      # @db.user options.db, database: null, username: options.install['audit_db_user'], password: options.install['audit_db_password'],
      #   header: 'User'
      #   if: options.db.engine in ['mysql', 'mariadb', 'postgresql']
      # @db.database options.db,
      #   header: 'Database'
      #   user: options.install['audit_db_user']
      #   database: options.install['audit_db_name']
      #   password: options.install['audit_db_password']
      #   if: options.db.engine in ['mysql', 'mariadb', 'postgresql']
      # @db.schema options.db,
      #   header: 'Schema'
      #   if: options.db.engine is 'postgresql'
      #   schema: options.install['audit_db_name']
      #   database: options.install['audit_db_name']
      #   password: options.install['audit_db_password']
      #   owner: options.install['audit_db_user']
      # @system.execute
      #   shy: true
      #   if: options.db.engine in ['mysql', 'mariadb']
      #   cmd: db.cmd options.db, """
      #   SET GLOBAL log_bin_trust_function_creators = 1;
      #   """
              
## Layout

      @system.tmpfs
        if_os: name: ['redhat','centos'], version: '7'
        mount: '/var/run/ranger'
        uid: options.user.name
        gid: options.user.name
        perm: '0750'

## SSL

      @call
        header: 'Configure SSL'
        if: (options.site['ranger.service.https.attrib.ssl.enabled'] is 'true')
      , ->
        @java.keystore_add
          header: 'SSL'
          keystore: options.site['ranger.service.https.attrib.keystore.file']
          storepass: options.site['ranger.service.https.attrib.keystore.pass']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.site['ranger.service.https.attrib.keystore.pass']
          name: options.site['ranger.service.https.attrib.keystore.keyalias']
          local: "#{options.ssl.cert.local}"
        @java.keystore_add
          keystore: options.site['ranger.truststore.file']
          storepass: options.site['ranger.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        @java.keystore_add
          keystore: '/usr/java/latest/jre/lib/security/cacerts'
          storepass: 'changeit'
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

## Ranger Admin Principal

      @krb5.addprinc options.krb5.admin,
        if: options.plugins.principal
        header: 'Ranger Repositories principal'
        principal: options.plugins.principal
        randkey: true
        password: options.plugins.password
      @krb5.addprinc options.krb5.admin,
        header: 'Ranger Web UI'
        principal: options.install['admin_principal']
        randkey: true
        keytab: options.install['admin_keytab']
        uid: options.user.name
        gid: options.user.name
        mode: 0o600
      @krb5.addprinc options.krb5.admin,
        header: 'Ranger Web UI'
        principal: options.install['lookup_principal']
        randkey: true
        keytab: options.install['lookup_keytab']
        uid: options.user.name
        gid: options.user.name
        mode: 0o600

## Upload Ranger Admin configurations

      @ambari.configs.update
        header: 'ranger-admin-site'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-admin-site'
        cluster_name: options.cluster_name
        properties: options.site

      @ambari.configs.update
        header: 'ranger-ugsync-site'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-ugsync-site'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-ugsync-site']

      @ambari.configs.update
        header: 'ranger-env'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-env'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-env']

      @ambari.configs.update
        header: 'admin-properties'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'admin-properties'
        cluster_name: options.cluster_name
        properties: options.install

      @call
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/solr/solrconfig.xml.#{options.download}.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              options.configurations = merge {}, options.configurations,
                'ranger-solr-configuration':
                  'content': content
              @ambari.configs.update
                header: 'ranger-solr-configuration'
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'ranger-solr-configuration'
                cluster_name: options.cluster_name
                properties: options.configurations['ranger-solr-configuration']
              @next callback
            catch err
              callback err

      @call
        header: 'admin-log4j'
        if: options.post_component and options.takeover
      , (_, callback)->
        ssh2fs.readFile null, "#{__dirname}/../resources/log4j.properties", (err, content) =>
          try
            throw err if err
            content = content.toString()
            @ambari.configs.update
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'admin-log4j'
              cluster_name: options.cluster_name
              properties: 
                content: content
            .next callback
          catch err
            callback err

## Provision Ambari Services

      @ambari.services.add
        header: 'Service RANGER'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'RANGER'

      @ambari.services.wait
        header: 'Service WAITED'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'RANGER'

      @ambari.services.component_add
        header: 'RANGER_ADMIN'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'RANGER_ADMIN'
        service_name: 'RANGER'
      
      @ambari.hosts.component_add
        header: 'RANGER_ADMIN ADD'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'RANGER_ADMIN'
        hostname: options.fqdn

      @ambari.hosts.component_wait
        header: 'RANGER_ADMIN'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'RANGER_ADMIN'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'RANGER_ADMIN'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'RANGER_ADMIN'
        hostname: options.fqdn

## Dependencies

    glob = require 'glob'
    path = require 'path'
    quote = require 'regexp-quote'
    db = require 'nikita/lib/misc/db'
    ssh2fs = require 'ssh2-fs'
    {merge} = require 'nikita/lib/misc'

[instruction-24-25]:http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.0/bk_command-line-upgrade/content/upgrade-ranger_24.html
