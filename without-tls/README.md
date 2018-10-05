## Setup without TLS

Go through the templates and replace placeholders:
* `<YOUR_DOMAIN>` in `prisma/ingress.yml` and `rabbitmq/ingress.yml`. Can be the load balancer URL, for example.
* Choose a rabbit user name in `rabbitmq/values.yml`. Look for `username: prisma`. Leave the password commented out, this way a secure one will be generated for you later.
* All the config stuff in `prisma/primary.yml` and `prisma/secondary.yml`. Note that you likely don't have the rabbit password yet (see above), so just leave it as it is for now.

Namespaces to use:
* `kubectl create namespace rabbit`
* `kubectl create namespace prisma`

Tiller / Helm config:
* `kubectl --namespace kube-system create serviceaccount tiller`
* `kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller`
* `helm init --service-account tiller`

Nginx:
* `helm install stable/nginx-ingress --namespace kube-system`

Services:
* `helm install --name rabbit-cluster --namespace rabbit -f rabbitmq/values.yml stable/rabbitmq`
* Get the rabbit password, if you didn't choose one before in the values.yml (info how to get it is on the console as part of the previous command's output).
* `kubectl apply -f rabbitmq/ingress.yaml --namespace rabbit`
* Make sure to replace the missing rabbit password in `prisma/primary.yml` and `prisma/secondary.yml`.
* `kubectl apply -f prisma/primary.yml`
* `kubectl apply -f prisma/secondary.yml`
* `kubectl apply -f prisma/ingress.yml`