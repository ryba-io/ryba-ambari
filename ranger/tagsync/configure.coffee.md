
# Configure

    module.exports = (service) ->
      options = service.options

## Identities

By default, merge group and user from the Ranger admin configuration.

      options.group = merge {}, service.deps.ranger_hdpadmin[0].options.group, options.group
      options.user = merge {}, service.deps.ranger_hdpadmin[0].options.user, options.user
      options.fqdn = service.node.fqdn

## Configuration

      options.configurations ?= {}
      options.configurations['atlas-tagsync-ssl'] ?= {}

## SSL

      options.ssl = merge {}, service.deps.ssl.options, options.ssl
      options.ssl.enabled ?= !!service.deps.ssl
      if options.ssl.enabled
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        throw Error "Required Property: keystore.password" if not options.ssl.keystore.password
        throw Error "Required Property: truststore.password" if not options.ssl.truststore.password
        options.default ?= {}
        # options.default['options.ssl'] ?= 'true'
        options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.keystore'] ?= "/etc/security/serverKeys/ranger-tagsync-keystore"
        options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.keystore.password'] ?= options.ssl.keystore.password
        options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.truststore'] ?= "/etc/security/serverKeys/ranger-tagsync-truststore"
        options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.truststore.password'] ?= options.ssl.truststore.password
        options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.keystore.credential.file'] ?= 'jceks://file{{atlas_tagsync_credential_file}}'

## Kerberos

      options.krb5 ?= {}
      options.krb5.enabled ?= service.deps.hadoop_core.options.core_site['hadoop.security.authentication'] is 'kerberos'
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin = service.deps.krb5_client.options.admin[options.krb5.realm]
      options.krb5.principal ?= "rangertagsync/_HOST@#{options.krb5.realm}"
      options.krb5.keytab ?= '/etc/security/keytabs/rangertagsync.service.keytab'

## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.takeover = service.deps.ambari_server.options.takeover
      options.baremetal = service.deps.ambari_server.options.baremetal

## Dependencies

    path = require 'path'
    {merge} = require 'nikita/lib/misc'

