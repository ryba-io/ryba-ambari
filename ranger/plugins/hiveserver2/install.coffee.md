
# Ranger Hive Plugin Install

    module.exports = header: 'Ranger Hive Plugin', handler: (options) ->
      version = null
      #https://mail-archives.apache.org/mod_mbox/incubator-ranger-user/201605.mbox/%3C363AE5BD-D796-425B-89C9-D481F6E74BAF@apache.org%3E

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdfs_mkdir', 'ryba/lib/hdfs_mkdir'
      @registry.register 'ranger_user', 'ryba/ranger/actions/ranger_user'
      @registry.register 'ranger_policy', 'ryba/ranger/actions/ranger_policy'
      @registry.register 'ranger_service', 'ryba/ranger/actions/ranger_service'

## Wait

      @call 'ryba/ranger/admin/wait', once: true, options.wait_ranger_admin

## Packages

      @service
        name: "ranger-hive-plugin"

# ## Ranger User
# 
#       @ranger_user
#         header: 'Ranger User'
#         username: options.ranger_admin.username
#         password: options.ranger_admin.password
#         url: options.install['POLICY_MGR_URL']
#         user: options.plugin_user

## Audit Layout

      @system.mkdir
        header: 'HDFS Spool Dir'
        if: options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
        target: options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR']
        uid: options.hive_user.name
        gid: options.hive_group.name
        mode: 0o0750
      @system.mkdir
        header: 'Solr Spool Dir'
        if: options.install['XAAUDIT.SOLR.IS_ENABLED'] is 'true'
        target: options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR']
        uid: options.hive_user.name
        gid: options.hive_group.name
        mode: 0o0750

Note, by default, we're are using the same Ranger principal for every
plugin and the principal is created by the Ranger Admin service. Chances
are that a customer user will need specific ACLs but this hasn't been
tested.

      @krb5.addprinc options.krb5.admin,
        header: 'Plugin Principal'
        principal: "#{options.service_repo.configs.username}"
        password: options.service_repo.configs.password

## SSL

      @call header: 'SSL', retry: 0, ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
          storepass: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

## Dependencies

    quote = require 'regexp-quote'
    path = require 'path'
    mkcmd = require 'ryba/lib/mkcmd'
    properties = require '../ryba/lib/properties'
    fs = require 'ssh2-fs'

[plugin]: https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_installing_manually_book/content/installing_ranger_plugins.html#installing_ranger_hive_plugin
