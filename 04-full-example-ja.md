# 全体の手順の例

## 開始する前に

ここでは、Vanilla Kubernetesを使用して、Contrastエージェントオペレータのインストールと、クラスタ管理者としてサンプルワークロードを組み込む方法について説明します。OpenShiftを使用してこの例を実行するには、Kubernetesのコマンドを同等のOpenShiftコマンドに置き換えてください。すべてのコマンドは、Bashなどのターミナル内で実行します。

Kubernetesおよび関連するソフトウェアの仕組みについて、基本的な理解があることを前提としています。必要に応じて、お客様の環境に合わせて手順を調整してください。

## 手順 1：オペレータのインストール

オペレータをインストールするには、オペレータのマニフェストをクラスタに適用する必要があります。Contrastは、クラスタに直接適用できる単一のインストール用YAMLファイルを提供し、適切なデフォルト値を提供します。お客様の環境に応じて追加で修正することも可能ですが、その場合は[Kustomize](https://kustomize.io/)などの設定管理ツールを使用することをお勧めします。

このインストール用YAMLファイルによって、contrast-agent-operatorネームスペースが作成されてインストールが行われます。このネームスペースを後で使用します。

```
% kubectl apply -f https://github.com/Contrast-Security-OSS/agent-operator/releases/latest/download/install-prod.yaml

namespace/contrast-agent-operator created
customresourcedefinition.apiextensions.k8s.io/agentconfigurations.agents.contrastsecurity.com created
customresourcedefinition.apiextensions.k8s.io/agentconnections.agents.contrastsecurity.com created
customresourcedefinition.apiextensions.k8s.io/agentinjectors.agents.contrastsecurity.com created
customresourcedefinition.apiextensions.k8s.io/clusteragentconfigurations.agents.contrastsecurity.com created
customresourcedefinition.apiextensions.k8s.io/clusteragentconnections.agents.contrastsecurity.com created
serviceaccount/contrast-agent-operator-service-account created
clusterrole.rbac.authorization.k8s.io/contrast-agent-operator-service-role created
clusterrolebinding.rbac.authorization.k8s.io/contrast-agent-operator-service-role-binding created
service/contrast-agent-operator created
deployment.apps/contrast-agent-operator created
poddisruptionbudget.policy/contrast-agent-operator created
mutatingwebhookconfiguration.admissionregistration.k8s.io/contrast-web-hook-configuration created
```

クラスタが集約されるのをしばらく待つと、オペレータのステータスがRunningになります。

```bash
% kubectl -n contrast-agent-operator get pods

NAME                                      READY   STATUS    RESTARTS   AGE
contrast-agent-operator-57f5cfbf7-9svtt   1/1     Running   0          27s
contrast-agent-operator-57f5cfbf7-fp4vp   1/1     Running   0          39s
```
オペレータを設定する準備ができました。
## 手順 2：オペレータの設定
クラスタのワークロードを組み込む前に、まずオペレータを設定する必要があります。
Kubernetesのシークレットを使用して、接続の認証キーを保存します。以下を実行すると、default-agent-connection-secretという名前でSecretが作成され、contrast-agent-operatorネームスペースに作成されます。
```bash
% kubectl -n contrast-agent-operator \
        create secret generic default-agent-connection-secret \
        --from-literal=apiKey=TODO \
        --from-literal=serviceKey=TODO \
        --from-literal=userName=TODO

secret/default-agent-connection-secret created
```

> 「TODO」を、お使いのContrastサーバインスタンスに相当する値に置き換えてください。[エージェントキーの検索](https://docs.contrastsecurity.com/en/find-the-agent-keys.html)では、Contrast Webインタフェースからエージェントキーを取得する方法を説明します。

接続の設定を完了するには、ClusterAgentConnectionが必要です。以下を実行すると、contrast-agent-operatorネームスペースでClusterAgentConnectionが作成され、前述の手順で作成したSecretのキー値を参照します。

```bash
% kubectl apply -f - <<EOF
apiVersion: agents.contrastsecurity.com/v1beta1
kind: ClusterAgentConnection
metadata:
  name: default-agent-connection
  namespace: contrast-agent-operator
spec:
  template:
    spec:
      url: https://app.contrastsecurity.com/Contrast
      apiKey:
        secretName: default-agent-connection-secret
        secretKey: apiKey
      serviceKey:
        secretName: default-agent-connection-secret
        secretKey: serviceKey
      userName:
        secretName: default-agent-connection-secret
        secretKey: userName
EOF

clusteragentconnection.agents.contrastsecurity.com/default-agent-connection created
```
> ClusterAgentConnectionの名前は重要ではなく、任意の名前を付けることができます。

これでオペレータが設定され、既存のワークロードにエージェントを組み込めるようになりました。

## 手順 3：ワークロードへの組み込み

この例では、Deploymentワークロードを使用して、ASP.&#8203;NET Core [サンプルアプリケーション](https://hub.docker.com/_/microsoft-dotnet-samples)にContrast .NET Core Agentを組み込む方法について説明します。

まず、ASP.&#8203;NET Coreのサンプルアプリケーションをクラスタにデプロイします。以下を実行すると、defaultのネームスペースにDeploymentが作成されます。

```bash
% kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-app
  namespace: default
  labels:
    arbitrary-label: arbitrary-value
spec:
  selector:
    matchLabels:
      app: hello-world-app
  template:
    metadata:
      labels:
        app: hello-world-app
    spec:
      containers:
        - image: contrast/sample-app-aspnetcore:latest
          name: hello-world-app
EOF

deployment.apps/hello-world-app created
```

クラスタが集約されるのをしばらく待つと、デプロイされたワークロードのステータスがRunningになります。

```bash
$% kubectl -n default get pods

NAME                               READY   STATUS    RESTARTS   AGE
hello-world-app-7479d5ff96-p28zx   1/1     Running   0          19s
```

次に、AgentInjector設定エンティティを使用して、.NET Coreエージェントを組み込むようにオペレータを設定します。この場合、AgentInjectorは、前述のDeploymentがデプロイされたのと同じネームスペース(この場合は default)に作成する必要があります。

```bash
% kubectl apply -f - <<EOF
apiVersion: agents.contrastsecurity.com/v1beta1
kind: AgentInjector
metadata:
  name: injector-for-hello-world
  namespace: default
spec:
  type: dotnet-core
  selector:
    labels:
      - name: arbitrary-label
        value: arbitrary-value
EOF

agentinjector.agents.contrastsecurity.com/injector-for-hello-world created
```

Hello-world-appのPodのログを見ると、Contrastの.NET Coreエージェントがアプリケーションに組み込まれていることを確認できます。

```bash
% kubectl -n default logs Deployment/hello-world-app

Defaulted container "hello-world-app" out of: hello-world-app, contrast-init (init)
warn: Microsoft.AspNetCore.DataProtection.Repositories.FileSystemXmlRepository[60]
      Storing keys in a directory '/root/.aspnet/DataProtection-Keys' that may not be persisted outside of the container. Protected data will be unavailable when container is destroyed.
warn: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[35]
      No XML encryptor configured. Key {0ad20893-267f-4635-99b0-1ee74bccbc8b} may be persisted to storage in unencrypted form.
           ___
       _.-|   |          |\__/,|   (`\     Contrast .NET Core Agent 2.1.13.0
      [   |   |          |o o  |__ _) )
       `-.|___|        _.( T   )  `  /     Contrast UI: https://app.contrastsecurity.com
        .--'-`-.     _((_ `^--' /_<  \     Mode:        Assess & Protect
      .+|______|__.-||__)`-'(((/  (((/

info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://[::]:80
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Production
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /app/
```

