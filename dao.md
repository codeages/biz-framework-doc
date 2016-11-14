# DAO

DAO(Data access object)即数据访问对象，封装所有对数据源进行操作的API。以数据库驱动的应用程序为例，通常数据库中的一张表，对应的会有一个Dao类去操作这个数据库表。BizFramework框架中定义了基础Dao借口`GeneralDaoInterface`，您可以直接接触