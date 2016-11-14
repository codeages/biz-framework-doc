# 数据库变更脚本(Migration)

BizFramework使用[phpmig](https://github.com/davedevelopment/phpmig)作为数据库变更脚本的引擎，并约定项目根目录下的`migrations`目录为默认的migration脚本所在目录。

## 基本使用

** 查看所有命令 **

```
bin/phpmig
```

** 生成一个Migration脚本类 **

```
bin/phpmig generate YourClassName
```

参数`YourClassName`为本次Migration脚本类的类名，请根据实际情况取名，表明意图。运行此命令后，会在`migrations`目录下，生成`VERSIONID_YourClassName.php`文件：

```php
<?php

use Phpmig\Migration\Migration;

class YourClassName extends Migration
{
    /**
     * Do the migration
     */
    public function up()
    {
        $container = $this->getContainer();
        $sql = "...";
        $container['db']->exec($sql);

    }

    /**
     * Undo the migration
     */
    public function down()
    {
        $container = $this->getContainer();
        $sql = "...";
        $container['db']->exec($sql);
    }
}

```

`up()`函数中，用于编写本次数据库变更的SQL。`down()`函数中，用于编写撤销本次数据库变更的SQL。

** 运行所有未执行过的Migration脚本 **

```
bin/phpmig migrate
```

** 重新执行某个具体版本的Migration脚本 **

```
bin/phpming redo VERSION_NO
```

** 回退最后执行过的一个Migration脚本 **

```
bin/phpmig rollback
```