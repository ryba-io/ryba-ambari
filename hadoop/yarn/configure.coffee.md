
# Hadoop takeover Configure

This module is separated as Ambari does not support isolated configuration. The modules
reads configuration from all the hadoop components and merge them in order to provide
hadoop main configuration files. As a consequence, the components will have more
properties than needed.

    module.exports = (service) ->
      options = service.options
      options.yarn ?= {}
      options.hdfs ?= {}


## Identities

      # Group for hadoop
      options.hadoop_group = name: options.hadoop_group if typeof options.hadoop_group is 'string'
      options.hadoop_group ?= {}
      options.hadoop_group.name ?= 'hadoop'
      options.hadoop_group.system ?= true
      options.hadoop_group.comment ?= 'Hadoop Group'
      # Groups
      options.yarn.group ?= {}
      options.yarn.group = name: options.yarn.group if typeof options.yarn.group is 'string'
      options.yarn.group.name ?= 'yarn'
      options.yarn.group.system ?= true
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
      options.yarn.user.limits.nproc ?= 64000

## Environment

      options.conf_dir ?= '/etc/hadoop/conf'
      # options.hadoop_lib_home ?= '/usr/hdp/current/hadoop-client/lib' # refered by oozie-env.sh, now hardcoded
      options.yarn.log_dir ?= '/var/log/hadoop'
      options.yarn.pid_dir ?= '/var/run/hadoop'
      # options.hdfs.secure_dn_pid_dir ?= '/var/run/hadoop/hdfs' # /$HADOOP_SECURE_DN_USER
      # options.hdfs.secure_dn_user ?= options.hdfs.user.name

## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.takeover = service.deps.ambari_server.options.takeover
      options.baremetal = service.deps.ambari_server.options.baremetal
      options.configurations ?= {}

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## HDFS, YARN, Mapred site
Merge hdfs_site, yarn_site, core_site configuration from each components.
        
      options.configurations['hdfs-site'] = {}
      options.configurations['mapred-site'] = {}
      options.configurations['yarn-site'] = {}
      options.configurations['yarn-site']['yarn.resourcemanager.webapp.address'] ?= 'localhost:8090'
      options.configurations['capacity-scheduler'] ?= {}
      options.configurations['capacity-scheduler']['capacity-scheduler'] ?= 'null'
      options.configurations['hadoop-metrics-properties'] = {}
      options.configurations['ssl-server'] = {}

## YARN Admin ACL

      options.configurations['yarn-site']['yarn.admin.acl'] ?= "#{options.yarn.user.name}, #{if service.deps.hdfs[0]? then (service.deps.hdfs[0].options.hdfs.user.name + ',dr.who') else 'dr.who'}"

## YARN Env
Inhertis Env properties from yarn_core components. For Components `YARN_RESOURCEMANAGER_OPTS`,
`YARN_NODEMANAGER_OPTS`,  `YARN_TIMELINESERVER_OPTS` properties will be rendered at
as install time, like in any other components.

      options.yarn_rm_opts ?= {}
      options.yarn_nm_opts ?= {}
      options.yarn_ts_opts ?= {}
      # 
      options.configurations['yarn-env'] ?= {}
      options.configurations['yarn-env']['JAVA_HOME'] ?= service.deps.java.options.java_home
      options.configurations['yarn-env']['HADOOP_YARN_HOME'] ?= '{{hadoop_yarn_home}}'
      options.configurations['yarn-env']['YARN_LOG_DIR'] ?= '{{yarn_log_dir_prefix}}/{{yarn_user}}'
      options.configurations['yarn-env']['YARN_PID_DIR'] ?= "{{yarn_pid_dir_prefix}}/{{yarn_user}}"
      options.configurations['yarn-env']['YARN_HEAPSIZE'] ?= '1024m'
      # ambari required properties
      options.configurations['yarn-env']['hadoop_yarn_home'] ?= options.yarn.user.home
      options.configurations['yarn-env']['yarn_user'] ?= options.yarn.user.name
      options.configurations['yarn-env']['yarn_tmp_dir'] ?= "#{options.yarn.log_dir}/tmp"
      options.configurations['yarn-env']['yarn_user_nofile_limit'] ?= options.yarn.user.limits.nofile
      options.configurations['yarn-env']['yarn_user_nproc_limit'] ?= options.yarn.user.limits.nproc
      options.configurations['yarn-env']['yarn_heapsize'] ?= options.configurations['yarn-env']['YARN_HEAPSIZE']
      options.configurations['yarn-env']['yarn_log_dir_prefix'] ?= options.yarn.log_dir
      options.configurations['yarn-env']['yarn_pid_dir_prefix'] ?= options.yarn.pid_dir
      options.configurations['yarn-env']['min_user_id'] ?= '1000'

## Hosts

      # YARN Hosts
      options.rm_hosts ?= []
      options.nm_hosts ?= []
      options.ts_hosts ?= []
      options.client_hosts ?= []

## SSL

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

## Ambari Agent
Register users to ambari agent's user list.
Disable group push as ambari by default remove all yarn user groups.

      for srv in service.deps.ambari_agent
        srv.options.users ?= {}
        srv.options.users['yarn'] ?= options.yarn.user
        srv.options.groups ?= {}
        srv.options.groups['yarn'] ?= options.yarn.group

## Ambari Config Groups
      
      options.config_groups ?= {}
      options.groups ?= {}
      for srv in service.deps.yarn
        for name in options.groups
          srv.options.config_groups ?= {}
          srv.options.config_groups[name] ?= {}
          srv.options.config_groups[name]['hosts'] ?= []
          srv.options.config_groups[name]['hosts'].push service.node.fqdn unless srv.options.config_groups[name]['hosts'].indexOf(service.node.fqdn) > -1


## Dependencies

    {merge} = require 'nikita/lib/misc'
