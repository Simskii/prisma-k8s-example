## Setup with TLS, using AWS Route53 and Let's Encrypt

Go through the templates and replace placeholders:
* Base64 encode your AWS Access Key: `echo -n '<YOUR_SECRET>' | base64`
* Put the encoded string into `r53secret.yml`
* `YOUR_CONTACT_MAIL` and `YOUR_AWS_ACCESS_KEY_ID` in `cert-issuer/cluster_issuer.yml`
* `<YOUR_DOMAIN>` in `prisma/ingress.yml` and `rabbitmq/ingress.yml`
* Choose a rabbit user name in `rabbitmq/values.yml`. Look for `username: prisma`. Leave the password commented out, this way a secure one will be generated for you later.
* All the config stuff in `prisma/primary.yml` and `prisma/secondary.yml`. Note that you likely don't have the rabbit password yet (see above), so just leave it as it is for now.

Namespaces to use:
* `kubectl create namespace rabbit`
* `kubectl create namespace certificates`
* `kubectl create namespace prisma`

Tiller / Helm config:
* `kubectl --namespace kube-system create serviceaccount tiller`
* `kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller`
* `helm init --service-account tiller`

Nginx / DNS / TLS:
* `helm install stable/nginx-ingress --namespace kube-system`
* Point AWS DNS record to cluster IP that you get from the nginx ingress controller. Depending on where you run (GKE, AWS, Azure, ...), this requires the correct ingress controller configuration to provision the required resources (load balancers, IPs). AKS (Azure / Aws) should just have the correct plugins installed that will automatically provision the underlying resources on nginx ingress installation.
* `helm install --name cert-manager stable/cert-manager --namespace certificates --set ingressShim.defaultIssuerName=letsencrypt-prod --set ingressShim.defaultIssuerKind=ClusterIssuer`
* `kubectl apply -f cert-issuer/r53secret.yml`
* `kubectl create -f cert-issuer/cluster_issuer.yml`

Services:
* `helm install --name rabbit-cluster --namespace rabbit -f rabbitmq/values.yml stable/rabbitmq`
* Get the rabbit password, if you didn't choose one before in the values.yml (info how to get it is on the console as part of the previous command's output).
* `kubectl apply -f rabbitmq/ingress.yaml --namespace rabbit`
* Make sure to replace the missing rabbit password in `prisma/primary.yml` and `prisma/secondary.yml`.
* `kubectl apply -f prisma/primary.yml`
* `kubectl apply -f prisma/secondary.yml`
* `kubectl apply -f prisma/ingress.yml`

Be patient, TLS configuration can take a while before the whole Let's Encrypt domain validation etc. is done. Tail the logs of the CertManager pod to see if there are any issues. It sometimes took up to 15 minutes to snap into place for me.