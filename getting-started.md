# 入门指引

<h2 id="introduce">简介</h2>

BizFramework 定位为简单、易用的**业务层**的框架，采用经典的Service/Dao模式来编写项目的业务逻辑，可以跟Symfony、Laravel、Silex、Lumen、Phalcon等框架搭配使用。

那么已经有了Symfony、Laravel这样流行的框架，BizFramework的意义又在哪里呢？

Symfony、Laravel等框架完整的Web开发框架，每个框架都有各自的业务层解决方案，并且跟Web框架本身绑定，无法做到各个项目之间共享业务层代码。而往往不同的项目会基于实际情况采用不同的Web开发框架，虽然各个项目之间的UI相差会比较大，但一些通用模块的业务逻辑其实是一致的，比如用户注册、私信等。

BizFramework的目标，给出一套组织业务代码的约定以及最佳实践，以让一些通用的模块的业务代码，能跨项目、跨Web开发框架的重用。使用BizFramework能给团队带来的好处有：

  * 提高生产效率，减少重复开发。
  * 能保证一些通用模块的质量，一些通用模块往往经过各个项目不断的锤炼，会有较高的质量。
  * 方便团队各Team之间人员流动，因为大家都采用了一致的业务层框架，很容易就能上手新项目。

同时BizFramework还有个目标，建设开源社区，在不断完善BizFramework的同时，建设一批通用的业务代码库。让业务代码可以跨公司、跨国家的重用。您只需`composer install your-needed-biz-module`。

<h2 id="install">安装</h2>

```
cd your-application-dir
composer require codeages/biz-framework
```

<h2 id="init">初始化</h2>

在您的程序初始化入口处中加入：

```php
$biz = new Biz($config);
```

`Biz`对象，是应用业务层的依赖注入容器(Dependency Injection Container)对象，一个应用中只能存在一个单例，在应用的其他地方可以通过`$biz = Biz::instance();`来获得Biz容器对象。由于BizFramework不涉及应用配置的处理，所以，你需从应用级框架中获得业务相关的配置，组装成`$config`数组，在初始化`Biz`对象时，作为参数传入。

!!! note "容器"
    `Biz`类直接继承自`Pimple\Container`，所以拥有Pimple的所有容器特性，请阅读[Pimple的文档](http://pimple.sensiolabs.org/)，来了解容器的使用。

<h2 id="directory">目录结构</h2>

以下为含`User`, `Article`两个业务模块的推荐的目录结构示例：

```
src/
  Biz/
    User/
      Dao/
        Impl/
          UserDaoImpl.php
        UserDao.php
      Service/
        Impl/
          UserServiceImpl.php
        UserService.php
    Article
      Dao/
        Impl/
          ArticleDaoImpl.php
          CategoryDaoImpl.php
        ArticleDao.php
        CategoryDao.php
      Service/
        Impl/
          ArticleServiceImpl.php
        ArticleService.php
```

  * 请在应用的`composer.json`的autoload部分加上`psr-4`的`autoload`规则，比如：
    ```
    "autoload": {
        "psr-4": { 
            "": "src/"
        }
    }
    ```
  * 约定应用级业务层的顶级命名空间为`Biz`，命名空间的第二级为模块名；
  * 约定*Service接口*的接口名以Service作为后缀，命名空间为`Biz\模块名\Service`, 上述例子中`UserService`的完整类名为`Biz\User\Service\UserService`；
  * 约定*Service实现类*的类名以ServiceImpl作为后缀，命名空间为`Biz\模块名\Service\Impl`, 上述例子中`UserServiceImpl`的完整类名为`Biz\User\Service\Impl\UserServiceImpl`；
  * *Dao接口、类名*的命名约定，同*Sevice接口、类名*的命名约定。

<h2 id="migration">创建数据库</h2>

在编写业务代码之前，我们首先需要创建数据库，BizFramework使用[phpmig](https://github.com/davedevelopment/phpmig)作为数据库变更脚本的引擎，并约定项目根目录下的`migrations`目录为默认的migration脚本所在目录。

以创建user表为例：

1. 生成migration脚本文件：

```
bin/phpmig generate user
```

我们会在migrations目录下看到，上述名利创建了`{date}_user.php`文件。

2. 我们打开由第1步生成的文件，修改内容为：

```
<?php
use Phpmig\Migration\Migration;

class User extends Migration
{
    public function up()
    {
        $container = $this->getContainer();

        $sql = "
            CREATE TABLE `user` (
              `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
              `username` varchar(32) NOT NULL,
              `email` varchar(128) NOT NULL,
              `password` varchar(128) NOT NULL,
              `salt` varchar(64) NOT NULL,
              `created_time` int(10) unsigned NOT NULL,
              `updated_time` int(10) unsigned NOT NULL,
              PRIMARY KEY (`id`),
              UNIQUE KEY `username` (`username`),
              UNIQUE KEY `email` (`email`),
              KEY `mobile` (`mobile`)
            ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
        ";
        $container['db']->exec($sql);
    }

    public function down()
    {
        $container = $this->getContainer();
        $container['db']->exec("DROP TABLE `user`");
    }
}

