
openssl req -x509 -days 3650 -nodes -newkey rsa:2048 -config openssl-andisec.cnf -keyout pkcs5-plain.pem -out cert.pem

openssl pkcs8 -in pkcs5-plain.pem -topk8 -nocrypt -out pkcs8-plain.pem

openssl pkcs8 -in pkcs5-plain.pem -topk8 -out pkcs8-encrypted.pem -passout pass:andisec@123

cp -a "${JAVA_HOME}/jre/lib/security/cacerts" ./cacerts.jks

keytool -importcert -keystore cacerts.jks -storepass changeit -alias andisec-self-signed-wh -file cert.pem

keytool -keystore cacerts.jks -storepass changeit -list | grep andisec-self-signed-wh -A1

## server.conf
http_enable_tls = true
http_tls_cert_file = /usr/andisec/graylog/security/cert.pem
http_tls_key_file = /usr/andisec/graylog/security/pkcs8-encrypted.pem
http_tls_key_password = andisec@123

## bin/graylogctl
-Djavax.net.ssl.trustStore=/usr/andisec/graylog/security/cacerts.jks -Djavax.net.ssl.trustStorePassword=changeit


keytool -delete -alias andisec-self-signed -keystore cacerts.jks -storepass changeit


curl -u 'admin:Andisec@123!@#' -H 'Accept: application/json' -X GET 'https://10.157.1.162:443/system/?pretty=true'


Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.157.1.161    0.0.0.0         UG    100    0        0 enp5s0
10.157.1.0      0.0.0.0         255.255.255.0   U     100    0        0 enp5s0


#Changed password for user kibana
#PASSWORD kibana = MFKLoNISC21PGJFyKOVR
#
#Changed password for user logstash_system
#PASSWORD logstash_system = 44fhGhRmNXr6Yliv8JJn
#
#Changed password for user elastic
#PASSWORD elastic = XijPUUBvWF2GXMVPnutL
~

curl -u elastic:XijPUUBvWF2GXMVPnutL 'http://localhost:9200/_xpack/license'

[root@node01 ~]# curl -u elastic:XijPUUBvWF2GXMVPnutL 'http://localhost:9200/_xpack/license'
{
  "license" : {
    "status" : "active",
    "uid" : "fcc9935f-c326-4c5c-a934-1335a02160fa",
    "type" : "trial",
    "issue_date" : "2021-03-16T03:58:01.763Z",
    "issue_date_in_millis" : 1615867081763,
    "expiry_date" : "2021-04-15T03:58:01.763Z",
    "expiry_date_in_millis" : 1618459081763,
    "max_nodes" : 1000,
    "issued_to" : "elasticsearch",
    "issuer" : "elasticsearch",
    "start_date_in_millis" : -1
  }
}

1.申请license
https://license.elastic.co/registration

2.安装license
curl -XPUT -u elastic:XijPUUBvWF2GXMVPnutL 'http://localhost:9200/_xpack/license?acknowledge=true' -H "Content-Type: application/json" -d @license.json

echo "Hello Graylog, let's be friends." | nc -w 1 -u 127.0.0.1 9099

ssh root@10.157.1.162


## mongo更改告警规则其他类型为非威胁

db.getCollection("event_definitions").update( { "rule_type": "9" }, {$set:{"threat_type": "0"}},{multi:true} );

db.getCollection("event_definitions").find({"threat_type": "0"} );


db.getCollection("event_definitions").remove({_id:ObjectId("6059ad8eb41b18786ae5ce18")})
db.getCollection("event_definitions").remove({_id:ObjectId("6059ad8eb41b18786ae5ce19")})
db.getCollection("event_definitions").remove({_id:ObjectId("6059ad8eb41b18786ae5ce1a")})




https://superuser.com/questions/1451895/err-ssl-key-usage-incompatible-solution

[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
# C = US
# ST = California
# L = Los Angeles
# O = Internet Corporation for Assigned Names and Numbers
# OU = IT Operations
CN = home.arpa
[v3_req]
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = home.arpa
DNS.2 = *.home.arpa
IP.1 = 192.168.1.1
IP.2 = fe80::123:4567:89ab:cdef