# プラグイン

<!--original
# Plugins
-->

プラグインシステムを使うことで、機能をRocketeerに加えたり、シンプルに一般的なタスクを再利用可能なモジュールにまとめることができます。プラグインはその中心に、`Rocketeer\Abstracts\AbstractPlugin` 抽象化クラスを継承します。

<!--original
You can add functionalities to Rocketeer or simply bundle common tasks into reusable modules by using the plugins system. A plugin at its core is a class implementing the `Rocketeer\Abstracts\AbstractPlugin` abstract.
-->

## プラグインの追加

<!--original
## Adding a plugin
-->

プラグインを追加するためには、`rocketeer plugin:install <package>`コマンドを呼び出す必要があります。例えば `rocketeer plugin:install anahkiasen/rocketeer-slack` のように、パッケージはパッケージのPackagistかGithubハンドルで始まります。


<!--original
To add a plugin, you need to call the `rocketeer plugin:install <package>` command, per example `rocketeer plugin:install anahkiasen/rocketeer-slack`, the package being the Packagist/Github handle of the package.
-->

追加を終えたら、プラグインのクラスを、`.rocketeer/config.php`の`plugins`配列に加えます。

<!--original
Once this is done, add the plugin's class to the `plugins` array in `.rocketeer/config.php`:
-->

```php
'plugins' => array(
	'Rocketeer\Plugins\Slack\RocketeerSlack',
),
```

続いて、多くの場合ではプラグインの設定をする必要があります。そのために、その設定を`rocketeer plugin:publish <package>`命令を使ってユーザーランドに書き出したいと思います。ここでは、例として `rocketeer plugin:publish anahkiasen/rocketeer-slack` を呼びます。

<!--original
Then, in most cases you'll need to configure said plugin. For this you'll want to publish its configuration in user land via the `rocketeer plugin:publish <package>` command. Here we'll call `rocketeer plugin:publish anahkiasen/rocketeer-slack` per example.
-->

これは、すべてのプラグインの設定を入れ込んだ`.rocketeer/plugins/rocketeers/rocketeer-slack`フォルダを作成します。

<!--original
This will create the `.rocketeer/plugins/rocketeers/rocketeer-slack` folder, with all the plugin's configuration files inside.
-->


## プラグインの作成

<!--original
## Creating a plugin
-->

およそほとんどのプラグインは、そのクラスに`register(Container $app)`と`onQueue(TasksQueue $queue)`という二つのメソッドをもちます。

<!--original
There's two methods a plugin will most likely have on its class are `register(Container $app)` and `onQueue(TasksQueue $queue)`.
-->

- 一つ目はRocketeerのコンテナに、最終的なインスタンスをバインドするのに使用されます。これは、それがオーバーライドされた時に最終的にコンテナをリターンする必要がある場合に適したメソッドです。
- 二つ目はアクションまたはタスクをロケッティアに追加するのに使います。**TaskQueue**クラスはロケッティアの外観の背後のクラスであり、あなたがよく知っているメソッドが有効です。`$queue->before('deploy', ...)`や `$queue->add('MyCustomTask')`などです。

<!--original
- The first one will be used to bind eventual instances into Rocketeer's container, that is a facultative method that if overridden needs to return the Container at the end.
- The second one is used to add actions or tasks to Rocketeer : the **TasksQueue** class is the one behind the Rocketeer facade so most of the methods you're familiar with are available on it : `$queue->before('deploy', ...)`, `$queue->add('MyCustomTask')` etc.
-->

これは現在のCampfireプラグインの非常に簡単にした一つのバージョン例です。依存関係として`rcrowe/Campfire`を使っています。

<!--original
Here is an example dumbed-down version of the current Campfire plugin, using `rcrowe/Campfire` as a dependency :
-->

```php
use rcrowe\Campfire;
use Illuminate\Container\Container;
use Rocketeer\Services\Tasks\TasksQueue;
use Rocketeer\Abstracts\AbstractPlugin;

class RocketeerCampfire extends AbstractPlugin
{
  /**
   * Bind additional classes to the Container
   *
   * @param Container $app
   *
   * @return void
   */
  public function register(Container $app)
  {
    $app->bind('campfire', function ($app) {
      return new Campfire(['domain' => '...', 'key' => '...', 'room' => '...']);
    });

    return $app;
  }

  /**
   * Register Tasks with Rocketeer
   *
   * @param TasksQueue $queue
   *
   * @return void
   */
  public function onQueue(TasksQueue $queue)
  {
    $queue->after('deploy', function ($task) {
      $application = $task->rocketeer->getApplicationName();

      $task->campfire->send($application. ' was deployed!');
    });
  }
}
```

見ての通り、どこかにとっておきプロジェクトをまたいで使うものとして、プラグインは _非常に_ シンプルなものです。

<!--original
As you can see a plugin can be something _really_ simple you can save up somewhere and reuse from project to project.
-->

## プラグインの設定

<!--original
## Plugin configurations
-->

プラグインは`config`フォルダをプラグインの`src`フォルダの下に作成することで、その設定をもつことができます。クラスの中でパスをセットする必要があります。例ではコンストラクタの中です。

<!--original
Plugins can have their own configuration, by creating a `config` folder in your plugin's `src` folder. You'll need to set the path to it on your class, in the constructor per example :
-->

```php
public function __construct()
{
  $this->configurationFolder = __DIR__.'/../../config';
}
```

その作成したフォルダの中に、オプションをPHP配列として入れるための、`config.php`ファイルを作成します。あなたのプラグインのための設定は`my-plugin::` 名前空間下のあなたのタスクの中のConfigクラスにあります。例えば  `$task->config->get('rocketeer-hipchat::myoption')` と実行することで、設定を取得できます。

<!--original
In that folder you can then create a `config.php` file to put your options as a PHP array. The configuration for your plugin will then be available via the Config class in your tasks, under the `my-plugin::` namespace, per example if your class is `RocketeerHipchat`, you'll get the configuration by doing `$task->config->get('rocketeer-hipchat::myoption')`.
-->
