
# Shinken Reactionner Status

    module.exports =  header: 'Shinken Reactionner Status', handler: (options) ->
      options = options.options if options.options?


      @service.status name: 'shinken-reactionner'
