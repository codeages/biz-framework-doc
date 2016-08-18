# Validation

（草稿)

### API说明

#### 代码示例：

  ```    
  //创建Validator实例，方可进行后续操作
  $validator = new Validator();
  ```

  ```
  //覆盖默认配置信息并返回当前生效的配置信息数组
  $validator->config($config);
  ```

  ```
  //添加自定义校验规则，ruleName字符串需符合约定的命名规则，function的第一项必须是要校验的value，$params 为可选参数
  $validator->addRule($ruleName, function($val, array $params = null){
      //code here
  });
  //获取自定义rule
  $validator->getExtRule($ruleName);
  ```

  ```
  //执行validate动作
  $validator->validate($data, array(
      'title' => 'required|max(255)',
      'author.name' => 'required|lengthRange(3,20)',
      'author.description' => 'required',
  ), array("filter" => true));
  ```

  ```
  //执行validate后可调用以下方法获取校验结果，validate之前调用无意义,每次执行validate会清空之前的校验结果
  //判断本次校验是否出错(true/false)
  $validator->fails()
  //获取本次校验生成的错误信息（array），如无错误，则为空数组
  $validator->errors()
  //当配置了$config["filter"]=true时, 可获取过滤后的数据，具体规则参见［配置信息］
  $validator->filteredData(): array
  ```


#### 待校验的数据

  - 格式为数组，key的命名不允许特殊符号，仅限字母、数字、下划线，其中数字不能作为开头；
  - value的数据类型限定为：数值、字符串、数组。  
  - key用于表达对 待处理数据（data）的key的映射  
    data中数据可能是一层或两层，比如  

    ```  
    [
      id: 11,
      name: 'James Bond',
      nickname: '007',
      profile: [
        post: 'Agent',
        partner: ['Jenny']
      ],
      stories:[
        [
          title: 'Dr. No',
          time: '1962',
          actor: '肖恩·康纳利'
        ],[
          ...  
          ...  
        ],[
          title: 'Spectre',
          time: '2015',
          actor: '丹尼尔·克雷格'
        ]
      ]
      ...
    ]

    映射：id , profle, profile.post, stories, stories.[].title(代表数组中每个元素的title)
    ```

#### 校验规则

  - key和data中的key对应，使用点(.)表示层级，比如author.name代表data.author.name；
  - rule的命名仅限字母、数字、下划线，不允许其它符号，数字不能在首位，区分大小写；
  - 多个rule使用(|)分隔，首尾可以有空格；
  - rule写法说明：
    - 无参数（此时“()”可省略）  
      比如 required, email, ip, int, float, bool
    - 有参数
      参数限定为数值、字符串（使用单引号／双引号引起来）、key（仅限$data第一层的key，不使用引号）。  
      比如：range(1,1000), length(30), max(100), min(100), sameWith(password), requiredWith(username), in('male', 'female')
  - 仅对规则中提到的key进行校验, 未指定的变量不校验;
  - 指定了数组中的变量但未指定数组时则默认数组可为空，仅在数组有数据时校验这些变量。   
  - 如果rules为空，则忽略对此key的校验；
  - 如无特殊配置，任一rule无效（存在语法错误或为定义）则终止校验，抛出InvalidArgumentException



#### 配置信息

  - 配置信息可以在调用validate方法时提供，或者validate前调用```$validator->config($config);```指定。
  - blocked: 可选值：true/false, 默认为true。true表示一发现错误就结束校验，否则校验全部key，注意如果对key指定了多个rule，则会返回第一个出错的rule的错误信息。
  - filter: 可选值：true/false, 默认为false。true表示是否根据校验规则中定义的key过滤掉data中的数据
  - throwException: 可选值：true/false, 默认为false。true表示出错时则终止校验并抛出ValidationException（同时视为blocked=true）
  - ignoreSyntaxError: 可选值：true/false，默认为false。true表示忽略无效rule。

#### 错误信息  

  - 校验错误以数组保存，key和[...]中的key对应，无数据则表示无错误；
  - 信息的大致描述风格：不是有效的IP，不能为空，不在1-1000范围内。
  - 执行validate(...)后通过$validator->errors()获取错误信息。
  - 执行validate(...)后通过$validator->filteredData()获取过滤后的数据（如果config["filter"]=true）。

#### 内置规则

  - 校验数据的类型: bool, int, float, string, date, jsonstr
  - 校验数据的范围: range(min, max), lenrange(min, max), min(min), max(max), minlen(max), maxlen(max) ...
  - 校验数据的业务特征: ip, email, cnid, urlhttp, cnmobile, ....
  - 校验数据的关联关系: required, requiredWith(xx), sameWith(xx), diffWith(xx), before(xx), after(xx), between(xx,yy), in(xx,yy,zz)

#### 规则拓展

  - 提供addRule($ruleName, $ruleFunc)来添加自定义rule，自定义rule的优先级高于内置rule，如果同名，则会覆盖；
  - ruleFunc的第一个参数必须是要处理的value，其它参数将使用array注入。
  - 例子：

    ```
    //规则定义
    $validator.addRule("myRule", function($val, $params){
      echo "val : $val, params: $params[0], $params[1]..."
    });
    //调用方式：（$val由容器根据key自动查找注入）
    myRule(params1, params2);
    ```

### FAQ

#### 是否可以直接使用reg校验？

    不支持, 可以自定义rule实现

#### 是否支持错误信息的国际化

    待定

#### 其它

    - 数据的过滤目前只对数据的第一级生效；
    - 数据层次限定为最多两层，即父子级。
    - key作为rule的参数时，仅提取key映射到的第一个value。

#### 展望

    - 根据实际业务需要拓展当前实现