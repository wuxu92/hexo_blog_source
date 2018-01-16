---
title: laravel 笔记
date: 2016-05-21 22:25:26
tags:
---

1. 关于路由，见一个请求转发到控制器： 

```php
Route::get('/', function () {
    return view('welcome');
});

// Controller_Class_Namw@function
Route::get('user/info', 'UserController@info');	

// or as:
// Route::get('user/info', [
//     'as'=>'userinfo', 'uses' => 'UserController@info'
// ]);

Route::get('phpinfo', function() {
    return phpinfo();
});
```

2. Controller需要使用命令生成，不能自己新建文件生成：

```bash
php artisan make:controller nameController --plain
```

3. 获得某个Route的url： `$url = route('userinfo')`,参数为路由别名。或者使用action方法，获得控制器下action的连接，`$url = action('UserContrller@info')`; `Route::currentRouteAction()` 可以获得当前的控制器和行为（action）
4. 在 Route 中可以指定中间件，使用 `middleware` 键指定控制器的中间件，也可以在控制器的构造器中指定中间件： `$this->middleware('auth');`;
5. Laravel提供隐式控制器，通过单个路由处理控制器的各种行为，有点类似于Yii的默认路由。 `Route::controller('user', UserController);` 不过要在控制器中加入方法，方法的名称应由它们所响应的 HTTP 动词作为开头，紧跟着首字母大写的 URI 所组成， 如 `getIndex` 会处理 get 请求的 `index`（默认） 动作。
6. 

