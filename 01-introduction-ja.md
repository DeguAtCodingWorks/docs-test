# はじめに
> 重要
>
> この機能はベータ版です。ベータ版は機能が変更されたり、予期しない動作をしたりする可能性があります。この機能を利用することで、お客様は[Contrastベータ版利用規約](https://docs.contrastsecurity.com/en/beta-terms-and-conditions.html "Contrast Beta Terms and Conditions")に同意することになります。

Contrastエージェントオペレータは、KubernetesクラスタおよびOpenShiftクラスタ内で実行される標準の[Kubernetesオペレータ](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)で、既存のワークロードへのContrastエージェントの組み込み、組み込まれたエージェントの設定、およびエージェントのアップグレードを自動化します。

まずは、[セットアップ](./setup/01-installation.md)のセクションをご覧いただき、[全体の例](./04-full-example.md)をご覧ください。

オペレータは、宣言的なKubernetesネイティブのリソースタイプを使用して設定します。リソースタイプについては、[設定リファレンス](./03-configuration-reference.md)に記載しています。
