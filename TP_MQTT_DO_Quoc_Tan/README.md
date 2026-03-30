# Securite_appliquee_IoT

## TP Clé MQTT - DO Quoc Tan

### Etape 1 - Créer l'environnement
mkdir -p ~/IoT/formation-Jour2/{certs,csr} 
cd ~/IoT/formation-Jour2 
chmod 700 certs csr 

### Etape 2 - CA root & Génération clés et CSR
openssl genrsa -out broker.key 2048 
openssl genrsa -out client.key 2048 

openssl req -new -key broker.key -out broker.csr -subj "/C=FR/ST=Ile-de-France/L=Paris/O=IB/OU=IB-Data/CN=MQTT-test" 

openssl req -new -key client.key -out client.csr -subj "/C=FR/ST=Ile-de-France/L=Paris/O=IB/OU=IB-Data/CN=iot-client-iba" 

### Etape 3 - Signature avec CA
openssl x509 -req -in broker.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out broker.crt -days 365 

openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365 

### Etape 4 - Sécurisation Fichier
chmod 600 *.key 
chmod 644 *.crt *.csr 

### Etape 5 - Configuration du Mosquitto (dans /etc/mosquitto/conf.d/mosquitto_MQTT.conf)
listener 8883 

cafile /etc/mosquitto/certs/ca.crt 

certfile /etc/mosquitto/certs/broker.crt 

keyfile /etc/mosquitto/certs/broker.key 

require_certificate true 

use_identity_as_username true 

### Etape 6 - TEST
#### **Avec certificat**
***Sur un terminal :***


mosquitto_pub -h localhost -p 8883   --cafile certs/ca.crt   --cert certs/client.crt   --key client.key   -t "test/secured" -m "mTLS OK"

***Sur un autre terminal :***


mosquitto_sub -h localhost -p 8883   --cafile certs/ca.crt   --cert certs/client.crt   --key client.key   -t "test/secured"

***Résultat :***


linux@linux-VirtualBox:~/IoT/formation-Jour2$ mosquitto_pub -h localhost -p 8883   --cafile certs/ca.crt   --cert certs/client.crt   --key client.key   -t "test/secured" -m "mTLS OK"

linux@linux-VirtualBox:~/IoT/formation-Jour2$ mosquitto_sub -h localhost -p 8883   --cafile certs/ca.crt   --cert certs/client.crt   --key client.key   -t "test/secured"
mTLS OK


#### **Sans certificat**
linux@linux-VirtualBox:~/IoT/formation-Jour2$ mosquitto_pub -h localhost -p 8883 --cafile certs/ca.crt -t test -m "nope" 

Error: Problem setting TLS options: File not found.







