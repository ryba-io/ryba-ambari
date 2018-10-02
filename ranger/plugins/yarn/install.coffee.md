
    module.exports = header: 'Ambari Ranger YARN Plugin install', handler: ({options}) ->
      version = null

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdfs_mkdir', 'ryba/lib/hdfs_mkdir'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'

## Wait

      @call 'ryba-ambari-takeover/ranger/hdpadmin/wait', once: true, options.wait_ranger_admin


## Packages

      @call header: 'Packages', ->
        @system.execute
          header: 'Setup Execution'
          shy: true
          cmd: """
          hdp-select versions | tail -1
          """
         , (err, {executed,stdout, stderr}) ->
            return  err if err or not executed
            version = stdout.trim() if executed
        @service
          name: "ranger-yarn-plugin"

## Layout


      @system.mkdir
        header: 'HDFS Spool Dir'
        if: options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
        target: options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR']
        uid: options.yarn_user.name
        gid: options.hadoop_group.name
        mode: 0o0750
      @system.mkdir
        target: options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR']
        uid: options.yarn_user.name
        gid: options.hadoop_group.name
        mode: 0o0750
        if: options.install['XAAUDIT.SOLR.IS_ENABLED'] is 'true'

## Upload configuration to Ambari

      @ambari.configs.update
        header: 'Upload ranger-yarn-plugin-properties'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-yarn-plugin-properties'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-yarn-plugin-properties']

      @ambari.configs.update
        header: 'Upload ranger-yarn-security'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-yarn-security'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-yarn-security']

      @ambari.configs.update
        header: 'Upload ranger-yarn-policymgr-ssl'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-yarn-policymgr-ssl'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-yarn-policymgr-ssl']

      @ambari.configs.update
        header: 'Upload ranger-yarn-audit'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-yarn-audit'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-yarn-audit']


Note, by default, we're are using the same Ranger principal for every
plugin and the principal is created by the Ranger Admin service. Chances
are that a customer user will need specific ACLs but this hasn't been
tested.

      # See [#96](https://github.com/ryba-io/ryba/issues/95): Ranger HDFS: should we use a dedicated principal
      @krb5.addprinc
        header: 'Ranger YARN Principal'
        # if: options.plugins.principal
        principal: "#{options.service_repo.configs.username}"
        password: options.service_repo.configs.password
      , options.krb5.admin

## Dependencies

    quote = require 'regexp-quote'
    path = require 'path'
    mkcmd = require 'ryba/lib/mkcmd'
    fs = require 'ssh2-fs'

[yarn-plugin]:(https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_installing_manually_book/content/installing_ranger_plugins.html#installing_ranger_yarn_plugin)
