
# Ambari Logsearch Server Check

      module.exports = header: 'Ambari Logsearch Server Check', handler: ({options}) ->
        @connection.assert
          servers: options.tcp
