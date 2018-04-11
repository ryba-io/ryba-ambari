
# Ranger HDFS Plugin Install

    module.exports = header: 'Ambari Ranger HDFS Plugin', handler: (options) ->

## Wait

      @call 'ryba/ranger/admin/wait', once: true, options.wait_ranger_admin

## HDFS Dependencies

      # @call 'ryba/hadoop/hdfs_client/install' #migation solved it with implicy hdfs_client requirement
      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdfs_mkdir', 'ryba/lib/hdfs_mkdir'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'

## HDP version

      version = null
      @system.execute
        header: 'HDP Version'
        shy: true
        cmd: """
        hdp-select versions | tail -1
        """
       , (err, executed,stdout, stderr) ->
          return  err if err or not executed
          version = stdout.trim() if executed

## Package

      @service
        header: 'Package'
        name: "ranger-hdfs-plugin"

## Layout

The value present in "XAAUDIT.HDFS.DESTINATION_DIRECTORY" contains variables
such as "%app-type% and %time:yyyyMMdd%".

migration: wdavidw 170918, NameNodes are not yet started.

      @system.mkdir
        header: 'HDFS Spool Dir'
        if: options.install['XAAUDIT.HDFS.IS_ENABLED'] is 'true'
        target: options.install['XAAUDIT.HDFS.FILE_SPOOL_DIR']
        uid: options.hdfs_user.name
        gid: options.hadoop_group.name
        mode: 0o0750
      @system.mkdir
        header: 'SOLR Spool Dir'
        if: options.install['XAAUDIT.SOLR.IS_ENABLED'] is 'true'
        target: options.install['XAAUDIT.SOLR.FILE_SPOOL_DIR']
        uid: options.hdfs_user.name
        gid: options.hadoop_group.name
        mode: 0o0750

## Upload Configuration to Ambari

      @ambari.configs.update
        header: 'Upload ranger-hdfs-plugin-properties'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hdfs-plugin-properties'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hdfs-plugin-properties']

      @ambari.configs.update
        header: 'Upload ranger-hdfs-security'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hdfs-security'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hdfs-security']

      @ambari.configs.update
        header: 'Upload ranger-hdfs-policymgr-ssl'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hdfs-policymgr-ssl'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hdfs-policymgr-ssl']

      @ambari.configs.update
        header: 'Upload ranger-hdfs-audit'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hdfs-audit'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hdfs-audit']

## Kerberos Principal

Note, by default, we're are using the same Ranger principal for every
plugin and the principal is created by the Ranger Admin service. Chances
are that a customer user will need specific ACLs but this hasn't been
tested.

      # See [#96](https://github.com/ryba-io/ryba/issues/95): Ranger HDFS: should we use a dedicated principal
      @krb5.addprinc options.krb5.admin,
        header: 'Plugin Principal'
        principal: "#{options.service_repo.configs.username}"
        password: options.service_repo.configs.password

## Dependencies

    quote = require 'regexp-quote'

[plugin]: https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_installing_manually_book/content/installing_ranger_plugins.html#installing_ranger_hdfs_plugin
[plugin-source]: https://github.com/apache/incubator-ranger/blob/ranger-0.6/agents-audit/src/main/java/org/apache/ranger/audit/utils/InMemoryJAASConfiguration.java
