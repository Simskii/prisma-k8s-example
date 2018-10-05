# Primsa K8s cluster with optional TLS (Let's encrypt)

This setup is supposed to be a starting point for people who couldn't get it to work or struggle to find the right configuration. Take it from here and configure it as needed, but don't use everything blindly in production, please.

It assumes the following is present:
- A K8s cluster (used v1.12). With configured `kubectl`.
- 3 instances with 2 vCPUs and 2GB Memory each. If you have less nodes or resources, make sure to adjust the resource requests and replicas (`rabbitmq/values.yml` + `prisma/{primary, secondary}.yml`).

Optional for the TLS configurations:
- AWS DNS hosted zone created and ready to use.
- AWS credentials (Key ID + Key) for a user that is able to get and set Route53 record sets at least.

## How to use this guide
Use the steps outlined in `with-tls` or `without-tls` to set up your cluster with or without TLS, respectively.

## Limitations
**What this setup doesn't do: Follow all (production) best practices, for example:**
- Use configmap or secrets to store some of the credentials
- Lock down Tiller access.
- Use LetsEncrypt dev endpoint for fake certificates.
- Additional prod configurations for RabbitMQ.
- Networking rules.
- RabbitMQ running on proper resources to cope with scale.
- Alerts and metrics configuration for Prisma and Rabbit, persistent volumes, etc.
- ...

## What doesn't work (yet)
- RabbitMQ management interface works only partially, as they use non-canonical URL segments that nginx rewrites just canonicalizes. I haven't found a solution yet that would make it work like Apache's `nocanon` route annotation. The overview works, the queue / exchange details don't.
- Proper usage of Prisma primary compute power. In this configuration it only serves the `/management` route, without serving traffic on `/` as well.

## A note on Azure MySQL
- Azure database user has to be of the form: `<username>@<host>`, for example if your host is `testdb.mysql.database.azure.com`, it's `<username>@testdb`.
- Azure uses SSL by default on DB connections. Use `ssl: true` in the Prisma configs in the `databases:` > `default:` key, next to the user name, host, etc. of the database. **IMPORTANT: You need to use the `prisma-prod:1.18-beta` image at minimum for SSL and MySQL to work.** Postgres just works with SSL in Prisma stable.