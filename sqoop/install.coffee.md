
# Sqoop Install

The only declared dependency is MySQL Client which install the MySQL JDBC
driver used by Sqoop.

    module.exports = header: 'Ambari Sqoop Install', handler: (options) ->

## Registry

      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"

## Upload Configurations

Environment passed to Hadoop.

      @ambari.configs.update
        if: options.post_component and options.takeover
        header: 'upload sqoop-site'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'sqoop-site'
        cluster_name: options.cluster_name
        properties: options.configurations['sqoop-site']


      @ambari.configs.update
        url: options.ambari_url
        if: options.post_component and options.takeover
        username: 'admin'
        merge: true
        password: options.ambari_admin_password
        config_type: 'sqoop-env'
        cluster_name: options.cluster_name
        properties: options.configurations['sqoop-env']

## Add SQOOP Service

      @ambari.services.add
        header: 'SQOOP Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'SQOOP'

      @ambari.services.wait
        header: 'SQOOP Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'SQOOP'

      @ambari.services.component_add
        header: 'SQOOP Add'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'SQOOP'
        service_name: 'SQOOP'

## Install Component

      @ambari.hosts.component_add
        header: 'SQOOP Host Add'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'SQOOP'
        hostname: options.fqdn

      @ambari.hosts.component_wait
        header: 'SQOOP Wait'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'SQOOP'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'SQOOP Install'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'SQOOP'
        hostname: options.fqdn

## Check

Make sure the sqoop client is available on this server, using the [HDP validation
command][validate].

      @system.execute
        header: 'Check Version'
        cmd: "sqoop version | grep 'Sqoop [0-9].*'"

## Mysql Connector

MySQL is by default usable by Sqoop. The driver installed after running the
"masson/commons/mysql/client" is copied into the Sqoop library folder.


      # @system.copy
      #   source: '/usr/share/java/mysql-connector-java.jar'
      #   target: '/usr/hdp/current/sqoop-client/lib/'
      # , next
      @system.link
        header: 'MySQL Connector'
        source: '/usr/share/java/mysql-connector-java.jar'
        target: '/usr/hdp/current/sqoop-client/lib/mysql-connector-java.jar'

## Dependencies

    path = require 'path'

[install]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.9.1/bk_installing_manually_book/content/rpm-chap10-1.html
[validate]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.9.1/bk_installing_manually_book/content/rpm-chap10-4.html
