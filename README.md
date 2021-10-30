create password file

$ mkdir -p secrets
$ touch secrets/vmq.passwd
$ docker run --rm -v $(PWD)/secrets:/mnt -it erlio/docker-vernemq vmq-passwd /mnt/vmq.passwd inner
$ docker run --rm -v $(PWD)/secrets:/mnt -it erlio/docker-vernemq vmq-passwd /mnt/vmq.passwd outer


register password file as Kubernetes Secret

$ kubectl create secret generic vernemq-passwd --from-file=./secrets/vmq.passwd

Register self-signed certificate files to Kubernetes Secret

create self-signed CA

$ openssl req -new -x509 -days 365 -extensions v3_ca -keyout secrets/ca.key -out secrets/ca.crt
create server certificate signed by self-signed CA

you have to spacify the FQDN of VerneMQ's MQTTS endpoint
$ openssl genrsa -out secrets/server.key 2048
$ openssl req -out secrets/server.csr -key secrets/server.key -new
$ openssl x509 -req -in secrets/server.csr -CA secrets/ca.crt -CAkey secrets/ca.key -CAcreateserial -out secrets/server.crt -days 365
register certifcation files as Kubernetes Secret

$ kubectl create secret generic vernemq-certifications --from-file=./secrets/ca.crt --from-file=./secrets/server.crt --from-file=./secrets/server.key

kubectl apply -f vernemq-cluster.yaml
