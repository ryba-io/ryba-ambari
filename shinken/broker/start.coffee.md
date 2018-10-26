
# Shinken Broker Start

    module.exports = header: 'Shinken Broker Start', handler: (options) ->
      options = options.options if options.options?

      @service.start name: 'shinken-broker'
