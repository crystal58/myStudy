## 模型 Models
接下来我们将接触Laravel最为强大的部分，Eloquent ORM，真正提高生产力的地方，借用库克的一句话：鹅妹子英！

运行一下命令：

php artisan make:model Article

php artisan make:model Page
> Laravel 4 时代，我们使用 Generator 插件来新建 Model。现在，Laravel 5 已经把 Generator 集成进了 Artisan。

现在，Artisan 帮我们在 `learnlaravel5/app/` 下创建了两个文件 `Article.php` 和 `Page.php`，这是两个 Model 类，他们都继承了 Laravel Eloquent 提供的 Model 类 `Illuminate\Database\Eloquent\Model`，且都在 `\App` 命名空间下。这里需要强调一下，用命令行的方式创建文件，和自己手动创建文件没有任何区别，你也可以尝试自己创建这两个 Model 类。

Model 即为 MVC 中的 M，翻译为 模型，负责跟数据库交互。在 Eloquent 中，数据库中每一张表对应着一个 Model 类（当然也可以对应多个）。

如果你从其他框架转过来，可能对这里一笔带过的 Model 部分很不适应，没办法，是因为 Eloquent 实在太强大了啦，真的没什么好做的，继承一下 Eloquent 类就能实现很多很多功能了。

如果你想深入地了解 Eloquent，可以阅读系列文章：深入理解 Laravel Eloquent（一）——基本概念及用法
接下来进行 Article 和 Page 类对应的 articles 表和 pages表的数据库迁移，进入 `learnlaravel5/database/migrations` 文件夹。

在 ***_create_articles_table.php 中修改：

Schema::create('articles', function(Blueprint $table)
{
	$table->increments('id');
	$table->string('title');
	$table->string('slug')->nullable();
	$table->text('body')->nullable();
	$table->string('image')->nullable();
	$table->integer('user_id');
	$table->timestamps();
});
在 ***_create_pages_table.php 中修改：

Schema::create('pages', function(Blueprint $table)
{
	$table->increments('id');
	$table->string('title');
	$table->string('slug')->nullable();
	$table->text('body')->nullable();
	$table->integer('user_id');
	$table->timestamps();
});
然后执行命令：

php artisan migrate
成功以后， tables 表和 pages 表已经出现在了数据库里，去看看吧~

##数据库填充 Seeder
在 `learnlaravel5/database/seeds/` 下新建 `PageTableSeeder.php` 文件，内容如下：

<?php

use Illuminate\Database\Seeder;
use App\Page;

class PageTableSeeder extends Seeder {

  public function run()
  {
    DB::table('pages')->delete();

    for ($i=0; $i < 10; $i++) {
      Page::create([
        'title'   => 'Title '.$i,
        'slug'    => 'first-page',
        'body'    => 'Body '.$i,
        'user_id' => 1,
      ]);
    }
  }

}
然后修改同一级目录下的 `DatabaseSeeder.php`中：

// $this->call('UserTableSeeder');
这一句为

$this->call('PageTableSeeder');
然后运行命令进行数据填充：

composer dump-autoload

php artisan db:seed
去看看 pages 表，是不是多了十行数据？