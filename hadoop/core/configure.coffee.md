
# Hadoop Core Configuration

    module.exports = (service) ->
      options = service.options
      options.fqdn ?= service.node.fqdn

## Identities

      options.group = merge {}, service.deps.hdfs[0].options.hdfs.group, options.group
      options.user = merge {}, service.deps.hdfs[0].options.hdfs.user, options.user
      options.hadoop_group = merge {}, service.deps.hdfs[0].options.hadoop_group, options.hadoop_group
      options.hdfs = merge {}, service.deps.hdfs[0].options.hdfs
      options.yarn = merge {}, service.deps.yarn[0].options.yarn

## Validation

HDFS does not accept underscore "_" inside the hostname or it fails on startup 
with the log message:

```
17/05/15 00:31:54 WARN hdfs.DFSUtil: Exception in creating socket address master_01.ambari.ryba:8020
java.lang.IllegalArgumentException: Does not contain a valid host:port authority: master_01.ambari.ryba:8020
```

      throw Error "Invalid Hostname: #{service.node.fqdn} should not contain \"_\"" if /_/.test service.node.fqdn

## Environment

      # Layout
      options.conf_dir ?= service.deps.hdfs[0].options.conf_dir
      # options.hadoop_lib_home ?= '/usr/hdp/current/hadoop-client/lib' # refered by oozie-env.sh, now hardcoded
      # options.hdfs.log_dir ?= service.deps.hdfs[0].options.log_dir
      # options.hdfs.pid_dir ?= service.deps.hdfs[0].options.pid_dir
      # options.hdfs.secure_dn_pid_dir ?= service.deps.hdfs[0].options.secure_dn_pid_dir
      # options.hdfs.secure_dn_user ?= service.deps.hdfs[0].options.secure_dn_user

## HA Configuration

      # options.nameservice ?= null
      # throw Error "Invalid Service Name" unless options.nameservice

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      # Spnego
      options.spnego ?= {}
      options.spnego.principal ?= "HTTP/#{service.node.fqdn}@#{options.krb5.realm}"
      options.spnego.keytab ?= '/etc/security/keytabs/spnego.service.keytab'
      
## Configuration

      options.core_site ?= {}
      # Set the authentication for the cluster. Valid values are: simple or kerberos
      options.core_site['hadoop.security.authentication'] ?= 'kerberos'
      # Enable authorization for different protocols.
      options.core_site['hadoop.security.authorization'] ?= 'true'
      # A comma-separated list of protection values for secured sasl
      # connections. Possible values are authentication, integrity and privacy.
      # authentication means authentication only and no integrity or privacy;
      # integrity implies authentication and integrity are enabled; and privacy
      # implies all of authentication, integrity and privacy are enabled.
      # hadoop.security.saslproperties.resolver.class can be used to override
      # the hadoop.rpc.protection for a connection at the server side.
      options.core_site['hadoop.rpc.protection'] ?= 'authentication'
      # Default group mapping
      options.core_site['hadoop.security.group.mapping'] ?= 'org.apache.hadoop.security.JniBasedUnixGroupsMappingWithFallback'
      # Get ZooKeeper Quorum
      zookeeper_quorum = service.deps.zookeeper_server
      .filter (srv) -> srv.options.config['peerType'] is 'participant'
      .map (srv)-> "#{srv.node.fqdn}:#{srv.options.config['clientPort']}"
      .join(',')
      options.core_site['ha.zookeeper.quorum'] ?= zookeeper_quorum

## Topology

      # Script imported from http://ofirm.wordpress.com/2014/01/09/exploring-the-hadoop-network-topology/
      options.core_site['net.topology.script.file.name'] ?= "#{options.conf_dir}/rack_topology.sh"
      options.topology = service.instances.filter (instance) ->
        instance.node.services.some (service) ->
          service.module in ['ryba-ambari-takeover/hadoop/hdfs_dn', 'ryba-ambari-takeover/hadoop/yarn_nm']
      .map (instance) ->
        throw Error "Required Node Option: ip for node #{JSON.stringify instance.node.id}" unless instance.node.ip
        id: instance.node.id, ip: instance.node.ip, rack: instance.node.rack
      # Validate rack
      if options.topology.some( (node) -> node.rack )
        for node in options.topology
          throw Error "Required Option: rack required in node #{node.id} because at least one rack is defined" unless node.rack
      options.rack_info = (options.topology.filter( (host) -> host.id is service.node.fqdn)[0])?.rack
      
