# サービス

<!--original
# Services
-->

タスクやストラタジー、バイナリなどを作成する際、Rocketeerは作業を簡単にしたりアクセスするための様々なサービスを提供しています。それらの全ては公開APIとして記載されていますが、そのうちのいくつかは特に重要です。ここでは、Rocketeerが何を提供しているか見て、内部でどのように動作しているか理解していきたいと思います。

<!--original
When creating tasks, strategies, binaries, etc., Rocketeer provides you with a variety of services you can access and use to ease your work. While all of them are describe in the public API, some are more important than others. This is a look at what Rocketeer provides you with, and gives you a little more insight on how things work under the hood.
-->

提供される全ての例は、カスタムタスクを想定しています。ただ、これはクロージャタスクでも同じように適用されます。つまり、もしタスククラスの中で`$this->foo->bar()`のようにできるとしたら、クロージャの中であれば次のようになります。

<!--original
All examples provided are from the viewpoint of a custom task. But it also applies to closure tasks, meaning if within a task class you can do `$this->foo->bar()`, within a closure you can do this:
-->

```php
Rocketeer::before('deploy', function (AbstractTask $task) {
  $task->foo->bar();
});
```

それぞれのサービスの隣の括弧内の文字列は、それを通じてアクセス可能なハンドルです。

<!--original
The string between brackets next to each service is the handle it can be access through.
-->

## タスクビルダー [builder]

<!--original
## Tasks Builder [builder]
-->

タスクビルダーはRocketeerの最も重要なサービスのひとつです。タスクから、簡単にバイナリー、タスク、ストラテジなどのインスタンスを得るための強力な手段です。許されているどんなタスクでもビルドすることができます: ワンライナー、クロージャ、インスタンス、クラス名など。

<!--original
The tasks builder is one of Rocketeer's most important services, its a powerful factory that allows you to quickly get instances of binaries, tasks, strategies and so on, from within your tasks. It can build tasks from anything that is allowed: one-liners, closures, instances, or class names.
-->

```php
// Build a task
$this->builder->buildTask('ls');

$this->builder->buildTask('Acme\MyCustomTask');

$this->builder->buildTask(function ($task) {
  return $task->run('ls');
}, 'My task', 'Lists the folders');

// Build multiple tasks
$this->builder->buildTasks(array(
  'ls',
  'Acme\MyCustomTask',
));

// Build the configured implementation of a strategy
$this->builder->buildStrategy('Deploy');

// Build a particular implementation of a strategy
$this->builder->buildStrategy('Deploy', 'Clone');

// Build the Command instance related to a Task
$this->builder->buildCommand('Deploy'); // DeployCommand
```

## タスクキュー [queue]

<!--original
## Tasks Queue [queue]
-->

タスクキューは、Rocketeerのもう一つの主だったクラスです。タスクの配列を受け取って、それから実行可能なパイプラインを構築します。同じ種類のタスクタイプ(クロージャ、文字列、クラス名など)を渡せるよう、全てはTasksBuilderを通して渡されます。

<!--original
The tasks queue is the other major class of Rocketeer, it receives an array of tasks, and builds a runnable pipeline from it. Anything it receives is passed through the TasksBuilder so you can pass the same kind of tasks types (closures, strings, class names, etc.) as above.
-->

```php
// Run an array of tasks
$this->queue->run(['cd /foo/bar', 'ls']);

// Run an array of tasks and return the last line
// You can specify the connection as second argument
$files = $this->queue->execute(['cd /foo/bar', 'ls'], 'production');

// Same as above but with reversed arguments
$files = $this->queue->on('production', ['cd /foo/bar', 'ls']);
```

`run`メソッドは`Rocketeer\Services\Tasks\Pipeline`のインスタンスを返します。`Pipeline`は`Illuminate\Support\Collection`を拡張したクラスで、`Rocketeer\Services\Tasks\Job`のインスタンスを保持します。

<!--original
The `run` method will return an instance of `Rocketeer\Services\Tasks\Pipeline`. The Pipeline is a class extending `Illuminate\Support\Collection` that stores instances of `Rocketeer\Services\Tasks\Job`.
-->

Rocketeerでは、ジョブとは実行に必要な情報を持つひとまとまりのキューです: キューがどんな接続を必要とするのか、どのステージか、どのサーバか、etc. あなたが、二つの接続、`production` と `staging`を持っていると想像してください。次のようなパイプラインを得るでしょう:

<!--original
Within Rocketeer, a Job is a bundled version of a queue, containing all the necessary information required to run it: what connection the queue needs to be run on, what stage, what server, etc. Imagine you have two connections, `production` and `staging`, you'll get the following pipeline:
-->

```php
$pipeline = $this->queue->run(['cd /foo/bar', 'ls']);

$firstJob = $pipeline->first(); // Job instance
$firstJob->connection; // production
$firstJob->queue; // [ClosureTask(cd /foo/Bar), ClosureTask(ls)]

$secondJob = $pipeline->get(1); // Job instance
$firstJob->connection; // staging
$firstJob->queue; // [ClosureTask(cd /foo/Bar), ClosureTask(ls)]
```