```

<h2 id="dao">编写Dao</h2>

以编写 User Dao 为例，我们首先需要创建`UserDao接口`：

```php
<?php
namespace Biz\User\Dao;

use Codeages\Biz\Framework\Dao\GeneralDaoInterface;

interface UserDao extends GeneralDaoInterface
{

}
```

这里我们直接继承了`Codeages\Biz\Framework\Dao\GeneralDaoInterface`，在`GeneralDaoInterface`中，我们声明了常用的Dao接口：

```php
<?php
namespace Codeages\Biz\Framework\Dao;

interface GeneralDaoInterface extends DaoInterface
{
    public function create($fields);

    public function update($id, array $fields);

    public function delete($id);

    public function get($id);

    public function search($conditions, $orderBy, $start, $limit);

    public function count($conditions);

    public function wave(array $ids, array $diffs);
}
```

同样我们的 User Dao 实现类，也可继承自`Codeages\Biz\Framework\Dao\GeneralDaoImpl` ：

```php
<?php
namespace Biz\User\Dao\Impl;

use Biz\User\Dao\UserDao;
use Codeages\Biz\Framework\Dao\GeneralDaoImpl;

class UserDaoImpl extends GeneralDaoImpl implements UserDao
{
    protected $table = 'user';

    public function declares()
    {
        return array(
            'timestamps' => array('created_time', 'update_time'),
            'serializes' => array(),
            'orderbys' => array(),
            'conditions' => array(
                'username = :username',
            ),
        );
    }
}
```

这样我们就拥有了`GeneralDaoInterface`接口所定义的所有方法功能。关于方法`declares()`的详细说明，请参见文档[编写Dao](dao.md)。

<h2 id="service">编写Service</h2>

以编写 User Service 为例，我们首先需创建`UserService`接口：

```php
<?php
namespace Biz\User\Service;

interface UserService
{
    public function getUser($id);

    // ...
}

```

然后创建 User Service 的实现类：

```php
<?php
namespace Biz\User\Service\Impl;

use Codeages\Biz\Framework\Service\BaseService;
use Biz\User\Service\UserService;

class UserServiceImpl extends BaseService implements UserService
{
    public function getUser($id)
    {
        return $this->getUserDao()->get($id);
    }

    // ...

    protected function getUserDao()
    {
        return $this->biz->dao('User:UserDao');
    }
}

```

这里我们 `UserServiceImpl` 继承了 `Codeages\Biz\Framework\Service\BaseService` ，使得 `UserServiceImpl` 可以自动被注入`Biz`容器对象。

在 `getUserDao()` 方法中，我们通过 `$this->biz->dao('User:UserDao')`，获得了 User Dao 对象实例`Biz\User\Dao\Impl\UserDaoImpl`，具体参见 [获取 Dao / Service 的实例对象](container.md#get-dao-service-instance)。

<h2 id="service-using">使用Service</h2>

以实现*显示用户的个人主页*为例，我们的`Controller`代码大致为：

```php
<?php

class UserController
{
    public function showAction($id)
    {
        $user = $this->getUserService()->getUser($id);
        // ...
        return $this->render('user.html.twig', array('user' => $user));
    }
    // ...

    protected function getUserService()
    {
        return $this->biz->service('User:UserService');
    }
}
```

其中 `getUserService()` 同上个例子中的 `getUserDao()` 原理类似，通过调用 `getUserService()` 我们获得了`Biz\User\Service\Impl\UserServiceImpl` 对象实例。

<h2 id="end">结语</h2>

BizFramework的入门指南，就到此为止。具体请看[使用指南](README.md#guides)。如您使用开源Web开发框架，以下文档叙述了BizFramework与各Web框架的整合使用：

* [BizFramework与Symfony的整合](biz-and-symfony.md)
* [BizFrameowrk与Sliex的整合](biz-and-sliex.md)
