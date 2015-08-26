#控制器
我们可以使用 Artisan 非常方便地构建控制器：

php artisan make:controller Admin/AdminHomeController
得到 `app/Http/Controllers/Admin/AdminHomeController.php` 文件。

在 `class AdminHomeController extends Controller {` 上面增加一行：

use App\Page;
修改 index() 的代码如下：

public function index()
{
  return view('AdminHome')->withPages(Page::all());
}
控制器中文文档：http://laravel-china.org/docs/5.0/controllers

控制器中涉及到了许多的命名空间知识，可以参考 PHP 命名空间 解惑。