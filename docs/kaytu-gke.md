# Installing Opengovernance on GKE

- make sure apis are enabled - especially caontainer.googleapi.com
- use terraform to setup infra
- download values.yaml
- update the `opengovernance.domain.main` and `dex.config.staticClients` fields in the yaml.
- use helm chart to deploy 
- create static ip - `gcloud compute addresses create web-static-ip --global`
- create A record for the IP
- create google managed certificate 
```
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: managed-certificate
  namespace: opengovernance
spec:
  domains:
    - kaytu.example.com
```
- install ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kaytu-ingress
  namespace: opengovernance
  annotations:
    kubernetes.io/ingress.global-static-ip-name: "web-static-ip"
    networking.gke.io/managed-certificates: managed-certificate
spec:
  ingressClassName: gce
  rules:
    - host: kaytu.example.com
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: nginx-proxy
              port:
                number: 80
```

