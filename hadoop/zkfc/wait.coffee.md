
# Hadoop ZKFC Wait

    module.exports = header: 'HDFS ZKFC Ambari Wait', handler: (options) ->

      @connection.wait
        servers: options.wait
