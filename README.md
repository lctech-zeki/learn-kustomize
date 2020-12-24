# learn-kustomize

[kustomize](https://kustomize.io/)

## 國際範例 Helloworld

### 建立 base

```sh
#- A 建立 base 
mkdir -p /helloworld-example 
DEMO_HOME=$(pwd)
BASE=$DEMO_HOME/base
mkdir -p $BASE
curl -s -o "$BASE/#1.yaml" \
  "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/examples/helloWorld/{configMap,deployment,kustomization,service}.yaml"

tree $DEMO_HOME
# /kustomize/helloworld-example
# └── base
#     ├── configMap.yaml
#     ├── deployment.yaml
#     ├── kustomization.yaml
#     └── service.yaml

kustomize build $BASE | kubectl apply -f -

# -test-
curl 172.20.3.57:30325
# <html><body>
# Version 1 : Good Morning!
# </body></html>
```

### 建立 Overlay

```sh
#- B 建立 Overlay
# env folder
OVERLAYS=$DEMO_HOME/overlays
mkdir -p $OVERLAYS/staging
mkdir -p $OVERLAYS/production
# add file
cat <<EOF > $OVERLAYS/staging/kustomization.yaml
namePrefix: staging-
commonLabels:
  variant: staging
  org: acmeCorporation
commonAnnotations:
  note: Hello, I am staging!
bases:
- ../../base
patches:
- map.yaml
EOF

cat <<EOF > $OVERLAYS/staging/map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: the-map
data:
  altGreeting: "Have a pineapple!"
  enableRisky: "true"
EOF

cat <<EOF > $OVERLAYS/production/kustomization.yaml
namePrefix: production-
commonLabels:
  variant: production
  org: acmeCorporation
commonAnnotations:
  note: Hello, I am production!
bases:
- ../../base
patches:
- deployment.yaml
EOF

cat <<EOF > $OVERLAYS/production/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: the-deployment
spec:
  replicas: 5
EOF

tree $DEMO_HOME
# /kustomize/helloworld-example
# ├── base
# │   ├── configMap.yaml
# │   ├── deployment.yaml
# │   ├── kustomization.yaml
# │   └── service.yaml
# └── overlays
#     ├── production
#     │   ├── deployment.yaml
#     │   └── kustomization.yaml
#     └── staging
#         ├── kustomization.yaml
#         └── map.yaml

kustomize build $OVERLAYS/staging | kubectl apply -f -

# -test-
curl 172.20.3.57:30630
# <html><body>
# Version 1 : <em>Have a pineapple!</em>
# </body></html>

```

在此範例中我們定義：

- Kubernetes 應用程式的名稱：
  - 以往我們的應用程式 v1 在跑時，想要測試 v2 版本，最簡單的方式就是複製 v1 版本的配置檔，並將其修改內容更改之後，再來去修改每個檔案中的名稱，否則重複名稱 Kubernetes 會無法建立。而 Kustomize 透過namePrefix參數為此應用程式在 Kubernetes 之中的名稱加個前綴字staging-，所以之後透過kubectl看到其名稱會變成staging-xxxxxx；
- 新增 label：
  - 這個配置不是必須的，但可以看到 Kustomize 可以這樣為此 Staging 的應用程式新增兩個 label；
- 新增 Annotations：
  - 這個配置也不是必須的，但可以看到 Kustomize 可以這樣為此 Staging 的應用程式新增 Annotations；
- 定義 Base：
  - 定義此應用程式的 Base；
- 定義 Patch：
  - 定義 Overlay 與 Base 之間需要 patches 的檔案。
