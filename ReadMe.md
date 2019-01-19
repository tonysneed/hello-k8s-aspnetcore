# Hello Kubernetes from ASPNET Core

Demonstrates deploying a Docker image of an ASPNET Core Web API to Kubernetes using Docker for Mac or Windows.

## Prerequisites

1. Install .NET Core SDK
2. Install Docker for Mac or Windows
3. Enable Kubernetes in Docker
    - Docker preferences, check Enable Kubernetes
4. Install Kubernetes dashboard
    - Instructions: https://github.com/kubernetes/dashboard
    ```
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
    ```
5. Create a sample user
    - Instructions: https://github.com/kubernetes/dashboard/wiki/Creating-sample-user
    - Create dashboard-adminuser.yaml file
    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
    name: admin-user
    namespace: kube-system
    ```
    - Run: `kubectl apply -f dashboard-adminuser.yaml`
    - Show bearer token and copy
    ```
    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
    ```
6. Show dashboard
    - Run: `kubectl proxy`
    - Go to: http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
    - Select token option and paste copied token

## DotNet

1. Create directory and cd into it.
2. From terminal run: `dotnet new webapi`
3. Run: `dotnet run .`
4. Browse to: http:localhost:5000/api/values

## Docker

1. Add following **Dockerfile** to project folder:

    ```docker
    FROM microsoft/dotnet:2.2-sdk-alpine AS build

    # Set working directory within container
    WORKDIR /app

    # Copy .csproj and restore as distinct layers
    COPY *.csproj ./
    RUN dotnet restore

    # Copy everything else and build
    COPY . ./
    RUN dotnet publish -c Release -o dist

    # Build runtime image
    FROM microsoft/dotnet:2.2-aspnetcore-runtime-alpine AS runtime
    COPY --from=build /app/dist ./
    ENTRYPOINT ["dotnet", "app.dll"]

    # docker build . -t tonysneed/aspnetapp-2.2
    # docker run -it --rm -p 8080:80 --name myapp tonysneed/aspnetapp-2.2
    ```

2. Run `docker build . -t tonysneed/aspnetapp-2.2`
3. Run `docker run -it --rm -p 8080:80 --name myapp tonysneed/aspnetapp-2.2`
4. Browse to http://localhost:8080/api/values
    - Press Ctrl+C to shut down

## Kubernetes

1. Create a deployment

    ```
    kubectl run hello-kubernetes --image=tonysneed/aspnetapp-2.2 --image-pull-policy=IfNotPresent --port=80
    ```
    
    - View the deployment in the dashboard.
    - See logs to verify service is running:  
    `kubectl get pods`  
    `kubectl logs ` (plus pod name)

2. Expose deployment

    ```
    kubectl expose deployment hello-kubernetes --type=NodePort service "hello-kubernetes" exposed
    ```

    > Note: You can safely **ignore** the following message:  
    `Error from server (NotFound): deployments.extensions "service" not found`

    - View service: `kubectl describe service hello-kubernetes`
    - Copy the **NodePort** and browse to app (substituting port):  
    `http://localhost:30334/api/values`

3. Scale deployment

    ```
    kubectl scale --replicas=3 deployment/hello-kubernetes
    ```

> Tutorial: https://rominirani.com/tutorial-getting-started-with-kubernetes-with-docker-on-mac-7f58467203fd
