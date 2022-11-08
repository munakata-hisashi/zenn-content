---
title: "Sourcery で Swift のイニシャライザを生成する"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Swift"]
published: false
---

Swift で DTO  などを定義するとき以下のようなイニシャライザを定義したいときがあります

```swift
public struct UserDTO {
    public let userId: String
    public let name: String
    public let installationId: String
    public let platform: String
    public let version: Int

    public init(userId: String, name: String, installationId: String, platform: String, version: Int) {
        self.userId = userId
        self.name = name
        self.installationId = installationId
        self.platform = platform
        self.version = version
    }
}
```

Swift の struct にはイニシャライザを定義していなくても、メンバーワイズイニシャライザという型が持っているプロパティを引数にとるイニシャライザが自動的に定義されるので初期化することができます。しかしメンバーワイズイニシャライザのアクセスレベルは、プロパティのアクセスレベルが private や fileprivate ならそれと同じになり、その他の場合は internal になります。そのため別のモジュールから初期化したい場合は、上記のような public なイニシャライザを定義する必要があります。

プロパティがたくさんあったりすると書くのが面倒です。Soucery というライブラリを使って struct を定義したらビルド時に勝手に生成されるようにします。

## Sourcery

[Sourcery](https://github.com/krzysztofzablocki/Sourcery) は 定義したテンプレートファイルにしたがって Swift のコードを生成してくれるライブラリです。以下のようにソースファイル、テンプレートファイル、生成する出力先のパスを指定して実行します。また、オプションは .sourcery.yml に書いておくと省略できます。

```bash
$ ./sourcery --sources <sources path> --templates <templates path> --output <output path>
```

```yaml
sources:
  - <sources path>
  - <sources path2>
templates:
  <templates path>
output:
  -<output path>
```

### Sourcery で使うプロトコルを定義

最初に Sourcery で利用するプロトコルを定義しておきます。

```swift
protocol AutoInitializable { }
```

### Stencil を使ってテンプレートファイルを書く

テンプレートファイルは [Stencil](https://github.com/stencilproject/Stencil) というテンプレート言語を使って記述します。Sourcery のリポジトリにいくつかテンプレートの例が上がっているので、目的によってはこれをそのまま使うこともできそうです。[https://github.com/krzysztofzablocki/Sourcery/tree/master/Templates/Templates](https://github.com/krzysztofzablocki/Sourcery/tree/master/Templates/Templates)

今回のイニシャライザ生成では自分で .stencil ファイルを書いていきます。

Stencil の基本的な記法は以下の 3 つです。

- {{ ... }} 変数を定義することができます。
- {% ... %} ドキュメントではタグという名前で、if や for などの制御構文が使えたりします。
- {# ... #} コメントを書けます。

今回は以下のようなテンプレートファイルを書きました。

```yaml
{% for type in types.implementing.AutoInitializable %}
{% map type.storedVariables into parameters using p %}{{p.name}}: {{p.typeName }}{% endmap %}
// sourcery:inline:{{ type.name }}.AutoInitializable
{{ type.accessLevel }} init({{parameters|join:", "}}) {
    {% for parameter in type.storedVariables %}
    self.{{ parameter.name }} = {{ parameter.name }}
    {% endfor %}
}
// sourcery:end
{% endfor %}
```

1行目の types は Sourcery で定義された Types 型になっていて types.implementing.プロトコル名 とすると、ソースコード中で、指定したプロトコルに準拠している型のリストを返してくれます。変数 type は Sourcery で定義されている Type 型になります。

2行目では、取得した型の引数の情報(storedVariables)から引数の名前と型名を取り出し、引数名: 型名 の形に整形しています。

Sourcery では他にもいろんな型が定義されています。[https://cdn.rawgit.com/krzysztofzablocki/Sourcery/master/docs/Types.html](https://cdn.rawgit.com/krzysztofzablocki/Sourcery/master/docs/Types.html)

また、Stencil そのものには map の機能はないですが、Sourcery では [StencilSwiftKit](https://github.com/SwiftGen/StencilSwiftKit) という Stencil の機能を拡張するライブラリを使って実現しているようです。

3行目以降では実際に出力したいコードを Type 型と、2行目で整形した型の情報を使って書いています。

通常では output オプションで指定したパスに新規のファイルとして生成されますが、今回はイニシャライザを生成したいので、元のソースコードにテンプレートから生成したコードを追加したいです。

```yaml
// sourcery:inline:{{ type.name }}.AutoInitializable
// sourcery:end
```

上の2行をテンプレートに追加し、ソースコードを出力させたい部分に以下のようにコメントをします。こうすると、コメントした部分の間に生成したコードをいれることができます。また、最初に定義した AutoInitializable プロトコルに準拠させて、Stencil ファイルでの処理を実行できるようにします。

```swift
public struct UserDTO: AutoInitiaizable {
    public let userId: String
    public let name: String
    public let installationId: String
    public let platform: String
    public let version: Int

    //sourcery:inline:AudienceInfo.AutoInitializable
    //sourcery:end
}
```

### 実行する

Xcode の Build phases に sourcery コマンドを実行するスクリプトを追加することで、ビルド後に以下のようなイニシャライザが追加されます。プロトコルへの準拠とコメントの追加をするだけで、イニシャライザを自動で生成できるようになりました。

```swift
public struct UserDTO: AutoInitiaizable {
    public let userId: String
    public let name: String
    public let installationId: String
    public let platform: String
    public let version: Int

    //sourcery:inline:AudienceInfo.AutoInitializable
    public init(userId: String, name: String, installationId: String, platform: String, version: Int) {
        self.userId = userId
        self.name = name
        self.installationId = installationId
        self.platform = platform
        self.version = version
    }
    //sourcery:end
}
```