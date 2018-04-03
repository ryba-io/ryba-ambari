
## Configure
This modules configures every hadoop plugin needed to enable Ranger. It configures
variables but also inject some function to be executed.

    module.exports = (service) ->
      options = service.options

## Identities

      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'ranger'
      options.group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'ranger'
      options.user.system ?= true
      options.user.comment ?= 'Ranger User'
      options.user.home ?= "/var/lib/#{options.user.name}"
      options.user.gid ?= options.group.name
      options.user.groups ?= 'hadoop'
      
## Environment
      
      # Layout
      options.conf_dir ?= '/etc/ranger/admin/conf'
      options.pid_dir ?= '/var/run/ranger/admin'
      options.log_dir ?= '/var/log/ranger/admin'
      # Misc
      options.clean_logs ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.fqdn ?= service.node.fqdn # Used by Solr embedded

## Kerberos

      options.krb5 ?= {}
      options.krb5.enabled ?= service.deps.hadoop_core.options.core_site['hadoop.security.authentication'] is 'kerberos'
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin = service.deps.krb5_client.options.admin[options.krb5.realm]

## SSL

      options.ssl = merge {}, service.deps.hadoop_core.options.ssl, options.ssl

## Log4j

      options.log4j ?= {}
      options.log4j['log4j.logger.xaaudit.org.apache.ranger.audit.provider.Log4jAuditProvider'] = 'INFO, hdfsAppender'
      options.log4j['log4j.appender.hdfsAppender'] = 'org.apache.log4j.HdfsRollingFileAppender'
      options.log4j['log4j.appender.hdfsAppender.hdfsDestinationDirectory'] = 'hdfs://%hostname%:8020/logs/application/%file-open-time:yyyyMMdd%'
      options.log4j['log4j.appender.hdfsAppender.hdfsDestinationFile'] = '%hostname%-audit.log'
      options.log4j['log4j.appender.hdfsAppender.hdfsDestinationRolloverIntervalSeconds'] = '86400'
      options.log4j['log4j.appender.hdfsAppender.localFileBufferDirectory'] = '/tmp/logs/application/%hostname%'
      options.log4j['log4j.appender.hdfsAppender.localFileBufferFile'] = '%file-open-time:yyyyMMdd-HHmm.ss%.log'
      options.log4j['log4j.appender.hdfsAppender.localFileBufferRolloverIntervalSeconds'] = '15'
      options.log4j['log4j.appender.hdfsAppender.localFileBufferArchiveDirectory'] = '/tmp/logs/archive/application/%hostname%'
      options.log4j['log4j.appender.hdfsAppender.localFileBufferArchiveFileCount'] = '12'
      options.log4j['log4j.appender.hdfsAppender.layout'] = 'org.apache.log4j.PatternLayout'
      options.log4j['log4j.appender.hdfsAppender.layout.ConversionPattern'] = '%d{yy/MM/dd HH:mm:ss} [%t]: %p %c{2}: %m%n'
      options.log4j['log4j.appender.hdfsAppender.encoding'] = 'UTF-8'

# Managed Users

Ranger enable to create users with its REST API. Required user can be specified in the
ranger config and ryba will create them.

User can be External and Internal. Only Internal users can be created from the ranger webui.

      # Ranger Manager Users
      # Dictionnary containing as a key the name of the ranger admin webui users
      # and value and user properties.
      options.users ?= {}
      options.users['ryba'] ?=
        "name": 'ryba'
        "firstName": 'ryba'
        "lastName": 'hadoop'
        "emailAddress": 'ryba@hadoop.ryba'
        "password": 'ryba1234-'
        'userSource': 1
        'userRoleList': ['ROLE_USER']
        'groups': []
        'status': 1
      options.lock = "/etc/ranger/#{Date.now()}"
      # Ranger Admin configuration
      options.current_password ?= 'admin'
      # TODO: wdavidw 10821, rename as admin_password
      options.admin ?= {}
      options.admin.username ?= 'admin'
      options.admin.password ?= 'rangerAdmin123'
      if not (/^.*[a-zA-Z]/.test(options.admin.password) and /^.*[0-9]/.test(options.admin.password) and options.admin.password.length > 8)
       throw Error "new passord's length must be > 8, must contain one alpha and numerical character at lest"
      options.conf_dir ?= '/etc/ranger/admin'
      options.site ?= {}
      options.site['ranger.service.http.enabled'] ?= 'true'
      options.site['ranger.service.http.port'] ?= '6080'
      options.site['ranger.service.https.port'] ?= '6182'
      options.site['ranger.service.host'] ?= options.fqdn
      options.install ?= {}
      options.install['PYTHON_COMMAND_INVOKER'] ?= 'python'
      # Needed starting from 2.5 version to not have problem during setup execution
      options.install['hadoop_conf'] ?= "#{service.deps.hadoop_core.options.conf_dir}"
      options.install['RANGER_ADMIN_LOG_DIR'] ?= "#{options.log_dir}"

