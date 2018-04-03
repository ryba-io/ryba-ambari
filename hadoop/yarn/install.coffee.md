
# Ambari Takeover

    module.exports = header: 'YARN Ambari Install', handler: (options) ->
      
## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari','configs','groups_add'], 'ryba-ambari-actions/lib/configs/groups/add'

## Identities

By default, the "hadoop-client" package rely on the "hadoop", "hadoop-hdfs",
"hadoop-mapreduce" and "hadoop-yarn" dependencies and create the following
entries:

```bash
cat /etc/passwd | grep hadoop
hdfs:x:496:497:Hadoop HDFS:/var/lib/hadoop-hdfs:/bin/bash
yarn:x:495:495:Hadoop Yarn:/var/lib/hadoop-yarn:/bin/bash
mapred:x:494:494:Hadoop MapReduce:/var/lib/hadoop-mapreduce:/bin/bash
cat /etc/group | egrep "hdfs|yarn|mapred"
hadoop:x:498:hdfs,yarn,mapred
hdfs:x:497:
yarn:x:495:
mapred:x:494:
```

Note, the package "hadoop" will also install the "dbus" user and group which are
not handled here.

      # for group in [options.hadoop_group, options.yarn.group]
      #   @system.group header: "Group #{group.name}", group
      # for user in [ options.yarn.user]
      #   @system.user header: "user #{user.name}", user

## Packages

Install the "hadoop-client" and "openssl" packages as well as their
dependecies.

The environment script "hadoop-env.sh" from the HDP companion files is also
uploaded when the package is first installed or upgraded. Be careful, the
original file will be overwritten with and user modifications. A copy will be
made available in the same directory after any modification.

      @call header: 'Packages', ->
        @service
          name: 'openssl-devel'
        @service
          name: 'hadoop-client'
        @hdp_select
          name: 'hadoop-client'


## Keytab Directory

      @system.mkdir
        header: 'Keytabs'
        target: '/etc/security/keytabs'
        uid: 'root'
        gid: 'root' # was hadoop_group.name
        mode: 0o0755

