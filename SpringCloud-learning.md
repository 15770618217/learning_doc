### Eureka

> 注册发现

##### 服务向Eureka注册实例使用IP代替名称配置：	

```
eureka.instance.preferIpAddress=true
eureka.instance.instance-id=${spring.cloud.client.ipAddress}:${server.port}
```

### Zuul

> 服务网关、限流、鉴权、路由转发

### Hystrix

> 断路器（当某服务出现问题，调用这个服务就会出现线程阻塞，Servlet容器的线程资源会被消耗完毕，导致服务瘫痪，禁止故障传播）

### 静态代理

> 必须实现接口，代理类也实现相同接口并传入目标类作为参数，实现相同方法，然后在该方法里面通过调用目标类方法前后做操作

```
void method(){
	//前置操作
	target.method();
	//后置操作
}
```

### 动态代理

> `jdk`基于反射机制，目标类需要实现接口

```
   public Object getProxyInstance() {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                (proxy1, method1, args1) -> {
                    start();
                    //执行目标对象方法
                    Object returnValue = method1.invoke(target, args1);
                    commit();
                    return returnValue;
                }
        );
    }
```

> `cglib`底层基于字节码处理框架`ASM`转换字节码动态生成目标类子类，**代理的类不能为final,否则报错；目标对象的方法如果为final/static,那么就不会被拦截,即不会执行目标对象额外的业务方法.**

### 分布式任务

> 分布式部署，任务只能执行一次

- 一台机器，将一些不太重要的定时任务，可以使用一个专门的服务台承载，然后使用单机跑，就算挂了只要我们再可接受的时间之内将其恢复，我们的业务也不会受到影响。

- 多台机器，加分布式锁，只要我们执行任务的时候首先获取一把分布式锁，如果获取失败那么久证明有其他服务已经再运行，如果获取成功那么证明没有服务在运行定时任务，那么就可以执行。

- 多台机器，利用ZooKeeper对Leader机器执行定时任务，有很多业务已经使用了ZK，那么执行定时任务的时候判断自己是否是Leader，如果不是则不执行，如果是则执行业务逻辑，这样也能达到我们的目的。

  **推荐使用`XXL-JOB`**、`elastic-job`

### 分布式事务

> 一般来说任何一个分布式事务框架都离不开三个关键字：重做记录，重试机制，幂等。而在我们的业务中同样也离不开这三个关键字。

- 重做记录：通过数据记录保存。
- 重试机制：定时任务或者消息队列自带的重试。
- 幂等：通过状态机加数据库行锁。（`唯一索引`，分库分表就不好使了）

### zookeeper

> 选举、监听、注册发现、事务锁

案例：

- 拆分定时微服务，部署两份，通过`zk选举`保证可用性（springcloud中判断当前服务ip是否所有服务列表中最小的）

- 多任务集群部署，通过监听父节点，当集群中有机器宕机，该服务的节点会被zk移除，其他服务收到消息，把宕机服务中的任务分发到其他机器中

```
zk选举（原理：同时在一个固定节点下创建临时顺序节点，序列最小的成为leader）
使用Curator（对zk衍生类库进行了封装）
1：LeaderLatch：初始化时候抢占Leader，之后节点故障才会进行节点变更
2：LeaderSelector：每次都进行一次抢占，执行完任务释放leader位置。
```




### AOP

**expression**常用方法

方法参数匹配：args()，@args()

方法描述匹配：execution()

目标类匹配：target()，@target()，within()，@within()

标有此注解的方法匹配：@annotation()

> @within和@target针对类的注解,@annotation是针对方法的注解

> 最靠近(..)的为方法名,靠近.*(..))的为类名或者接口名,如上例的JoinPointObjP2.*(..))

> 当一个实现了接口的类被AOP的时候,用getBean方法必须cast为接口类型,不能为该类的类型.

### GIT

- ```
  git remote add origin（） https://github.com/15770618217/cloud-learning.git
  ```

- ```
  git push -u origin master	# 推送并关联
  ```

- pull的时候允许之前的历史merge

  ```
  git pull origin master --allow-unrelated-histories
  ```

- 本地分支与远程分支建立联系，pull或者push不需要指定远程分支

  ```
  git branch -m old_branch new_branch  # Rename branch locally
  git push origin :old_branch # Delete the old branch
  git push --set-upstream origin new_branch # Push the new branch,set local branch to track thw new remote
  ```

  ```
  git branch --set-upstream-to=origin/originBranch localBranch
  ```

  ```
  git branch --set-upstream origin branch_name
  ```

- **git reset** 回滚提交（暂存区或者本地库）

  - add 后 git reset 暂存区恢复到工作区

  - commit 后 git reset HEAD^ 修改又恢复到工作区

    ```
    git reset --mixed HEAD^ （默认，修改回滚到工作区）
    git reset --soft HEAD^	（修改被放到暂存区）
    git reset --hard HEAD^	（修改被直接丢弃）
    ```

- **git revert** 回滚提交（已push到远程）