# Kerberos

[Starting from 2.5][ranger-upgrade-24-25], Ranger supports Kerberos Authentication for secured cluster

      if options.krb5.enabled
        options.install['spnego_principal'] ?= "HTTP/#{service.node.fqdn}@#{options.krb5.realm}"
        options.install['spnego_keytab'] ?= '/etc/security/keytabs/spnego.service.keytab'
        options.install['token_valid'] ?= '30'
        options.install['cookie_domain'] ?= "#{service.node.fqdn}"
        options.install['cookie_path'] ?= '/'
        options.install['admin_principal'] ?= "rangeradmin/#{service.node.fqdn}@#{options.krb5.realm}"
        options.install['admin_keytab'] ?= '/etc/security/keytabs/ranger.admin.service.keytab'
        options.install['lookup_principal'] ?= "rangerlookup/#{service.node.fqdn}@#{options.krb5.realm}"
        options.install['lookup_keytab'] ?= "/etc/security/keytabs/ranger.lookup.service.keytab"
        # equivalent to ranger-admin-site properties
        options.site['ranger.admin.kerberos.principal'] ?= options.install['admin_principal']
        options.site['ranger.admin.kerberos.keytab'] ?= options.install['admin_keytab']
        options.site['ranger.lookup.kerberos.principal'] ?= options.install['lookup_principal']
        options.site['ranger.lookup.kerberos.keytab'] ?= options.install['lookup_keytab']
        options.site['ranger.spnego.kerberos.principal'] ?= options.install['spnego_principal']
        options.site['ranger.spnego.kerberos.keytab'] ?= options.install['spnego_keytab']
        options.site['ranger.admin.kerberos.cookie.domain'] ?= options.install['cookie_domain']
        options.site['ranger.admin.kerberos.cookie.path'] ?= options.install['cookie_path']
        if options.solr_type in ['cloud','cloud_docker']
          #Configuring in memory jaas property for ranger to sol
          options.site['xasecure.audit.destination.solr.force.use.inmemory.jaas.config'] ?= 'true'
          options.site['xasecure.audit.jaas.inmemory.loginModuleName'] ?= 'com.sun.security.auth.module.Krb5LoginModule'
          options.site['xasecure.audit.jaas.inmemory.loginModuleControlFlag'] ?= 'required'
          options.site['xasecure.audit.jaas.inmemory.Client.option.useKeyTab'] ?= 'true'
          options.site['xasecure.audit.jaas.inmemory.Client.option.debug'] ?= 'true'
          options.site['xasecure.audit.jaas.inmemory.Client.option.doNotPrompt'] ?= 'yes'
          options.site['xasecure.audit.jaas.inmemory.Client.option.storeKey'] ?= 'yes'
          options.site['xasecure.audit.jaas.inmemory.Client.option.serviceName'] ?= 'solr'
          options.site['xasecure.audit.jaas.inmemory.Client.option.keyTab'] ?= options.install['admin_keytab']
          options.site['xasecure.audit.jaas.inmemory.Client.option.principal'] ?= options.install['admin_principal']

# Audit Storage

Ranger can store  audit to different storage type.
- HDFS ( Long term and scalable storage)
- SOLR ( short term storage & Ranger WEBUi)
- DB Flavor ( mid term storage & Ranger WEBUi)
We do not advice to use DB Storage as it is not efficient to query when it grows up.
Hortonworks recommandations are to enable SOLR and HDFS Storage.

      options.install['audit_store'] ?= 'solr'
      options.site['ranger.audit.source.type'] ?= options.install['audit_store']

## Solr Audit Configuration

Here SOLR configuration is discovered and ranger admin is set up.

Ryba support both Solr Cloud mode and Solr Standalone installation. 

The `solr_type` option designates the type of solr service (ie standalone, embedded, cloud, cloud indocker)
used for Ranger.
The type requires differents instructions/configuration for ranger plugin audit to work.
- Solr Standalone `ryba/solr/standalone`
  Ryba default. You need to set `ryba/solr/standalone` on one host.
- Solr Standalone embedded
  No need to have `ryba/solr/standalone` on one host, Solr will be installed on the same host as Ranger Admin.
  Change property `solr_type` to `embedded` to use it.
- Solr Cloud `ryba/solr/cloud`
  Changes  property `solr_type` to `cloud` and deploy `ryba/solr/cloud`
  module on at least one host.
