# Ranger Admin Wait

Wait for Ranger Admin Policy Manager to start.

    module.exports = header: 'Ambari Ranger Admin Wait', handler: (options) ->

## HTTP

Wait for the Ranger Admin server to accept HTTP connections.

      @wait.execute
        cmd: """
        curl --fail -H "Content-Type: application/json" -k -X GET \
          -u #{options.wait.http.username}:#{options.wait.http.password} \
          "#{options.wait.http.url}"
        """
        code_skipped: [1,7,22]
