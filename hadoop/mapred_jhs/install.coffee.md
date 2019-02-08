
# MapReduce JobHistoryServer Install

Install and configure the MapReduce Job History Server (JHS).

Run the command `./bin/ryba install -m ryba-ambari-takeover/hadoop/mapred_jhs` to install the
Job History Server.

    module.exports = header: 'Mapreduce Ambari JHS Install', handler: ({options}) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'

## IPTables

      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: rules


## Layout

Create the log and pid directories.

      @call header: 'Layout', ->
        @system.mkdir
          target: options.mapred_site['mapreduce.jobhistory.recovery.store.leveldb.path']
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o0750
          parent: true
          if: options.mapred_site['mapreduce.jobhistory.recovery.store.class'] is 'org.apache.hadoop.mapreduce.v2.hs.HistoryServerLeveldbStateStoreService'

      #fix overriden property by ambari when kerberos is installed
      # ats.service.keytab become yarn.service.keytab
      # @ambari.configs.update
      #   header: 'Fix hadoop-env'
      #   if: options.takeover
      #   url: options.ambari_url
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   config_type: 'hadoop-env'
      #   cluster_name: options.cluster_name
      #   properties:
      #     'hdfs_principal_name': options.hdfs_krb5_user.principal

## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'

[keys]: https://github.com/apache/hadoop-common/blob/trunk/hadoop-hdfs-project/hadoop-hdfs/src/main/java/org/apache/hadoop/hdfs/DFSConfigKeys.java
