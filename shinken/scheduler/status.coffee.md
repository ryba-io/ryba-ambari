
# Shinken Scheduler Status

    module.exports =  header: 'Shinken Scheduler Status', handler: (options) ->
      options = options.options if options.options?

      @service.status name: 'shinken-scheduler'
