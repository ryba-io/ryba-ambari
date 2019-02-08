
# Phoenix Configuration

    module.exports = (service) ->
      {options, deps, nodes} = service

A Phoenix Client must have one of instance HBase Master, HBase RegionServer or
HBase Client.

      has_hbase = deps.hbase_master_local or deps.hbase_regionserver_local or deps.hbase_client_local
      throw Error "Invalid Configuration: Phoenix Client without HBase on node #{service.node.id}" unless has_hbase

## Kerberos

      # Kerberos Test Principal
      options.test_krb5_user ?= deps.test_user.options.krb5.user

## Environment

      options.conf_dir  ?= '/etc/hadoop/conf'
      options.hbase_conf_dir ?= '/etc/hbase/conf'
      # Misc
      options.hostname = service.node.hostname

## Configuration

      options.site = merge deps.hbase[0].options.configurations['hbase-site'], options.site
      options.admin = merge deps.hbase[0].options.admin, options.admin
      options.configurations ?= {}
      options.configurations['hbase-site'] ?= {}
      options.configurations['hbase-site']['phoenix.schema.isNamespaceMappingEnabled'] = 'true'
      options.configurations['hbase-site']['phoenix.schema.mapSystemTablesToNamespace'] = 'true'
      options.configurations['hbase-site']['hbase.defaults.for.version.skip'] = 'true'
      options.configurations['hbase-site']['hbase.regionserver.wal.codec'] = 'org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec'
      options.configurations['hbase-site']['hbase.table.sanity.checks'] = 'true'
      options.configurations['hbase-site']['hbase.region.server.rpc.scheduler.factory.class'] = 'org.apache.hadoop.hbase.ipc.PhoenixRpcSchedulerFactory'
      options.configurations['hbase-site']['hbase.rpc.controllerfactory.class'] = 'org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory'
        
## Test

      options.test = merge {}, deps.test_user.options, options.test
      options.test.namespace ?= "ryba_check_client_#{service.node.hostname}"
      options.test.table ?= 'a_table'
      options.hostname = service.node.hostname
      
## Wait

      options.wait_hbase_master = service.deps.hbase_master[0].options.wait
      options.wait_hbase_regionserver = service.deps.hbase_regionserver[0].options.wait

## Dependencies

    string = require 'nikita/lib/misc/string'
    {merge} = require 'nikita/lib/misc'
    appender = require 'ryba/lib/appender'
