
# Ranger Knox Plugin Install

    module.exports = header: 'Ranger Knox Plugin', handler: ({options}) ->
      version = null

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdfs_mkdir', 'ryba/lib/hdfs_mkdir'
      @registry.register 'ranger_service', 'ryba/ranger/actions/ranger_service'

## Wait

      @call 'ryba/ranger/admin/wait', once: true, options.wait_ranger_admin

## Packages

      @service
        name: "ranger-knox-plugin"

Note, by default, we're are using the same Ranger principal for every
plugin and the principal is created by the Ranger Admin service. Chances
are that a customer user will need specific ACLs but this hasn't been
tested.

      @krb5.addprinc options.krb5.admin,
        header: 'Plugin Principal'
        principal: "#{options.service_repo.configs.username}"
        password: options.service_repo.configs.password

## Audit Layout

The value present in "XAAUDIT.HDFS.DESTINATION_DIRECTORY" contains variables
such as "%app-type% and %time:yyyyMMdd%".

      # @hdfs_mkdir
      #   header: 'HDFS Audit'
      #   if: options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
      #   target: "/#{options.user.name}/audit/#{options.service_repo.type}"
      #   mode: 0o0750
      #   parent:
      #     mode: 0o0711
      #     user: options.user.name
      #     group: options.group.name
      #   user: options.knox_user.name
      #   group: options.knox_group.name
      #   krb5_user: options.hdfs_krb5_user
      @system.mkdir
        target: options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR']
        uid: options.knox_user.name
        gid: options.hadoop_group.name
        mode: 0o0750
        if: options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
      @system.mkdir
        header: 'Solr Spool Dir'
        if: options.install['XAAUDIT.SOLR.IS_ENABLED'] is 'true'
        target: options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR']
        uid: options.knox_user.name
        gid: options.hadoop_group.name
        mode: 0o0750


      @call header: 'SSL', ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
          storepass: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

        @call
          if: options.importCerts?
        , (_, cb) ->
          {truststore, configurations} = options
          tmp_location = "/tmp/ryba_cacert_#{Date.now()}"
          @each options.importCerts, ({options}, callback) ->
            {source, local, name} = options.value
            @file.download
              header: 'download cacert'
              source: source
              target: "#{tmp_location}/cacert"
              local: true
            @java.keystore_add
              header: "add cacert to #{name}"
              keystore: configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
              storepass: configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
              caname: name
              cacert: "#{tmp_location}/cacert"
            @next callback
          @system.remove
            target: tmp_location
          @next cb
## Dependencies

    quote = require 'regexp-quote'
    path = require 'path'
    mkcmd = require 'ryba/lib/mkcmd'

[plugin]: https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_installing_manually_book/content/installing_ranger_plugins.html#installing_ranger_knox_plugin
