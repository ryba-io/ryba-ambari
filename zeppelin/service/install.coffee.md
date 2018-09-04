
# Ambari Logsearch Install

    module.exports =  header: 'Ambari Zeppelin Install', handler: ({options}) ->
    
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



      
## Upload Default Configuration

      @ambari.configs.default
        header: 'ZEPPELIN Configuration'
        url: options.ambari_url
        if: options.post_component and options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        stack_name: options.stack_name
        stack_version: options.stack_version
        discover: true
        configurations: options.configurations
        target_services: 'ZEPPELIN'

      @ambari.kerberos.descriptor.update
        header: 'Kerberos Artifact'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        stack_name: options.stack_name
        stack_version: options.stack_version
        cluster_name: options.cluster_name
        service: 'ZEPPELIN'
        identities: options.identities['zeppelin_master']
        source: 'COMPOSITE'

      @krb5.addprinc options.krb5.admin,
        header: 'Create Principal'
        principal: options.krb5.principal
        password: options.krb5.password

      @krb5.ktutil.add options.krb5.admin,
        header: 'ZEPPELIN Server keytab'
        principal: options.krb5.principal
        password: options.krb5.password
        keytab: options.krb5.keytab
        kadmin_server: options.krb5.admin.admin_server
        mode: 0o0640
        uid: options.user.name
        gid: options.hadoop_group.name 

## Add LOGSEARCH Service

      @ambari.services.add
        header: 'ZEPPELIN Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'ZEPPELIN'

      @ambari.services.wait
        header: 'ZEPPELIN Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'ZEPPELIN'

      @ambari.services.component_add
        header: 'ZEPPELIN_MASTER Add'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZEPPELIN_MASTER'
        service_name: 'ZEPPELIN'
        
        
      for host in options.master_hosts
        @ambari.hosts.component_add
          header: 'ZEPPELIN_MASTER Host Add'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'ZEPPELIN_MASTER'
          hostname: host

## Dependencies

    ssh2fs = require 'ssh2-fs'
    {merge} = require 'nikita/lib/misc'