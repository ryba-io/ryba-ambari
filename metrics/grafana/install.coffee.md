
# Ambari Metrics Grafana Install

    module.exports =  header: 'Ambari Metrics Grafana Install', handler: ({options}) ->
    
## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'

## IPTables

| Service    | Port  | Proto  | Parameter          |
|------------|-------|--------|--------------------|
| Grafana UI | 3000  | https  | server.http_port  |

      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['ams-grafana-ini']['port'] , protocol: 'tcp', state: 'NEW', comment: "Grafana Port ui" }
        ]

## SSL

      @file.download
        header: 'SSL Cert'
        source: options.ssl.cert.source
        target: options.configurations['ams-grafana-ini']['cert_file']
        local: options.ssl.cert.local
      @file.download
        header: 'SSL Key'
        source: options.ssl.key.source
        target: options.configurations['ams-grafana-ini']['cert_key']
        local: options.ssl.key.local


### METRICS_GRAFANA component wait
Wait for the NODEMANAGER component to be declared on the host

      @ambari.hosts.component_wait
        header: 'METRICS_GRAFANA WAITED'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'METRICS_GRAFANA'
        hostname: options.fqdn

### METRICS_GRAFANA component install
Put the METRICS_GRAFANA component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'METRICS_GRAFANA set installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'METRICS_GRAFANA'
        hostname: options.fqdn
