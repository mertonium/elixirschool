---
version: 1.1.0
title: 並行性
---

Elixir の売りの 1 つは並行性(コンカレンシー)に対応していることです。 Erlang VM (BEAM)のおかげで、並行処理は Elixir では思ったよりも簡単です。並行性のモデルはアクターに依存しています。アクターとはメッセージパッシングによって他のプロセスと相互通信を行う、制御されたプロセスのことです。

このレッスンでは Elixir とともに納められている並行モジュールを見ていきます。続く章では並行モジュールを実装している OTP の振舞を取り扱います。

{% include toc.html %}

## プロセス

Erlang VM (BEAM)内のプロセスは軽量で、全ての CPU 間で実行されます。ネイティブスレッドのように見えますがそれより単純ですし、 Elixir アプリケーション内に数千もの並行プロセスを持つことは珍しくありません。

新しいプロセスを作る最も簡単な方法は、匿名/名前付き関数を引数に取る `spawn` です。新しいプロセスが作られると、 _プロセス識別子_ 別名 PID が返ります。これはアプリケーション内部でプロセスを一意に識別するものです。

手始めに、モジュールを作り実行したい関数を定義します:

```elixir
defmodule Example do
  def add(a, b) do
    IO.puts(a + b)
  end
end

iex> Example.add(2, 3)
5
:ok
```

この関数を非同期的に評価するには、 `spawn/3` を用います:

```elixir
iex> spawn(Example, :add, [2, 3])
5
#PID<0.80.0>
```

### メッセージパッシング

相互通信をするのに、プロセスはメッセージパッシングを頼ります。これには 2 つの主な部品、 `send/2` と `receive` があります。 `send/2` 関数を使うとメッセージを PID 宛に送ることができます。受信するには `receive` を使ってメッセージをマッチし、マッチするものが見つからない場合は中断されずに実行中のままとなります。

```elixir
defmodule Example do
  def listen do
    receive do
      {:ok, "hello"} -> IO.puts("World")
    end

    listen
  end
end

iex> pid = spawn(Example, :listen, [])
#PID<0.108.0>

iex> send pid, {:ok, "hello"}
World
{:ok, "hello"}

iex> send pid, :ok
:ok
```

`listen/0` 関数が再帰的であることに気づいたかもしれません。これはプロセスで複数のメッセージを扱うためです。再帰がないと、プロセスは初めのメッセージを受け取り処理した後に終了するでしょう。

### プロセスのリンク

`spawn` の問題の 1 つとして知られているのは、プロセスが強制終了した場合です。これに備えるためには、複数プロセスを `spawn_link` でリンクする必要があります。二者間でリンクしたプロセスはプロセス終了の通知をお互いに受け取ります(訳注: つまりどちらのプロセスも終了します。):

```elixir
defmodule Example do
  def explode, do: exit(:kaboom)
end

iex> spawn(Example, :explode, [])
#PID<0.66.0>

iex> spawn_link(Example, :explode, [])
** (EXIT from #PID<0.57.0>) evaluator process exited with reason: :kaboom
```

リンクしたプロセスに現在のプロセスを強制終了させたくない場合もあるでしょう。そのためには `Process.flag/2` 使用して終了を捕捉する必要があります。この関数は `trap_exit` フラグのために Erlang の [process_flag/2](http://erlang.org/doc/man/erlang.html#process_flag-2) 関数を使います。終了を捕捉すると(`trap_exit` が `true` になっている)、 `{:EXIT, from_pid, reason}` というタプルのメッセージとして受け取ります。

```elixir
defmodule Example do
  def explode, do: exit(:kaboom)

  def run do
    Process.flag(:trap_exit, true)
    spawn_link(Example, :explode, [])

    receive do
      {:EXIT, from_pid, reason} -> IO.puts("Exit reason: #{reason}")
    end
  end
end

iex> Example.run
Exit reason: kaboom
:ok
```

### プロセス監視

2 つのプロセスをリンクせずに、しかし通知はそのまま維持したいとすると、どうでしょうか。そうした場合には `spawn_monitor` でプロセス監視を用いることができます。プロセスを監視すると、プロセスが強制終了した際にメッセージを受け取ります。現在のプロセスを強制終了させたり、プロセス終了時に明示的な捕捉をする必要はありません。

```elixir
defmodule Example do
  def explode, do: exit(:kaboom)

  def run do
    {pid, ref} = spawn_monitor(Example, :explode, [])

    receive do
      {:DOWN, ref, :process, from_pid, reason} -> IO.puts("Exit reason: #{reason}")
    end
  end
end

iex> Example.run
Exit reason: kaboom
:ok
```

## Agent

Agent は状態を維持するバックグラウンドプロセス周りを抽象化したものです。アプリケーションやノードの内部で他のプロセスから Agent に接続することができます。 Agent の状態は関数の戻り値に与えられます:

```elixir
iex> {:ok, agent} = Agent.start_link(fn -> [1, 2, 3] end)
{:ok, #PID<0.65.0>}

iex> Agent.update(agent, fn (state) -> state ++ [4, 5] end)
:ok

iex> Agent.get(agent, &(&1))
[1, 2, 3, 4, 5]
```

Agent に名前を付けると、 PID の代わりに名前で参照することができます:

```elixir
iex> Agent.start_link(fn -> [1, 2, 3] end, name: Numbers)
{:ok, #PID<0.74.0>}

iex> Agent.get(Numbers, &(&1))
[1, 2, 3]
```

## Task

Task は関数をバックグラウンドで実行し、後でその戻り値を受け取る方法を提供します。アプリケーションの動作を妨げることなく、実行コストの高い演算を処理する時に特に役立てることができます。

```elixir
defmodule Example do
  def double(x) do
    :timer.sleep(2000)
    x * 2
  end
end

iex> task = Task.async(Example, :double, [2000])
%Task{pid: #PID<0.111.0>, ref: #Reference<0.0.8.200>}

# 何か処理を行う

iex> Task.await(task)
4000
```
