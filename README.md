# Dotnet core Cloud Run example

This is a minimal example of using:

- .net Core
- Cloud Code
- Skaffold

## Create a new project

```bash
dotnet new webapp -o MyWebApp --no-https -f net6.0
```

## Support custom ports

In order to work with products such as Cloud Run we will need to support custom ports. Edit your `Program.cs`
file so that the last 2 lines read:

```csharp
var port = Environment.GetEnvironmentVariable("PORT") ?? "8080";
app.Run($"http://0.0.0.0:{port}");
```

## Add a couple configuration files to the root

Create a basic kubernetes config to deploy this sample application. Name it MyWebApp/k8.yaml.
Note that you need to specify your project ID.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-web-app
  namespace: ""
spec:
  containers:
  - name: my-web-app
    image: gcr.io/${PROJECT_ID}/my-web-app
    resources:
      requests:
        cpu: "250m"
      limits:
        cpu: "200m"
```

## Configure skaffold to speed up local development


Create the following MyWebapp/skaffold.yaml. Note we are using buildpacks which means we have to write zero config
for our .net core project to be build and containerized properly.

```yaml
apiVersion: skaffold/v2beta29
kind: Config
build:
  artifacts:
  - image: gcr.io/${PROJECT_ID}/my-web-app
    buildpacks:
      builder: "gcr.io/buildpacks/builder:v1"
      trustBuilder: true
deploy:
  kubectl:
    manifests:
      - k8.yaml
profiles:
- name: gcb
  build:
    googleCloudBuild: {}
```

## Configure your kubernetes cluster

Within Cloud Code open up "cloud code: kubernetes clusters" and add/select one. I used GKE autopilot which will take a few
minutes to initially size for your workload but is a great option if you want GCP to manage your node pools.

This will also result in your local kubeconfig being set so kubectl works correctly. If you don't configure k8s through cloud code,
you should just be able to manually configure kubectl

## Develop/deploy

Start skaffold by running:

```bash
cd MyWebApp
skaffold dev
```

This will build your .net core web server, publish a container, and deploy that to kubernetes. Your filesystem will be watched so that every time you hit save
on a file your code will be immediately redeployed. In order to view your website I recommend utilizing kubectl port forwarding in another terminal.

```bash
while true; do kubectl port-forward my-web-app 8080:8080 ; done
```

Now you can utilize cloud code's "Web preview" functionality to view your .net core website.

## Optional: test reload

Go change the page title. Open up Pages/index.cshtml and change the title to something else. Like:

```csharp
ViewData["Title"] = "Hello Skaffold!";
```

As soon as you hit "save" a rebuild process will kick off. This can take 2 minutes or so.
