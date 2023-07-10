# セキュリティポリシー
Contrastエージェントオペレータは、様々なセキュリティポリシーでクラスタをサポートします。この機能は`バージョン0.15.0`で初めて追加されました。
## サポートされている実装
### Podのセキュリティアドミッション
エージェントオペレータは、[Podのセキュリティアドミッション](https://kubernetes.io/docs/concepts/security/pod-security-admission/)(バージョン1.25で最初の安定版)を使用して、[Podセキュリティの標準](https://kubernetes.io/docs/concepts/security/pod-security-standards/)を適用するKubernetesクラスタでの実行をサポートします。オペレータと全ての組み込みは、`強制`モード (`最新`バージョン) で `制限された`ポリシーに対してテストされます。

バージョン`0.15.0`未満のオペレータからアップグレードする場合、既存のクラスタはオペレータワークロードに`CONTRAST_RUN_INIT_CONTAINER_AS_NON_ROOT=true`を適用して、この機能を設定する必要があります。新規インストールの場合は、必要ありません。[追加の注意事項](#additional-notes)をご覧ください。
> [Podセキュリティポリシー](https://kubernetes.io/docs/concepts/security/pod-security-policy/)(Kubernetesバージョン1.21で非推奨)はサポートされていません。クラスタは、[Podセキュリティの標準](https://kubernetes.io/docs/concepts/security/pod-security-admission/)を適用するために、Podのセキュリティアドミッションに移行する必要があります。
### OpenShift Security Context Constraints
エージェントオペレータは、組み込みのアドミッション/ミューティングコントローラを使用して、[Security Context Constraints(SCCs)](https://docs.openshift.com/container-platform/4.12/authentication/managing-security-context-constraints.html)を利用するOpenShiftクラスタでの実行をサポートします。オペレータと全ての組み込みは、`制限された`ポリシーに対してテストされます。

デフォルトの`制限付き`ポリシーでseccompポリシーの設定が禁止されているOpenShiftクラスタで実行する場合は、`CONTRAST_SUPPRESS_SECCOMP_PROFILE=true`がオペレーターのワークロードに適用されていることを確認してください。
### サードパーティのアドミッションプラグイン
サードパーティのセキュリティポリシー、アドミッションプラグインは、Contrastエージェントオペレータと連動する可能性がありますが、現時点では相互作用はテストされておらず、正式にサポートされていません。
## 追加の注意事項
`CONTRAST_RUN_INIT_CONTAINER_AS_NON_ROOT=true`が適用されている場合、またはOpenShift環境で実行する場合は、以下の最小エージェントバージョンを使用する必要があります：

|種類|最小バージョン|
| :-: | :-: |
|`dotnet-core`|`2.4.4`|
|`java`|`4.11.0`|
|`nodejs`|`4.30.0`|
|`nodejs-protect`|`5.2.0`|
|`php`|`1.8.0`|

