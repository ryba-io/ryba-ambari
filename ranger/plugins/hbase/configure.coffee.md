
## Ranger HBase Plugin Configure

    module.exports = (service) ->
      options = service.options

## Identities

      options.group = merge {}, service.deps.ranger_admin.options.group, options.group or {}
      options.user = merge {}, service.deps.ranger_admin.options.user, options.user or {}
      options.hbase_user = service.deps.hbase_master[0].options.user
      options.hbase_group = service.deps.hbase_master[0].options.group
      options.hadoop_group = service.deps.hbase_master[0].options.hadoop_group
      options.hdfs_krb5_user = service.deps.hadoop_core.options.hdfs.krb5_user

## Kerberos

      options.krb5 ?= {}
      options.krb5.enabled ?= service.deps.hadoop_core.options.core_site['hadoop.security.authentication'] is 'kerberos'
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin = service.deps.krb5_client.options.admin[options.krb5.realm]

## Access

      options.ranger_admin ?= service.deps.ranger_admin.options.admin
      options.hdfs_install ?= service.deps.ranger_hdfs[0].options.install

## Environment

      # Layout
      options.conf_dir ?= []
      options.conf_dir.push service.deps.hbase_master[0].options.conf_dir if service.deps.hbase_master.some (srv) -> srv.node.fqdn is service.node.fqdn
      options.conf_dir.push service.deps.hbase_regionserver.options.conf_dir if service.deps.hbase_regionserver
      # Java
      options.jre_home ?= service.deps.java.options.jre_home

## Plugin User

      options.plugin_user = 
        "name": options.hbase_user.name
        "firstName": ''
        "lastName": ''
        "emailAddress": ''
        "password": ''
        'userSource': 1
        'userRoleList': ['ROLE_USER']
        'groups': []
        'status': 1
      # service.deps.ranger_admin.options.users['hbase'] ?= options.ranger_user

## Configuration

      #misc
      options.master_fqdn = service.deps.hbase_master[0].node.fqdn
      options.fqdn = service.node.fqdn
      options.install ?= {}
      # migration: wdavidw 170902, used in hbase/rest/check, should be moved
      # options.policy_name ?= "Ranger-Ryba-HBase-Policy"
      options.install ?= {}
      options.install['PYTHON_COMMAND_INVOKER'] ?= 'python'
      options.install['CUSTOM_USER'] ?= "#{options.hbase_user.name}"

## Ranger Coprocessor

      # Master Hbase site
      for srv in service.deps.hbase
        for prop in ['hbase.coprocessor.master.classes','hbase.coprocessor.region.classes'] 
          srv.options.configurations['hbase-site'][prop] = srv.options.configurations['hbase-site'][prop].split(',') unless Array.isArray  srv.options.configurations['hbase-site'][prop]
          index = srv.options.configurations['hbase-site'][prop].indexOf('org.apache.hadoop.hbase.security.access.AccessController')
          if index >= 0
            srv.options.configurations['hbase-site'][prop].splice(index, 1)
          srv.options.configurations['hbase-site'][prop].push 'org.apache.ranger.authorization.hbase.RangerAuthorizationCoprocessor' unless 'org.apache.ranger.authorization.hbase.RangerAuthorizationCoprocessor' in   srv.options.configurations['hbase-site'][prop]

## HBase regionserver env

Some ranger plugins needs to have the configuration file on their classpath to 
make configuration effective.

      # migration: wdavidw 170902, code is ready but commented for now, maybe
      # it should apply to hbase master as well.
      # for srv in service.deps.hbase_regionserver
      #   core_site_path = "#{srv.options.conf_dir}/core-site.xml"
      #   unless srv.options.env['HBASE_CLASSPATH']
      #     srv.options.env['HBASE_CLASSPATH'] = "$HBASE_CLASSPATH:#{core_site_path}"
      #   else if (srv.options.env['HBASE_CLASSPATH'].indexOf(":#{core_site_path}") is -1)
      #     srv.options.env['HBASE_CLASSPATH'] += ":#{core_site_path}"

## Admin properties

      options.install['POLICY_MGR_URL'] ?= service.deps.ranger_admin.options.install['policymgr_external_url']
      options.install['REPOSITORY_NAME'] ?= 'hadoop-ryba-hbase'

