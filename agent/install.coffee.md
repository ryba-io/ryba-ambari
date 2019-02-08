# Ambari Agent Install

The ambari server must be set in the configuration file.

    module.exports = header: 'Ambari Agent Install', handler: ({options}) ->

## Identities

By default, the "ambari-agent" package does not create any identities.
Create System Service Account & user/client accounts

      @call ->
        for name, group of options.groups
          @system.group
            if: (group.uid in options.only) or (group.name in options.only) or (options.only.length is 0)
            header: "Group #{name}", group
        for name, user of options.users
          @system.user
            if: (user.uid in options.only) or (user.name in options.only) or (options.only?.length is 0)
            header: "User #{name}", user

## Certificate Directory

      @system.mkdir
        header: 'Keytabs'
        target: '/etc/security/serverKeys'
        uid: 'root'
        gid: 'root' # was hadoop_group.name
        mode: 0o0755

## Java default Truststore
      @java.keystore_add
        header: "Java default Truststore"
        keystore: '/usr/java/latest/jre/lib/security/cacerts'
        storepass: 'changeit'
        caname: "hadoop_root_ca"
        cacert: "#{options.ssl.cacert.source}"
        local: "#{options.ssl.cacert.local}"

      @call
        if: options.importCerts?
      , (_, cb) ->
        truststore =
          target: '/usr/java/default/jre/lib/security/cacerts'
          password: 'changeit'
        tmp_location = "/tmp/ryba_cacert_#{Date.now()}"
        @each options.importCerts, ({options}, callback) ->
          {source, local, name} = options.value
          @file.download
            header: 'download cacert'
            source: source
            target: "#{tmp_location}/cacert"
            local: true
          @java.keystore_add
            header: "add cacert to #{name}"
            keystore: truststore.target
            storepass: truststore.password
            caname: name
            cacert: "#{tmp_location}/cacert"
          @next callback
        @system.remove
          target: tmp_location
        @next cb


## Yum packages

      @call
        header: 'MySQL Client'
        if: options.hive_hcatalog
      , ->
        @service
          if: options.hive_db.engine in ['mariadb', 'mysql']
          name: 'mysql'
        @service
          if: options.hive_db.engine in ['mariadb', 'mysql']
          name: 'mysql-connector-java'
        @service
          if: options.hive_db.engine is 'postgresql'
          name: 'postgresql'
        @service
          if: options.hive_db.engine is 'postgresql'
          name: 'postgresql-jdbc'

