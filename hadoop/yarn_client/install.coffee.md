
# YARN Client Install

    module.exports = header: 'YARN Client Install', handler: (options) ->

## Register

      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"

## Identities

By default, the "hadoop-yarn" package create the following entries:

```bash
cat /etc/passwd | grep yarn
yarn:x:2403:2403:Hadoop YARN User:/var/lib/hadoop-yarn:/bin/bash
cat /etc/group | grep yarn
hadoop:x:499:yarn
```

      @system.group header: 'Group', options.group
      @system.user header: 'User', options.user

## Packages Installation

      @call header: 'Packages', ->
        @service
          name: 'hadoop'
        @service
          name: 'hadoop-yarn'
        @service
          name: 'hadoop-client'


### YARN_CLIENT component wait
Wait for the YARN_CLIENT component to be declared on the host

      @ambari.hosts.component_wait
        header: 'YARN_CLIENT WAITED'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'YARN_CLIENT'
        hostname: options.fqdn

### YARN_CLIENT component install
Put the YARN_CLIENT component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'YARN_CLIENT set installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'YARN_CLIENT'
        hostname: options.fqdn

