
# Ambari Logsearch Configuration

    module.exports = (service) ->
      options = service.options

## Identities

      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'logsearch'
      options.group.system ?= true

      # Hadoop Group is also defined in ryba/hadoop/core
      options.hadoop_group = name: options.hadoop_group if typeof options.hadoop_group is 'string'
      options.hadoop_group ?= {}
      options.hadoop_group.name ?= 'hadoop'
      options.hadoop_group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'logsearch'
      options.user.system ?= true
      options.user.gid = options.group.name
      options.user.comment ?= 'Ambari Logsearch User'
      options.user.home ?= '/var/run/logsearch'
      options.user.groups ?= 'hadoop'
      options.user.limits ?= {}
      options.user.limits.nofile ?= 64000
      options.user.limits.nproc ?= 32000

      # options.group = merge service.deps.ambari_server.options.group, options.group
      # options.user = merge service.deps.ambari_server.options.user, options.user
      # options.test_user = merge service.deps.ambari_server.options.test_user, options.test_user
      # options.test_group = merge service.deps.ambari_server.options.test_group, options.test_group

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      # options.conf_dir ?= '/etc/ambari-server/conf'
      options.sudo ?= false
      options.admin ?= {}
      options.configurations ?= {}

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## Ambari Logsearch Solr Configuration

      options.configurations['logsearch-env'] ?= {}
      options.configurations['logfeeder-env'] ?= {}
      options.configurations['logsearch-common-env'] ?= {}
      options.configurations['logsearch-admin-json'] ?= {}
      options.admin_user ?= 'ambari_logsearch_admin'
      throw Error 'Missing Logsearch Portal admin password' unless options.admin_password?
      options.configurations['logsearch-admin-json']['logsearch_admin_username'] ?= options.admin_user 
      options.configurations['logsearch-admin-json']['logsearch_admin_password'] ?= options.admin_password
      if options.solr_external
          throw Error "Missing Solr options.solr.cluster_config.version property example: 6.3.0" unless options.solr.cluster_config.version?
          throw Error "Unexpected version format. Solr version should look like 6.3.0" unless /[0-9](.[0-9]){2}/.test options.solr.cluster_config.version
          throw Error "Missing Solr options.solr.cluster_config.ssl_enabled property example: true" unless options.solr.cluster_config.ssl_enabled?
          throw Error "Missing Solr options.solr.cluster_config.zk_node: master01.metal.ryba:2181" unless options.solr.cluster_config.zk_node?
          throw Error "Znode Path must start with / character" unless options.solr.cluster_config.zk_node[0] is '/'
          throw Error "Missing Solr options.solr.cluster_config.zk_quorum: master01.metal.ryba:2181" unless options.solr.cluster_config.zk_quorum?
          throw Error "Missing Solr options.solr.cluster_config.authentication: kerberos" unless options.solr.cluster_config.authentication?
          #Ambari log search solr external configuration
          options.configurations['logsearch-common-env']['logsearch_use_external_solr'] ?= 'true'
          options.configurations['logsearch-common-env']['logsearch_external_solr_kerberos_enabled'] ?= if options.solr.cluster_config.authentication is 'kerberos' then 'true' else 'false'
          options.configurations['logsearch-common-env']['logsearch_external_solr_ssl_enabled'] ?= options.solr.cluster_config.ssl_enabled
          options.configurations['logsearch-common-env']['logsearch_external_solr_zk_znode'] ?= options.solr.cluster_config.zk_node
          options.configurations['logsearch-common-env']['logsearch_external_solr_zk_quorum'] ?= options.solr.cluster_config.zk_quorum
      
## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
