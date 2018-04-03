
# Oozie Client Install

Install and configure an Oozie client environment.

The `oozie` command doesnt reference any configuration. It expect the
environmental variable "OOZIE_URL" to connect to the server.

Additionnal oozie properties may be defined inside the "OOZIE_CLIENT_OPTS"
environmental variables. For example, HDP declare its version as
"-Dhdp.version=${HDP_VERSION}".

    module.exports = header: 'Oozie Client Install', handler: (options) ->

## Register

      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"

# Install Component

      @ambari.hosts.component_wait
        header: 'OOZIE_CLIENT WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'OOZIE_CLIENT'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'OOZIE_CLIENT INSTALL'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'OOZIE_CLIENT'
        hostname: options.fqdn

## Profile

Expose the "OOZIE_URL" environmental variable to every users.

      @file
        header: 'Profile Env'
        target: '/etc/profile.d/oozie.sh'
        # export OOZIE_CLIENT_OPTS='-Djavax.net.ssl.trustStore=/etc/hadoop/conf/truststore'
        content: """
        #!/bin/bash
        export OOZIE_URL=#{options.oozie_site['oozie.base.url']}
        """
        mode: 0o0755

## SSL

Over HTTPS, the certificate must be imported into the JRE's keystore for the
client to submit jobs. Setting the java property "javax.net.ssl.trustStore"
in the "OOZIE_CLIENT_OPTS" environmental variable (both in shell and
"oozie-env.sh" file) is enough to retrieve the oozie status but is not honored
when submiting an Oozie job (erreur inside the mapreduce action).

At the moment, we only support adding the certificate authority into the default
Java location ("$JRE_HOME/lib/security/cacerts").

```
keytool -keystore ${JAVA_HOME}/jre/lib/security/cacerts -delete -noprompt -alias tomcat
keytool -keystore ${JAVA_HOME}/jre/lib/security/cacerts -import -alias tomcat -file master3_cert.pem
```

      @java.keystore_add
        header: 'JKS Truststore'
        keystore: "#{options.jre_home or options.java_home}/lib/security/cacerts"
        storepass: "changeit"
        caname: "ryba_cluster" # was tomcat
        cacert: options.ssl.cacert.source
        local: options.ssl.cacert.local
