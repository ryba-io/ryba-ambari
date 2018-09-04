
# Ambari Takeover

    module.exports = header: 'HADOOP Takeover', handler: ({options}) ->

## Register

      @registry.register ['ambari','cluster','node_add'], 'ryba-ambari-actions/lib/cluster/node_add'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_update'], "ryba-ambari-actions/lib/hosts/component_update"


## Kerberos
Create HDFS Headless keytab.

      @call ->
        add_entry_cmd = ''
        for enc in ['aes256-cts-hmac-sha1-96','aes128-cts-hmac-sha1-96','des3-cbc-sha1','arcfour-hmac']
          add_entry_cmd+= "add_entry -password -p #{options.hdfs.krb5_user.principal} -k 1 -e #{enc}\n#{options.hdfs.krb5_user.password}\n"
        add_entry_cmd += "wkt #{options.hdfs.krb5_user.keytab}\n "

        @krb5.addprinc options.krb5.admin,
          header: 'hdfs principal'
          principal: options.hdfs.krb5_user.principal
          password: options.hdfs.krb5_user.password
        @system.execute
          header: 'hdfs headless keytab'
          cmd: """
            echo '#{add_entry_cmd}' | ktutil
          """
          unless_exec: "/usr/bin/kinit -kt #{options.hdfs.krb5_user.keytab} #{options.hdfs.krb5_user.principal}"
        @system.chmod
          target: options.hdfs.krb5_user.keytab
          mode: 0o0644
          uid: options.hdfs.user.name
          gid: options.hdfs.group.name
      
## Layout
the pid dire name is composed from the hadoop_pid_dir_prefix and hdfs user name.
as a consequence ambari does not detect any pid files and marks the services as stopped.
We have the problem because we do not use the hadoop-env provided by ambari but, ryba's one.
To fix we need to create a symbolic link from ambari's target and ryba's target

        # @system.mkdir
        #   header: 'create pid dir'
        #   target: "#{options.configurations['hadoop-env']['hadoop_pid_dir_prefix']}/#{options.configurations['hadoop-env']['hdfs_user']}"
        #   uid: options.hdfs.user.name
        #   gid: options.hdfs.group.name
        # @system.link
        #   header: 'link pid dir'
        #   source: "#{options.configurations['hadoop-env']['HADOOP_PID_DIR']}"
        #   target: "#{options.configurations['hadoop-env']['hadoop_pid_dir_prefix']}/#{options.configurations['hadoop-env']['hdfs_user']}" 

## Add Components and Hosts
Steps to add components are:
 - Adding Component Service on cluster level
 - Wait for Service to be available
 - Add Service Component on cluster level
 - Add Specific Component on host
 - Update State on component level

### HDFS Service

      @call
        if: options.post_component
      , ->
        @ambari.services.add
          header: 'HDFS Service'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          name: 'HDFS'

        @ambari.services.wait
          header: 'HDFS Service WAITED'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          name: 'HDFS'

        @ambari.services.component_add
          header: 'NAMENODE'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'NAMENODE'
          service_name: 'HDFS'

        @ambari.services.component_add
          header: 'ZKFC'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'ZKFC'
          service_name: 'HDFS'

        @ambari.services.component_add
          header: 'HDFS_CLIENT'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HDFS_CLIENT'
          service_name: 'HDFS'

        @ambari.services.component_add
          header: 'DATANODE'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'DATANODE'
          service_name: 'HDFS'

        @ambari.services.component_add
          header: 'JOURNALNODE'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'JOURNALNODE'
          service_name: 'HDFS'
          
### MAPREDUCE2 Service

      @call
        if: options.post_component
      , ->
        @ambari.services.add
          header: 'MAPREDUCE2 Service'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          name: 'MAPREDUCE2'

        @ambari.services.wait
          header: 'MAPREDUCE2 Service WAITED'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          name: 'MAPREDUCE2'

        @ambari.services.component_add
          header: 'HISTORYSERVER'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HISTORYSERVER'
          service_name: 'MAPREDUCE2'
      