## 手順 4：オペレータのアンインストール(任意)

クラスタを元の状態に復元するには、まず既存のAgentInjectorを削除します。

```bash
% kubectl -n default delete agentinjector injector-for-hello-world

agentinjector.agents.contrastsecurity.com "injector-for-hello-world" deleted
```

その後、オペレータは、組み込まれた全てのワークロードを組み込み前の状態に復元します。クラスタが集約されたら、オペレータを安全に削除できます。

```bash
% kubectl delete -f https://github.com/Contrast-Security-OSS/agent-operator/releases/latest/download/install-prod.yaml

namespace "contrast-agent-operator" deleted
customresourcedefinition.apiextensions.k8s.io "agentconfigurations.agents.contrastsecurity.com" deleted
customresourcedefinition.apiextensions.k8s.io "agentconnections.agents.contrastsecurity.com" deleted
customresourcedefinition.apiextensions.k8s.io "agentinjectors.agents.contrastsecurity.com" deleted
customresourcedefinition.apiextensions.k8s.io "clusteragentconfigurations.agents.contrastsecurity.com" deleted
customresourcedefinition.apiextensions.k8s.io "clusteragentconnections.agents.contrastsecurity.com" deleted
serviceaccount "contrast-agent-operator-service-account" deleted
clusterrole.rbac.authorization.k8s.io "contrast-agent-operator-service-role" deleted
clusterrolebinding.rbac.authorization.k8s.io "contrast-agent-operator-service-role-binding" deleted
service "contrast-agent-operator" deleted
deployment.apps "contrast-agent-operator" deleted
poddisruptionbudget.policy "contrast-agent-operator" deleted
mutatingwebhookconfiguration.admissionregistration.k8s.io "contrast-web-hook-configuration" deleted
```

