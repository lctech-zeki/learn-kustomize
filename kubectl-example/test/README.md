# kustomiz with kubectl -k

[web](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/#before-you-begin)

```sh
# configMapGenerator
cat <<EOF >application.properties
FOO=Bar
EOF

cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-1
  files:
  - application.properties
EOF

kubectl kustomize ./

# apiVersion: v1
# data:
#   application.properties: |
#     FOO=Bar
# kind: ConfigMap
# metadata:
#   name: example-configmap-1-8mbdf7882g
```

```sh
# configMapGenerator2
cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-2
  literals:
  - FOO=Bar
EOF

kubectl kustomize ./

# apiVersion: v1
# data:
#   FOO: Bar
# kind: ConfigMap
# metadata:
#   name: example-configmap-2-g2hdhfc6tk

```

```sh
# secretGenerator
cat <<EOF >./password.txt
username=admin
password=secret
EOF

cat <<EOF >./kustomization.yaml
secretGenerator:
- name: example-secret-1
  files:
  - password.txt
EOF

# apiVersion: v1
# data:
#   password.txt: dXNlcm5hbWU9YWRtaW4KcGFzc3dvcmQ9c2VjcmV0Cg==
# kind: Secret
# metadata:
#   name: example-secret-1-t2kt65hgtb
# type: Opaque
```

```sh
cat <<EOF >./kustomization.yaml
secretGenerator:
- name: example-secret-2
  literals:
  - username=admin
  - password=secret
EOF

# apiVersion: v1
# data:
#   password: c2VjcmV0
#   username: YWRtaW4=
# kind: Secret
# metadata:
#   name: example-secret-2-t52t6g96d8
# type: Opaque
```
