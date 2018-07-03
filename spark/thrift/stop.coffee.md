
# Spark Job Thrift Server Stop

Stops the Spark Job HISTORYSERVER via AMBARI's REST API.

    module.exports = header: 'Ambari Spark Thrift Server Stop', handler: (options) ->
    
## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Start Service

Start the Ranger-Admin service. Using ambari REST API, the following is the
CURL equivalent command

```
curl 
```

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'SPARK_JOBHISTORYSERVER'
        hostname: options.fqdn