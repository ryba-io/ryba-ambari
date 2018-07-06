
# Configure

    module.exports = (service) ->
      options = service.options

## Identities

By default, merge group and user from the Ranger admin configuration.

      options.group = merge {}, service.deps.ranger_admin[0].options.group, options.group
      options.user = merge {}, service.deps.ranger_admin[0].options.user, options.user
      options.fqdn = service.node.fqdn

## Environment

      options.conf_dir ?= '/etc/ranger/usersync/conf'
      options.log_dir ?= '/var/log/ranger'
      options.pid_dir ?= '/var/run/ranger'
      options.site ?= {}
      options.install ?= {}
      options.site ?= {}

Setup Scripts are used to install ranger-usersync tool. Setup scripts read properties 
from two files:
* First is `/usr/hdp/current/ranger-usersync/install.properties` file (documented).
* Second is `/usr/hdp/current/ranger-usersync/conf.dist/ranger-usersync-default.xml`.
Setup process creates files in `/etc/ranger/usersync/conf` dir and outputs final
 properties to `ranger-ugsync-site.xml` file.

## Policy Admin Tool

      options.install['POLICY_MGR_URL'] ?= service.deps.ranger_admin[0].options.install['policymgr_external_url']


## User Synchronization Process

      options.install['unix_user'] ?= options.user.name
      options.install['unix_group'] ?= options.group.name
      options.install['hadoop_conf'] ?= '/etc/hadoop/conf'
      options.install['logdir'] ?= '/var/log/ranger/usersync'

Nonetheless some of the properties are hard coded to `/usr/hdp/current/ranger-usersync/setup.py`
file. Administrators can override following properties.

      setup = options.setup ?= {}
      setup['pidFolderName'] ?= options.pid_dir
      setup['logFolderName'] ?= options.log_dir


SSl properties are not documented, they are extracted from setup.py scripts.

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
        options.default['ranger.usersync.keystore.file'] ?= "/etc/security/serverKeys/ranger-usersync-keystore"
        options.default['ranger.usersync.keystore.password'] ?= options.ssl.keystore.password
        options.default['ranger.usersync.truststore.file'] ?= "/etc/security/serverKeys/ranger-usersync-truststore"
        options.default['ranger.usersync.truststore.password'] ?= options.ssl.truststore.password

## Kerberos

      options.krb5 ?= {}
      options.krb5.enabled ?= service.deps.hadoop_core.options.core_site['hadoop.security.authentication'] is 'kerberos'
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin = service.deps.krb5_client.options.admin[options.krb5.realm]
      options.krb5.principal ?= "rangerusersync/_HOST@#{options.krb5.realm}"
      options.krb5.keytab ?= '/etc/security/keytabs/rangerusersync.service.keytab'
      
## Env

      options.heap_size ?= '256m'
      options.opts ?= {}
      options.opts['javax.net.ssl.trustStore'] ?= '/etc/hadoop/conf/truststore'
      options.opts['javax.net.ssl.trustStorePassword'] ?= 'ryba123'

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

[ambari-conf-example]:(https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.0/bk_Ranger_Install_Guide/content/ranger-usersync_settings.html)
[ranger-usersync]:(http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_installing_manually_book/content/install_and_start_user_sync_ranger.html)
