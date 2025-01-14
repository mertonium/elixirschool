---
version: 1.0.3
title: ドキュメント
---

Elixir コードのドキュメント。

{% include toc.html %}

## 注釈

どれくらいコメントを書くとか、何がドキュメントの質を決めるとかいう話はプログラミングの世界では結論の出ていない問題です。しかし自身やそのコードベースに関わる人たちにとって大切だという点では、誰もが同意できるでしょう。

Elixir はドキュメントを _第一級市民_ として扱い、プロジェクトのドキュメントにアクセスしたり生成したりするために各種の関数を用意しています。Elixir コアはコードベースに注釈を付けるために多くの異なった属性を提供しています。3 つの方法を見ていきましょう:

- `#` - インラインドキュメント用。
- `@moduledoc` - モジュールレベルのドキュメント用。
- `@doc` - 関数レベルのドキュメント用。

### インラインドキュメント

おそらく最も単純にコードコメントを付ける方法はインラインコメントを用いることです。Ruby や Python と同じように、Elixir のインラインコメントは `#` で示します。あなたがいた環境によって、よく*パウンド* や _ハッシュ_ などとして知られています。

この Elixir スクリプトを見てください (greeting.exs):

```elixir
# コンソールに 'Hello, chum.' と出力します。
IO.puts "Hello, " <> "chum."
```

Elixir はこのスクリプトを実行する際に、 `#` からその行末までを全て無視し、不要なデータとみなします。スクリプトの処理やパフォーマンスには影響しないかもしれませんが、コメントを読んだプログラマが挙動を理解するにはあまり明確ではないでしょう。単一行コメントの乱用は避けるように心がけてください！コードベースへのポイ捨ては誰かにとって好ましくない悪夢となるかもしれません。控えめに用いるのが最良です。

### モジュールのドキュメント

`@moduledoc` アノテータはインラインドキュメントをモジュールレベルで行うためのものです。通常はファイル最上部で `defmodule` の宣言をした直後に置かれます。次の例は `@moduledoc` 修飾子内の 1 行コメントを示します。

```elixir
defmodule Greeter do
  @moduledoc """
  ある人を歓迎する `hello/1` 関数を提供します
  """

  def hello(name) do
    "Hello, " <> name
  end
end
```

このモジュールドキュメントには、IEx 内で `h` ヘルパー関数を用いてアクセスすることができます。

```elixir
iex> c("greeter.ex")
[Greeter]

iex> h Greeter

                Greeter

ある人を歓迎する `hello/1` 関数を提供します
```

### 関数のドキュメント化

Elixir がモジュールレベルで注釈する機能を与えてくれるのと同じく、関数をドキュメント化するためにも似たような注釈が可能です。 `@doc` アノテータは関数レベルでのインラインドキュメントを行うためのものです。 `@doc` アノテータは注釈を付けたい関数の直前に置かれます。

```elixir
defmodule Greeter do
  @moduledoc """
  ...
  """

  @doc """
  Hello メッセージを表示します

  ## パラメータ

    - name: 人名を表現する文字です。

  ## 例

      iex> Greeter.hello("Sean")
      "Hello, Sean"

      iex> Greeter.hello("pete")
      "Hello, pete"

  """
  @spec hello(String.t()) :: String.t()
  def hello(name) do
    "Hello, " <> name
  end
end
```

IEx を再び起動し、上記の関数をモジュール名付きで ヘルパーコマンド (`h`) に渡すと、次のような結果が得られるはずです:

```elixir
iex> c("greeter.ex")
[Greeter]

iex> h Greeter.hello

                def hello(name)

Hello メッセージを表示します

パラメータ

  • name: 人名を表現する文字です。

例

    iex> Greeter.hello("Sean")
    "Hello, Sean"

    iex> Greeter.hello("pete")
    "Hello, pete"

iex>
```

ドキュメント内でマークアップが使え、ターミナルがきちんと描画したことに気づきましたか？本当にクールで、Elixir の広大なエコシステムへの素晴らしい追加機能であることは置いておくとしても、ExDoc がその場で HTML ドキュメントを生成してくれるのを見ると、人はより興味を持ちます。

**注:** `@spec` アノテーションはコードの静的解析に使われます。詳しい説明は[仕様と型](../../advanced/typespec)を参照してください。

## ExDoc

