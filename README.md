# Laravel Introduction
## Index
1. Request Life Cycle
2. Basic Concept
    - Dependency Injection
    - Container
    - IoC Container
## Request Life Cycle
`Laravel` 是一个单入口的框架, 对后端的请求都会被 `web server` (如: `Nginx`, `Apache` 等) 转到 `./public/index.php` 文件, 它包含了整个 `application` 的生命周期.
以下引用自 `./public/index.php` 文件, 部分 `comment` 解释了其作用:
```
<?php

use Illuminate\Contracts\Http\Kernel;
use Illuminate\Http\Request;

define('LARAVEL_START', microtime(true));

if (file_exists($maintenance = __DIR__.'/../storage/framework/maintenance.php')) {
    require $maintenance;
}

/*
|--------------------------------------------------------------------------
| 这里导入了所有的 `composer` 包
|--------------------------------------------------------------------------
|
*/
require __DIR__.'/../vendor/autoload.php';

/*
|--------------------------------------------------------------------------
| 下面一句创建了整个 `application` 的服务容器, 并加载了所有的服务
|--------------------------------------------------------------------------
*/
$app = require_once __DIR__.'/../bootstrap/app.php';

/*
|--------------------------------------------------------------------------
| `App\Http\Kernel` 类在 `./app/Http/Kernel.php` 文件, 实现了 `Illuminate\Contracts\Http\Kernel` 定义的所有 `attributes`, 包含了所有的 `middleware` 信息
| 所以获取了整个路由的信息, 包括在 `./routes` 文件夹下建立的路由, 以及预定义和在 `./app/Http/Middleware`下建立的 `middleware`
| 该类继承自 `Illuminate\Foundation\Http`, 实现了 `handle()` 函数, 
|--------------------------------------------------------------------------
*/
$kernel = $app->make(Kernel::class);

/*
|--------------------------------------------------------------------------
| Laravel 的请求类直接沿用来的 Symfony 的设定, 并支持 RESTFul 路由
| 这里 `Illuminate\Http\Request` 下的静态方法 `capture()` 根据传入的数据重新打包成了 `Symfony\Component\HttpFoundation\Request` 类, 
| `handle` 方法将打包好的请求类传入 `Controller` 控制器下对应的方法, 控制器类继承 `Illuminate\Routing\Controller` 返回 `\Illuminate\Http\Response` 实例
| 最后通过 `send()` 方法发送客户端
|--------------------------------------------------------------------------
*/
$response = $kernel->handle(
    $request = Request::capture()
)->send();

/*
|--------------------------------------------------------------------------
| 请求循环结束, 停止服务, 销毁垃圾, 并执行对应的事件
|--------------------------------------------------------------------------
*/
$kernel->terminate($request, $response);
```

