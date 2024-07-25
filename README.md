# Go Web Application

This is a simple website written in Golang. It uses the `net/http` package to serve HTTP requests.

## Running the server locally

To run the server locally, execute the following command:

```bash
go run main.go
```

The server will start on port 8080. You can access it by navigating to `http://localhost:8080/courses` in your web browser

## Deployment Workflow

1. Write the Dockerfile

2. Setup the `Github action` flow for CI build

3. Setup the `tag rules` in the github repository

4. Setup the required tokens, username, password for the Github action flow

5. Set the Github action based on the proper tag push based on the environment

6. Configure Helm templates

7. We will deploy two deployments bassed on the environments, so environment specific values.yaml files should be specified

8. Configure the minikube cluster

9. Start Minikube and Enable Ingress

```sh
minikube start
minikube addons enable ingress
```

10. Create the namespaces required

```sh
kubectl create namespace web-dev
kubectl create namespace web-prod
kubectl create namespace argocd
kubectl create namespace prometheus
kubectl create namespace grafana
```

11. Create the argocd components

```sh
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

12. Configure the argocd-server, change it to Service type LoadBalancer

```sh
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

For Windows

```sh
kubectl patch svc argocd-server -n argocd -p '{\"spec\": {\"type\": \"LoadBalancer\"}}'
```

13. Configure the `hosts` file
> For Windows the location : _C:\Windows\System32\drivers\etc\hosts_

run the command

```sh
kubectl get nodes -o wide
```

or for minikube

```yaml
minikube ip
```

configure the hosts file in the following pattern

```yaml
192.168.59.101 go-web-portfolio.com
192.168.59.101 go-web-portfolio-dev.com
```

NOTE `192.168.59.101` should be replaced with your node ip

14.  Fetch the ardocd secret and decode it with base64 --decode

```yaml

```

15. Open the argocd from browser
> http://<node ip>:<argocd service nodeport>

16. Configure the argocd setup

- Create two projects with the name web-go-dev, web-go-prod
- The sync should be automatic
- Check the auto-heal option
- Provide the correct Github URL
- This will detect the chart folder
- While providing the vaules files provide the values.yaml, values-dev.yaml for the web-dev project
- Provide the values.yaml, values-prod.yaml for web-prod project

17. Commit all the changes in the Github

18. Create a tag with the following patterm
> mismatch of pattern will not run the pipeline

**Pattern of the tag** `<dev|prod>-release-<3 digits>-<tag message>`

to trigger the pipeline for dev
```yaml
git tag dev-release-001-July25release
git push --tag
```

to trigger the pipeline for prod
```yaml
git tag prod-release-001-July25release
git push --tag
```

## Setup the Cluster and Workloads Monitoring with Prometheus and Grafana

19. Setup the Prometheus configuration on the cluster via Helm charts

Add Prometheus Helm Repository
```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts -n prometheus
```
Update Helm Repository
```sh
helm repo update
```
Install the Prometheus Helm Chart
```sh
helm install prometheus prometheus-community/prometheus -n prometheus
```
Expose Prometheus Service
> This is required to access prometheus-server using the browser
```sh
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext -n prometheus
```

20. Setup the Grafana configuration on the Cluster via Helm Charts

Add Grafana Helm Repository
```sh
helm repo add grafana https://grafana.github.io/helm-charts
```
Update Helm Repository
```sh
helm repo update
```
Install the Prometheus Helm Chart
```sh
helm install grafana grafana/grafana -n grafana
```
Expose Grafana Service
> This is required to access grafana-server using the browser
```sh
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext -n grafana
```

21. Fetch the Grafana secret which will require to Login to grafana from the browser

22. Setup Grafana

23. Configure the dashboards for monitoring