- **git rebase**

  ```
  git checkout fixbug
  git rebase master
  ```

  根据当前分支（fixbug）与 master 最近的共同祖先后续的提交历史，生成一个补丁文件，以基底（master）最后一个提交历史为新出发点，放在后面，从而改写fixbug的提交历史，使他成为master的直接下游

  ```
  git rebase -i head~3	#将前3次commit合并成一个commit  -i（interactive）
  ```




- **git fsck**

  ```
  //查看“丢失的”对象们
  git fsck --lost-found
  ```





- **git stash**

  ```
  git stash save -u "stash备注"
  
  -u 将新增的文件（untrack files） 也stash起来
  -a 将新增（untrack）的文件与忽略（.ignore）的文件都stash
  
  git stash list
  git stash pop stash@{0}		# 弹出并删除
  git stash apply stash@{0}	# 应用不删除
  ```

  



​	


- **git rebase onto  target_branch  log_A  log_B(左开右闭]  ** target_branch不是最新的活动分支，此时活动的分支是log_B

- **cherry pick**

  ```
  git cherry-pick commit_id_1 commit_id_2 commit_id_3	 # 将commit1、2、3提交合并到当前分支
  ```

  

- **改动内容追加到某个commit上**

  ```
  1、git rebase log_id	--interactive	# 将HEAD移到需要更改的commit上
  2、找到需要更改的commit, 将行首的pick改成edit, 按esc, 输入:wq退出
  3、git add 
  4、git commit --amend
  5、git rebase --continue	 # 移动HEAD到最新的commit处（有冲突解决后继续 --continue）
  ```

  

### shell

```
nohup java -jar /home/jar.d/hugo.jar > /home/logs/sum.log 2>&1 & 
```

> 》/home/hogs/sum.log 指日志输出到sum.log
>
> 2 > &1 指2(**表示stderr标准错误**)的输出也等同于1（**表示stdout标准输出，系统默认值是1**）的输出到指定日志文件

```
ps -ef | grep hugo.jar | grep -v grep | awk '{print $2}'
```

