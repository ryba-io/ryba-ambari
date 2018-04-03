
# Hadoop takeover Configure

This module is separated as Ambari does not support isolated configuration. The modules
reads configuration from all the hadoop components and merge them in order to provide
hadoop main configuration files. As a consequence, the components will have more
properties than needed.

    module.exports = (service) ->
      options = service.options
      options.hdfs ?= {}


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

## Environment

      options.conf_dir ?= '/etc/hadoop/conf'
      # options.hadoop_lib_home ?= '/usr/hdp/current/hadoop-client/lib' # refered by oozie-env.sh, now hardcoded
      options.hdfs.log_dir ?= '/var/log/hadoop/hdfs'
      options.hdfs.pid_dir ?= '/var/run/hadoop'
      options.hdfs.secure_dn_pid_dir ?= '/var/run/hadoop/hdfs' # /$HADOOP_SECURE_DN_USER
      options.hdfs.secure_dn_user ?= options.hdfs.user.name


## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## HDFS kerberos User

      # HDFS Super User
      options.hdfs.krb5_user ?= {}
      options.hdfs.krb5_user.principal ?= "#{options.hdfs.user.name}@#{options.krb5.realm}"
      options.hdfs.krb5_user.keytab ?= '/etc/security/keytabs/hdfs.headless.keytab'
      options.hdfs.smoke_user.principal ?= "#{options.hdfs.user.name}-#{service.deps.ambari_server.options.cluster_name}@#{options.krb5.realm}"
      options.hdfs.smoke_user.keytab ?= '/etc/security/keytabs/hdfs.headless.keytab'
      throw Error "Required Property: hdfs.krb5_user.password" unless options.hdfs.krb5_user.password
      throw Error "Required Property: hdfs.smoke_user.password" unless options.hdfs.smoke_user.password
      options.identities ?= {}
      options.identities['hdfs'] ?= {}
      options.identities['hdfs']['principal'] ?= {}
      options.identities['hdfs']['principal']['configuration'] ?= 'hadoop-env/hdfs_principal_name'
      options.identities['hdfs']['principal']['type'] ?= 'user'
      options.identities['hdfs']['principal']['local_username'] ?= options.hdfs.user.name
      options.identities['hdfs']['principal']['value'] ?='${hadoop-env/hdfs_user}@${realm}'#options.hdfs.krb5_user.principal
      options.identities['hdfs']['name'] ?= 'hdfs'
      options.identities['hdfs']['keytab'] ?= {}
      options.identities['hdfs']['keytab']['owner'] ?= {}
      options.identities['hdfs']['keytab']['owner']['access'] ?= 'r' 
      options.identities['hdfs']['keytab']['owner']['name'] ?= options.hdfs.user.name 
      options.identities['hdfs']['keytab']['group'] ?= {}
      options.identities['hdfs']['keytab']['group']['access'] ?= 'r'
      options.identities['hdfs']['keytab']['group']['name'] ?= options.hadoop_group.name
      options.identities['hdfs']['keytab']['file'] ?= options.hdfs.krb5_user.keytab
      options.identities['hdfs']['keytab']['configuration'] ?= 'hadoop-env/hdfs_user_keytab'


## HDFS Configuration
Merge hdfs_site, yarn_site, core_site configuration from each components.

      options.configurations ?= {}
      options.configurations['core-site'] = {}
      # ambari missing properties
      options.configurations['core-site']['io.compression.codec.lzo.class'] ?= 'com.hadoop.compression.lzo.LzoCodec'
        
      options.configurations['hdfs-site'] = {}
        
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
      options.configurations['hdfs-site']['dfs.namenode.startup.delay.block.deletion.sec'] = '3601'
      # options.configurations['hdfs-site']['dfs.namenode.checkpoint.dir'] ?= "/var/hdfs/checkpoint"
        
      options.configurations['mapred-site'] = {}
        
      options.configurations['yarn-site'] = {}
      
      options.configurations['capacity-scheduler'] ?= {}

      options.configurations['hadoop-metrics-properties'] = {}

      options.configurations['ssl-server'] = {}
        