- Solr Cloud on docker `ryba/solr/cloud_docker`
  Changes  property `solr_type` to `cloud_docker`.
  Important:
    For this to work you need to deploy `ryba/solr/cloud_docker` module on at least on host.
    AND you also need to setup a solr cluster in your configuration, for ryba being able to configure
      ranger with this cluster. 
    Ryba configures Ranger by using one of the cluster available
    You can configure it by using `config.ryba.solr.cloud_docker.clusters` property.
    Ryba will search by default for an instance named `ranger_cluster` which is set
    by the property `cluster_name`.
    An example is available in [the ryba-cluster config file][ryba-cluster-conf].

Note July 2016:
The previous properties works only with (HDP 2.4) `solr.BasicAuthPlugin` (in solr cluster config).
And it is configured by Ryba only in ryba/solr/cloud_docker installation.

If no `ryba/solr/*` is configured Ranger admin deploys a `ryba/solr/standalone` 
on the same host than `ryba/ranger/admin` module.

## Example

To use the embedded Solr mode, configure ranger-admin as follows:

```json
{ "ranger": {
    "admin": {
      "solr_type": "embedded"
    }
} }
```

If you have configured a Solr Cloud Docker in your cluster, you can configure like this:

```json
{ "ranger": {
    "admin": {
      "solr_type": "cloud_docker"
    }
} }
```

      options.solr_type ?= 'embedded'
      options.solr_client_source ?= service.deps.solr_client.options.source if service.deps.solr_client
      options.solr_client_source = if options.solr_client_source is 'HDP'
      then '/opt/lucidworks-hdpsearch/solr'
      else '/usr/solr/current'      # solr = {}
      solrs_urls = ''
      # solr_ctx = {}
      # Retention period in day to keep audit logs
      options.audit_retention_period ?= '1095' #value in days. default to 3 years.
      options.retention ?=  "+#{options.audit_retention_period}"
      switch options.solr_type
        when 'embedded'
          options.solr ?= {}
          options.solr.group ?= {}
          options.solr.group = name: options.solr.group if typeof options.solr.group is 'string'
          options.solr.group.name ?= 'solr'
          options.solr.group.system ?= true
          options.solr.user ?= {}
          options.solr.user ?= name: options.solr.user if typeof options.solr.user is 'string'
          options.solr.user.name ?= 'solr'
          options.solr.user.gid ?= options.solr.group.name
          options.solr.user.home ?= "/var/lib/#{options.solr.user.name}"
          options.solr.user.system ?= true
          options.solr.user.comment ?= 'Solr User'
          options.solr.user.groups ?= 'hadoop'
          options.solr.fqdn = service.node.fqdn
          options.solr.version ?= '5.5.2'
          options.solr.root_dir ?= '/usr'
          options.solr.install_dir ?= "#{options.solr.root_dir}/solr/#{options.solr.version}"
          options.solr.latest_dir = '/opt/lucidworks-hdpsearch/solr'
          options.solr.pid_dir ?= '/var/run/solr'
          options.solr.log_dir ?= '/var/log/solr'
          options.solr.conf_dir ?= '/etc/solr/conf'
          options.solr.env ?= {}
          options.solr.dir_factory ?= "${solr.directoryFactory:solr.NRTCachingDirectoryFactory}"
          options.solr.lock_type = 'native'
          options.solr.conf_source = "#{__dirname}/../resources/solr/solr_5.xml.j2"
          if options.krb5.enabled
            options.solr.principal ?= "#{options.solr.user.name}/#{service.node.fqdn}@#{options.krb5.realm}"
            options.solr.keytab ?= '/etc/security/keytabs/solr.service.keytab'
          options.solr.ssl = merge options.solr.ssl or {}, service.deps.hadoop_core.options.ssl
          # lucasbak 11102017
          # in HDP 2.5.3 SSL enabled solr sink is not supported
          options.solr.ssl.enabled = false
          options.solr.port ?= if options.solr.ssl.enabled then 19983 else 18983
          options.solr.ssl_truststore_path ?= "#{options.solr.conf_dir}/truststore"
          options.solr.ssl_truststore_pwd ?= 'solr123'
          options.solr.ssl_keystore_path ?= "#{options.solr.conf_dir}/keystore"
          options.solr.ssl_keystore_pwd ?= 'solr123'
          options.solr.env['SOLR_JAVA_HOME'] ?= service.deps.java.options.java_home
          options.solr.env['SOLR_HOST'] ?= service.node.fqdn
          options.solr.env['SOLR_HEAP'] ?= "512m"
          options.solr.env['SOLR_PORT'] ?= "#{options.solr.port}"
          options.solr.env['ENABLE_REMOTE_JMX_OPTS'] ?= 'false'
          if options.solr.ssl.enabled
            options.solr.env['SOLR_SSL_KEY_STORE'] ?= options.solr.ssl_keystore_path
            options.solr.env['SOLR_SSL_KEY_STORE_PASSWORD'] ?= options.solr.ssl_keystore_pwd
            options.solr.env['SOLR_SSL_TRUST_STORE'] ?= options.solr.ssl_truststore_path
            options.solr.env['SOLR_SSL_TRUST_STORE_PASSWORD'] ?= options.solr.ssl_truststore_pwd
            options.solr.env['SOLR_SSL_NEED_CLIENT_AUTH'] ?= 'false'
          options.solr.jre_home ?= service.deps.java.options.jre_home
          solrs_urls = "#{if options.solr.ssl.enabled then 'https://' else 'http://'}#{service.node.fqdn}:#{options.solr.port}/solr/ranger_audits"
          options.install['audit_solr_zookeepers'] ?= 'NONE'
        when 'external'
          options.solr.cluster_config ?= {}
          options.solr.cluster_config.ranger_collection_dir ?= '/tmp/ranger-infra'
          throw Error "Missing Solr options.solr.cluster_config.user property example: solr" unless options.solr.cluster_config.user?
          throw Error "Missing Solr options.solr.cluster_config.ssl_enabled property example: true" unless options.solr.cluster_config.ssl_enabled?
          throw Error "Missing Solr options.solr.cluster_config.hosts: ['master01.ryba', 'master02.ryba']" unless options.solr.cluster_config.hosts?
          throw Error "Missing Solr options.solr.cluster_config.zk_urls: master01.metal.ryba:2181" unless options.solr.cluster_config.zk_urls?
          throw Error "Missing Solr options.solr.cluster_config.zk_connect: master01.metal.ryba:2181/solr_infra" unless options.solr.cluster_config.zk_connect?
          throw Error "Missing Solr options.solr.cluster_config.master: master01.metal.ryba" unless options.solr.cluster_config.master?
          throw Error "Missing Solr options.solr.cluster_config.port: 8983" unless options.solr.cluster_config.port?
          throw Error "Missing Solr options.solr.cluster_config.authentication: kerberos" unless options.solr.cluster_config.authentication?
          if options.solr.cluster_config.authentication? is 'kerberos'
            throw Error "Missing Solr options.solr.cluster_config.admin_principal: " unless options.solr.cluster_config.admin_principal?
            throw Error "Missing Solr options.solr.cluster_config.admin_password: " unless options.solr.cluster_config.admin_password?
          options.solr.cluster_config.collection ?=
            'name': 'ranger_audits'
            'numShards': options.solr.cluster_config['hosts'].length
            'replicationFactor': options.solr.cluster_config['hosts'].length-1
            'maxShardsPerNode': options.solr.cluster_config['hosts'].length
            'collection.configName': 'ranger_audits'
          options.install['audit_solr_urls'] ?= options.solr.cluster_config.hosts.map( (host) ->
              "#{if options.solr.cluster_config.ssl_enabled then 'https://' else 'http://'}#{host}:#{options.solr.cluster_config.port}")
          options.install['audit_solr_zookeepers'] ?= options.solr.cluster_config.zk_connect
        # when 'cloud'
        #   throw Error 'No Solr Cloud Server configured' unless service.deps.solr_cloud.length > 0
        #     # options.solr_admin_user ?= 'solr'
        #     # options.solr_admin_password ?= 'SolrRocks' #Default
        #   options.solr.ssl = service.deps.solr_cloud[0].options.ssl
        #   options.solr.cluster_config ?=
        #     user: service.deps.solr_cloud[0].options.user
        #     atlas_collection_dir: "#{options.user.home}/ranger-infra"
        #     hosts: service.deps.solr_cloud.map (srv) -> srv.node.fqdn
        #     zk_urls: service.deps.solr_cloud[0].options.zkhosts
        #     zk_connect: service.deps.solr_cloud[0].options.zk_connect
        #     master: service.deps.solr_cloud[0].node.fqdn
        #     port: service.deps.solr_cloud[0].options.port
        #     authentication: service.deps.hadoop_core.options.core_site['hadoop.security.authentication']
        #     ssl_enabled: service.deps.solr_cloud[0].options.ssl.enabled
        #   if service.deps.hadoop_core.options.core_site['hadoop.security.authentication'] is 'kerberos'
        #     options.solr.cluster_config.admin_principal = service.deps.solr_cloud[0].options.admin_principal
        #     options.solr.cluster_config.admin_password  = service.deps.solr_cloud[0].options.admin_password
        #   urls = service.deps.solr_cloud[0].options.zk_connect.split(',').map( (host) -> "#{host}/#{service.deps.solr_cloud[0].options.zk_node}").join(',')
        #   options.install['audit_solr_urls'] ?= options.solr.cluster_config.hosts.map( (host) ->
        #       "#{if options.solr.cluster_config.ssl_enabled then 'https://' else 'http://'}#{host}:#{options.solr.cluster_config.port}")
        #   options.install['audit_solr_zookeepers'] = 'NONE'
          # break;

