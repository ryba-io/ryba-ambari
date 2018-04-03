
# Knox Stop

    module.exports = header: 'Ambari Knox Stop', handler: ->

## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Stop Service

Stop the KNOX_GATEWAY Component. Using ambari REST API, the following is the
CURL equivalent command

```
curl 
```

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'KNOX_GATEWAY'
        hostname: options.fqdn