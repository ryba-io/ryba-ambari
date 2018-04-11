
# HADOOP YARN NodeManager Install

    module.exports = header: 'YARN NM Ambari Install', handler: (options) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"

## Wait

      @call 'masson/core/krb5_client/wait', once: true, options.wait_krb5_client
      @call 'ryba-ambari-takeover/hadoop/hdfs_nn/wait', once: true, options.wait_hdfs_nn, conf_dir: options.hadoop_conf_dir

## IPTables

| Service    | Port | Proto  | Parameter                          |
|------------|------|--------|------------------------------------|
| nodemanager | 45454 | tcp  | yarn.nodemanager.address           | x
| nodemanager | 8040  | tcp  | yarn.nodemanager.localizer.address |
| nodemanager | 8042  | tcp  | yarn.nodemanager.webapp.address    |
| nodemanager | 8044  | tcp  | yarn.nodemanager.webapp.https.address    |

IPTables rules are only inserted if the parameter "iptables.action" is set to 
"start" (default value).

      nm_port = options.yarn_site['yarn.nodemanager.address'].split(':')[1]
      nm_localizer_port = options.yarn_site['yarn.nodemanager.localizer.address'].split(':')[1]
      nm_webapp_port = options.yarn_site['yarn.nodemanager.webapp.address'].split(':')[1]
      nm_webapp_https_port = options.yarn_site['yarn.nodemanager.webapp.https.address'].split(':')[1]
      options.iptables_rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: nm_port, protocol: 'tcp', state: 'NEW', comment: "YARN NM Container" }
      options.iptables_rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: nm_localizer_port, protocol: 'tcp', state: 'NEW', comment: "YARN NM Localizer" }
      options.iptables_rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: nm_webapp_port, protocol: 'tcp', state: 'NEW', comment: "YARN NM Web UI" }
      options.iptables_rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: nm_webapp_https_port, protocol: 'tcp', state: 'NEW', comment: "YARN NM Web Secured UI" }
      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: options.iptables_rules

## Packages

Install the "hadoop-yarn-nodemanager" service, symlink the rc.d startup script
inside "/etc/init.d" and activate it on startup.

      @call header: 'Packages', ->
        @service
          name: 'hadoop-yarn-nodemanager'
        @hdp_select
          # name: 'hadoop-yarn-client' # Not checked
          name: 'hadoop-yarn-nodemanager'
        @service # Seems like NM complain with message "java.lang.ClassNotFoundException: Class org.apache.hadoop.mapred.ShuffleHandler not found"
          name: 'hadoop-mapreduce'
        @hdp_select
          name: 'hadoop-client'
        @system.tmpfs
          if_os: name: ['redhat','centos'], version: '7'
          mount: "#{options.pid_dir}"
          uid: options.user.name
          gid: options.hadoop_group.name
          perm: '0755'
        @call
          if: options.yarn_site['spark.shuffle.service.enabled'] is 'true'
          header: 'Spark Worker Shuffle Package'
        , ->
          @service
            name: 'spark_*-yarn-shuffle'
          @system.execute
            cmd: """
            file_lib=`ls /usr/hdp/current/spark-client/lib/* | grep yarn-shuffle.jar`
            file_aux=`ls /usr/hdp/current/spark-client/aux/* | grep yarn-shuffle.jar`
            file=''
            if [ -f "$file_lib" ] ; 
              then file=$file_lib ; 
            else if [ -f "$file_aux" ] ; 
              then file=$file_aux ; 
            fi;
            name=`basename $file`
            target="/usr/hdp/current/hadoop-yarn-nodemanager/lib/${name}"
            source=`readlink $target`
            if [ "$source" == "$file" ] ;
              then exit 3 ;
              else
                rm -f $target;
                ln -s $file $target;
                exit 0;
                fi;
            fi;
            """
            code_skipped: 3

      @call header: 'Layout', ->
        @system.mkdir
          target: options.yarn_site['yarn.nodemanager.log-dirs'].split ','
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o0755
          parent: true
        @system.mkdir
          target: options.yarn_site['yarn.nodemanager.local-dirs'].split ','
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o0755
          parent: true
        @system.mkdir
          target: options.yarn_site['yarn.nodemanager.recovery.dir'] 
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o0750
          parent: true

## Capacity Planning

Naive discovery of the memory and CPU allocated by this NodeManager.

It is recommended to use the "capacity" script prior install Hadoop on
your cluster. It will suggest you relevant values for your servers with a
global view of your system. In such case, this middleware is bypassed and has
no effect. Also, this isnt included inside the configuration because it need an
SSH connection to the node to gather the memory and CPU informations.

      @call
        header: 'Capacity Planning'
        unless: options.yarn_site['yarn.nodemanager.resource.memory-mb'] and options.yarn_site['yarn.nodemanager.resource.cpu-vcores']
      , ->
        # diskNumber = options.yarn_site['yarn.nodemanager.local-dirs'].length
        options.yarn_site['yarn.nodemanager.resource.memory-mb'] ?= Math.round @meminfo.MemTotal / 1024 / 1024 * .8
        options.yarn_site['yarn.nodemanager.resource.cpu-vcores'] ?= @cpuinfo.length

## Container Executor

Important: path seems hardcoded to "../etc/hadoop/container-executor.cfg", 
running `/usr/hdp/current/hadoop-yarn-client/bin/container-executor` will print
"Configuration file ../etc/hadoop/container-executor.cfg not found." if missing.

