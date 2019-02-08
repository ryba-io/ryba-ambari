
# Ambari Logsearch Configuration

    module.exports = (service) ->
      options = service.options

## Identities

      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'zeppelin'
      options.group.system ?= true

      # Hadoop Group is also defined in ryba/hadoop/core
      options.hadoop_group = name: options.hadoop_group if typeof options.hadoop_group is 'string'
      options.hadoop_group ?= {}
      options.hadoop_group.name ?= 'hadoop'
      options.hadoop_group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'zeppelin'
      options.user.system ?= true
      options.user.gid = options.group.name
      options.user.comment ?= 'Ambari Zeppelin User'
      options.user.home ?= '/var/run/zeppelin'
      options.user.groups ?= 'hadoop'
      options.user.limits ?= {}
      options.user.limits.nofile ?= 64000
      options.user.limits.nproc ?= 32000
      
      options.livy_ssl_enabled ?= (service.deps.spark_livy_server?.length > 0) and service.deps.spark_livy_server?[0].options.ssl.enabled
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

## Configurations

      options.configurations ?= {}
      options.configurations['zeppelin-config'] ?= {}

## Ambari Zeppelin Configuration

      options.configurations['zeppelin-env'] ?= {}
      
## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