## Solr Audit Database Bootstrap

Create the `ranger_audits` collection('cloud')/core('standalone').

      if options.install['audit_store'] is 'solr'
        options.install['audit_solr_urls'] ?= solrs_urls
        options.install['audit_solr_user'] ?= 'ranger'
        options.install['audit_solr_password'] ?= 'ranger123'
        # options.install['audit_solr_zookeepers'] = 'NONE'

When Basic authentication is used, the following property can be set to add 
users to solr `cluster_config.ranger.solr_users`:
  -  An object describing all the users used by the different plugins which will
  write audit to solr.
  - By default if no user are provided, Ryba configure only one user named ranger
  to audit to solr.

Example:

```cson
ranger.admin.cluster_config.ranger.solr_users =
  name: 'my_plugin_user'
  secret: 'my_plugin_password'
```

        options.solr_users ?= []
        if options.solr_users.length is 0
          options.solr_users.push {
            name: "#{options.install['audit_solr_user']}"
            secret:"#{options.install['audit_solr_password']}"
          }

## Ranger Admin SSL & Credentials


Configure SSL for Ranger policymanager (webui).

      options.site['ranger.service.https.attrib.ssl.enabled'] ?= 'true'
      if options.site['ranger.service.https.attrib.ssl.enabled'] is 'true'
        #credential store
        throw Error 'No password provided for credential store' unless options.credential_password?
        options.site['ranger.credential.provider.path'] ?= '/etc/ranger/admin/rangeradmin.jceks'
        #advanced ranger-admin-site
        options.site['ranger.https.attrib.keystore.file'] ?= "/etc/ranger/admin/conf/ranger-admin-keystore.jks"
        # options.site['ranger.https.attrib.keystore.pass'] ?= "/etc/ranger/admin/conf/ranger-admin-keystore.jks"
        options.site['ranger.service.https.attrib.keystore.pass'] ?= options.credential_password
        options.site['ranger.service.https.attrib.keystore.keyalias'] ?= service.node.hostname
        options.site['ranger.service.https.attrib.clientAuth'] ?= 'false'
        #custom ranger-admin-site
        options.site['ranger.service.https.attrib.keystore.file'] ?= options.site['ranger.https.attrib.keystore.file']
        options.site['ranger.service.https.attrib.keystore.credential.alias'] ?= 'rangeradmin.keystore'
        # options.site['ranger.https.attrib.keystore.file'] ?= "/etc/security/serverKeys/ranger-admin-keystore"
        # since ranger 0.7 HDP 2.6.1 Ranger accepts trsutstore properties
        options.site['ranger.truststore.file'] ?= '/etc/ranger/admin/conf/truststore'
        options.site['ranger.truststore.password'] ?= 'ryba123'
        options.site['ranger.truststore.alias'] ?= 'rangeradmin.truststore'
        options.install['policymgr_https_keystore_file'] ?= options.site['ranger.https.attrib.keystore.file']
        options.install['policymgr_https_keystore_password'] ?= options.site['ranger.service.https.attrib.keystore.pass']
        # options.site['ranger.truststore.file'] ?= '/etc/ranger/admin/conf/ranger-admin-keystore.jks'
      #       javax_net_ssl_keyStore=
      # javax_net_ssl_keyStorePassword=
      # javax_net_ssl_trustStore=
      # javax_net_ssl_trustStorePassword=



