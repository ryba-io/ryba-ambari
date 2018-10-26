
# Shinken Receiver Status

    module.exports =  header: 'Shinken Receiver Status', handler: (options) ->
      options = options.options if options.options?

      @service.status name: 'shinken-receiver'
