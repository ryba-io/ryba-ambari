
# Shinken Reactionner Wait

    module.exports = header: 'Shinken Reactionner Wait', handler: (options) ->
      options = options.options if options.options?

      @connection.wait options.wait.tcp
