

# Ambari Logsearch Server Install

    module.exports =  header: 'Ambari Ranger Usersync Install', handler: (options) ->
  
## Registry

      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"


## Add components


      @call (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/usersync-log4j.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              # options.configurations = merge {}, options.configurations,
              #   'ranger-solr-configuration':
              #     'content': content
              # 
              @ambari.configs.update
                header: 'usersync-log4j'
                debug: true
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'usersync-log4j'
                cluster_name: options.cluster_name
                properties:
                  content: content.toString()
              @next callback
            catch err
              callback err
              
      @krb5.addprinc options.krb5.admin,
        header: 'Principal'
        principal: options.krb5.principal.replace '_HOST', options.fqdn
        randkey: true
        keytab:  options.krb5.keytab
        uid: options.user.name
        gid: options.user.name
        mode: 0o600

      @ambari.services.component_add
        header: 'RANGER_USERSYNC'
        url: options.ambari_url
        if: options.post_component
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'RANGER_USERSYNC'
        service_name: 'RANGER'
      
      @ambari.hosts.component_add
        header: 'RANGER_USERSYNC ADD'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'RANGER_USERSYNC'
        hostname: options.fqdn
      
      @ambari.hosts.component_wait
        header: 'RANGER_USERSYNC'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'RANGER_USERSYNC'
        hostname: options.fqdn
      
      @ambari.hosts.component_install
        header: 'RANGER_USERSYNC'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'RANGER_USERSYNC'
        hostname: options.fqdn
    
## SSL

      @call header: 'SSL', retry: 0, ->
        @java.keystore_add
          keystore: options.default['ranger.usersync.truststore.file']
          storepass: options.default['ranger.usersync.truststore.password']
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.default['ranger.usersync.keystore.file']
          storepass: options.default['ranger.usersync.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass:options.default['ranger.usersync.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
          uid: options.user.name
          gid: options.group.name
        @java.keystore_add
          keystore: options.default['ranger.usersync.keystore.file']
          storepass: options.default['ranger.usersync.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name

## Dependencies

    ssh2fs = require 'ssh2-fs'
      