
# Shinken Receiver Start

    module.exports = header: 'Shinken Receiver Start', handler: (options) ->
      options = options.options if options.options?

      @service.start name: 'shinken-receiver'
