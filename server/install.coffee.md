
# Ambari Server Install

See the Ambari documentation relative to [Software Requirements][sr] before
executing this module.

    module.exports = header: 'Ambari Server Install', handler: ({options}) ->
      
## Registry

| Service    | Port  | Proto | Parameter       |
|------------|-------|-------|-----------------|
| HST SERVER | 9000  |  tcp  |  HTTP Port      |
| HST COM    | 9440  |  tcp  |  HTTPS Port     |
| Analyzer   | 9060  |  tcp  |  HTTPS Port     |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      @tools.iptables
        header: 'Iptables'
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: 9000, protocol: 'tcp', state: 'NEW', comment: "SMARTSENSE SERVER" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: 9440, protocol: 'tcp', state: 'NEW', comment: "SMARTSENSE AGENT" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: 9060, protocol: 'tcp', state: 'NEW', comment: "Acitivty Analyzer" }
        ]
        if: options.iptables


      @registry.register ['ambari','cluster','add'], "ryba-ambari-actions/lib/cluster/add"
      @registry.register ['ambari','cluster','provisioning_state'], "ryba-ambari-actions/lib/cluster/provisioning_state"
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','configs','groups_add'], 'ryba-ambari-actions/lib/configs/groups/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'

      options.cluster_env_stack_properties['stack_features'] = fs.readFileSync("#{options.stack_features_file}").toString()
      # options.cluster_env_stack_properties['stack_tools'] = fs.readFileSync('/home/bakalian/ryba/ryba-env-metal/resources/stack_tools.json').toString()
      options.cluster_env_stack_properties['repo_suse_rhel_template'] = fs.readFileSync("#{options.stack_repo_suse_file}").toString()
      options.cluster_env_stack_properties['stack_packages'] = fs.readFileSync("#{options.stack_package}").toString()
      options.cluster_env_stack_properties['stack_tools'] = fs.readFileSync("#{options.stack_tools}").toString()

      @ambari.cluster.add
        header: 'Cluster add'
        if: options.takeover
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


      
      @system.execute
        header: 'Keberos Credential'
        if: options.takeover
        cmd: """
          curl --request POST \
            -u admin:#{options.ambari_admin_password} \
            --insecure \
            --url #{options.ambari_url}/api/v1/clusters/#{ options.cluster_name}/credentials/kdc.admin.credential \
            --header 'x-requested-by: ambari' \
            --data '{"Credential":{"principal":"#{options.krb5.admin.kadmin_principal}","key":"#{options.krb5.admin.kadmin_password}","type":"persisted"}}'
        """
      
      @ambari.kerberos.descriptor.update
        header: 'Kerberos Artifact'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        stack_name: options.stack_name
        stack_version: options.stack_version
        cluster_name: options.cluster_name
        identities: options.identities['smokeuser']
        source: 'COMPOSITE'
              
      @ambari.services.add
        header: 'KERBEROS Service'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'KERBEROS'

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


      @krb5.addprinc options.krb5.admin,
        header: 'Explorer keytab'
        principal:  options.explorer_user.principal.replace '_HOST', options.fqdn
        randkey: true
        keytab:  options.explorer_user.keytab
        uid: options.explorer_user.name
        gid: options.explorer_group.name

      @krb5.addprinc options.krb5.admin,
        header: 'Activity keytab'
        principal:  options.analyzer_user.principal.replace '_HOST', options.fqdn
        randkey: true
        keytab:  options.analyzer_user.keytab
        uid: options.analyzer_user.name
        gid: options.analyzer_group.name

      @call
        if: options.importCerts?
      , (_, cb) ->
        {truststore} = options
        tmp_location = "/tmp/ryba_cacert_#{Date.now()}"
        @each options.importCerts, ({options}, callback) ->
          {source, local, name} = options.value
          @file.download
            header: 'download cacert'
            source: source
            target: "#{tmp_location}/cacert"
            local: true
          @java.keystore_add
            header: "add cacert to #{name}"
            keystore: truststore.target
            storepass: truststore.password
            caname: name
            cacert: "#{tmp_location}/cacert"
          @next callback
        @system.remove
          target: tmp_location
        @next cb

## Dependencies

    fs = require 'fs'

[sr]: http://docs.hortonworks.com/HDPDocuments/Ambari-2.2.2.0/bk_Installing_HDP_AMB/content/_meet_minimum_system_requirements.html
