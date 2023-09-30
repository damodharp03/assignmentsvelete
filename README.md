# assignmentsveletei

##Part 1 - Containerinzation


1. Install Node on linux machin

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

nvm install 18

```


2. Install the npx app and containierize it

```
npx degit sveltejs/template svelte-docker
cd svelte-docker

### USe following docker file 

FROM node:18 AS build

WORKDIR /app

COPY package.json ./
COPY package-lock.json ./
RUN npm install
COPY . ./
RUN npm run build

FROM nginx:1.19-alpine
COPY --from=build /app/public /usr/share/nginx/html
```

Run following command

```
docker build -t svlete:latest .
docker images
```

## Phase 2: Create AWS resources

1. Configure AWS cli using aws configure to connect to target account

2. Create the public ECR Repo 

```
aws ecr create-repository --repository-name devopsec-assignment-ecr --image-scanning-configuration scanOnPush=true --region us-east-2
{
    "repository": {
        "repositoryUri": "929644394706.dkr.ecr.us-east-2.amazonaws.com/devopsec-assignment-ecr",
        "imageScanningConfiguration": {
            "scanOnPush": true
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        },
        "registryId": "929644394706",
        "imageTagMutability": "MUTABLE",
        "repositoryArn": "arn:aws:ecr:us-east-2:929644394706:repository/devopsec-assignment-ecr",
        "repositoryName": "devopsec-assignment-ecr",
        "createdAt": 1632826592.0
    }
}


```

3. Tag the image and push the image to repo

```
docker tag svlete:latest 929644394706.dkr.ecr.us-east-2.amazonaws.com/devopsec-assignment-ecr:svlete

```
4. Perform docker login

```
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 929644394706.dkr.ecr.us-east-2.amazonaws.com
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

```

5. Perform docker push

```

docker push 929644394706.dkr.ecr.us-east-2.amazonaws.com/devopsec-assignment-ecr:svlete
The push refers to repository [929644394706.dkr.ecr.us-east-2.amazonaws.com/devopsec-assignment-ecr]
2d6b3f412323: Pushed
0913ad2bbaf2: Pushed
d1d5915c84e4: Pushed
45b0de752fd9: Pushed
78744c48c9d3: Pushed
6cb051b7bcc1: Pushed
7cf0f434f498: Pushed
8555e663f65b: Pushed
d00da3cd7763: Pushed
4e61e63529c2: Pushed
799760671c38: Pushed
svlete: digest: sha256:dba73db3e9d37fc17ea746171020446de87fe42ad663358e52941282b16a6d80 size: 2643
```


## Phase 3 

Create helm templates
```
helm create svlete
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
Creating svlete

```

Modify the values.yaml and Chart.yaml for the Imagepath, Pull Policy, Ingress details and tag (Templates are uploaded under helmcart folder)

Installation of helm charts on EKS cluster

```
helm install svlete svlete/ --values svlete/values.yaml
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
NAME: svlete
LAST DEPLOYED: Tue Sep 28 17:14:06 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:

```

1. Get the application URL by running these commands:

```
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=svlete,app.kubernetes.io/instance=svlete" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```
Validation of PODS using helm charts
```
kubectl get pods -n default
NAME                       READY   STATUS    RESTARTS   AGE
svlete-58cb7d9d6b-rtjjl   0/1     Running   1          75s
[root@ip-172-31-20-248 svleteassignment]# kubeclt logs -f svlete-58cb7d9d6b-rtjjl -n default
-bash: kubeclt: command not found
[root@ip-172-31-20-248 svleteassignment]# kubectl logs -f  svlete-58cb7d9d6b-rtjjl -n default
 * Serving Flask app "svletes.factory" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 320-245-633
```
Contents of values.yaml
```
cat values.yaml
# Default values for svlete.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: 929644394706.dkr.ecr.us-east-2.amazonaws.com/devopsec-assignment-ecr
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "svlete"

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}
```
