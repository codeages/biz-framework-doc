# 入门指南

## 安装

```
cd your-application-dir
composer require codeages/biz-framework
```

## 初始化

BizFramework 定位为简单、易用的业务层的框架，采用经典的Service/Dao模式来编写应用的业务逻辑，可以跟Symfony2、Laravel、Silex、Lumen、Phalcon等框架搭配使用。

在程序初始化入口处中加入：

```php
$biz = new Biz($config);
```

`Biz`对象，是应用业务层的依赖注入容器(Dependency Injection Container)对象，一个应用中只能存在一个单例，在应用的其他地方可以通过`$biz = Biz::instance();`来获得Biz容器对象。由于BizFramework不涉及应用配置的处理，所以，你需从应用级框架中获得业务相关的配置，组装成`$config`数组，在初始化`Biz`对象时，作为参数传入。

!!! note "容器"
    `Biz`类直接继承自`Pimple\Container`，所以拥有Pimple的所有容器特性，请阅读[Pimple的文档](http://pimple.sensiolabs.org/)，来了解容器的使用。

### 目录结构