ExDoc は公式の Elixir プロジェクトで、[GitHub](https://github.com/elixir-lang/ex_doc) で見つけることができます。ExDoc は **HTML (HyperText Markup Language) とオンラインドキュメント** を生成します。最初に、アプリケーションのために Mix プロジェクトを作成しましょう:

```bash
$ mix new greet_everyone

* creating README.md
* creating .gitignore
* creating .formatter.exs
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/greet_everyone.ex
* creating test
* creating test/test_helper.exs
* creating test/greet_everyone_test.exs

Your Mix project was created successfully.
You can use "mix" to compile it, test it, and more:

    cd greet_everyone
    mix test

Run "mix help" for more commands.

$ cd greet_everyone

```

次に `@doc` アノテータのレッスンからコードをコピー & ペーストして、 `lib/greeter.ex` ファイルに保存し、コマンドラインから全てがまだ動くことを確認します。Mix プロジェクト内で作業をするので IEx を先ほどとは若干異なり、 `iex -S mix` コマンドで起動する必要があります:

```bash
iex> h Greeter.hello

                def hello(name)

Hello メッセージを表示します

パラメータ

  • name: 人名を表現する文字です。

例

    iex> Greeter.hello("Sean")
    "Hello, Sean"

    iex> Greeter.hello("pete")
    "Hello, pete"
```

### インストール

全てがうまくいき、上記のような出力が表示されていれば、ExDoc の構築準備ができていることを意味します。 `mix.exs` ファイルに、 `:earmark` と `:ex_doc` の 2 つの必要な依存関係を追加してください:

```elixir
  def deps do
    [{:earmark, "~> 0.1", only: :dev},
    {:ex_doc, "~> 0.11", only: :dev}]
  end
```

`only: :dev`というキーバリューのペアを指定することで、本番環境ではこれらの依存パッケージをダウンロードしたりコンパイルしたりしないようにします。それはそれとして、Earmark はなぜ必要なのでしょう。これは Elixir プログラミング言語向けの Markdown パーサで、ExDoc が `@moduledoc` や `@doc` 内のドキュメントを綺麗な見た目の HTML に変換するためのものです。

ここで注記しておくと、Earmark を使わなくても良いです。 Pandoc や Hoedown 、 Cmark といった他のマークアップツールに変更することもできます。ただし、[ここ](https://github.com/elixir-lang/ex_doc#changing-the-markdown-tool)を読み、若干の追加設定を行う必要があるでしょう。このチュートリアルでは、単に Earmark を用います。

### ドキュメント生成

さて続けましょう。コマンドラインから次の 2 つのコマンドを実行してください:

```bash
$ mix deps.get # ExDoc と Earmark を取得します。
$ mix docs # ドキュメントを作成します。

Docs successfully generated.
View them at "doc/index.html".
```

全てが計画どおりなら、上記の例にあるような出力メッセージが表示されるでしょう。次に、Mix プロジェクト内部を見ていきましょう。 **doc/** ディレクトリがあるはずです。中身は生成されたドキュメントになります。インデックスページをブラウザで開くと、以下のように表示されるはずです:

![ExDoc Screenshot 1]({% asset documentation_1.png @path %})

Earmark がマークダウンを描画し、 ExDoc がわかりやすいフォーマットで表示してくれています。

![ExDoc Screenshot 2]({% asset documentation_2.png @path %})

これで Github や自身のウェブサイト、あるいは非常に一般的な [HexDocs](https://hexdocs.pm/) へデプロイすることができるようになりました。

## ベストプラクティス

ドキュメントの追加は、言語のベストプラクティスガイドラインに加えられるべきです。 Elixir はかなり若い言語なので、エコシステムの成長に伴っていまだに多くの新基準が発見されている状況です。それでも、 Elixir のコミュニティはベストプラクティスの確率に尽力しています。ベストプラクティスについてさらなる詳細を知りたい場合は、[The Elixir Style Guide](https://github.com/niftyn8/elixir_style_guide) をご覧ください。

- モジュールには常にドキュメントを書いてください。

```elixir
defmodule Greeter do
  @moduledoc """
  これは良いドキュメントです。
  """

end
```

- モジュールのドキュメントを書くつもりがない場合は、空のままに**してはいけません**。以下のように、モジュールに `false` の注釈を付けることを検討してください:

```elixir
defmodule Greeter do
  @moduledoc false

end
```

- モジュールのドキュメントで関数に言及する場合は、以下のようにバッククォートを使用してください:

```elixir
defmodule Greeter do
  @moduledoc """
  ...

  このモジュールには `hello/1` 関数もあります。
  """

  def hello(name) do
    IO.puts("Hello, " <> name)
  end
end
```

- 以下のように、 `@moduledoc` 以下のコードは 1 行空けて区別してください:

```elixir
defmodule Greeter do
  @moduledoc """
  ...

  このモジュールには `hello/1` 関数もあります。
  """

  alias Goodbye.bye_bye
  # などなど...

  def hello(name) do
    IO.puts("Hello, " <> name)
  end
end
```

- 関数内で markdown を使うと、 IEx や ExDoc で読みやすくなります。

```elixir
defmodule Greeter do
  @moduledoc """
  ...
  """

  @doc """
  Hello メッセージを表示します

  ## パラメータ

  - name: 人名を表現する文字です。

  ## 例

      iex> Greeter.hello("Sean")
      "Hello, Sean"

      iex> Greeter.hello("pete")
      "Hello, pete"

  """
  @spec hello(String.t()) :: String.t()
  def hello(name) do
    "Hello, " <> name
  end
end
```

- ドキュメントにはいくつかコード例も含むように心がけてください。そうすることで、[ExUnit.DocTest][] を用いてモジュールや関数、マクロ内にあるコード例から自動テストを生成することもできるようになります。これには、テストケースから `doctest/1` マクロを呼び出し、ガイドラインに沿った例を書く必要があります。詳細は[公式ドキュメント][exunit.doctest] にあります。

[exunit.doctest]: https://hexdocs.pm/ex_unit/ExUnit.DocTest.html
