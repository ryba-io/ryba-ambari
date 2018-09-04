
# Ranger Admin Start

Start the ranger admin service server. You can also start the server
manually with the following command:

```
service ranger-admin start
systemctl start ranger-admin
```

    module.exports = header: 'Ambari Ranger Admin Start', handler: ({options}) ->
    
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
        component_name: 'RANGER_ADMIN'
        hostname: options.fqdn

