# タスク

<!--original
# Tasks
-->

タスクはRocketeerの重要なコンセプトの一つです。これまであなたが見てきたコマンドのほとんどは、裏では事前に定義されたタスクを使っています。**Rocketeer\Tasks\Setup** や **Rocketeer\Tasks\Deploy** のようにです。それらの多くはRocketeerによってすでに定義されていますが、Rocketeerに、同じようにあなたのサーバーでカスタムした動作をさせるタスクをあなた自身で追加できます。

<!--original
An important concept in Rocketeer are Tasks. Most of the commands you see right above are using predefined Tasks underneath, like **Rocketeer\Tasks\Setup** or **Rocketeer\Tasks\Deploy**. A lot of those are already defined by Rocketeer but you can add your own tasks to Rocketeer to execute custom actions on your server too.
-->

一つのタスクは以下の三つから一つのことができます。

<!--original
A task can be one of a three things:
-->

- 最新リリースフォルダで動作する単純な一行のコマンド。`composer install`や一行コマンドを寄せ集めたコマンドなど.
- より高度な動作を許容するRocketeerのコアヘルパへのアクセスを与えるクロージャー。
- そして、最後は完全に快適なコントロールが与えられる`Rocketeer\Abstracts\AbstractTask`を継承したクラス。すべてのカスタムメイドのタスクは少なくとも一つ`execute`メソッドを保つ必要があります。以上です。

<!--original
- A simple one-line command which will be executed in the latest release's folder. Like `composer install` or an array of one-line commands.
- A closure, giving you access to Rocketeer's core helpers which allow you to perform more advanced actions.
- And finally a class, extending the `Rocketeer\Abstracts\AbstractTask` class, giving you full at-home control. All custom-made Tasks must have at least an `execute` method. And that's all.
-->

_これは意図的にですが、_レベルごとに、あとちょっとのコントロールと快適さをあなたに与えてくれます。もしあなたがクロージャーが与えてくれるものよりもコントロールを必要とするならば、多分クラスが必要なのです。

<!--original
Each level gives you a little more control and comfort - _this is intentional_. If you need more control than what Closures give you then you probably need a class.
-->

-----

<!--original
-----
-->

## Rocketeerのタスク内でのフック

<!--original
## Hooking into Rocketeer's Tasks
-->

多くのユーザーがやるであろうことは、すでにあるRocketeerのタスクに、フックをすることです。Rocketeerは、- 最低限 -タスク実行の前と後に実行可能なビルトインのイベントシステムを提供しています。

<!--original
What most users will do is to hook into existing Rocketeer's Tasks. Rocketeer provides a built-in event system that allows you - at minimum - to execute actions before or after a Task's execution.
-->

### 設定ファイルへのタスクの定義

<!--original
### Defining Tasks in the config file
-->

Rocketeerの設定ファイルの`hooks.php`ファイルで、任意のタスクにフックすることができます。その書式はとっても簡単です。以下に、上で述べた三つのタイプのタスクでの例を示します。

<!--original
You can hook into any task via the `hooks.php` file in Rocketeer's config file. The syntax is pretty straightforward. Below you can see an example of three types of Tasks mentioned above:
-->

```php
<?php
'after' => array(
  'setup' => array(

    // Commands
    'composer install',

    // Actual Tasks classes
    'Rocketeer\Tasks\Cleanup',
    'MyNamespace\MyTaskClass',

    // Closures
    function($task) {
      $tests = $task->runForCurrentRelease('phpunit --coverage-html=tests/coverage');

      if ($tests) {
        $task->command->info('Tests ran perfectly dude !');
      } else {
        $task->command->error('Aw man, tests failed and stuff');
      }
    },

  ),
?>
```

### ファサードを用いたタスクの定義

<!--original
### Defining Tasks using the facade
-->

Rocketeerは、また、あなたが使えるファサードも提供します。これは、クロージャーを使うと汚れてしまうなどの、設定ファイルに何かを入れたくない時に、特に有用です。

<!--original
Rocketeer also provides you with a facade you can use. It's especially useful if you don't want to put stuff in the config file, as it can get dirty when using closures.
-->

Rocketeerは、次の2つのことを許容します。すべてのフックを含んだ`.rocketeer/events.php`ファイルの作成。または、複数のクラスでクラスごとに1ファイルにするなど多数のファイルがあるならば、単純に`.rocketeer/events/`フォルダを作成して、各々のタスクを中にいれると、Rocketeerは自動的にロードします。

