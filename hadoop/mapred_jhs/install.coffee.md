
# MapReduce JobHistoryServer Install

Install and configure the MapReduce Job History Server (JHS).

Run the command `./bin/ryba install -m ryba-ambari-takeover/hadoop/mapred_jhs` to install the
Job History Server.

    module.exports = header: 'Mapreduce Ambari JHS Install', handler: (options) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'

## IPTables

| Service    | Port  | Proto | Parameter                                 |
|------------|-------|-------|-------------------------------------------|
| jobhistory | 10020 | tcp   | mapreduce.jobhistory.address              |
| jobhistory | 19888 | http  | mapreduce.jobhistory.webapp.address       |
| jobhistory | 19889 | https | mapreduce.jobhistory.webapp.https.address |
| jobhistory | 13562 | tcp   | mapreduce.shuffle.port                    |
| jobhistory | 10033 | tcp   | mapreduce.jobhistory.admin.address        |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      jhs_shuffle_port = options.mapred_site['mapreduce.shuffle.port']
      jhs_port = options.mapred_site['mapreduce.jobhistory.address'].split(':')[1]
      jhs_admin_port = options.mapred_site['mapreduce.jobhistory.admin.address'].split(':')[1]
      rules = [
        { chain: 'INPUT', jump: 'ACCEPT', dport: jhs_port, protocol: 'tcp', state: 'NEW', comment: "MapRed JHS Server" }
        { chain: 'INPUT', jump: 'ACCEPT', dport: jhs_shuffle_port, protocol: 'tcp', state: 'NEW', comment: "MapRed JHS Shuffle" }
        { chain: 'INPUT', jump: 'ACCEPT', dport: jhs_admin_port, protocol: 'tcp', state: 'NEW', comment: "MapRed JHS Admin Server" }
      ]
      if options.mapred_site['mapreduce.jobhistory.http.policy'] is 'HTTP_ONLY'
        jhs_webapp_port = options.mapred_site['mapreduce.jobhistory.webapp.address'].split(':')[1]
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: jhs_webapp_port, protocol: 'tcp', state: 'NEW', comment: "MapRed JHS WebApp" }
      else
        jhs_webapp_https_port = options.mapred_site['mapreduce.jobhistory.webapp.https.address'].split(':')[1]
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: jhs_webapp_https_port, protocol: 'tcp', state: 'NEW', comment: "MapRed JHS WebApp" }
      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: rules

## Service

Install the "hadoop-mapreduce-historyserver" service, symlink the rc.d startup
script inside "/etc/init.d" and activate it on startup.

      @call header: 'Service', ->
        @service
          name: 'hadoop-mapreduce-historyserver'
        @hdp_select
          name: 'hadoop-mapreduce-client' # Not checked
          name: 'hadoop-mapreduce-historyserver'
        @system.tmpfs
          header: 'Run dir'
          if_os: name: ['redhat','centos'], version: '7'
          mount: "#{options.pid_dir}"
          uid: options.user.name
          gid: options.hadoop_group.name
          perm: '0755'

## Layout

Create the log and pid directories.

      @call header: 'Layout', ->
        @system.mkdir
          target: "#{options.log_dir}"
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o0755
        @system.mkdir
          target: "#{options.pid_dir}"
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o0755
        @system.mkdir
          target: options.mapred_site['mapreduce.jobhistory.recovery.store.leveldb.path']
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o0750
          parent: true
          if: options.mapred_site['mapreduce.jobhistory.recovery.store.class'] is 'org.apache.hadoop.mapreduce.v2.hs.HistoryServerLeveldbStateStoreService'

## Kerberos

Create the Kerberos service principal by default in the form of
"jhs/{host}@{realm}" and place its keytab inside
"/etc/security/keytabs/jhs.service.keytab" with ownerships set to
"mapred:hadoop" and permissions set to "0600".

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos'
        principal: options.mapred_site['mapreduce.jobhistory.principal']
        randkey: true
        keytab: options.mapred_site['mapreduce.jobhistory.keytab']
        uid: options.user.name
        gid: options.hadoop_group.name
        mode: 0o0600

## HDFS Layout

Layout is inspired by [Hadoop recommandation](http://hadoop.apache.org/docs/r2.1.0-beta/hadoop-project-dist/hadoop-common/ClusterSetup.html)

      @system.execute
        header: 'HDFS Layout'
        cmd: mkcmd.hdfs options.hdfs_krb5_user, """
        modified=""
        if ! hdfs --config #{options.hadoop_conf_dir} dfs -test -d #{options.mapred_site['yarn.app.mapreduce.am.staging-dir']}/history; then
          hdfs --config #{options.hadoop_conf_dir} dfs -mkdir -p #{options.mapred_site['yarn.app.mapreduce.am.staging-dir']}/history
          hdfs --config #{options.hadoop_conf_dir} dfs -chmod 0755 #{options.mapred_site['yarn.app.mapreduce.am.staging-dir']}/history
          hdfs --config #{options.hadoop_conf_dir} dfs -chown #{options.user.name}:#{options.hadoop_group.name} #{options.mapred_site['yarn.app.mapreduce.am.staging-dir']}/history
          modified=1
        fi
        if ! hdfs --config #{options.hadoop_conf_dir} dfs -test -d /app-logs; then
          hdfs --config #{options.hadoop_conf_dir} dfs -mkdir -p /app-logs
          hdfs --config #{options.hadoop_conf_dir} dfs -chmod 1777 /app-logs
          hdfs --config #{options.hadoop_conf_dir} dfs -chown #{options.user.name} /app-logs
          modified=1
        fi
        if [ $modified != "1" ]; then exit 2; fi
        """
        code_skipped: 2

### HISTORYSERVER component wait
Wait for the HISTORYSERVER component declared on the host

      @ambari.hosts.component_wait
        header: 'HISTORYSERVER WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HISTORYSERVER'
        hostname: options.fqdn

### HISTORYSERVER component install
Put the HISTORYSERVER component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'HISTORYSERVER set installed'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HISTORYSERVER'
        hostname: options.fqdn

      #fix overriden property by ambari when kerberos is installed
      # ats.service.keytab become yarn.service.keytab
      @ambari.configs.update
        header: 'Fix hadoop-env'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hadoop-env'
        cluster_name: options.cluster_name
        properties:
          'hdfs_principal_name': options.hdfs_krb5_user.principal

## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'

[keys]: https://github.com/apache/hadoop-common/blob/trunk/hadoop-hdfs-project/hadoop-hdfs/src/main/java/org/apache/hadoop/hdfs/DFSConfigKeys.java