Configuration for HTTP

      options.core_site['hadoop.http.filter.initializers'] ?= 'org.apache.hadoop.security.AuthenticationFilterInitializer'
      options.core_site['hadoop.http.authentication.type'] ?= 'kerberos'
      options.core_site['hadoop.http.authentication.token.validity'] ?= '36000'
      options.core_site['hadoop.http.authentication.signature.secret.file'] ?= '/etc/hadoop/hadoop-http-auth-signature-secret'
      options.core_site['hadoop.http.authentication.simple.anonymous.allowed'] ?= 'false'
      options.core_site['hadoop.http.authentication.kerberos.principal'] ?= "HTTP/_HOST@#{options.krb5.realm}"
      options.core_site['hadoop.http.authentication.kerberos.keytab'] ?= options.spnego.keytab
      # Cluster domain
      unless options.core_site['hadoop.http.authentication.cookie.domain']
        domains = service.deps.hadoop_core.map( (srv) -> srv.node.fqdn.split('.').slice(1).join('.') ).filter( (el, pos, self) -> self.indexOf(el) is pos )
        throw Error "Multiple domains, set 'hadoop.http.authentication.cookie.domain' manually" if domains.length isnt 1
        options.core_site['hadoop.http.authentication.cookie.domain'] = domains[0]

Configuration for auth\_to\_local

The local name will be formulated from exp.
The format for exp is [n:string](regexp)s/pattern/replacement/g.
The integer n indicates how many components the target principal should have. 
If this matches, then a string will be formed from string, substituting the realm 
of the principal for $0 and the n‘th component of the principal for $n. 
If this string matches regexp, then the s//[g] substitution command will be run 
over the string. The optional g will cause the substitution to be global over 
the string, instead of replacing only the first match in the string.
The rule apply with priority order, so we write rules from the most specific to
the most general:
There is 4 identified cases:

*   The principal is a 'sub-service' principal from our internal realm. It replaces with the corresponding service name
*   The principal is from our internal realm. We apply DEFAULT rule (It takes the first component of the principal as a
    username. Only apply on the internal realm)
*   The principal is NOT from our realm, and would be mapped to an admin user like hdfs. It maps it to 'nobody'
*   The principal is NOT from our internal realm, and do NOT match any admin account.
    It takes the first component of the principal as username.

Notice that the third rule will disallow admin account on multiple clusters.
the property must be overriden in a config file to permit it. 

      esc_realm = quote options.krb5.realm
      options.core_site['hadoop.security.auth_to_local'] ?= """

            RULE:[2:$1@$0]([rn]m@#{esc_realm})s/.*/yarn/
            RULE:[2:$1@$0](jhs@#{esc_realm})s/.*/mapred/
            RULE:[2:$1@$0]([nd]n@#{esc_realm})s/.*/hdfs/
            RULE:[2:$1@$0](hm@#{esc_realm})s/.*/hbase/
            RULE:[2:$1@$0](rs@#{esc_realm})s/.*/hbase/
            RULE:[2:$1@$0](opentsdb@#{esc_realm})s/.*/hbase/
            DEFAULT
            RULE:[1:$1](yarn|mapred|hdfs|hive|hbase|oozie)s/.*/nobody/
            RULE:[2:$1](yarn|mapred|hdfs|hive|hbase|oozie)s/.*/nobody/
            RULE:[1:$1]
            RULE:[2:$1]

      """


Configuration for proxy users

      options.core_site['hadoop.proxyuser.HTTP.hosts'] ?= '*'
      options.core_site['hadoop.proxyuser.HTTP.groups'] ?= '*'


# SSL

Hortonworks mentions 2 strategies to [configure SSL][hdp_ssl], the first one
involves Self-Signed Certificate while the second one use a Certificate
Authority.

For now, only the second approach has been tested and is supported. For this, 
you are responsible for creating your own Private Key and Certificate Authority
(see bellow instructions) and for declaring with the 
"hdp.private\_key\_location" and "hdp.cacert\_location" property.

It is also recommendate to configure the 
"hdp.core\_site['ssl.server.truststore.password']" and 
"hdp.core\_site['ssl.server.keystore.password']" passwords or an error will be 
thrown.

Here's how to generate your own Private Key and Certificate Authority:

