
# Zookeeper Server Install

    module.exports = header: 'ZooKeeper Server Ambari Install', handler: (options) ->

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

## Wait

      @call 'masson/core/krb5_client/wait', options.wait_krb5_client
      @call 'ryba/ambari/server/wait', rest: options.wait_ambari_rest

## IPTables

| Service    | Port | Proto  | Parameter             |
|------------|------|--------|-----------------------|
| zookeeper  | 2181 | tcp    | zookeeper.port        |
| zookeeper  | 2888 | tcp    | zookeeper.peer_port   |
| zookeeper  | 3888 | tcp    | zookeeper.leader_port |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      rules = [
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.peer_port, protocol: 'tcp', state: 'NEW', comment: "Zookeeper Peer" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.leader_port, protocol: 'tcp', state: 'NEW', comment: "Zookeeper Leader" }
      ]
      if options.env["JMXPORT"]?
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: parseInt(options.env["JMXPORT"],10), protocol: 'tcp', state: 'NEW', comment: "Zookeeper JMX" }

We open the client port if:
- the node is an observer
- the node is participant but there is no other observer on the cluster

      if options.config['peerType'] is 'observer' or not options.has_observers
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.config['clientPort'], protocol: 'tcp', state: 'NEW', comment: "Zookeeper Client" }
      @tools.iptables
        header: 'IPTables'
        rules: rules
        if: options.iptables

## Packages

Follow the [HDP recommandations][install] to install the "zookeeper" package
which has no dependency.

      @call header: 'Packages', ->
        @service
          name: 'nc' # Used by check
        @service
          name: 'zookeeper-server'
        @hdp_select
          name: 'zookeeper-server'
        @hdp_select
          name: 'zookeeper-client'
        @system.tmpfs
          if_os: name: ['redhat','centos'], version: '7'
          mount: options.pid_dir
          uid: options.user.name
          gid: options.group.name
          perm: '0750'

## Kerberos

      @call header: 'Kerberos', ->
        @krb5.addprinc options.krb5.admin,
          principal: options.krb5.principal.replace '_HOST', options.fqdn
          randkey: true
          keytab: options.krb5.keytab
          uid: options.user.name
          gid: options.hadoop_group.name

## Layout

Create the data, pid and log directories with the correct permissions and
ownerships.

      @call header: 'Layout', ->
        @system.mkdir
          target: options.config['dataDir']
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o755
        @system.mkdir
          target: options.pid_dir
          uid: options.user.name
          gid: options.group.name
          mode: 0o755
        @system.mkdir
          target: options.log_dir
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o755

## Environment

Note, environment is enriched at runtime if a super user is generated
(see above).

      @call
        if: options.post_component
        header: 'Zookeeper Env'
      , ->
        options.env['JAVA_OPTS'] = ''
        options.env['JAVA_OPTS'] += " -D#{k}=#{v}" for k, v of options.opts.java_properties
        options.env['JAVA_OPTS'] += " #{k}#{v}" for k, v of options.opts.jvm
        @file
          header: 'Render'
          target: "#{options.cache_dir}/zookeeper-env.sh"
          content: ("export #{k}=\"#{v}\"" for k, v of options.env).join '\n'
          backup: true
          eof: true
          ssh: false
        @call
          header: 'Read'
          if: options.takeover
        , (_, callback)->
          ssh2fs.readFile null, "#{options.cache_dir}/zookeeper-env.sh", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload'
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'zookeeper-env'
                cluster_name: options.cluster_name
                properties: 
                  content: content
                  zk_log_dir: options.log_dir
                  zk_pid_dir: '/var/run/zookeeper'
                  zk_user: options.user.name
                  tickTime: options.config['tickTime']
                  initLimit: options.config['initLimit']
                  syncLimit: options.config['syncLimit']
                  clientPort: options.config['clientPort']
                  zookeeper_keytab_path: options.krb5.keytab
                  zookeeper_principal_name: options.krb5.principal
              .next callback
            catch err
              callback err
        

## Configure

Update the file "zoo.cfg" with the properties defined by the
"ryba.zookeeper.config" configuration.

      @ambari.configs.update
        if: options.post_component and options.takeover
        header: 'Upload zoo.cfg'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'zoo.cfg'
        cluster_name: options.cluster_name
        properties: options.config


## Log4J

Write the ZooKeeper logging configuration file.

      @ambari.configs.update
        if: options.post_component and options.takeover
        header: 'Upload Log4j'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'zookeeper-log4j'
        cluster_name: options.cluster_name
        properties: options.log4j.properties

## Schedule Purge Transaction Logs

A ZooKeeper server will not remove old snapshots and log files when using the
default configuration (see autopurge below), this is the responsibility of the
operator.

The PurgeTxnLog utility implements a simple retention policy that administrators
can use. Its expected arguments are "dataLogDir [snapDir] -n count".

Note, Automatic purging of the snapshots and corresponding transaction logs was
introduced in version 3.4.0 and can be enabled via the following configuration
parameters autopurge.snapRetainCount and autopurge.purgeInterval.

```
/usr/bin/java \
  -cp /usr/hdp/current/zookeeper-server/zookeeper.jar:/usr/hdp/current/zookeeper-server/lib/*:/usr/hdp/current/zookeeper-server/conf \
  org.apache.zookeeper.server.PurgeTxnLog  /var/zookeeper/data/ -n 3
```

      @cron.add
        header: 'Schedule Purge'
        if: options.purge
        cmd: """
        /usr/bin/java -cp /usr/hdp/current/zookeeper-server/zookeeper.jar:/usr/hdp/current/zookeeper-server/lib/*:/usr/hdp/current/zookeeper-server/conf \
          org.apache.zookeeper.server.PurgeTxnLog \
          #{options.config.dataLogDir or ''} #{options.config.dataDir} -n #{options.retention}
        """
        when: options.purge
        user: options.user.name

## Write myid

myid is a unique id that must be generated for each node of the zookeeper cluster

      @file
        header: 'Write id'
        content: options.id
        target: "#{options.config['dataDir']}/myid"
        uid: options.user.name
        gid: options.hadoop_group.name

## Add ZOOKEEPER COMPONENT
add the ZOOKEEPER component in ambari before addin ZOOKEEPER_SERVER COMPONENT

      @ambari.services.add
        header: 'ADD Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'ZOOKEEPER'

      @ambari.services.wait
        header: 'WAIT Service'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'ZOOKEEPER'

      @ambari.services.component_add
        header: 'ADD COMPONENT TO SERVICE'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZOOKEEPER_SERVER'
        service_name: 'ZOOKEEPER'

      @ambari.hosts.component_add
        header: 'ADD COMPONENT TO HOST'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZOOKEEPER_SERVER'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'set Installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZOOKEEPER_SERVER'
        hostname: options.fqdn


## Dependencies

    ssh2fs = require 'ssh2-fs'


## Resources

* [ZooKeeper Resilience](http://blog.cloudera.com/blog/2014/03/zookeeper-resilience-at-pinterest/)
* [HDP Install Instructions]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1-latest/bk_installing_manually_book/content/rpm-zookeeper-1.html
