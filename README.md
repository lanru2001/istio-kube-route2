# istio-kube-route2

## Steps to install Istio with ALB:
- Step 1 : Install istio and set values for profile to default and ingressgateway type to NodePort 
- Step 2 : Install ALB ingress controller
- Step 3 : Write ingress.yaml for istioingressgateway as follows


``` bash 
#Download Istio 
curl -L https://istio.io/downloadIstio | sh -

#Change to the Istio package directory
 cd istio-1.17.2
 
#Add the istioctl client to your path (Linux or macOS):
export PATH=$PWD/bin:$PATH

#Install istio and set values for ingressgateway type to NodePort 
istioctl install   --set  profile=default   --set  values.gateways.istio-ingressgateway.type=NodePort  --set meshConfig.outboundTrafficPolicy.mode=ALLOW_ANY    --set meshConfig.accessLogFile=/dev/stdout                                                                                                                                                       

#Install ALB ingress controller
Load Balancer Controller Installation
https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/deploy/installation/

#Write ingress.yaml for istioingressgateway as follows:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: istio-ingress
  namespace: istio-system
  annotations:
    alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig":
      { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
    alb.ingress.kubernetes.io/backend-protocol: HTTP
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:060866400178:certificate/703dcf11-8357-4465-9162-db1c20565348
    alb.ingress.kubernetes.io/healthcheck-path: "/healthz/ready"
    alb.ingress.kubernetes.io/healthcheck-port: "31100"
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: '30'
    alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '5'
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
    alb.ingress.kubernetes.io/target-type: instance
    alb.ingress.kubernetes.io/subnets: subnet-25a08809,subnet-acfd4ecb,subnet-c81548c4,subnet-edcc34d2
    alb.ingress.kubernetes.io/security-groups: sg-079b741ff8602879c
    alb.ingress.kubernetes.io/scheme: internet-facing
    kubernetes.io/ingress.class: alb
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: ssl-redirect
            port:
              name: use-annotation
        path: /
        pathType: Prefix
      - backend:
          service:
            name: istio-ingressgateway
            port:
              number: 80
        path: /
        pathType: Prefix

```

# Configure service to use Nodeport and name should start with http-service-name

```bash 
service:
   name: http-manager
   type: NodePort      #ClusterIP
   port: 80
   targetPort: 80
   protocol: TCP
  
```
  
# How Istio ALB Ingress and Kubernetes integrate ?
 
![image](https://github.com/atolanrewaju2022/Istio-kube-route/assets/135293313/7bc7350d-546b-4e67-80f9-8a0358cdd1ab)

  
