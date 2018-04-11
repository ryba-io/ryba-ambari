
# Ambari Logsearch Server Wait

      module.exports = header: 'Ambari Logsearch Server Wait', handler: (options) ->
        
        @connection.wait
          servers: options.tcp
