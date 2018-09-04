
# Ambari Takeover

    module.exports = header: 'HBase Ambari Install', handler: ({options}) ->
      
## Register

      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_update'], "ryba-ambari-actions/lib/hosts/component_update"
      @registry.register ['ambari','configs','groups_add'], 'ryba-ambari-actions/lib/configs/groups/add'
      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'

      @call
        if: options.post_component
      , ->
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

## Kerberos
Create HDFS Headless keytab.

      @krb5.addprinc options.krb5.admin,
        header: 'HBase principal'
        principal: options.admin.principal
        password: options.admin.password

      @krb5.ktutil.add options.krb5.admin,
        header: 'HBase Headless keytab'
        principal: options.admin.principal
        password: options.admin.password
        keytab: options.admin.keytab
        kadmin_server: options.krb5.admin.admin_server
        mode: 0o0640
        uid: options.user.name      
        gid: options.hadoop_group.name      

## RegionServers

Upload the list of registered RegionServers.

      regionservers = for fqdn, active of options.regionservers
        continue unless active
        fqdn
      @file
        header: 'Registered RegionServers'
        target: "#{options.conf_dir}/regionservers"
        content: (
          for fqdn, active of options.regionservers
            continue unless active
            fqdn
        ).join '\n'
        uid: options.user.name
        gid: options.hadoop_group.name
        eof: true
        mode: 0o640

## Render Files

      @call
        if: options.post_component
        header: 'HBase Env'
      , ->
          HBASE_MASTER_OPTS = options.master_opts.base
          HBASE_MASTER_OPTS += " -D#{k}=#{v}" for k, v of options.master_opts.java_properties
          HBASE_MASTER_OPTS += " #{k}#{v}" for k, v of options.master_opts.jvm
          HBASE_REGIONSERVER_OPTS = options.regionserver_opts.base
          HBASE_REGIONSERVER_OPTS += " -D#{k}=#{v}" for k, v of options.regionserver_opts.java_properties
          HBASE_REGIONSERVER_OPTS += " #{k}#{v}" for k, v of options.regionserver_opts.jvm
          HBASE_OPTS = options.base
          HBASE_OPTS += " -D#{k}=#{v}" for k, v of options.opts.java_properties
          HBASE_OPTS += " #{k}#{v}" for k, v of options.opts.jvm
          @file.render
            header: 'Render'
            source: "#{__dirname}/../resources/hbase-env.sh.ambari.j2"
            target: "#{options.cache_dir}/hbase-env.sh"
            ssh: false
            context: merge options.configurations['hbase-env'],
              HBASE_MASTER_OPTS: HBASE_MASTER_OPTS
              HBASE_REGIONSERVER_OPTS: HBASE_REGIONSERVER_OPTS
              HBASE_OPTS: HBASE_OPTS

## Log4j

      @file
        header: 'HBase Log4j'
        if: options.post_component
        target: "#{options.cache_dir}/hbase-log4j.properties"
        source: "#{__dirname}/../resources/log4j.properties"
        ssh: false
        write: for k, v of options.hbase_log4j.properties
          match: RegExp "#{k}=.*", 'm'
          replace: "#{k}=#{v}"
          append: true
        
### HDFS Service
      
      @ambari.services.add
        header: 'HBASE Service'
        if: options.post_component and options.takeover 
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HBASE'

## HBASE-SITE
Update hbase-site.xml

      @ambari.configs.update
        header: 'Upload hbase-site'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hbase-site'
        cluster_name: options.cluster_name
        properties: options.configurations['hbase-site']

## HBase Log4j

      @ambari.configs.update
        header: 'Upload hbase-logj4j'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hbase-log4j'
        cluster_name: options.cluster_name
        properties: options.hbase_log4j

## HBase Policy

      @ambari.configs.update
        header: 'Upload hbase-policy'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hbase-policy'
        cluster_name: options.cluster_name
        properties: options.configurations['hbase-policy']


## HBASE-ENV
Render hadoop-env.sh and yarn-env.sh files, before uploading to Ambari Server.

      @call
        header: 'HBase Env'
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/hbase-env.sh", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Update to Ambari'
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hbase-env'
                cluster_name: options.cluster_name
                properties: merge {}, options.configurations['hbase-env'], content: content
              .next callback
            catch err
              callback err


## Log4j

      @call
        header: 'HBase Log4j'
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/hbase-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Update To ambari'
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'hbase-log4j'
                cluster_name: options.cluster_name
                properties: 
                  content: content
              .next callback
            catch err
              callback err
              
## Upload Ranger-plugin configuration to Ambari

      @ambari.configs.update
        header: 'Upload ranger-hbase-plugin-properties'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hbase-plugin-properties'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hbase-plugin-properties']

      @ambari.configs.update
        header: 'Upload ranger-hbase-security'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hbase-security'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hbase-security']

      @ambari.configs.update
        header: 'Upload ranger-hbase-policymgr-ssl'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hbase-policymgr-ssl'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hbase-policymgr-ssl']

      @ambari.configs.update
        header: 'Upload ranger-hbase-audit'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hbase-audit'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hbase-audit']

## Add Component
add HBASE_MASTER, HBASE_REGIONSERVER, HBASE_CLIENT, HBASE_REST_SERVER, HBASE_THRIFT_SERVER
components to cluster but NOT in `INSTALLED` desired state.

### Wait HBase Service

      @call
        if: options.post_component and options.takeover
      , ->

        @ambari.services.wait
          header: 'HBASE Service WAITED'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          name: 'HBASE'

        @ambari.services.component_add
          header: 'HBASE_MASTER'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HBASE_MASTER'
          service_name: 'HBASE'

        @ambari.services.component_add
          header: 'HBASE_REGIONSERVER'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HBASE_REGIONSERVER'
          service_name: 'HBASE'

        @ambari.services.component_add
          header: 'HBASE_CLIENT'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HBASE_CLIENT'
          service_name: 'HBASE'

### HBASE_MASTER COMPONENT
      
        for host in options.master_hosts
          @ambari.hosts.component_add
            header: 'HBASE_MASTER ADD'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'HBASE_MASTER'
            hostname: host

### HBASE_REGIONSERVER COMPONENT
      
        for host in options.regionserver_hosts
          @ambari.hosts.component_add
            header: 'HBASE_REGIONSERVER ADD'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'HBASE_REGIONSERVER'
            hostname: host

### HBASE_CLIENT COMPONENT
      
        for host in options.client_hosts
          @ambari.hosts.component_add
            header: 'HBASE_CLIENT ADD'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'HBASE_CLIENT'
            hostname: host

## Ambari Config Groups
          
      @call
        if: options.post_component and options.takeover
      , ->
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

## Dependencies

    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
