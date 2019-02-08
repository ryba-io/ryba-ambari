
# Knox Configure

    module.exports = (service) ->
      options = service.options

## Identities

      options.user ?= merge {}, service.deps.knox[0].options.user, options.user
      options.group ?= merge {}, service.deps.knox[0].options.group, options.group

## Environment

      # Layout
      options.conf_dir ?= service.deps.knox[0].options.conf_dir
      options.log_dir ?= service.deps.knox[0].options.log_dir
      options.pid_dir ?= service.deps.knox[0].options.pid_dir
      options.bin_dir ?= service.deps.knox[0].options.bin_dir
      # Misc
      options.fqdn = service.node.fqdn
      options.hostname = service.node.hostname
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'

## Kerberos

      if service.deps.krb5_client
        options.krb5 ?= merge {}, service.deps.knox[0].options.krb5, options.krb5
        options.krb5_user ?= merge {}, service.deps.knox[0].options.krb5_user, options.krb5_user
        throw Error 'Required Options: "realm"' unless options.krb5.realm
        throw Error 'Required Options: "krb5_user.principal"' unless options.krb5_user.principal
        throw Error 'Required Options: "krb5_user.keytab"' unless options.krb5_user.keytab

## Test

      options.ranger_admin ?= service.deps.ranger_admin.options.admin if service.deps.ranger_admin
      options.test = merge {}, service.deps.test_user.options, service.deps.knox[0].options.test, options.test
      throw Error 'Missing options: test.user.password' unless options.test.user.password
      throw Error 'Password length must be higher or equal to 8' unless options.test.user.password.length >= 8
      if service.deps.ranger_admin?
        service.deps.ranger_admin.options.users ?= {}
        service.deps.ranger_admin.options.users[options.test.user.name] ?=
          "name": options.test.user.name
          "firstName": options.test.user.name
          "lastName": 'hadoop'
          "emailAddress": "#{options.test.user.name}@hadoop.ryba"
          "password": options.test.user.password
          'userSource': 1
          'userRoleList': ['ROLE_USER']
          'groups': []
          'status': 1

## Env

Knox reads its own env variable to retrieve configuration.

      options.env ?= {}
      options.env.app_mem_opts ?= '-Xmx8192m'
      options.env.app_log_dir ?= "#{options.log_dir}"
      options.env.app_log_opts ?= ''
      options.env.app_dbg_opts ?= ''

## Java

      options.java_home = service.deps.java.options.java_home
      options.jre_home = service.deps.java.options.jre_home

## SSL

      options.ssl = merge {}, service.deps.ssl?.options, options.ssl
      options.ssl.enabled ?= !!service.deps.ssl
      # options.truststore ?= {}
      if options.ssl.enabled
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        throw Error "Required Property: keystore.password" if not options.ssl.keystore.password
        throw Error "Required Property: truststore.password" if not options.ssl.truststore.password
      

## Configuration

      # Configuration
      options.gateway_site ?= {}
      options.gateway_site['gateway.port'] ?= '8443'
      options.gateway_site['gateway.path'] ?= 'gateway'
      options.gateway_site['java.security.krb5.conf'] ?= '/etc/krb5.conf'
      options.gateway_site['java.security.auth.login.config'] ?= "#{options.conf_dir}/krb5JAASLogin.conf"
      options.gateway_site['gateway.hadoop.kerberos.secured'] ?= 'true'
      options.gateway_site['sun.security.krb5.debug'] ?= 'true'

## Proxy Users

      enrich_proxy_user = (srv) ->
        srv.options['configurations'] ?= {}
        srv.options['configurations']['core-site']["hadoop.proxyuser.#{options.user.name}.groups"] ?= '*'
        hosts = srv.options['configurations']['core-site']["hadoop.proxyuser.#{options.user.name}.hosts"] or []
        hosts = hosts.split ',' unless Array.isArray hosts
        for instance in service.instances
          hosts.push instance.node.fqdn unless instance.node.fqdn in hosts
        hosts = hosts.join ','
        srv.options['configurations']['core-site']["hadoop.proxyuser.#{options.user.name}.hosts"] ?= hosts
      enrich_proxy_user srv for srv in service.deps.hdfs
      for srv in service.deps.httpfs or []
        srv.options.httpfs_site["httpfs.proxyuser.#{options.user.name}.groups"] ?= '*'
        hosts = srv.options.httpfs_site["httpfs.proxyuser.#{options.user.name}.hosts"] or []
        hosts = hosts.split ',' unless Array.isArray hosts
        for instance in service.instances
          hosts.push instance.node.fqdn unless instance.node.fqdn in hosts
        hosts = hosts.join ','
        srv.options.httpfs_site["httpfs.proxyuser.#{options.user.name}.hosts"] ?= hosts
      for srv in service.deps.oozie or []
        srv.options.configurations['oozie-site']["oozie.service.ProxyUserService.proxyuser.#{options.user.name}.groups"] ?= '*'
        hosts = srv.options.configurations['oozie-site']["oozie.service.ProxyUserService.proxyuser.#{options.user.name}.hosts"] or []
        hosts = hosts.split ',' unless Array.isArray hosts
        for instance in service.instances
          hosts.push instance.node.fqdn unless instance.node.fqdn in hosts
        hosts = hosts.join ','
        srv.options.configurations['oozie-site']["oozie.service.ProxyUserService.proxyuser.#{options.user.name}.hosts"] ?= hosts


## Ambari Knox Topologies

      options.topologies ?= service.deps.knox[0].options.topologies
      options.realm_passwords ?= merge {} , service.deps.knox[0].options.realm_passwords, options.realm_passwords

## Ambari configurations
      
      options.configurations ?= {}

## Wait

      options.wait_ranger_admin = service.deps.ranger_admin.options.wait if service.deps.ranger_admin
      options.wait ?= {}
      options.wait.tcp = for srv in service.deps.knox_server
        host: srv.node.fqdn
        port: options.gateway_site['gateway.port']

## Dependencies

    appender = require 'ryba/lib/appender'
    {merge} = require 'nikita/lib/misc'

[knox-conf-example]:https://github.com/apache/knox/blob/master/gateway-release/home/templates/sandbox.knoxrealm2.xml
