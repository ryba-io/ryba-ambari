
# Ambari Smartsense Configuration

    module.exports = (service) ->
      options = service.options

## Identities

      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'smartsense'
      options.group.system ?= true

      # Hadoop Group is also defined in ryba/hadoop/core
      options.hadoop_group = name: options.hadoop_group if typeof options.hadoop_group is 'string'
      options.hadoop_group ?= {}
      options.hadoop_group.name ?= 'hadoop'
      options.hadoop_group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'smartsense'
      options.user.system ?= true
      options.user.gid = options.group.name
      options.user.comment ?= 'Ambari Logsearch User'
      options.user.home ?= '/var/run/smartsense'
      options.user.groups ?= 'hadoop'
      options.user.limits ?= {}
      options.user.limits.nofile ?= 64000
      options.user.limits.nproc ?= 32000

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      # options.conf_dir ?= '/etc/ambari-server/conf'
      options.sudo ?= false
      options.admin ?= {}
      options.configurations ?= {}
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## Ambari Logsearch Solr Configuration

      options.configurations['activity-zeppelin-site'] ?= {}

      
## Ambari Logsearch Server Configuration

      options.hst_server_hosts ?= []

## Ambari Logsearch Feeder Configuration

      options.hst_agent_hosts ?= []

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