## SSL

      @call
        header: 'KAFKA SSL'
        if: -> options.kafka_broker
      , ->
        @java.keystore_add
          keystore: options.configurations['kafka-broker']['ssl.keystore.location']
          storepass: options.configurations['kafka-broker']['ssl.keystore.password']
          key: options.ssl.key.source
          cert: options.ssl.cert.source
          keypass: options.configurations['kafka-broker']['ssl.key.password']
          name: options.ssl.key.name
          local: options.ssl.cert.local
          user: options.users.kafka.name
          group: options.groups.kafka.name
        @java.keystore_add
          keystore: options.configurations['kafka-broker']['ssl.keystore.location']
          storepass: options.configurations['kafka-broker']['ssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local
          user: options.users.kafka.name
          group: options.groups.kafka.name
        # imports kafka broker server hadoop_root_ca CA truststore
        @java.keystore_add
          keystore: options.configurations['kafka-broker']['ssl.truststore.location']
          storepass: options.configurations['kafka-broker']['ssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local

      @call header: 'HADOOP SSL CLIENT', ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ssl-client']['ssl.client.truststore.location']
          storepass: options.configurations['ssl-client']['ssl.client.truststore.password']
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local
          gid:  options.hadoop_group.name
          mode: 0o640
        @java.keystore_add
          keystore: options.configurations['ssl-server']['ssl.server.truststore.location']
          storepass: options.configurations['ssl-server']['ssl.server.truststore.password']
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local
          gid:  options.hadoop_group.name
          mode: 0o640
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ssl-server']['ssl.server.keystore.location']
          storepass: options.configurations['ssl-server']['ssl.server.keystore.password']
          # caname: "hadoop_root_ca"
          # cacert: "#{options.ssl.cacert}"
          key: options.ssl.key.source
          cert: options.ssl.cert.source
          keypass: options.configurations['ssl-server']['ssl.server.keystore.keypassword']
          name: options.ssl.key.name
          local: options.ssl.key.local
          mode: 0o640
          gid:  options.hadoop_group.name
        @java.keystore_add
          keystore: options.configurations['ssl-server']['ssl.server.keystore.location']
          storepass: options.configurations['ssl-server']['ssl.server.keystore.password']
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local
          gid:  options.hadoop_group.name
          mode: 0o640

## SSL

      @call header: 'Ranger Atlas SSL', if: options.configurations['ranger-atlas-policymgr-ssl']?
      , ->
        @java.keystore_add
          keystore: "#{options.jre_home}/lib/security/cacerts"
          storepass: 'changeit'
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
          storepass: options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-atlas-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

      @call header: 'Ranger HDFS SSL', if: options.configurations['ranger-hdfs-policymgr-ssl']?
      , ->
        @java.keystore_add
          keystore: "#{options.jre_home}/lib/security/cacerts"
          storepass: 'changeit'
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
          storepass: options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-hdfs-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

      @call header: 'Ranger YARN SSL', if: options.configurations['ranger-yarn-policymgr-ssl']?
      , ->
        @java.keystore_add
          keystore: "#{options.jre_home}/lib/security/cacerts"
          storepass: 'changeit'
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ranger-yarn-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
          storepass: options.configurations['ranger-yarn-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ranger-yarn-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-yarn-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['ranger-yarn-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['ranger-yarn-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-yarn-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

      @call header: 'Ranger HBase SSL', if: options.configurations['ranger-hbase-policymgr-ssl']?
      , ->
        @java.keystore_add
          keystore: "#{options.jre_home}/lib/security/cacerts"
          storepass: 'changeit'
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
          storepass: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-hbase-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

      @call header: 'Ranger Hiveserver2 SSL', if: options.configurations['ranger-hive-policymgr-ssl']?
      , ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
          storepass: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-hive-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"


      @call header: 'Ranger Kafka SSL',  if: options.configurations['ranger-kafka-policymgr-ssl']?
      , ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
          storepass: options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-kafka-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

      @call header: 'Ranger Knox SSL', if: options.configurations['ranger-knox-policymgr-ssl']?
      , ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
          storepass: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore']
          storepass: options.configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"


        @call
          if: options.knox_importCerts?
        , (_, cb) ->
          {truststore, configurations} = options
          tmp_location = "/tmp/ryba_cacert_#{Date.now()}"
          @each options.importCerts, ({options}, callback) ->
            {source, local, name} = options.value
            @file.download
              header: 'download cacert'
              source: source
              target: "#{tmp_location}/cacert"
              local: true
            @java.keystore_add
              header: "add cacert to #{name}"
              keystore: configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.truststore']
              storepass: configurations['ranger-knox-policymgr-ssl']['xasecure.policymgr.clientssl.truststore.password']
              caname: name
              cacert: "#{tmp_location}/cacert"
            @next callback
          @system.remove
            target: tmp_location
          @next cb

      @call header: 'Ambari Infra SSL', if: options.configurations['infra-solr-env']?
      , ->
        @call
          if: options.importCerts?
        , (_, cb) ->
          truststore =
            target: options.configurations['infra-solr-env']['infra_solr_truststore_location']
            password: options.configurations['infra-solr-env']['infra_solr_truststore_password']
          tmp_location = "/tmp/ryba_cacert_#{Date.now()}"
          @each options.importCerts, ({options}, callback) ->
            {source, local, name} = options.value
            @file.download
              header: 'download cacert'
              source: source
              target: "#{tmp_location}/cacert"
              local: true
            @java.keystore_add
              header: "add cacert to #{name}"
              keystore: truststore.target
              storepass: truststore.password
              caname: name
              cacert: "#{tmp_location}/cacert"
            @next callback
          @system.remove
            target: tmp_location
          @next cb
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['infra-solr-env']['infra_solr_truststore_location']
          storepass: options.configurations['infra-solr-env']['infra_solr_truststore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.users['ambari-infra'].name
          gid: options.groups['ambari-infra'].name
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['infra-solr-env']['infra_solr_keystore_location']
          storepass: options.configurations['infra-solr-env']['infra_solr_keystore_password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['infra-solr-env']['infra_solr_keystore_password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
          uid: options.users['ambari-infra'].name
          gid: options.groups['ambari-infra'].name
        @java.keystore_add
          keystore: options.configurations['infra-solr-env']['infra_solr_keystore_location']
          storepass: options.configurations['infra-solr-env']['infra_solr_keystore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.users['ambari-infra'].name
          gid: options.groups['ambari-infra'].name

      @call header: 'HDFS NN SSL', if: options.hdfs_nn_ssl_client? , ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.hdfs_nn_ssl_client['ssl.client.truststore.location']
          storepass: options.hdfs_nn_ssl_client['ssl.client.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.hdfs_nn_ssl_server['ssl.server.keystore.location']
          storepass: options.hdfs_nn_ssl_server['ssl.server.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.hdfs_nn_ssl_server['ssl.server.keystore.keypassword']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.hdfs_nn_ssl_server['ssl.server.keystore.location']
          storepass: options.hdfs_nn_ssl_server['ssl.server.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

      @call header: 'Grafana SSL', if: options.ambari_grafana , ->
        # Client: import certificate to all hosts
        @file.download
          header: 'SSL Cert'
          source: options.ssl.cert.source
          target: options.configurations['ams-grafana-ini']['cert_file']
          local: options.ssl.cert.local
        @file.download
          header: 'SSL Key'
          source: options.ssl.key.source
          target: options.configurations['ams-grafana-ini']['cert_key']
          local: options.ssl.key.local

      @call header: 'LogSearch Feeder SSL', if: options.logsearch_feeder, ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['logfeeder-env']['logfeeder_truststore_location']
          storepass: options.configurations['logfeeder-env']['logfeeder_truststore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.logsearch_user.name
          gid: options.logsearch_group.name
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['logfeeder-env']['logfeeder_keystore_location']
          storepass: options.configurations['logfeeder-env']['logfeeder_keystore_password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['logfeeder-env']['logfeeder_keystore_password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
          uid: options.logsearch_user.name
          gid: options.logsearch_group.name
        @java.keystore_add
          keystore: options.configurations['logfeeder-env']['logfeeder_keystore_location']
          storepass: options.configurations['logfeeder-env']['logfeeder_keystore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.logsearch_user.name
          gid: options.logsearch_group.name

      @call header: 'LogSearch Server SSL',  if: options.logsearch_server, ->
        return unless options.configurations['logsearch-env']?['logsearch_truststore_location']?
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['logsearch-env']['logsearch_truststore_location']
          storepass: options.configurations['logsearch-env']['logsearch_truststore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.logsearch_user.name
          gid: options.logsearch_group.name
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['logsearch-env']['logsearch_keystore_location']
          storepass: options.configurations['logsearch-env']['logsearch_keystore_password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['logsearch-env']['logsearch_keystore_password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
          uid: options.logsearch_user.name
          gid: options.logsearch_group.name
        @java.keystore_add
          keystore: options.configurations['logsearch-env']['logsearch_keystore_location']
          storepass: options.configurations['logsearch-env']['logsearch_keystore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.logsearch_user.name
          gid: options.logsearch_group.name

      @call header: 'OOZIE SSL Server', if: options.oozie_server, ->
        return unless options.oozie_ssl?
        @java.keystore_add
          header: 'SSL'
          keystore: options.oozie_ssl.keystore.target
          storepass: options.oozie_ssl.keystore.password
          key: options.ssl.key.source
          cert: options.ssl.cert.source
          keypass: options.oozie_ssl.keystore.password
          name: options.ssl.key.name
          local: options.ssl.key.local
        @java.keystore_add
          keystore: options.oozie_ssl.keystore.target
          storepass: options.oozie_ssl.keystore.password
          caname: 'hadoop_root_ca'
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local
        # fix oozie pkix build exceptionm when oozie server connects to hadoop mr
        @java.keystore_add
          keystore: options.oozie_ssl.truststore.target
          storepass: options.oozie_ssl.truststore.password
          caname: 'hadoop_root_ca'
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local

      @call header: 'SmartSense Explorer SSL', if: options.smartsense_explorer, ->
        return unless options.configurations['activity-zeppelin-site']['zeppelin.ssl.truststore.path']?
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['activity-zeppelin-site']['zeppelin.ssl.truststore.path']
          storepass: options.configurations['activity-zeppelin-site']['zeppelin.ssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.path']
          storepass: options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.path']
          storepass: options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

      @call header: 'Zeppelin Master SSL', if: options.zeppelin_master, ->
        return unless options.configurations['zeppelin-config']['zeppelin.ssl.truststore.path']?
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['zeppelin-config']['zeppelin.ssl.truststore.path']
          storepass: options.configurations['zeppelin-config']['zeppelin.ssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.zeppelin_user.name
          gid: options.zeppelin_group.name
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['zeppelin-config']['zeppelin.ssl.keystore.path']
          storepass: options.configurations['zeppelin-config']['zeppelin.ssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['zeppelin-config']['zeppelin.ssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
          uid: options.zeppelin_user.name
          gid: options.zeppelin_group.name
        @java.keystore_add
          keystore: options.configurations['zeppelin-config']['zeppelin.ssl.keystore.path']
          storepass: options.configurations['zeppelin-config']['zeppelin.ssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.zeppelin_user.name
          gid: options.zeppelin_group.name
  
      @call header: 'Hive Client Truststore', if: options.hive_client, ->

        @java.keystore_add
          header: 'Client SSL'
          keystore: options.hive_client_truststore_location
          storepass: options.hive_client_truststore_password
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

      @call
        header: 'Hive Server2 SSL'
        if: -> options.hive_server2
      , ->
        return unless options.configurations['hive-site']['hive.server2.use.SSL'] is 'true'
        @java.keystore_add
          keystore: options.configurations['hive-site']['hive.server2.keystore.path']
          storepass: options.configurations['hive-site']['hive.server2.keystore.password']
          key: options.ssl.key.source
          cert: options.ssl.cert.source
          keypass: options.configurations['hive-site']['hive.server2.keystore.password']
          name: options.ssl.key.name
          local: options.ssl.key.local
        @java.keystore_add
          keystore: options.configurations['hive-site']['hive.server2.keystore.path']
          storepass: options.configurations['hive-site']['hive.server2.keystore.password']
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local

## ZOOKEEPER

      rules = []
      if options.zookeeper

| Service    | Port | Proto  | Parameter             |
|------------|------|--------|-----------------------|
| zookeeper  | 2181 | tcp    | zookeeper.port        |
| zookeeper  | 2888 | tcp    | zookeeper.peer_port   |
| zookeeper  | 3888 | tcp    | zookeeper.leader_port |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.zookeeper_peer_port, protocol: 'tcp', state: 'NEW', comment: "Zookeeper Peer" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.zookeeper_leader_port, protocol: 'tcp', state: 'NEW', comment: "Zookeeper Leader" }
        # if options.zookeeper?.env["JMXPORT"]?
        #   rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: parseInt(options.zookeeper.env["JMXPORT"],10), protocol: 'tcp', state: 'NEW', comment: "Zookeeper JMX" }
        if options.is_zookeeper_observer
          rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['zoo.cfg']['clientPort'], protocol: 'tcp', state: 'NEW', comment: "Zookeeper Client" }

We open the client port if:
- the node is an observer
- the node is participant but there is no other observer on the cluster

        if options.is_zookeeper_observer
          rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['zoo.cfg']['clientPort'], protocol: 'tcp', state: 'NEW', comment: "Zookeeper Client" }

| Service      | Port  | Proto       | Parameter          |
|--------------|-------|-------------|--------------------|
| Kafka Broker | 9092  | http        | port               |
| Kafka Broker | 9093  | https       | port               |
| Kafka Broker | 9094  | sasl_http   | port               |
| Kafka Broker | 9096  | sasl_https  | port               |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      if options.kafka_broker
        for protocol in options.kafka_protocols
          rules.push {chain: 'INPUT', jump: 'ACCEPT', dport: options.kafka_ports[protocol], protocol: 'tcp', state: 'NEW', comment: "Kafka Broker #{protocol}"}

## HDFS
    
      if options.hdfs_dn

| Service   | Port       | Proto     | Parameter                  |
|-----------|------------|-----------|----------------------------|
| datanode  | 50010/1004 | tcp/http  | dfs.datanode.address       |
| datanode  | 50075/1006 | tcp/http  | dfs.datanode.http.address  |
| datanode  | 50475      | tcp/https | dfs.datanode.https.address |
| datanode  | 50020      | tcp       | dfs.datanode.ipc.address   |
| journalnode | 8485 | tcp    | hdp.hdfs.site['dfs.journalnode.rpc-address']   |
| journalnode | 8480 | tcp    | hdp.hdfs.site['dfs.journalnode.http-address']  |
| journalnode | 8481 | tcp    | hdp.hdfs.site['dfs.journalnode.https-address'] |
| namenode | 50070 | tcp   | dfs.namdnode.http-address  |
| namenode | 50470 | tcp   | dfs.namenode.https-address |
| namenode | 8020  | tcp   | fs.defaultFS               |
| namenode  | 8019  | tcp   | dfs.ha.zkfc.port           |

The "dfs.datanode.address" default to "50010" in non-secured mode. In non-secured
mode, it must be set to a value below "1024" and default to "1004".

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

        [_, dn_address] = options.configurations['hdfs-site']['dfs.datanode.address'].split ':'
        [_, dn_http_address] = options.configurations['hdfs-site']['dfs.datanode.http.address'].split ':'
        [_, dn_https_address] = options.configurations['hdfs-site']['dfs.datanode.https.address'].split ':'
        [_, dn_ipc_address] = options.configurations['hdfs-site']['dfs.datanode.ipc.address'].split ':'
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: dn_address, protocol: 'tcp', state: 'NEW', comment: "HDFS DN Data" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: dn_http_address, protocol: 'tcp', state: 'NEW', comment: "HDFS DN HTTP" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: dn_https_address, protocol: 'tcp', state: 'NEW', comment: "HDFS DN HTTPS" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: dn_ipc_address, protocol: 'tcp', state: 'NEW', comment: "HDFS DN Meta" }
  
      if options.hdfs_jn
        #jn
        rpc = options.configurations['hdfs-site']['dfs.journalnode.rpc-address'].split(':')[1]
        http = options.configurations['hdfs-site']['dfs.journalnode.http-address'].split(':')[1]
        https = options.configurations['hdfs-site']['dfs.journalnode.https-address'].split(':')[1]
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: rpc, protocol: 'tcp', state: 'NEW', comment: "HDFS JournalNode" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: http, protocol: 'tcp', state: 'NEW', comment: "HDFS JournalNode" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: https, protocol: 'tcp', state: 'NEW', comment: "HDFS JournalNode" }


      if options.hdfs_nn
        # NN
        unless options.nameservice
          [_, port_rcp] = options.configurations['core-site']['fs.defaultFS'].split ':'
          [_, port_rcp] = options.configurations['hdfs-site']['dfs.namenode.http-address'].split ':'
          [_, port_rcp] = options.configurations['hdfs-site']['dfs.namenode.https-address'].split ':'
        else
          [_, port_rcp] = options.configurations['hdfs-site']["dfs.namenode.rpc-address.#{options.nameservice}.#{options.hostname}"].split ':'
          [_, port_http] = options.configurations['hdfs-site']["dfs.namenode.http-address.#{options.nameservice}.#{options.hostname}"].split ':'
          [_, port_https] = options.configurations['hdfs-site']["dfs.namenode.https-address.#{options.nameservice}.#{options.hostname}"].split ':'
          rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: port_rcp, protocol: 'tcp', state: 'NEW', comment: "HDFS NN IPC" }
          rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: port_http, protocol: 'tcp', state: 'NEW', comment: "HDFS NN HTTP" }
          rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: port_https, protocol: 'tcp', state: 'NEW', comment: "HDFS NN HTTPS" }

      if options.hdfs_zkfc
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['hdfs-site']['dfs.ha.zkfc.port'], protocol: 'tcp', state: 'NEW', comment: "ZKFC IPC" }

      if options.mapreduce

| Service    | Port        | Proto | Parameter                                   |
|------------|-------------|-------|---------------------------------------------|
| mapreduce  | 59100-59200 | http  | yarn.app.mapreduce.am.job.client.port-range |


IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

        jobclient = options.configurations['mapred-site']['yarn.app.mapreduce.am.job.client.port-range']
        jobclient = jobclient.replace '-', ':'
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: jobclient, protocol: 'tcp', state: 'NEW', comment: "MapRed Client Range" }

      if options.mapred_jhs
    
| Service    | Port  | Proto | Parameter                                 |
|------------|-------|-------|-------------------------------------------|
| jobhistory | 10020 | tcp   | mapreduce.jobhistory.address              |
| jobhistory | 19888 | http  | mapreduce.jobhistory.webapp.address       |
| jobhistory | 19889 | https | mapreduce.jobhistory.webapp.https.address |
| jobhistory | 13562 | tcp   | mapreduce.shuffle.port                    |
| jobhistory | 10033 | tcp   | mapreduce.jobhistory.admin.address        |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

        jhs_shuffle_port = options.configurations['mapred-site']['mapreduce.shuffle.port']
        jhs_port = options.configurations['mapred-site']['mapreduce.jobhistory.address'].split(':')[1]
        jhs_admin_port = options.configurations['mapred-site']['mapreduce.jobhistory.admin.address'].split(':')[1]
        rules.push [
          { chain: 'INPUT', jump: 'ACCEPT', dport: jhs_port, protocol: 'tcp', state: 'NEW', comment: "MapRed JHS Server" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: jhs_shuffle_port, protocol: 'tcp', state: 'NEW', comment: "MapRed JHS Shuffle" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: jhs_admin_port, protocol: 'tcp', state: 'NEW', comment: "MapRed JHS Admin Server" }
        ]...
        if options.configurations['mapred-site']['mapreduce.jobhistory.http.policy'] is 'HTTP_ONLY'
          jhs_webapp_port = options.configurations['mapred-site']['mapreduce.jobhistory.webapp.address'].split(':')[1]
          rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: jhs_webapp_port, protocol: 'tcp', state: 'NEW', comment: "MapRed JHS WebApp" }
        else
          jhs_webapp_https_port = options.configurations['mapred-site']['mapreduce.jobhistory.webapp.https.address'].split(':')[1]
          rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: jhs_webapp_https_port, protocol: 'tcp', state: 'NEW', comment: "MapRed JHS WebApp" }
    
      if options.yarn_nm

| Service    | Port | Proto  | Parameter                          |
|------------|------|--------|------------------------------------|
| nodemanager | 45454 | tcp  | yarn.nodemanager.address           | x
| nodemanager | 8040  | tcp  | yarn.nodemanager.localizer.address |
| nodemanager | 8042  | tcp  | yarn.nodemanager.webapp.address    |
| nodemanager | 8044  | tcp  | yarn.nodemanager.webapp.https.address    |
| resourcemanager | 8025  | tcp    | yarn.resourcemanager.resource-tracker.address | x
| resourcemanager | 8050  | tcp    | yarn.resourcemanager.address                  | x
| scheduler       | 8030  | tcp    | yarn.resourcemanager.scheduler.address        | x
| resourcemanager | 8088  | http   | yarn.resourcemanager.webapp.address           | x
| resourcemanager | 8090  | https  | yarn.resourcemanager.webapp.https.address     |
| resourcemanager | 8141  | tcp    | yarn.resourcemanager.admin.address            | x
| timeline  | 10200  | tcp/http  | yarn.timeline-service.address              |
| timeline  | 8188   | tcp/http  | yarn.timeline-service.webapp.address       |
| timeline  | 8190   | tcp/https | yarn.timeline-service.webapp.https.address |

        nm_port = options.configurations['yarn-site']['yarn.nodemanager.address'].split(':')[1]
        nm_localizer_port = options.configurations['yarn-site']['yarn.nodemanager.localizer.address'].split(':')[1]
        nm_webapp_port = options.configurations['yarn-site']['yarn.nodemanager.webapp.address'].split(':')[1]
        nm_webapp_https_port = options.configurations['yarn-site']['yarn.nodemanager.webapp.https.address'].split(':')[1]
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: nm_port, protocol: 'tcp', state: 'NEW', comment: "YARN NM Container" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: nm_localizer_port, protocol: 'tcp', state: 'NEW', comment: "YARN NM Localizer" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: nm_webapp_port, protocol: 'tcp', state: 'NEW', comment: "YARN NM Web UI" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: nm_webapp_https_port, protocol: 'tcp', state: 'NEW', comment: "YARN NM Web Secured UI" }
        
        if options.tez
          rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['tez-site']['tez.am.client.am.port-range'].replace('-',':'), protocol: 'tcp', state: 'NEW', comment: "Tez AM Range" }

      if options.yarn_rm

        id = if options.configurations['yarn-site']['yarn.resourcemanager.ha.enabled'] is 'true' then ".#{options.hostname}" else ''
        # Application
        rpc_port = options.configurations['yarn-site']["yarn.resourcemanager.address#{id}"].split(':')[1]
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: rpc_port, protocol: 'tcp', state: 'NEW', comment: "YARN RM Application Submissions" }
        # Scheduler
        s_port = options.configurations['yarn-site']["yarn.resourcemanager.scheduler.address#{id}"].split(':')[1]
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: s_port, protocol: 'tcp', state: 'NEW', comment: "YARN Scheduler" }
        # RM Scheduler
        admin_port = options.configurations['yarn-site']["yarn.resourcemanager.admin.address#{id}"].split(':')[1]
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: admin_port, protocol: 'tcp', state: 'NEW', comment: "YARN RM Scheduler" }
        # HTTP
        if options.configurations['yarn-site']['yarn.http.policy'] in ['HTTP_ONLY', 'HTTP_AND_HTTPS']
          http_port = options.configurations['yarn-site']["yarn.resourcemanager.webapp.address#{id}"].split(':')[1]
          rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: http_port, protocol: 'tcp', state: 'NEW', comment: "YARN RM Web UI" }
        # HTTPS
        if options.configurations['yarn-site']['yarn.http.policy'] in ['HTTPS_ONLY', 'HTTP_AND_HTTPS']
          https_port = options.configurations['yarn-site']["yarn.resourcemanager.webapp.https.address#{id}"].split(':')[1]
          rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: https_port, protocol: 'tcp', state: 'NEW', comment: "YARN RM Web UI" }
        # Resource Tracker
        rt_port = options.configurations['yarn-site']["yarn.resourcemanager.resource-tracker.address#{id}"].split(':')[1]
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: rt_port, protocol: 'tcp', state: 'NEW', comment: "YARN RM Application Submissions" }

      if options.yarn_ts

        [_, rpc_port] = options.configurations['yarn-site']['yarn.timeline-service.address'].split ':'
        [_, http_port] = options.configurations['yarn-site']['yarn.timeline-service.webapp.address'].split ':'
        [_, https_port] = options.configurations['yarn-site']['yarn.timeline-service.webapp.https.address'].split ':'
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: rpc_port, protocol: 'tcp', state: 'NEW', comment: "Yarn Timeserver RPC" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: http_port, protocol: 'tcp', state: 'NEW', comment: "Yarn Timeserver HTTP" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: https_port, protocol: 'tcp', state: 'NEW', comment: "Yarn Timeserver HTTPS" }

      if options.ambari_infra
        
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['infra-solr-env']['infra_solr_port'], protocol: 'tcp', state: 'NEW', comment: "Ambari Infra Logsearch" }

