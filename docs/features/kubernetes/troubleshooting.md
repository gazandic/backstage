---
id: troubleshooting
title: Troubleshooting Kubernetes
sidebar_label: Troubleshooting
description: Troubleshooting for Kubernetes
---

## Kubernetes is not showing up on Service Entities

Sometimes, Kubernetes is not showing up on service entities, we should test your
k8s cluster are already connected to Backstage or not.

```curl
curl --location --request POST '{{backstage-backend-url}}:{{backstage-backend-port}}/api/kubernetes/services/:service-entity-name' \
--header 'Content-Type: application/json' \
--data-raw '{
    "entity": {
        "metadata": {
            "name": <service-entity-name>
        }
    }
}
'
```

The curl response should have resources from Kubernetes:

```json
# curl response
{
  "items": [
    {
      "cluster": {
        "name": <cluster-name>
      },
      "resources": [
        {
          "type": "services",
          "resources": [
            {
              "metadata": {
                "creationTimestamp": "2022-03-13T13:52:46.000Z",
                "labels": {
                  "app": <service-entity-name>,
                  "backstage": <selector>,
                  "backstage.io/kubernetes-id": <service-entity-name>
                },
                "name": <service-entity-name>,
                "namespace": <namespace>
              },
              ....
            }
          ]
        },
        ....
        {
          "type": "pods",
          "resources": [
            ,,,,
          ]
        }
      ],
      "errors": []
    }
  ]
}

```

Kubernetes will not be showing anything when catalog info annotations unmatch with
k8s related yaml label (service.yaml, deployment.yaml, etc). We recommend you for
using label selector with adding two labels:

`backstage: <selector>` and `backstage.io/kubernetes-id: <entity-service-name>`.

```yaml
# k8s related yaml (service.yaml, deployment.yaml, ingress.yaml)
metadata:
  {
    creationTimestamp: '2022-03-13T13:52:46.000Z',
    labels:
      {
        'app': <service-entity-name>,
        'backstage': <selector>,
        'backstage.io/kubernetes-id': <service-entity-name>,
      },
    'name': <service-entity-name>,
    'namespace': <namespace>,
  }
```

and the catalog info annotations would use label selector:

```yaml
# catalog-info.yaml (backstage)
annotations:
  backstage.io/kubernetes-label-selector: 'backstage=<selectors'
```