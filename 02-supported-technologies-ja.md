# サポート対象テクノロジ

## Kubernetes/OpenShiftのサポート

| Kubernetesバージョン | OpenShiftバージョン | Operatorバージョン | サポート終了   |
|--------------------|-------------------|------------------|----------------|
| v1.26              |                   | v0.14.0以降        | 2024年2月24日 |
| v1.25              |                   | v0.8.0以降         | 2023年10月27日|
| v1.24              |                   | v0.4.0以降         | 2023年9月29日 |
| v1.23              | v4.10             | v0.1.0以降         | 2023年2月28日 |
| v1.22              | v4.9              | v0.1.0以降         | 2022年10月28日|
| v1.21              | v4.8              | v0.1.0以降         | 2022年6月28日 |

- Contrastエージェントオペレータは、アップストリームである[Kubernetesコミュニティのサポートポリシー](https://kubernetes.io/releases/patch-releases/#support-period).サポート終了日については、 [Kubernetesのリリースページ](https://kubernetes.io/releases/#release-history)記載されています。
- OpenShiftのサポートは、含まれるKubernetesのバージョンに依存します。例えば、OpenShift v4.10はKubernetes v1.23を使用しており、Contrastでは2023年2月28日までサポートします。OpenShiftに含まれるKubernetesバージョンについては、Red Hatの[サポート記事](https://access.redhat.com/solutions/4870701)を参照してください。
- Contrastエージェントオペレータは、Linuxのamd64ホスト上での実行のみをサポートし、互換性のないノードでのスケジューリングは拒否されます。また、Contrastエージェントが他のプラットフォームをサポートしている場合でも、エージェントオペレータはLinuxのamd64ホスト上で実行されているワークロードの組込のみをサポートします。Windowsやarm64でのKubernetesのサポートを希望される場合は、[Contrastサポート](https://support.contrastsecurity.com/hc/ja-jp)にお問い合わせください。

## エージェントタイプ

| エージェント                 | エージェントタイプ     | サポート状況 | 互換性情報                                                                                            |
|-----------------------|----------------|----------------|----------------------------------------------------------------------------------------------------------------|
| .NET Core             | dotnet-core    | サポート対象      | [.NET Coreのサポート対象テクノロジ](https://docs.contrastsecurity.jp/ja/-net-core-supported-technologies.html) |
| Java                  | java           | サポート対象     | [Javaのサポート対象テクノロジ](https://docs.contrastsecurity.jp/ja/java-supported-technologies.html)           |
| NodeJS                | nodejs         | ベータ版           | [Node.jsのサポート対象テクノロジ](https://docs.contrastsecurity.jp/ja/node-js-supported-technologies.html )     |
| NodeJS (Protectのみ) | nodejs-protect | ベータ版          | [Node.jsのサポート対象テクノロジ](https://docs.contrastsecurity.jp/ja/node-js-supported-technologies.html )     |
| PHP                   | php            | ベータ版           | [PHPのサポート対象テクノロジ](https://docs.contrastsecurity.jp/ja/php-supported-technologies.html)             |

- Node.jsとPHPアプリケーションでのエージェントの組み込みはベータ版です。ベータ版は機能が変更されたり、予期しない動作をしたりする可能性があります。この機能を利用することで、お客様は[Contrastベータ版利用規約](https://docs.contrastsecurity.jp/ja/beta-terms-and-conditions.html "Contrast Beta Terms and Conditions")に同意することになります。
- Node.jsエージェントを組み込むと、エージェントを組み込まれたアプリケーションの起動時間が大幅に増加する場合があります。起動時間が許容できない場合は、コンパイル時にエージェントを組み込むことをお勧めします。コンパイル時にNode.jsエージェントを組み込む場合は、オペレータによるランタイム時の組み込みは無効化して下さい。詳しくは、[リライターCLI](https://docs.contrastsecurity.jp/ja/node-js-agent-rewriter-cli.html)を参照してください。