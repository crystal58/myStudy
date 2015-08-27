#数据库操作

#读取/写入连接

有时候你可能希望使用特定数据库连接进行 SELECT 操作，同时使用另外的连接进行 INSERT 、 UPDATE 、以及 DELETE 操作。 Laravel 让这些变得轻松简单，并确保你不论在使用原始查找、查找构建器、或者是 Eloquent ORM 使用的都是正确的连接。

来看看如何配置读取/写入连接，让我们来看以下的例子：

'mysql' => [
    'read' => [
        'host' => '192.168.1.1',
    ],
    'write' => [
        'host' => '196.168.1.2'
    ],
    'driver'    => 'mysql',
    'database'  => 'database',
    'username'  => 'root',
    'password'  => '',
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
],
注意我们加了两个键值到配置文件数组中： read 及 write。 两个键值都包含了单一键值的数组：host。read 及 write 的其余数据库配置会从mysql 数组中合并。 所以，如果我们想要覆写配置值，只要将选项放入 read 和 write 数组即可。 所以在上面的例子里， 192.168.1.1 将被用作「读取」连接，而 192.168.1.2 将被用作「写入」连接。数据库凭证、 前缀、字符编码配置、以及其他所有的配置会共用 mysql 数组里的配置。

#增删改查
执行 Select 查找

$results = DB::select('select * from users where id = ?', [1]);
select 方法会返回一个 array 结果。

You may also execute a query using named bindings:

$results = DB::select('select * from users where id = :id', ['id' => 1]);
执行 Insert 语法

DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
执行 Update 语法

DB::update('update users set votes = 100 where name = ?', ['John']);
执行 Delete 语法

DB::delete('delete from users');
注意： update 和 delete 语法会返回在操作中所影响的数据笔数。

#数据库事务处理

你可以使用 transaction 方法，去执行一组数据库事务处理的操作：

DB::transaction(function()
{
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
});
注意： 在 transaction 闭包若抛出任何异常会导致事务自动回滚。
有时候你可能需要自己开始一个事务：

DB::beginTransaction();
你可以通过 rollback 的方法回滚事务：

DB::rollback();
最后，你可以通过 commit 的方法提交事务：

DB::commit();