## Service Definition

      options.service_repo ?=
        'name': options.install['REPOSITORY_NAME']
        'description': 'HBase Repo'
        'type': 'hbase'
        'isEnabled': true
        'configs':
          # 'username': 'ranger_plugin_hbase'
          # 'password': 'RangerPluginHBase123!'
          'username': service.deps.hbase[0].options.admin.principal
          'password': service.deps.hbase[0].options.admin.password
          'hadoop.security.authorization': service.deps.hadoop_core.options.core_site['hadoop.security.authorization']
          'hbase.master.kerberos.principal': service.deps.hbase_master[0].options.hbase_site['hbase.master.kerberos.principal']
          'hadoop.security.authentication': service.deps.hadoop_core.options.core_site['hadoop.security.authentication']
          'hbase.security.authentication': service.deps.hbase_master[0].options.hbase_site['hbase.security.authentication']
          'hbase.zookeeper.property.clientPort': service.deps.hbase_master[0].options.hbase_site['hbase.zookeeper.property.clientPort']
          'hbase.zookeeper.quorum': service.deps.hbase_master[0].options.hbase_site['hbase.zookeeper.quorum']
          'zookeeper.znode.parent': service.deps.hbase_master[0].options.hbase_site['zookeeper.znode.parent']
          'policy.download.auth.users': "#{options.hbase_user.name}" #from ranger 0.6
          'tag.download.auth.users': "#{options.hbase_user.name}"
          'policy.grantrevoke.auth.users': "#{options.hbase_user.name}"
          'commonNameForCertificate': ''
      options.install['XAAUDIT.SUMMARY.ENABLE'] ?= 'true'
      options.install['UPDATE_XAPOLICIES_ON_GRANT_REVOKE'] ?= 'true'

## Audit

### HDFS Storage

      options.install['XAAUDIT.HDFS.IS_ENABLED'] ?= 'true'
      if options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
        # migration: lucasbak 11102017
        # honored but not used by plugin
        # options.install['XAAUDIT.HDFS.LOCAL_BUFFER_DIRECTORY'] ?= "#{service.deps.ranger_admin.options.conf_dir}/%app-type%/audit"
        # options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE_DIRECTORY'] ?= "#{service.deps.ranger_admin.options.conf_dir}/%app-type%/archive"
        options.install['XAAUDIT.HDFS.ENABLE'] ?= 'true'
        options.install['XAAUDIT.HDFS.HDFS_DIR'] ?= "#{service.deps.hdfs_client.options.core_site['fs.defaultFS']}/#{options.user.name}/audit"
        options.install['XAAUDIT.HDFS.DESTINATION_DIRECTORY'] ?= "#{service.deps.hdfs_client.options.core_site['fs.defaultFS']}/#{options.user.name}/audit/%app-type%/%time:yyyyMMdd%"
        options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR'] ?= "#{service.deps.hbase_master[0].options.log_dir}/audit/hdfs/spool"
        options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE_DIRECTORY'] ?= '/var/log/ranger/%app-type%/archive'
        options.install['XAAUDIT.HDFS.DESTINATION_FILE'] ?= '%hostname%-audit.log'
        options.install['XAAUDIT.HDFS.DESTINATION_FLUSH_INTERVAL_SECONDS'] ?= '900'
        options.install['XAAUDIT.HDFS.DESTINATION_ROLLOVER_INTERVAL_SECONDS'] ?= '86400'
        options.install['XAAUDIT.HDFS.DESTINATION _OPEN_RETRY_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FILE'] ?= '%time:yyyyMMdd-HHmm.ss%.log'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_FLUSH_INTERVAL_SECONDS'] ?= '60'
        options.install['XAAUDIT.HDFS.LOCAL_BUFFER_ROLLOVER_INTERVAL_SECONDS'] ?= '600'
        options.install['XAAUDIT.HDFS.LOCAL_ARCHIVE _MAX_FILE_COUNT'] ?= '5'

### Solr Storage

      if service.deps.ranger_admin.options.install['audit_store'] is 'solr'
        options.install['XAAUDIT.SOLR.IS_ENABLED'] ?= 'true'
        options.install['XAAUDIT.SOLR.ENABLE'] ?= 'true'
        options.install['XAAUDIT.SOLR.URL'] ?= service.deps.ranger_admin.options.install['audit_solr_urls']
        options.install['XAAUDIT.SOLR.USER'] ?= service.deps.ranger_admin.options.install['audit_solr_user']
        options.install['XAAUDIT.SOLR.ZOOKEEPER'] ?= service.deps.ranger_admin.options.install['audit_solr_zookeepers']
        options.install['XAAUDIT.SOLR.PASSWORD'] ?= service.deps.ranger_admin.options.install['audit_solr_password']
        options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR'] ?= "#{service.deps.hbase_master[0].options.log_dir}/audit/solr/spool"

