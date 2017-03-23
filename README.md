# https://answers.splunk.com/answers/7164/how-do-i-set-up-ssl-forwarding-with-new-self-signed-certificates-and-authentication.html
# http://docs.splunk.com/Documentation/Splunk/6.5.2/Security/Aboutsecuringdatafromforwarders

#[INDEXER]

mkdir -p $SPLUNK_HOME/etc/certs
export OPENSSL_CONF=$SPLUNK_HOME/openssl/openssl.cnf
$SPLUNK_HOME/bin/genRootCA.sh -d $SPLUNK_HOME/etc/certs/
$SPLUNK_HOME/bin/splunk createssl server-cert -d $SPLUNK_HOME/etc/certs/ -n splunk-idx -c splunk-idx.nhs.net -p -l 2048
$SPLUNK_HOME/bin/splunk createssl server-cert -d $SPLUNK_HOME/etc/certs/ -n forwarder -p

#inputs.conf
EOF
[SSL]
rootCA = $SPLUNK_HOME/etc/certs/cacert.pem
serverCert = $SPLUNK_HOME/etc/certs/splunk-idx.pem
sslPassword = changeme
requireClientCert = false

[splunktcp-ssl:9997]
compressed = true
EOF


[FORWARDER]

#outputs.conf
EOF
[tcpout]
defaultGroup = splunkssl

[tcpout:splunkssl]
server = 192.168.1.100:9997
compressed = true

[tcpout-server://192.168.1.100:9997]
sslRootCAPath = $SPLUNK_HOME/etc/certs/cacert.pem
sslCertPath = $SPLUNK_HOME/etc/certs/forwarder.pem
sslPassword = changeme2
sslVerifyServerCert = true
sslCommonNameToCheck = splunk-idx-01.example.com
EOF
