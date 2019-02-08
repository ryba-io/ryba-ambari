
# Hadoop takeover Configure

This module is separated as Ambari does not support isolated configuration. The modules
reads configuration from all the hadoop components and merge them in order to provide
hadoop main configuration files. As a consequence, the components will have more
properties than needed.

    module.exports = (service) ->
      options = service.options
      options.mapred ?= {}
      options.configurations ?= {}

## Identities

      # Group for hadoop
      options.hadoop_group = name: options.hadoop_group if typeof options.hadoop_group is 'string'
      options.hadoop_group ?= {}
      options.hadoop_group.name ?= 'hadoop'
      options.hadoop_group.system ?= true
      options.hadoop_group.comment ?= 'Hadoop Group'
      # Groups
      options.mapred.group ?= {}
      options.mapred.group = name: options.mapred.group if typeof options.mapred.group is 'string'
      options.mapred.group.name ?= 'mapred'
      options.mapred.group.system ?= true
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
      options.mapred.user.limits.nofile ?= 32000
      options.mapred.user.limits.nproc ?= 32000

## Environment

      options.conf_dir ?= '/etc/hadoop/conf'
      options.mapred.log_dir ?= '/var/log/hadoop-mapreduce'
      options.mapred.pid_dir_prefix ?= '/var/run/hadoop'
      options.mapred.pid_dir ?= "#{options.mapred.pid_dir_prefix}/#{options.mapred.user.name}"

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## HDFS kerberos User

      # HDFS Super User
      options.hdfs_krb5_user ?= service.deps.hdfs.options.hdfs.krb5_user
      throw Error "Required Property: hdfs.krb5_user.password" unless options.hdfs_krb5_user.password

## Ambari Configuration

      options.configurations ?= {}

## HDFS, YARN, Mapred site
Merge hdfs_site, yarn_site, core_site configuration from each components.

      options.configurations['mapred-site'] ?= {}

## HADOOP, YARN Env
Inhertis Env properties from hadoop_core components. For Components `HADOOP_DATANODE_OPTS`,
`HADOOP_NAMENODE_OPTS`,  `HADOOP_JOURNALNODE_OPTS` properties will be rendered at
as install time, like in any other components.
For this reason components system opts are regiester.

      options.mapred_jhs_opts ?= {}
      #opts

## MAPRED Env

      options.configurations['mapred-env'] ?= {}
      # ambari required properties
      options.configurations['mapred-env']['mapred_user'] ?= options.mapred.user.name
      options.configurations['mapred-env']['mapred_user_nofile_limit'] ?= options.mapred.user.limits.nofile
      options.configurations['mapred-env']['mapred_user_nproc_limit'] ?= options.mapred.user.limits.nproc
      options.configurations['mapred-env']['jobhistory_heapsize'] ?= '1024'
      options.configurations['mapred-env']['mapred_log_dir_prefix'] ?= options.mapred.log_dir
      options.configurations['mapred-env']['mapred_pid_dir_prefix'] ?= options.mapred.pid_dir_prefix
      options.configurations['mapred-env']['mapred_jobstatus_dir'] ?= '/var/mapred/jhs'

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
