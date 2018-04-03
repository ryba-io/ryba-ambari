
# Zookeeper Client Install

    module.exports = header: 'ZooKeeper Client Ambari Install', handler: (options) ->

## Register

      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['file', 'jaas'], 'ryba/lib/file_jaas'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','cluster','node_add'], 'ryba-ambari-actions/lib/cluster/node_add'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"

## Users & Groups

By default, the "zookeeper" package create the following entries:

```bash
cat /etc/passwd | grep zookeeper
zookeeper:x:497:498:ZooKeeper:/var/run/zookeeper:/bin/bash
cat /etc/group | grep hadoop
hadoop:x:498:hdfs
```

      @system.group header: "Group #{options.hadoop_group.name}", options.hadoop_group
      @system.group header: "Group #{options.group.name}", options.group
      @system.user header: "User #{options.user.name}", options.user

## Packages

Follow the [HDP recommandations][install] to install the "zookeeper" package
which has no dependency.

      @call header: 'Packages', ->
        @service
          name: 'zookeeper'
        @hdp_select
          name: 'zookeeper-client'

## Kerberos

Create the JAAS client configuration file.

      @file.jaas
        header: 'Kerberos'
        target: "#{options.conf_dir}/zookeeper-client.jaas"
        content: Client:
          useTicketCache: 'true'
        mode: 0o644

## Environment

Generate the "zookeeper-env.sh" file.

      @file
        header: 'Environment'
        target: "#{options.conf_dir}/zookeeper-env.sh"
        content: ("export #{k}=\"#{v}\"" for k, v of options.env).join '\n'
        backup: true
        eof: true

      @ambari.services.wait
        header: 'WAIT Service'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'ZOOKEEPER'

      @ambari.services.component_add
        header: 'ADD COMPONENT TO SERVICE'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZOOKEEPER_CLIENT'
        service_name: 'ZOOKEEPER'

      @ambari.hosts.component_add
        header: 'ADD COMPONENT TO HOST'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZOOKEEPER_CLIENT'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'set Installed'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZOOKEEPER_CLIENT'
        hostname: options.fqdn
