# laravel 命令

## php artisan migrate
   执行database/migrations/ 目录下的php文件

## php artisan make:model Article
   创建model

##composer dump-autoload
  php artisan db:seed
  执行database/seeds/下的某个文件，如PageTableSeeder.php， 填充数据，需要去掉DatabaseSeeder.php文件 $this->call('PageTableSeeder') 这行代码的注释，call()方法的参数跟前面文件的一致

##php artisan make:controller Admin/AdminHomeController
  创建controller

##composer dump-autoload，重新加载一下composer的auto-load
