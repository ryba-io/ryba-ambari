
# Spark Job Thrift Server Start

Start the Spark Job HISTORYSERVER via AMBARI's REST API.

    module.exports = header: 'Ambari Spark Thrift Server Start', handler: ({options}) ->
    
## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Start Service

Start the Ranger-Admin service. Using ambari REST API, the following is the
CURL equivalent command

```
curl 
```

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'SPARK_JOBHISTORYSERVER'
        hostname: options.fqdn

