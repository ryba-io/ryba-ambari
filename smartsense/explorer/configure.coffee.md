
# Ambari Logsearch Server Configuration

    module.exports = (service) ->
      options = service.options

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.conf_dir ?= '/etc/smartsense-activity/conf'
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.configurations ?= {}
      options.configurations['activity-zeppelin-site'] ?= {}
      options.download = service.deps.smartsense_service[0].options.download

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## SSL
  
      options.ssl = merge {}, service.deps.ssl.options, options.ssl 
      options.ssl.enabled ?= !!service.deps.ssl
      # options.truststore ?= {}
      throw Error "Required Option: ssl.cert" if  not options.ssl.cert
      throw Error "Required Option: ssl.key" if not options.ssl.key
      throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
      throw Error "Required Property: keystore.password" if not options.ssl.keystore.password
      throw Error "Required Property: truststore.password" if not options.ssl.truststore.password
      options.configurations['activity-zeppelin-site'] ?= {}
      options.configurations['activity-zeppelin-site']['zeppelin.ssl.key.manager.password'] ?= options.ssl.keystore.password
      options.configurations['activity-zeppelin-site']['zeppelin.ssl'] ?= 'true'
      options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.path'] ?= '/etc/security/serverKeys/activity-explorer-keystore'
      options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.password'] ?= options.ssl.keystore.password
      options.configurations['activity-zeppelin-site']['zeppelin.ssl.truststore.path'] ?= '/etc/security/serverKeys/activity-explorer-truststore'
      options.configurations['activity-zeppelin-site']['zeppelin.ssl.truststore.password'] ?= options.ssl.truststore.password

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