> [https://gitee.com/52itstyle/spring-boot-seckill/blob/master/Java%20Basis/JVM%20%E7%BA%BF%E4%B8%8A%E6%95%85%E9%9A%9C%E6%8E%92%E6%9F%A5%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C.md](https://gitee.com/52itstyle/spring-boot-seckill/blob/master/Java Basis/JVM 线上故障排查基本操作.md)

> linux源码安装，configure、make、make install不指定prefix，则可执行文件默认放在/usr /local/bin，库文件默认放在/usr/local/lib，配置文件默认放在/usr/local/etc。其它的资源文件放在/usr /local/share。过程中用到configure --prefix  --with；其中--prefix指的是安装路径，--with指的是安装本文件所依赖的库文件

### mongdb

**内存读写，异步刷入磁盘，支持全表索引，数据库想要速度必须减少磁盘IO，mongodb采用B-数据结构存储索引Key，mysql采用B+数据结构存储索引Key**

- B+树只能在叶子节点存储数据，各节点存储索引。查询效率更加稳定，数据全部存储在叶子节点，查询时间复杂度固定为 O(log n)。B+树叶节点两两相连可大大增加区间访问性（利用磁盘预读（局部性原理）），可使用在范围查询等，而B-树每个节点 key 和 data 在一起，则无法区间查找。
- B+树内节点不存储数据，所有 data 存储在叶节点导致查询时间复杂度固定为 log n。而B-树查询时间复杂度不固定，与 key 在树中的位置有关，最好为O(1)
- 

> [集群读优先策略](https://blog.csdn.net/leining_chris/article/details/47312645)

###### 数据写入顺序

- 写入到机器内存
  - 写入到journal（复制集情况下，写到journal同步也写到oplog中，副本从oplog中拉取数据）
- 写入到硬盘

###### 两种集群方式：**主从**、**复制集**，海量数据使用**分片**

- 主从：主写从读，主挂了从不能顶替
- 复制集：推选出主，主写从读，主挂了集合中会自动推选出新的主

> 当海量数据使用写主读从的方式，服务器负载增大，会发生主从数据不一致问题，即同步oplog日志数据到从库产生延迟
>
> **解决：**
>
> 1、更改ReadProference策略（读策略）,设置成ReadPreference.`primary`()，而平常使用ReadPreference.`secondaryPreferred`()，为了降低主库写的压力，违背初衷
>
> 2、设置写WriteConcern策略，WriteConcern.JOURNALED（日志级别，在操作日志刷盘后才返回，最快可设置等待30ms，默认100ms刷一次硬盘）性能低
>
> ​		WriteConcern.ACKNOWLEDGED（默认，写入机器内存后即返回，可能在日志刷盘的时候丢失数据）
>
> ​		WriteConcern.Majority（`推荐配置`当数据写入大多数集群中的机器才算成功，`解决这种问题`：OPLOG没有复制到从节点并且当主节点宕机后，Mongodb复制集会重新选举台secondary为primary。当secondary为primary后假设此时原先的primary被恢复了，启动之后成为了secondary节点，此时新的primary发现secondary有x数据而自己没有，那么此时就会出现数据回滚的情况，secondary会把x数据进行回滚到一个磁盘文件上（rollback_db_name），回头需要人工去处理）
>
> 3、采用分片吧

###### 排序异常

> mongodb排序是将数据查出来放到内存中排序的，但是默认mongodb设置的内存为32MB，所以报错。

**解决：**

- 加大内存
- 改排序字段加上索引

### mysql group语句

```
group by xxx  having  (having对每个分组操作，where对每条数据操作)

group_concat([distinct] 要连接的字段 [order by 排序字段 asc/desc ] [separator '分隔符'] )

concat(str1,str2,...)

concat_ws(separator, str1, str2,...)
```

### spring

![![](https://img-blog.csdn.net/20160317082518227)]

#### Bean周期

1. 实例化Bean

   - 如果实现InstantiationAwareBeanPostProcess接口，`实例化之前`，调用postProcessBeforeInstantiation()方法
   - 根据配置情况调用Bean构造函数或工厂方法实例化Bean
   - *populateBean*
   - 如果容器注册了InstantiationAwareBeanPostProcessor接口，在实例化Bean之后，调用该接口的postProcessAfterInstantiation()方法，可在这里对已经实例化的对象进行一些"梳妆打扮"
  - 如果Bean配置了属性信息，容器在这一步着手将配置值设置到Bean对应的属性中，不过在设置每个属性之前将先调用InstantiationAwareBeanPostProcessor接口的postProcessPropertyValues()方法

2. 初始化和使用Bean
   
   - **BeanNameAware的setBeanName()：**
   
   - **BeanFactoryAware的setBeanFactory()：**
   
   - **BeanPostProcessors的ProcessBeforeInitialization()**
   
   - **initializingBean的afterPropertiesSet()：**
   
   - **Bean定义文件中定义init-method：**
   
   - **BeanPostProcessors的ProcessaAfterInitialization()**
3. 销毁Bean  

   - **DisposableBean的destroy()**
   - **Bean定义文件中定义destroy-method**
   

#### Processor

​	[](https://blog.csdn.net/leileibest_437147623/article/details/80894569)

1. BeanFactoryPostProcessor	（针对整个工厂生产出来的BeanDefinition作出修改或者注册）
   - postProcessBeanDefinitionRegistry	(实现类**ConfigurationClassPostProcessor** 实现@Import、@ImportResource...涉及到的BeanDifinition注册到BeanDefinitionRegistry)
     - 修改、新增beanDefinition
   - postProcessBeanFactory （实现类**PropertyPlaceholderConfigurer**）
     - 实例化之前，读取Bean定义并做修改，比如属性修改
2. BeanPostProcessor   (作用于bean实例化、初始化前后执行)
   - BeanPostProcessors是在实例化后，初始化方法执行前、后分别执行2个方法
   - InstantiationAwareBeanPostProcessor拓展了BeanPostProcessors接口,可以在实例化Bean前（调用postProcessBeforeInstantiation方法）、后(postProcessAfterInstantiation)提供扩展的回调接口



### springboot

##### 启动流程

![](E:\learning\images\584866-20180125085800006-1407600300.png)



### mybatis

##### 多数据源切换

```
extends AbstractRoutingDataSource{
	//通过Thread设置当前线程使用哪个数据源
	@Override
    protected Object determineCurrentLookupKey() {
        return DynamicDataSourceContextHolder.getDataSourceKey();
    }
}
```

##### 四大核心类

```
执行过程顺序：
	Executor -> StatementHandler -> paramterHandler ->resultHandler
Executor执行prepareStatement() -> 设置参数 parameterize() -> 执行executeUpdate() -> resultHandler结果映射

插件拦截器Plugin：
	动态代理，执行方法的invoke方法前调用 interceptor.intercepte

```

![1573459501241](E:\learning\images\1573459501241.png)

### mysql

###### `exists`与`in`取舍

```
IN：先执行IN子查询，放到内存，然后外查询，最后做笛卡尔积  （适用于子查询记录小，外查询记录多）
exists: 先执行外查询，在循环外查询记录的size边，判断是否满足子查询条件（只需其中一条数据满足即可，适用于外查询记录少，子查询记录多）
```

###### `not in` 与 `not exists`

```
not in：内外表都进行全表扫描，没有用索引
not exists：能用到索引  任何时候用not exists都比not in快
```

#### 测试造数据

```
=======================================
DROP PROCEDURE IF EXISTS user_group_test;
delimiter $$	#将结束符设置为$$
CREATE PROCEDURE user_group_test()
BEGIN
DECLARE num INT DEFAULT(0);
WHILE num < 50 do
	INSERT INTO user_group_permission(groupId, userGroupId,userGroupName,`order`,createTime) VALUES (num, SUBSTRING(RAND(),3,9),CONCAT('用户组名称',num),SUBSTRING(RAND(),3,9),UNIX_TIMESTAMP());
	
	SET num = num + 3;
END WHILE;
END
$$
delimiter ;		#将结束符还原为；
CALL user_group_test();
=======================================
```

