
# Ranger Knox Plugin Install

    module.exports = header: 'Ranger Knox Plugin', handler: (options) ->
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


## Dependencies

    quote = require 'regexp-quote'
    path = require 'path'
    mkcmd = require 'ryba/lib/mkcmd'

[plugin]: https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_installing_manually_book/content/installing_ranger_plugins.html#installing_ranger_knox_plugin