| Service    | Port  | Proto  | Parameter          |
|------------|-------|--------|--------------------|
| Grafana UI | 3000  | https  | server.http_port  |

      if options.ambari_grafana

        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['ams-grafana-ini']['port'] , protocol: 'tcp', state: 'NEW', comment: "Grafana Port ui" }
      
      if options.logsearch_server
        
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['logsearch-env']['logsearch_ui_port'], protocol: 'tcp', state: 'NEW', comment: "Ambari Logsearch ui" }

      if options.hbase_master


| Service             | Port  | Proto | Info                   |
|---------------------|-------|-------|------------------------|
| HBase Master        | 60000 | http  | hbase.master.port      |
| HMaster Info Web UI | 60010 | http  | hbase.master.info.port |

        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['hbase-site']['hbase.master.port'], protocol: 'tcp', state: 'NEW', comment: "HBase Master" }
        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['hbase-site']['hbase.master.info.port'], protocol: 'tcp', state: 'NEW', comment: "HMaster Info Web UI" }

      if options.hbase_regionserver

| Service                      | Port  | Proto | Info                         |
|------------------------------|-------|-------|------------------------------|
| HBase Region Server          | 60020 | http  | hbase.regionserver.port      |
| HMaster Region Server Web UI | 60030 | http  | hbase.regionserver.info.port |

        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['hbase-site']['hbase.regionserver.port'], protocol: 'tcp', state: 'NEW', comment: "HBase RegionServer" }
        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['hbase-site']['hbase.regionserver.info.port'], protocol: 'tcp', state: 'NEW', comment: "HBase RegionServer Info Web UI" }

      if options.hbase_rest

