
# Ranger HBase Plugin Install

    module.exports = header: 'Ranger HBase Plugin', handler: (options) ->

## Wait

      @call 'ryba-ambari-takeover/ranger/hdpadmin/wait', once: true, options.wait_ranger_admin

## Register

      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'

## Packages

      @service
        name: "ranger-hbase-plugin"

## Audit Layout

Matchs step 1 in [hdfs plugin configuration][plugin]. Instead of using the web ui
we execute this task using the rest api.


      @system.mkdir
        header: 'HDFS Spool Dir'
        if: options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
        target: options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR']
        uid: options.hbase_user.name
        gid: options.hadoop_group.name
        mode: 0o0750
      @system.mkdir
        header: 'Solr Spool Dir'
        if: options.install['XAAUDIT.SOLR.IS_ENABLED'] is 'true'
        target: options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR']
        uid: options.hbase_user.name
        gid: options.hbase_group.name
        mode: 0o0750

## Upload configuration to Ambari

      @ambari.configs.update
        header: 'Upload ranger-hbase-plugin-properties'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hbase-plugin-properties'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hbase-plugin-properties']

      @ambari.configs.update
        header: 'Upload ranger-hbase-security'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hbase-security'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hbase-security']

      @ambari.configs.update
        header: 'Upload ranger-hbase-policymgr-ssl'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hbase-policymgr-ssl'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hbase-policymgr-ssl']

      @ambari.configs.update
        header: 'Upload ranger-hbase-audit'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hbase-audit'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hbase-audit']

Note, by default, we're are using the same Ranger principal for every
plugin and the principal is created by the Ranger Admin service. Chances
are that a customer user will need specific ACLs but this hasn't been
tested.

      @krb5.addprinc options.krb5.admin,
        header: 'Plugin Principal'
        principal: "#{options.service_repo.configs.username}"
        password: options.service_repo.configs.password

## SSL

The Ranger Plugin does not use its truststore configuration when using solrJClient.
Must add certificate to JAVA Cacerts file manually.

TODO: remove CA from JAVA_HOME cacerts in a future version.

      @java.keystore_add
        keystore: "#{options.jre_home}/lib/security/cacerts"
        storepass: 'changeit'
        caname: "hadoop_root_ca"
        cacert: "#{options.ssl.cacert.source}"
        local: "#{options.ssl.cacert.local}"
## SSL

      @call header: 'SSL', ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
          storepass: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"


## Dependencies

    quote = require 'regexp-quote'
    path = require 'path'
    mkcmd = require 'ryba/lib/mkcmd'
    properties = require '../ryba/lib/properties'
    fs = require 'ssh2-fs'

[plugin]:(https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_installing_manually_book/content/installing_ranger_plugins.html#installing_ranger_hbase_plugin)
[perms-fix]: https://community.hortonworks.com/questions/23717/ranger-solr-on-hdp-234-unable-to-refresh-policies.html
