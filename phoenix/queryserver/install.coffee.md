
# Phoenix QueryServer Install

Please refer to the Apache Phoenix QueryServer [documentation][phoenix-doc].

    module.exports =  header: 'Phoenix QueryServer Install', handler: ({options}) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'

## Identities

      @system.group header: 'Group', options.group
      @system.user header: 'User', options.user

## IPTables

| Service             | Port  | Proto  | Parameter                     |
|---------------------|-------|--------|-------------------------------|
| Phoenix QueryServer | 8765  | HTTP   | phoenix.queryserver.http.port |

      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.phoenix_site['phoenix.queryserver.http.port'], protocol: 'tcp', state: 'NEW', comment: "Phoenix QueryServer port" }
        ]

## Packages

      @service header: 'Packages', name: 'phoenix'
      @hdp_select name: 'phoenix-server'

## Kerberos

We use the SPNEGO keytab, so we let hadoop/core handle principal & keytab

      # @krb5.addprinc options.krb5.admin,
      #     header: 'Kerberos'
      #     if: options.phoenix_site['hbase.security.authentication'] is 'kerberos'
      #     principal: options.phoenix_site['phoenix.queryserver.kerberos.principal'].replace '_HOST', options.fqdn
      #     randkey: true
      #     keytab: options.phoenix_site['phoenix.queryserver.keytab.file']
      #     uid: options.user.name
      #     gid: options.group.name

      @ambari.configs.update
        header: 'hbase-site'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hbase-site'
        cluster_name: options.cluster_name
        properties:
          'phoenix.queryserver.kerberos.principal': options.phoenix_site['phoenix.queryserver.kerberos.principal']
          'phoenix.queryserver.keytab.file': options.phoenix_site['phoenix.queryserver.keytab.file']
          
# ## Layout
# 
#       @call header: 'Layout', ->
#         @system.mkdir
#           target: options.pid_dir
#           uid: options.user.name
#           gid: options.user.name
#         @system.mkdir
#           target: options.conf_dir
#           uid: options.user.name
#           gid: options.group.name
#         @system.mkdir
#           target: options.log_dir
#           uid: options.user.name
#           gid: options.group.name
# 
# ## Service
# 
#       @call header: 'Service', ->
#         @service.init
#           header: 'Init Script'
#           target: '/etc/init.d/phoenix-queryserver'
#           source: "#{__dirname}/../resources/phoenix-queryserver.j2"
#           local: true
#           context: options
#           mode: 0o0755
#         @system.tmpfs
#           if_os: name: ['redhat','centos'], version: '7'
#           mount: options.pid_dir
#           uid: options.user.name
#           gid: options.group.name
#           perm: '0750'
# 
# ## HBase Site
# 
#       @hconfigure
#         header: 'HBase Site'
#         target: "#{options.conf_dir}/hbase-site.xml"
#         source: "#{__dirname}/../../hbase/resources/hbase-site.xml"
#         local: true
#         properties: options.phoenix_site
#         backup: true
#         oef: true
# 
# ## Env
# 
#       @file.render
#         header: 'Env'
#         target: "#{options.conf_dir}/hbase-env.sh"
#         source: "#{__dirname}/../resources/hbase-env.sh.j2"
#         local: true
#         context: options
#         eof: true

[phoenix-doc]: https://phoenix.apache.org/server