### Database Storage

      # Deprecated
      # migration: wdavidw 170902, in favor of what ?
      options.install['XAAUDIT.DB.IS_ENABLED'] ?= 'false'
      if options.install['XAAUDIT.DB.IS_ENABLED'] is 'true'
        options.install['XAAUDIT.DB.FLAVOUR'] ?= 'MYSQL'
        switch options.install['XAAUDIT.DB.FLAVOUR']
          when 'MYSQL'
            options.install['SQL_CONNECTOR_JAR'] ?= '/usr/share/java/mysql-connector-java.jar'
            options.install['XAAUDIT.DB.HOSTNAME'] ?= service.deps.ranger_admin.options.install['db_host']
            options.install['XAAUDIT.DB.DATABASE_NAME'] ?= service.deps.ranger_admin.options.install['audit_db_name']
            options.install['XAAUDIT.DB.USER_NAME'] ?= service.deps.ranger_admin.options.install['audit_db_user']
            options.install['XAAUDIT.DB.PASSWORD'] ?= service.deps.ranger_admin.options.install['audit_db_password']
          when 'ORACLE'
            throw Error 'Ryba does not support ORACLE Based Ranger Installation'
          else
            throw Error "Apache Ranger does not support chosen DB FLAVOUR"
      else
          options.install['XAAUDIT.DB.HOSTNAME'] ?= 'NONE'
          options.install['XAAUDIT.DB.DATABASE_NAME'] ?= 'NONE'
          options.install['XAAUDIT.DB.USER_NAME'] ?= 'NONE'
          options.install['XAAUDIT.DB.PASSWORD'] ?= 'NONE'

## Ambari Configuration

        options.configurations ?= {}
        options.configurations['ranger-hbase-security'] ?= {}
        options.configurations['ranger-hbase-security']['ranger.plugin.hbase.service.name'] ?= options.service_repo.name
        options.configurations['ranger-hbase-security']['ranger.plugin.hbase.policy.rest.url'] ?= options.install['POLICY_MGR_URL']
        options.configurations['ranger-hbase-security']['ranger.plugin.hbase.policy.cache.dir'] ?= "/etc/ranger/#{options.service_repo.name}/policycache"
        options.configurations['ranger-hbase-security']['ranger.plugin.hbase.policy.pollIntervalMs'] ?= "30000"
        options.configurations['ranger-hbase-security']['ranger.plugin.hbase.policy.rest.ssl.config.file'] ?= "#{service.deps.hbase_master[0].options.conf_dir}/ranger-policymgr-ssl.xml"
        options.configurations['ranger-hbase-security']['ranger.plugin.hbase.policy.source.impl'] ?= 'org.apache.ranger.admin.client.RangerAdminRESTClient'

        options.configurations['ranger-hbase-plugin-properties'] ?= {}
        # options.configurations['ranger-hbase-plugin-properties'] ?= merge {}, options.service_repo.configs,
        #   options.install, options.configurations['ranger-hbase-plugin-properties']
        options.configurations['ranger-hbase-plugin-properties']['ranger-hbase-plugin-enabled'] ?= 'Yes' 
        options.configurations['ranger-hbase-plugin-properties']['REPOSITORY_CONFIG_USERNAME'] ?= options.service_repo.configs.username
        options.configurations['ranger-hbase-plugin-properties']['REPOSITORY_CONFIG_PASSWORD'] ?= options.service_repo.configs.password
        options.configurations['ranger-hbase-plugin-properties']['common.name.for.certificate'] ?= options.service_repo.configs['commonNameForCertificate']
        options.configurations['ranger-hbase-plugin-properties']['hadoop.rpc.protection'] ?= options.service_repo.configs['hadoop.rpc.protection']
        options.configurations['ranger-hbase-plugin-properties']['policy_user'] ?= options.service_repo.configs['policy.download.auth.users']
        for k, v of options.install
          if k.indexOf('XAAUDIT') isnt -1
            options.configurations['ranger-hbase-plugin-properties'][k] ?= v

## Plugin - Ranger Admin SSL
configure `policy-mgr-ssl` ambari configuration to make the plugin communicate overs tls with ranger-admin
 
 
        options.ssl = merge {}, service.deps.ssl?.options, options.ssl
        options.configurations['ranger-hbase-policymgr-ssl'] ?= {}
        if service.deps.ranger_admin.options.site['ranger.service.https.attrib.ssl.enabled'] is 'true'
          options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore'] ?= "#{service.deps.hadoop_core.options.ssl.conf_dir}/ranger-hbase-plugin-keystore"
          options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.keystore.password']
          options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.truststore'] ?= "#{service.deps.hadoop_core.options.ssl.conf_dir}/ranger-hbase-plugin-truststore"
          options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password'] ?= service.deps.hadoop_core.options.ssl_server['ssl.server.truststore.password']
          options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.credential.file'] ?= "jceks://file/etc/ranger/#{options.service_repo.name}/cred.jceks"
          options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.credential.file'] ?=  "jceks://file/etc/ranger/#{options.service_repo.name}/cred.jceks"

