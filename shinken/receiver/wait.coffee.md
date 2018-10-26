
# Shinken Receiver Wait

    module.exports = header: 'Shinken Receiver Wait', handler: (options) ->
      options = options.options if options.options?

      @connection.wait options.wait.tcp
