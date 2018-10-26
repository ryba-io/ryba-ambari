
# Shinken Reactionner Start

    module.exports = header: 'Shinken Reactionner Start', handler: (options) ->
      options = options.options if options.options?


      @service.start name: 'shinken-reactionner'