The parent directory must be owned by root or it will print: "Caused by:
ExitCodeException exitCode=24: File /etc/hadoop/conf must be owned by root,
but is owned by 2401"

      @call header: 'Container Executor', ->
        ce_group = options.container_executor['yarn.nodemanager.linux-container-executor.group']
        @system.chown
          target: "#{options.home}/bin/container-executor"
          uid: 'root'
          gid: ce_group
        @system.chmod
          target: "#{options.home}/bin/container-executor"
          mode: 0o6050
        @file.ini
          # target: "#{options.conf_dir}/container-executor.cfg"
          target: "#{options.home}/etc/hadoop/container-executor.cfg"
          content: options.container_executor
          uid: 'root'
          gid: ce_group
          mode: 0o0640
          separator: '='
          backup: true

## Cgroups Configuration

YARN Nodemanager can be configured to mount automatically the cgroup path on start.
If `yarn.nodemanager.linux-container-executor.cgroups.mount` is set to true,
Ryba just mkdirs the path.
Is `yarn.nodemanager.linux-container-executor.cgroups.mount` is set to false,
it creates and persist a cgroup for yarn by registering into the /etc/cgconfig configuration.
Note: For now (December 2016 - HDP 2.5.3.0), yarn does not support `systemctl` cgroups
on Centos/Redhat7 OS. Legacy cgconfig and cgroup-tools package must be used. (masson/core/cgroups)

      @call
        header: 'Cgroups Auto'
        if: -> options.yarn_site['yarn.nodemanager.linux-container-executor.cgroups.mount'] is 'true'
      , ->
        @service
          name: 'libcgroup'
        # .execute
        #   cmd: 'mount -t cgroup -o cpu cpu /cgroup'
        #   code_skipped: 32
        @system.mkdir
          target: "#{options.yarn_site['yarn.nodemanager.linux-container-executor.cgroups.mount-path']}/cpu"
          mode: 0o1777
          parent: true
      @call
        header: 'Cgroups Manual'
        unless: -> options.yarn_site['yarn.nodemanager.linux-container-executor.cgroups.mount'] is 'true'
      , ->
        @system.cgroups
          target: '/etc/cgconfig.d/yarn.cgconfig.conf'
          merge: false
          groups: options.cgroup
        , (err, _, cgroups) ->
          options.yarn_site['yarn.nodemanager.linux-container-executor.cgroups.mount-path'] = cgroups.mount
        @call ->
          # migration: wdavidw 170827, using store is a bad, very bad idea, ensure it works in the mean time
          # lucasbak 180127 not using store anymore
          throw Error 'YARN NM Ambari Cgroup is undefined' unless options.yarn_site['yarn.nodemanager.linux-container-executor.cgroups.mount-path']
          # @hconfigure
          #   header: 'YARN Site'
          #   target: "#{options.conf_dir}/yarn-site.xml"
          #   properties: options.yarn_site
          #   merge: true
          #   backup: true
        @service.restart
          name: 'cgconfig'
          if: -> @status -2

## Kerberos User

Create the Kerberos user to the Node Manager service. By default, it takes the
form of "rm/{fqdn}@{realm}"

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos'
        principal: options.yarn_site['yarn.nodemanager.principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.yarn_site['yarn.nodemanager.keytab']
        uid: options.user.name
        gid: options.hadoop_group.name

## Ulimit

Increase ulimit for the HDFS user. The HDP package create the following
files:

```bash
cat /etc/security/limits.d/yarn.conf
yarn   - nofile 32768
yarn   - nproc  65536
```

Note, a user must re-login for those changes to be taken into account. See
the "ryba-ambari-takeover/hadoop/hdfs" module for additional information.

      @system.limits
        header: 'Ulimit'
        user: options.user.name
      , options.user.limits

### HDFS Layout

Create the YARN log directory defined by the property 
"yarn.nodemanager.remote-app-log-dir". The default value in the HDP companion
files is "/app-logs". The command `hdfs dfs -ls /` should print:

```
drwxrwxrwt   - yarn   hadoop            0 2014-05-26 11:01 /app-logs
```

Layout is inspired by [Hadoop recommandation](http://hadoop.apache.org/docs/r2.1.0-beta/hadoop-project-dist/hadoop-common/ClusterSetup.html)

      # Note, YARN NM must have deployed HDFS Client conf files in order to wait for HDFS NN
      @call 'ryba-ambari-takeover/hadoop/hdfs_nn/wait', once: true, options.wait_hdfs_nn, conf_dir: options.hadoop_conf_dir
      remote_app_log_dir = options.yarn_site['yarn.nodemanager.remote-app-log-dir']
      @system.execute
        header: 'HDFS layout'
        cmd: mkcmd.hdfs options.hdfs_krb5_user, """
        hdfs --config #{options.hadoop_conf_dir} dfs -mkdir -p #{remote_app_log_dir}
        hdfs --config #{options.hadoop_conf_dir} dfs -chown #{options.user.name}:#{options.hadoop_group.name} #{remote_app_log_dir}
        hdfs --config #{options.hadoop_conf_dir} dfs -chmod 1777 #{remote_app_log_dir}
        """
        unless_exec: "[[ hdfs dfs -d #{remote_app_log_dir} ]]"
        code_skipped: 2

### NODEMANAGER component wait
Wait for the NODEMANAGER component to be declared on the host

      @ambari.hosts.component_wait
        header: 'NODEMANAGER WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'NODEMANAGER'
        hostname: options.fqdn

### NODEMANAGER component install
Put the NODEMANAGER component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'NODEMANAGER set installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'NODEMANAGER'
        hostname: options.fqdn

## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
