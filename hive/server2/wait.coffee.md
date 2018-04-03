
# Hive Server2 Wait

Wait for the RPC or HTTP ports depending on the configured transport mode.

    module.exports = header: 'Ambari Hive Server2 Wait', handler: (options) ->

## Thrift TCP/HTTP Port

      @connection.wait
        header: 'Thrift'
        servers: options.thrift
