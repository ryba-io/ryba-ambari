
# Hadoop YARN ResourceManager Start

Start the ResourceManager server. You can also start the server manually with the
following two commands:

```
service hadoop-yarn-resourcemanager start
su -l yarn -c "export HADOOP_LIBEXEC_DIR=/usr/hdp/current/hadoop-client/libexec && /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh --config /etc/hadoop/conf start resourcemanager"
```

    module.exports = header: 'YARN RM Ambari Start', retry: 3, handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

Wait for Kerberos, Zookeeper, HDFS, YARN and the MapReduce History Server. The
History Server must be started because the ResourceManager will try to load
the history of MR jobs from there.

      @call 'masson/core/krb5_client/wait', once: true, options.wait_krb5_client
      @call 'ryba-ambari-takeover/zookeeper/server/wait', once: true, options.wait_zookeeper_server
      @call 'ryba-ambari-takeover/hadoop/hdfs_dn/wait', once: true, options.wait_hdfs_dn
      @call 'ryba-ambari-takeover/hadoop/yarn_ts/wait', once: true, options.wait_yarn_ts
      @call 'ryba-ambari-takeover/hadoop/mapred_jhs/wait', once: true, options.wait_mapred_jhs

## Cleanup

Ensure the service pid is removed on retry.

      @system.remove
        target: "#{options.pid_dir}/yarn-#{options.user.name}-resourcemanager.pid"
        if: options.attempt > 0

## Start Service

Start the Yarn ResourceManager service. Using ambari REST API, the following is the
CURL equivalent command

```
curl 
```

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'RESOURCEMANAGER'
        hostname: options.fqdn
