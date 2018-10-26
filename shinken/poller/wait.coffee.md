
# Shinken Poller Wait

    module.exports = header: 'Shinken Poller Wait', handler: (options) ->
      options = options.options if options.options?

      @connection.wait options.wait.tcp