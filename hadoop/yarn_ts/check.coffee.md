
# Hadoop YARN Timeline Server Check

Check the Timeline Server.

    module.exports = header: 'YARN ATS Ambari Check', handler: ({options}) ->


## Wait

      @connection.wait
        header: 'Webapp'
        servers: options.wait.webapp
        retry: 3
        sleep: 3000

## Assert

Ensure The the server to be started.

      @connection.assert
        header: 'Webapp'
        servers: options.wait.webapp
        retry: 3
        sleep: 3000

Check the HTTP server with a JMX request.

      protocol = if options.yarn_site['yarn.http.policy'] is 'HTTP_ONLY' then 'http' else 'https'
      address_key = if protocol is 'http' then "address" else "https.address"
      address = options.yarn_site["yarn.timeline-service.webapp.#{address_key}"]
      @system.execute
        header: 'HTTP Port'
        cmd: mkcmd.hdfs options.hdfs_krb5_user, "curl --negotiate -k -u : #{protocol}://#{address}/jmx?qry=Hadoop:service=ApplicationHistoryServer,name=JvmMetrics"
      , (err, data) ->
        throw err if err
        data = JSON.parse data.stdout
        throw Error "Invalid Response" unless Array.isArray data?.beans

# Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
