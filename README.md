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
