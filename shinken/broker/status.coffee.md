
# Shinken Broker Status

    module.exports =  header: 'Shinken Broker Status', handler: (options) ->
      options = options.options if options.options?

      @service.status name: 'shinken-broker'