| Service                    | Port  | Proto | Info                   |
|----------------------------|-------|-------|------------------------|
| HBase REST Server          | 60080 | http  | hbase.rest.port        |
| HBase REST Server Web UI   | 60085 | http  | hbase.rest.info.port   |

        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['hbase-site']['hbase.rest.port'], protocol: 'tcp', state: 'NEW', comment: "HBase Master" }
        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['hbase-site']['hbase.rest.info.port'], protocol: 'tcp', state: 'NEW', comment: "HMaster Info Web UI" }
      
      if options.oozie_server

| Service | Port  | Proto | Info                      |
|---------|-------|-------|---------------------------|
| oozie   | 11443 | http  | Oozie HTTP secure server  |
| oozie   | 11001 | http  | Oozie Admin server        |


        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: url.parse(options.configurations['oozie-site']['oozie.base.url']).port, protocol: 'tcp', state: 'NEW', comment: "Oozie HTTP/HTTPS Server" }
        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: '11001', protocol: 'tcp', state: 'NEW', comment: "Oozie HTTP Server" }


      @call if: options.hbase_rest, ->
        @krb5.addprinc options.krb5.admin,
          header: 'Kerberos'
          principal: options.configurations['hbase-site']['hbase.rest.kerberos.principal'].replace '_HOST', options.fqdn
          randkey: true
          keytab: options.configurations['hbase-site']['hbase.rest.keytab.file']
          uid: options.hbase_user.name
          gid: options.hbase_group.name

      if options.smartsense_explorer

        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: 9060, protocol: 'tcp', state: 'NEW', comment: "SMARTSENSE ACTIVITY EXPLORER ui" }

      if options.smartsense_agent
        
        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: 9440, protocol: 'tcp', state: 'NEW', comment: "SmartSense Agent" }
        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: 9441, protocol: 'tcp', state: 'NEW', comment: "SmartSense Agent Secure" }

      if options.zeppelin_master

        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['zeppelin-config']['zeppelin.server.port'], protocol: 'tcp', state: 'NEW', comment: "HTTP ZEPPELIN" }
        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['zeppelin-config']['zeppelin.server.ssl.port'], protocol: 'tcp', state: 'NEW', comment: "HTTPS ZEPPELIN" }

      if options.hive_hcatalog

