#路由
在'/app/Http/routes.php' 的末尾添加以下代码：

Route::group(['prefix' => 'admin', 'namespace' => 'Admin'], function()
{
  Route::get('/', 'AdminHomeController@index');
  Route::resource('article', 'ArticleController');
});
表示创建了一个路由组。

1. `'prefix' => 'admin'` 表示这个路由组的 url 前缀是 /admin，也就是说中间那一行代码 `Route::get('/'` 对应的链接不是 http://fuck.io:88/ 而是 http://fuck.io:88/admin ，如果这段代码是 `Route::get('fuck'` 的话，那么 URL 就应该是 http://fuck.io:88/admin/fuck 。

2. `'namespace' => 'Admin'` 表示下面的 `AdminHomeController@index` 不是在 `\App\Http\Controllers\AdminHomeController@index` 而是在 `\App\Http\Controllers\Admin\AdminHomeController@index`，加上了一个命名空间的前缀。

Laravel 5 把命名空间全部隔开，控制器在 `\App\Http\Controllers`，模型在 `\App`，让我们在刚上手的时候就体验命名空间分离的感觉，总体上其实是会降低学习成本的。

<hr>
## Middleware

在群组共享属性数组的 middleware 参数定义中间件列表，这些中间件就会应用到群组内的所有路由上。中间件将会按在列表内指定的顺序执行：

Route::group(['middleware' => ['foo', 'bar']], function()
{
    Route::get('/', function()
    {
        // Has Foo And Bar Middleware
    });

    Route::get('user/profile', function()
    {
        // Has Foo And Bar Middleware
    });

});

中间件的列子说明：

Route::group(['prefix' => 'admin', 'namespace' => 'Admin', 'middleware' => 'auth'], function()
{
  Route::get('/', 'AdminHomeController@index');
  Route::resource('article', 'ArticleController');
});
之前的路由，不登陆也可以查看admin的页面，上面代码中只有一处变化：给 `Route::group()` 的第一个参数（一个数组）增加了一项 `'middleware' => 'auth'`。
现在访问admin(如果没有登录) ，应该会跳转到登陆页面。如果没有跳转，可能是没有退出登陆。


<hr>