# tkmybatis入门
`tkmybatis`是基于mybatis的一个封装，对常用的一些操作数据库的增删改查进行了标准封装。

## pom导入

```
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>2.1.5</version>
</dependency>
```

## mapper层编写
```java
public interface SmsRecordMapper extends Mapper<UserEntity> {
}
```
`UserEntity`为数据库对象，只需要这样写mapper即可，内部无需添加任何方法

## DB层编写
使用`@Table`指定对应的数据库表名称
```java
@Data
@Table(name = "user")
public class UserEntity extends DbEntity {
    @Id
    private Long id;

    private String name;

    private Integer age;
}
```

## Service层调用

```java
@Resource
private DictionaryMapper dictionaryMapper;
```
## 封装方法举例
### 根据条件查询list
```java
UserEntity ex = new UserEntity()
ex.setName("张三");
ex.setAge(11);
List<UserEntity> list = dictionaryMapper.select(ex);
```

### 根据条件查询一个 ，相当于limit 1
```java
UserEntity ex = new UserEntity()
ex.setName("张三");
ex.setAge(11);
UserEntity entity = dictionaryMapper.selectOne(ex);
```
### 根据模板查询 比较灵活的条件查询
```java
dictionaryMapper.selectByExample(UserEntity entity);
Example example = new Example(UserEntity.class);
Example.Criteria criteria = example.createCriteria();
criteria.andEqualTo("name", "张三");
criteria.andEqualTo("age", 11);
// 有效期大于当前时间
criteria.andGreaterThan("expireTime", LocalDateTime.now());
List<UserEntity> list = dictionaryMapper.selectByExample(example);
```
### 查询全表
```java
List<UserEntity> list = dictionaryMapper.selectAll();
```
### 根据主键查询
```java
UserEntity ex = new UserEntity()
ex.setId(1);
UserEntity entity = dictionaryMapper.selectByPrimaryKey(ex);
```

### 根据条件计算总数
```java
UserEntity ex = new UserEntity()
ex.setName("张三");
ex.setAge(11);
int num = dictionaryMapper.selectCount(ex);
```

除了以上的还有其他很多，当然也支持自定义查询和删除。
这个使用比较方便，开发效率变的比较高，可以不用自己手写Mapper层已经mapper层的SQL
