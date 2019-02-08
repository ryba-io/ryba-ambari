
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

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## HDFS, YARN, Mapred site
Merge hdfs_site, yarn_site, core_site configuration from each components.

      options.configurations ?= {}
      options.configurations['yarn-site'] ?= {}
      options.configurations['yarn-site']['yarn.resourcemanager.webapp.address'] ?= 'localhost:8090'

## YARN Admin ACL

      options.configurations['yarn-site']['yarn.admin.acl'] ?= "#{options.yarn.user.name}, #{if service.deps.hdfs[0]? then (service.deps.hdfs[0].options.hdfs.user.name + ',dr.who') else 'dr.who'}"

## YARN Env
Inhertis Env properties from yarn_core components. For Components `YARN_RESOURCEMANAGER_OPTS`,
`YARN_NODEMANAGER_OPTS`,  `YARN_TIMELINESERVER_OPTS` properties will be rendered at
as install time, like in any other components.

      # ambari required properties
      options.configurations['yarn-env'] ?= {}
      options.configurations['yarn-env']['hadoop_yarn_home'] ?= options.yarn.user.home
      options.configurations['yarn-env']['yarn_user'] ?= options.yarn.user.name
      options.configurations['yarn-env']['yarn_tmp_dir'] ?= "#{options.yarn.log_dir}/tmp"
      options.configurations['yarn-env']['yarn_user_nofile_limit'] ?= options.yarn.user.limits.nofile
      options.configurations['yarn-env']['yarn_user_nproc_limit'] ?= options.yarn.user.limits.nproc
      options.configurations['yarn-env']['yarn_heapsize'] ?= '1024'
      options.configurations['yarn-env']['yarn_log_dir_prefix'] ?= options.yarn.log_dir
      options.configurations['yarn-env']['yarn_pid_dir_prefix'] ?= options.yarn.pid_dir
      options.configurations['yarn-env']['min_user_id'] ?= '1000'

## SSL

      options.ssl = merge {}, service.deps.ssl?.options, options.ssl
      options.ssl.enabled ?= !!service.deps.ssl
      if options.ssl.enabled
        options.ssl_client ?= {}
        options.ssl_server ?= {}
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert

## Dependencies

    {merge} = require 'nikita/lib/misc'
