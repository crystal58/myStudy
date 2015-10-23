#Laravel Autoload 自动加载

laravel 自动加载是通过verdor/autoload.php加载如下四个文件完成的\r\n
1.PSR0加载方式—对应的文件就是 vendor/composer/autoload_namespaces.php \r\n
2.PSR4加载方式—对应的文件就是 vendor/composer/autoload_psr4.php \r\n
3.其他加载类的方式—对应的文件就是 vendor/composer/autoload_classmap.php \r\n
4.加载公用方法—对应的文件就是 vendor/composer/autoload_files.php \r\n

这四个文件是通过composer dump-autoload命令自动生成的，
需要在composer.json文件， require相对应的模块

怎么样自定义自动加载方式

如果某些文件，需要自动自定义加载方式，可以在Composer.json文件中定义

"autoload" : {
        //以第一种方式自动加载，表示app目录下的所有类的命名空间都是以Apppsr0开始且遵循psr0规范（注意：您的laravel中没有此项，作为示意例子）
        "psr-0" : {
            "AppPsr0": "apppsr0/"
        }，
        //以第二种方式自动加载，表示app目录下的所有类的命名空间都是以App开始且遵循psr4规范
        "psr-4" : {
            "App\\": "app/"
        }，
        //以第三种加载方式自动加载，它会将所有.php和.inc文件中的类提出出来然后以类名作为key，类的路径作为值
        "classmap" : ["database"],
        //以第四种加载方式自动加载,composer会把这些文件都include进来（注意：您的laravel中没有此项，作为示意例子）
        "files" : ["common/util.php"]

    }