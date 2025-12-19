Monitoring Stack on kind (Prometheus + Thanos + Grafana + Exporters + Headlamp)

This repository deploys a minimal monitoring stack on a local kind Kubernetes cluster.
Components included:

Prometheus (with Thanos Sidecar)

Thanos Query

Grafana

Node Exporter

Kube State Metrics

Blackbox Exporter

Headlamp (cluster monitoring UI)

No Thanos Store Gateway or Compactor are used.

Namespace used: monitoring

1. Create Namespace
kubectl create ns monitoring

2. Apply Core Monitoring Components
kubectl apply -f prometheus-config.yaml
kubectl apply -f prometheus-sts.yaml
kubectl apply -f prometheus-svc.yaml
kubectl apply -f thanos-query.yaml
kubectl apply -f thanos-svc.yaml
kubectl apply -f grafana.yaml
kubectl apply -f node-exporter.yaml
kubectl apply -f kube-state-metrics.yaml

3. Apply Additional Optional Components
kubectl apply -f blackbox-config.yaml
kubectl apply -f blackbox-deploy.yaml
kubectl apply -f blackbox-svc.yaml
kubectl apply -f headlamp-rbac.yaml
kubectl apply -f headlamp.yaml

4. Verify Pods
kubectl get pods -n monitoring


Expected running components:

prometheus-0

thanos-query-*

grafana-*

node-exporter-*

kube-state-metrics-*

blackbox-exporter-* (if applied)

headlamp-* (if applied)

5. Verify Services
kubectl get svc -n monitoring


All services are ClusterIP by default.

6. Port-Forward Access (Bind to 0.0.0.0)

Use these commands to expose each service externally from the EC2 host or workstation.

Prometheus (9090)
kubectl port-forward --address=0.0.0.0 svc/prometheus -n monitoring 9090:9090

Thanos Query (9091 forwarded to 9090 inside)
kubectl port-forward --address=0.0.0.0 svc/thanos-query -n monitoring 9091:9090

Grafana (3000)
kubectl port-forward --address=0.0.0.0 svc/grafana -n monitoring 3000:3000

Node Exporter (9100)
kubectl port-forward --address=0.0.0.0 svc/node-exporter -n monitoring 9100:9100

Kube State Metrics (8080)
kubectl port-forward --address=0.0.0.0 svc/kube-state-metrics -n monitoring 8080:8080

Blackbox Exporter (9115)
kubectl port-forward --address=0.0.0.0 svc/blackbox-exporter -n monitoring 9115:9115

Headlamp (4466)
kubectl port-forward --address=0.0.0.0 svc/headlamp -n monitoring 4466:443

7. Access URLs

Replace PUBLIC_IP with your EC2 host or workstation IP:

Prometheus: http://PUBLIC_IP:9090

Thanos Query: http://PUBLIC_IP:9091

Grafana: http://PUBLIC_IP:3000

Node Exporter: http://PUBLIC_IP:9100/metrics

Kube State Metrics: http://PUBLIC_IP:8080/metrics

Blackbox Exporter: http://PUBLIC_IP:9115/probe?target=https://google.com&module=http_2xx

Headlamp UI: http://PUBLIC_IP:4466

8. Grafana Login Credentials

Default user: admin
Default password: admin

9. Thanos Discovery Confirmation

Check Thanos Query logs:

kubectl logs deploy/thanos-query -n monitoring | grep store


You should see Prometheus sidecar registered.

10. Safety Notice

Binding to --address=0.0.0.0 makes the forwarded ports public if your EC2 Security Group allows inbound access.
It is strongly recommended to restrict access using:

Security Group rules

SSH tunneling instead of public exposure

Auth on Grafana and Headlamp

11. Cleanup

To delete everything:

kubectl delete ns monitoring


Or delete objects individually:

kubectl delete -f .