```
openssl genrsa -out hadoop.key 2048
openssl req -x509 -new -key hadoop.key -days 300 -out hadoop.pem -subj "/C=FR/ST=IDF/L=Paris/O=Adaltas/CN=adaltas.com/emailAddress=david@adaltas.com"
```

You can see the content of the root CA certificate with the command:

```
openssl x509 -text -noout -in hadoop.pem
```

You can list the content of the keystore with the command:

```
keytool -list -v -keystore truststore
keytool -list -v -keystore keystore -alias hadoop
```

[hdp_ssl]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1-latest/bk_reference/content/ch_wire-https.html

      options.ssl = merge {}, service.deps.ssl?.options, options.ssl
      options.ssl.enabled ?= !!service.deps.ssl
      options.ssl.conf_dir ?= '/etc/security/serverKeys'
      if options.ssl.enabled
        options.ssl_client ?= {}
        options.ssl_server ?= {}
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert
        # SSL for HTTPS connection and RPC Encryption
        options.core_site['hadoop.ssl.require.client.cert'] ?= 'false'
        options.core_site['hadoop.ssl.hostname.verifier'] ?= 'DEFAULT'
        options.core_site['hadoop.ssl.keystores.factory.class'] ?= 'org.apache.hadoop.security.ssl.FileBasedKeyStoresFactory'
        options.core_site['hadoop.ssl.server.conf'] ?= 'ssl-server.xml'
        options.core_site['hadoop.ssl.client.conf'] ?= 'ssl-client.xml'

### SSL Client

The "ssl_client" options store information used to write the "ssl-client.xml"
file in the Hadoop XML configuration format. Some information are derived the 
the truststore options exported from the SSL service and merged above:

```json
{ password: 'Truststore123-',
  target: '/etc/security/jks/truststore.jks',
  caname: 'ryba_root_ca' }
```

        options.ssl_client['ssl.client.truststore.password'] ?= options.ssl.truststore.password
        throw Error "Required Option: ssl_client['ssl.client.truststore.password']" unless options.ssl_client['ssl.client.truststore.password']
        options.ssl_client['ssl.client.truststore.location'] ?= "#{options.conf_dir}/truststore"
        options.ssl_client['ssl.client.truststore.type'] ?= 'jks'
        options.ssl_client['ssl.client.keystore.password'] ?= options.ssl.keystore.password
        throw Error "Required Option: ssl_client['ssl.client.keystore.password']" unless options.ssl_client['ssl.client.keystore.password']
        options.ssl_client['ssl.client.keystore.location'] ?= "#{options.conf_dir}/keystore"
        options.ssl_client['ssl.client.keystore.type'] ?= 'jks'
        options.ssl_client['ssl.client.truststore.reload.interval'] ?= '10000'

### SSL Server

The "ssl_server" options store information used to write the "ssl-server.xml"
file in the Hadoop XML configuration format. Some information are derived the 
the keystore options exported from the SSL service and merged above:

```json
{ password: 'Keystore123-',
  keypass: 'Keystore123-',
  target: '/etc/security/jks/keystore.jks',
  name: 'master01',
  caname: 'ryba_root_ca' },
```

        options.ssl_server['ssl.server.keystore.password'] ?= options.ssl.keystore.password
        throw Error "Required Option: ssl_server['ssl.server.keystore.password']" unless options.ssl_server['ssl.server.keystore.password']
        options.ssl_server['ssl.server.keystore.location'] ?= "#{options.conf_dir}/keystore"
        options.ssl_server['ssl.server.keystore.type'] ?= 'jks'
        options.ssl_server['ssl.server.keystore.keypassword'] ?= options.ssl.keystore.keypass
        throw Error "Required Option: ssl_server['ssl.server.keystore.keypassword']" unless options.ssl_server['ssl.server.keystore.keypassword']
        options.ssl_server['ssl.server.truststore.location'] ?= "#{options.conf_dir}/truststore"
        options.ssl_server['ssl.server.truststore.password'] ?= options.ssl_client['ssl.client.truststore.password']
        options.ssl_server['ssl.server.truststore.type'] ?= 'jks'
        options.ssl_server['ssl.server.truststore.reload.interval'] ?= '10000'

## Metrics

Configuration of Hadoop metrics system. 

The File sink is activated by default. The Ganglia and Graphite sinks are
automatically activated if the "ryba/retired/ganglia/collecto" and
"ryba/graphite/collector" are respectively registered on one of the nodes of the
cluster. You can disable any of those sinks by setting its class to false, here
how:

