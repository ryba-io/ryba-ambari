
# Hadoop takeover Configure

This module is separated as Ambari does not support isolated configuration. The modules
reads configuration from all the hadoop components and merge them in order to provide
hadoop main configuration files. As a consequence, the components will have more
properties than needed.

    module.exports = (service) ->
      options = service.options

## Identities

      # Group for hadoop
      options.hadoop_group = name: options.hadoop_group if typeof options.hadoop_group is 'string'
      options.hadoop_group ?= {}
      options.hadoop_group.name ?= 'hadoop'
      options.hadoop_group.system ?= true
      options.hadoop_group.comment ?= 'Hadoop Group'
      # Groups
      options.hdfs.group ?= {}
      options.hdfs.group = name: options.hdfs.group if typeof options.hdfs.group is 'string'
      options.hdfs.group.name ?= 'hdfs'
      options.hdfs.group.system ?= true
      options.yarn.group ?= {}
      options.yarn.group = name: options.yarn.group if typeof options.yarn.group is 'string'
      options.yarn.group.name ?= 'yarn'
      options.yarn.group.system ?= true
      options.mapred.group ?= {}
      options.mapred.group = name: options.mapred.group if typeof options.mapred.group is 'string'
      options.mapred.group.name ?= 'mapred'
      options.mapred.group.system ?= true
      # Unix user hdfs
      options.hdfs.user ?= {}
      options.hdfs.user = name: options.hdfs.user if typeof options.hdfs.user is 'string'
      options.hdfs.user.name ?= 'hdfs'
      options.hdfs.user.system ?= true
      options.hdfs.user.gid = options.hdfs.group.name
      options.hdfs.user.groups ?= 'hadoop'
      options.hdfs.user.comment ?= 'Hadoop HDFS User'
      options.hdfs.user.home ?= '/var/lib/hadoop-hdfs'
      options.hdfs.user.limits ?= {}
      options.hdfs.user.limits.nofile ?= 64000
      options.hdfs.user.limits.nproc ?= true
      # Unix user for yarn
      options.yarn.user ?= {}
      options.yarn.user = name: options.yarn.user if typeof options.yarn.user is 'string'
      options.yarn.user.name ?= 'yarn'
      options.yarn.user.system ?= true
      options.yarn.user.gid = options.yarn.group.name
      options.yarn.user.groups ?= 'hadoop'
      options.yarn.user.comment ?= 'Hadoop YARN User'
      options.yarn.user.home ?= '/var/lib/hadoop-yarn'
      options.yarn.user.limits ?= {}
      options.yarn.user.limits.nofile ?= 64000
      options.yarn.user.limits.nproc ?= true
      # Unix user for mapred
      options.mapred.user ?= {}
      options.mapred.user = name: options.mapred.user if typeof options.mapred.user is 'string'
      options.mapred.user.name ?= 'mapred'
      options.mapred.user.system ?= true
      options.mapred.user.gid = options.mapred.group.name
      options.mapred.user.groups ?= 'hadoop'
      options.mapred.user.comment ?= 'Hadoop MapReduce User'
      options.mapred.user.home ?= '/var/lib/hadoop-mapreduce'
      options.mapred.user.limits ?= {}
      options.mapred.user.limits.nofile ?= 64000
      options.mapred.user.limits.nproc ?= true

## Environment

      options.conf_dir ?= '/etc/hadoop/conf'
      # options.hadoop_lib_home ?= '/usr/hdp/current/hadoop-client/lib' # refered by oozie-env.sh, now hardcoded
      options.hdfs.log_dir ?= '/var/log/hadoop'
      options.hdfs.pid_dir ?= '/var/run/hadoop'
      options.hdfs.secure_dn_pid_dir ?= '/var/run/hadoop/hdfs' # /$HADOOP_SECURE_DN_USER
      options.hdfs.secure_dn_user ?= options.hdfs.user.name

## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name

      options.configurations ?= {}

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## HDFS kerberos User

      options.hdfs = merge {}, service.deps.hadoop_core[0].options.hdfs, options.hdfs
      # HDFS Super User
      options.hdfs.krb5_user ?= {}
      options.hdfs.krb5_user.principal ?= "#{options.hdfs.user.name}@#{options.krb5.realm}"
      options.hdfs.krb5_user.keytab ?= '/etc/security/keytabs/hdfs.headless.keytab'
      throw Error "Required Property: hdfs.krb5_user.password" unless options.hdfs.krb5_user.password


## HDFS, YARN, Mapred site
Merge hdfs_site, yarn_site, core_site configuration from each components.

      options.configurations['core-site'] = {}merge {}, options.configurations['core-site'],
        service.deps.hdfs_nn[0].options.core_site, service.deps.hdfs_jn[0].options.core_site,
        service.deps.hdfs_dn[0].options.core_site, service.deps.hadoop_core[0].options.core_site,
        service.deps.yarn_nm[0].options.core_site, service.deps.yarn_rm[0].options.core_site,
        service.deps.yarn_ts[0].options.core_site, service.deps.yarn_client[0].options.core_site,
        service.deps.mapred_jhs[0].options.core_site, service.deps.mapred_client[0].options.core_site,
        service.deps.zkfc[0].options.core_site
      # ambari missing properties
      options.configurations['core-site']['io.compression.codec.lzo.class'] ?= 'com.hadoop.compression.lzo.LzoCodec'
        
      options.configurations['hdfs-site'] = merge {}, options.configurations['hdfs-site'],
        service.deps.hdfs_nn[0].options.hdfs_site, service.deps.hdfs_jn[0].options.hdfs_site,
        service.deps.hdfs_dn[0].options.hdfs_site, service.deps.hadoop_core[0].options.hdfs_site,
        service.deps.yarn_nm[0].options.hdfs_site, service.deps.yarn_rm[0].options.hdfs_site,
        service.deps.yarn_ts[0].options.hdfs_site, service.deps.yarn_client[0].options.hdfs_site,
        service.deps.mapred_jhs[0].options.hdfs_site, service.deps.mapred_client[0].options.hdfs_site,
        service.deps.zkfc[0].options.hdfs_site
        
      # ambari missing properties
      options.configurations['hdfs-site']['dfs.hosts.exclude'] = '/etc/hadoop/conf/dfs.exclude'
      options.configurations['hdfs-site']['dfs.hosts.include'] = '/etc/hadoop/conf/dfs.include'
      options.configurations['hdfs-site']['dfs.hosts'] = '/etc/hadoop/conf/dfs.include'
      options.configurations['hdfs-site']['dfs.client.retry.policy.enabled'] ?= "true"
      options.configurations['hdfs-site']['dfs.content-summary.limit'] ?= "5000"
      options.configurations['hdfs-site']['dfs.encrypt.data.transfer.cipher.suites'] ?= "AES/CTR/NoPadding"
      options.configurations['hdfs-site']['dfs.https.port'] ?= "50470"
      options.configurations['hdfs-site']['dfs.namenode.accesstime.precision'] ?= "0"
      options.configurations['hdfs-site']['dfs.namenode.audit.log.async'] ?= "true"
      options.configurations['hdfs-site']['dfs.namenode.fslock.fair'] ?= "false"
      options.configurations['hdfs-site']['nfs.exports.allowed.hosts'] ?= "* rw"
      options.configurations['hdfs-site']['nfs.file.dump.dir'] ?= "/tmp/.hdfs-nfs"
      options.configurations['hdfs-site']['dfs.namenode.checkpoint.dir'] ?= "/var/hdfs/nn"
      # options.configurations['hdfs-site']['dfs.namenode.checkpoint.dir'] ?= "/var/hdfs/checkpoint"
        
      options.configurations['mapred-site'] = merge {}, options.configurations['mapred-site'],
        service.deps.hdfs_nn[0].options.mapred_site, service.deps.hdfs_jn[0].options.mapred_site,
        service.deps.hdfs_dn[0].options.mapred_site, service.deps.hadoop_core[0].options.mapred_site,
        service.deps.yarn_nm[0].options.mapred_site, service.deps.yarn_rm[0].options.mapred_site,
        service.deps.yarn_ts[0].options.mapred_site, service.deps.yarn_client[0].options.mapred_site,
        service.deps.mapred_jhs[0].options.mapred_site, service.deps.mapred_client[0].options.mapred_site
        
      options.configurations['yarn-site'] = merge {}, options.configurations['yarn-site'],
        service.deps.hdfs_nn[0].options.yarn_site, service.deps.hdfs_jn[0].options.yarn_site,
        service.deps.hdfs_dn[0].options.yarn_site, service.deps.hadoop_core[0].options.yarn_site,
        service.deps.yarn_nm[0].options.yarn_site, service.deps.yarn_rm[0].options.yarn_site,
        service.deps.yarn_ts[0].options.yarn_site, service.deps.yarn_client[0].options.yarn_site,
        service.deps.mapred_jhs[0].options.yarn_site, service.deps.mapred_client[0].options.yarn_site
      
      options.configurations['capacity-scheduler'] ?= merge {}, options.configurations['capacity-scheduler'], service.deps.yarn_rm[0].options.capacity_scheduler

      options.configurations['hadoop-metrics-properties'] = merge {}, options.configurations['hadoop-metrics-properties'],
        service.deps.hdfs_nn[0].options.metrics.config, service.deps.hdfs_jn[0].options.metrics.config,
        service.deps.hdfs_dn[0].options.metrics.config, service.deps.hadoop_core[0].options.metrics.config,
        service.deps.yarn_nm[0].options.metrics.config, service.deps.yarn_rm[0].options.metrics.config,
        service.deps.mapred_jhs[0].options.metrics.config

      options.configurations['ssl-server'] = merge {}, service.deps.hadoop_core[0].options.ssl_server, options.configurations['ssl-server']
        
