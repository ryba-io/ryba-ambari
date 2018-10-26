
# Shinken Poller Status

    module.exports =  header: 'Shinken Poller Status', handler: (options) ->
      options = options.options if options.options?

      @service.status name: 'shinken-poller'