# Ranger Admin Databases

Configures the Ranger WEBUi (policymanager) database. For now only mysql is supported.

      options.db ?= {}
      options.db.engine ?= service.deps.db_admin.options.engine
      options.db = merge {}, service.deps.db_admin.options[options.db.engine], options.db
      switch options.db.engine
        when 'mysql', 'mariadb'
          options.install['DB_FLAVOR'] ?= 'MYSQL' # we support only mysql for now
          options.install['SQL_CONNECTOR_JAR'] ?= '/usr/hdp/current/ranger-admin/lib/mysql-connector-java.jar'
          # not setting these properties on purpose, we manage manually databases inside mysql
          options.install['db_root_user'] = options.db.admin_username
          options.install['db_root_password'] ?= options.db.admin_password
          if not options.install['db_root_user'] and not options.install['db_root_password']
          then throw Error "account with privileges for creating database schemas and users is required"
          options.install['db_host'] ?= options.db.host
          # Ranger Policy Database
          throw Error "mysql host not specified" unless options.install['db_host']
          options.install['db_name'] ?= 'ranger'
          options.install['db_user'] ?= 'rangeradmin'
          throw Error 'Required Options: install.db_password' unless options.install['db_password']
          options.install['audit_db_name'] ?= 'ranger_audit'
          options.install['audit_db_user'] ?= 'rangerlogger'
          throw Error 'Required Options: install.audit_db_password' unless options.install['audit_db_password']
        else throw Error 'For now only mysql engine is supported'
      # fix ranger.jpa.jdbc.url
      options.site['ranger.jpa.jdbc.credential.alias'] ?= 'rangeradmin.db'
      options.site['ranger.jpa.jdbc.url'] ?= "jdbc:mysql://#{options.db.host}:#{options.db.port}/#{options.install['db_name']}"
      options.site['ranger.jpa.jdbc.driver'] ?= options.db.java.driver
      options.site['ranger.jpa.jdbc.user'] ?= options.install['db_user']
      options.site['ranger.jpa.jdbc.password'] ?= options.install['db_password']



