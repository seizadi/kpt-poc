# kpt-poc

[kpt](https://github.com/GoogleContainerTools/kpt) tool open-sourced by google
This project is a PoC to use the tool and become familiar with it. You can read
about [kpt](https://kpt.dev/) in this [book](https://kpt.dev/book)

## Install
MacOs:

```sh
wget -o /usr/local/bin/kpt https://github.com/GoogleContainerTools/kpt/releases/download/v1.0.0-beta.13/kpt_darwin_amd64
chmod +x /usr/local/bin/kpt
```

Test installation:
```sh
kpt version
```

Setup zsh autocompletion:
```sh
kpt completion zsh > /usr/local/share/zsh/site-functions/_kpt
```

Make sure your autocompletion is turned on for zsh:
```sh
echo "autoload -U compinit; compinit" >> ~/.zshrc
```

Run using docker
```sh
docker run gcr.io/kpt-dev/kpt:v1.0.0-beta.13 version
```
An image which includes kpt based upon the Google cloud-sdk alpine image.
***TODO: Should find a similar one that has aws-sdk***
```sh
docker run gcr.io/kpt-dev/kpt-gcloud:v1.0.0-beta.13 version
```

## Prerequisites
   * Git
   * Docker
   * Kubernetes (Optional for deployment)

## Quickstart

Download sample package

```sh
kpt pkg get https://github.com/GoogleContainerTools/kpt/package-examples/nginx@v0.7
```

Package Tree
```sh
cd nginx
kpt pkg tree
```

Package under source control
```sh
git init; git add .; git commit -m "Pristine nginx package"
```

Search and Replace
```sh
kpt fn eval --image gcr.io/kpt-fn/search-replace:v0.1 -- by-path='spec.**.app' put-value=my-nginx
kpt fn eval wordpress -i search-replace:v0.1 -- 'by-path=spec.selector.tier'

```


check diff
```sh
git diff
...
diff --git a/deployment.yaml b/deployment.yaml
...
@@ -20,11 +19,11 @@ spec:
   replicas: 4
   selector:
     matchLabels:
-      app: nginx
+      app: my-nginx
...
```

Add function to mutate the manifest when it is rendered to cluster

```sh
cat >> Kptfile <<EOF
  mutators:
    - image: gcr.io/kpt-fn/set-labels:v0.1
      configMap:
        env: dev
EOF
```

Now render it
```bash
kpt fn render
```

To apply package to cluster
live init to initialize inventory
```bash
kpt live init
```
to deploy
```bash
kpt live apply --reconcile-timeout=15m
```

Update package
Checkin changes
```bash
git add .; git commit -m "My customizations"
```
Update package version
```bash
kpt pkg update @v0.8
```
Merge changes to the cluster
```bash
kpt live apply --reconcile-timeout=15m
```
Now delete the package from cluster
```bash
kpt live destroy
```
Get more complex package
```bash
❯ kpt pkg get https://github.com/GoogleContainerTools/kpt.git/package-examples/wordpress@v0.7
❯ cd wordpress
❯ kpt tree
error: unknown command "tree" for "kpt"
❯ kpt pkg tree
Package "wordpress"
├── [Kptfile]  Kptfile wordpress
├── [service.yaml]  Service wordpress
├── deployment
│   ├── [deployment.yaml]  Deployment wordpress
│   └── [volume.yaml]  PersistentVolumeClaim wp-pv-claim
└── Package "mysql"
    ├── [Kptfile]  Kptfile mysql
    ├── [deployment.yaml]  PersistentVolumeClaim mysql-pv-claim
    ├── [deployment.yaml]  Deployment wordpress-mysql
    
    └── [deployment.yaml]  Service wordpress-mysql
```

You can also get remote Git repos that don't have kptfile and just have manifest,
kptfile is created on local file system.
```bash
kpt pkg get https://github.com/kubernetes/examples/staging/cockroachdb
❯ cd cockroachdb
❯ ls
Kptfile                      README.md                    demo.sh
OWNERS                       cockroachdb-statefulset.yaml minikube.sh
❯ kpt pkg tree
Package "cockroachdb"
├── [Kptfile]  Kptfile cockroachdb
├── [cockroachdb-statefulset.yaml]  Service cockroachdb
├── [cockroachdb-statefulset.yaml]  StatefulSet cockroachdb
├── [cockroachdb-statefulset.yaml]  PodDisruptionBudget cockroachdb-budget
└── [cockroachdb-statefulset.yaml]  Service cockroachdb-public

```