## Plugin Audit

        options.configurations['ranger-hbase-audit'] ?= {}
        options.configurations['ranger-hbase-audit']['xasecure.audit.is.enabled'] ?= 'true'
        # audit to hdfs
        options.configurations['ranger-hbase-audit']['xasecure.audit.destination.hdfs'] ?= options.install['XAAUDIT.HDFS.IS_ENABLED']
        options.configurations['ranger-hbase-audit']['xasecure.audit.destination.hdfs.batch.filespool.dir'] ?= options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR']
        options.configurations['ranger-hbase-audit']['xasecure.audit.destination.hdfs.dir'] ?= options.install['XAAUDIT.HDFS.HDFS_DIR']
        # audit to solr
        options.configurations['ranger-hbase-audit']['xasecure.audit.destination.solr'] ?= options.install['XAAUDIT.SOLR.IS_ENABLED']
        options.configurations['ranger-hbase-audit']['xasecure.audit.destination.solr.batch.filespool.dir'] ?= options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR']
        options.configurations['ranger-hbase-audit']['xasecure.audit.destination.solr.zookeepers'] ?= options.install['XAAUDIT.SOLR.ZOOKEEPER']
        options.configurations['ranger-hbase-audit']['xasecure.audit.solr.solr_url'] ?= options.install['XAAUDIT.SOLR.URL']
        ## JAAS in memory configuration
        options.configurations['ranger-hbase-audit']['xasecure.audit.destination.solr.force.use.inmemory.jaas.config'] ?= 'true'
        options.configurations['ranger-hbase-audit']['xasecure.audit.jaas.inmemory.loginModuleName'] ?= 'com.sun.security.auth.module.Krb5LoginModule'
        options.configurations['ranger-hbase-audit']['xasecure.audit.jaas.inmemory.loginModuleControlFlag'] ?= 'required'
        options.configurations['ranger-hbase-audit']['xasecure.audit.jaas.inmemory.Client.option.useKeyTab'] ?= 'true'
        options.configurations['ranger-hbase-audit']['xasecure.audit.jaas.inmemory.Client.option.debug'] ?= 'true'
        options.configurations['ranger-hbase-audit']['xasecure.audit.jaas.inmemory.Client.option.doNotPrompt'] ?= 'yes'
        options.configurations['ranger-hbase-audit']['xasecure.audit.jaas.inmemory.Client.option.storeKey'] ?= 'yes'
        options.configurations['ranger-hbase-audit']['xasecure.audit.jaas.inmemory.Client.option.serviceName'] ?= 'solr'
        options.configurations['ranger-hbase-audit']['xasecure.audit.jaas.Client.option.keyTab '] ?= service.deps.hbase_master[0].options.hbase_site['hbase.master.kerberos.keytab']
        options.configurations['ranger-hbase-audit']['xasecure.audit.jaas.inmemory.Client.option.keyTab'] ?= service.deps.hbase_master[0].options.hbase_site['hbase.master.kerberos.keytab']
        options.configurations['ranger-hbase-audit']['xasecure.audit.jaas.inmemory.Client.option.principal'] ?= service.deps.hbase_master[0].options.hbase_site['hbase.master.kerberos.principal']
        options.configurations['ranger-hbase-audit']['xasecure.audit.provider.summary.enabled'] ?= 'true'

## Ambari

        #ambari server configuration
        options.post_component = service.instances[0].node.fqdn is service.node.fqdn
        options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
        options.ambari_url ?= service.deps.ambari_server.options.ambari_url
        options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
        options.cluster_name ?= service.deps.ambari_server.options.cluster_name
        options.takeover = service.deps.ambari_server.options.takeover

## Enable Plugin in Ranger Admin

        service.deps.ranger_admin.options.configurations['ranger-env']['ranger-hbase-plugin-enabled'] = 'Yes'

## Enrich HBase Service  with Ranger Properties
For HBase, ranger related properties should be posted before any service is installed or
started, as Ambari required to configuration dictionnaries to exist `ranger-hbase-plugin-properties` 
        
        for srv in service.deps.hbase
          srv.options.configurations ?= {}
          srv.options.configurations['ranger-hbase-security'] ?= merge {}, srv.options.configurations['ranger-hbase-security'], options.configurations['ranger-hbase-security']
          srv.options.configurations['ranger-hbase-plugin-properties'] ?= merge {}, srv.options.configurations['ranger-hbase-plugin-properties'], options.configurations['ranger-hbase-plugin-properties']
          srv.options.configurations['ranger-hbase-policymgr-ssl'] ?= merge {}, srv.options.configurations['ranger-hbase-policymgr-ssl'], options.configurations['ranger-hbase-policymgr-ssl']
          srv.options.configurations['ranger-hbase-audit'] ?= merge {}, srv.options.configurations['ranger-hbase-audit'], options.configurations['ranger-hbase-audit']

## Wait

      options.wait_ranger_admin = service.deps.ranger_admin.options.wait

## Dependencies

    {merge} = require 'nikita/lib/misc'
