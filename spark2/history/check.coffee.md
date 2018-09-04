
# Apache Spark Check

Run twice "[Spark Pi][Spark-Pi]" example for validating installation . The configuration is a 10 stages run.
[Spark on YARN][Spark-yarn] cluster can turn into two different mode :  yarn-client mode and yarn-cluster mode.
Spark programs are divided into a driver part and executors part.
The driver program manages the executors task.

    module.exports = header: 'Ambari Spark2 History Server Check', handler: ({options}) ->

## Wait

      @connection.wait
        header: 'Wait'
        servers: options.wait.http
        retry: 10
        sleep: 3000
