
# Ambari Logsearch Server Configuration

    module.exports = (service) ->
      options = service.options

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.conf_dir ?= '/etc/ambari-server/conf'
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.configurations ?= {}
      options.configurations['logsearch-env'] ?= {}
      options.configurations['logsearch-common-env'] ?= {}
      options.download = service.deps.logsearch_service[0].options.download
      options.ambari_infra_instance = !!service.deps.ambari_infra_instance?.length

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      if service.deps.logsearch_service[0].options.solr_external
        if service.deps.logsearch_service[0].options.configurations['logsearch-common-env']['logsearch_external_solr_kerberos_enabled'] is 'true'
          options.configurations['logsearch-env']['logsearch_external_solr_kerberos_keytab'] ?= '/etc/security/keytabs/logsearch.service.keytab'
          options.configurations['logsearch-env']['logsearch_external_solr_kerberos_principal'] ?= "logsearch/_HOST@#{options.krb5.realm}"
      if options.ambari_infra_instance
        options.configurations['infra-solr-env'] ?= service.deps.ambari_infra_instance[0].options.configurations['infra-solr-env']

## SSL
  
      options.ssl = merge {}, service.deps.ssl.options, options.ssl 
      options.ssl.enabled ?= !!service.deps.ssl
      # options.truststore ?= {}
      if options.ssl.enabled or service.deps.logsearch_service[0].options.configurations['logsearch-common-env']['logsearch_external_solr_ssl_enabled']
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        throw Error "Required Property: keystore.password" if not options.ssl.keystore.password
        throw Error "Required Property: truststore.password" if not options.ssl.truststore.password
        options.configurations['logsearch-env'] ?= {}
        options.configurations['logsearch-env']['logsearch_ui_protocol'] ?= 'https'
        options.configurations['logsearch-env']['logsearch_ui_port'] ?= '61889'
        options.configurations['logsearch-env']['logsearch_truststore_location'] ?= '/etc/security/serverKeys/logsearch-portal-truststore'
        options.configurations['logsearch-env']['logsearch_truststore_password'] ?= options.ssl.truststore.password
        options.configurations['logsearch-env']['logsearch_keystore_location'] ?= '/etc/security/serverKeys/logsearch-portal-keystore'
        options.configurations['logsearch-env']['logsearch_keystore_password'] ?= options.ssl.keystore.password

## Logsearch Env
        
        options.configurations['logsearch-env'] ?= {}
        options.configurations['logsearch-env']['logsearch_ui_protocol'] ?= 'http'
        options.configurations['logsearch-env']['logsearch_ui_port'] ?= '61888'

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
