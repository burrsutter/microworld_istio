Assumes Istio is Installed under istio-system

 https://docs.google.com/document/d/1FDd80Ye6mXcPjq9DiwKjVlNyzuYHUYbOUVYKVbnXp7s/edit?usp=sharing

 Veer also has some excellent materials:
 
 https://github.com/VeerMuchandi/istio-on-openshift

### Setup Project
```
oc new-projet microworld

oc adm policy add-scc-to-user privileged -z default -n microworld
```

```
./minishift oc-env
```

```
export PATH="/Users/burr/.minishift/profiles/istio-demo/cache/oc/v3.7.0-rc.0:$PATH"
```

```
./minishift docker-env
```

```
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.102:2376"
export DOCKER_CERT_PATH="/Users/burr/.minishift/profiles/istio-demo/certs"
export DOCKER_API_VERSION="1.24"
```

# microspringboot1
```
cd microspringboot1
```

### Create the fat jar
```
mvn clean compile package
```

### Inject Fabric8 Maven Plugin 
##### note: these projects already have it
```
mvn io.fabric8:fabric8-maven-plugin:3.5.28:setup
```

### Generate the kubernetes yaml files
```
mvn fabric8:resource
```

### generate the Dockerfile
```
mvn fabric8:build
```

Note: You can Ctrl-C instead of waiting, you just need the Dockerfile generated

### Build Docker image
```
docker build -t microspringboot1:latest target/docker/microspringboot1/latest/build
```

### Add istioctl to your PATH

### Inject istio apply your Deployment into Openshift
```
oc apply -f <(istioctl kube-inject -f target/classes/META-INF/fabric8/kubernetes/microspringboot1-deployment.yml)
```

### Create the Service
```
oc create -f target/classes/META-INF/fabric8/kubernetes/microspringboot1-svc.yml
oc get services
```

### Create the Route
```
oc expose service microspringboot1
oc get routes
```

### Hit microspringboot1 endpoints
```
curl microspringboot1-microworld.192.168.99.102.nip.io
```

# microspringboot2
```
cd microspringboot2
```

### Follow the same basic steps above, replacing "microspringboot1" with "2"

### Hit microspringboot2 endpoints
```
curl microspringboot2-microworld.192.168.99.102.nip.io/api/customer

curl -X POST -d "customer=C100" microspringboot2-microworld.192.168.99.102.nip.io/api/customer/orders
```

# microspringboot3

### Follow the same basic steps above, replacing "microspringboot1" with "3"

### Hit microspringboot3 endpoints
```
curl microspringboot3-microworld.192.168.99.102.nip.io/api/customer

curl -X POST -d "customer=C100" microspringboot3-microworld.192.168.99.102.nip.io/api/customer/orders
```

# Check out Grafana, Jaeger and Service Graph dashboards

