
# Ambari Server Configuration

## Minimal Example

```json
{ "config": {
  "admin_password": "MySecret"
  "db": {
    "password": "MySecret"
  }
} }
```

## Database Encryption

```json
{ "config": {
  "master_key": "MySecret",
} }
```

## LDAP Connection

```json
{ "config": {
  "client.security": "ldap",
  "authentication.ldap.useSSL": true,
  "authentication.ldap.primaryUrl": "master3.ryba:636",
  "authentication.ldap.baseDn": "ou=users,dc=ryba",
  "authentication.ldap.bindAnonymously": false,
  "authentication.ldap.managerDn": "cn=admin,ou=users,dc=ryba",
  "authentication.ldap.managerPassword": "XXX",
  "authentication.ldap.usernameAttribute": "cn"
} }
```

    module.exports = (service) ->
      options = service.options

      options.group = merge service.deps.ambari_server_local.options.group, options.group
      options.user = merge service.deps.ambari_server_local.options.user, options.user
      options.test_user = merge service.deps.ambari_server_local.options.test_user, options.test_user
      options.test_group = merge service.deps.ambari_server_local.options.test_group, options.test_group
      options.analyzer_user = merge service.deps.ambari_server_local.options.analyzer_user, options.analyzer_user
      options.analyzer_group = merge service.deps.ambari_server_local.options.analyzer_group, options.analyzer_group
      options.explorer_user = merge service.deps.ambari_server_local.options.explorer_user, options.explorer_user
      options.explorer_group = merge service.deps.ambari_server_local.options.explorer_group, options.explorer_group

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.conf_dir ?= '/etc/ambari-server/conf'
      options.sudo ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.master_key ?= null
      options.admin ?= {}
      options.krb5 ?= merge {}, service.deps.ambari_server_local.options.krb5, options.krb5
      options.racks ?= {}

## Client Rest API Url

      options.ambari_url ?= service.deps.ambari_server_local.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server_local.options.admin_password
      options.takeover ?= service.deps.ambari_server_local.options.takeover
      options.baremetal ?= service.deps.ambari_server_local.options.baremetal
      throw Error 'Required options.cluster_name for provisioning' unless options.cluster_name
      throw Error 'Required options.stack_name  (HDP) for provisioning' unless options.stack_name
      throw Error 'Required options.stack_version  (2.6) for provisioning' unless options.stack_version

## Config cluster-env

      options.configurations ?= {}
      #tells ambari that packages are already installed
      service.deps.ambari_server_local.options.config['packages.pre.installed'] ?= true
      options.cluster_env_global_properties ?=
        'override_uid': 'true'
        'smokeuser': options.test_user.name
        'host_sys_prepped': 'false'
        'sysprep_skip_create_users_and_groups': 'true'
        "security_enabled" : "true"
        "user_group" : "hadoop"
        "agent_mounts_ignore_list" : ""
        "alerts_repeat_tolerance" : "1"
        "fetch_nonlocal_groups" : "false"
        "ignore_groupsusers_create" : "false"
        "ignore_bad_mounts" : "false"
        "manage_dirs_on_root" : "true"
        "managed_hdfs_resource_property_names" : ""
        "one_dir_per_partition" : "false"
        "recovery_enabled" : "true"
        "recovery_lifetime_max_count" : "1024"
        "recovery_max_count" : "6"
        "recovery_retry_interval" : "5"
        "recovery_type" : "AUTO_START"
        "recovery_window_in_minutes" : "60"
        "sysprep_skip_copy_fast_jar_hdfs" : "false"
        "sysprep_skip_copy_oozie_share_lib_to_hdfs" : "false"
        "sysprep_skip_copy_tarballs_hdfs" : "false"
        "sysprep_skip_setup_jce" : "false"

      options.cluster_env_stack_properties ?=
        "stack_root" : JSON.stringify(HDP: "/usr/hdp")

## Kerberos Service provisioning

      if options.krb5?
        options.test_user.principal ?= "#{options.test_user.name}-#{options.cluster_name}@#{options.krb5.realm}"
        # options.test_user.principal ?= "#{options.test_user.name}-#{options.cluster_name}@#{options.krb5.realm}"
        options.test_user.password ?= 'ambari-qa123'
        options.test_user.keytab ?= '/etc/security/keytabs/smokeuser.headless.keytab'
        options.cluster_env_global_properties['smokeuser_principal_name'] ?= options.test_user.principal
        options.cluster_env_global_properties['smokeuser_keytab'] ?= options.test_user.keytab

## Config krb5-conf

        options.configurations['krb5-conf'] ?= {}
        options.configurations['krb5-conf']['manage_krb5_conf'] ?= 'false'
        throw Error "Missing configurations['krb5-conf']['content']" if options.configurations['krb5-conf']['manage_krb5_conf'] is 'true' and not options.configurations['krb5-conf']['content']?
        options.configurations['krb5-conf']['conf_dir'] ?= '/etc'
        throw Error "Missing krb5-conf domains realms" if ((options.configurations['krb5-conf']['manage_krb5_conf'] is 'true') and !(options.configurations['krb5-conf']['domains']?))
        # options.configurations['krb5-conf']['domains'] ?= '.metal.ryba,metal.ryba'


## Config kerberos-env

        options.configurations['kerberos-env'] ?= {}
        options.configurations['kerberos-env']['encryption_types'] ?= 'aes des3-cbc-sha1 rc4 des-cbc-md5'
        options.configurations['kerberos-env']['realm'] ?= options.krb5.realm
        options.configurations['kerberos-env']['executable_search_paths'] ?= '/usr/bin, /usr/kerberos/bin, /usr/sbin, /usr/lib/mit/bin, /usr/lib/mit/sbin'
        options.configurations['kerberos-env']['kdc_type'] ?= 'mit-kdc'
        options.configurations['kerberos-env']['preconfigure_services'] ?= 'DEFAULT'
        options.configurations['kerberos-env']['service_ckeck_principal_name'] ?= "#{options.cluster_name}"
        options.configurations['kerberos-env']['service_ckeck_retry_count'] ?= '9'
        options.configurations['kerberos-env']['service_ckeck_retry_period_sec'] ?= '15'
        options.configurations['kerberos-env']['create_ambari_principal'] ?= 'false'
        options.configurations['kerberos-env']['manage_identities'] ?= if options.kerberos_managed then 'true' else 'false'
        options.configurations['kerberos-env']['keytab_dir'] ?= '/etc/security/keytabs'
        options.configurations['kerberos-env']['kdc_hosts'] ?= service.deps.krb5_client.options.wait.kdc_tcp.map( (kdc) -> "#{kdc.host}:#{kdc.port}").join(',')
        options.configurations['kerberos-env']['admin_server_host'] ?= options.krb5.admin.admin_server
        options.configurations['kerberos-env']['manage_auth_to_local'] ?= 'false'
        options.configurations['kerberos-env']['install_packages'] ?= 'true'
        options.configurations['kerberos-env']['ad_create_attributes_template'] ?= '{
            "objectClass": ["top", "person", "organizationalPerson", "user"],
            "cn": "$principal_name",
            #if( $is_service )
            "servicePrincipalName": "$principal_name",
            #end
            "userPrincipalName": "$normalized_principal",
            "unicodePwd": "$password",
            "accountExpires": "0",
            "userAccountControl": "66048"
          }'
        options.configurations['kerberos-env']['kdc_create_attributes'] ?= ''
        options.configurations['kerberos-env']['group'] ?= 'ambari-managed-principal'
        options.configurations['kerberos-env']['password_length'] ?= '20'
        options.configurations['kerberos-env']['password_min_lowercase_letters'] ?= '1'
        options.configurations['kerberos-env']['service_check_principal_name'] ?= "${cluster_name|toLower()}-${short_date}"
        options.configurations['kerberos-env']['password_chat_timeout'] ?= '5'
        options.configurations['kerberos-env']['password_min_punctuation'] ?= '1'
        options.configurations['kerberos-env']['set_password_expiry'] ?= 'false'
        options.configurations['kerberos-env']['container_dn'] ?= ''
        options.configurations['kerberos-env']['case_insensitive_username_rules'] ?= 'true'
        options.configurations['kerberos-env']['password_min_whitespace'] ?= '0'
        options.configurations['kerberos-env']['password_min_uppercase_letters'] ?= '1'
        options.configurations['kerberos-env']['password_min_digits'] ?= '1'