| Service        | Port  | Proto | Parameter            |
|----------------|-------|-------|----------------------|
| Hive Metastore | 9083  | http  | hive.metastore.uris  |
| Hive Web UI    | 9999  | http  | hive.hwi.listen.port |

        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['hive-site']['hive.metastore.port'], protocol: 'tcp', state: 'NEW', comment: "Hive Metastore" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['hive-site']['hive.hwi.listen.port'], protocol: 'tcp', state: 'NEW', comment: "Hive Web UI" }

      if options.hive_server2

| Service        | Port  | Proto | Parameter            |
|----------------|-------|-------|----------------------|
| Hive Server    | 10001 | tcp   | env[HIVE_PORT]       |


        hive_server_port = if options.configurations['hive-site']['hive.server2.transport.mode'] is 'binary'
        then options.configurations['hive-site']['hive.server2.thrift.port']
        else options.configurations['hive-site']['hive.server2.thrift.http.port']
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: hive_server_port, protocol: 'tcp', state: 'NEW', comment: "Hive Server" }
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.hive_jmx_port , protocol: 'tcp', state: 'NEW', comment: "HiveServer2 JMX" } if options.hive_jmx_port?

  
      if options.hive_webhcat
      
| Service | Port  | Proto | Info                |
|---------|-------|-------|---------------------|
| webhcat | 50111 | http  | WebHCat HTTP server |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['webhcat-site']['templeton.port'], protocol: 'tcp', state: 'NEW', comment: "WebHCat HTTP Server" }

      if options.phoenix_queryserver

