# Data Encryption
K8s suppors the ability to encrypt secret data at rest. For that, k8s needs an encryption key, config file and send them to controllers.

## Encryption Key
```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```
## The Encryption Config File
Create the encryption-config.yaml encryption config file:
```
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```
                                   
Copiar contenido a controladres
