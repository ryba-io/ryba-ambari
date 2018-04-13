
# Ambari Metrics Collector Configuration

    module.exports = (service) ->
      options = service.options

      options.group = merge service.deps.ambari_server.options.group, options.group
      options.user = merge service.deps.ambari_server.options.user, options.user
      options.test_user = merge service.deps.ambari_server.options.test_user, options.test_user
      options.test_group = merge service.deps.ambari_server.options.test_group, options.test_group

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.conf_dir ?= '/etc/ambari-server/conf'
      options.sudo ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.master_key ?= null
      options.admin ?= {}
      options.krb5 ?= merge {}, service.deps.ambari_server.options.krb5, options.krb5
      options.configurations ?= {}

## Kerberos


      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

### Embedded HBase Security

      options.configurations['ams-hbase-security-site'] ?= {}
      #zookeeper client principals
      options.configurations['ams-hbase-security-site']['ams.zookeeper.principal'] ?= "amszk/_HOST@#{options.krb5.realm}"
      options.configurations['ams-hbase-security-site']['ams.zookeeper.keytab'] ?= "/etc/security/keytabs/ams-zk.service.keytab"
      # hbase embedded principal
      # master
      options.configurations['ams-hbase-security-site']['hbase.master.kerberos.principal'] ?= "amshbase/_HOST@#{options.krb5.realm}"
      options.configurations['ams-hbase-security-site']['hbase.master.keytab.file'] ?= "/etc/security/keytabs/ams-hbase.master.keytab"
      # regionserver
      options.configurations['ams-hbase-security-site']['hbase.regionserver.kerberos.principal'] ?= "amshbase/_HOST@#{options.krb5.realm}"
      options.configurations['ams-hbase-security-site']['hbase.regionserver.keytab.file'] ?= "/etc/security/keytabs/ams-hbase.regionserver.keytab"
      # client
      options.configurations['ams-hbase-security-site']['hbase.myclient.principal'] ?= "amshbase/_HOST@#{options.krb5.realm}"
      options.configurations['ams-hbase-security-site']['hbase.myclient.keytab'] ?= "/etc/security/keytabs/ams.collector.keytab"


## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.takeover = service.deps.ambari_server.options.takeover
      options.baremetal = service.deps.ambari_server.options.baremetal

## Ambari Metrics Service Configuration

      for srv in service.deps.metrics_service

        #register host
        srv.options.collector_hosts ?= []
        srv.options.collector_hosts.push service.node.fqdn if srv.options.collector_hosts.indexOf(service.node.fqdn) is -1

## Wait

      options.wait = {}
      options.wait_ambari_rest = service.deps.ambari_server.options.wait.rest

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
