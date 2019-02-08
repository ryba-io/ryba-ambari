
# Kafka Broker Install

    module.exports = header: 'Ambari Kafka Broker Install', handler: ({options}) ->

## Register

      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['file', 'jaas'], 'ryba/lib/file_jaas'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"

## Wait

      @call 'masson/core/krb5_client/wait', once: true, options.wait_krb5_client

## IPTables

| Service      | Port  | Proto       | Parameter          |
|--------------|-------|-------------|--------------------|
| Kafka Broker | 9092  | http        | port               |
| Kafka Broker | 9093  | https       | port               |
| Kafka Broker | 9094  | sasl_http   | port               |
| Kafka Broker | 9096  | sasl_https  | port               |

## Package

Install the Kafka consumer package and set it to the latest version. Note, we
select the "kafka-broker" hdp directory. There is no "kafka-consumer"
directories.

      @system.tmpfs
        mount: options.pid_dir
        uid: options.user.name
        gid: options.group.name
        perm: '0750'

Modify bin scripts to set $KAFKA_HOME variable to match /etc/kafka-broker/conf.
Replace KAFKA_BROKER_CMD kafka-broker conf directory path
This Fixs are needed to be able to isolate confs betwwen broker and client

      # @call header: 'Fix Startup Script', handler: ->
      #   # @file
      #   #   target: "/usr/hdp/current/kafka-broker/bin/kafka"
      #   #   write: [
      #   #     match: /^KAFKA_BROKER_CMD=(.*)/m
      #   #     replace: "KAFKA_BROKER_CMD=\"$KAFKA_HOME/bin/kafka-server-broker-start.sh #{options.conf_dir}/server.properties\""
      #   #   ]
      #   #   backup: true
      #   #   eof: true
      #   @file
      #     target: '/usr/hdp/current/kafka-broker/bin/kafka-server-start.sh'
      #     write: [
      #           match: RegExp "^exec.*$", 'mg'
      #           replace: "exec /usr/hdp/current/kafka-broker/bin/kafka-run-broker-class.sh $EXTRA_ARGS kafka.Kafka #{options.conf_dir}/server.properties # RYBA DON'T OVERWRITE"
      #       ]
      #     backup: true
      #     eof: true
      #     mode: 0o755
      #   @system.copy
      #     source: '/usr/hdp/current/kafka-broker/bin/kafka-run-class.sh'
      #     target: '/usr/hdp/current/kafka-broker/bin/kafka-run-broker-class.sh'
      #     mode: 0o0755
      #   @file
      #     target: '/usr/hdp/current/kafka-broker/bin/kafka-run-broker-class.sh'
      #     write: [
      #       match: RegExp "^KAFKA_ENV=.*$", 'mg'
      #       replace: "KAFKA_ENV=#{options.conf_dir}/kafka-env.sh # RYBA DON'T OVERWRITE"
      #     ,
      #       match: RegExp "KAFKA_GC_LOG_OPTS=\"[^\"]+\"", 'mg'
      #       replace: """
      #       if [ -z "$KAFKA_GC_LOG_OPTS" ]; then
      #           KAFKA_GC_LOG_OPTS="-Xloggc:$LOG_DIR/$GC_LOG_FILE_NAME -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps "
      #         fi
      #       """
      #       replace: "KAFKA_ENV=#{options.conf_dir}/kafka-env.sh # RYBA DON'T OVERWRITE"
      #     ]
      #     backup: true
      #     eof: true
      #     mode: 0o755

# ## Kerberos
# 
# Broker Server principal, keytab and JAAS
# 
#       @call
#         header: 'Kerberos'
#         if: (options.config['zookeeper.set.acl'] is 'true') and options.manage_identities
#         handler: ->
#           @krb5.addprinc options.krb5.admin,
#             header: 'Broker Server Kerberos'
#             principal: options.kerberos.principal.replace '_HOST', options.fqdn
#             randkey: true
#             keytab: options.kerberos.keyTab
#             uid: options.user.name
#             gid: options.group.name
# 
# Kafka Superuser principal generation
# 
#           @krb5.addprinc options.krb5.admin,
#             header: 'Kafka Superuser kerberos'
#             principal: options.admin.principal
#             password: options.admin.password
# 
# # SSL Server
# 
# Upload and register the SSL certificate and private key.
# SSL is enabled at least for inter broker communication


# 
# ## Layout
# 
# Directories in which Kafka data is stored. Each new partition that is created
# will be placed in the directory which currently has the fewest partitions.
# 
#       @system.mkdir (
#         header: "Data dir #{dir}"
#         target: dir
#         uid: options.user.name
#         gid: options.group.name
#         mode: 0o0750
#         parent: true
#       ) for dir in options.config['log.dirs'].split ','

## Dependencies

    glob = require 'glob'
    path = require 'path'
    quote = require 'regexp-quote'
