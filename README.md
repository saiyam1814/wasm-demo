# wasm-demo

## install spin(Mac)
brew tap fermyon/tap
brew install fermyon/tap/spin

## install spin Linux 
curl -fsSL https://developer.fermyon.com/downloads/install.sh | bash
sudo mv spin /usr/local/bin/


## create application 

```
spin new
Pick a template to start your application with: http-rust (HTTP request handler using Rust)
Enter a name for your new application: wasmdemo
Description: 
HTTP base: /
HTTP path: /...

```

## Edit the file 
``` vi wasmdemo/src/lib.rs 
```
change this line `.body(Some("Hello, Kodekloud".into()))?)`

## Compile 
`spin build`
## Create Kubernetes cluster(Any cluster should work Civo or Kodekloud playgrounds)

## install KWASM
```
helm repo add kwasm http://kwasm.sh/kwasm-operator/
helm install -n kwasm --create-namespace kwasm-operator kwasm/kwasm-operator
```
## Mark nodes to be able to run wasm workloads
kubectl annotate node --all kwasm.sh/kwasm-node=true

## Create Dockerfile
```
FROM scratch
COPY spin.toml /spin.toml
COPY target/wasm32-wasi/release/wasmdemo.wasm /target/wasm32-wasi/release/wasmdemo.wasm
ENTRYPOINT ["/spin.toml"]
```

## Build an image 
```
docker buildx build --platform wasi/wasm --provenance=false -t saiyam911/kodekloud:latest .
```
## Push the image 

```
docker push saiyam911/kodekloud:latest
```
## Create a runtimeclass
```
echo '
  apiVersion: node.k8s.io/v1                                           
  kind: RuntimeClass
  metadata:
    name: wasmtime-spin
  handler: spin' | kubectl apply -f -
```

## create deployment file 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kodekloud-wasm
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kodekloud-wasm
  template:
    metadata:
      labels:
        app: kodekloud-wasm
    spec:
      runtimeClassName: wasmtime-spin
      containers:
        - name: kodekloud-wasm
          image: saiyam911/kodekloud
          command:
            - /
```