## SSL

      @call header: 'SSL', retry: 0, ->
        @hconfigure
          target: "#{options.conf_dir}/ssl-server.xml"
          properties: options.configurations['ssl-server']
        @hconfigure
          target: "#{options.conf_dir}/ssl-client.xml"
          properties: options.configurations['ssl-client']
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ssl-client']['ssl.client.truststore.location']
          storepass: options.configurations['ssl-client']['ssl.client.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ssl-server']['ssl.server.keystore.location']
          storepass: options.configurations['ssl-server']['ssl.server.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['ssl-server']['ssl.server.keystore.keypassword']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['ssl-server']['ssl.server.keystore.location']
          storepass: options.configurations['ssl-server']['ssl.server.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"


## Render Files

      @call
        header: 'Yarn Env'
        if: options.post_component
      , ->
          YARN_RESOURCEMANAGER_OPTS = options.yarn_rm_opts.base
          YARN_RESOURCEMANAGER_OPTS += " -D#{k}=#{v}" for k, v of options.yarn_rm_opts.java_properties
          YARN_RESOURCEMANAGER_OPTS += " #{k}#{v}" for k, v of options.yarn_rm_opts.jvm
          YARN_NODEMANAGER_OPTS = options.yarn_nm_opts.base
          YARN_NODEMANAGER_OPTS += " -D#{k}=#{v}" for k, v of options.yarn_nm_opts.java_properties
          YARN_NODEMANAGER_OPTS += " #{k}#{v}" for k, v of options.yarn_nm_opts.jvm
          YARN_TIMELINESERVER_OPTS = options.yarn_ts_opts.base
          YARN_TIMELINESERVER_OPTS += " -D#{k}=#{v}" for k, v of options.yarn_ts_opts.java_properties
          YARN_TIMELINESERVER_OPTS += " #{k}#{v}" for k, v of options.yarn_ts_opts.jvm
          @file.render
            header: 'Render'
            source: "#{__dirname}/../resources/yarn-env.sh.j2"
            target: "#{options.cache_dir}/yarn-env.sh"
            ssh: false
            context: merge options.configurations['yarn-env'],
              YARN_RESOURCEMANAGER_OPTS: YARN_RESOURCEMANAGER_OPTS
              YARN_NODEMANAGER_OPTS: YARN_NODEMANAGER_OPTS
              YARN_HISTORYSERVER_OPTS: YARN_TIMELINESERVER_OPTS
              YARN_TIMELINESERVER_OPTS: YARN_TIMELINESERVER_OPTS

## Log4j

      @file
        header: 'YARN Log4j'
        if: options.post_component
        target: "#{options.cache_dir}/yarn-log4j.properties"
        source: "#{__dirname}/../resources/log4j.properties"
        ssh: false
        write: for k, v of options.yarn_log4j.properties
          match: RegExp "#{k}=.*", 'm'
          replace: "#{k}=#{v}"
          append: true
        
## Site

      @hconfigure
        if: options.post_component
        target: "#{options.cache_dir}/ssl-server.xml"
        properties: options.configurations['ssl-server']
        ssh: false
      @hconfigure
        if: options.post_component
        target: "#{options.cache_dir}/ssl-client.xml"
        ssh: false
        properties: options.configurations['ssl-client']

### YARN Service
      
      @ambari.services.add
        header: 'YARN Service'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'YARN'

## Upload Configs

      @ambari.configs.update
        header: 'Scheduler to ambari'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'capacity-scheduler'
        cluster_name: options.cluster_name
        properties: options.configurations['capacity-scheduler']

## YARN-SITE
Update yarn-site.xml

      @ambari.configs.update
        header: 'Upload Yarn Site'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'yarn-site'
        cluster_name: options.cluster_name
        properties: options.configurations['yarn-site']
      
## Hadoop Policy

      @ambari.configs.update
        header: 'Hadoop Policy'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hadoop-policy'
        cluster_name: options.cluster_name
        properties: options.hadoop_policy

## Hadoop Log4j

      @ambari.configs.update
        header: 'YARN Log4j'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'yarn-log4j'
        cluster_name: options.cluster_name
        properties: options.yarn_log4j


## YARN-ENV
Render yarn-env.sh files, before uploading to Ambari Server.

      @call
        header: 'Yarn Env'
        if: options.post_component
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/yarn-env.sh", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Update to Ambari'
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'yarn-env'
                cluster_name: options.cluster_name
                properties: 
                  content: content
                  yarn_user: options.configurations['yarn-env']['yarn_user']
                  yarn_tmp_dir: options.configurations['yarn-env']['yarn_tmp_dir']
                  yarn_user_nofile_limit: options.configurations['yarn-env']['yarn_user_nofile_limit']
                  yarn_user_nproc_limit: options.configurations['yarn-env']['yarn_user_nproc_limit']
                  yarn_heapsize: options.configurations['yarn-env']['yarn_heapsize']
                  min_user_id: options.configurations['yarn-env']['min_user_id']
                  nodemanager_heapsize: options.configurations['yarn-env']['nodemanager_heapsize']
                  resourcemanager_heapsize: options.configurations['yarn-env']['resourcemanager_heapsize']
                  apptimelineserver_heapsize: options.configurations['yarn-env']['apptimelineserver_heapsize']
                  hadoop_yarn_home: options.configurations['yarn-env']['hadoop_yarn_home']
                  yarn_log_dir_prefix: options.configurations['yarn-env']['yarn_log_dir_prefix']
                  hadoop_libexec_dir: options.configurations['yarn-env']['hadoop_libexec_dir']
                  yarn_pid_dir_prefix: options.configurations['yarn-env']['yarn_pid_dir_prefix']
              .next callback
            catch err
              callback err


## Log4j

      @call
        header: 'YARN Log4j'
        if: options.post_component
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/yarn-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'config update'
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'yarn-log4j'
                cluster_name: options.cluster_name
                properties: 
                  content: content
              .next callback
            catch err
              callback err

      @ambari.configs.update
        header: 'Upload mapred-site'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'mapred-site'
        cluster_name: options.cluster_name
        properties: options.configurations['mapred-site']


      @call
        header: 'SSl Server'
        if: options.post_component
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/ssl-server.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'ssl-server'
              cluster_name: options.cluster_name
              properties: props
            @next callback

      @call
        header: 'SSl Client'
        if: options.post_component
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/ssl-client.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'ssl-client'
              cluster_name: options.cluster_name
              properties: props
            @next callback
      
      @ambari.configs.update
        header: 'config update'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ganglia-env'
        cluster_name: options.cluster_name
        properties:
          gmond_user: 'ganglia'
          gmetad_user: 'ganglia'
      
## Ambari Config Groups
          
      @call
        if: options.post_component
      , ->
        @each options.config_groups, (opts, cb) ->
          {key, value} = opts
          @ambari.configs.groups_add
            header: "#{key}"
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            group_name: key
            tag: key
            description: "#{key} config groups"
            hosts: value.hosts
            desired_configs: 
              type: value.type
              tag: value.tag
              properties: value.properties
          @next cb

## Upload Ranger Related Properties

      @ambari.configs.update
        header: 'Upload ranger-yarn-plugin-properties'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-yarn-plugin-properties'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-yarn-plugin-properties']

      @ambari.configs.update
        header: 'Upload ranger-yarn-security'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-yarn-security'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-yarn-security']

      @ambari.configs.update
        header: 'Upload ranger-yarn-policymgr-ssl'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-yarn-policymgr-ssl'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-yarn-policymgr-ssl']

      @ambari.configs.update
        header: 'Upload ranger-yarn-audit'
        if : options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-yarn-audit'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-yarn-audit']



