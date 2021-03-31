bin/elasticsearch-plugin install file:///x-pack-6.2.4.zip

bin/x-pack/certutil ca

bin/x-pack/certutil cert --ca elastic-stack-ca.p12

cp  elastic-stack-ca.p12 config/cert

----elasticsearch.yml-----
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: cert/elastic-stack-ca.p12
xpack.security.transport.ssl.trusttore.path: cert/elastic-stack-ca.p12
----elasticsearch.yml-----

chown -R elasticsearch:elasticsearch config/cert
chmod -R 777 config/cert

重启es

bin/x-pack/setup-passwords auto

bin/kibana-plugin install file:///x-pack-6.2.4.zip

----kibana.yml------
elasticsearch.username:
elasticsearch.password:
----kibana.yml------

重启kibana

在剩余节点装x-pack,重启es