# Ranger Admin Policymanager Access

Defined how Ranger authenticates users (the xusers)  to the webui. By default
only users created within the webui are allowed.

      protocol = if options.site['ranger.service.https.attrib.ssl.enabled'] == 'true' then 'https' else 'http'
      port = options.site["ranger.service.#{protocol}.port"]
      options.install['policymgr_external_url'] ?= "#{protocol}://#{service.node.fqdn}:#{port}"
      options.site['ranger.externalurl'] ?= options.install['policymgr_external_url']
      options.install['policymgr_http_enabled'] ?= 'true'
      options.install['unix_user'] ?= options.user.name
      options.install['unix_group'] ?= options.group.name
      #Policy Admin Tool Authentication
      # NONE enables only users created within the Policy Admin Tool 
      options.install['authentication_method'] ?= 'NONE'
      unix_props = ['remoteLoginEnabled','authServiceHostName','authServicePort']
      active_dir_props = ['xa_ldap_ad_domain','xa_ldap_ad_url']
      switch options.install['authentication_method']
        when 'UNIX'
          options.site['ranger.authentication.method'] ?= 'UNIX'
          throw Error "missing property: #{prop}" unless options.install[prop] for prop in unix_props
        when 'LDAP'
          options.site['ranger.authentication.method'] ?= 'LDAP'
          # if !options.site['ranger.ldap.base.dn']
            # [opldp_srv_ctx] = service.deps.'masson/core/openldap_server'
            # throw Error 'no openldap server configured' unless opldp_srv_ctx?
            # {openldap_server} = opldp_srv_ctx.config
            # options.install['xa_ldap_url'] ?= "#{openldap_server.uri}"
            # options.install['xa_ldap_userDNpattern'] ?= "cn={0},ou=users,#{openldap_server.suffix}"
            # options.install['xa_ldap_groupSearchBase'] ?=  "ou=groups,#{openldap_server.suffix}"
            # options.install['xa_ldap_groupSearchFilter'] ?= "(uid={0},ou=groups,#{openldap_server.suffix})"
            # options.install['xa_ldap_groupRoleAttribute'] ?= 'cn'
            # options.install['xa_ldap_userSearchFilter'] ?= '(uid={0})'
            # options.install['xa_ldap_base_dn'] ?= "#{openldap_server.suffix}"
            # options.install['xa_ldap_bind_dn'] ?= "#{openldap_server.root_dn}"
            # options.install['xa_ldap_bind_password'] ?= "#{openldap_server.root_password}"
          throw Error "missing property: #{prop}" unless options.install[prop] for prop in [
            'ranger.ldap.base.dn'
            'ranger.ldap.bind.dn'
            'ranger.ldap.bind.password'
            'ranger.ldap.user.searchfilter'
            'ranger.ldap.user.dnpattern'
            'ranger.ldap.default.role'
            'ranger.ldap.group.searchbase'
            'ranger.ldap.group.searchfilter'
            'ranger.ldap.group.roleattribute'
            'ranger.ldap.referral'
            'ranger.ldap.url'
          ]
        when 'ACTIVE_DIRECTORY'
          throw Error "missing property: #{prop}" unless options.install[prop] for prop in active_dir_props
        when 'NONE'
          options.site['ranger.authentication.method'] ?= 'NONE'
          break;
        else
          throw Error 'selected authentication_method is not supported by Ranger'


## Ranger Environment

      options.heap_size ?= options.heapsize ?= '1024m'
      options.opts ?= {}
      # options.opts['javax.net.ssl.trustStore'] ?= '/etc/hadoop/conf/truststore'
      # options.opts['javax.net.ssl.trustStorePassword'] ?= 'ryba123'

## Ranger PLUGINS

Plugins are HDP Packages which once enabled, allow Ranger to manage ACL for services.
For now Ranger support policy management for:

- HDFS
- YARN
- HBASE
- KAFKA
- Hive 
- SOLR 

Plugins should be configured before the service is started and/or configured.
Ryba injects function to the different contexts.

      options.plugins ?= {}
      options.plugins.principal ?= "#{options.user.name}@#{options.krb5.realm}"
      options.plugins.password ?= 'rangerAdmin123'

