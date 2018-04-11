
# Ambari Server Configuration

## Minimal Example

```json
{ "config": {
  "admin_password": "MySecret"
  "db": {
    "password": "MySecret"
  }
} }
```

## Database Encryption

```json
{ "config": {
  "master_key": "MySecret",
} }
```

## LDAP Connection

```json
{ "config": {
  "client.security": "ldap",
  "authentication.ldap.useSSL": true,
  "authentication.ldap.primaryUrl": "master3.ryba:636",
  "authentication.ldap.baseDn": "ou=users,dc=ryba",
  "authentication.ldap.bindAnonymously": false,
  "authentication.ldap.managerDn": "cn=admin,ou=users,dc=ryba",
  "authentication.ldap.managerPassword": "XXX",
  "authentication.ldap.usernameAttribute": "cn"
} }
```

    module.exports = (service) ->
      options = service.options

      options.group = merge service.deps.ambari_server_local.options.group, options.group
      options.user = merge service.deps.ambari_server_local.options.user, options.user
      options.test_user = merge service.deps.ambari_server_local.options.test_user, options.test_user
      options.test_group = merge service.deps.ambari_server_local.options.test_group, options.test_group

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.conf_dir ?= '/etc/ambari-server/conf'
      options.sudo ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.master_key ?= null
      options.admin ?= {}
      options.krb5 ?= merge {}, service.deps.ambari_server_local.options.krb5, options.krb5

## Client Rest API Url

      options.ambari_url ?= service.deps.ambari_server_local.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server_local.options.admin_password

      throw Error 'Required options.cluster_name for provisioning' unless options.cluster_name
      throw Error 'Required options.stack_name  (HDP) for provisioning' unless options.stack_name
      throw Error 'Required options.stack_version  (2.6) for provisioning' unless options.stack_version

## Config cluster-env

      options.configurations ?= {}
      #tells ambari that packages are already installed
      service.deps.ambari_server_local.options.config['packages.pre.installed'] ?= true
      options.cluster_env_global_properties ?=
        'override_uid': 'false'
        'smokeuser': options.test_user.name
        'host_sys_prepped': 'false'
        'sysprep_skip_create_users_and_groups': 'true'
        "security_enabled" : "true"
        "user_group" : "hadoop"
        "agent_mounts_ignore_list" : ""
        "alerts_repeat_tolerance" : "1"
        "fetch_nonlocal_groups" : "false"
        "ignore_groupsusers_create" : "false"
        "ignore_bad_mounts" : "false"
        "manage_dirs_on_root" : "true"
        "managed_hdfs_resource_property_names" : ""
        "one_dir_per_partition" : "false"
        "recovery_enabled" : "true"
        "recovery_lifetime_max_count" : "1024"
        "recovery_max_count" : "6"
        "recovery_retry_interval" : "5"
        "recovery_type" : "AUTO_START"
        "recovery_window_in_minutes" : "60"
        "repo_ubuntu_template" : "{{package_type}} {{base_url}} {{components}}"

      options.cluster_env_stack_properties ?=
        "stack_root" : JSON.stringify(HDP: "/usr/hdp")

## Kerberos Service provisioning

      if options.krb5?
        # options.test_user.principal ?= "#{options.test_user.name}@#{options.krb5.realm}"
        options.test_user.principal ?= "#{options.test_user.name}-#{options.cluster_name}@#{options.krb5.realm}"
        options.test_user.password ?= 'ambari-qa123'
        options.test_user.keytab ?= '/etc/security/keytabs/smokeuser.headless.keytab'
        options.cluster_env_global_properties['smokeuser_principal_name'] ?= options.test_user.principal
        options.cluster_env_global_properties['smokeuser_keytab'] ?= options.test_user.keytab

## Config krb5-conf

        options.configurations['krb5-conf'] ?= {}
        options.configurations['krb5-conf']['manage_krb5_conf'] ?= 'false'

## Config kerberos-env
        
        options.configurations['kerberos-env'] ?= {}
        options.configurations['kerberos-env']['realm'] ?= options.krb5.realm
        options.configurations['kerberos-env']['executable_search_paths'] ?= '/usr/bin, /usr/kerberos/bin, /usr/sbin, /usr/lib/mit/bin, /usr/lib/mit/sbin'
        options.configurations['kerberos-env']['install_packages'] ?= 'false'
        options.configurations['kerberos-env']['kdc_type'] ?= 'mit-kdc'
        # test with manage identities to true
        options.configurations['kerberos-env']['manage_identities'] ?= 'false'
        options.configurations['kerberos-env']['manage_auth_to_local'] ?= 'false'
        options.configurations['kerberos-env']['keytab_dir'] ?= '/etc/security/keytabs'
        options.configurations['kerberos-env']['kdc_hosts'] ?= service.deps.krb5_client.options.wait.kdc_tcp.map( (kdc) -> "#{kdc.host}:#{kdc.port}").join(',')
        options.configurations['kerberos-env']['admin_server_host'] ?= options.krb5.admin.admin_server
          
          # {"find": "terms", "field": "@hostname", "query": ''}
