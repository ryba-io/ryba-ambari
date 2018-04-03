
# MapReduce JobHistoryServer Wait

    module.exports = header: 'Mapreduce Ambari JHS Wait', handler: (options) ->

## TCP

      @connection.wait
        header: 'TCP'
        servers: options.tcp

## HTTP

      @connection.wait
        header: 'HTTP'
        servers: options.webapp