## Wait

      options.wait_solr ?= switch options.solr_type
        when 'external' then  options.solr.cluster_config['hosts'].map (host) ->
          host: host, port: options.solr.cluster_config['port']
      options.wait_krb5_client = service.deps.krb5_client.options.wait
      options.wait = {}
      options.wait.http = {}
      options.wait.http.username = 'admin'
      options.wait.http.password = options.admin.password
      options.wait.http.url = "#{options.install['policymgr_external_url']}/service/users/1"

## Ambari Configuration

      options.configurations ?= {}
      # ranger-env
      options.configurations['ranger-env'] ?= {}
      options.configurations['ranger-env']['ranger_user'] ?= options.user.name
      options.configurations['ranger-env']['ranger_group'] ?= options.group.name
      options.configurations['ranger-env']['ranger_pid_dir'] ?= options.pid_dir
      # options.configurations['ranger-env']['ranger_usersync_log_dir'] ?= service.deps.ranger_usersync.options.log_dir
      options.configurations['ranger-env']['ranger_admin_log_dir'] ?= options.log_dir
      options.configurations['ranger-env']['create_db_dbuser'] ?= 'false'
      options.configurations['ranger-env']['admin_username'] ?= 'admin'
      options.configurations['ranger-env']['admin_password'] ?= options.admin.password
      options.configurations['ranger-env']['xasecure.audit.destination.hdfs'] ?= 'true'
      options.configurations['ranger-env']['xasecure.audit.destination.hdfs.dir'] ?= "#{service.deps.hdfs_nn[0].options.core_site['fs.defaultFS']}/ranger/audit"
      options.configurations['ranger-env']['xasecure.audit.destination.solr'] ?= "true"
      options.configurations['ranger-env']['ranger_solr_config_set'] ?= "ranger_audits"
      options.configurations['ranger-env']['ranger_solr_collection_name'] ?= "ranger_audits"
      options.configurations['ranger-env']['ranger_solr_shards'] ?= "1"
      options.configurations['ranger-env']['ranger_solr_replication_factor'] ?= "1"
      switch options.solr_type
        when 'embedded'
          options.configurations['ranger-env']['is_solrCloud_enabled'] ?= "false"
          options.site['ranger.audit.solr.urls'] ?= options.install['audit_solr_urls']
          
        else
          options.configurations['ranger-env']['is_solrCloud_enabled'] ?= "true"
          options.configurations['ranger-env']['is_external_solrCloud_enabled'] ?= "true"
          options.configurations['ranger-env']['is_external_solrCloud_kerberos'] ?= 'true'
      options.configurations['ranger-env']['ranger-hdfs-plugin-enabled'] ?= 'No'#No
      options.configurations['ranger-env']['ranger-yarn-plugin-enabled'] ?= 'No'
      options.configurations['ranger-env']['ranger-hbase-plugin-enabled'] ?= 'No'
      options.configurations['ranger-env']['ranger-hive-plugin-enabled'] ?= 'No'
      options.configurations['ranger-env']['ranger-kafka-plugin-enabled'] ?= 'No'
      options.configurations['ranger-env']['ranger-knox-plugin-enabled'] ?= 'No'
      options.configurations['ranger-env']['ranger-atlas-plugin-enabled'] ?= 'No'
      options.configurations['ranger-env']['ranger-stop-plugin-enabled'] ?= 'No'
      options.configurations['ranger-env']['xml_configurations_supported'] ?= 'true'
      #admin-site

