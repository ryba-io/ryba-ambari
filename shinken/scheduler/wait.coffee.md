
# Shinken Scheduler Wait

    module.exports = header: 'Shinken Scheduler Wait', handler: (options) ->
      options = options.options if options.options?

      @connection.wait options.wait.http