## Add Component
add RESOURCEMANAGER, NODEMANAGER, APP_TIMELINE_SERVER, YARN_CLIENT
component to cluster but NOT in `INSTALLED` desired state.

### Wait YARN Service

      @ambari.services.wait
        header: 'YARN Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'YARN'

      @ambari.services.component_add
        header: 'RESOURCEMANAGER'
        if: options.post_component
        username: 'admin'
        url: options.ambari_url
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'RESOURCEMANAGER'
        service_name: 'YARN'

      @ambari.services.component_add
        header: 'NODEMANAGER'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'NODEMANAGER'
        service_name: 'YARN'
        
      @ambari.services.component_add
        header: 'APP_TIMELINE_SERVER'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'APP_TIMELINE_SERVER'
        service_name: 'YARN'
        
      @ambari.services.component_add
        header: 'YARN_CLIENT' 
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'YARN_CLIENT'
        service_name: 'YARN'

### RESOURCEMANAGER COMPONENT
      
      for host in options.rm_hosts
        @ambari.hosts.component_add
          header: 'RESOURCEMANAGER ADD'
          if: options.post_component
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'RESOURCEMANAGER'
          hostname: host

### NODEMANAGER COMPONENT
      
      for host in options.nm_hosts
        @ambari.hosts.component_add
          header: 'NODEMANAGER ADD'
          if: options.post_component
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'NODEMANAGER'
          hostname: host

### APP_TIMELINE_SERVER COMPONENT
      
      
      for host in options.ts_hosts
        @ambari.hosts.component_add
          header: 'APP_TIMELINE_SERVER ADD'
          if: options.post_component
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'APP_TIMELINE_SERVER'
          hostname: host

### YARN_CLIENT COMPONENT
      
      for host in options.client_hosts
        @ambari.hosts.component_add
          header: 'YARN_CLIENT ADD'
          if: options.post_component
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'YARN_CLIENT'
          hostname: host


## Dependencies

    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
