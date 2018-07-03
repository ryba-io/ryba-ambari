# Ranger Admin Stop

Stop the ranger admin service server. You can also stop the server
manually with the following command:

```
service ranger-admin stop
```

    module.exports = header: 'Ambari Ranger Admin Stop', handler: (options) ->
    
## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Stop Service

Stop the RANGER_ADMIN RANGER's service component. 
```
  curl ...
```

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'RANGER_ADMIN'
        hostname: options.fqdn

## Clean Logs

      @system.execute
        header: 'Clean Logs'
        if: options.clean_logs
        cmd: 'rm /var/log/ranger/admin/*'
        code_skipped: 1