## Kerberos Descriptor
Ambari organizes principal and keytab configuration known as Kerberos descriptor,
into three parts:
  - STACK:
    It holds Ambari predefined configurations for principal names
  - USER:
    It is the user defined configurations which could be empty
  - COMPOSITE:
    IT gather all STACK properties overrided with USER's

These properties can be read from the API with the following url:
  `https://master01.metal.ryba:8442/api/v1/clusters/ryba_test/kerberos_descriptors`
  
However this endpoints are only read only. Tp set the USER part, the Ambari's administrator
must use the [artifacts_data endpoint](https://community.hortonworks.com/content/supportkb/49441/missing-kerberos-descriptor-when-using-the-rest-ap.html)
the POST Object should look like.
```json

{
 "artifact_data" : {
    "identities" : [
      {
        "principal" : {
          "configuration" : null,
          "type" : "service",
          "local_username" : null,
          "value" : "HTTP/_HOST@${realm}"
        },
        "name" : "spnego",
        "keytab" : {
          "owner" : {
            "access" : "r",
            "name" : "root"
          },
          "file" : "${keytab_dir}/spnego.service.keytab",
          "configuration" : null,
          "group" : {
            "access" : "r",
            "name" : "${cluster-env/user_group}"
          }
        },
              {
        "principal" : {
          "configuration" : "cluster-env/smokeuser_principal_name",
          "type" : "user",
          "local_username" : "${cluster-env/smokeuser}",
          "value" : "${cluster-env/smokeuser}-${cluster_name}@${realm}"
        },
        "name" : "smokeuser",
        "keytab" : {
          "owner" : {
            "access" : "r",
            "name" : "${cluster-env/smokeuser}"
          },
          "file" : "${keytab_dir}/smokeuser.headless.keytab",
          "configuration" : "cluster-env/smokeuser_keytab",
          "group" : {
            "access" : "r",
            "name" : "${cluster-env/user_group}"
          }
        }
      }
```

To configure it inside RYBA, `options.identities` dictionnary is used.
The key is the identity name, and the value should contain principal informations.
    
        options.identities ?= {}
        options.identities['smokeuser'] ?= {}
        options.identities['smokeuser']['keytab'] ?= {}
        options.identities['smokeuser']['keytab']['owner'] ?= {}
        options.identities['smokeuser']['keytab']['owner']['access'] ?= 'r' 
        options.identities['smokeuser']['keytab']['owner']['name'] ?= '${cluster-env/smokeuser}' 
        options.identities['smokeuser']['keytab']['group'] ?= {}
        options.identities['smokeuser']['keytab']['group']['access'] ?= 'r'
        options.identities['smokeuser']['keytab']['group']['name'] ?= '${cluster-env/user_group}'
        options.identities['smokeuser']['keytab']['file'] ?= "${keytab_dir}/smokeuser.headless.keytab"
        options.identities['smokeuser']['keytab']['configuration'] ?= 'cluster-env/smokeuser_keytab'
        options.identities['smokeuser']['principal'] ?= {}
        options.identities['smokeuser']['principal']['configuration'] ?= 'cluster-env/smokeuser_principal_name'
        options.identities['smokeuser']['principal']['type'] ?= 'user'
        options.identities['smokeuser']['principal']['local_username'] ?= '${cluster-env/smokeuser}'
        options.identities['smokeuser']['principal']['value'] ?= '${cluster-env/smokeuser}@${realm}'

## Config Groups

      options.config_groups ?= {}

## Ambari Alert Definitions fix

      options.alerts_definitions ?= {}
      options.alerts_definitions['mapreduce_history_server_process'] ?=
        AlertDefinition:
          source:
            default_port: '19889'
            uri : "{{mapred-site/mapreduce.jobhistory.webapp.address}}"

## Wait

      options.wait = {}
      options.wait.rest = for srv in service.deps.ambari_server
        clusters_url: url.format
          protocol: if srv.options.config['api.ssl'] is true
          then 'https'
          else 'http'
          hostname: srv.options.fqdn
          port: if srv.options.config['api.ssl'] is true
          then srv.options.config['client.api.ssl.port']
          else srv.options.config['client.api.port']
          pathname: '/api/v1/clusters'
        oldcred: "admin:#{srv.options.current_admin_password}"
        newcred: "admin:#{srv.options.admin_password}"

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
