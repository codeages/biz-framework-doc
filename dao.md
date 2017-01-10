# DAO

DAO(Data access object)即数据访问对象，封装所有对数据源进行操作的API。以数据库驱动的应用程序为例，通常数据库中的一张表，对应的会有一个Dao类去操作这个数据库表。

## 通用的Dao基类

BizFramework框架中定义了基础Dao接口`GeneralDaoInterface`，以及对应的实现：`GeneralDaoImpl`。`GeneralDaoInterface`接口如下，定义了常用的增删改查的数据操作方法。

```php
<?php
namespace Codeages\Biz\Framework\Dao;

interface GeneralDaoInterface
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

我们在声明自己的Dao接口时，可以继承`GeneralDaoInterface`，例如：

```php
use Codeages\Biz\Framework\Dao\GeneralDaoInterface;

interface ExampleDao extends GeneralDaoInterface
{
    public function getByName($name);
}
```

同样，在编写Dao实现类时，我们继承`GeneralDaoImpl`类，例如：

```php
<?php
namespace Biz\Dao\Impl;

use Codeages\Biz\Framework\Dao\GeneralDaoImpl;
use Biz\Dao\ExampleDao;

class ExampleDaoImpl extends GeneralDaoImpl implements UserDao
{
    protected $table = 'example';

    public function getByName($name)
    {
        return $this->getByField('name', $name);
    }

    public function declares()
    {
        return array(
            'timestamps' => array('created_time', 'update_time'),
            'serializes' => array(),
            'orderbys' => array(),
            'conditions' => array(
                'type = :type',
            ),
        );
    }
}
```

其中属性`$table`表明此Dao类所对应的数据库表名。每个Dao类，须申明`declares()`方法，该方法返回一个array类型的配置：

| Key             |类型     |说明                     |
|-----------------|--------|------------------------|
| timestamps      | array  | 定义数据创建、新增时的时间戳自动更新的字段。第1个字段表示创建时自动更新的字段、第2个字段表示数据更新时自动更新的字段。
| serializes      | array  | 定义需序列化的字段，序列化方式有`json` |
| orderbys        | array  | 定义可排序的字段 |
| conditions      | array  | 定义可被search方法调用的查询条件 |

## 命名约定

通常Dao类中，只会存在一种数据对象，所以方法名命名的时候我们约定省略名词。即由原先的`动词+名词`的方式，变为只有`动词`的方式。

约定有：

  * 通常每个数据表都应该存在递增`id`主键字段；
  * 查询单行数据，方法名以`get`开头；
  * 查询多行数据，方法名以`find`开头；
  * 根据指定条件查询用`By`字段；如根据主键`id`查询，则省略`ById`；多个条件用`And`连接；方法参数的顺序跟方法名的查询条件一致；
  * 分页查询多行数据，以`$start`, `$limit`作为分页参数，加在方法参数的末尾；`$start`表示要查询的起始行数，`$limit`表示要查询的行数。

### 命名示例

| 方法名              | 说明                      |
|--------------------|--------------------------|
| get($id)           | 根据主键`id`查询单行数据     |
| getByEmail($email) | 根据`Email`字段查询单行数据        |
| getByNameAndType($name, $type) | 根据`Name`和`Type`字段查询单行数据 |
| findByIds(array $ids) | 根据一批`id`查询多行数据 |
| findByType($type, $start, $limit) | 根据`Type`分页查询多行数据 |

## condition

## 最佳实践准则

### 准则一：Dao类中应避免Join查询

