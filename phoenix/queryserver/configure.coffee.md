
# Phoenix QueryServer Configuration

    module.exports = (service) ->
      options = service.options

## Users and Groups

      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'phoenix'
      options.user.system ?= true
      options.user.comment ?= 'Phoenix User'
      options.user.home ?= '/var/lib/phoenix'
      options.user.groups ?= 'hadoop'
      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'phoenix'
      options.group.system ?= true
      options.user.gid = options.group.name
      options.fqdn = service.node.fqdn

## Environment

      # Layout
      options.conf_dir ?= '/etc/phoenix/conf'
      options.log_dir ?= '/var/log/phoenix'
      options.pid_dir ?= '/var/run/phoenix'
      # Misc
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## QueryServer Configuration

      options.phoenix_site ?= {}
      options.phoenix_site['phoenix.queryserver.http.port'] ?= '8765'
      options.phoenix_site['phoenix.queryserver.metafactory.class'] ?= 'org.apache.phoenix.queryserver.server.PhoenixMetaFactoryImpl'
      options.phoenix_site['phoenix.queryserver.serialization'] ?= 'PROTOBUF'
      options.phoenix_site['phoenix.queryserver.keytab.file'] ?= '/etc/security/keytabs/spnego.service.keytab'
      options.phoenix_site['phoenix.queryserver.kerberos.principal'] ?= "HTTP/_HOST@#{options.krb5.realm}"
      options.phoenix_site['avatica.connectioncache.concurrency'] ?= '10'
      options.phoenix_site['avatica.connectioncache.initialcapacity'] ?= '100'
      options.phoenix_site['avatica.connectioncache.maxcapacity'] ?= '1000'
      options.phoenix_site['avatica.connectioncache.expiryduration'] ?= '10'
      options.phoenix_site['avatica.connectioncache.expiryunit'] ?= 'MINUTES'
      options.phoenix_site['avatica.statementcache.concurrency'] ?= '100'
      options.phoenix_site['avatica.statementcache.initialcapacity'] ?= '1000'
      options.phoenix_site['avatica.statementcache.maxcapacity'] ?= '10000'
      options.phoenix_site['avatica.statementcache.expiryduration'] ?= '5'
      options.phoenix_site['avatica.statementcache.expiryunit'] ?= 'MINUTES'
      options.phoenix_site[k] ?= v for k, v of service.deps.hbase_client[0].options.hbase_site
      
## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.takeover = service.deps.ambari_server.options.takeover
      options.baremetal = service.deps.ambari_server.options.baremetal

## Other Configurations

      options.host = service.node.fqdn
      options.java_home = service.deps.java.options.java_home

## Ambari Agent - Register Hosts
Register users to ambari agent's user list.

      for srv in service.deps.ambari_agent
        srv.options.users ?= {}
        srv.options.users['phoenix'] ?= options.user
        srv.options.groups ?= {}
        srv.options.groups['phoenix'] ?= options.group

## Dependencies

    string = require 'nikita/lib/misc/string'
    {merge} = require 'nikita/lib/misc'
    appender = require 'ryba/lib/appender'
