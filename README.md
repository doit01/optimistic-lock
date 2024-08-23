for db lock
共享锁（S锁），又称为读锁，如果数据对象加上共享锁之后，则该数据库对象可以被其他事务查看但无法修改和删除数据。
排他锁（X锁），又称为写锁、独占锁，如果数据对象加上排他锁，则其他的事务不能对它读取【如果读取没有加锁是可以读取的】和修改
排他锁指的是一个事务在一行数据加上排他锁后，其他事务不能再在其上加其他的锁。mysql InnoDB引擎默认的修改数据语句，update,delete,insert都会自动给涉及到的数据加上排他锁，select语句默认不会加任何锁类型，如果加排他锁可以使用select …for update语句，加共享锁可以使用select … lock in share mode语句。 所以加过排他锁的数据行在其他事务中是不能修改数据的，也不能通过for update和lock in share mode锁的方式查询数据，但可以直接通过select …from…查询数据，因为普通查询没有任何锁机制

                        
原文链接：https://blog.csdn.net/weixin_43606158/article/details/107961780

# optimistic-lock
#### 给Entity添加version属性（使用@Version注解），会自动开启乐观锁
### Retry

#### pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```
#### 需要重试的方法加 @Retryable 注解

@Document
@Data
public class User {
    @Id
    private String id;
      private String name;
       @Version
    private Integer _version;
    }
    
    @RestController
@RequestMapping(path = "/user")
public class UserController {
    @RequestMapping(path = "updateWithRetry", method = RequestMethod.GET)
    public void updateWithRetry() {
        userService.updateWithRetry();
    }
}



import org.springframework.dao.OptimisticLockingFailureException;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.Date;
import java.util.List;
import java.util.UUID;
@Service
@SuppressWarnings("Duplicates")
public class UserService {


    @Retryable(OptimisticLockingFailureException.class)
    public void updateWithRetry() {
        System.out.println("updateWithRetry called.");
        User user = this.getOrCreateUser();

        User sameUser = this.getOrCreateUser();
        sameUser.setName("Another Tom Cat");
        sameUser.setUpdateTime(new Date());
        userRepository.save(sameUser);

        user.setAge(12);
        userRepository.save(user);
    }
}


public interface UserRepository extends MongoRepository<User, String> {
}