## Kerberos Descriptor
Ambari organizes principal and keytab configuration known as Kerberos descriptor,
into three parts:
  - STACK:
    It holds Ambari predefined configurations for principal names
  - USER:
    It is the user defined configurations which could be empty
  - COMPOSITE:
    IT gather all STACK properties overrided with USER's

These properties can be read from the API with the following url:
  `https://master01.metal.ryba:8442/api/v1/clusters/ryba_test/kerberos_descriptors`

However this endpoints are only read only. Tp set the USER part, the Ambari's administrator
must use the [artifacts_data endpoint](https://community.hortonworks.com/content/supportkb/49441/missing-kerberos-descriptor-when-using-the-rest-ap.html)
the POST Object should look like.
```json

{
 "artifact_data" : {
    "identities" : [
      {
        "principal" : {
          "configuration" : null,
          "type" : "service",
          "local_username" : null,
          "value" : "HTTP/_HOST@${realm}"
        },
        "name" : "spnego",
        "keytab" : {
          "owner" : {
            "access" : "r",
            "name" : "root"
          },
          "file" : "${keytab_dir}/spnego.service.keytab",
          "configuration" : null,
          "group" : {
            "access" : "r",
            "name" : "${cluster-env/user_group}"
          }
        },
              {
        "principal" : {
          "configuration" : "cluster-env/smokeuser_principal_name",
          "type" : "user",
          "local_username" : "${cluster-env/smokeuser}",
          "value" : "${cluster-env/smokeuser}-${cluster_name}@${realm}"
        },
        "name" : "smokeuser",
        "keytab" : {
          "owner" : {
            "access" : "r",
            "name" : "${cluster-env/smokeuser}"
          },
          "file" : "${keytab_dir}/smokeuser.headless.keytab",
          "configuration" : "cluster-env/smokeuser_keytab",
          "group" : {
            "access" : "r",
            "name" : "${cluster-env/user_group}"
          }
        }
      }
```

To configure it inside RYBA, `options.identities` dictionnary is used.
The key is the identity name, and the value should contain principal informations.

        options.identities ?= {}
        # options.identities['smokeuser'] ?= {}
        # options.identities['smokeuser']['keytab'] ?= {}
        # options.identities['smokeuser']['keytab']['owner'] ?= {}
        # options.identities['smokeuser']['keytab']['owner']['access'] ?= 'r'
        # options.identities['smokeuser']['keytab']['owner']['name'] ?= '${cluster-env/smokeuser}'
        # options.identities['smokeuser']['keytab']['group'] ?= {}
        # options.identities['smokeuser']['keytab']['group']['access'] ?= 'r'
        # options.identities['smokeuser']['keytab']['group']['name'] ?= '${cluster-env/user_group}'
        # options.identities['smokeuser']['keytab']['file'] ?= "${keytab_dir}/smokeuser.headless.keytab"
        # options.identities['smokeuser']['keytab']['configuration'] ?= 'cluster-env/smokeuser_keytab'
        # options.identities['smokeuser']['principal'] ?= {}
        # options.identities['smokeuser']['principal']['configuration'] ?= 'cluster-env/smokeuser_principal_name'
        # options.identities['smokeuser']['principal']['type'] ?= 'user'
        # options.identities['smokeuser']['principal']['local_username'] ?= '${cluster-env/smokeuser}'
        # options.identities['smokeuser']['principal']['value'] ?= '${cluster-env/smokeuser}@${realm}'
        # options.identities['smokeuser']['name'] ?= 'smokeuser'

        options.post_component = service.instances[0].node.fqdn is service.node.fqdn

## Config Groups

      options.config_groups ?= {}

## Ambari Alert Definitions fix

      options.alerts_definitions ?= {}
      options.alerts_definitions['mapreduce_history_server_process'] ?=
        AlertDefinition:
          source:
            default_port: '19889'
            uri : "{{mapred-site/mapreduce.jobhistory.webapp.https.address}}"

## Wait

      options.ssl ?= merge {}, service.deps.ambari_server[0].options.ssl
      options.truststore ?= service.deps.ambari_server[0].options.truststore
      options.wait = {}
      options.wait_ambari_agent = service.deps.ambari_agent[0].options.wait
      options.wait.rest = for srv in service.deps.ambari_server
        clusters_url: url.format
          protocol: if srv.options.config['api.ssl'] is true
          then 'https'
          else 'http'
          hostname: srv.options.fqdn
          port: if srv.options.config['api.ssl'] is true
          then srv.options.config['client.api.ssl.port']
          else srv.options.config['client.api.port']
          pathname: '/api/v1/clusters'
        oldcred: "admin:#{srv.options.current_admin_password}"
        newcred: "admin:#{srv.options.admin_password}"

## Ambari Agent
Register users to ambari agent's user list.

      for srv in service.deps.ambari_agent
        srv.options.users ?= {}
        srv.options.users['ambari_user'] ?= options.user
        srv.options.users['test_user'] ?= options.test_user
        srv.options.users['analyzer_user'] ?= options.analyzer_user
        srv.options.users['explorer_user'] ?= options.explorer_user
        srv.options.groups ?= {}
        srv.options.groups['ambari_group'] ?= options.ambari_group
        srv.options.groups['test_group'] ?= options.test_group
        srv.options.groups['analyzer_group'] ?= options.analyzer_group
        srv.options.groups['explorer_group'] ?= options.explorer_group

## Services

      options.services ?= {}
      options.groups ?= []

## ZOOKEEPER Service

      if service.deps.zookeeper_server?.length > 0
        for srv in service.deps.zookeeper_server
          continue unless srv.options?.groups?
          for name in srv.options.groups
            options.config_groups ?= {}
            options.config_groups[name] ?= {}
            options.config_groups[name]['hosts'] ?= []
            options.config_groups[name]['hosts'].push srv.node.fqdn unless options.config_groups[name]['hosts'].indexOf(srv.node.fqdn) > -1
        options.configurations['zoo.cfg'] ?= {}
        options.configurations['zookeeper-env'] ?= {}
        options.services['ZOOKEEPER'] ?= {}
        options.services['ZOOKEEPER']['ZOOKEEPER_SERVER'] ?= {}
        options.services['ZOOKEEPER']['ZOOKEEPER_SERVER']['hosts'] ?= service.deps.zookeeper_server.map (srv) -> srv.node.fqdn
        exports.enrich_config service.deps.zookeeper_server[0].options.config, options.configurations['zoo.cfg']
        options.zookeeper ?= {}
        options.zookeeper.retention ?= service.deps.zookeeper_server[0].options.retention
        options.zookeeper.purge ?= service.deps.zookeeper_server[0].options.purge
        options.zookeeper_user ?= service.deps.zookeeper_server[0].options.user

## Ambari Infra Service

      if service.deps.ambari_infra_service?.length > 0
        options.ambari_infra = true
        options.configurations['infra-solr-env'] ?= {}
        options.services['AMBARI_INFRA'] ?= {}
        if service.deps.ambari_infra_instance?.length > 0
          options.services['AMBARI_INFRA']['INFRA_SOLR'] ?= {}
          options.services['AMBARI_INFRA']['INFRA_SOLR']['hosts'] ?= service.deps.ambari_infra_instance.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.ambari_infra_service[0].options.configurations['infra-solr-env'], options.configurations['infra-solr-env']
          exports.enrich_config service.deps.ambari_infra_instance[0].options.configurations['infra-solr-env'], options.configurations['infra-solr-env']


## RANGER Service

      if service.deps.ranger_hdpadmin?.length > 0
        options.services['RANGER'] ?= {}
        options.configurations['ranger-env'] ?= {}
        options.configurations['ranger-solr-env'] ?= {}
        options.configurations['ranger-admin-site'] ?= {}
        options.configurations['ranger-ugsync-site'] ?= {}
        options.configurations['admin-properties'] ?= {}
        exports.enrich_config service.deps.ranger_hdpadmin[0].options.configurations['ranger-env'], options.configurations['ranger-env']
        exports.enrich_config service.deps.ranger_hdpadmin[0].options.configurations['ranger-solr-env'], options.configurations['ranger-solr-env']
        exports.enrich_config service.deps.ranger_hdpadmin[0].options.configurations['ranger-admin-site'], options.configurations['ranger-admin-site']
        exports.enrich_config service.deps.ranger_hdpadmin[0].options.configurations['ranger-ugsync-site'], options.configurations['ranger-ugsync-site']
        exports.enrich_config service.deps.ranger_hdpadmin[0].options.configurations['admin-properties'], options.configurations['admin-properties']
        options.ranger_db ?= merge {}, service.deps.ranger_hdpadmin[0].options.db, options.ranger_db
        options.services['RANGER']['RANGER_ADMIN'] ?= {}
        options.services['RANGER']['RANGER_ADMIN']['hosts'] = service.deps.ranger_hdpadmin.map (srv) -> srv.node.fqdn
        options.ranger_ssl = service.deps.ranger_hdpadmin[0].options.ssl
        if options.stack_version[0] > 2
          for prop in [
            'rangerusersync_user_password'
            'rangertagsync_user_password'
            'keyadmin_user_password'
          ]
          
            throw Error "Missing password ranger-env.#{prop} HDP 3" unless options.configurations['ranger-env'][prop]

### Ranger Plugins

        if service.deps.ranger_hdfs?.length > 0
          options.configurations['ranger-env']['ranger-hdfs-plugin-enabled'] ?= 'Yes' # done also on the plugin configuration
          options.configurations['ranger-hdfs-security'] ?= merge {}, service.deps.ranger_hdfs[0].options.configurations['ranger-hdfs-security'], options.configurations['ranger-hdfs-security']
          options.configurations['ranger-hdfs-plugin-properties'] ?= merge {}, service.deps.ranger_hdfs[0].options.configurations['ranger-hdfs-plugin-properties'], options.configurations['ranger-hdfs-plugin-properties']
          options.configurations['ranger-hdfs-policymgr-ssl'] ?= merge {}, service.deps.ranger_hdfs[0].options.configurations['ranger-hdfs-policymgr-ssl'], options.configurations['ranger-hdfs-policymgr-ssl']
          options.configurations['ranger-hdfs-audit'] ?= merge {}, service.deps.ranger_hdfs[0].options.configurations['ranger-hdfs-audit'], options.configurations['ranger-hdfs-audit']
        if service.deps.ranger_yarn?.length > 0
          options.configurations['ranger-env']['ranger-yarn-plugin-enabled'] ?= 'Yes' # done also on the plugin configuration
          options.configurations['ranger-yarn-security'] ?= merge {}, service.deps.ranger_yarn[0].options.configurations['ranger-yarn-security'], options.configurations['ranger-yarn-security']
          options.configurations['ranger-yarn-plugin-properties'] ?= merge {}, service.deps.ranger_yarn[0].options.configurations['ranger-yarn-plugin-properties'], options.configurations['ranger-yarn-plugin-properties']
          options.configurations['ranger-yarn-policymgr-ssl'] ?= merge {}, service.deps.ranger_yarn[0].options.configurations['ranger-yarn-policymgr-ssl'], options.configurations['ranger-yarn-policymgr-ssl']
          options.configurations['ranger-yarn-audit'] ?= merge {}, service.deps.ranger_yarn[0].options.configurations['ranger-yarn-audit'], options.configurations['ranger-yarn-audit']
        if service.deps.ranger_hive?.length > 0
          options.configurations['ranger-env']['ranger-hive-plugin-enabled'] ?= 'Yes' # done also on the plugin configuration
          options.configurations['ranger-hive-security'] ?= merge {}, service.deps.ranger_hive[0].options.configurations['ranger-hive-security'], options.configurations['ranger-hive-security']
          options.configurations['ranger-hive-plugin-properties'] ?= merge {}, service.deps.ranger_hive[0].options.configurations['ranger-hive-plugin-properties'], options.configurations['ranger-hive-plugin-properties']
          options.configurations['ranger-hive-policymgr-ssl'] ?= merge {}, service.deps.ranger_hive[0].options.configurations['ranger-hive-policymgr-ssl'], options.configurations['ranger-hive-policymgr-ssl']
          options.configurations['ranger-hive-audit'] ?= merge {}, service.deps.ranger_hive[0].options.configurations['ranger-hive-audit'], options.configurations['ranger-hive-audit']
        if service.deps.ranger_hbase?.length > 0
          options.configurations['ranger-env']['ranger-hbase-plugin-enabled'] ?= 'Yes' # done also on the plugin configuration
          options.configurations['ranger-hbase-security'] ?= merge {}, service.deps.ranger_hbase[0].options.configurations['ranger-hbase-security'], options.configurations['ranger-hbase-security']
          options.configurations['ranger-hbase-plugin-properties'] ?= merge {}, service.deps.ranger_hbase[0].options.configurations['ranger-hbase-plugin-properties'], options.configurations['ranger-hbase-plugin-properties']
          options.configurations['ranger-hbase-policymgr-ssl'] ?= merge {}, service.deps.ranger_hbase[0].options.configurations['ranger-hbase-policymgr-ssl'], options.configurations['ranger-hbase-policymgr-ssl']
          options.configurations['ranger-hbase-audit'] ?= merge {}, service.deps.ranger_hbase[0].options.configurations['ranger-hbase-audit'], options.configurations['ranger-hbase-audit']
        if service.deps.ranger_kafka?.length > 0
          options.configurations['ranger-env']['ranger-kafka-plugin-enabled'] ?= 'Yes' # done also on the plugin configuration
          options.configurations['ranger-kafka-security'] ?= merge {}, service.deps.ranger_kafka[0].options.configurations['ranger-kafka-security'], options.configurations['ranger-kafka-security']
          options.configurations['ranger-kafka-plugin-properties'] ?= merge {}, service.deps.ranger_kafka[0].options.configurations['ranger-kafka-plugin-properties'], options.configurations['ranger-kafka-plugin-properties']
          options.configurations['ranger-kafka-policymgr-ssl'] ?= merge {}, service.deps.ranger_kafka[0].options.configurations['ranger-kafka-policymgr-ssl'], options.configurations['ranger-kafka-policymgr-ssl']
          options.configurations['ranger-kafka-audit'] ?= merge {}, service.deps.ranger_kafka[0].options.configurations['ranger-kafka-audit'], options.configurations['ranger-kafka-audit']
        if service.deps.ranger_knox?.length > 0
          options.configurations['ranger-env']['ranger-knox-plugin-enabled'] ?= 'Yes' # done also on the plugin configuration
          options.configurations['ranger-knox-security'] ?= merge {}, service.deps.ranger_knox[0].options.configurations['ranger-knox-security'], options.configurations['ranger-knox-security']
          options.configurations['ranger-knox-plugin-properties'] ?= merge {}, service.deps.ranger_knox[0].options.configurations['ranger-knox-plugin-properties'], options.configurations['ranger-knox-plugin-properties']
          options.configurations['ranger-knox-policymgr-ssl'] ?= merge {}, service.deps.ranger_knox[0].options.configurations['ranger-knox-policymgr-ssl'], options.configurations['ranger-knox-policymgr-ssl']
          options.configurations['ranger-knox-audit'] ?= merge {}, service.deps.ranger_knox[0].options.configurations['ranger-knox-audit'], options.configurations['ranger-knox-audit']
        if service.deps.ranger_atlas?.length > 0
          options.configurations['ranger-env']['ranger-atlas-plugin-enabled'] ?= 'Yes' # done also on the plugin configuration
          options.configurations['ranger-atlas-security'] ?= merge {}, service.deps.ranger_atlas[0].options.configurations['ranger-atlas-security'], options.configurations['ranger-atlas-security']
          options.configurations['ranger-atlas-plugin-properties'] ?= merge {}, service.deps.ranger_atlas[0].options.configurations['ranger-atlas-plugin-properties'], options.configurations['ranger-atlas-plugin-properties']
          options.configurations['ranger-atlas-policymgr-ssl'] ?= merge {}, service.deps.ranger_atlas[0].options.configurations['ranger-atlas-policymgr-ssl'], options.configurations['ranger-atlas-policymgr-ssl']
          options.configurations['ranger-atlas-audit'] ?= merge {}, service.deps.ranger_atlas[0].options.configurations['ranger-atlas-audit'], options.configurations['ranger-atlas-audit']

## HDFS Service

      if service.deps.hdfs?.length > 0
        options.services['HDFS'] ?= {}
        for srv in service.deps.hdfs
          continue unless srv.options?.groups?
          for name in srv.options.groups
            options.config_groups ?= {}
            options.config_groups[name] ?= {}
            options.config_groups[name]['hosts'] ?= []
            options.config_groups[name]['hosts'].push srv.node.fqdn unless options.config_groups[name]['hosts'].indexOf(srv.node.fqdn) > -1
        options.groups.push service.deps.hdfs.map( (srv) -> srv.options.groups)...
        options.configurations['core-site'] ?= {}
        options.configurations['hdfs-site'] ?= {}
        options.configurations['hadoop-env'] ?= {}
        options.configurations['ssl-server'] ?= {}
        options.configurations['ssl-client'] ?= {}
        options.configurations['hadoop-policy'] ?= {}
        options.core_hosts ?= service.deps.hadoop_core.map (srv) -> srv.node.fqdn
        for srv in service.deps.hadoop_core
          options.racks[srv.node.fqdn] ?= srv.options.rack_info
        if service.deps.hadoop_core?.length > 0
          options.hadoop_group = merge {}, service.deps.hdfs[0].options.hadoop_group, options.hadoop_group
          options.hdfs_user = merge {}, service.deps.hdfs[0].options.hdfs.user
          options.hdfs_group = merge {}, service.deps.hdfs[0].options.hdfs.group
          options.hdfs = merge {}, service.deps.hdfs[0].options.hdfs
          options.racks ?= {}
          exports.enrich_config service.deps.hadoop_core[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hadoop_core[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hadoop_core[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']
          exports.enrich_config service.deps.hadoop_core[0].options.ssl_server, options.configurations['ssl-server']
          exports.enrich_config service.deps.hadoop_core[0].options.ssl_client, options.configurations['ssl-client']
          exports.enrich_config service.deps.hadoop_core[0].options.hadoop_policy, options.configurations['hadoop-policy']
          options.topology = service.deps.hadoop_core[0].options.topology
          esc_realm = quote options.krb5.realm
          options.configurations['core-site']['hadoop.security.auth_to_local'] ?= """
          
                RULE:[2:$1@$0]([rn]m@#{esc_realm})s/.*/yarn/
                RULE:[2:$1@$0](jhs@#{esc_realm})s/.*/mapred/
                RULE:[2:$1@$0]([nd]n@#{esc_realm})s/.*/hdfs/
                RULE:[1:$1@$0](hdfs-#{options.cluster_name}@#{esc_realm})s/.*/hdfs/
                RULE:[1:$1@$0](hbase-#{options.cluster_name}@#{esc_realm})s/.*/hbase/
                RULE:[1:$1@$0](ambari-#{options.cluster_name}@#{esc_realm})s/.*/ambari/
                RULE:[1:$1@$0](ambari-qa-#{options.cluster_name}@#{esc_realm})s/.*/ambari-qa/
                RULE:[1:$1@$0](spark-#{options.cluster_name}@#{esc_realm})s/.*/spark/
                RULE:[2:$1@$0](hm@#{esc_realm})s/.*/hbase/
                RULE:[2:$1@$0](rs@#{esc_realm})s/.*/hbase/
                RULE:[2:$1@$0](opentsdb@#{esc_realm})s/.*/hbase/
                DEFAULT
                RULE:[1:$1](yarn|mapred|hdfs|hive|hbase|oozie)s/.*/nobody/
                RULE:[2:$1](yarn|mapred|hdfs|hive|hbase|oozie)s/.*/nobody/
                RULE:[1:$1]
                RULE:[2:$1]
          
          """
        if service.deps.hdfs_client?.length > 0
          options.services['HDFS']['HDFS_CLIENT'] ?= {} 
          options.services['HDFS']['HDFS_CLIENT']['hosts'] = service.deps.hdfs_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hdfs_client[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hdfs_client[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hdfs_client[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']

        if service.deps.hdfs_nn?.length > 0
          options.services['HDFS']['NAMENODE'] ?= {}
          options.services['HDFS']['NAMENODE']['hosts'] = service.deps.hdfs_nn.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hdfs_nn[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hdfs_nn[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hdfs_nn[0].options.configurations['hdfs-site'], options.configurations['hdfs-site']
          exports.enrich_config service.deps.hdfs_nn[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']
          options.configurations['hdfs-site']['dfs.cluster.administrators'] ?= "#{options.hdfs_user.name},hdfs-#{service.deps.ambari_server_local.options.cluster_name}"

        if service.deps.hdfs_dn?.length > 0
          options.services['HDFS']['DATANODE'] ?= {}
          options.services['HDFS']['DATANODE']['hosts'] = service.deps.hdfs_dn.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hdfs_dn[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hdfs_dn[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hdfs_dn[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']

        if service.deps.hdfs_jn?.length > 0
          options.services['HDFS']['JOURNALNODE'] ?= {}
          options.services['HDFS']['JOURNALNODE']['hosts'] = service.deps.hdfs_jn.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hdfs_jn[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hdfs_jn[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hdfs_jn[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']

        if service.deps.hdfs_zkfc?.length > 0
          options.services['HDFS']['ZKFC'] ?= {}
          options.services['HDFS']['ZKFC']['hosts'] = service.deps.hdfs_zkfc.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hdfs_zkfc[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hdfs_zkfc[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hdfs_zkfc[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']

## YARN Service

      if service.deps.yarn?.length > 0
        for srv in service.deps.yarn
          continue unless srv.options?.groups?
          for name in srv.options.groups
            options.config_groups ?= {}
            options.config_groups[name] ?= {}
            options.config_groups[name]['hosts'] ?= []
            options.config_groups[name]['hosts'].push srv.node.fqdn unless options.config_groups[name]['hosts'].indexOf(srv.node.fqdn) > -1
        options.services['YARN'] ?= {}
        options.configurations['yarn-site'] ?= {}
        options.configurations['capacity-scheduler'] ?= {}
        options.configurations['yarn-env'] ?= {}
        options.yarn_user = merge {}, service.deps.yarn[0].options.yarn.user
        options.yarn_group = merge {}, service.deps.yarn[0].options.yarn.group
        exports.enrich_config service.deps.yarn[0].options.configurations['yarn-site'], options.configurations['yarn-site']
        exports.enrich_config service.deps.yarn[0].options.configurations['yarn-env'], options.configurations['yarn-env']
        if service.deps.yarn_client?.length > 0
          options.services['YARN']['YARN_CLIENT'] ?= {} 
          options.services['YARN']['YARN_CLIENT']['hosts'] = service.deps.yarn_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.yarn_client[0].options.yarn_site, options.configurations['yarn-site']
          exports.enrich_config service.deps.yarn_client[0].options.configurations['yarn-env'], options.configurations['yarn-env']
        if service.deps.yarn_rm?.length > 0
          options.services['YARN']['RESOURCEMANAGER'] ?= {} 
          options.services['YARN']['RESOURCEMANAGER']['hosts'] = service.deps.yarn_rm.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.yarn_rm[0].options.yarn_site, options.configurations['yarn-site']
          exports.enrich_config service.deps.yarn_rm[0].options.configurations['yarn-env'], options.configurations['yarn-env']
          exports.enrich_config service.deps.yarn_rm[0].options.capacity_scheduler, options.configurations['capacity-scheduler']
        if service.deps.yarn_nm?.length > 0
          options.services['YARN']['NODEMANAGER'] ?= {} 
          options.services['YARN']['NODEMANAGER']['hosts'] = service.deps.yarn_nm.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.yarn_nm[0].options.yarn_site, options.configurations['yarn-site']
          exports.enrich_config service.deps.yarn_nm[0].options.configurations['yarn-env'], options.configurations['yarn-env']
        if service.deps.yarn_ts?.length > 0
          options.services['YARN']['APP_TIMELINE_SERVER'] ?= {} 
          options.services['YARN']['APP_TIMELINE_SERVER']['hosts'] = service.deps.yarn_ts.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.yarn_ts[0].options.yarn_site, options.configurations['yarn-site']
          exports.enrich_config service.deps.yarn_ts[0].options.configurations['yarn-env'], options.configurations['yarn-env']

## MAPREDUCE2 Service

      if service.deps.mapreduce?.length > 0
        options.configurations['mapred-env'] ?= {}
        options.configurations['mapred-site'] ?= {}
        options.services['MAPREDUCE2'] ?= {}
        options.mapred_user = merge {}, service.deps.mapreduce[0].options.mapred.user
        options.mapred_group = merge {}, service.deps.mapreduce[0].options.mapred.group
        if service.deps.mapred_client?.length > 0
          options.services['MAPREDUCE2']['MAPREDUCE2_CLIENT'] ?= {} 
          options.services['MAPREDUCE2']['MAPREDUCE2_CLIENT']['hosts'] = service.deps.mapred_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.mapred_client[0].options.mapred_site, options.configurations['mapred-site']
          exports.enrich_config service.deps.mapred_client[0].options.configurations['mapred-env'], options.configurations['mapred-env']
        if service.deps.mapred_jhs?.length > 0
          options.services['MAPREDUCE2']['HISTORYSERVER'] ?= {} 
          options.services['MAPREDUCE2']['HISTORYSERVER']['hosts'] = service.deps.mapred_jhs.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.mapred_jhs[0].options.mapred_site, options.configurations['mapred-site']
          exports.enrich_config service.deps.mapred_jhs[0].options.configurations['mapred-env'], options.configurations['mapred-env']

## HBASE Service

      if service.deps.hbase?.length > 0
        options.services['HBASE'] ?= {}
        for srv in service.deps.hbase
          continue unless srv.options?.groups?
          for name in srv.options.groups
            options.config_groups ?= {}
            options.config_groups[name] ?= {}
            options.config_groups[name]['hosts'] ?= []
            options.config_groups[name]['hosts'].push srv.node.fqdn unless options.config_groups[name]['hosts'].indexOf(srv.node.fqdn) > -1
        options.configurations['hbase-env'] ?= {}
        options.configurations['hbase-policy'] ?= {}
        options.configurations['hbase-site'] ?= {}
        options.hbase_user = merge {}, service.deps.hbase[0].options.user
        options.hbase_group = merge {}, service.deps.hbase[0].options.group
        options.hbase_admin = service.deps.hbase[0].options.admin
        exports.enrich_config service.deps.hbase_client[0].options.configurations['hbase-site'], options.configurations['hbase-site']
        exports.enrich_config service.deps.hbase_client[0].options.configurations['hbase-env'], options.configurations['hbase-env']
        if service.deps.hbase_client?.length > 0
          options.services['HBASE']['HBASE_CLIENT'] ?= {} 
          options.services['HBASE']['HBASE_CLIENT']['hosts'] = service.deps.hbase_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hbase_client[0].options.hbase_site, options.configurations['hbase-site']
          exports.enrich_config service.deps.hbase_client[0].options.configurations['hbase-env'], options.configurations['hbase-env']
        if service.deps.hbase_master?.length > 0
          options.services['HBASE']['HBASE_MASTER'] ?= {} 
          options.services['HBASE']['HBASE_MASTER']['hosts'] = service.deps.hbase_master.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hbase_master[0].options.hbase_site, options.configurations['hbase-site']
          exports.enrich_config service.deps.hbase_master[0].options.configurations['hbase-env'], options.configurations['hbase-env']
        if service.deps.hbase_regionserver?.length > 0
          options.services['HBASE']['HBASE_REGIONSERVER'] ?= {} 
          options.services['HBASE']['HBASE_REGIONSERVER']['hosts'] = service.deps.hbase_regionserver.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hbase_regionserver[0].options.hbase_site, options.configurations['hbase-site']
          exports.enrich_config service.deps.hbase_regionserver[0].options.configurations['hbase-env'], options.configurations['hbase-env']
        if service.deps.hbase_rest?.length > 0
          krb5_username = /^(.+?)[@\/]/.exec(service.deps.hbase_rest[0].options.hbase_site['hbase.rest.kerberos.principal'])?[1]
          throw Error 'Invalid HBase Rest principal' unless krb5_username
          options.configurations['hbase-site']["hadoop.proxyuser.#{krb5_username}.groups"] ?= '*'
          options.configurations['hbase-site']["hadoop.proxyuser.#{krb5_username}.hosts"] ?= '*'
          options.configurations['core-site']["hadoop.proxyuser.#{krb5_username}.groups"] ?= '*'
          options.configurations['core-site']["hadoop.proxyuser.#{krb5_username}.hosts"] ?= '*'

## HIVE Service

      if service.deps.hive?.length > 0
        options.services['HIVE'] ?= {}
        for srv in service.deps.hive
          continue unless srv.options?.groups?
          for name in srv.options.groups
            options.config_groups ?= {}
            options.config_groups[name] ?= {}
            options.config_groups[name]['hosts'] ?= []
            options.config_groups[name]['hosts'].push srv.node.fqdn unless options.config_groups[name]['hosts'].indexOf(srv.node.fqdn) > -1
        options.configurations['hive-site'] ?= {}
        options.configurations['hive-env'] ?= {}
        options.configurations['hive-interactive-site'] ?= {}
        options.configurations['hive-interactive-env'] ?= {}
        options.configurations['webhcat-site'] ?= {}
        options.configurations['webhcat-env'] ?= {}
        options.hive_user ?= merge {}, service.deps.hive[0].options.user
        options.hive_group ?= merge {}, service.deps.hive[0].options.group
        exports.enrich_config service.deps.hive[0].options.configurations['hive-site'], options.configurations['hive-site']
        exports.enrich_config service.deps.hive[0].options.configurations['hive-env'], options.configurations['hive-env']
        if service.deps.hive_client?.length > 0
          options.services['HIVE']['HIVE_CLIENT'] ?= {} 
          options.services['HIVE']['HIVE_CLIENT']['hosts'] = service.deps.hive_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hive_client[0].options.hive_site, options.configurations['hive-site']
          exports.enrich_config service.deps.hive_client[0].options.configurations['hive-env'], options.configurations['hive-env']
        if service.deps.hive_beeline?.length > 0
          options.services['HIVE']['HIVE_CLIENT'] ?= {} 
          options.services['HIVE']['HIVE_CLIENT']['hosts'] = service.deps.hive_beeline.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hive_beeline[0].options.hive_site, options.configurations['hive-site']
          exports.enrich_config service.deps.hive_beeline[0].options.configurations['hive-env'], options.configurations['hive-env']
        if service.deps.hive_server2?.length > 0
          options.services['HIVE']['HIVE_SERVER'] ?= {} 
          options.services['HIVE']['HIVE_SERVER']['hosts'] = service.deps.hive_server2.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hive_server2[0].options.hive_site, options.configurations['hive-site']
          exports.enrich_config service.deps.hive_server2[0].options.configurations['hive-env'], options.configurations['hive-env']
          exports.enrich_config service.deps.hive_server2[0].options.configurations['hiveserver2-site'], options.configurations['hiveserver2-site']
        if service.deps.hcatalog?.length > 0
          options.services['HIVE']['HIVE_METASTORE'] ?= {} 
          options.services['HIVE']['HIVE_METASTORE']['hosts'] = service.deps.hcatalog.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hcatalog[0].options.hive_site, options.configurations['hive-site']
          exports.enrich_config service.deps.hcatalog[0].options.configurations['hive-env'], options.configurations['hive-env']
          options.hive_db ?= service.deps.hive_metastore[0].options.db
        if service.deps.webhcat?.length > 0
          options.hive_webhcat = service.deps.webhcat.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
          options.services['HIVE']['WEBHCAT_SERVER'] ?= {}
          options.services['HIVE']['WEBHCAT_SERVER']['hosts'] ?= service.deps.webhcat.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.webhcat[0].options.hive_site, options.configurations['hive-site']
          exports.enrich_config service.deps.webhcat[0].options.configurations['hive-env'], options.configurations['hive-env']
          exports.enrich_config service.deps.webhcat[0].options.webhcat_site, options.configurations['webhcat-site']
        options.configurations['core-site']["hadoop.proxyuser.#{options.hive_user.name}.groups"] ?= '*'
        options.configurations['core-site']["hadoop.proxyuser.#{options.hive_user.name}.hosts"] ?= [service.deps.hive_server2.map((srv) -> srv.node.fqdn )...,service.deps.hcatalog.map((srv) -> srv.node.fqdn )]
        options.configurations['webhcat-site']['webhcat.proxyuser.hue.groups'] ?= '*'
        options.configurations['webhcat-site']['webhcat.proxyuser.hue.hosts'] ?= '*'
        options.configurations['webhcat-site']['webhcat.proxyuser.knox.groups'] ?= '*'
        options.configurations['webhcat-site']['webhcat.proxyuser.knox.hosts'] ?= '*'

## Oozie Service

      if service.deps.oozie_service?.length > 0
        options.services['OOZIE'] ?= {}
        for srv in service.deps.oozie_service
          continue unless srv.options?.groups?
          for name in srv.options.groups
            options.config_groups ?= {}
            options.config_groups[name] ?= {}
            options.config_groups[name]['hosts'] ?= []
            options.config_groups[name]['hosts'].push srv.node.fqdn unless options.config_groups[name]['hosts'].indexOf(srv.node.fqdn) > -1
        options.configurations['oozie-site'] ?= {}
        options.configurations['oozie-env'] ?= {}
        options.oozie_user ?= merge {}, service.deps.oozie_service[0].options.user
        options.oozie_group ?= merge {}, service.deps.oozie_service[0].options.group
        exports.enrich_config service.deps.oozie_service[0].options.configurations['oozie-site'], options.configurations['oozie-site']
        exports.enrich_config service.deps.oozie_service[0].options.configurations['oozie-env'], options.configurations['oozie-env']
        if service.deps.oozie_server.length > 0
          options.oozie_db ?= service.deps.oozie_server[0].options.db
          options.services['OOZIE']['OOZIE_SERVER'] ?= {} 
          options.services['OOZIE']['OOZIE_SERVER']['hosts'] = service.deps.oozie_server.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.oozie_server[0].options.oozie_site, options.configurations['oozie-site']
          exports.enrich_config service.deps.oozie_server[0].options.configurations['oozie-env'], options.configurations['oozie-env']
        if service.deps.oozie_client.length > 0
          options.services['OOZIE']['OOZIE_CLIENT'] ?= {} 
          options.services['OOZIE']['OOZIE_CLIENT']['hosts'] = service.deps.oozie_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.oozie_client[0].options.oozie_site, options.configurations['oozie-site']
          exports.enrich_config service.deps.oozie_client[0].options.configurations['oozie-env'], options.configurations['oozie-env']
        options.configurations['core-site']["hadoop.proxyuser.#{options.oozie_user.name}.groups"] ?= '*'
        options.configurations['core-site']["hadoop.proxyuser.#{options.oozie_user.name}.hosts"] ?= service.deps.oozie_server.map (srv) -> srv.node.fqdn 

## Ambari Metrics

      if service.deps.ambari_metrics_service?.length > 0
        options.services['AMBARI_METRICS'] ?= {}
        options.configurations['ams-env'] ?= {}
        options.configurations['ams-grafana-ini'] ?= {}
        options.configurations['ams-grafana-env'] ?= {}
        options.configurations['ams-hbase-security-site'] ?= {}
        exports.enrich_config service.deps.ambari_metrics_service[0].options.configurations['ams-env'], options.configurations['ams-env']
        exports.enrich_config service.deps.ambari_metrics_service[0].options.configurations['ams-grafana-ini'], options.configurations['ams-grafana-ini']
        exports.enrich_config service.deps.ambari_metrics_service[0].options.configurations['ams-grafana-env'], options.configurations['ams-grafana-env']
        exports.enrich_config service.deps.ambari_metrics_service[0].options.configurations['ams-hbase-security-site'], options.configurations['ams-hbase-security-site']
        if service.deps.ambari_metrics_collector.length > 0
          options.services['AMBARI_METRICS']['METRICS_COLLECTOR'] ?= {} 
          options.services['AMBARI_METRICS']['METRICS_COLLECTOR']['hosts'] = service.deps.ambari_metrics_collector.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.ambari_metrics_collector[0].options.configurations['ams-env'], options.configurations['ams-env']
          exports.enrich_config service.deps.ambari_metrics_collector[0].options.configurations['ams-grafana-ini'], options.configurations['ams-grafana-ini']
          exports.enrich_config service.deps.ambari_metrics_collector[0].options.configurations['ams-grafana-env'], options.configurations['ams-grafana-env']
          exports.enrich_config service.deps.ambari_metrics_collector[0].options.configurations['ams-hbase-security-site'], options.configurations['ams-hbase-security-site']
        if service.deps.ambari_metrics_monitor.length > 0
          options.services['AMBARI_METRICS']['METRICS_MONITOR'] ?= {} 
          options.services['AMBARI_METRICS']['METRICS_MONITOR']['hosts'] = service.deps.ambari_metrics_monitor.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.ambari_metrics_monitor[0].options.configurations['ams-env'], options.configurations['ams-env']
          exports.enrich_config service.deps.ambari_metrics_monitor[0].options.configurations['ams-grafana-ini'], options.configurations['ams-grafana-ini']
          exports.enrich_config service.deps.ambari_metrics_monitor[0].options.configurations['ams-grafana-env'], options.configurations['ams-grafana-env']
          exports.enrich_config service.deps.ambari_metrics_monitor[0].options.configurations['ams-hbase-security-site'], options.configurations['ams-hbase-security-site']
        if service.deps.ambari_metrics_grafana.length > 0
          options.services['AMBARI_METRICS']['METRICS_GRAFANA'] ?= {} 
          options.services['AMBARI_METRICS']['METRICS_GRAFANA']['hosts'] = service.deps.ambari_metrics_grafana.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.ambari_metrics_grafana[0].options.configurations['ams-env'], options.configurations['ams-env']
          exports.enrich_config service.deps.ambari_metrics_grafana[0].options.configurations['ams-grafana-ini'], options.configurations['ams-grafana-ini']
          exports.enrich_config service.deps.ambari_metrics_grafana[0].options.configurations['ams-grafana-env'], options.configurations['ams-grafana-env']
          exports.enrich_config service.deps.ambari_metrics_grafana[0].options.configurations['ams-hbase-security-site'], options.configurations['ams-hbase-security-site']
        # options.configurations['core-site']["hadoop.proxyuser.#{options.ams_user.name}.groups"] ?= '*'
        # options.configurations['core-site']["hadoop.proxyuser.#{options.ams_user.name}.hosts"] ?= service.deps.ambari_metrics_monitor.map (srv) -> srv.node.fqdn 

## LOGSEARCH Service

      if service.deps.logsearch_service?.length > 0
        options.services['LOGSEARCH'] ?= {}
        options.configurations['logfeeder-env'] ?= {}
        options.configurations['logsearch-env'] ?= {}
        options.configurations['logsearch-common-env'] ?= {}
        options.configurations['logsearch-admin-json'] ?= {}
        exports.enrich_config service.deps.logsearch_service[0].options.configurations['logfeeder-env'], options.configurations['logfeeder-env']
        exports.enrich_config service.deps.logsearch_service[0].options.configurations['logsearch-common-env'], options.configurations['logsearch-common-env']
        exports.enrich_config service.deps.logsearch_service[0].options.configurations['logsearch-admin-json'], options.configurations['logsearch-admin-json']
        exports.enrich_config service.deps.logsearch_service[0].options.configurations['logsearch-env'], options.configurations['logsearch-env']
        if service.deps.logsearch_server.length > 0
          options.logsearch_server = true
          options.services['LOGSEARCH']['LOGSEARCH_SERVER'] ?= {} 
          options.services['LOGSEARCH']['LOGSEARCH_SERVER']['hosts'] = service.deps.logsearch_server.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.logsearch_server[0].options.configurations['logfeeder-env'], options.configurations['logfeeder-env']
          exports.enrich_config service.deps.logsearch_server[0].options.configurations['logsearch-common-env'], options.configurations['logsearch-common-env']
          exports.enrich_config service.deps.logsearch_server[0].options.configurations['logsearch-env'], options.configurations['logsearch-env']
        if service.deps.logsearch_feeder.length > 0
          options.services['LOGSEARCH']['LOGSEARCH_LOGFEEDER'] ?= {} 
          options.services['LOGSEARCH']['LOGSEARCH_LOGFEEDER']['hosts'] = service.deps.logsearch_feeder.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.logsearch_feeder[0].options.configurations['logfeeder-env'], options.configurations['logfeeder-env']
          exports.enrich_config service.deps.logsearch_feeder[0].options.configurations['logsearch-common-env'], options.configurations['logsearch-common-env']
          exports.enrich_config service.deps.logsearch_feeder[0].options.configurations['logsearch-env'], options.configurations['logsearch-env']

## SMARTSENSE Service

      if service.deps.smartsense_service?.length > 0
        options.services['SMARTSENSE'] ?= {}
        options.configurations['activity-zeppelin-site'] ?= {}
        if service.deps.smartsense_explorer?.length > 0
          options.services['SMARTSENSE']['ACTIVITY_EXPLORER'] ?= {} 
          options.services['SMARTSENSE']['ACTIVITY_EXPLORER']['hosts'] = service.deps.smartsense_explorer.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.smartsense_explorer[0].options.configurations['activity-zeppelin-site'], options.configurations['activity-zeppelin-site']
        if service.deps.smartsense_server?.length > 0
          options.services['SMARTSENSE']['HST_SERVER'] ?= {} 
          options.services['SMARTSENSE']['HST_SERVER']['hosts'] = service.deps.smartsense_server.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.smartsense_server[0].options.configurations['activity-zeppelin-site'], options.configurations['activity-zeppelin-site']
        if service.deps.smartsense_agent?.length > 0
          options.services['SMARTSENSE']['HST_AGENT'] ?= {} 
          options.services['SMARTSENSE']['HST_AGENT']['hosts'] = service.deps.smartsense_agent.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.smartsense_agent[0].options.configurations['activity-zeppelin-site'], options.configurations['activity-zeppelin-site']

## SPARK 1 Service

      if service.deps.spark_service?.length > 0
        options.services['SPARK'] ?= {}
        options.configurations['spark-defaults'] ?= {}
        options.configurations['spark-env'] ?= {}
        if service.deps.spark_hs?.length > 0
          options.services['SPARK']['SPARK_JOBHISTORYSERVER'] ?= {} 
          options.services['SPARK']['SPARK_JOBHISTORYSERVER']['hosts'] = service.deps.spark_hs.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.spark_hs[0].options.conf, options.configurations['spark-defaults']
        if service.deps.spark_client.length > 0
          options.services['SPARK']['SPARK_CLIENT'] ?= {} 
          options.services['SPARK']['SPARK_CLIENT']['hosts'] = service.deps.spark_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.spark_client[0].options.conf, options.configurations['spark-defaults']
        if options.configurations['spark-defaults']['spark.history.kerberos.enabled'] is 'true'
          options.configurations['spark-defaults']['spark.history.kerberos.principal'] ?="#{service.deps.spark_service[0].options.user.name}-#{options.cluster_name}@#{options.krb5.realm}"
          options.configurations['spark-defaults']['spark.history.kerberos.keytab'] ?= '/etc/security/keytabs/spark.headless.keytab'

## SPARK 2 Service

      if service.deps.spark2_service?.length > 0
        options.services['SPARK2'] ?= {}
        options.configurations['spark2-defaults'] ?= {}
        options.configurations['spark2-env'] ?= {}
        options.configurations['livy2-conf'] ?= {}
        if service.deps.spark2_hs?.length > 0
          options.services['SPARK2']['SPARK2_JOBHISTORYSERVER'] ?= {} 
          options.services['SPARK2']['SPARK2_JOBHISTORYSERVER']['hosts'] = service.deps.spark2_hs.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.spark2_hs[0].options.conf, options.configurations['spark2-defaults']
        if service.deps.spark2_client.length > 0
          options.services['SPARK2']['SPARK2_CLIENT'] ?= {} 
          options.services['SPARK2']['SPARK2_CLIENT']['hosts'] = service.deps.spark2_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.spark2_client[0].options.conf, options.configurations['spark2-defaults']
        if options.configurations['spark2-defaults']['spark.history.kerberos.enabled'] is 'true'
          options.configurations['spark2-defaults']['spark.history.kerberos.principal'] ?="#{service.deps.spark2_service[0].options.user.name}-#{options.cluster_name}@#{options.krb5.realm}"
          options.configurations['spark2-defaults']['spark.history.kerberos.keytab'] ?= '/etc/security/keytabs/spark.headless.keytab'
        if service.deps.spark2_livy.length > 0
          options.services['SPARK2']['LIVY2_SERVER'] ?= {} 
          options.services['SPARK2']['LIVY2_SERVER']['hosts'] = service.deps.spark2_livy.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.spark2_livy[0].options.configurations['livy2-conf'], options.configurations['livy2-conf']


## SQOOP Service

      if service.deps.sqoop?.length > 0
        options.configurations['sqoop-site'] ?= {}
        options.services['SQOOP'] ?= {}
        options.services['SQOOP']['SQOOP'] ?= {}
        options.services['SQOOP']['SQOOP']['hosts'] = service.deps.sqoop.map (srv) -> srv.node.fqdn

## PIG Service

      if service.deps.pig?.length > 0
        options.services['PIG'] ?= {}
        options.services['PIG']['PIG'] ?= {}
        options.services['PIG']['PIG']['hosts'] = service.deps.pig.map (srv) -> srv.node.fqdn


## TEZ Service

      if service.deps.tez?.length > 0
        options.tez = 
        options.services['TEZ'] ?= {}
        options.services['TEZ']['TEZ_CLIENT'] ?= {}
        options.services['TEZ']['TEZ_CLIENT']['hosts'] = service.deps.tez.map (srv) -> srv.node.fqdn

## KNOX Service

      if service.deps.knox_service?.length > 0
        options.services['KNOX'] ?= {}
        options.configurations['gateway-site'] ?= {}
        options.knox_user = service.deps.knox_service[0].options.user
        options.knox_group = service.deps.knox_service[0].options.group
        exports.enrich_config service.deps.knox_service[0].options.configurations['gateway-site'], options.configurations['gateway-site']
        if service.deps.knox_server?.length > 0
          options.knox_opts = service.deps.knox_server?[0].options
          options.knox_server = service.deps.knox_server.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
          options.services['KNOX']['KNOX_GATEWAY'] ?= {} 
          options.services['KNOX']['KNOX_GATEWAY']['hosts'] = service.deps.knox_server.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.knox_server[0].options.configurations['gateway-site'], options.configurations['gateway-site']
          if service.deps.oozie_server?.length > 0
            options.configurations['oozie-site']['oozie.service.ProxyUserService.proxyuser.knox.hosts'] ?= service.deps.knox_server.map( (srv) -> srv.node.fqdn ).join(',')
            options.configurations['oozie-site']['oozie.service.ProxyUserService.proxyuser.knox.groups'] ?= '*'
        if service.deps.knox_service.length > 0
          exports.enrich_config service.deps.knox_service[0].options.configurations['knox-env'], options.configurations['knox-env']

## KAFKA Service

      if service.deps.kafka_service?.length > 0
        options.services['KAFKA'] ?= {}
        for srv in service.deps.kafka_service
          continue unless srv.options?.groups?
          for name in srv.options.groups
            options.config_groups ?= {}
            options.config_groups[name] ?= {}
            options.config_groups[name]['hosts'] ?= []
            options.config_groups[name]['hosts'].push srv.node.fqdn unless options.config_groups[name]['hosts'].indexOf(srv.node.fqdn) > -1
        options.configurations['kafka-broker'] ?= {}
        options.configurations['kafka-env'] ?= {}
        options.configurations['kafka-log4j'] ?= {}
        options.kafka_user = service.deps.kafka_service[0].options.user
        options.kafka_group = service.deps.kafka_service[0].options.group
        exports.enrich_config service.deps.kafka_service[0].options.config, options.configurations['kafka-broker']
        exports.enrich_config service.deps.kafka_service[0].options.configurations['kafka-env'], options.configurations['kafka-env']
        if service.deps.kafka_broker.length > 0
          options.services['KAFKA']['KAFKA_BROKER'] ?= {} 
          options.services['KAFKA']['KAFKA_BROKER']['hosts'] = service.deps.kafka_broker.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.kafka_broker[0].options.config, options.configurations['kafka-broker']
          exports.enrich_config service.deps.kafka_broker[0].options.configurations['kafka-env'], options.configurations['kafka-env']
          options.kafka_env ?= service.deps.kafka_broker[0].options.env

## Zeppelin

      if service.deps.zeppelin_service?.length > 0
        options.services['ZEPPELIN'] ?= {}
        options.configurations['zeppelin-config'] ?= {}
        options.configurations['zeppelin-env'] ?= {}
        options.zeppelin_user = service.deps.zeppelin_service[0].options.user
        options.zeppelin_group = service.deps.zeppelin_service[0].options.group
        if service.deps.zeppelin_master.length > 0
          options.services['ZEPPELIN']['ZEPPELIN_MASTER'] ?= {} 
          options.services['ZEPPELIN']['ZEPPELIN_MASTER']['hosts'] = service.deps.zeppelin_master.map (srv) -> srv.node.fqdn
          options.configurations['zeppelin-env']['zeppelin.server.kerberos.principal'] ?= "#{options.zeppelin_user.name}-#{options.cluster_name}@#{options.krb5.realm}"
          options.configurations['zeppelin-env']['zeppelin.server.kerberos.keytab'] ?= '/etc/security/keytabs/zeppelin.server.kerberos.keytab'
          exports.enrich_config service.deps.zeppelin_master[0].options.configurations['zeppelin-config'], options.configurations['zeppelin-config']
          exports.enrich_config service.deps.zeppelin_master[0].options.configurations['zeppelin-env'], options.configurations['zeppelin-env']

## Config Groups
`config_groups` contains final object that install will submit to ambari.
`groups` is the array of config_groups name to which the host belongs to.

      options.config_groups ?= {}
      options.groups ?= []

# ## Ambari Agent
# Register users to ambari agent's user list.
# 
#       for srv in service.deps.ambari_agent
#         srv.options.users ?= {}
#         srv.options.users['hdfs'] ?= options.hdfs.user
#         srv.options.users['yarn'] ?= options.yarn.user
#         srv.options.users['mapred'] ?= options.mapred.user
#         srv.options.groups ?= {}
#         srv.options.groups['hdfs'] ?= options.hdfs.group
#         srv.options.groups['yarn'] ?= options.yarn.group
#         srv.options.groups['mapred'] ?= options.mapred.group
#         srv.options.groups['hadoop_group'] ?= options.hadoop_group

## Ambari Agent Hosts
    
      options.hosts ?= service.deps.ambari_agent.map (srv) -> srv.node.fqdn

## Rack Awareness
      
      options.racks ?= []

## Utilities

    exports.enrich_config = (source, target) ->
      target ?= {}
      for k, v of source
        target[k] ?= v

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
    quote = require 'regexp-quote'