```json
{ "ryba": { "metrics": 
  "*.sink.file.class": false,
  "*.sink.ganglia.class": false, 
  "*.sink.graphite.class": false
 } }
```

Metric prefix can be defined globally with the usage of glob expression or per
context. Here's an exemple:

```json
{ "metrics":
    config:
      "*.sink.*.metrics_prefix": "default",
      "*.sink.file.metrics_prefix": "file_prefix", 
      "namenode.sink.ganglia.metrics_prefix": "master_prefix",
      "resourcemanager.sink.ganglia.metrics_prefix": "master_prefix"
}
```

Syntax is "[prefix].[source|sink].[instance].[options]".  According to the
source code, the list of supported prefixes is: "namenode", "resourcemanager",
"datanode", "nodemanager", "maptask", "reducetask", "journalnode",
"historyserver", "nimbus", "supervisor".

      options.metrics = merge {}, service.deps.metrics?.options, options.metrics

      # Hadoop metrics
      options.metrics ?= {}
      options.metrics.sinks ?= {}
      options.metrics.sinks.file_enabled ?= true
      options.metrics.sinks.ganglia_enabled ?= false
      options.metrics.sinks.graphite_enabled ?= false
      # default sampling period, in seconds
      options.metrics.config ?= {}
      options.metrics.config ?= {}
      options.metrics.config['*.period'] ?= '60'
      # File sink
      if options.metrics.sinks.file_enabled
        options.metrics.config["*.sink.file.#{k}"] ?= v for k, v of service.deps.metrics.options.sinks.file.config if service.deps.metrics?.options?.sinks?.file_enabled
        options.metrics.config['nodemanager.sink.file.filename'] ?= 'nodemanager-metrics.out'
        options.metrics.config['mrappmaster.sink.file.filename'] ?= 'mrappmaster-metrics.out'
        options.metrics.config['jobhistoryserver.sink.file.filename'] ?= 'jobhistoryserver-metrics.out'
      # Ganglia sink, accepted properties are "servers" and "supportsparse"
      if options.metrics.sinks.ganglia_enabled
        options.metrics.config["*.sink.ganglia.#{k}"] ?= v for k, v of options.sinks.ganglia.config if service.deps.metrics?.options?.sinks?.ganglia_enabled
      # Graphite Sink
      if options.metrics.sinks.graphite_enabled
        throw Error 'Unvalid metrics sink, please provide ryba.metrics.sinks.graphite.config.server_host and server_port' unless options.metrics.sinks.graphite.config.server_host? and options.metrics.sinks.graphite.config.server_port?
        options.metrics.config["*.sink.graphite.#{k}"] ?= v for k, v of service.deps.metrics.options.sinks.graphite.config if service.deps.metrics?.options?.sinks?.graphite_enabled

## Log4j

      options.log4j = merge {}, service.deps.log4j?.options, options.log4j
      options.log4j.hadoop_root_logger ?= 'INFO,RFA'
      options.log4j.hadoop_security_logger ?= 'INFO,RFAS'
      options.log4j.hadoop_audit_logger ?= 'INFO,RFAAUDIT'

      options.core_site['io.serializations'] ?= 'org.apache.hadoop.io.serializer.WritableSerialization'
      options.core_site['ipc.client.connect.max.retries'] ?= '50'
      options.core_site['ipc.client.connection.maxidletime'] ?= '30000'
      options.core_site['ipc.client.idlethreshold'] ?= '8000'
      options.core_site['mapreduce.jobtracker.webinterface.trusted'] ?= 'false'


## HDFS Configuration

      enrich_config = (source, target) ->
        for k, v of source
          target[k] ?= v

      for srv in service.deps.hdfs
        # Register Java Options
        srv.options ?= {}
        srv.options.hadoop_opts ?= options.hadoop_opts
        srv.options.hadoop_classpath ?= options.hadoop_classpath
        srv.options.hadoop_heap ?= options.hadoop_heap
        srv.options.hadoop_client_opts ?= options.hadoop_client_opts
        #topology
        srv.options.topology ?= options.topology
        # *-site.xml
        srv.options.configurations ?= {}
        srv.options.configurations['core-site'] ?= {}
        srv.options.configurations['hdfs-site'] ?= {}
        srv.options.configurations['yarn-site'] ?= {}
        srv.options.configurations['mapred-site'] ?= {}
        srv.options.configurations['ssl-server'] ?= {}
        srv.options.configurations['ssl-client'] ?= {}
        # enrich hdfs service
        enrich_config options.core_site, srv.options.configurations['core-site']
        enrich_config options.hdfs_site, srv.options.configurations['hdfs-site']
        enrich_config options.yarn_site, srv.options.configurations['yarn-site']
        enrich_config options.mapred_site, srv.options.configurations['mapred-site']
        enrich_config options.ssl_server, srv.options.configurations['ssl-server']
        enrich_config options.ssl_client, srv.options.configurations['ssl-client']

