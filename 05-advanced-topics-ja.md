# 高度なトピック

## .NET Coreエージェントによるプロファイラチェーンのサポート

.NET Coreエージェントは、Contrastエージェントオペレータと組み合わせて、[Dynatraceオペレータ](https://github.com/Dynatrace/dynatrace-operator)を使用するDynatraceでのプロファイラチェーンをサポートします。Dynatraceオペレータを使用してDynatraceエージェントをワークロードに組み込むと、自動的にサポートが有効になります。

| ベンダ    | バージョン         | サポート対象の検証日 |
|-----------|-----------------|----------------------|
| Dynatrace | Operator v0.6.0 | 2022年6月9日           |

今後のDynatraceのバージョンによっては、チェーンが壊れる可能性があります。チェーンによって非互換性が生じる可能性があり、`agent.dotnet.enable_chaining: false`オプションを使用して無効にすることができます。
