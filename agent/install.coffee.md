# Ambari Agent Install

The ambari server must be set in the configuration file.

    module.exports = header: 'Ambari Agent Install', handler: (options) ->

## Registry

      @registry.register ['ambari','cluster','node_add'], 'ryba-ambari-actions/lib/cluster/node_add'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','hosts','rack'], 'ryba-ambari-actions/lib/hosts/rack'

## Wait

      @call 'ryba/ambari/server/wait', rest: options.wait_ambari_rest


## Identities

By default, the "ambari-agent" package does not create any identities.
Create System Service Account & user/client accounts

      @system.group  header: 'Test Group', options.test_group
      @system.user  header: 'Test User', options.test_user

      @system.group  header: 'Analyzer Group', options.analyzer_group
      @system.user  header: 'Analyzer User', options.analyzer_user

      @system.group  header: 'Explorer Group', options.explorer_group
      @system.user  header: 'Explorer User', options.explorer_user
      @call ->
        for name, group of options.groups
          @system.group
            if: (group.uid in options.only) or (group.name in options.only) or (options.only.length is 0)
            header: "Group #{name}", group
        for name, user of options.users
          @system.user
            if: (user.uid in options.only) or (user.name in options.only) or (options.only?.length is 0)
            header: "User #{name}", user

## Kerberos Test User
Create ambari-qa principal with its keytab

      @krb5.addprinc options.krb5.admin,
        header: 'Smokeuser principal'
        principal: options.test_user.principal
        password: options.test_user.password
        
      @krb5.ktutil.add options.krb5.admin,
        header: 'Smokeuser keytab'
        principal: options.test_user.principal
        password: options.test_user.password
        keytab: options.test_user.keytab
        kadmin_server: options.krb5.admin.admin_server
        mode: 0o0644
        uid: 'root'
        gid: 'root'

## Add Hosts

      @ambari.hosts.add
        header: 'Register host'
        if: options.takeover or options.baremetal
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        hostname: options.fqdn

      @ambari.cluster.node_add
        header: "Add host to Cluster"
        if: options.takeover or options.baremetal
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        hostname: options.fqdn
        cluster_name: options.cluster_name

## Rack info

      @ambari.hosts.rack
        header: "Set rack"
        if: options.rack_info
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        hostname: options.fqdn
        rack_info: options.rack_info

## Dependencies

    path = require 'path'
    misc = require 'nikita/lib/misc'
