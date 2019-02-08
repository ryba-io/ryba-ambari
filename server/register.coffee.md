
# Ambari Server Install

    module.exports = header: 'Ambari Server Register', handler: ({options}) ->
      {ambari_admin_password , ambari_url, cluster_name, takeover, baremetal, post_component, stack_name, stack_version, configurations } = options

## Wait

      @call 'ryba-ambari-takeover/ambari/agent/wait', options.wait_ambari_agent

## Registry

      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari','cluster','provisioning_state'], "ryba-ambari-actions/lib/cluster/provisioning_state"
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','configs','groups_add'], 'ryba-ambari-actions/lib/configs/groups/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','cluster','add'], "ryba-ambari-actions/lib/cluster/add"
      @registry.register ['ambari','cluster','node_add'], 'ryba-ambari-actions/lib/cluster/node_add'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','hosts','rack'], 'ryba-ambari-actions/lib/hosts/rack'
      
      options.cluster_env_stack_properties['stack_features'] = fs.readFileSync("#{options.stack_features_file}").toString()
      # options.cluster_env_stack_properties['stack_tools'] = fs.readFileSync('/home/bakalian/ryba/ryba-env-metal/resources/stack_tools.json').toString()
      options.cluster_env_stack_properties['repo_suse_rhel_template'] = fs.readFileSync("#{options.stack_repo_suse_file}").toString()
      options.cluster_env_stack_properties['stack_packages'] = fs.readFileSync("#{options.stack_package}").toString()
      options.cluster_env_stack_properties['stack_tools'] = fs.readFileSync("#{options.stack_tools}").toString()
      
      @ambari.cluster.add
        header: 'Cluster add'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        name: options.cluster_name
        security_type: 'KERBEROS'
        version: "#{options.stack_name}-#{options.stack_version}"

      @system.execute
        header: 'VDF File'
        if: options.vdf_source? and options.takeover
        cmd: """
          curl --fail --request POST \
            -u admin:#{options.ambari_admin_password} \
            --insecure \
            --url #{options.ambari_url}/api/v1/version_definitions \
            --header 'x-requested-by: ambari' \
            --data '{
               "VersionDefinition": {
                 "version_url": "#{options.vdf_source}"
               }
              }'
        """
        code_skipped: 22

      @ambari.cluster.provisioning_state
        header: 'Set Installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        name: options.cluster_name
        provisioning_state: 'INSTALLED'

      @ambari.configs.update
        header: 'cluster-env stack'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'cluster-env'
        cluster_name: options.cluster_name
        properties: options.cluster_env_stack_properties

      @ambari.configs.update
        header: 'cluster-env main'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'cluster-env'
        cluster_name: options.cluster_name
        properties: options.cluster_env_global_properties

      @ambari.configs.update
        header: 'upload krb5-conf'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'krb5-conf'
        cluster_name: options.cluster_name
        properties: options.configurations['krb5-conf']

      @ambari.configs.update
        header: 'upload kerberos-env'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'kerberos-env'
        cluster_name: options.cluster_name
        properties: options.configurations['kerberos-env']

      @ambari.services.add
        header: 'KERBEROS Service'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'KERBEROS'

      @system.execute
        header: 'Keberos Credential'
        if: options.takeover
        cmd: """
          curl --request POST \
            -u admin:#{options.ambari_admin_password} \
            --insecure \
            --url #{options.ambari_url}/api/v1/clusters/#{options.cluster_name}/credentials/kdc.admin.credential \
            --header 'x-requested-by: ambari' \
            --data '{"Credential":{"principal":"#{options.krb5.admin.kadmin_principal}","key":"#{options.krb5.admin.kadmin_password}","type":"persisted"}}'
        """
        unless_exec: """
          curl --request GET \
            -u admin:#{options.ambari_admin_password} \
            --insecure \
            --url #{options.ambari_url}/api/v1/clusters/#{ options.cluster_name}/credentials/kdc.admin.credential \
            --header 'x-requested-by: ambari' | grep '"cluster_name" : "#{options.cluster_name}"'
          """

      @ambari.services.component_add
        header: 'KERBEROS_CLIENT'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'KERBEROS_CLIENT'
        service_name: 'KERBEROS'