<!--original
Rocketeer allows two things: creating a `.rocketeer/events.php` file where all your hooks are contained or if you have more files like classes and want one file per task, you can simply create a `.rocketeer/events/` folder and every task you put in it will automatically be loaded by Rocketeer.
-->

```php
<?php
use Rocketeer\Facades\Rocketeer;

Rocketeer::before('deploy', function($task) {
  $task->command->info('Sup guys');
});

Rocketeer::after(['update', 'deploy'], array(
  'composer install',
  'bower install'
));

Rocketeer::after('deploy', 'MyClass');
?>
```

第一引数には、作用させたいタスク名（または、タスク名の配列）を与え、つづいてあなたのタスクを指定します。再度になりますが、あなたは、3つのタイプのタスクを使えます。文字列、クロージャ、クラスです。

<!--original
As a first argument you give the name of a Task (or an array of names) you'd like to act on and then your task. Again, you can use the three types of tasks: strings, closures or classes.
-->

## 独自タスクを定義する

<!--original
## Defining your own Tasks
-->

Rocketeerはまた、タスクランニングシステムとあわせて独自タスクの作成、管理、実行も提供します。これらは、`.rocketeer/tasks.php`ファイルに入れるか、もし多数ある場合には、それらが自動的にロードされる`./rocketeer/tasks/`フォルダに入れることができます。

<!--original
Rocketeer also provides you with a task-running system to create, manage and run your own tasks. You can put those either in a `.rocketeer/tasks.php` file or - if you have a lot of them - in a `./rocketeer/tasks/` folder which will load all the files in it.
-->

### ファサードを使う

<!--original
### Via the facade
-->

```php
<?php
Rocketeer::task('composer', 'composer install');

Rocketeer::task('composer', array(
  'composer self-update',
  'composer install',
));

Rocketeer::task('phpunit', function ($task) {
  return $task->runForCurrentRelease('phpunit');
}, 'Runs the PHPUnit tests');
?>
```

`task` メソッドは3つの引数をとります。タスク名、実行内容（1行コマンド, クロージャ, クラス）、そして、何をするかの説明です。


<!--original
The `task` method takes three arguments: name of the task, its execution (one-line command, closure, class) and a description of what it does.
-->

これらのタスクは、自動的にRocketeerに登録されます。これらは、`php rocketeer composer`などのようにCLIを使うか、ファサードを使って実行することができます。

<!--original
These tasks will be automatically registered with Rocketeer. You'll then be able to execute them either via the CLI by doing for example `php rocketeer composer` or via the facade:
-->

```php
Rocketeer::execute('composer');

Rocketeer::execute(['composer', 'phpunit']);
```

### クラスを使う

<!--original
### Via classes
-->

```php
<?php
namespace MyTasks;

class Migrate extends \Rocketeer\Abstracts\AbstractTask
{
  /**
   * Description of the Task
   *
   * @var string
   */
  protected $description = 'Migrates the database';

  /**
   * Executes the Task
   *
   * @return void
   */
  public function execute()
  {
    $this->explainer->line('Running migrations');

    return $this->runForCurrentRelease('php artisan migrate --seed');
  }
}
?>
```

クラスは、自動的にRocketeerに登録されないので、登録は手動で行う必要があります。次のいずれかで行います。設定ファイルを使って、`tasks.custom`配列の中で。

<!--original
Classes aren't automatically registered with Rocketeer so you'll need to do that manually. You can either do so via the config file in the `tasks.custom` array:
-->

```php
'custom' => array(
  'MyTasks\Migrate',
),
```

または、ファサードで。

<!--original
or via the facade:
-->

```php
Rocketeer::add('MyTasks\Migrate');
```

そしてほら！ジャーン！

<!--original
And there you go, tadah!
-->