一度パイプラインが構築されると、`--pretend`フラグが渡されるか次第で、同期的か非同期的に実行されます。

<!--original
Once the pipeline is built it is either run synchronously or asynchronously depending on whether the `--pretend` flag was passed.
-->

## タスクハンドラ [tasks]

<!--original
## Tasks Handler [tasks]
-->

タスクハンドラは関連するタスクやプラグイン、イベントを登録します。それが`Rocketeer`のファサードの後ろの主なタスクなので、そのメソッド(before, after, listenTo, task)のほとんどに精通している必要があります:

<!--original
The tasks handler registers the tasks, plugins, and events that surrounds them. It's the main task behind the `Rocketeer` facade so you should be familiar with most of its methods (before, after, listenTo, task):
-->

```php
// Register tasks and events
$this->tasks->task('list-files', 'ls', 'Lists files in a folder');

$this->tasks->before('Deploy', 'list-files');

// Get bound events
$events = $this->tasks->getTasksListeners('deploy', 'before');
```

## ステップビルダー [steps()]

<!--original
## Steps Builder [steps()]
-->

ステップビルダーはタスクに一連の内部的なコマンドや呼び出しを実行させ、最初の失敗で停止できるようにするものです。インターフェースの使いやすさはそのままに。次のような、架空のタスクの中の一連のコマンドを見てみましょう:

<!--original
The steps builder is what allows tasks to run a series of internal commands and calls and halts on the first failure, while keeping a fluent interface. Take the following series of commands inside an imaginary task:
-->

```php
$this->executeTask('Migrate');
$this->symlink('foo', 'bar');
```

ここで、Migrateタスクが失敗した場合は、symlinkを実行したくないとしたら、`steps()`を単純に前につけます:

<!--original
Now ideally we would want to not execute the symlink if the Migrate task fails, for this, we simply prepend those calls with `steps()`:
-->

```php
$this->steps()->executeTask('Migrate');
$this->steps()->symlink('foo', 'bar');
```

これで、これらの「ステップ」はタスクの`steps`プロパティに保持されます。一度完了すれば、安全に順番を追ってそれらを実行することができます。ステップが失敗しない限りは。

<!--original
This will store those "steps" within a `steps` property on the tasks. Once this is done we can safely run them sequentially until one fails:
-->

```php
$this->steps()->executeTask('Migrate');
$this->steps()->symlink('foo', 'bar');

// Run the steps until one fails
if (!$this->runSteps()) {
	return $this->halt();
}
```

## キューの説明 [explainer]

<!--original
## Queue Explainer [explainer]
-->

キューの「説明(explainer)」は、RocketeerのCLI出力をコントロールするものです。ユーザに何が起きているかを説明し、どのタスクがどのタスクやイベントによって引き起こされ、何が進行しているか、その結果、そして実行にどのくらいの時間がかかるのか、など。

<!--original
The queue explainer is what drives Rocketeer's CLI output, its job is to explain to the user what is happening, what tasks are fired by what task or event, what is their progress, their result, how long they took to execute, etc.
-->

デフォルトでは、タスクの実行時、「説明」はあなたがセットした`name`と`description`プロパティから、自動的に情報を表示します。ですが、タスクのフローの中で「説明」に追加の詳細を提供することも可能です:

<!--original
By default when executing a task, the explainer will automatically display information about it from the `name` and `description` property you set on it (or passed to `Rocketeer::task`). But you can provide additional details to the explainer during the flow of your task:
-->

```php
$this->explainer->line('Doing some stuff');
if (!$this->somethingThatMayFail()) {
  $this->explainer->error('Something crashed here');
}

$this->explainer->success('All went well!');
```

「説明」はCommandインスタンスの`line`メソッドに、それを渡しています。つまり、[Symfonyドキュメント](http://symfony.com/doc/current/components/console/introduction.html#coloring-the-output)に記述された前景色と背景色を追加することも可能です。

<!--original
The explainer passes that to the Command instance's `line` method, meaning you can add foreground and background colors as described in [Symfony's documentation](http://symfony.com/doc/current/components/console/introduction.html#coloring-the-output):
-->

```php
$this->explainer->line('<info>Something</info> tried and <bg=red>failed</bg=red>');
```

たぶんお気付きのように、「説明」はあなたのタスクの「ツリー」を実行時に構築します。もしタスクやイベントの内部で呼び出されるなら、タスクはネストされます:

<!--original
As you may have noticed, the explainer builds a "tree" of your tasks at runtime, nesting tasks if they are fired within a task, or an event:
-->

```
|-- Running: Deploy (Deploys the website) [~14.25s]
|---- Running: Primer (Run local checks to ensure deploy can proceed)
|---- Running: CreateRelease (Creates a new release on the server) [~5.98s]
|------ Running strategy for Deploy: Sync
```

これは、コマンドをクロージャの中にラッピングすることで可能です。

<!--original
You can do this to by wrapping some commands in a closure:
-->

```php
$this->explainer->displayBelow(function() {
  // Whatever is done here will be displayed one level below in the tree
});
```
