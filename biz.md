## Service / Dao 的自动寻址



'' => 'Biz',
'ExamplePlugin' => 'ExamplePlugin\Biz',



'User.UserService'  => 'Biz\User\Service\Impl\UserServiceImpl',

'ExamplePlugin.ExampleService' => 'ExamplePlugin\Biz\Service\Impl\ExampleServiceImpl'


'ExampleProject\' => 'Example',

Example.ExampleService => 'ExampleProject\Service\Impl\ExampleServiceImpl',

'' => '',

Example.ExampleSerivce => Service\Impl\ExampleServiceImpl

| 别名  | 实际类名 |
|       | 