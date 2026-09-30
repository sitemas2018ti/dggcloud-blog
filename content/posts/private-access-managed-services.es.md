---
title: "Acceso privado a servicios gestionados: PrivateLink, Private Endpoint y Private Service Connect"
slug: private-access-managed-services
date: 2026-09-23
description: "Tres formas de consumir servicios gestionados sin salir a Internet, con el ejemplo de cómo crear cada una desde la CLI."
clouds: [aws, azure, gcp]
tags: [networking, security, private-connectivity]
---

Consumir un servicio gestionado (un gestor de secretos, un almacén de claves o las APIs del propio proveedor) sin pasar por Internet es un requisito habitual en entornos regulados. Los tres proveedores lo resuelven con una IP privada dentro de tu red que actúa como puerta de entrada al servicio.

## AWS: interface VPC endpoints (PrivateLink)

Un endpoint de tipo `Interface` crea interfaces de red en tus subredes. Con `--private-dns-enabled`, el nombre público del servicio resuelve a esas IPs privadas dentro de la VPC.

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

El Private Endpoint se asocia a un recurso concreto (por ejemplo, un Key Vault) y a un sub-recurso (`--group-id`). La resolución DNS se gestiona con zonas privadas como `privatelink.vaultcore.azure.net`.

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

## Google Cloud: Private Service Connect para APIs de Google

Con PSC reservas una IP interna global y creas una regla de reenvío que apunta al paquete de APIs de Google. Después configuras DNS para que `*.googleapis.com` resuelva a esa IP.

```bash
gcloud compute addresses create psc-googleapis \
  --global --purpose=PRIVATE_SERVICE_CONNECT \
  --addresses=10.255.255.254 --network=vpc-prod

gcloud compute forwarding-rules create pscapis \
  --global --network=vpc-prod \
  --address=psc-googleapis \
  --target-google-apis-bundle=all-apis
```

## Qué tener en cuenta

El DNS es donde fallan la mayoría de despliegues. En entornos híbridos, los resolvers on-premises tienen que reenviar las zonas privadas hacia la nube (Route 53 Resolver inbound endpoints, Azure DNS Private Resolver o políticas de servidor entrante de Cloud DNS). Diseña esto antes de crear el primer endpoint.

## Documentación oficial

- [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html)
- [Azure Private Endpoint](https://learn.microsoft.com/azure/private-link/private-endpoint-overview)
- [Private Service Connect](https://cloud.google.com/vpc/docs/private-service-connect)