![職人](http://i.imgur.com/jwdQ2Ly.png)

<!--original
![artisan](http://i.imgur.com/jwdQ2Ly.png)
-->

## タスクの実行

<!--original
## Executing tasks
-->

タスクが定義されたら、それはコマンドラインとファサードの2箇所に配置されます。

<!--original
Once a task is defined, it will be available in two places: the command line interface and the facade.
-->

もし、タスク名`composer`で登録したら、例えば、次のように実行できます。

<!--original
If you registered a task named `composer`, e.g. you'll be able to do this:
-->

```php
$ php rocketeer composer
```

もしくはPHPコード（`tasks.php`の中）で。

<!--original
Or in your PHP code (in `tasks.php`):
-->

```php
Rocketeer::execute('composer');
```

タスクの配列を渡すことで、複数のタスクを実行できます。Rocketeerは渡したものを常に処理することを覚えておくことはとても重要です。これは、タスクと考えられるものを何でも渡せることを意味します。
- コマンド文字列
- クロージャー
- タスク名
- タスクのクラス

<!--original
You can execute multiple tasks by passing an array of tasks. One thing that is crucial to remember is that Rocketeer will always process the queue you pass to it. That means you can pass anything that is considered as a task:
- A string command.
- A closure.
- The name of a task.
- The class of a task.
-->

```php
<?php
Rocketeer::execute(array(
  'my-task',

  'composer install',

  'Rocketeer\Tasks\Deploy',

  function ($task) {
    return $task->run('ls');
  },
));
?>
```

### ローカルでのタスクの実行

<!--original
### Executing tasks in local
-->

ローカルサーバー上で何かを実行するような快適さをもってコマンド郡をローカルで実行する必要のあるような場合。これは実タスククラスでのみ実現できます。ローカルプロパティを*true*に設定することのみでできます。

<!--original
In some cases you need to execute a series of commands in local and have the same comfort as you'd have executing things on the local server. This is only possible with actual Tasks classes. All you need to do is to set the `local` property to *true*:
-->

```php
class MyTask extends Rocketeer\Abstracts\AbstractTask
{
  protected $local = true;
}
```

これにより、Rocketeerは、すべての呼び出しを`LocalConnection`に委任します。これは、任意のConnectionクラスと同じように振る舞いますが、ローカルシステム上で実行されます。これは、デプロイの準備に有用で、`Primer`タスクが動作する方法です。

<!--original
From there, Rocketeer will delegate all calls to a `LocalConnection` class that acts the same as any Connection class but runs commands in the local system. This is useful for preparing the deploy and is how the `Primer` task works.
-->

-----

<!--original
-----
-->

## タスクを書く

<!--original
## Writing Tasks
-->

### コアメソッド

<!--original
### Core methods
-->

タスクのコアメソッドは、`run`メソッドです。これは、ほぼ全ての他のヘルパーの下に位置します。
これは単にリモートサーバーでコマンドを実行し出力を返します。

<!--original
The core method of any Task is the `run` method. This is the one that lays at the bottom of nearly every other helper.
It just runs commands on the remote server and returns the output.
-->

```php
<?php
$folders = $this->run('ls');
?>
```

また、実行するコマンドの配列を渡すことも可能です。これは重要なので気を止めておいてください。
全ての`run`からの呼び出しは、自己完結型です。このようになります。（`pwd`は、カレントフォルダを返します。）

<!--original
You can also pass to it an array of commands to execute. Now, note this because it's important: every call to `run` is self contained. Meaning this (`pwd` returns the current folder):
-->

```php
<?php
// Returns /
$this->run('cd first-folder');
$folder = $this->run('pwd');

// Returns /first-folder/
$folder = $this->run(array(
  'cd first-folder',
  'pwd',
));
?>
```

フォルダにある複数のタスクを自動的に実行するためには、2つのヘルパがあります。`runInFolder`と`runForCurrentRelease`です。最初のものは、フォルダにある1つ以上のタスクを実行します。もう一方はカレントリリースのフォルダある1つ以上のタスクを実行します。

<!--original
To automate running of tasks in folders two helpers exist: `runInFolder` and `runForCurrentRelease`. The first one will run one or more tasks in a folder, while the other will run one or more tasks in the current release's folder.
-->

```php
<?php
$this->run(array(
  'cd /home/www/website/releases/123456789',
  'ls',
));

// Is the same as

$this->runInFolder('releases/123456789', 'ls');

// Is the same as

$this->runForCurrentRelease('ls');
?>
```

### フォルダヘルパ

<!--original
### Folder helpers
-->

2,3のフォルダ・ファイルを操作する方法も存在します。 これらはとても基本的で抽象的な低レベルなbashコマンドたちですが、しかし、これらは有用です。

<!--original
A few folder/file manipulation methods are also present. They're very basic and just abstracting low-level bash commands but, hey, they're good to have:
-->

```php
<?php
$this->move('folder/file.php', 'new-folder/file.php');

$array = $this->listContents('folder');

$boolean = $this->fileExists('file.php');
$boolean = $this->fileExists('folder');

$this->createFolder('folder');
$this->removeFolder('folder');

$this->symlink('folder-a', 'folder-b');

$phpunit = $this->which('phpunit', 'vendor/bin/phpunit'); // Second argument is fallback
?>
```

### タスクに関連付けられるメソッド

<!--original
### Tasks-related methods
-->

いくつかのメソッドは、Rocketeerの別のタスクで使われ、独自のタスクをつくるのに使うことができます。それら全ては、カレントのリリースに関連付けされます。

<!--original
Some methods are used by other Rocketeer tasks and can be utilized to create your own ones. All of them are relative to the current release.
-->

```php
<?php
// Run tests
$boolean = $this->runTests();
$boolean = $this->runTests('--stop-on-failure');

// Run migrations
$this->runMigrations();
$this->runMigrations(true); // Seeds the database too

// Run Composer
$this->runComposer();

// Set folders as web-writtable
$this->setPermissions('public/images/users');
?>
```

### 外部メソッド

<!--original
### External methods
-->

タスクはまたRocketeerの他のクラスにアクセスすることができます。他のタスクを呼び出すことが可能です。

<!--original
Tasks also have access to the other classes of Rocketeer. You can call other tasks:
-->

```php
<?php
$this->executeTask('Rollback');
```

そして、他のクラスのメソッドを呼び出すことができます。すべてのタスクとストラテジは連なるコアクラスとそのメソッドにアクセスします。

<!--original
And call other classes's methods. All tasks and strategies have access to the following core classes and their methods:
-->

- **command** は、現在実行されているコマンドのオプションや引数を加えるためのインスタンスです。
- **scm** は、（git, SVNといった）カレントSCMのバイナリのインスタンスです。
- **rocketeer** は、カレントアプリケーションおよびその設定を取得を司ります。
- **connections** はカレント接続/ステージ/サーバーおよびそれぞれの資格情報を取得するハンドルです。
- **remote** は、`run`メソッドを使用しているクラスでのサーバーへのエントリポイントです。
- **explainer** は、出力の表示を司ります。たとえば、タスクの中で進行を表示するなどで使います。
- **paths** で、ローカルおよびリモートのパスを見つけます。
- **releasesManager** は、リリースおよびそのパスを扱います。
- **localStorage** クラスは、ローカルストレージのファイル更新を保持し、リモートサーバーの状態と資格情報を追跡します。
- **builder** は、タスクとストラテジをその場で構築するのに使用します。
- **queue** は、あなたのタスク中のキューの中でタスクを実行させるのに使用します。
- **tasks** はタスクおよびそのイベントの登録を扱います。これは、Rocketeerのファサードに隠蔽されたクラスです。

<!--original
- **command** is the instance of the command currently running, to fetch options and arguments;
- **scm** is the binary instance of the current SCM (git, SVN, etc.);
- **rocketeer** is responsible for getting information on the current app and its configuration;
- **connections** handles getting the current connection/stage/server and its respective credentials;
- **remote** is your entry point to the server, it's the class the `run` method uses;
- **explainer** is responsible for displaying the output, you'll use it to display e.g. progress in your tasks;
- **paths** finds local and remote paths;
- **releasesManager** handles releases and their paths;
- **localStorage** class keeps the local storage file up to date, it tracks remote server's state and credentials;
- **builder** allows you to build tasks and strategies on the fly;
- **queue** allows you to run tasks in a queue within your tasks;
- **tasks** handles registration of a tasks and their events, it's the class behind the Rocketeer facade.
-->

それぞれのサービスに存在するメソッドのリストは、[APIドキュメント](http://rocketeer.autopergamene.eu/api/namespaces/Rocketeer.html)より直接見ることができます。
これらのタスクは、自動的にRocketeerに登録されます。これらは、プロパティとしてアクセスされます。例えば、カレントリリースのフォルダーを取得するには以下のようにします。

<!--original
You can find a list of the methods available for each of these services directly in the [API documentation](http://rocketeer.autopergamene.eu/api/namespaces/Rocketeer.html).
They are accessed as a property, e.g. to get a folder in the current release you would do the following:
-->

```php
$folder $task->releasesManager->getCurrentReleasePath('some/folder');
```
