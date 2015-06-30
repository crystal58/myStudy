#php 的运行模式
PHP运行模式有4钟：
1）cgi 通用网关接口（Common Gateway Interface)）
2） fast-cgi 常驻 (long-live) 型的 CGI
3） cli  命令行运行   （Command Line Interface）
4）web模块模式 （apache等web服务器运行的模块模式）

1.  CGI（Common Gateway Interface）
         CGI即通用网关接口(Common Gateway Interface)，它是一段程序, 通俗的讲CGI就象是一座桥，把网页和WEB服务器中的执行程序连接起来，它把HTML接收的指令传递给服务器的执行程序，再把服务器执行程序的结果返还给HTML页。CGI 的跨平台性能极佳，几乎可以在任何操作系统上实现。 CGI已经是比较老的模式了，这几年都很少用了。
         每有一个用户请求，都会先要创建cgi的子进程，然后处理请求，处理完后结束这个子进程，这就是fork-and-execute模式。 当用户请求数量非常多时，会大量挤占系统的资源如内存，CPU时间等，造成效能低下。所以用cgi方式的服务器有多少连接请求就会有多少cgi子进程，子进程反复加载是cgi性能低下的主要原因。
         如果不想把 PHP 嵌入到服务器端软件（如 Apache）作为一个模块安装的话，可以选择以 CGI 的模式安装。或者把 PHP 用于不同的 CGI 封装以便为代码创建安全的 chroot 和 setuid 环境。这样每个客户机请求一个php文件，Web服务器就调用php.exe（win下是php.exe,linux是php）去解释这个文件，然后再把解释的结果以网页的形式返回给客户机。 这种安装方式通常会把 PHP 的可执行文件安装到 web 服务器的 cgi-bin 目录。CERT 建议书 CA-96.11 建议不要把任何的解释器放到 cgi-bin 目录。
         这种方式的好处是把web server和具体的程序处理独立开来，结构清晰，可控性强，同时缺点就是如果在高访问需求的情况下，cgi的进程fork就会成为很大的服务器负担，想 象一下数百个并发请求导致服务器fork出数百个进程就明白了。这也是为什么cgi一直背负性能低下，高资源消耗的恶名的原因。

        CGI模式安装：
     CGI已经是比较老的模式了，这几年都很少用了,所以我们只是为了测试。

     安装CGI模式需要注释掉

     LoadModule php5_module modules/libphp5.so 这行。如果不注释这行会一直走到handler模式。也就是模块模式。

     然后在httpd.conf增加action：

     Action application/x-httpd-php /cgi-bin/

     如果在/cgi-bin/目录找不到php-cgi.可自行从php的bin里面cp一个。

     然后重启apache,再打开测试页面发现Server API变成：CGI/FastCGI。说明成功切换为cgi模式。

          问题：
1)  如果cgi程序放在/usr/local/httpd/cgi-bin/里无法执行，遇到403或500错误的话

打开apache错误日志 有如下提示： Permission denied: exec of

