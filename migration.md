# 数据库变更脚本(Migration)

BF整合了[phpmig](https://github.com/davedevelopment/phpmig)做为Migration的引擎。BF默认使用项目的`/migrations`目录作为当前项目的migration脚本的存放地。

## 使用

### 查看所有命令

```
bin/phpmig
```

### 生成一个Migration脚本类

```
bin/phpmig generate ClassName ./migrations
```

第一个参数`ClassName`为本次Migration脚本类的类名，请根据实际情况取名，表明意图。
第二个参数`./migrations`为Migration脚本类的存放目录，请使用约定值`./migrations`。

### 运行所有未执行过的Migration脚本

```
bin/phpmig migrate
```

### 重新执行某个具体版本的Migration脚本

```
bin/phpming redo VERSION_NO
```

### 回退最后执行过的一个Migration脚本

```
bin/phpmig rollback
```