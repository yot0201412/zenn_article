---
title: "Goのanalysisパッケージを使った静的解析を実装する"
emoji: "🦔"
type: "tech" 
topics: ["go", "静的解析"]
published: true
published_at: 2023-12-09 09:00
publication_name: "micin"
---


この記事は [MICIN Advent Calendar 2023](https://adventar.org/calendars/9595) の 9 日目の記事です。
前回は外山さんの[LINEプラットフォームの導入と活用](https://zenn.dev/masatomoty/articles/d074c58f7d9eff)でした。

---

## はじめに

MICINでMiROHAのバックエンド開発を担当している小林です。この記事では、Goのanalysisパッケージを用いた静的解析の自前実装についてご紹介します。自分で静的解析を実装する機会がない方も多いと思いますが、楽しんでもらえると嬉しいです。
また、今回静的解析を実装する上で[tenntenn](https://tenntenn.dev/ja/)さんの[こちらの記事](https://zenn.dev/tenntenn/books/d168faebb1a739/viewer/9d590e)を参考にさせてもらいました。
analysisパッケージのAnalyzerを用いた静的解析の利点や実装方法について詳しく説明されていますので、こちらも合わせて見てもらえると良いかと思います。

## Analyzerの出力について

analysisパッケージを用いた実装の紹介に入る前にAnalyzerの出力方法について紹介したいと思います。
Analyzerの出力方法は次の3つがあります。

- Diagnostic
- Result
- Fact

1つ目のDiagnosticは静的解析の結果、検出した箇所を出力したいときに利用します。Diagnosticは内部にソースコード上の位置（どのファイルの何行目か）を持つことができ、出力時にはそれ情報と共に任意のメッセージを出力できます。そのため、静的解析結果として何かコードが間違えている箇所を指摘したい場合は、Diagnosticを利用して結果を出力することになります。
2つ目のResultですが、こちらは解析ツールを利用するユーザに対して伝える解析結果ではなく、**他Analyzerに伝える解析結果**になります。
最後にFactですが、こちらもResultと同じく他Analyzerと解析内容の情報を共有するためのものです。ただ、Resultが解析**結果**を伝えるものであるのに対して、Factは解析途中で見つけた**事実**を記録し他Analyzerと共有できるものになっています。FactはResultと異なり、パッケージや型情報に紐付けられるという特徴を持っています。
せっかくなので、この3つの出力形式を全て使って静的解析を実装します。

## 実装

今回は例として、「hogeから始まる名前のファイル内のHogeから始まる関数」を検出する静的解析を実装してみます。
Analyzerを用いた静的解析で実装するものは、大きく分けてAnalyzerとmain関数の2種類だけです。この規模ならAnalyzerは1つだけで十分ですが、今回はResultやFactも利用した実装(2つのAnalyzer間で情報の受け渡しがあるもの)を実装したいため、前処理としてhogeから始まる名前のファイルとHogeから始まる関数を見つけるAnalyzerと結果をユーザ向けに出力するAnalyzerの2つに分けて実装していきたいと思います。よって今回実装するものは以下の3つになります。

- 前処理Analyzer(Hogeから始まる関数の場所を記録する)
- 出力Analyzer(ユーザに対して解析結果を出力する)
- main関数

フォルダ構成は下記のようになります。

```
.
├── analyzers
│   ├── output
│   │   └── output.go
│   └── findhoge
│       ├── find_hoge.go
│       ├── find_hoge_test.go
│       └── testdata/src/facttest
│           └── facttest.go
├── go.mod
├── go.sum
└── main.go
```

それではひとつずつ見ていきます。

### 前処理Analyzer

単体のAnalyzerで静的解析する場合であれば、Analyzerのフィールドのうち、Name, Doc, Run, Requiresの4つを設定するだけで静的解析を実装可能です。
今回実装する前処理Analyzerでは、FactとResultを用いて解析結果を出力Analyzerに伝える必要があるため、その4つに加えて、ResultType、FactTypesも設定が必要です。

では、前処理Analyzerの実装を見ていきます。今回はFact(`HogeFuncFact`)をHogeから始まる関数に紐付けます。また、Result(`HogeFileMap`)にhogeから名前が始まるファイル一覧を記録したいと思います。

```go
package findhoge

var Analyzer = &analysis.Analyzer{
  Name: "FindHoge",
  Doc:  "Find hoge",
  Run:  run,
  FactTypes: []analysis.Fact{
    new(HogeFuncFact),
  },
  Requires:   []*analysis.Analyzer{},
  ResultType: reflect.TypeOf(new(HogeFileMap)),
}

type HogeFuncFact struct {}

func (HogeFuncFact) AFact() {}

type HogeFileMap map[*ast.File]struct{}

```

`HogeFuncFact`がFactとして振る舞うために、AFactメソッドを実装してFactインタフェースを満たすようにしています。このAFactメソッドはFactであることを明示的に示すためのものであり、何か特別な実装が必要なものではありません。
続いて、run関数について見ていきたいと思います。

```go
func run(pass *analysis.Pass) (interface{}, error) {
  var res FindTestFuncResult = make(map[*ast.File]struct{})
  for _, f := range pass.Files {
    filePath := pass.Fset.File(f.Pos()).Name()
    fileName := filepath.Base(filePath)
    if strings.HasPrefix(fileName, "hoge") {
      res[f] = struct{}{}
    }

    for _, decl := range f.Decls {
      if decl, ok := decl.(*ast.FuncDecl); ok && strings.HasPrefix(decl.Name.Name, "Hoge") {
        if obj, ok := pass.TypesInfo.Defs[decl.Name].(*types.Func); ok {
          pass.ExportObjectFact(obj, new(HogeFuncFact))
        }
      }
    }
  }
  return &res, nil
}
```

4, 5行目では、`*ast.File`から直接ファイル名を取得できないため、一度Fsetからファイルパスを取得し、そこからファイル名を取得するという処理を行なっています。

`File.Decls`にファイル内で宣言されているもの、例えばimport, var, type, 関数などが格納されています。その中で今回は関数を取得したいので、型アサーションで関数の宣言かどうか判別しています。あとは、関数名の先頭がHogeから始まる関数があった場合は、そのファイルに存在することをResultに記録し、型情報にFactを紐付けています。

### 出力Analyzer

続いて、出力Analyzerも見ていきます。

```go
package output

var Analyzer = &analysis.Analyzer{
  Name: "outputHogeFunc",
  Doc:  "Output Hoge func",
  Run:  run,
  Requires: []*analysis.Analyzer{
    findhoge.Analyzer,
  },
}

func run(pass *analysis.Pass) (interface{}, error) {
  result := pass.ResultOf[findhoge.Analyzer].(*findhoge.HogeFileMap)
  if len(*result) == 0 {
    return nil, nil
  }
  for _, f := range pass.Files {
    _, ok := (*result)[f]
    if !ok {
      continue
    }
    for _, decl := range f.Decls {
      if decl, ok := decl.(*ast.FuncDecl); ok {
        if obj, ok := pass.TypesInfo.Defs[decl.Name].(*types.Func); ok {
          isHogeFunc := pass.ImportObjectFact(obj, new(findhoge.HogeFuncFact))
          if isHogeFunc {
            pass.Report(analysis.Diagnostic{
              Pos: decl.Pos(),
              Message: "This is Hoge function.",
            })
          }
        }
      }
    }
  }
  return nil, nil
}

```

こちらは前処理Analyzerに比べて設定するフィールドは少ないです。使用するFactも返却するResultがないため、ResultTypeとFactTypesを設定していません。代わりに前処理Analyzer(`findhoge.Analyzer`)の解析出力を利用したいため、依存関係(Requires)の設定が必要になります。
run関数は、初めに`pass.ResultOf`で`HogeFileMap`を取得しています。hogeから始まる名前のファイルだった場合は、先ほどと同様、関数定義をイテレートしながら`pass.ImportObjectFact()`を使い型情報に`HogeFuncFact`が紐付けられているか確認しています。Hoge関数があった場合は、`pass.Report(d analysis.Diagnostic)`を使って静的解析結果をユーザに伝えています。
ちなみに、`domain/model/hogefuga.go`ファイルで検出された場合、以下のように出力されます。

```shell
domain/model/hogefuga.go:13:6: This is Hoge function.
```

実装者が指定せずともファイル名や行数を出力してくれる点がとても親切です。

### main関数

```go
package main

func main() {
  unitchecker.Main(
    findhoge.Analyzer,
    output.Analyzer,
  )
}
```

こちらはとてもシンプルで、`golang.org/x/tools/go/analysis/unitchecker`パッケージが提供しているMain関数に実行したいAnalyzerを引数として渡すだけになります。また、単一のAnalyzerからなる静的解析の場合は、`golang.org/tools/go/analysis/singlechecker`を用いることもできます。

### ビルドと実行方法

使い方はとても簡単で、build後、`go vet`から実行可能です。

```shell
# build
go build -o find-hoge-analysis .
# 実行
go vet -vettool=find-hoge-analysis ./...
```

## テスト

Analyzerはテストを書くための仕組みが備わっているため、やってみたいと思います。
今回は例として、`findhoge.Analyzer`の`FindHogeFact`のテストを書いてみます。テストでは`"golang.org/x/tools/go/analysis/analysistest"`パッケージを利用します。
Analyzerのテストは、`testdata`フォルダ配下に配置した**パッケージ**に対して行います。下記の例では、`analysistest.TestData()`で`testdata`フォルダのパスを取得し、そのフォルダ配下にある`facttest`パッケージを使って`findhoge.Analyzer`のテストを行なっています。

```go
package findhoge_test

func Test(t *testing.T) {
  testData := analysistest.TestData()
  analysistest.Run(t, testData, findhoge.Analyzer, "facttest")
}
```

テスト対象パッケージのファイルは下記のように記載します。Analyzerの実行結果としてFactが紐づいてほしい箇所に、コメントで`want 対象の名前: "FactのString()の出力結果"`と記載することでアサーションできます。
Factは何も設定しないと`String()`で`&{}`を返却してしまうため、わかりやすい名前を返却するよう設定しておくと良いです。

```go
package findhoge

func (*HogeFuncFact) String() string { return "hogeFuncFact" }
```

```go
package facttest

func HogeSample() {} // want HogeSample:`hogeFuncFact`


func Fuga() {}
```

テストは、通常のテストと同様`go test`から実行可能です。何もコメントがない場所にFactが紐付けばテストに引っかかるので、`Fuga()`を書いておけくだけで、Factが紐づかないケースのテストも可能です。

## まとめ

今回は、Analyzerの出力方法を3種類使いつつ静的解析を実装してみました。
一番に感じたことは、簡単に書けるということでした。あと、このパッケージをGoチームが提供してくれているという安心感も感じます。Testもテスト対象パッケージを用意し、コメントでアサーションの内容を記載するだけなので割と迷わず作成できました。
ただ一方で、実装してみたはいいもののFactとResultの有効的な使い方はまだまだ分からないなとも感じました。他の方の記事や、linterツールの実装を見に行くと`"golang.org/x/tools/go/ast/inspector"`の`inspect.Analyzer`のResultを利用している例が多く見つかります。また、Factが使われている実装例では、[staticcheck](https://staticcheck.dev/)でDeprecateされたコードにFactを紐付けている実装([リンク](\https://github.com/dominikh/go-tools/blob/v0.4.6/analysis/facts/deprecated/deprecated.go))があります。Deprecateされたコードの呼び出しを検知するために、事前にDeprecateされた箇所にFactを紐付けているという良い実装例だと思います。これらの実装から使い方を勉強して、もう少し雰囲気を掴んでみようと思います。

Go公式では、すでに色々なAnalyzerを提供してくれています。興味が湧いた方は、こちらをチェックしてみてください。
[golang.org/x/tools/go/analysis/passes](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes)
自前実装した静的解析をうまく使えば、既存の静的解析だけでは届かないコード品質の標準化が可能です。この記事で少しでも興味を持ってもらえると嬉しいです。

---

MICINではメンバーを大募集しています。
「とりあえず話を聞いてみたい」でも大歓迎ですので、お気軽にご応募ください！

https://recruit.micin.jp/