可以检查cgi程序的属性，按Linux contexts文件 里定义的，/usr/local/httpd/cgi-bin/里必须是httpd_sys_script_exec_t 属性。  通过ls -Z查看，如果不是则通过如下命令更改：     chcon -t httpd_sys_script_exec_t /var/www/cgi-bin/*.cgi     如果是虚拟主机里的cgi，则参考问题2使之能正常使用普通的功能后，再通过chcon设置cgi文件的context为

httpd_sys_script_exec_t即可。chcon -R -t httpd_sys_script_exec_t cgi-bin/

2) apache错误提示：.... malformed header from script. Bad header=
根据提示说明有header有问题，查看文件输出的第一句话是什么，应该类似于如下

Content-type: text/plain; charset=iso-8859-1\n\n

或者Content-type:text/html\n\n

注意：声明好Content-type后要输出两个空行。

3）apache错误提示： Exec format error

脚本解释器设置错误。脚本第一行应该以'#!解释器路径'的形式, 填写脚本解释器的路径，如果是PERL程序，常见的路径为:     #!/usr/bin/perl 或 #!/usr/local/bin/perl           如果是PHP程序，不需要填写解释器路径，系统会自动找到PHP。

2.  Fastcgi模式

 fast-cgi 是cgi的升级版本，FastCGI 像是一个常驻 (long-live) 型的 CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去 fork 一次 (这是 CGI 最为人诟病的 fork-and-execute 模式)。
 FastCGI的工作原理是：
(1)、Web Server启动时载入FastCGI进程管理器【PHP的FastCGI进程管理器是PHP-FPM(php-FastCGI Process Manager)】（IIS ISAPI或Apache Module);

(2)、FastCGI进程管理器自身初始化，启动多个CGI解释器进程 (在任务管理器中可见多个php-cgi.exe)并等待来自Web Server的连接。

(3)、当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。

(4)、FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器（运行在 WebServer中）的下一个连接。在正常的CGI模式中，php-cgi.exe在此便退出了。

在CGI模式中，你可以想象 CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部dll扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作。


Fastcgi的优点：

1）从稳定性上看, fastcgi是以独立的进程池运行来cgi,单独一个进程死掉,系统可以很轻易的丢弃,然后重新分 配新的进程来运行逻辑.

2）从安全性上看,Fastcgi支持分布式运算. fastcgi和宿主的server完全独立, fastcgi怎么down也不会把server搞垮.

3）从性能上看, fastcgi把动态逻辑的处理从server中分离出来, 大负荷的IO处理还是留给宿主server, 这样宿主server可以一心一意作IO,对于一个普通的动态网页来说, 逻辑处理可能只有一小部分, 大量的图片等静态

FastCGI缺点：说完了好处，也来说说缺点。从我的实际使用来看，用FastCGI模式更适合生产环境的服务器。但对于开发用机器来说就不太合适。因为当使用 Zend Studio调试程序时，由于 FastCGI会认为 PHP进程超时，从而在页面返回 500错误。这一点让人非常恼火，所以我在开发机器上还是换回了 ISAPI模式。

安装fastcgi模式：
安装apache路径是/usr/local/httpd/
安装php路径是/usr/local/php/
1）安装mod_fastcgi
wget http://www.fastcgi.com/dist/mod_fastcgi-2.4.6.tar.gz
tar zxvf mod_fastcgi-2.4.6.tar.gz
cd mod_fastcgi-2.4.6
cp Makefile.AP2 Makefile
vi Makefile，编辑top_dir = /usr/local/httpd
make
make install
安装完后，
/usr/local/httpd/modules/多出一个文件：mod_fcgid.so
2）重新编译php
./configure --prefix=/usr/local/php --enable-fastcgi --enable-force-cgi-redirect --disable-cli
make
make install
这样编译后，在PHP的bin目录下的php-cgi就是fastcgi模式的php解释器了
安装成功后,执行
php -v 输出
PHP 5.3.2 (cgi-fcgi).
这里输出带了cgi-fcgi
注意：
1. 编译参数不能加 –with-apxs=/usr/local/httpd/bin/apxs 否则安装出来的php执行文件是cli模式的
2  如果编译时不加--disable-cli则输出 PHP 5.3.2(cli)

3)配置apache
需要配置apache来以fastcgi模式运行php程序
vi httpd.conf
我们使用虚拟机的方式实现：

#加载fastcgi模块
LoadModule fastcgi_module     modules/mod_fastcgi.so
#//以静态方式执行fastcgi 启动了10进程
FastCgiServer /usr/local/php/bin/php-cgi  -processes 10 -idle-timeout 150 -pass-header HTTP_AUTHORIZATION
<VirtualHost *:80>
     #
     DocumentRoot   /usr/local/httpd/fcgi-bin
     ServerName www.fastcgitest.com

     ScriptAlias /fcgi-bin/   /usr/local/php/bin/   #定义目录映射 /fcgi-bin/ 代替 /usr/local/php/bin/
     Options +ExecCGI
     AddHandler fastcgi-script .php .fcgi         #.php结尾的请求都要用php-fastcgi来处理
     AddType application/x-httpd-php .php     #增加MIME类型
     Action application/x-httpd-php /fcgi-bin/php-cgi  #设置php-fastcgi的处理器： /usr/local/php/bin/php-cgi

 <Directory /usr/local/httpd/fcgi-bin/>
      Options Indexes ExecCGI
      Order allow,deny
      allow from all
 </Directory>
</VirtualHost>

或者
<IfModule mod_fastcgi>ScriptAlias /fcgi-bin/ "/usr/local/php/bin" #定义目录映射FastCgiServer /usr/local/php/bin/php-cgi   -processes 10 #配置fastcgi server，<Directory "/usr/local/httpd/fcgi-bin/">SetHandler fastcgi-scriptOptions FollowSymLinksOrder allow,denyAllow from all</Directory>AddType application/x-httpd-php .php&nbsp; #增加MIME类型AddHandler php-fastcgi .php   #.php结尾的请求都要用php-fastcgi来处理Action php-fastcgi /fcgi-bin/php-cgi #设置php-fastcgi的处理器
</IfModule>



4）.restart 下apache,查看phpinfo，如果服务器信息是：
Apache/2.2.11 (Unix) mod_fastcgi/2.4.6之类的就说明安装成功了。
如果出现403的错误，查看下/usr/local/httpd/fcgi-bin/是否有足够的权限。
或者
<Directory />
    Options FollowSymLinks
    AllowOverride None
    Order deny,allow
    Deny from all
</Directory>
改为：
<Directory />
    Options FollowSymLinks
    AllowOverride None
Order allow,deny
Allow from all
</Directory>
就可以了。
ps -ef|grep  php-cgi可以看见10个fastcgi进程在跑。

3.   CLI模式

cli是php的命令行运行模式，大家经常会使用它，但是可能并没有注意到（例如：我们在linux下经常使用 "php -m"查找PHP安装了那些扩展就是PHP命令行运行模式；有兴趣的同学可以输入php -h去深入研究该运行模式）

1.让 PHP 运行指定文件。
php script.php
php -f script.php
以上两种方法（使用或不使用 -f 参数）都能够运行脚本的script.php。您可以选择任何文件来运行，您指定的 PHP 脚本并非必须要以 .php 为扩展名，它们可以有任意的文件名和扩展名。
2.在命令行直接运行 PHP 代码。
php -r "print_r(get_defined_constants());"
在使用这种方法时，请您注意外壳变量的替代及引号的使用。
注: 请仔细阅读以上范例，在运行代码时没有开始和结束的标记符！加上 -r 参数后，这些标记符是不需要的，加上它们会导致语法错误。
3.通过标准输入（stdin）提供需要运行的 PHP 代码。
以上用法给我们提供了非常强大的功能，使得我们可以如下范例所示，动态地生成 PHP 代码并通过命令行运行这些代码：
$ some_application | some_filter | php | sort -u >final_output.txt

4. 模块模式

模块模式是以mod_php5模块的形式集成，此时mod_php5模块的作用是接收Apache传递过来的PHP文件请求，并处理这些请求，然后将处理后的结果返回给Apache。如果我们在Apache启动前在其配置文件中配置好了PHP模块（mod_php5）， PHP模块通过注册apache2的ap_hook_post_config挂钩，在Apache启动的时候启动此模块以接受PHP文件的请求。
         除了这种启动时的加载方式，Apache的模块可以在运行的时候动态装载，这意味着对服务器可以进行功能扩展而不需要重新对源代码进行编译，甚至根本不需要停止服务器。我们所需要做的仅仅是给服务器发送信号HUP或者AP_SIG_GRACEFUL通知服务器重新载入模块。但是在动态加载之前，我们需要将模块编译成为动态链接库。此时的动态加载就是加载动态链接库。 Apache中对动态链接库的处理是通过模块mod_so来完成的，因此mod_so模块不能被动态加载，它只能被静态编译进Apache的核心。这意味着它是随着Apache一起启动的。

        Apache是如何加载模块的呢？我们以前面提到的mod_php5模块为例。首先我们需要在Apache的配置文件httpd.conf中添加一行：

该运行模式是我们以前在windows环境下使用apache服务器经常使用的，而在模块化(DLL)中，PHP是与Web服务器一起启动并运行的。（是apache在CGI的基础上进行的一种扩展，加快PHP的运行效率）
LoadModule php5_module modules/mod_php5.so
这里我们使用了LoadModule命令，该命令的第一个参数是模块的名称，名称可以在模块实现的源码中找到。第二个选项是该模块所处的路径。如果需要在服务器运行时加载模块，可以通过发送信号HUP或者AP_SIG_GRACEFUL给服务器，一旦接受到该信号，Apache将重新装载模块，而不需要重新启动服务器。


6.  php在Nginx中运行模式（Nginx+ PHP-FPM）
使用FastCGI方式现在常见的有两种stack：ligthttpd+spawn-fcgi;另外一种是nginx+PHP-FPM(也可以用spawn-fcgi)。

A、如上面所说该两种结构都采用FastCGI对PHP支持，因此HTTPServer完全解放出来，可以更好地进行响应和并发处理。因此lighttpd和nginx都有small, but powerful和efficient的美誉。

B、该两者还可以分出一个好坏来，spawn-fcgi由于是lighttpd的一部分，因此安装了lighttpd一般就会使用spawn-fcgi对php支持，但是目前有用户说ligttpd的spwan-fcgi在高并发访问的时候，会出现上面说的内存泄漏甚至自动重启fastcgi。即：PHP脚本处理器当机，这个时候如果用户访问的话，可能就会出现白页(即PHP不能被解析或者出错)。

另一个：首先nginx不像lighttpd本身含带了fastcgi(spawn-fcgi)，因此它完全是轻量级的，必须借助第三方的FastCGI处理器才可以对PHP进行解析，因此其实这样看来nginx是非常灵活的，它可以和任何第三方提供解析的处理器实现连接从而实现对PHP的解析(在nginx.conf中很容易设置)。nginx可以使用spwan-fcgi(需要一同安装lighttpd，但是需要为nginx避开端口，一些较早的blog有这方面安装的教程)，但是由于spawn-fcgi具有上面所述的用户逐渐发现的缺陷，现在慢慢减少使用nginx+spawn-fcgi组合了。

C、由于spawn-fcgi的缺陷，现在出现了新的第三方(目前还是，听说正在努力不久将来加入到PHP core中)的PHP的FastCGI处理器，叫做PHP-FPM(具体可以google)。它和spawn-fcgi比较起来有如下优点：

由于它是作为PHP的patch补丁来开发的，安装的时候需要和php源码一起编译，也就是说编译到php core中了，因此在性能方面要优秀一些；

同时它在处理高并发方面也优于spawn-fcgi，至少不会自动重启fastcgi处理器。具体采用的算法和设计可以google了解。

因此，如上所说由于nginx的轻量和灵活性，因此目前性能优越，越来越多人逐渐使用这个组合：nginx+PHP/PHP-FPM


6.  总结
目前在

HTTPServer这块基本可以看到有三种stack比较流行：
（1）Apache+mod_php5

（2）lighttp+spawn-fcgi

（3）nginx+PHP-FPM

三者后两者性能可能稍优，但是Apache由于有丰富的模块和功能，目前来说仍旧是老大。有人测试nginx+PHP-FPM在高并发情况下可能会达到Apache+mod_php5的5~10倍，现在nginx+PHP-FPM使用的人越来越多。

文章来源：http://blog.csdn.net/hguisu/article/details/7386882