
# Hadoop YARN Timeline Server Start

Start the Yarn Application History Server. You can also start the server
manually with the following command:

```
service hadoop-yarn-timelineserver start
su -l yarn -c "/usr/hdp/current/hadoop-yarn-timelineserver/sbin/yarn-daemon.sh --config /etc/hadoop/conf start timelineserver"
```

The ATS requires HDFS to be operationnal or an exception is trown: 
"java.lang.IllegalArgumentException: java.net.UnknownHostException: {cluster name}".

    module.exports = header: 'YARN ATS Ambari Start', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

Wait for Kerberos and the HDFS NameNode.

      @call 'masson/core/krb5_client/wait', once: true, options.wait_krb5_client
      @call 'ryba-ambari-takeover/hadoop/hdfs_nn/wait', once: true, options.wait_hdfs_nn, conf_dir: options.conf_dir

## Start Service

Start the Yarn NodeManager service. Using ambari REST API, the following is the
CURL equivalent command

```
curl 
```

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'APP_TIMELINE_SERVER'
        hostname: options.fqdn
