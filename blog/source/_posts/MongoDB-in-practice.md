---
title: MongoDB in practice
photos:
  - /images/pose.jpg
date: 2017-04-12 02:33:25
tags: [MongoDB]
category: [MongoDB]
---
#MongoDB in Priactice

---
## 1. install
安装路径 `G:\Tools\MongoDB`
## 2. onfig
新建`mongodb.config`放到`G:\Tools\MongoDB\bin`
内容

```
## 数据库文件目录 
dbpath=G:/Tools/MongoDB/data 
## 日志目录 
logpath=G:/Tools/MongoDB/log/mongo.log 
diaglog=3

```

<!--more-->
## 3. start mongodb
`mongod --config g:\Tools\MongoDB\bin\mongodb.config`
**将MongoDB服务器作为Windows服务运行**
`mongod --config g:\Tools\MongoDB\bin\mongodb.config --install`
## 4. connect mongodb
`mongo`
we can use **robomongo** to operator mongodb with GUI
## 5. Spring Support
    1. use springdata JAP
```java
public interface PersonDao extends MongoRepository<Person, ObjectId> {

    @Query(value = "{'age' : {'$gte' : ?0, '$lte' : ?1}, 'name':?2 }",fields="{ 'name' : 1, 'age' : 1}")
    List<Person> findByAge(int age1, int age2, String name);

}
```
    2. use mongoTemplate
```java
@Repository("personMongoImpl")
public class PersonMongoImpl implements PersonMongoDao {

    @Resource
    private MongoTemplate mongoTemplate;

    @Override
    public List<Person> findAll() {
        return mongoTemplate.findAll(Person.class,"person");
    }

    @Override
    public void insertPerson(Person person) {
        mongoTemplate.insert(person,"person");
    }

    @Override
    public void removePerson(String userName) {
        mongoTemplate.remove(Query.query(Criteria.where("name").is(userName)),"person");
    }

    @Override
    public void updatePerson() {
        mongoTemplate.updateMulti(Query.query(Criteria.where("age").gt(3).lte(5)), Update.update("age",3),"person");
    }

    @Override
    public List<Person> findForRequery(String userName) {
        return mongoTemplate.find(Query.query(Criteria.where("name").is(userName)),Person.class);
    }
}
```