## HADOOP, YARN Env
Inhertis Env properties from hadoop_core components. For Components `HADOOP_DATANODE_OPTS`,
`HADOOP_NAMENODE_OPTS`,  `HADOOP_JOURNALNODE_OPTS` properties will be rendered at
as install time, like in any other components.
For this reason components system opts are regiester.

      options.hdfs_nn_opts ?= service.deps.hdfs_nn[0].options.opts
      options.hdfs_dn_opts ?= service.deps.hdfs_dn[0].options.opts
      options.hdfs_jn_opts ?= service.deps.hdfs_jn[0].options.opts
      #opts
      options.configurations['hadoop-env'] ?= 
        HADOOP_ROOT_LOGGER: service.deps.hadoop_core[0].options.log4j.root_logger
        HADOOP_SECURITY_LOGGER: service.deps.hadoop_core[0].options.log4j.security_logger
        HDFS_AUDIT_LOGGER: service.deps.hadoop_core[0].options.log4j.audit_logger
        HADOOP_HEAPSIZE: service.deps.hadoop_core[0].options.hadoop_heap
        HADOOP_LOG_DIR: service.deps.hadoop_core[0].options.hdfs.log_dir
        HADOOP_PID_DIR: "#{service.deps.hadoop_core[0].options.hdfs.pid_dir}/#{service.deps.hadoop_core[0].options.hdfs.user.name}"
        HADOOP_CLIENT_OPTS: service.deps.hadoop_core[0].options.hadoop_client_opts
        HADOOP_OPTS: service.deps.hadoop_core[0].options.hadoop_opts
        HADOOP_SECURE_DN_USER: service.deps.hdfs_dn[0].options.user.name
        HADOOP_SECURE_DN_LOG_DIR: service.deps.hdfs_dn[0].options.log_dir
        HADOOP_SECURE_DN_PID_DIR: "#{service.deps.hadoop_core[0].options.hdfs.pid_dir}/#{service.deps.hadoop_core[0].options.hdfs.user.name}"
        java_home: service.deps.hdfs_nn[0].options.java_home
        HADOOP_NAMENODE_INIT_HEAPSIZE: service.deps.hdfs_nn[0].options.hadoop_namenode_init_heap
        namenode_heapsize: service.deps.hdfs_nn[0].options.heapsize
        namenode_newsize: service.deps.hdfs_nn[0].options.newsize
        # Ambari required
        hadoop_pid_dir_prefix: service.deps.hadoop_core[0].options.hdfs.pid_dir
        hdfs_log_dir_prefix: service.deps.hadoop_core[0].options.hdfs.log_dir
        hadoop_root_logger: service.deps.hadoop_core[0].options.log4j.root_logger
        hadoop_heapsize: service.deps.hadoop_core[0].options.hadoop_heap
        namenode_heapsize: service.deps.hdfs_nn[0].options.heapsize
        namenode_opt_newsize: service.deps.hdfs_nn[0].options.newsize
        namenode_opt_maxnewsize: service.deps.hdfs_nn[0].options.newsize
        hdfs_user: service.deps.hadoop_core[0].options.hdfs.user.name
        hdfs_user_keytab: options.hdfs.krb5_user.keytab
        hdfs_user_nofile_limit: 128000
        hdfs_user_nproc_limit: 65536
        hdfs_principal_name: options.hdfs.krb5_user.principal
        hdfs_tmp_dir: "#{service.deps.hadoop_core[0].options.hdfs.log_dir}/tmp"
        hadoop_root_logger: service.deps.hadoop_core[0].options.log4j.root_logger
        

      options.yarn_rm_opts ?= service.deps.yarn_rm[0].options.opts
      options.yarn_nm_opts ?= service.deps.yarn_nm[0].options.opts
      options.yarn_ts_opts ?= service.deps.yarn_ts[0].options.opts

      options.configurations['yarn-env'] ?=
        JAVA_HOME: service.deps.yarn_rm[0].options.java_home
        HADOOP_YARN_HOME: '{{hadoop_yarn_home}}'
        YARN_LOG_DIR: '{{yarn_log_dir_prefix}}/$USER'
        YARN_PID_DIR: '{{yarn_pid_dir_prefix}}'
        YARN_HEAPSIZE: options.heapsize
        YARN_RESOURCEMANAGER_HEAPSIZE: service.deps.yarn_rm[0].heapsize
        YARN_ROOT_LOGGER: service.deps.hadoop_core[0].options.log4j.root_logger
        HADOOP_LIBEXEC_DIR: service.deps.yarn_nm[0].options.libexec
        YARN_HEAPSIZE: '1024m'
        YARN_NODEMANAGER_HEAPSIZE: service.deps.yarn_nm[0].heapsize
        YARN_OPTS: service.deps.hadoop_core[0].options.java_opts
        yarn_user: service.deps.hadoop_core[0].options.yarn.user.name
        yarn_tmp_dir: "#{service.deps.hadoop_core[0].options.yarn.log_dir}/tmp"
        yarn_user_nofile_limit: 128000
        yarn_user_nproc_limit: 65536
        yarn_heapsize: '1024m'
        nodemanager_heapsize: service.deps.yarn_nm[0].options.heapsize
        resourcemanager_heapsize: service.deps.yarn_rm[0].options.heapsize
        apptimelineserver_heapsize: service.deps.yarn_ts[0].options.heapsize
        hadoop_yarn_home: service.deps.hadoop_core[0].options.yarn.user.home
        yarn_log_dir_prefix: service.deps.hadoop_core[0].options.yarn.log_dir
        hadoop_libexec_dir: service.deps.yarn_nm[0].options.libexec
        yarn_pid_dir_prefix: service.deps.hadoop_core[0].options.yarn.pid_dir

