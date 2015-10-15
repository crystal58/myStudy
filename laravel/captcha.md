Laravel的验证码库gregwar/captcha

在Laravel中有很多图片验证码的库可以使用，本篇介绍其中之一：gregwar/captcha，这个库比较简单，在Laravel中比较常用。下面我们就来介绍下使用细节：

首先, composer.json中如下加入配置：

"require": {
        ...
        "gregwar/captcha": "dev-master"
    },
然后，已成习惯的命令：

composer update
接下来就可以正常使用了，根据具体的开发需求，可以有很多种方式去使用。

可以将验证码图片保存文件：
<?php

$builder->save('out.jpg');
可以直接输出图片到网页：
<?php

header('Content-type: image/jpeg');
$builder->output();
可以生成内联图片：
<img src="<?php echo $builder->inline(); ?>" />
以下演示了其中一种使用方式，直接输出图片到网页。

我定义了一个Controller：

<?php namespace App\Http\Controllers;

use App\Http\Requests;
use App\Http\Controllers\Controller;

use Illuminate\Http\Request;

//引用对应的命名空间
use Gregwar\Captcha\CaptchaBuilder;
use Session;

class KitController extends Controller {

    /**
     * Display a listing of the resource.
     *
     * @return Response
     */
    public function captcha($tmp)
    {
        //生成验证码图片的Builder对象，配置相应属性
        $builder = new CaptchaBuilder;
        //可以设置图片宽高及字体
        $builder->build($width = 100, $height = 40, $font = null);
        //获取验证码的内容
        $phrase = $builder->getPhrase();

        //把内容存入session
        Session::flash('milkcaptcha', $phrase);
        //生成图片
        header("Cache-Control: no-cache, must-revalidate");
        header('Content-Type: image/jpeg');
        $builder->output();
    }

}
下面我们可以设置相应的router访问这个验证码图片, 修改router.php：

Route::get('kit/captcha/{tmp}', 'KitController@captcha');
现在可以通过具体的url，可以访问看到这张图片了

表单内部写的比较简单，看看即可：

<input type="text" name="captcha" class="form-control" style="width: 300px;">
          <a onclick="javascript:re_captcha();" ><img src="{{ URL('kit/captcha/1') }}"  alt="验证码" title="刷新图片" width="100" height="40" id="c2c98f0de5a04167a9e427d883690ff6" border="0"></a>

<script>
  function re_captcha() {
    $url = "{{ URL('kit/captcha') }}";
        $url = $url + "/" + Math.random();
        document.getElementById('c2c98f0de5a04167a9e427d883690ff6').src=$url;
  }
</script>
最后就是在form提交页面验证相应验证码，库中也为我们提供了相应方法：

if($builder->testPhrase($userInput)) {
    //用户输入验证码正确
}
else {
    //用户输入验证码错误
}
至此，验证码就完成了。

原创地址：http://www.jianshu.com/p/8e4ac7852b5a


补充：

为了安全，可以为session加密

编辑app/config/app.php，增加配置项：

'key' => '你的key，作为加密用',

加密：
Crypt::encrypt($phrase);

解密：
Crypt::decrypt($phrase)