### YARN Service

      @call
        if: options.post_component
      , ->
        @ambari.services.add
          header: 'YARN Service'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          name: 'YARN'

        @ambari.services.wait
          header: 'YARN Service WAITED'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          name: 'YARN'

        @ambari.services.component_add
          header: 'YARN'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'RESOURCEMANAGER'
          service_name: 'YARN'
          
        @ambari.services.component_add
          header: 'YARN'
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'NODEMANAGER'
          service_name: 'YARN'
          


### DATANODE COMPONENT

        for host in options.hdfs_dn_hosts
          @ambari.hosts.component_add
            header: 'DATANODE ADD'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'DATANODE'
            hostname: host

          @ambari.hosts.component_update
            header: 'DATANODE UPDATE'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'DATANODE'
            hostname: host
            properties: 'HostRoles': state: 'INSTALLED'

### JOURNALNODE COMPONENT

        for host in options.hdfs_jn_hosts
          @ambari.hosts.component_add
            header: 'JOURNALNODE ADD'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'JOURNALNODE'
            hostname: host

          @ambari.hosts.component_update
            header: 'JOURNALNODE UPDATE'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'JOURNALNODE'
            hostname: host
            properties: 'HostRoles': state: 'INSTALLED'

### NAMENODE COMPONENT

        for host in options.hdfs_nn_hosts
          @ambari.hosts.component_add
            header: 'NAMENODE ADD'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'NAMENODE'
            hostname: host

          @ambari.hosts.component_update
            header: 'NAMENODE UPDATE'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'NAMENODE'
            hostname: host
            properties: 'HostRoles': state: 'INSTALLED'

### ZKFC

        for host in options.hdfs_nn_hosts
          @ambari.hosts.component_add
            header: 'ZKFC ADD'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'ZKFC'
            hostname: host

          @ambari.hosts.component_update
            header: 'ZKFC UPDATE'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'ZKFC'
            hostname: host
            properties: 'HostRoles': state: 'INSTALLED'


### HDFS_CLIENT COMPONENT

        # for host in options.hdfs_client_hosts
        #   @ambari.hosts.component_add
        #     header: 'HDFS_CLIENT ADD'
        #     url: options.ambari_url
        #     username: 'admin'
        #     password: options.ambari_admin_password
        #     cluster_name: options.cluster_name
        #     component_name: 'HDFS_CLIENT'
        #     hostname: host
        # 
        #   @ambari.hosts.component_update
        #     header: 'HDFS_CLIENT UPDATE'
        #     url: options.ambari_url
        #     username: 'admin'
        #     password: options.ambari_admin_password
        #     cluster_name: options.cluster_name
        #     component_name: 'HDFS_CLIENT'
        #     hostname: host
        #     properties: 'HostRoles': state: 'INSTALLED'

## HISTORYSERVER

        for host in options.mapred_jhs_hosts
          @ambari.hosts.component_add
            header: 'HISTORYSERVER ADD'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'HISTORYSERVER'
            hostname: host

          @ambari.hosts.component_update
            header: 'HISTORYSERVER UPDATE'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'HISTORYSERVER'
            hostname: host
            properties: 'HostRoles': state: 'INSTALLED'

## YARN

        for host in options.yarn_nm_hosts
          @ambari.hosts.component_add
            header: 'NODEMANAGER ADD'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'NODEMANAGER'
            hostname: host

          @ambari.hosts.component_update
            debug: true
            header: 'NODEMANAGER UPDATE'
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            component_name: 'NODEMANAGER'
            hostname: host
            properties: 'HostRoles': state: 'INSTALLED'


        # for host in options.yarn_rm_hosts
        #   @ambari.hosts.component_add
        #     header: 'RESOURCEMANAGER ADD'
        #     url: options.ambari_url
        #     username: 'admin'
        #     password: options.ambari_admin_password
        #     cluster_name: options.cluster_name
        #     component_name: 'RESOURCEMANAGER'
        #     hostname: 'master02.metal.ryba'
        # 
        #   @ambari.hosts.component_update
        #     debug: true
        #     header: 'RESOURCEMANAGER UPDATE'
        #     url: options.ambari_url
        #     username: 'admin'
        #     password: options.ambari_admin_password
        #     cluster_name: options.cluster_name
        #     component_name: 'RESOURCEMANAGER'
        #     hostname: 'master02.metal.ryba'
        #     properties: 'HostRoles': state: 'INSTALLED'


## Dependencies