## SSL

      options.configurations['ssl-client'] ?= service.deps.hadoop_core[0].options.ssl_client
      options.configurations['ssl-server'] ?= service.deps.hadoop_core[0].options.ssl_server

## Hadoop Policy

      options.hadoop_policy ?= 
        "security.admin.operations.protocol.acl" : "hadoop"
        "security.client.datanode.protocol.acl" : "*"
        "security.client.protocol.acl" : "*"
        "security.datanode.protocol.acl" : "*"
        "security.inter.datanode.protocol.acl" : "*"
        "security.inter.tracker.protocol.acl" : "*"
        "security.job.client.protocol.acl" : "*"
        "security.job.task.protocol.acl" : "*"
        "security.namenode.protocol.acl" : "*"
        "security.refresh.policy.protocol.acl" : "hadoop"
        "security.refresh.usertogroups.mappings.protocol.acl" : "hadoop"

## Log4j

      options.hdfs_log4j = merge {}, service.deps.hdfs_nn[0].options.log4j.properties,
        service.deps.hdfs_dn[0].options.log4j.properties,
        service.deps.hadoop_core[0].options.log4j.properties

      options.yarn_log4j = merge {}, service.deps.yarn_rm[0].options.log4j.properties,
        service.deps.hadoop_core[0].options.log4j.properties

