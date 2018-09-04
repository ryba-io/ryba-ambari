
# Apache Zeeplin WEBUI am manages the executors task.

    module.exports = header: 'Ambari Zeppelin Master Check', handler: ({options}) ->

## Wait

      @connection.wait
        header: 'Wait'
        servers: options.wait.http
        retry: 10
        sleep: 3000
