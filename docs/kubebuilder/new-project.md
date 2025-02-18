# プロジェクトの雛形作成

それではさっそく`kubebuilder init`コマンドを利用して、プロジェクトの雛形を生成しましょう。

```console
$ mkdir markdown-view
$ cd markdown-view
$ kubebuilder init --domain zoetrope.github.io --repo github.com/zoetrope/markdown-view
```

`--domain`で指定した名前はCRDのグループ名に使われます。
あなたの所属する組織が保持するドメインなどを利用して、ユニークでvalidな名前を指定してください。

`--repo`にはgo modulesのmodule名を指定します。
GitHubにリポジトリを作る場合は`github.com/<user_name>/<product_name>`を指定します。

コマンドの実行に成功すると、下記のようなファイルが生成されます。

```
├── Dockerfile
├── Makefile
├── PROJECT
├── README.md
├── config
│    ├── default
│    │    ├── kustomization.yaml
│    │    ├── manager_auth_proxy_patch.yaml
│    │    └── manager_config_patch.yaml
│    ├── manager
│    │    ├── controller_manager_config.yaml
│    │    ├── kustomization.yaml
│    │    └── manager.yaml
│    ├── prometheus
│    │    ├── kustomization.yaml
│    │    └── monitor.yaml
│    └── rbac
│         ├── auth_proxy_client_clusterrole.yaml
│         ├── auth_proxy_role.yaml
│         ├── auth_proxy_role_binding.yaml
│         ├── auth_proxy_service.yaml
│         ├── kustomization.yaml
│         ├── leader_election_role.yaml
│         ├── leader_election_role_binding.yaml
│         ├── role_binding.yaml
│         └── service_account.yaml
├── go.mod
├── go.sum
├── hack
│    └── boilerplate.go.txt
└── main.go
```

Kubebuilderによって生成されたgo.modおよびMakefileには、少し古いバージョンのcontroller-runtimeとcontroller-genが使われている場合があります。
必要に応じて、最新のバージョンを利用するように以下のように書き換えておきましょう。
なお、go.modを書き換えた後は`go mod tidy`コマンドを実行してください。

- go.mod

```diff
-       sigs.k8s.io/controller-runtime v0.12.1
+       sigs.k8s.io/controller-runtime v0.12.3
```

- Makefile

```diff
-CONTROLLER_TOOLS_VERSION ?= v0.9.0
+CONTROLLER_TOOLS_VERSION ?= v0.9.2
```


それでは生成されたファイルをそれぞれ見ていきましょう。

## Makefile

コード生成やコントローラーのビルドなどをおこなうためのMakefileです。

`make help`でターゲットの一覧を確認してみましょう。

```console
make help

Usage:
  make <target>

General
  help             Display this help.

Development
  manifests        Generate WebhookConfiguration, ClusterRole and CustomResourceDefinition objects.
  generate         Generate code containing DeepCopy, DeepCopyInto, and DeepCopyObject method implementations.
  fmt              Run go fmt against code.
  vet              Run go vet against code.
  test             Run tests.

Build
  build            Build manager binary.
  run              Run a controller from your host.
  docker-build     Build docker image with the manager.
  docker-push      Push docker image with the manager.

Deployment
  install          Install CRDs into the K8s cluster specified in ~/.kube/config.
  uninstall        Uninstall CRDs from the K8s cluster specified in ~/.kube/config. Call with ignore-not-found=true to ignore resource not found errors during deletion.
  deploy           Deploy controller to the K8s cluster specified in ~/.kube/config.
  undeploy         Undeploy controller from the K8s cluster specified in ~/.kube/config. Call with ignore-not-found=true to ignore resource not found errors during deletion.

Build Dependencies
  kustomize        Download kustomize locally if necessary.
  controller-gen   Download controller-gen locally if necessary.
  envtest          Download envtest-setup locally if necessary.
```

## PROJECT

ドメイン名やリポジトリのURLや生成したAPIの情報などが記述されています。
基本的にこのファイルを編集することはあまりないでしょう。

## hack/boilerplate.go.txt

自動生成されるソースコードの先頭に挿入されるボイラープレートです。

デフォルトではApache 2 Licenseの文面が記述されているので、必要に応じて書き換えてください。

## main.go

これから作成するカスタムコントローラーのエントリーポイントとなるソースコードです。

ソースコード中に`//+kubebuilder:scaffold:imports`, `//+kubebuilder:scaffold:scheme`, `//+kubebuilder:scaffold:builder`などのコメントが記述されています。
Kubebuilderはこれらのコメントを目印にソースコードの自動生成をおこなうので、決して削除しないように注意してください。

## config

configディレクトリ配下には、カスタムコントローラーをKubernetesクラスターにデプロイするためのマニフェストが生成されます。

実装する機能によっては必要のないマニフェストも含まれているので、適切に取捨選択してください。

### manager

カスタムコントローラーのDeploymentリソースのマニフェストです。
カスタムコントローラーのコマンドラインオプションの変更をおこなった場合など、必要に応じて書き換えてください。

### rbac

各種権限を設定するためのマニフェストです。

`auth_proxy_`から始まる4つのファイルは、[kube-auth-proxy][]用のマニフェストです。
kube-auth-proxyを利用するとメトリクスエンドポイントへのアクセスをRBACで制限できます。

`leader_election_role.yaml`と`leader_election_role_binding.yaml`は、リーダーエレクション機能を利用するために必要な権限です。

`role.yaml`と`role_binding.yaml`は、コントローラーが各種リソースにアクセスするための権限を設定するマニフェストです。
この2つのファイルは基本的に自動生成されるものなので、開発者が編集する必要はありません。

必要のないファイルを削除した場合は、`kustomization.yaml`も編集してください。

### prometheus

Prometheus Operator用のカスタムリソースのマニフェストです。
Prometheus Operatorを利用している場合、このマニフェストを適用するとPrometheusが自動的にカスタムコントローラーのメトリクスを収集してくれるようになります。

### default

上記のマニフェストをまとめて利用するための設定が記述されています。

`manager_auth_proxy_patch.yaml`は、[kube-auth-proxy][]を利用するために必要なパッチです。
kube-auth-proxyを利用しない場合は削除しても問題ありません。

`manager_config_patch.yaml`は、カスタムコントローラーのオプションを引数ではなくConfigMapで指定するためのパッチファイルです。

利用するマニフェストに応じて、`kustomization.yaml`を編集してください。

[kube-auth-proxy]: https://github.com/brancz/kube-rbac-proxy