## Register Hosts

      @call ->
        @each options.hosts, ({options}, cb) ->
          {key, value} = options
          @ambari.hosts.add
            header: 'Register host'
            url: ambari_url
            username: 'admin'
            password: ambari_admin_password
            hostname: key

          @ambari.cluster.node_add
            header: "Add host to Cluster"
            url: ambari_url
            username: 'admin'
            password: ambari_admin_password
            hostname: key
            cluster_name: cluster_name

          @ambari.hosts.component_add
            header: 'KERBEROS_CLIENT ADD'
            url: ambari_url
            username: 'admin'
            password: ambari_admin_password
            cluster_name: cluster_name
            component_name: 'KERBEROS_CLIENT'
            hostname: key
          @next cb

## Register Services
services:
  'HDFS':
    'DATANODE': ['master01.metal.ryba','master02.metal.ryba']
  'YARN':
    'NODEMANAGER': ['worker01.metal.ryba','master02.metal.ryba']

      @each options.services, ({options}, cb) ->
        {key, value} = options
        service = key
        components = value
        # register service
        @ambari.services.add
          header: "Register #{service}"
          if: post_component and takeover
          url: ambari_url
          username: 'admin'
          password: ambari_admin_password
          cluster_name: cluster_name
          name: service
        @ambari.services.wait
          header: "Register #{service} WAITED"
          url: ambari_url
          username: 'admin'
          password: ambari_admin_password
          cluster_name: cluster_name
          name: service
        # register components
        @each components, ({options}, cb) ->
          component = options.key
          hosts = options.value
          @ambari.services.component_add
            header: "Register #{component}"
            url: ambari_url
            username: 'admin'
            password: ambari_admin_password
            cluster_name: cluster_name
            component_name: component
            service_name: service
          @each hosts.hosts, ({options}, cb) ->
            host = options.key
            @ambari.hosts.component_add
              header: "Register #{component} host #{host}"
              url: ambari_url
              username: 'admin'
              password: ambari_admin_password
              cluster_name: cluster_name
              component_name:  component
              hostname: host
            @ambari.hosts.component_install
              header: "Register #{component} host #{host}"
              url: ambari_url
              username: 'admin'
              password: ambari_admin_password
              cluster_name: cluster_name
              component_name:  component
              hostname: host
            @next cb
          @next cb
        @next cb

## Custom Configuration

      KAFKA_HEAP_OPTS = options.kafka_env['KAFKA_HEAP_OPTS']
      @file.render
        header: 'Render spark-env'
        target: "#{options.cache_dir}/kafka-env.sh"
        source: "#{__dirname}/../kafka/resources/kafka-env.sh.ambari.j2"
        local: true
        context:
          KAFKA_HEAP_OPTS: KAFKA_HEAP_OPTS
        backup: true
        ssh:false
      @call (opts, cb) ->
        ssh2fs.readFile null, "#{options.cache_dir}/kafka-env.sh", (err, content) =>
          try
            throw err if err
            content = content.toString()
            options.configurations['kafka-env'] ?= {}
            options.configurations['kafka-env']['content'] = content
            cb()
          catch err
            cb err

## Upload Configurations
Merges default configuration from Ambari with ryba configuration

      @each options.services, ({options}, cb) ->
        {key} = options
        @ambari.configs.default
          header: "#{key} Configuration"
          url: ambari_url
          if: post_component and takeover
          username: 'admin'
          password: ambari_admin_password
          cluster_name: cluster_name
          stack_name: stack_name
          stack_version: stack_version
          discover: true
          configurations: configurations
          target_services: key
        @next cb

## Create Config groups

      {ambari_url, ambari_admin_password, cluster_name} = options
      @each options.config_groups, ({options}, cb) ->
        {key, value} = options
        @ambari.configs.groups_add
          header: "#{key}"
          url: ambari_url
          username: 'admin'
          password: ambari_admin_password
          cluster_name: cluster_name
          group_name: key
          tag: key
          description: "#{key} config groups"
          hosts: value.hosts
          desired_configs:
            type: value.type
            tag: value.tag
            properties: value.properties
        @next cb

## Rack AwareNess

      {ambari_url, ambari_admin_password, cluster_name, racks} = options
      @each options.racks, ({options}, cb) ->
        {key, value} = options
        @ambari.hosts.rack
          header: "Set rack"
          if: value?
          url: ambari_url
          username: 'admin'
          password: ambari_admin_password
          cluster_name: cluster_name
          hostname: key
          rack_info: value
        @next cb


## Dependencies

    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    db = require 'nikita/lib/misc/db'

[sr]: http://docs.hortonworks.com/HDPDocuments/Ambari-2.2.2.0/bk_Installing_HDP_AMB/content/_meet_minimum_system_requirements.html
