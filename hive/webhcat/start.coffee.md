
# WebHCat Start

Run the command `./bin/ryba start -m ryba-ambari-takeover/hive/webhcat` to start the WebHCat
server using Ryba.

By default, the pid of the running server is stored in
"/var/run/webhcat/webhcat.pid".


    module.exports = header: 'WebHCat Start', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

Wait for Kerberos, Zookeeper, Hadoop and Hive HCatalog.

      @call 'masson/core/krb5_client/wait', once: true, options.wait_krb5_client
      @call 'ryba-ambari-takeover/zookeeper/server/wait', once: true, options.wait_zookeeper_server
      @call 'ryba-ambari-takeover/hive/hcatalog/wait', once: true, options.wait_hive_hcatalog

## Service

Start the WebHCat server. You can also start the server manually with one of the
following two commands:

```
su -l hive -c "/usr/hdp/current/hive-webhcat/sbin/webhcat_server.sh start"
```

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'WEBHCAT_SERVER'
        hostname: options.fqdn
