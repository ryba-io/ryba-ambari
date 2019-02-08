
# Ambari Infra Instance Configuration

    module.exports = (service) ->
      options = service.options

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.sudo ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.configurations ?= {}
      options.configurations['infra-solr-env'] ?= {}
      options.configurations['infra-solr-env']['infra_solr_port'] ?= '8886'

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      options.configurations['infra-solr-env']['infra_solr_kerberos_keytab'] ?= "/etc/security/keytabs/ambari-infra-solr.service.keytab"
      options.configurations['infra-solr-env']['infra_solr_kerberos_principal'] ?= "infra-solr/_HOST@#{options.krb5.realm}"
      options.configurations['infra-solr-env']['infra_solr_web_kerberos_keytab'] ?= "/etc/security/keytabs/spnego.service.keytab"
      options.configurations['infra-solr-env']['infra_solr_web_kerberos_principal'] ?= "HTTP/_HOST@#{options.krb5.realm}"

## SSL
  
      options.ssl = merge {}, service.deps.ssl.options, options.ssl 
      options.ssl.enabled ?= !!service.deps.ssl
      # options.truststore ?= {}
      if options.ssl.enabled
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        throw Error "Required Property: keystore.password" if not options.ssl.keystore.password
        throw Error "Required Property: truststore.password" if not options.ssl.truststore.password
        options.configurations['infra-solr-env']['infra_solr_ssl_enabled'] ?= 'Yes'
        options.configurations['infra-solr-env']['infra_solr_truststore_location'] ?= '/etc/security/serverKeys/infra.solr.truststore.jks'
        options.configurations['infra-solr-env']['infra_solr_truststore_password'] ?= options.ssl.truststore.password
        options.configurations['infra-solr-env']['infra_solr_keystore_location'] ?= '/etc/security/serverKeys/infra.solr.keystore.jks'
        options.configurations['infra-solr-env']['infra_solr_keystore_password'] ?= options.ssl.keystore.password

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
