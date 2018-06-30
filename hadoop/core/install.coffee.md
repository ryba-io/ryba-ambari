
# Hadoop Core Install

    module.exports = header: 'Hadoop Core Install', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','rack'], 'ryba-ambari-actions/lib/hosts/rack'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      

## Packages

Install the "hadoop-client" and "openssl" packages as well as their
dependecies.

The environment script "hadoop-env.sh" from the HDP companion files is also
uploaded when the package is first installed or upgraded. Be careful, the
original file will be overwritten with and user modifications. A copy will be
made available in the same directory after any modification.

      @call header: 'Packages', ->
        @service
          name: 'openssl-devel'
        @service
          name: 'hadoop-client'
        @hdp_select
          name: 'hadoop-client'


## Keytab Directory

      @system.mkdir
        header: 'Keytabs'
        target: '/etc/security/keytabs'
        uid: 'root'
        gid: 'root' # was hadoop_group.name
        mode: 0o0755

## SPNEGO

Create the SPNEGO service principal in the form of "HTTP/{host}@{realm}" and place its
keytab inside "/etc/security/keytabs/spnego.service.keytab" with ownerships set to "hdfs:hadoop"
and permissions set to "0660". We had to give read/write permission to the group because the
same keytab file is for now shared between hdfs and yarn services.

      @call header: 'SPNEGO', ->
        @krb5.addprinc
          principal: options.spnego.principal
          randkey: true
          keytab: options.spnego.keytab
          uid: 'root'
          gid: options.hadoop_group.name
          mode: 0o660 # need rw access for hadoop and mapred users
        , options.krb5.admin
        @system.execute # Validate keytab access by the hdfs user
          cmd: "su -l #{options.hdfs.user.name} -c \"klist -kt /etc/security/keytabs/spnego.service.keytab\""
          if: -> @status -1

## Web UI

This action follow the ["Authentication for Hadoop HTTP web-consoles"
recommendations](http://hadoop.apache.org/docs/r1.2.1/HttpAuthentication.html).

      @system.execute
        header: 'WebUI'
        cmd: 'dd if=/dev/urandom of=/etc/hadoop/hadoop-http-auth-signature-secret bs=1024 count=1'
        unless_exists: '/etc/hadoop/hadoop-http-auth-signature-secret'

## Topology

Configure the topology script to enable rack awareness to Hadoop.

      @call header: 'Topology', ->
        @file
          target: "#{options.conf_dir}/rack_topology.sh"
          source: "#{__dirname}/../resources/rack_topology.sh"
          local: true
          uid: options.hdfs.user.name
          gid: options.hadoop_group.name
          mode: 0o755
          backup: true
        @file
          target: "#{options.conf_dir}/rack_topology.data"
          content: options.topology
            .map (node) ->
              "#{node.ip}  #{node.rack or ''}"
            .join '\n'
          uid: options.hdfs.user.name
          gid: options.hadoop_group.name
          mode: 0o755
          backup: true
          eof: true

## SSL

      @call header: 'SSL', retry: 0, ->
        @system.mkdir
          target: options.ssl.conf_dir
          gid: options.hadoop_group.name
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.ssl_client['ssl.client.truststore.location']
          storepass: options.ssl_client['ssl.client.truststore.password']
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local
        @java.keystore_add
          keystore: options.ssl_server['ssl.server.truststore.location']
          storepass: options.ssl_server['ssl.server.truststore.password']
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.ssl_server['ssl.server.keystore.location']
          storepass: options.ssl_server['ssl.server.keystore.password']
          # caname: "hadoop_root_ca"
          # cacert: "#{options.ssl.cacert}"
          key: options.ssl.key.source
          cert: options.ssl.cert.source
          keypass: options.ssl_server['ssl.server.keystore.keypassword']
          name: options.ssl.key.name
          local: options.ssl.key.local
        @java.keystore_add
          keystore: options.ssl_server['ssl.server.keystore.location']
          storepass: options.ssl_server['ssl.server.keystore.password']
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local

## Rack info

      @ambari.hosts.rack
        header: "Set rack"
        if: options.rack_info and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        hostname: options.fqdn
        rack_info: options.rack_info
