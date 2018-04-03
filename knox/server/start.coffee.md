
# Knox Start

    module.exports = header: 'Ambari Knox Server Start', handler: (options) ->

## Wait
Knox doesn't seem to re-sync when ranger-admin is not available. Add wait to ensure plugin
does not stop syncing.

      @call 'ryba/ranger/admin/wait', once: true, options.wait_ranger_admin if options.wait_ranger_admin

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
        component_name: 'KNOX_GATEWAY'
        hostname: options.fqdn

