---
title: "Private access to managed services: PrivateLink, Private Endpoint and Private Service Connect"
slug: private-access-managed-services
date: 2026-09-23
description: "Three ways to consume managed services without going over the Internet, with a CLI example for each."
clouds: [aws, azure, gcp]
tags: [networking, security, private-connectivity]
---

Reaching a managed service (a secrets manager, a key vault or the provider's own APIs) without traversing the Internet is a common requirement in regulated environments. All three providers solve it with a private IP inside your network that fronts the service.

## AWS: interface VPC endpoints (PrivateLink)

An `Interface` endpoint creates network interfaces in your subnets. With `--private-dns-enabled`, the service's public name resolves to those private IPs inside the VPC.

```bash
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-0123456789abcdef0 \
  --vpc-endpoint-type Interface \
  --service-name com.amazonaws.eu-west-1.secretsmanager \
  --subnet-ids subnet-aaa subnet-bbb \
  --security-group-ids sg-0123456789abcdef0 \
  --private-dns-enabled
```

## Azure: Private Endpoint

A Private Endpoint targets a specific resource (for example, a Key Vault) and sub-resource (`--group-id`). DNS resolution relies on private zones such as `privatelink.vaultcore.azure.net`.

```bash
az network private-endpoint create \
  --name pe-kv-prod \
  --resource-group rg-security \
  --vnet-name vnet-prod \
  --subnet snet-private-endpoints \
  --private-connection-resource-id "$(az keyvault show -n kv-prod --query id -o tsv)" \
  --group-id vault \
  --connection-name kv-prod-conn
```

## Google Cloud: Private Service Connect for Google APIs

With PSC you reserve a global internal IP and create a forwarding rule targeting the Google APIs bundle. Then you configure DNS so `*.googleapis.com` resolves to that IP.

```bash
gcloud compute addresses create psc-googleapis \
  --global --purpose=PRIVATE_SERVICE_CONNECT \
  --addresses=10.255.255.254 --network=vpc-prod

gcloud compute forwarding-rules create pscapis \
  --global --network=vpc-prod \
  --address=psc-googleapis \
  --target-google-apis-bundle=all-apis
```

## What to watch for

DNS is where most rollouts break. In hybrid environments, on-premises resolvers must forward the private zones to the cloud (Route 53 Resolver inbound endpoints, Azure DNS Private Resolver or Cloud DNS inbound server policies). Design this before creating the first endpoint.

## Official documentation

- [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html)
- [Azure Private Endpoint](https://learn.microsoft.com/azure/private-link/private-endpoint-overview)
- [Private Service Connect](https://cloud.google.com/vpc/docs/private-service-connect)