## Env
        # enrich hdfs hadoop-env service
        srv.options.configurations['hadoop-env'] ?= {}
        srv.options.configurations['hadoop-env']['HADOOP_ROOT_LOGGER'] ?= options.log4j.root_logger
        srv.options.configurations['hadoop-env']['HADOOP_SECURITY_LOGGER'] ?= options.log4j.security_logger
        srv.options.configurations['hadoop-env']['HDFS_AUDIT_LOGGER'] ?= options.log4j.audit_logger
        srv.options.configurations['hadoop-env']['HADOOP_CLIENT_OPTS'] ?= options.hadoop_client_opts
        srv.options.configurations['hadoop-env']['HADOOP_OPTS'] ?= options.hadoop_opts
        srv.options.configurations['hadoop-env']['hadoop_root_logger'] ?= options.log4j.root_logger
        srv.options.configurations['hadoop-env']['hadoop_root_logger'] ?= options.log4j.root_logger
        # enrich hdfs yarn-env service
        srv.options.configurations['yarn-env'] ?= {}
        srv.options.configurations['yarn-env']['YARN_ROOT_LOGGER'] ?= options.log4j.root_logger
        srv.options.configurations['yarn-env']['YARN_OPTS'] ?= options.java_opts
      
## Metrics Properties

        srv.options.configurations['hadoop-metrics-properties'] ?= {}
        enrich_config options.metrics.config, srv.options.configurations['hadoop-metrics-properties'] if service.deps.metrics?

## Log4j Properties

      srv.options.hdfs_log4j ?= {}
      enrich_config options.log4j.properties, options.hdfs_log4j if service.deps.log4j?

## Ambari Server

      service.deps.ambari_server.options.identities ?= {}
      service.deps.ambari_server.options.identities ?= {}
      service.deps.ambari_server.options.identities['spnego'] ?= {}
      service.deps.ambari_server.options.identities['spnego']['principal'] ?= {}
      service.deps.ambari_server.options.identities['spnego']['principal']['configuration'] ?= ''
      service.deps.ambari_server.options.identities['spnego']['principal']['type'] ?= 'service'
      service.deps.ambari_server.options.identities['spnego']['principal']['local_username'] ?= null
      service.deps.ambari_server.options.identities['spnego']['principal']['value'] ?= options.core_site['hadoop.http.authentication.kerberos.principal']
      service.deps.ambari_server.options.identities['spnego']['name'] ?= 'spnego'
      service.deps.ambari_server.options.identities['spnego']['keytab'] ?= {}
      service.deps.ambari_server.options.identities['spnego']['keytab']['owner'] ?= {}
      service.deps.ambari_server.options.identities['spnego']['keytab']['owner']['access'] ?= 'r' 
      service.deps.ambari_server.options.identities['spnego']['keytab']['owner']['name'] ?= '${cluster-env/spnego}' 
      service.deps.ambari_server.options.identities['spnego']['keytab']['group'] ?= {}
      service.deps.ambari_server.options.identities['spnego']['keytab']['group']['access'] ?= 'r'
      service.deps.ambari_server.options.identities['spnego']['keytab']['group']['name'] ?= '${cluster-env/user_group}'
      service.deps.ambari_server.options.identities['spnego']['keytab']['file'] ?= options.core_site['hadoop.http.authentication.kerberos.keytab']
      service.deps.ambari_server.options.identities['spnego']['keytab']['configuration'] ?= null

## Ambari REST API

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.stack_name = service.deps.ambari_server.options.stack_name
      options.stack_version = service.deps.ambari_server.options.stack_version
      options.takeover = service.deps.ambari_server.options.takeover
      options.baremetal = service.deps.ambari_server.options.baremetal

## Dependencies

    path = require 'path'
    quote = require 'regexp-quote'
    {merge} = require 'nikita/lib/misc'
