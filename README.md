# eacs

Extensible Access Control System.

System was designed to be secure and highly customizable. Each access control module has full logic on itself and as such can be customized for wide variety of applications (door opening, machine usage restriction, vending machine etc.). These modules depend on external microservices which provide various resources. Example deployment diagram shown bellow.

![deployment diagram](https://raw.githubusercontent.com/chemicstry/eacs/master/images/deployment.png)

## Microservices

Access control modules connect to microservices using JSON-RPC protocol via WebSocket (HTTPS).

- [Tag Authentication Service](https://github.com/chemicstry/eacs-tag-auth) is responsible for authenticating cryptographic RFID tags. This service establishes tunnel with an RFID tag and authenticates it without leaking keys.
- [User Authentication Service](https://github.com/chemicstry/eacs-user-auth) is LDAP database adapter, which provides simple API to query users and permissions based on tag UID.
- [Message (Event) Bus Service](https://github.com/chemicstry/eacs-message-bus) is MQTT broker that allows access control modules to emit events to external applications.

Bellow you can see a sequence diagram of all microservices during typical tag authentication.

![sequence diagram](https://raw.githubusercontent.com/chemicstry/eacs/master/images/sequence_diagram.png)

## Installation

Each microservice can be run on its own and on different machines. However, a simplified docker installation is available which is described bellow.

Installation requires Docker, Node.js and opnessl.

1. Clone git repository

```
git clone https://github.com/chemicstry/eacs
cd eacs/docker
```

3. Generate TLS keypair which is used to verify service authenticity

```
openssl req -newkey rsa:2048 -nodes -keyout tls_key.pem -x509 -days 365 -out tls_cert.pem
```

4. Install JWT token generation utility. JWT tokens are used for access control module authentication inside microservices

```
npm install -g eacs-token-gen
```

5. Generate JWT keys and token with `first_module` identifier and `eacs-user-auth:auth_uid` permission for the first access control module, it will be needed later

```
eacs-token-gen genkeys
eacs-token-gen token first_module eacs-user-auth:auth_uid 
```

6. Enter LDAP connection details in `docker-compose.yml`
7. Start microservices

```
docker-compose up
```

8. Your previous terminal is now done for good. Open new terminal and clone ESP32 access control module repository

```
git clone https://github.com/chemicstry/eacs-esp32
cd eacs-esp32
```

9. Instructions from this point are unclear. Stop here to prevent your body parts getting stuck somewhere. In case you continue, you need ESP-IDF and toolchain to compile module software. You also have to set microservice IP addresses, SHA256 certificate fingerprints and JWT token. Once everything is configured and flashed, ESP32 should connect to microservices.

10. Profit