#查询构造器
 Laravel 查询构造器使用 PDO 参数绑定，以保护应用程序免于 SQL 注入，因此传入的参数不需额外转义特殊字符。
 从数据表中取得所有的数据列

 $users = DB::table('users')->get();

 foreach ($users as $user)
 {
     var_dump($user->name);
 }
 从数据表中分块查找数据列

 DB::table('users')->chunk(100, function($users)
 {
     foreach ($users as $user)
     {
         //
     }
 });
 通过在 闭包 中返回 false 来停止处理接下来的数据列：

 DB::table('users')->chunk(100, function($users)
 {
     //

     return false;
 });
 从数据表中取得单一数据列

 $user = DB::table('users')->where('name', 'John')->first();

 var_dump($user->name);
 从数据表中取得单一数据列的单一字段

 $name = DB::table('users')->where('name', 'John')->pluck('name');
 取得单一字段值的列表

 $roles = DB::table('roles')->lists('title');
 这个方法将会返回数据表 role 的 title 字段值的数组。你也可以通过下面的方法，为返回的数组指定自定义键值。

 $roles = DB::table('roles')->lists('title', 'name');
 指定查询子句 (Select Clause)

 $users = DB::table('users')->select('name', 'email')->get();

 $users = DB::table('users')->distinct()->get();

 $users = DB::table('users')->select('name as user_name')->get();
 增加查询子句到现有的查询中

 $query = DB::table('users')->select('name');

 $users = $query->addSelect('age')->get();
 使用 where 及运算符

 $users = DB::table('users')->where('votes', '>', 100)->get();
 「or」语法

 $users = DB::table('users')
                     ->where('votes', '>', 100)
                     ->orWhere('name', 'John')
                     ->get();
 使用 Where Between

 $users = DB::table('users')
                     ->whereBetween('votes', [1, 100])->get();
 使用 Where Not Between

 $users = DB::table('users')
                     ->whereNotBetween('votes', [1, 100])->get();
 使用 Where In 与数组

 $users = DB::table('users')
                     ->whereIn('id', [1, 2, 3])->get();

 $users = DB::table('users')
                     ->whereNotIn('id', [1, 2, 3])->get();
 使用 Where Null 找有未配置的值的数据

 $users = DB::table('users')
                     ->whereNull('updated_at')->get();
 Dynamic Where Clauses

 You may even use "dynamic" where statements to fluently build where statements using magic methods:

 $admin = DB::table('users')->whereId(1)->first();

 $john = DB::table('users')
                     ->whereIdAndEmail(2, 'john@doe.com')
                     ->first();

 $jane = DB::table('users')
                     ->whereNameOrAge('Jane', 22)
                     ->first();
 排序(Order By)、分群(Group By) 及 Having

 $users = DB::table('users')
                     ->orderBy('name', 'desc')
                     ->groupBy('count')
                     ->having('count', '>', 100)
                     ->get();
 偏移(Offset) 及 限制(Limit)

 $users = DB::table('users')->skip(10)->take(5)->get();

 Joins

 查询构造器也可以使用 join 语法，看看下面的例子：

 基本的 Join 语法

 DB::table('users')
             ->join('contacts', 'users.id', '=', 'contacts.user_id')
             ->join('orders', 'users.id', '=', 'orders.user_id')
             ->select('users.id', 'contacts.phone', 'orders.price')
             ->get();
 Left Join 语法

 DB::table('users')
         ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
         ->get();
 你也可以指定更高级的 join 子句：

 DB::table('users')
         ->join('contacts', function($join)
         {
             $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
         })
         ->get();
 如果你想在你的 join 中使用 where 型式的子句，你可以在 join 子句里使用 where 或 orWhere 方法。下面的方法将会比较 contacts 数据表中的 user_id 的数值，而不是比较两个字段。

 DB::table('users')
         ->join('contacts', function($join)
         {
             $join->on('users.id', '=', 'contacts.user_id')
                  ->where('contacts.user_id', '>', 5);
         })
         ->get();

 高级 Wheres

 群组化参数

 有些时候你需要更高级的 where 子句，如「where exists」或嵌套的群组化参数。Laravel 的查询构造器也可以处理这样的情况：

 DB::table('users')
             ->where('name', '=', 'John')
             ->orWhere(function($query)
             {
                 $query->where('votes', '>', 100)
                       ->where('title', '<>', 'Admin');
             })
             ->get();
 上面的查找语法会产生下方的 SQL：

 select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
 Exists 语法

 DB::table('users')
             ->whereExists(function($query)
             {
                 $query->select(DB::raw(1))
                       ->from('orders')
                       ->whereRaw('orders.user_id = users.id');
             })
             ->get();
 上面的查找语法会产生下方的 SQL：

 select * from users
 where exists (
     select 1 from orders where orders.user_id = users.id
 )

 聚合

 查找产生器也提供各式各样的聚合方法，如 count、max、min、avg 及 sum。

 使用聚合方法

 $users = DB::table('users')->count();

 $price = DB::table('orders')->max('price');

 $price = DB::table('orders')->min('price');

 $price = DB::table('orders')->avg('price');

 $total = DB::table('users')->sum('votes');

 原生表达式

 有些时候你需要使用原生表达式在查找语句里，这样的表达式会成为字串插入至查找，因此要小心勿建立任何 SQL 注入点。要建立原生表达式，你可以使用 DB::raw 方法：

 使用原生表达式

 $users = DB::table('users')
                      ->select(DB::raw('count(*) as user_count, status'))
                      ->where('status', '<>', 1)
                      ->groupBy('status')
                      ->get();

 添加

 添加数据进数据表

 DB::table('users')->insert(
     ['email' => 'john@example.com', 'votes' => 0]
 );
 添加自动递增 (Auto-Incrementing) ID 的数据至数据表

 如果数据表有自动递增的ID，可以使用 insertGetId 添加数据并返回该 ID：

 $id = DB::table('users')->insertGetId(
     ['email' => 'john@example.com', 'votes' => 0]
 );
 注意: 当使用 PostgreSQL 时，insertGetId 方法会预期自动增加的字段是以「id」为命名。
 添加多个数据进数据表

 DB::table('users')->insert([
     ['email' => 'taylor@example.com', 'votes' => 0],
     ['email' => 'dayle@example.com', 'votes' => 0]
 ]);

 更新

 更新数据表中的数据

 DB::table('users')
             ->where('id', 1)
             ->update(['votes' => 1]);
 自增或自减一个字段的值

 DB::table('users')->increment('votes');

 DB::table('users')->increment('votes', 5);

 DB::table('users')->decrement('votes');

 DB::table('users')->decrement('votes', 5);
 也能够同时指定其他要更新的字段：

 DB::table('users')->increment('votes', 1, ['name' => 'John']);

 删除

 删除数据表中的数据

 DB::table('users')->where('votes', '<', 100)->delete();
 删除数据表中的所有数据

 DB::table('users')->delete();
 清空数据表

 DB::table('users')->truncate();

 Unions

 查询构造器也提供一个快速的方法去「合并 (union)」两个查找的结果：

 $first = DB::table('users')->whereNull('first_name');

 $users = DB::table('users')->whereNull('last_name')->union($first)->get();
 unionAll 方法也可以使用，它与 union 方法的使用方式一样。


 悲观锁定 (Pessimistic Locking)

 查询构造器提供了少数函数协助你在 SELECT 语句中做到「悲观锁定」。

 想要在 SELECT 语句中加上「Shard lock」，只要在查找语句中使用 sharedLock 函数：

 DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
 要在 select 语法中使用「锁住更新(lock for update)」时，你可以使用 lockForUpdate 方法：

 DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();