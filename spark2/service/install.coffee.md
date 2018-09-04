
# Apache Spark Install

[Spark Installation][Spark-install] following hortonworks guidelines to install
Spark requires HDFS and Yarn. Install spark in Yarn cluster mode.

Resources:

[Tips and Tricks from Altic Scale][https://www.altiscale.com/blog/tips-and-tricks-for-running-spark-on-hadoop-part-2-2/)   

    module.exports = header: 'Ambari Spark2 Client Install', handler: ({options}) ->

## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'


## Upload Default Configuration

      # @call -> console.log options.configurations
      @ambari.configs.default
        header: 'SPARK2 Configuration'
        url: options.ambari_url
        if: options.post_component and options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        stack_name: options.stack_name
        stack_version: options.stack_version
        discover: true
        configurations: options.configurations
        target_services: 'SPARK2'

## Kerberos Descriptor Artifact

      @ambari.kerberos.descriptor.update
        header: 'Kerberos Artifact'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        stack_name: options.stack_name
        stack_version: options.stack_version
        cluster_name: options.cluster_name
        service: 'SPARK2'
        identities: options.identities['spark2']
        source: 'COMPOSITE'

      @krb5.addprinc options.krb5.admin,
        header: 'Create Principal'
        principal: options.krb5.principal
        password: options.krb5.password

      @krb5.ktutil.add options.krb5.admin,
        header: 'SPARK2 Headless keytab'
        principal: options.krb5.principal
        password: options.krb5.password
        keytab: options.krb5.keytab
        kadmin_server: options.krb5.admin.admin_server
        mode: 0o0640
        uid: options.user.name
        gid: options.hadoop_group.name 

## Add SPARK2 Service

      @ambari.services.add
        header: 'SPARK2 Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'SPARK2'

      @ambari.services.wait
        header: 'SPARK2 Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'SPARK2'

      @ambari.services.component_add
        header: 'SPARK2 Add'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'SPARK2_CLIENT'
        service_name: 'SPARK2'

      @ambari.services.component_add
        header: 'SPARK2_JOBHISTORYSERVER Add'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'SPARK2_JOBHISTORYSERVER'
        service_name: 'SPARK2'

## Install SPARK2 CLIENT Component

      for host in options.client_hosts
        @ambari.hosts.component_add
          header: 'SPARK2_CLIENT Host Add'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'SPARK2_CLIENT'
          hostname: host

      for host in options.history_hosts
        @ambari.hosts.component_add
          header: 'SPARK2_JOBHISTORYSERVER Host Add'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'SPARK2_JOBHISTORYSERVER'
          hostname: host

## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
    quote = require 'regexp-quote'
    string = require 'nikita/lib/misc/string'

[spark-conf]:https://spark.apache.org/docs/latest/configuration.html
