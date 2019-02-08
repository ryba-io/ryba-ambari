
# Ambari Metrics Collector Configuration

    module.exports = (service) ->
      options = service.options

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.conf_dir ?= '/etc/ambari-server/conf'
      options.sudo ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.master_key ?= null
      options.admin ?= {}
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



## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