| Service             | Port  | Proto  | Parameter                     |
|---------------------|-------|--------|-------------------------------|
| Phoenix QueryServer | 8765  | HTTP   | phoenix.queryserver.http.port |

        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: 8765, protocol: 'tcp', state: 'NEW', comment: "Phoenix QueryServer port" }

      if options.ranger_admin


| Service              | Port  | Proto       | Parameter          |
|----------------------|-------|-------------|--------------------|
| Ranger policymanager | 6080  | http        | port               |
| Ranger policymanager | 6182  | https       | port               |

        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['ranger-admin-site']['ranger.service.http.port'], protocol: 'tcp', state: 'NEW', comment: "Ranger Admin HTTP WEBUI" }
        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['ranger-admin-site']['ranger.service.https.port'], protocol: 'tcp', state: 'NEW', comment: "Ranger Admin HTTPS WEBUI" }

      if options.knox_server
        rules.push  { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['gateway-site']['gateway.port'], protocol: 'tcp', state: 'NEW', comment: "Knox Gateway" }
    
      @tools.iptables
        header: 'Iptables'
        rules: rules
        if: options.iptables

## Yarn Cgroups with CGCONFIG

      @call
        header: 'Cgroups Auto'
        if: -> (options.configurations['yarn-site']['yarn.nodemanager.linux-container-executor.cgroups.mount'] is 'true') and options.yarn_nm
      , ->
        return unless options.yarn_nm
        @service
          name: 'libcgroup'
        # .execute
        #   cmd: 'mount -t cgroup -o cpu cpu /cgroup'
        #   code_skipped: 32
        @system.mkdir
          target: "#{options.configurations['yarn-site']['yarn.nodemanager.linux-container-executor.cgroups.mount-path']}/cpu"
          mode: 0o1777
          parent: true
      @call
        header: 'Cgroups Manual'
        unless: -> (options.configurations['yarn-site']['yarn.nodemanager.linux-container-executor.cgroups.mount'] is 'true')
      , ->
        return unless options.yarn_nm
        @system.cgroups
          target: '/etc/cgconfig.d/yarn.cgconfig.conf'
          merge: false
          groups: options.cgroup
        , (err, data) ->
          options.configurations['yarn-site']['yarn.nodemanager.linux-container-executor.cgroups.mount-path'] = data.cgroups.mount
        @call ->
          # migration: wdavidw 170827, using store is a bad, very bad idea, ensure it works in the mean time
          # lucasbak 180127 not using store anymore
          throw Error 'YARN NM Ambari Cgroup is undefined' unless options.configurations['yarn-site']['yarn.nodemanager.linux-container-executor.cgroups.mount-path']
          # @hconfigure
          #   header: 'YARN Site'
          #   target: "#{options.conf_dir}/yarn-site.xml"
          #   properties: options.configurations['yarn-site']
          #   merge: true
          #   backup: true
        @service.restart
          name: 'cgconfig'
          if: -> @status -2

      # todo Version Ambari Check
      if options.ambari_infra
        @file.download
          header: 'Ambari Infra params.py'
          source: "#{__dirname}/../ambari_infra/resources/params.py"
          target: "/var/lib/ambari-agent/cache/common-services/AMBARI_INFRA/0.1.0/package/scripts/params.py"
          local: true
          mode: 0o755
        @file.download
          header: 'Ambari Infra setup_infra_solr.py'
          source: "#{__dirname}/../ambari_infra/resources/setup_infra_solr.py"
          target: "/var/lib/ambari-agent/cache/common-services/AMBARI_INFRA/0.1.0/package/scripts/setup_infra_solr.py"
          local: true
          mode: 0o755
        @file.download
          header: 'Logsearch Server params.py'
          source: "#{__dirname}/../logsearch/resources/params.py"
          target: "/var/lib/ambari-agent/cache/common-services/LOGSEARCH/0.5.0/package/scripts/params.py"
          local: true
          mode: 0o755
        @file.download
          header: 'SolrCloudUtil.py'
          source: "#{__dirname}/../server/resources/solr_cloud_util.py"
          target: "/usr/lib/ambari-agent/lib/resource_management/libraries/functions/solr_cloud_util.py"
          local: true
          mode: 0o755

## Dependencies

    path = require 'path'
    misc = require 'nikita/lib/misc'
    url = require 'url'
