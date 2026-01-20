# udonsharp-coding-skill

UdonSharpのコーディングを行う際に、AIエージェントがUdonSharpの知識と設計ノウハウを教えるためのAgent Skills

- UdonSharpでの制限
- ネットワーク同期に関するノウハウ 

が含まれており、何度もAIエージェントに対してUdonSharp特有の問題を教えなくても、これを考慮したコードが書かれるハズ。

## インストール方法

### Claude Code

Pluginとして導入できます。

1. Claude Codeを起動
2. コマンドパレットで以下を入力して、リポジトリを追加
```
/plugin marketplace add kurotori4423/udonsharp-coding-skill
```
3. 以下のコマンドでスキルをインストール
```
/plugin install udonsharp-coding
```

### Claude Code 以外 (GitHub Copilot や CODEXなど)

[OpenSkills](https://github.com/numman-ali/openskills) でのインストールがオススメ。

`Node.js`と`npm`が必要です。

導入したいプロジェクトのルートフォルダで以下のコマンドを実行
```
npx openskills install kurotori4423/udonsharp-coding-skill
```

## 使い方

導入した状態でUdonSharpを書かせたりすれば、エージェントはこのスキルを見てUdonSharpのベストプラクティスに従って実装します。