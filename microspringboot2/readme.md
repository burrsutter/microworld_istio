if the namespace/project is not yet created

oc new-projet microworld

oc adm policy add-scc-to-user privileged -z default -n microworld

./minishift oc-env
export PATH="/Users/burr/.minishift/profiles/istio-demo/cache/oc/v3.7.0-rc.0:$PATH"

./minishift docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.102:2376"
export DOCKER_CERT_PATH="/Users/burr/.minishift/profiles/istio-demo/certs"
export DOCKER_API_VERSION="1.24"

# create the fat jar
mvn clean compile package

# if your project does not yet have fabric8
mvn io.fabric8:fabric8-maven-plugin:3.5.28:setup

# generate the kubernetes yaml files
mvn fabric8:resource

# generate the Dockerfile
mvn fabric8:build

# and build the docker image
docker build -t microspringboot2:latest target/docker/microspringboot2/latest/build

# add istioctl to your PATH

# and inject istio apply your Deployment into Openshift
oc apply -f <(istioctl kube-inject -f target/classes/META-INF/fabric8/kubernetes/microspringboot2-deployment.yml)

# create the Service
oc create -f target/classes/META-INF/fabric8/kubernetes/microspringboot2-svc.yml

oc get services

# create the Route
oc expose service microspringboot2

oc get routes

# hit your endpoints
curl microspringboot2-microworld.192.168.99.102.nip.io/api/customer

curl -X POST -d "customer=C100" microspringboot2-microworld.192.168.99.102.nip.io/api/customer/orders