## Ranger Usersync

      options.configurations['ranger-ugsync-site'] ?= {}
      options.configurations['ranger-ugsync-site']['ranger.usersync.enabled'] ?=  "false"
      options.configurations['ranger-ugsync-site']['ranger.usersync.credstore.filename'] ?=  "/usr/hdp/current/ranger-usersync/conf/ugsync.jceks"
      options.configurations['ranger-ugsync-site']['ranger.usersync.filesource.file'] ?=  "/tmp/usergroup.txt"
      options.configurations['ranger-ugsync-site']['ranger.usersync.filesource.text.delimiter'] ?=  ","
      options.configurations['ranger-ugsync-site']['ranger.usersync.group.memberattributename'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.group.nameattribute'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.group.objectclass'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.group.search.first.enabled'] ?=  "false"
      options.configurations['ranger-ugsync-site']['ranger.usersync.group.searchbase'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.group.searchenabled'] ?=  "false"
      options.configurations['ranger-ugsync-site']['ranger.usersync.group.searchfilter'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.group.searchscope'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.group.usermapsyncenabled'] ?=  "true"
      options.configurations['ranger-ugsync-site']['ranger.usersync.kerberos.keytab'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.kerberos.principal'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.keystore.file'] ?=  "/usr/hdp/current/ranger-usersync/conf/unixauthservice.jks"
      options.configurations['ranger-ugsync-site']['ranger.usersync.keystore.password'] ?=  "ryba123"
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.bindalias'] ?=  "testldapalias"
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.binddn'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.bindkeystore'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.groupname.caseconversion'] ?=  "none"
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.ldapbindpassword'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.referral'] ?=  "ignore"
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.searchBase'] ?=  "dc=hadoop,dc=apache,dc=org"
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.url'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.user.groupnameattribute'] ?=  "memberof, ismemberof"
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.user.nameattribute'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.user.objectclass'] ?=  "person"
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.user.searchbase'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.user.searchfilter'] ?=  ""
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.user.searchscope'] ?=  "sub"
      options.configurations['ranger-ugsync-site']['ranger.usersync.ldap.username.caseconversion'] ?=  "none"
      options.configurations['ranger-ugsync-site']['ranger.usersync.logdir'] ?=  "{{usersync_log_dir}}"
      options.configurations['ranger-ugsync-site']['ranger.usersync.pagedresultsenabled'] ?=  "true"
      options.configurations['ranger-ugsync-site']['ranger.usersync.pagedresultssize'] ?=  "500"
      options.configurations['ranger-ugsync-site']['ranger.usersync.passwordvalidator.path'] ?=  "./native/credValidator.uexe"
      options.configurations['ranger-ugsync-site']['ranger.usersync.policymanager.baseURL'] ?=  "{{ranger_external_url}}"
      options.configurations['ranger-ugsync-site']['ranger.usersync.policymanager.maxrecordsperapicall'] ?=  "1000"
      options.configurations['ranger-ugsync-site']['ranger.usersync.policymanager.mockrun'] ?=  "false"
      options.configurations['ranger-ugsync-site']['ranger.usersync.policymgr.alias'] ?=  'ranger'
      options.configurations['ranger-ugsync-site']['ranger.usersync.policymgr.keystore'] ?=  "/usr/hdp/current/ranger-usersync/conf/ugsync.jceks"
      options.configurations['ranger-ugsync-site']['ranger.usersync.policymgr.username'] ?=  'ranger'
      options.configurations['ranger-ugsync-site']['ranger.usersync.port'] ?=  "5151"
      options.configurations['ranger-ugsync-site']['ranger.usersync.sink.impl.class'] ?=  "org.apache.ranger.unixusersync.process.PolicyMgrUserGroupBuilder"
      options.configurations['ranger-ugsync-site']['ranger.usersync.sleeptimeinmillisbetweensynccycle'] ?=  "60000"
      options.configurations['ranger-ugsync-site']['ranger.usersync.source.impl.class'] ?=  "org.apache.ranger.unixusersync.process.UnixUserGroupBuilder"
      options.configurations['ranger-ugsync-site']['ranger.usersync.ssl'] ?=  "true"
      options.configurations['ranger-ugsync-site']['ranger.usersync.truststore.file'] ?=  "/usr/hdp/current/ranger-usersync/conf/mytruststore.jks"
      options.configurations['ranger-ugsync-site']['ranger.usersync.truststore.password'] ?=  "ryba123"
      options.configurations['ranger-ugsync-site']['ranger.usersync.unix.group.file'] ?=  "/etc/group"
      options.configurations['ranger-ugsync-site']['ranger.usersync.unix.minUserId'] ?=  "500"
      options.configurations['ranger-ugsync-site']['ranger.usersync.unix.password.file'] ?=  "/etc/passwd"
      options.configurations['ranger-ugsync-site']['ranger.usersync.user.searchenabled'] ?=  "false"

## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name


## Dependencies

    quote = require 'regexp-quote'
    {merge} = require 'nikita/lib/misc'

[ranger-2.4.0]:(http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_installing_manually_book/content/configure-the-ranger-policy-administration-authentication-moades.html)
[ranger-ssl]:(https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_Security_Guide/content/configure_non_ambari_ranger_ssl.html) 
[ranger-ldap]:(https://community.hortonworks.com/articles/16696/ranger-ldap-integration.html)
[ranger-api-object]:(https://community.hortonworks.com/questions/10826/rest-api-url-to-configure-ranger-objects.html)
[ranger-solr]:(https://community.hortonworks.com/articles/15159/securing-solr-collections-with-ranger-kerberos.html)
[hdfs-repository]: (http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_Ranger_User_Guide/content/hdfs_repository.html)
[hdfs-repository-0.4.1]:(https://cwiki.apache.org/confluence/display/RANGER/REST+APIs+for+Policy+Management?src=contextnavpagetreemode)
[user-guide-0.5]:(https://cwiki.apache.org/confluence/display/RANGER/Apache+Ranger+0.5+-+User+Guide)
[ryba-cluster-conf]: https://github.com/ryba-io/ryba-cluster/blob/master/conf/config.coffee
[ranger-upgrade-24-25]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.0/bk_command-line-upgrade/content/upgrade-ranger_24.html
