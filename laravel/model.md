## 模型 Models
接下来我们将接触Laravel最为强大的部分，Eloquent ORM

运行一下命令：

php artisan make:model Article

php artisan make:model Page

Artisan 帮我们在 `app/` 下创建了两个文件 `Article.php` 和 `Page.php`，这是两个 Model 类，他们都继承了 Laravel Eloquent 提供的 Model 类 `Illuminate\Database\Eloquent\Model`，且都在 `\App` 命名空间下。这里需要强调一下，用命令行的方式创建文件，和自己手动创建文件没有任何区别，你也可以尝试自己创建这两个 Model 类。

Model 即为 MVC 中的 M，翻译为 模型，负责跟数据库交互。在 Eloquent 中，数据库中每一张表对应着一个 Model 类（当然也可以对应多个）。

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

注意在默认情况下，在数据库表里需要有 updated_at 和 created_at 两个字段。如果您不想设定或自动更新这两个字段，则将类里的 $timestamps 属性设为 false即可。


##数据库填充 Seeder
在 `database/seeds/` 下新建 `PageTableSeeder.php` 文件，内容如下：

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

## 数据库操作
以users表为例

取出所有记录

$users = User::all();

根据主键取出一条数据

$user = User::find(1);

Eloquent 模型结合查询语法

$users = User::where('votes', '>', 100)->take(10)->get();

foreach ($users as $user)
{
    var_dump($user->name);
}
Eloquent 聚合查询

当然，您也可以使用查询构造器的聚合查询方法。

$count = User::where('votes', '>', 100)->count();
如果没办法使用流畅接口产生出查询语句，也可以使用 whereRaw 方法：

$users = User::whereRaw('age > ? and votes = 100', [25])->get();