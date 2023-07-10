# 設定リファレンス
ここでは、Contrastエージェントオペレータがアクセスする全ての設定エンティティのスキーマについて説明します。一部のエンティティは、オプション(任意)となります。

- [AgentConfiguration](#AgentConfiguration)
- [AgentConnection](#AgentConnection)
- [AgentInjector](#AgentInjector)
- [ClusterAgentConfiguration](#ClusterAgentConfiguration)
- [ClusterAgentConnection](#ClusterAgentConnection)
## AgentConfiguration
~~~ yaml
apiVersion: agents.contrastsecurity.com/v1beta1
kind: AgentConfiguration
metadata:
  name: example-agent-configuration
  namespace: default
spec:
  yaml: |
    server:
      environment: QA
  suppressDefaultServerName: false
  suppressDefaultApplicationName: false
  initContainer:
    securityContext: {}
~~~

|プロパティ|型|必須|デフォルト値|説明|
|-------------------------------------|-------------------|----------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
|spec.yaml|string|任意||Contrast_security.yamlファイル。複数行に対応しています。|
|spec.suppressDefaultServerName|boolean|任意|False|Falseの場合、デフォルト(通常はPod名)を使用せずに、挿入されるワークロード('kubernetes-{namespace}')にContrastサーバ名が自動的に設定されます。|
|spec.suppressDefaultApplicationName|boolean|任意|False|Falseの場合、デフォルト(エージェントが生成)を使用せずに、挿入されるワークロード(ワークロード名)にContrastアプリケーション名が自動的に設定されます。|
|spec.initContainer.securityContext|V1SecurityContext|任意|null|指定された場合、生成されたInitコンテナのセキュリティコンテキストを上書きします。[SecurityContext v1 core](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#securitycontext-v1-core)を参照してください。|
~~~ yaml

~~~
- 接続キーは、YAMLファイルで指定しても無視されるので、YAMLファイルで指定しないでください。
## AgentConnection
~~~ yaml
apiVersion: agents.contrastsecurity.com/v1beta1
kind: AgentConnection
metadata:
  name: example-agent-connection
  namespace: default
spec:
  url: https://app.contrastsecurity.com/Contrast
  apiKey:
    secretName: example-agent-connection-secret
    secretKey: apiKey
  serviceKey:
    secretName: example-agent-connection-secret
    secretKey: serviceKey
  userName:
    secretName: example-agent-connection-secret
    secretKey: userName
~~~

|プロパティ|型|必須|デフォルト値|説明|
|----------------------------|--------|----------| ------------- |---------------------------------------------------------------------|
|spec.url|string|◯||ContrastサーバのURL|
|spec.apiKey.secretName|string|◯||apiKey(APIキー)を指定しているSecretの名前|
|spec.apiKey.secretKey|string|◯||apiKey(APIキー)を指定しているSecretでの値に対するキー|
|spec.serviceKey.secretName|string|◯||serviceKey(エージェントサービスキー)を指定しているSecretの名前|
|spec.serviceKey.secretKey|string|◯||serviceKey(エージェントサービスキー)を指定しているSecretでの値に対するキー|
|spec.userName.secretName|string|◯||userName(エージェントユーザ名)を指定しているSecretの名前|
|spec.userName.secretKey|string|◯||userName(エージェントユーザ名)を指定しているSecretでの値に対するキー|
~~~ yaml

~~~
- セキュリティ上、これらのSecretはAgentConnectionと同じネームスペースに含まれる必要があります。
## AgentInjector
~~~ yaml
apiVersion: agents.contrastsecurity.com/v1beta1
kind: AgentInjector
metadata:
  name: example-injector-dotnet-core
  namespace: default
spec:
  enabled: true
  version: latest
  type: dotnet-core
  image:
    registry: docker.io/contrast
    name: agent-dotnet-core
    pullSecretName: contrastdotnet-pull-secret
    pullPolicy: Always
  selector:
    images:
      - "*"
    labels:
      - name: app
        value: example-*
  connection:
    name: example-agent-connection
  configuration:
    name: example-agent-configuration
~~~

|プロパティ|型|必須|デフォルト値|説明|
|----------------------------|--------|----------| ------------- |---------------------------------------------------------------------|
|spec.enabled|boolean|任意|TRUE|このエージェントの組み込みを有効または無効にします。|
|spec.version|string|任意|latest|組み込むContrastエージェントのバージョン。リテラルの'latest'は、最新バージョンを組み込みます。バージョンの部分一致に対応しています。例えば、'2' を指定すると、バージョン '2.1.0' が選択されます。|
|spec.type|agentType|◯||組み込むエージェントのタイプ。'dotnet-core'、'java'、'nodejs'、'php'のいずれかを指定できます。|
|spec.image.registry|string|任意|docker.io/contrast|エージェントイメージをダウンロードするために使用するイメージレジストリ。このレジストリには、組み込まれるPodとオペレータがアクセスできる必要があります。|
|spec.image.name|string|任意|{タイプによる}|組み込むイメージに使用する名前|
|spec.image.pullSecretName|string|任意||PodのimagePullSecretsリストに追加するプルシークレットの名前|
|spec.image.pullPolicy|string|任意|Always|Contrastのイメージを取得する際に使用するプルポリシー。詳細は、KubernetesのimagePullPolicyを参照してください。|
|spec.selector.images|string[]|任意|Podにあるすべてのコンテナを選択|エージェントを組み込むコンテナイメージ。Globパターン(ワイルドカード)に対応しています。空(デフォルト)の場合、Pod内のすべてのコンテナを選択します。|
|spec.selector.labels|labelSelector[]|任意|ネームスペースにあるすべてのワークロードを選択|Deployment/StatefulSet/DaemonSet/DeploymentConfigのラベルで、そのPodがエージェントを組み込みむ対象となります。空(デフォルト)の場合、名前空間内のすべてのワークロードを選択します。|
|spec.connection.name|string|任意|ClusterAgentConnectionで指定されたAgentConnectionがデフォルト|AgentConnectionのリソース名。同じネームスペース内に存在する必要があります。|
|spec.configuration.name|string|任意|ClusterAgentConfigurationで指定されたAgentConfigurationがデフォルト|AgentConfigurationのリソース名。同じネームスペース内に存在する必要があります。|
~~~ yaml

~~~
- 既存のAgentInjectorを無効にすると、選択したワークロードから全てのエージェントの組み込みが削除されます。
- 参照するAgentConnectionとAgentConfigurationは、AgentInjectorと同じネームスペースに存在する必要があります。
- カスタムレジストリを使用する場合、挿入されるPodとオペレータの両方が、デフォルトのプルシークレットまたはカスタムのプルシークレットを通じて、アクセスできる必要があります。
- 本番前の環境でエージェントを使用する場合は、エージェントバージョンの`latest`を推奨します。
- AgentInjectorは、Deployment、StatefulSet、DaemonSet、およびDeploymentConfig(OpenShift上)ワークロードを選択することをサポートします。Podを直接挿入することはサポートされていません。
- 選択したワークロードが1つのPodで多くのコンテナを作成する場合、spec.selector.imagesを使用して、組み込まれるコンテナをフィルタリングすることができます。

**labelSelector**

|プロパティ|型|必須|デフォルト値|説明|
|----------|--------|----------|---------------|---------------------------------------------------------------|
|name|string|◯||一致させるラベルの名前|
|value|string|◯||一致させるラベルの値。Globパターン(ワイルドカード)に対応しています。|

- ラベルの選択(複数)は、論理AND演算を使用して累積します。

**agentType**

|エージェント|エージェントタイプ|
|-------------------------|----------------|
|.NET Core|dotnet-core|
|Java|java|
|Node.js|nodejs|
|NodeJS (Protectのみ)|nodejs-protect|
|PHP|php|

- タイプの情報については、[オペレータのサポート対象テクノロジ](./02-supported-technologies.md)も参照してください。
## ClusterAgentConfiguration
~~~ yaml
apiVersion: agents.contrastsecurity.com/v1beta1
kind: ClusterAgentConfiguration
metadata:
  name: default-agent-configuration
  namespace: contrast-agent-operator
spec:
  namespaces:
    - default
  template:
    spec:
      yaml: |
        server:
          environment: QA
~~~

|プロパティ|型|必須|デフォルト値|説明|
|-----------------|--------------------|----------|-----------------|------------------------------------------------------------------------------------------|
|spec.namespaces|string[]|任意|全てのネームスペース|このAgentConfigurationテンプレートを適用するネームスペース。Globパターン(ワイルドカード)に対応しています。|
|spec.template|AgentConfiguration|◯||'spec.namespaces'で選択したネームスペースに適用するデフォルトのAgentConfiguration|
~~~ yaml

~~~
- セキュリティ上、ClusterAgentConfigurationマニフェストは、オペレータと同じネームスペースにデプロイする必要があります。
## ClusterAgentConnection
~~~ yaml
apiVersion: agents.contrastsecurity.com/v1beta1
kind: ClusterAgentConnection
metadata:
  name: default-agent-connection
  namespace: contrast-agent-operator
spec:
  namespaces:
    - default
  template:
    spec:
      url: http://app.contrastsecurity.com/Contrast
      apiKey:
        secretName: default-agent-connection-secret
        secretKey: apiKey
      serviceKey:
        secretName: default-agent-connection-secret
        secretKey: serviceKey
      userName:
        secretName: default-agent-connection-secret
        secretKey: userName
~~~

|プロパティ|型|必須|デフォルト値|説明|
|-----------------|-----------------|----------|-----------------|---------------------------------------------------------------------------------------|
|spec.namespaces|string[]|任意|全てのネームスペース|このAgentConfigurationテンプレートを適用するネームスペース。Globパターン(ワイルドカード)に対応しています。|
|spec.template|AgentConnection|◯||'spec.namespaces'で選択したネームスペースに適用するデフォルトのAgentConnection|
~~~ yaml

~~~
- セキュリティ上、ClusterAgentConnectionマニフェストは、オペレータと同じネームスペースにデプロイする必要があります。
- ClusterAgentConnectionで参照するSecretは、ClusterAgentConnectionエンティティをデプロイするのと同じネームスペースに存在する必要があります。