## HADOOP, YARN Env
Inhertis Env properties from hadoop_core components. For Components `HADOOP_DATANODE_OPTS`,
`HADOOP_NAMENODE_OPTS`,  `HADOOP_JOURNALNODE_OPTS` properties will be rendered at
as install time, like in any other components.
For this reason components system opts are regiester.

      options.hdfs_nn_opts ?= {}
      options.hdfs_dn_opts ?= {}
      options.hdfs_jn_opts ?= {}
      options.zkfc_opts ?= {}
      #opts
      options.configurations['hadoop-env'] ?= {}
      options.configurations['hadoop-env']['HADOOP_LOG_DIR'] ?= options.hdfs.log_dir
      options.configurations['hadoop-env']['HADOOP_PID_DIR'] ?= "#{options.hdfs.pid_dir}/#{options.hdfs.user.name}"
      options.configurations['hadoop-env']['HADOOP_SECURE_DN_PID_DIR'] ?= "#{options.hdfs.pid_dir}/#{options.hdfs.user.name}"
      options.configurations['hadoop-env']['proxyuser_group'] ?= 'users'
      options.configurations['hadoop-env']['hdfs_user'] ?= options.hdfs.user.name
      options.configurations['hadoop-env']['hdfs_principal_name'] ?= options.hdfs.krb5_user.principal
      options.configurations['hadoop-env']['hdfs_user_keytab'] ?= options.hdfs.krb5_user.keytab
      options.configurations['hadoop-env']['hdfs_user_nofile_limit'] ?= options.hdfs.user.limits.nofile
      options.configurations['hadoop-env']['hdfs_user_nproc_limit'] ?= options.hdfs.user.limits.nproc
      # Ambari required
      options.configurations['hadoop-env']['hadoop_pid_dir_prefix'] ?= options.hdfs.pid_dir
      options.configurations['hadoop-env']['hdfs_log_dir_prefix'] ?= options.hdfs.log_dir
      options.configurations['hadoop-env']['hdfs_tmp_dir'] ?= "#{options.hdfs.log_dir}/tmp"
      options.configurations['hadoop-env']['hadoop_conf_dir'] ?= "#{options.conf_dir}"

## Hosts

      # Hdfs Hosts
      options.nn_hosts ?= []
      options.jn_hosts ?= []
      options.dn_hosts ?= []
      options.zkfc_hosts ?= []
      options.client_hosts ?= []

## SSL

      # options.configurations['ssl-client'] ?= service.deps.hadoop_core[0].options.ssl_client
      # options.configurations['ssl-server'] ?= service.deps.hadoop_core[0].options.ssl_server
      
      options.ssl = merge {}, service.deps.ssl?.options, options.ssl
      options.ssl.enabled ?= !!service.deps.ssl
      if options.ssl.enabled
        options.ssl_client ?= {}
        options.ssl_server ?= {}
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert

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

## Ambari REST API

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.stack_name = service.deps.ambari_server.options.stack_name
      options.stack_version = service.deps.ambari_server.options.stack_version

## Ambari Agent
Register users to ambari agent's user list.

      for srv in service.deps.ambari_agent
        srv.options.users ?= {}
        srv.options.users['hdfs'] ?= options.hdfs.user
        srv.options.groups ?= {}
        srv.options.groups['hdfs'] ?= options.hdfs.group
        srv.options.groups['hadoop_group'] ?= options.hadoop_group

## Ambari Config Groups
`config_groups` contains final object that install will submit to ambari.
`groups` is the array of config_groups name to which the host belongs to.

      options.config_groups ?= {}
      options.groups ?= []
      for srv in service.deps.hdfs
        for name in options.groups
          srv.options.config_groups ?= {}
          srv.options.config_groups[name] ?= {}
          srv.options.config_groups[name]['hosts'] ?= []
          srv.options.config_groups[name]['hosts'].push service.node.fqdn unless srv.options.config_groups[name]['hosts'].indexOf(service.node.fqdn) > -1
          
## Dependencies

    {merge} = require 'nikita/lib/misc'
