

# Ambari Logsearch Server Install

    module.exports =  header: 'Ambari Ranger Tagsync Install', handler: (options) ->
      
## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'

## Configuration

      @ambari.configs.update
        header: 'atlas-tagsync-ssl'
        debug: true
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'atlas-tagsync-ssl'
        cluster_name: options.cluster_name
        properties: options.configurations['atlas-tagsync-ssl']

      @call (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/tagsync-log4j.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              # options.configurations = merge {}, options.configurations,
              #   'ranger-solr-configuration':
              #     'content': content
              # 
              @ambari.configs.update
                header: 'tagsync-log4j'
                debug: true
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'tagsync-log4j'
                cluster_name: options.cluster_name
                properties:
                  content: content.toString()
              @next callback
            catch err
              callback err
## SSL

      @call header: 'SSL', retry: 0, ->
        @java.keystore_add
          keystore: options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.truststore']
          storepass: options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name

        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
          uid: options.user.name
          gid: options.group.name
        @java.keystore_add
          keystore: options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['atlas-tagsync-ssl']['xasecure.policymgr.clientssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name
          
## Dependencies

    ssh2fs = require 'ssh2-fs'
      