## Components Mapping to host
      
      options.hdfs_nn_hosts ?= service.deps.hdfs_nn.map( (node) -> node.node.fqdn )
      options.hdfs_dn_hosts ?= service.deps.hdfs_dn.map( (node) -> node.node.fqdn )
      options.hdfs_jn_hosts ?= service.deps.hdfs_jn.map( (node) -> node.node.fqdn )
      options.yarn_rm_hosts ?= service.deps.yarn_rm.map( (node) -> node.node.fqdn )
      options.yarn_nm_hosts ?= service.deps.yarn_nm.map( (node) -> node.node.fqdn )
      options.mapred_jhs_hosts ?= service.deps.mapred_jhs.map( (node) -> node.node.fqdn )
      # options.hdfs_jn_hosts ?= service.deps.hdfs_jn[0].instances.map( (node) -> node.fqdn )
      # options.yarn_rm_hosts ?= service.deps.yarn_rm[0].instances.map( (node) -> node.fqdn )
      # options.yarn_nm_hosts ?= service.deps.yarn_nm[0].instances.map( (node) -> node.fqdn )
      # options.yarn_ts_hosts ?= service.deps.yarn_ts[0].instances.map( (node) -> node.fqdn )
      # options.mapred_jhs_hosts ?= service.deps.mapred_jhs_hosts[0].instances.map( (node) -> node.fqdn )
      # options.hdfs_client_hosts ?= service.deps.hdfs_client[0].instances.map( (node) -> node.fqdn )
      # options.yarn_client_hosts ?= service.deps.yarn_client[0].instances.map( (node) -> node.fqdn )
      # options.mapred_client_hosts ?= service.deps.mapred_client[0].instances.map( (node) -> node.fqdn )

## metrics

        # register datanode properties
        # options.config_group.desired_configs ?= []
        # hdfs_site_discovered = false
        # hdfs_env_discovered = false
        # for config in options.config_group.desired_configs
        #   if config.type is 'hdfs-site'
        #     hdfs_site_discovered = true
        #     config.properties = merge {}, config.properties, options.hdfs_site
        #   if config.type is 'hdfs-site'
        #     hdfs_site_discovered = true
        #     config.properties = merge {}, config.properties, options.hdfs_site
        # options.config_groups ?=
        #   tag: "worker_config_group"
        #   cluster_name: options.cluster_name
        #   group_name: 'worker_config_group'
        #   desired_configs: 
        # options.config_groups
        #   desired_configs =
        # type: 'zoo.cfg'
        # tag: 'slow_zookeeper'
        # properties: 'tickTime': '5000'
## Dependencies

    {merge} = require 'nikita/lib/misc'
