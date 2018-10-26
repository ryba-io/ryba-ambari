
# Shinken Scheduler Start

    module.exports = header: 'Shinken Scheduler Start', handler: (options) ->
      options = options.options if options.options?

      @service.start name: 'shinken-scheduler'
