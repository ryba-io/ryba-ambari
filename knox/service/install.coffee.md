
# Knox Install

    module.exports = header: 'Ambari Knox Install', handler: (options) ->
      console.log 'TODO put kknox takeover config'

## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'


## Upload Default Configuration BareMetal

      @ambari.configs.default
        header: 'Ambari Knox Configuration'
        url: options.ambari_url
        if: options.post_component and options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        stack_name: options.stack_name
        stack_version: options.stack_version
        configurations: options.configurations
        target_services: 'KNOX'
        discover: true

## Upload Default Configuration Takeover

      # @ambari.kerberos.descriptor.update
      #   header: 'Kerberos Artifact'
      #   if: options.post_component
      #   url: options.ambari_url
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   stack_name: options.stack_name
      #   stack_version: options.stack_version
      #   cluster_name: options.cluster_name
      #   service: 'KNOX'
      #   identities: options.identities['spark2']
      #   source: 'COMPOSITE'

      # @krb5.addprinc options.krb5.admin,
      #   header: 'Create Principal'
      #   principal: options.krb5_user.principal
      #   password: options.krb5.password
      # 
      # @krb5.ktutil.add options.krb5.admin,
      #   header: 'Ambari Knox Headless keytab'
      #   principal: options.krb5.principal
      #   password: options.krb5.password
      #   keytab: options.krb5.keytab
      #   kadmin_server: options.krb5.admin.admin_server
      #   mode: 0o0640
      #   uid: options.user.name
      #   gid: options.hadoop_group.name 

## Add KNOX Service

      @ambari.services.add
        header: 'Ambari Knox Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'KNOX'

      @ambari.services.wait
        header: 'Ambari Knox Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'KNOX'

      @ambari.services.component_add
        header: 'Ambari Knox_GATEWAY Add'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'KNOX_GATEWAY'
        service_name: 'KNOX'

## Add KNOX_GATEWAY Component

      for host in options.server_hosts
        @ambari.hosts.component_add
          header: 'Ambari Knox_GATEWAY Host Add'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'KNOX_GATEWAY'
          hostname: host

