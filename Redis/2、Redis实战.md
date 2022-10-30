# 1、短信登陆



## 1.1、导入黑马点评项目



### 1.1.1、导入SQL

导入课前资料中提供的 hmdp.sql SQL文件。MySQL采用5.7及以上版本。

其中的表有：

- tb_blog：商户日记表
- tb_blog_comments
- tb_follow：用户关注表
- tb_seckill_voucher
- tb_shop：商户信息表
- tb_shop_type：商户类型表
- tb_sign
- tb_user：用户表
- tb_user_info：用户详情表
- tb_voucher：优惠券表
- tb_voucher_order：优惠券的订单表



### 1.1.2、有关当前模型

手机或者app端发起请求，请求我们的nginx服务器，nginx基于七层模型走的事HTTP协议，可以实现基于Lua直接绕开tomcat访问redis，也可以作为静态资源服务器，轻松扛下上万并发， 负载均衡到下游tomcat服务器，打散流量，我们都知道一台4核8G的tomcat，在优化和处理简单业务的加持下，大不了就处理1000左右的并发， 经过nginx的负载均衡分流后，利用集群支撑起整个项目，同时nginx在部署了前端项目后，更是可以做到动静分离，进一步降低tomcat服务的压力，这些功能都得靠nginx起作用，所以nginx是整个项目中重要的一环。



在tomcat支撑起并发流量后，我们如果让tomcat直接去访问Mysql，根据经验Mysql企业级服务器只要上点并发，一般是16或32 核心cpu，32 或64G内存，像企业级mysql加上固态硬盘能够支撑的并发，大概就是4000起~7000左右，上万并发， 瞬间就会让Mysql服务器的cpu，硬盘全部打满，容易崩溃，所以我们在高并发场景下，会选择使用mysql集群，同时为了进一步降低Mysql的压力，同时增加访问的性能，我们也会加入Redis，同时使用Redis集群使得Redis对外提供更好的服务。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653059409865.png" alt="1653059409865" style="zoom:80%;" />



### 1.1.3、导入后端项目

将hm-dianping项目导入idea后，在浏览器访问http://localhost:8081/shop-type/list，如果可以看到数据则证明没有问题。



### 1.1.4、导入前端工程

将资料中的nginx文件夹复制到任意不包含中文、特殊字符和空格的目录下



### 1.1.5、运行前端项目

在nginx所在目录下打开一个CMD窗口，输入命令：

```sh
start nginx.exe
```



打开浏览器，F12打开控制台，改为手机模式，然后再访问http://127.0.0.1:8080，有界面显示则证明没有问题。



## 1.2、基于Session实现登录流程

**发送验证码**：

用户在提交手机号后，会校验手机号是否合法，如果不合法，则要求用户重新输入手机号

如果手机号合法，后台此时生成对应的验证码，同时将验证码进行保存，然后再通过短信的方式将验证码发送给用户

**短信验证码登录、注册**：

用户将验证码和手机号进行输入，后台从session中拿到当前验证码，然后和用户输入的验证码进行校验，如果不一致，则无法通过校验，如果一致，则后台根据手机号查询用户，如果用户不存在，则为用户创建账号信息，保存到数据库，无论是否存在，都会将用户信息保存到session中，方便后续获得当前登录信息

**校验登录状态**：

用户在请求时候，会从cookie中携带者JsessionId到后台，后台通过JsessionId从session中拿到用户信息，如果没有session信息，则进行拦截，如果有session信息，则将用户信息保存到threadLocal中，并且放行

![1653066208144](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653066208144-1666666524134.png)



> ThreadLocal：线程域。如果直接将用户请求数据保存到一个变量中，可能会有并发安全问题。而ThreadLocal会将数据保存到每一个线程的内部，这样每个线程都有自己的空间，不会互相干扰。



## 1.3、实现发送短信验证码功能

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653067054461.png" alt="1653067054461" style="zoom:80%;" />



### 1.3.1、发送验证码

```java
@Override
public Result sendCode(String phone, HttpSession session) {
    // 1. 校验手机号
    if (RegexUtils.isPhoneInvalid(phone)) {
        // 2. 如果不符合，返回错误信息
        return Result.fail("手机号格式错误");
    }
    // 3. 符合，生成一个验证码
    String code = RandomUtil.randomNumbers(6);
    // 4. 保存验证码到session
    session.setAttribute("code", code);
    // 5. 发送验证码
    log.debug("发送短信验证码成功，验证码：{}", code);
    // 返回ok
    return Result.ok();
}
```



### 1.3.2、登录

```java
@Override
public Result login(LoginFormDTO loginForm, HttpSession session) {
    // 1. 校验手机号和验证码
    String phone = loginForm.getPhone();
    if (RegexUtils.isPhoneInvalid(phone)) {
        // 2. 如果不符合，返回错误信息
        return Result.fail("手机号格式错误");
    }
    // 3. 校验验证码
    Object cacheCode = session.getAttribute("code");
    String code = loginForm.getCode();
    if (cacheCode == null || !cacheCode.toString().equals(code)) {
        // 4. 不一致，报错
        return Result.fail("验证码错误");
    }
    // 5. 一致，根据手机号查询用户
    User user = query().eq("phone", phone).one();
    // 6. 判断用户是否存在
    if (user == null) {
        // 7. 不存在，创建新用户并保存
        user = createUserWithPhone(phone);
    }
    // 8. 保存用户信息到session中
    session.setAttribute("user", user);
    return Result.ok();
}

private User createUserWithPhone(String phone) {
    // 1. 创建用户
    User user = new User();
    user.setPhone(phone);
    user.setNickName(SystemConstants.USER_NICK_NAME_PREFIX + RandomUtil.randomString(10));
    // 2. 保存用户
    save(user);
    return user;
}
```



## 1.4、实现登录拦截功能



### 1.4.1、tomcat运行原理

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653068196656.png" alt="1653068196656"  />

当用户发起请求时，会访问我们向tomcat注册的端口，任何程序想要运行，都需要有一个线程对当前端口号进行监听，tomcat也不例外，当监听线程知道用户想要和tomcat连接连接时，会由监听线程创建socket连接，socket都是成对出现的，用户通过socket互相传递数据，当tomcat端的socket接受到数据后，此时监听线程会从tomcat的线程池中取出一个线程执行用户请求，在我们的服务部署到tomcat后，线程会找到用户想要访问的工程，然后用这个线程转发到工程中的controller，service，dao中，并且访问对应的DB，在用户执行完请求后，再统一返回，再找到tomcat端的socket，再将数据写回到用户端的socket，完成请求和响应。

通过以上讲解，我们可以得知：每个用户其实对应都是去找tomcat线程池中的一个线程来完成工作的， 使用完成后再进行回收，既然每个请求都是独立的，所以在每个用户去访问我们的工程时，我们可以使用threadlocal来做到线程隔离，每个线程操作自己的一份数据。



### 1.4.2、Threadlocal

如果看过threadLocal的源码，你会发现在threadLocal中，无论是他的put方法和他的get方法， 都是先获得当前用户的线程，然后从线程中取出线程的成员变量map，只要线程不一样，map就不一样，所以可以通过这种方式来做到线程隔离。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653068874258.png" alt="1653068874258" style="zoom:80%;" />



### 1.4.3、定义拦截器

```java
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 1. 获取session
        HttpSession session = request.getSession();
        // 2. 获取session中的用户
        Object user = session.getAttribute("user");
        // 3.判断用户是否存在
        if (user == null) {
            // 4. 不存在，直接拦截，返回401状态码
            response.setStatus(401);
            return false;
        }
        // 5. 存在，保存用户信息到ThreadLocal
        UserHolder.saveUser((User) user);
        // 6. 放行
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 移除用户
        UserHolder.removeUser();
    }
}
```



### 1.4.4、使拦截器生效

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .excludePathPatterns(
                        "/user/code",
                        "/user/login",
                        "/blog/got",
                        "/shop/**",
                        "/shop-type/**",
                        "/upload/**",
                        "/voucher/**"
                );
    }
}
```



## 1.5、隐藏用户敏感信息

我们通过浏览器观察到此时用户的全部信息都在，这样极为不安全，所以我们应当在返回用户信息之前，将用户的敏感信息进行隐藏，采用的核心思路就是书写一个UserDto对象，这个UserDto对象就没有敏感信息了，我们在返回前，将有用户敏感信息的User对象转化成没有敏感信息的UserDto对象，那么就能够避免这个问题了。

**在登录方法处修改**

```java
//7.保存用户信息到session中
session.setAttribute("user", BeanUtils.copyProperties(user,UserDTO.class));
```

**在拦截器处：**

```java
//5.存在，保存用户信息到Threadlocal
UserHolder.saveUser((UserDTO) user);
```

**在UserHolder处：将user对象换成UserDTO**

```java
public class UserHolder {
    private static final ThreadLocal<UserDTO> tl = new ThreadLocal<>();

    public static void saveUser(UserDTO user){
        tl.set(user);
    }

    public static UserDTO getUser(){
        return tl.get();
    }

    public static void removeUser(){
        tl.remove();
    }
}
```



## 1.6、session共享问题

**核心思路分析**：

每个tomcat中都有一份属于自己的session，假设用户第一次访问第一台tomcat，并且把自己的信息存放到第一台服务器的session中，但是第二次这个用户访问到了第二台tomcat，那么在第二台服务器上，肯定没有第一台服务器存放的session，此时整个登录拦截功能就会出现问题，我们能如何解决这个问题呢？早期的方案是session拷贝，就是说虽然每个tomcat上都有不同的session，但是每当任意一台服务器的session修改时，都会同步给其他的Tomcat服务器的session，这样的话，就可以实现session的共享了。

但是这种方案具有两个大问题：

1. 每台服务器中都有完整的一份session数据，服务器压力过大
2. session拷贝数据时，可能会出现延迟

所以后来采用的方案都是基于redis来完成，我们把session换成redis，redis数据本身就是共享的，就可以避免session共享的问题了

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653069893050.png" alt="1653069893050" style="zoom:80%;" />



## 1.7、Redis代替session



### 1.7.1、设计key的结构

首先我们要思考一下利用redis来存储数据，那么到底使用哪种结构呢？由于存入的数据比较简单，我们可以考虑使用String，或者是使用哈希，如下图，如果使用String，它的value会多占用一点空间，如果使用哈希，则他的value中只会存储他数据本身，如果不是特别在意内存，其实使用String就可以。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653319261433.png" alt="1653319261433" style="zoom:80%;" />



### 1.7.2、设计key的具体细节

我们可以使用String结构，就是一个简单的key，value键值对的方式，但是关于key的处理，session他是每个用户都有自己的session，但是redis的key是共享的，就不能使用code了

在设计key的时候，需要满足两点：

- key要具有唯一性
- key要方便携带

如果采用 phone：手机号这个的数据来存储当然是可以的，但是如果把这样的敏感数据存储到redis中并且从页面中带过来毕竟不太合适，所以我们在后台生成一个随机串 token，然后让前端带来这个 token 就能完成我们的整体逻辑了



### 1.7.3、整体访问流程

当注册完成后，用户去登录会去校验用户提交的手机号和验证码，是否一致，如果一致，则根据手机号查询用户信息，不存在则新建，最后将用户数据保存到redis，并且生成 token 作为 redis 的 key，当我们校验用户是否登录时，会去携带着 token 进行访问，从 redis 中取出 token 对应的 value，判断是否存在这个数据，如果没有则拦截，如果存在则将其保存到threadLocal中，并且放行。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653319474181.png" alt="1653319474181" style="zoom: 80%;" />



## 1.8、基于Redis的短信登陆

**UserServiceImpl代码**

```java
@Resource
private StringRedisTemplate stringRedisTemplate;

@Override
public Result sendCode(String phone, HttpSession session) {
    // 1. 校验手机号
    if (RegexUtils.isPhoneInvalid(phone)) {
        // 2. 如果不符合，返回错误信息
        return Result.fail("手机号格式错误");
    }
    // 3. 符合，生成一个验证码
    String code = RandomUtil.randomNumbers(6);
    // 4. 保存验证码到 redis
    stringRedisTemplate.opsForValue().set(RedisConstants.LOGIN_CODE_KEY + phone, code, RedisConstants.LOGIN_CODE_TTL, TimeUnit.MINUTES);
    // 5. 发送验证码
    log.debug("发送短信验证码成功，验证码：{}", code);
    // 返回ok
    return Result.ok();
}

@Override
public Result login(LoginFormDTO loginForm, HttpSession session) {
    // 1. 校验手机号和验证码
    String phone = loginForm.getPhone();
    if (RegexUtils.isPhoneInvalid(phone)) {
        // 2. 如果不符合，返回错误信息
        return Result.fail("手机号格式错误");
    }
    // 3. 从redis获取验证码并校验
    String cacheCode = stringRedisTemplate.opsForValue().get(RedisConstants.LOGIN_CODE_KEY + phone);
    String code = loginForm.getCode();
    if (cacheCode == null || !cacheCode.equals(code)) {
        // 4. 不一致，报错
        return Result.fail("验证码错误");
    }
    // 5. 一致，根据手机号查询用户
    User user = query().eq("phone", phone).one();
    // 6. 判断用户是否存在
    if (user == null) {
        // 7. 不存在，创建新用户并保存
        user = createUserWithPhone(phone);
    }
    // 8. 保存用户信息到 redis 中
    // 8.1 随机生成token，作为登录令牌
    String token = UUID.randomUUID().toString(true);
    // 8.2 将User对象转为HashMap存储
    UserDTO userDTO = BeanUtil.copyProperties(user, UserDTO.class);
    Map<String, Object> userMap = 
        BeanUtil.beanToMap(userDTO, new HashMap<>(),
                           CopyOptions.create()
                           .setIgnoreNullValue(true)
                           // 将值转为字符串类型
                           .setFieldValueEditor((fieldName, fieldValue) -> fieldValue.toString()));
    // 8.3 存储
    stringRedisTemplate.opsForHash().putAll(RedisConstants.LOGIN_USER_KEY + token, userMap);
    // 8.4 设置token有效期
    stringRedisTemplate.expire(RedisConstants.LOGIN_USER_KEY + token, RedisConstants.LOGIN_USER_TTL, TimeUnit.MINUTES);
    // 8.5 返回token
    return Result.ok(token);
}
```



## 1.9、解决状态登录刷新问题



### 1.9.1、初始方案思路总结

在这个方案中，他确实可以使用对应路径的拦截，同时刷新登录token令牌的存活时间，但是现在这个拦截器他只是拦截需要被拦截的路径，假设当前用户访问了一些不需要拦截的路径，那么这个拦截器就不会生效，所以此时令牌刷新的动作实际上就不会执行。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653320822964.png" alt="1653320822964" style="zoom: 80%;" />



### 1.9.2、优化方案

既然之前的拦截器无法对不需要拦截的路径生效，那么我们可以添加一个拦截器，在第一个拦截器中拦截所有的路径，把第二个拦截器做的事情放入到第一个拦截器中，同时刷新令牌，因为第一个拦截器有了threadLocal的数据，所以此时第二个拦截器只需要判断拦截器中的user对象是否存在即可，完成整体刷新功能。

![1653320764547](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653320764547.png)

**RefreshTokenInterceptor**

```java
public class RefreshTokenInterceptor implements HandlerInterceptor {

    private StringRedisTemplate stringRedisTemplate;

    public RefreshTokenInterceptor(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 1. 获取请求头中的token
        String token = request.getHeader("authorization");
        if (StrUtil.isBlank(token)) {
            return true;
        }
        // 2. 基于 token获取 redis中的用户
        Map<Object, Object> userMap = stringRedisTemplate.opsForHash().entries(RedisConstants.LOGIN_USER_KEY + token);
        // 3.判断用户是否存在
        if (userMap.isEmpty()) {
            return true;
        }
        // 5. 将查询到的hash数据转为User对象
        UserDTO userDTO = BeanUtil.fillBeanWithMap(userMap, new UserDTO(), false);
        // 6. 存在，保存用户信息到ThreadLocal
        UserHolder.saveUser(userDTO);
        // 7. 刷新token有效期
        stringRedisTemplate.expire(RedisConstants.LOGIN_USER_KEY + token, RedisConstants.LOGIN_USER_TTL, TimeUnit.MINUTES);
        // 8. 放行
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 移除用户
        UserHolder.removeUser();
    }
}
```



**LoginInterceptor**

```java
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 1. 判断是否需要拦截 (ThreadLocal中是否有用户)
        if (UserHolder.getUser() == null) {
            // 没有，需要拦截，设置状态码
            response.setStatus(401);
            // 拦截
            return false;
        }
        // 有用户则放行
        return true;
    }
}
```



**MvcConfig**

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 拦截器默认按add顺序执行
        // 也可以通过order控制，order越大执行优先级越低
        registry.addInterceptor(new RefreshTokenInterceptor(stringRedisTemplate)).order(0);
        registry.addInterceptor(new LoginInterceptor())
                .excludePathPatterns(
                        "/user/code",
                        "/user/login",
                        "/blog/got",
                        "/shop/**",
                        "/shop-type/**",
                        "/upload/**",
                        "/voucher/**"
                ).order(1);
    }
}
```



# 2、商户查询缓存



## 2.1、什么是缓存

实际开发中，系统需要 "避震器"，防止过高的数据访问猛冲系统，导致其操作线程无法及时处理信息而瘫痪；

这在实际开发中对企业讲，对产品口碑，用户评价都是致命的；所以企业非常重视缓存技术；

**缓存(**Cache)，就是数据交换的**缓冲区**，俗称的缓存就是**缓冲区内的数据**，一般从数据库中获取，存储于本地代码。

```java
Static final ConcurrentHashMap<K,V> map = new ConcurrentHashMap<>(); // 本地用于高并发

static final Cache<K,V> USER_CACHE = CacheBuilder.newBuilder().build(); // 用于redis等缓存

Static final Map<K,V> map =  new HashMap(); // 本地缓存
```

由于其被**Static**修饰，所以随着类的加载而被加载到**内存之中**，作为本地缓存，由于其又被**final**修饰，所以其引用和对象之间的关系是固定的，不能改变，因此不用担心赋值 (=) 导致缓存失效；



### 2.1.1、为什么要使用缓存

一句话:因为**速度快，好用**

缓存数据存储于代码中，而代码运行在内存中，内存的读写性能远高于磁盘，缓存可以大大降低**用户访问并发量带来的**服务器读写压力。

实际开发过程中，企业的数据量，少则几十万，多则几千万，这么大数据量，如果没有缓存来作为 "避震器"，系统是几乎撑不住的，所以企业会大量运用到缓存技术；

但是缓存也会增加代码复杂度和运营的成本：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220523214414123.png" alt="image-20220523214414123" style="zoom: 67%;" />



### 2.1.2、如何使用缓存

实际开发中，会构筑多级缓存来使系统运行速度进一步提升，例如：本地缓存与redis中的缓存并发使用

**浏览器缓存**：主要是存在于浏览器端的缓存

**应用层缓存：**可以分为tomcat本地缓存，比如之前提到的map，或者是使用redis作为缓存

**数据库缓存：**在数据库中有一片空间是 buffer pool，增改查数据都会先加载到mysql的缓存中

**CPU缓存：**当代计算机最大的问题是cpu性能提升了，但内存读写速度没有跟上，所以为了适应当下的情况，增加了cpu的L1，L2，L3级的缓存

![image-20220523212915666](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220523212915666.png)



## 2.2、添加商户缓存

在查询商户信息时，我们是直接操作从数据库中去进行查询的，大致逻辑是这样，直接查询数据库肯定慢，所以我们需要增加缓存

```java
@GetMapping("/{id}")
public Result queryShopById(@PathVariable("id") Long id) {
    //这里是直接查询数据库
    return shopService.queryById(id);
}
```



### 2.2.1、缓存模型和思路

标准的操作方式就是查询数据库之前先查询缓存，如果缓存数据存在，则直接从缓存中返回，如果缓存数据不存在，再查询数据库，然后将数据存入redis。

![1653322097736](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653322097736.png)



### 2.2.2、代码实现

代码思路：如果缓存有，则直接返回，如果缓存不存在，则查询数据库，然后存入redis。

```java
@Override
public Result queryById(Long id) {
    // 1. 从Redis查询商铺缓存
    String shopJson = stringRedisTemplate.opsForValue().get(RedisConstants.CACHE_SHOP_KEY + id);
    // 2. 判断是否存在
    if (StrUtil.isNotBlank(shopJson)) {
        // 3. 存在，直接返回
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return Result.ok(shop);
    }
    // 4. 不存在，根据id查询数据库
    Shop shop = getById(id);
    // 5. 不存在，返回错误
    if (shop == null) {
        return Result.fail("店铺不存在！");
    }
    // 6. 存在，写入Redis
    stringRedisTemplate.opsForValue().set(RedisConstants.CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(shop));
    // 7. 返回
    return Result.ok(shop);
}
```



## 2.3、缓存更新策略

缓存更新是redis为了节约内存而设计出来的一个东西，主要是因为内存数据宝贵，当我们向redis插入太多数据，此时就可能会导致缓存中的数据过多，所以redis会对部分数据进行更新，或者把他叫为淘汰更合适。

**内存淘汰：**redis自动进行，当redis内存达到设定的max-memery的时候，会自动触发淘汰机制，淘汰掉一些不重要的数据 (可以自己设置策略方式)

**超时剔除：**当我们给redis设置了过期时间ttl之后，redis会将超时的数据进行删除，方便继续使用缓存

**主动更新：**可以手动调用方法把缓存删掉，通常用于解决缓存和数据库不一致问题

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221026134706665.png" alt="image-20221026134706665" style="zoom: 80%;" />



业务场景：

- 低一致性需求：使用内存淘汰机制。例如店铺类型的查询缓存
- 高一致性需求：主动更新，并以超时剔除作为兜底方案。例如店铺详情查询的缓存



### 2.3.1、数据库缓存不一致解决方案

由于我们的**缓存的数据源来自于数据库**，而数据库的**数据是会发生变化的**，因此，如果当数据库中**数据发生变化，而缓存却没有同步**，此时就会有**一致性问题存在**，其后果是：

用户使用缓存中的过时数据，就会产生类似多线程数据安全问题，从而影响业务，产品口碑等。

有如下几种方案：

- Cache Aside Pattern 人工编码方式：缓存调用者在更新完数据库后再去更新缓存，也称之为双写方案
- Read/Write Through Pattern：由系统本身完成，数据库与缓存的问题交由系统本身去处理
- Write Behind Caching Pattern：调用者只操作缓存，其他线程去异步处理数据库，实现最终一致

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653322857620.png" alt="1653322857620" style="zoom: 67%;" />



### 2.3.2、数据库和缓存不一致采用什么方案

综合考虑使用方案一，但是方案一调用者如何处理呢？

操作缓存和数据库时有三个问题需要考虑：

- 删除缓存还是更新缓存？
  - 更新缓存：每次更新数据库都更新缓存，无效写操作较多
  - 删除缓存：更新数据库时让缓存失效，查询时再更新缓存
- 如何保证缓存与数据库的操作的同时成功或失败？
  - 单体系统，将缓存与数据库操作放在一个事务
  - 分布式系统，利用TCC等分布式事务方案

- 先操作缓存还是先操作数据库？
  - 先删除缓存，再操作数据库
  - 先操作数据库，再删除缓存



如果采用第一个方案，那么假设我们每次操作数据库后，都操作缓存，但是中间如果没有人查询，那么这个更新动作实际上只有最后一次生效，中间的更新动作意义并不大，我们可以把缓存删除，等待再次查询时，将缓存中的数据加载出来。

我们应当是先操作数据库，再删除缓存。因为，如果选择第一种方案，在两个线程并发来访问时，假设线程1先来，他先把缓存删了，此时线程2过来，他查询缓存数据并不存在，此时他写入缓存，当他写入缓存后，线程1再执行更新动作，实际上写入缓存的就是旧的数据，新的数据被旧数据覆盖了。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653323595206.png" alt="1653323595206" style="zoom:80%;" />

这两种方案都会出现问题，但是方案二的概率会更低一些，因为查询完数据库后就执行写入缓存的操作，写缓存的时间较短，出现并发问题的概率也就低了。



### 2.3.3、总结

1. 低一致性需求：使用Redis自带的内存淘汰机制
2. 高一致性需求：主动更新，并以超时剔除作为兜底方案
   - 读操作：
     - 缓存命中则直接返回
     - 缓存未命中则查询数据库，并写入缓存，设定超时时间
   - 写操作：
     - 先写数据库，然后再删除缓存
     - 要确保数据库与缓存操作的原子性



## 2.4、实现商铺的双写一致

修改ShopController中的业务逻辑，满足下面的需求：

- 根据id查询店铺时，如果缓存未命中，则查询数据库，将数据库结果写入缓存，并设置超时时间；

- 根据id修改店铺时，先修改数据库，再删除缓存



### 2.4.1、修改queryById方法

**设置redis缓存时添加过期时间**

```java
@Override
public Result queryById(Long id) {
    // 1. 从Redis查询商铺缓存
    String shopJson = stringRedisTemplate.opsForValue().get(RedisConstants.CACHE_SHOP_KEY + id);
    // 2. 判断是否存在
    if (StrUtil.isNotBlank(shopJson)) {
        // 3. 存在，直接返回
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return Result.ok(shop);
    }
    // 4. 不存在，根据id查询数据库
    Shop shop = getById(id);
    // 5. 不存在，返回错误
    if (shop == null) {
        return Result.fail("店铺不存在！");
    }
    // 6. 存在，写入Redis，并设置过期时间
    stringRedisTemplate.opsForValue().set(RedisConstants.CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(shop), RedisConstants.CACHE_SHOP_TTL, TimeUnit.MINUTES);
    // 7. 返回
    return Result.ok(shop);
}
```



### 2.4.2、添加update方法

代码分析：采用删除策略来解决双写问题，当我们修改了数据之后，然后把缓存中的数据进行删除，查询时发现缓存中没有数据，则会从mysql中加载最新的数据，从而避免数据库和缓存不一致的问题。

```java
@Override
@Transactional
public Result update(Shop shop) {
    Long id = shop.getId();
    if (id == null) {
        return Result.fail("店铺id不能为空");
    }
    // 1. 更新数据库
    updateById(shop);
    // 2. 删除缓存
    stringRedisTemplate.delete(RedisConstants.CACHE_SHOP_KEY + id);
    return null;
}
```



## 2.5、缓存穿透

缓存穿透：缓存穿透是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求都会打到数据库。

常见的解决方案有两种：

* 缓存空对象
  * 优点：实现简单，维护方便
  * 缺点：
    * 额外的内存消耗
    * 可能造成短期的不一致
* 布隆过滤
  * 优点：内存占用较少，没有多余key
  * 缺点：
    * 实现复杂
    * 存在误判可能



**缓存空对象思路分析**：当我们客户端访问不存在的数据时，先请求redis，但是此时redis中没有数据，此时会访问到数据库，但是数据库中也没有数据，这个数据穿透了缓存，直击数据库，我们都知道数据库能够承载的并发不如redis这么高，如果大量的请求同时过来访问这种不存在的数据，这些请求就都会访问到数据库，简单的解决方案就是哪怕这个数据在数据库中也不存在，我们也把这个数据存入到redis中去，这样，下次用户过来访问这个不存在的数据，那么在redis中也能找到这个数据就不会进入到缓存了。



**布隆过滤**：布隆过滤器其实采用的是哈希思想来解决这个问题，通过一个庞大的二进制数组，走哈希思想去判断当前这个要查询的这个数据是否存在，如果布滤器判断存在，则放行，这个请求会去访问redis，哪怕此时redis中的数据过期了，但是数据库中一定存在这个数据，在数据库中查询出来这个数据后，再将其放入到redis中，假设布隆过滤器判断这个数据不存在，则直接返回。这种方式优点在于节约内存空间，存在误判，误判原因在于：布隆过滤器走的是哈希思想，可能存在哈希冲突

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653326156516.png" alt="1653326156516" style="zoom:80%;" />



## 2.6、解决商品查询的缓存穿透

在原来的逻辑中，我们如果发现这个数据在mysql中不存在，直接就返回404了，这样是会存在缓存穿透问题的

现在的逻辑中：如果这个数据不存在，我们不会返回404，还是会把这个数据写入到Redis中，并且将value设置为空，当再次发起查询时，我们如果发现命中之后，判断这个value是否是null，如果是null，则是之前写入的数据，证明是缓存穿透数据，如果不是，则直接返回数据。

![1653327124561](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653327124561.png)



```java
@Override
public Result queryById(Long id) {
    // 1. 从Redis查询商铺缓存
    String shopJson = stringRedisTemplate.opsForValue().get(RedisConstants.CACHE_SHOP_KEY + id);
    // 2. 判断是否存在
    if (StrUtil.isNotBlank(shopJson)) {
        // 3. 存在，直接返回
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return Result.ok(shop);
    }
    // 判断命中的是否是空值
    if (shopJson != null) {
        // 返回一个错误信息
        return Result.fail("店铺信息不存在！");
    }
    // 4. 不存在，根据id查询数据库
    Shop shop = getById(id);
    // 5. 不存在，返回错误
    if (shop == null) {
        // 将空值写入redis
        stringRedisTemplate.opsForValue().set(RedisConstants.CACHE_SHOP_KEY + id, "", RedisConstants.CACHE_NULL_TTL, TimeUnit.MINUTES);
        // 返回错误信息
        return Result.fail("店铺不存在！");
    }
    // 6. 存在，写入Redis，并设置过期时间
    stringRedisTemplate.opsForValue().set(RedisConstants.CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(shop), RedisConstants.CACHE_SHOP_TTL, TimeUnit.MINUTES);
    // 7. 返回
    return Result.ok(shop);
}
```



**小结**：

缓存穿透产生的原因是什么？

- 用户请求的数据在缓存中和数据库中都不存在，不断发起这样的请求，给数据库带来巨大压力

缓存穿透的解决方案有哪些？

* 缓存null值
* 布隆过滤
* 增强id的复杂度，避免被猜测id规律
* 做好数据的基础格式校验
* 加强用户权限校验
* 做好热点参数的限流



## 2.7、缓存雪崩

缓存雪崩是指在同一时段大量的缓存key同时失效或者Redis服务宕机，导致大量请求到达数据库，带来巨大压力。

解决方案：

* 给不同的Key的TTL添加随机值
* 利用Redis集群提高服务的可用性
* 给缓存业务添加降级限流策略
* 给业务添加多级缓存

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653327884526.png" alt="1653327884526" style="zoom:80%;" />



## 2.8、缓存击穿

缓存击穿问题也叫热点Key问题，就是一个被高并发访问并且缓存重建业务较复杂的key突然失效了，无数的请求访问会在瞬间给数据库带来巨大的冲击。

常见的解决方案有两种：

* 互斥锁
* 逻辑过期

逻辑分析：假设线程1在查询缓存之后，本来应该去查询数据库，然后把这个数据重新加载到缓存的，此时只要线程1走完这个逻辑，其他线程就都能从缓存中加载这些数据了，但是假设在线程1没有走完的时候，后续的线程2，线程3，线程4同时过来访问当前这个方法， 那么他们就会同一时刻来访问查询缓存，都没查到，接着同一时间去访问数据库，同时的去执行数据库代码，对数据库访问压力过大。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653328022622.png" alt="1653328022622" style="zoom:80%;" />



### 2.8.1、互斥锁

因为锁能实现互斥性。假设线程过来，只能一个人一个人的来访问数据库，从而避免对于数据库访问压力过大，但这也会影响查询的性能，因为此时会让查询的性能从并行变成了串行，我们可以采用tryLock方法 + double check来解决这样的问题。

假设现在线程1过来访问，他查询缓存没有命中，但是此时他获得到了锁的资源，那么线程1就会一个人去执行逻辑，假设现在线程2过来，线程2在执行过程中，并没有获得到锁，那么线程2就可以进行到休眠，直到线程1把锁释放后，线程2获得到锁，然后再来执行逻辑，此时就能够从缓存中拿到数据了。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653328288627.png" alt="1653328288627" style="zoom:80%;" />



### 2.8.2、逻辑过期

我们之所以会出现这个缓存击穿问题，主要原因是在于我们对key设置了过期时间，假设我们不设置过期时间，其实就不会有缓存击穿的问题，但是不设置过期时间，这样数据不就一直占用我们内存了吗，我们可以采用逻辑过期方案。

我们把过期时间设置在 redis的value中，注意：这个过期时间并不会直接作用于redis，而是我们后续通过逻辑去处理。假设线程1去查询缓存，然后从value中判断出来当前的数据已经过期了，此时线程1去获得互斥锁，那么其他线程会进行阻塞，获得了锁的线程他会开启一个线程2去进行 以前的重构数据的逻辑，直到新开的线程完成这个逻辑后，才释放锁， 而线程1直接进行返回，假设现在线程3过来访问，由于线程线程2持有着锁，所以线程3无法获得锁，线程3也直接返回数据，只有等到新开的线程2把重建数据构建完后，其他线程才能走返回正确的数据。

这种方案巧妙在于，异步的构建缓存，缺点在于在构建完缓存之前，返回的都是脏数据。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653328663897.png" alt="1653328663897" style="zoom:80%;" />



### 2.8.3、对比

**互斥锁方案：**由于保证了互斥性，所以数据一致，且实现简单，因为仅仅只需要加一把锁而已，也没其他的事情需要操心，所以没有额外的内存消耗，缺点在于有锁就有死锁问题的发生，且只能串行执行性能肯定受到影响

**逻辑过期方案：** 线程读取过程中不需要等待，性能好，有一个额外的线程持有锁去进行重构数据，但是在重构数据完成前，其他的线程只能返回之前的数据，且实现起来麻烦

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653357522914.png" alt="1653357522914" style="zoom:80%;" />



## 2.9、互斥锁解决缓存击穿

相较于原来从缓存中查询不到数据后直接查询数据库而言，现在的方案是 进行查询之后，如果从缓存没有查询到数据，则进行互斥锁的获取，获取互斥锁后，判断是否获得到了锁，如果没有获得到，则休眠，过一会再进行尝试，直到获取到锁为止，才能进行查询

如果获取到了锁的线程，再去进行查询，查询后将数据写入redis，再释放锁，返回数据，利用互斥锁就能保证只有一个线程去执行操作数据库的逻辑，防止缓存击穿

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653357860001.png" alt="1653357860001" style="zoom:80%;" />

利用redis的setnx方法来表示获取锁，该方法含义是redis中如果没有这个key，则插入成功并返回1，在stringRedisTemplate中返回true，如果有这个key则插入失败，则返回0，在stringRedisTemplate返回false，我们可以通过true，或者是false，来表示是否有线程成功插入key，成功插入的key的线程我们认为他就是获得到锁的线程。

```java
private boolean tryLock(String key) {
    Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", RedisConstants.LOCK_SHOP_TTL, TimeUnit.SECONDS);
    // 避免自动拆箱时出现异常
    return BooleanUtil.isTrue(flag);
}

private void unLock(String key) {
    stringRedisTemplate.delete(key);
}
```

**操作代码：**

```java
public Shop queryWithMutex(Long id) {
    // 1. 从Redis查询商铺缓存
    String shopJson = stringRedisTemplate.opsForValue().get(RedisConstants.CACHE_SHOP_KEY + id);
    // 2. 判断是否存在
    if (StrUtil.isNotBlank(shopJson)) {
        // 3. 存在，直接返回
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return shop;
    }
    // 判断命中的是否是空值
    if (shopJson != null) {
        // 返回一个错误信息
        return null;
    }
    // 4. 实现缓存重建
    // 4.1 获取互斥锁
    Shop shop = null;
    try {
        boolean isLock = tryLock(RedisConstants.LOCK_SHOP_KEY + id);
        // 4.2 判断是否获取成功
        if (!isLock) {
            // 4.3 失败，则休眠并重试
            Thread.sleep(50);
            return queryWithMutex(id);
        }
        // 4.4 成功，根据id查询数据库
        shop = getById(id);
        // 模拟重建的延时，增加并发几率
        Thread.sleep(200);
        // 5. 不存在，返回错误
        if (shop == null) {
            // 将空值写入redis
            stringRedisTemplate.opsForValue().set(RedisConstants.CACHE_SHOP_KEY + id, "", RedisConstants.CACHE_NULL_TTL, TimeUnit.MINUTES);
            // 返回错误信息
            return null;
        }
        // 6. 存在，写入Redis，并设置过期时间
        stringRedisTemplate.opsForValue().set(RedisConstants.CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(shop), RedisConstants.CACHE_SHOP_TTL, TimeUnit.MINUTES);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }finally {
        // 7. 释放互斥锁
        unLock(RedisConstants.LOCK_SHOP_KEY + id);
    }
    // 8. 返回
    return shop;
}
```



## 2.10、逻辑过期解决缓存击穿

**需求：修改根据id查询商铺的业务，基于逻辑过期方式来解决缓存击穿问题**

当用户开始查询redis时，判断是否命中，如果没有命中则直接返回空数据，不查询数据库，而一旦命中后，将value取出，判断value中的过期时间是否满足，如果没有过期，则直接返回redis中的数据，如果过期，则在开启独立线程后直接返回之前的数据，独立线程去重构数据，重构完成后释放互斥锁。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653360308731.png" alt="1653360308731" style="zoom:80%;" />

因为现在redis中存储的数据的value需要带上过期时间，此时要么修改原来的实体类，要么新建一个类封装实体类和过期时间



1. 新建一个实体类，我们采用第二个方案，这个方案，对原来代码没有侵入性。

   ```java
   @Data
   public class RedisData {
       private LocalDateTime expireTime;
       private Object data;
   }
   ```

2. 在**ShopServiceImpl** 新增此方法，利用单元测试进行缓存预热

   ```java
   @Override
   public void saveShopToRedis(Long id, Long expireSeconds) {
       // 1. 查询店铺数据
       Shop shop = getById(id);
       // 2. 封装逻辑过期时间
       RedisData redisData = new RedisData();
       redisData.setData(shop);
       redisData.setExpireTime(LocalDateTime.now().plusSeconds(expireSeconds));
       // 3. 写入Redis
       stringRedisTemplate.opsForValue().set(RedisConstants.CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(redisData));
   }
   ```

   **测试类**

   ```java
   @Test
   void testSaveShop() {
       shopService.saveShopToRedis(1L, 10L);
   }
   ```

3. 业务代码

   ```java
   private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);
   
   public Shop queryWithLogicalExpire(Long id) {
       // 1. 从Redis查询商铺缓存
       String shopJson = stringRedisTemplate.opsForValue().get(RedisConstants.CACHE_SHOP_KEY + id);
       // 2. 判断是否存在
       if (StrUtil.isBlank(shopJson)) {
           // 3. 不存在，直接返回null
           return null;
       }
       // 4. 命中，需要先把json反序列化为对象
       RedisData redisData = JSONUtil.toBean(shopJson, RedisData.class);
       Shop shop = JSONUtil.toBean((JSONObject) redisData.getData(), Shop.class);
       LocalDateTime expireTime = redisData.getExpireTime();
       // 5. 判断是否过期
       if (expireTime.isAfter(LocalDateTime.now())) {
           // 5.1 未过期，直接返回店铺信息
           return shop;
       }
       // 5.2 已过期，需要缓存重建
       // 6. 缓存重建
       // 6.1 获取互斥锁
       boolean isLock = tryLock(RedisConstants.LOCK_SHOP_KEY + id);
       // 6.1 判断是否获取锁成功
       if (isLock) {
           // 6.3 成功，开启独立线程，实现缓存重建
           CACHE_REBUILD_EXECUTOR.submit(() -> {
               try {
                   // 重建缓存
                   saveShopToRedis(id, 20L);
               } catch (Exception e) {
                   throw new RuntimeException(e);
               }finally {
                   // 释放锁
                   unLock(RedisConstants.LOCK_SHOP_KEY + id);
               }
           });
       }
       // 6.4 失败，返回过期的店铺信息
       return shop;
   }
   ```



## 2.11、封装Redis工具类

基于StringRedisTemplate封装一个缓存工具类，满足下列需求：

- 将任意Java对象序列化为json并存储在string类型的key中，并且可以设置TTL过期时间
- 将任意Java对象序列化为json并存储在string类型的key中，并且可以设置逻辑过期时间，用于处理缓存击穿问题
- 根据指定的key查询缓存，并反序列化为指定类型，利用缓存空值的方式解决缓存穿透问题
- 根据指定的key查询缓存，并反序列化为指定类型，需要利用逻辑过期解决缓存击穿问题

```java
@Slf4j
@Component
public class CacheClient {
    private final StringRedisTemplate stringRedisTemplate;

    public CacheClient(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    public void set(String key, Object value, Long time, TimeUnit timeUnit) {
        String json = JSONUtil.toJsonStr(value);
        stringRedisTemplate.opsForValue().set(key, json, time, timeUnit);
    }

    public void setWithLogicalExpire(String key, Object value, Long time, TimeUnit timeUnit) {
        // 设置逻辑过期
        RedisData redisData = new RedisData();
        redisData.setData(value);
        redisData.setExpireTime(LocalDateTime.now().plusSeconds(timeUnit.toSeconds(time)));
        String json = JSONUtil.toJsonStr(redisData);
        stringRedisTemplate.opsForValue().set(key, json);
    }

    public <R, ID> R queryWithPassThrough(
            String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit timeUnit) {
        String key = keyPrefix + id;
        // 1. 从Redis查询商铺缓存
        String json = stringRedisTemplate.opsForValue().get(key);
        // 2. 判断是否存在
        if (StrUtil.isNotBlank(json)) {
            // 3. 存在，直接返回
            return JSONUtil.toBean(json, type);
        }
        // 判断命中的是否是空值
        if (json != null) {
            // 返回一个错误信息
            return null;
        }
        // 4. 不存在，根据id查询数据库
        R r = dbFallback.apply(id);
        // 5. 不存在，返回错误
        if (r == null) {
            // 将空值写入redis
            stringRedisTemplate.opsForValue().set(key, "", RedisConstants.CACHE_NULL_TTL, TimeUnit.MINUTES);
            // 返回错误信息
            return null;
        }
        // 6. 存在，写入Redis，并设置过期时间
        set(key, r, time, timeUnit);
        // 7. 返回
        return r;
    }

    private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);
    
    public <R, ID> R queryWithLogicalExpire(
            String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit timeUnit) {
        String key = keyPrefix + id;
        // 1. 从Redis查询商铺缓存
        String json = stringRedisTemplate.opsForValue().get(RedisConstants.CACHE_SHOP_KEY + id);
        // 2. 判断是否存在
        if (StrUtil.isBlank(json)) {
            // 3. 不存在，直接返回null
            return null;
        }
        // 4. 命中，需要先把json反序列化为对象
        RedisData redisData = JSONUtil.toBean(json, RedisData.class);
        R r = JSONUtil.toBean((JSONObject) redisData.getData(), type);
        LocalDateTime expireTime = redisData.getExpireTime();
        // 5. 判断是否过期
        if (expireTime.isAfter(LocalDateTime.now())) {
            // 5.1 未过期，直接返回店铺信息
            return r;
        }
        // 5.2 已过期，需要缓存重建
        // 6. 缓存重建
        // 6.1 获取互斥锁
        boolean isLock = tryLock(RedisConstants.LOCK_SHOP_KEY + id);
        // 6.1 判断是否获取锁成功
        if (isLock) {
            // 6.3 成功，开启独立线程，实现缓存重建
            CACHE_REBUILD_EXECUTOR.submit(() -> {
                try {
                    // 重建缓存
                    // 查询数据库
                    R r1 = dbFallback.apply(id);
                    // 写入redis
                    setWithLogicalExpire(key, r1, time, timeUnit);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }finally {
                    // 释放锁
                    unLock(RedisConstants.LOCK_SHOP_KEY + id);
                }
            });
        }
        // 6.4 失败，返回过期的店铺信息
        return r;
    }

    private boolean tryLock(String key) {
        Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", RedisConstants.LOCK_SHOP_TTL, TimeUnit.SECONDS);
        // 避免自动拆箱时出现异常
        return BooleanUtil.isTrue(flag);
    }

    private void unLock(String key) {
        stringRedisTemplate.delete(key);
    }
}
```



# 3、优惠券秒杀



## 3.1、全局唯一ID

每个店铺都可以发布优惠券：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653362612286.png" alt="1653362612286" style="zoom: 80%;" />

当用户抢购时，就会生成订单并保存到tb_voucher_order这张表中，而订单表如果使用数据库自增ID就存在一些问题：

* id的规律性太明显
* 受单表数据量的限制

场景分析：如果我们的id具有太明显的规则，用户或者说商业对手很容易猜测出来我们的一些敏感信息，比如商城在一天时间内，卖出了多少单，这明显不合适。

场景分析二：随着我们商城规模越来越大，mysql的单表的容量不宜超过500W，数据量过大之后，我们要进行拆库拆表，但拆分表了之后，他们从逻辑上讲他们是同一张表，所以他们的id是不能一样的， 于是乎我们需要保证id的唯一性。

**全局ID生成器**，是一种在分布式系统下用来生成全局唯一ID的工具，一般要满足下列特性：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653363100502.png" alt="1653363100502"  />

为了增加ID的安全性，我们可以不直接使用Redis自增的数值，而是拼接一些其它信息：

![1653363172079](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653363172079.png)

ID的组成部分

- 符号位：1bit，永远为0
- 时间戳：31bit，以秒为单位，可以使用69年
- 序列号：32bit，秒内的计数器，支持每秒产生2^32个不同ID



## 3.2、Redis实现全局唯一Id

```java
@Component
public class RedisIdWork {
    /**
     * 开始时间戳
     */
    private static final long BEGIN_TIMESTAMP = 1640995200L;
    /**
     * 序列号的位数
     */
    private static final int COUNT_BITS = 32;

    private StringRedisTemplate stringRedisTemplate;

    public RedisIdWork(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    public Long nextId(String keyPrefix) {
        // 1. 生成时间戳
        LocalDateTime now = LocalDateTime.now();
        long nowSecond = now.toEpochSecond(ZoneOffset.UTC);
        long timestamp = nowSecond - BEGIN_TIMESTAMP;
        // 2. 生成序列号，每天下的单用当天日期作为key
        // 2.1 获取当前日期，加 :便于统计
        String date = now.format(DateTimeFormatter.ofPattern("yyyy:MM:dd"));
        Long count = stringRedisTemplate.opsForValue().increment("icr" + keyPrefix + ":" + date);
        // 3. 拼接并返回
        // 位运算符：<<转为二进制后，向左移动指定位数
        // 时间戳为long类型，向左移动32位后，空出序列号的位置，空位补0
        // 采用 | 或运算，把序列号加进去即可，0|0 = 0, 0|1 = 1...
        return timestamp << COUNT_BITS | count;
    }
}
```



测试类：

```java
@Test
void testIdWork() throws InterruptedException {
    // 设置300个线程等待
    CountDownLatch latch = new CountDownLatch(300);
    Runnable task = () -> {
        for (int i = 0; i < 100; i++) {
            Long id = redisIdWork.nextId("order");
            System.out.println("id = " + id);
        }
        // 每个线程执行完毕就减一
        latch.countDown();
    };
    long begin = System.currentTimeMillis();
    for (int i = 0; i < 300; i++) {
        es.submit(task);
    }
    // 等待300个线程执行完毕
    latch.await();
    long end = System.currentTimeMillis();
    System.out.println("time = " + (end - begin));
}
```



> CountDownLatch：
>
> 名为信号枪，主要作用是同步协调在多线程的等待于唤醒问题
>
> 如果没有CountDownLatch ，那么由于程序是异步的，当异步程序没有执行完时，主线程就已经执行完了，然后我们期望的是分线程全部走完之后，主线程再走，所以我们此时需要使用到CountDownLatch
>
> CountDownLatch 中有两个最重要的方法
>
> 1. countDown
>
> 2. await
>
> await 方法 是阻塞方法，我们担心分线程没有执行完时，main线程就先执行，所以使用await可以让main线程阻塞，那么什么时候main线程不再阻塞呢？当CountDownLatch内部维护的变量变为0时，就不再阻塞，直接放行。
>
> 那么什么时候 CountDownLatch维护的变量变为0 呢，我们只需要调用一次countDown，内部变量就减少1，我们让分线程和变量绑定，执行完一个分线程就减少一个变量，当分线程全部走完，CountDownLatch 维护的变量就是0，此时await就不再阻塞，统计出来的时间也就是所有分线程执行完后的时间。



**小结**：

全局唯一ID生成策略

- UUID
- Redis自增
- snowflake (雪花)算法
- 数据库自增



Redis自增ID策略：

- 每天一个key，方便统计订单量
- ID构造是 时间戳 + 计数器



## 3.3、添加优惠券

每个店铺都可以发布优惠券，分为平价券和特价券。平价券可以任意购买，而特价券需要秒杀抢购：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653365145124.png" alt="1653365145124" style="zoom:80%;" />

tb_voucher：优惠券的基本信息，优惠金额、使用规则等

tb_seckill_voucher：优惠券的库存、开始抢购时间，结束抢购时间。特价优惠券才需要填写这些信息

平价卷由于优惠力度并不是很大，所以是可以任意领取

而代金券由于优惠力度大，所以像第二种卷，就得限制数量，从表结构上也能看出，特价卷除了具有优惠卷的基本信息以外，还具有库存，抢购时间，结束时间等等字段

**新增普通卷代码**：VoucherController

```java
@PostMapping
public Result addVoucher(@RequestBody Voucher voucher) {
    voucherService.save(voucher);
    return Result.ok(voucher.getId());
}
```

**新增秒杀卷代码**：VoucherController

```java
@PostMapping("seckill")
public Result addSeckillVoucher(@RequestBody Voucher voucher) {
    voucherService.addSeckillVoucher(voucher);
    return Result.ok(voucher.getId());
}
```

VoucherServiceImpl

```java
@Override
@Transactional
public void addSeckillVoucher(Voucher voucher) {
    // 保存优惠券
    save(voucher);
    // 保存秒杀信息
    SeckillVoucher seckillVoucher = new SeckillVoucher();
    seckillVoucher.setVoucherId(voucher.getId());
    seckillVoucher.setStock(voucher.getStock());
    seckillVoucher.setBeginTime(voucher.getBeginTime());
    seckillVoucher.setEndTime(voucher.getEndTime());
    seckillVoucherService.save(seckillVoucher);
    // 保存秒杀库存到Redis中
    stringRedisTemplate.opsForValue().set(SECKILL_STOCK_KEY + voucher.getId(), voucher.getStock().toString());
}
```



## 3.4、实现秒杀下单

下单核心思路：当我们点击抢购时，会触发右侧的请求，我们只需要编写对应的controller即可

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653365839526.png" alt="1653365839526" style="zoom:80%;" />



下单时需要判断两点：

* 秒杀是否开始或结束，如果尚未开始或已经结束则无法下单
* 库存是否充足，不足则无法下单

下单核心逻辑分析：

当用户开始进行下单，我们应当去查询优惠卷信息，查询到优惠卷信息，判断是否满足秒杀条件

比如时间是否充足，如果时间充足，则进一步判断库存是否足够，如果两者都满足，则扣减库存，创建订单，然后返回订单id，如果有一个条件不满足则直接结束。

![1653366238564](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653366238564.png)

```java
@Override
@Transactional
public Result seckillVoucher(Long voucherId) {
    // 1. 查询优惠券
    SeckillVoucher voucher = seckillVoucherService.getById(voucherId);
    // 2. 判断秒杀是否已经开始
    if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
        // 尚未开始
        return Result.fail("秒杀尚未开始！");
    }
    // 3. 判断秒杀是否已经结束
    if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
        return Result.fail("秒杀已经结束！");
    }
    // 4. 判断库存是否充足
    if (voucher.getStock() < 1) {
        // 库存不足
        return Result.fail("库存不足！");
    }
    // 5. 扣减库存
    boolean success = seckillVoucherService.update()
        .setSql("stock = stock - 1")
        .eq("voucher_id", voucherId).update();
    if (!success) {
        return Result.fail("库存不足！");
    }
    // 6. 创建订单
    VoucherOrder voucherOrder = new VoucherOrder();
    // 订单id
    Long orderId = redisIdWork.nextId("order:");
    voucherOrder.setId(orderId);
    // 用户id
    Long userId = UserHolder.getUser().getId();
    voucherOrder.setUserId(userId);
    // 代金券id
    voucherOrder.setVoucherId(voucherId);
    save(voucherOrder);
    // 7. 返回订单id
    return Result.ok(orderId);
}
```



## 3.5、库存超卖问题分析

有关超卖问题分析：在我们原有代码中是这么写的：

```java
// 4. 判断库存是否充足
if (voucher.getStock() < 1) {
    // 库存不足
    return Result.fail("库存不足！");
}
// 5. 扣减库存
boolean success = seckillVoucherService.update()
    .setSql("stock = stock - 1")
    .eq("voucher_id", voucherId).update();
if (!success) {
    return Result.fail("库存不足！");
}
```

假设线程1过来查询库存，判断出来库存大于1，正准备去扣减库存，但是还没有来得及去扣减，此时线程2过来，线程2也去查询库存，发现这个数量一定也大于1，那么这两个线程都会去扣减库存，最终多个线程相当于一起去扣减库存，此时就会出现库存的超卖问题。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653368335155.png" alt="1653368335155" style="zoom:80%;" />

超卖问题是典型的多线程安全问题，针对这一问题的常见解决方案就是加锁。对于加锁，我们通常有两种解决方案：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653368562591.png" alt="1653368562591" style="zoom:80%;" />

**悲观锁：**

 悲观锁可以实现对于数据的串行化执行，比如syn和lock都是悲观锁的代表。悲观锁中又可以再细分为公平锁，非公平锁，可重入锁等

**乐观锁：**

会有一个版本号，每次操作数据会对版本号+1，再提交回数据时，会去校验是否比之前的版本大1，如果大1，则进行操作成功，这套机制的核心逻辑在于，如果在操作过程中，版本号只比原来大1，那么就意味着操作过程中没有人对他进行过修改，他的操作就是安全的，如果不大1，则数据被修改过，当然乐观锁还有一些变种的处理方式比如cas

乐观锁的典型代表就是cas，利用cas进行无锁化机制加锁，var5 是操作前读取的内存值，while中的var1+var2 是预估值，如果预估值 == 内存值，则代表中间没有被人修改过，此时就将新值去替换 内存值

其中do while 是为了在操作失败时，再次进行自旋操作，即把之前的逻辑再操作一次。

```java
int var5;
do {
    var5 = this.getIntVolatile(var1, var2);
} while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

return var5;
```



**课程中的使用方式：**

课程中的使用方式是没有像cas一样带自旋的操作，也没有对version的版本号+1，他的操作逻辑是在操作时，对版本号进行+1 操作，然后要求version 如果是1 的情况下才能操作，那么第一个线程在操作后，数据库中的version变成了2，但是他自己满足version=1 ，所以没有问题，此时线程2执行，线程2 最后也需要加上条件version =1 ，但是现在由于线程1已经操作过了，所以线程2，操作时就不满足version=1 的条件了，所以线程2无法执行成功

![1653369268550](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653369268550.png)



## 3.6、乐观锁解决超卖问题



### 3.6.1、方案一

VoucherOrderServiceImpl 在扣减库存时，改为：

```java
boolean success = seckillVoucherService.update()
            .setSql("stock= stock -1") //set stock = stock -1
            .eq("voucher_id", voucherId).eq("stock",voucher.getStock()) //where id = ？ and stock = ?
    		.update();
```

以上逻辑的核心含义是：只要我扣减库存时的库存和之前我查询到的库存是一样的，就意味着没有人在中间修改过库存，那么此时就是安全的，但是以上这种方式通过测试发现会有很多失败的情况。

失败的原因在于：在使用乐观锁过程中假设100个线程同时都拿到了100的库存，然后大家一起去进行扣减，但是100个人中只有1个人能扣减成功，其他的人在处理时，他们在扣减时，库存已经被修改过了，所以此时其他线程都会失败。



### 3.6.2、方案二

之前的方式要修改前后都保持一致，但是这样我们分析过，成功的概率太低，所以我们的乐观锁需要变一下，改成stock大于0 即可

```java
boolean success = seckillVoucherService.update()
            .setSql("stock= stock -1")
            .eq("voucher_id", voucherId).update().gt("stock",0); //where id = ? and stock > 0
```



针对cas中的自旋压力过大，我们可以使用Longaddr这个类去解决

Java8 提供的一个对AtomicLong改进后的一个类，LongAdder

大量线程并发更新一个原子性的时候，天然的问题就是自旋，会导致并发性问题，当然这也比我们直接使用syn来的好

所以利用这么一个类，LongAdder来进行优化

如果获取某个值，则会对cell和base的值进行递增，最后返回一个完整的值

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653370271627.png" alt="1653370271627" style="zoom:80%;" />



### 3.6.3、总结

超卖在这样的线程安全问题，解决方案有哪些？

1. 悲观锁：添加同步锁，让线程串行执行
   - 优点：简单粗暴
   - 缺点：性能一般
2. 乐观锁：不加锁，在更新时判断是否有其他线程在修改
   - 优点：性能好
   - 缺点：存在成功率低的问题



## 3.7、一人一单

需求：修改秒杀业务，要求同一个优惠券，一个用户只能下一单

**现在的问题在于：**

目前的情况是，一个人可以无限制的抢这个优惠卷，我们应当增加一层逻辑，让一个用户只能下一个单，而不是让一个用户下多个单

具体操作逻辑：比如时间是否充足，如果时间充足，则进一步判断库存是否足够，然后再根据优惠卷id和用户id查询是否已经下过这个订单，如果下过这个订单，则不再下单，否则进行下单

![1653371854389](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653371854389.png)

**初步代码--增加一人一单逻辑**

```java
@Override
@Transactional
public Result seckillVoucher(Long voucherId) {
    // 1. 查询优惠券
    SeckillVoucher voucher = seckillVoucherService.getById(voucherId);
    // 2. 判断秒杀是否已经开始
    if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
        // 尚未开始
        return Result.fail("秒杀尚未开始！");
    }
    // 3. 判断秒杀是否已经结束
    if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
        return Result.fail("秒杀已经结束！");
    }
    // 4. 判断库存是否充足
    if (voucher.getStock() < 1) {
        // 库存不足
        return Result.fail("库存不足！");
    }

    // 5. 一人一单
    Long userId = UserHolder.getUser().getId();
    // 5.1 查询订单
    int count = query().eq("user_id", userId).eq("voucher_id", voucherId).count();
    // 5.2 判断是否存在
    if (count > 0) {
        // 用户已经购买过了
        return Result.fail("用户已经购买过一次！");
    }

    // 6. 扣减库存
    boolean success = seckillVoucherService.update()
        .setSql("stock = stock - 1")
        .eq("voucher_id", voucherId).gt("stock", 0)
        .update();
    if (!success) {
        return Result.fail("库存不足！");
    }

    // 7. 创建订单
    VoucherOrder voucherOrder = new VoucherOrder();
    // 订单id
    Long orderId = redisIdWork.nextId("order:");
    voucherOrder.setId(orderId);
    // 用户id
    voucherOrder.setUserId(userId);
    // 代金券id
    voucherOrder.setVoucherId(voucherId);
    save(voucherOrder);
    // 8. 返回订单id
    return Result.ok(orderId);
}
```

**存在问题**：现在的问题还是和之前一样，并发查询数据库，都不存在订单，所以我们还是需要加锁，但是乐观锁比较适合更新数据，而现在是插入数据，所以我们需要使用悲观锁操作。

**注意**：在这里提到了非常多的问题，我们需要慢慢的来思考，首先我们的初始方案是封装了一个createVoucherOrder方法，同时为了确保他线程安全，在方法上添加了一把synchronized 锁。

但是这样添加锁，锁的粒度太粗了，在使用锁过程中，控制**锁粒度**是一个非常重要的事情，因为如果锁的粒度太大，会导致每个线程进来都会锁住，所以我们需要去控制锁的粒度，以下这段代码需要修改为：

intern() 这个方法是从常量池中拿到数据，如果我们直接使用userId.toString() 他拿到的对象实际上是不同的对象，new出来的对象，我们使用锁必须保证锁必须是同一把，所以我们需要使用intern()方法

```java
@Transactional
public synchronized Result createVoucherOrder(Long voucherId) {
    // 5. 一人一单
    Long userId = UserHolder.getUser().getId();
    synchronized (userId.toString().intern()) {
        // 5.1 查询订单
        int count = query().eq("user_id", userId).eq("voucher_id", voucherId).count();
        // 5.2 判断是否存在
        if (count > 0) {
            // 用户已经购买过了
            return Result.fail("用户已经购买过一次！");
        }

        // 6. 扣减库存
        boolean success = seckillVoucherService.update()
            .setSql("stock = stock - 1")
            .eq("voucher_id", voucherId).gt("stock", 0)
            .update();
        if (!success) {
            return Result.fail("库存不足！");
        }

        // 7. 创建订单
        VoucherOrder voucherOrder = new VoucherOrder();
        // 订单id
        Long orderId = redisIdWork.nextId("order:");
        voucherOrder.setId(orderId);
        // 用户id
        voucherOrder.setUserId(userId);
        // 代金券id
        voucherOrder.setVoucherId(voucherId);
        save(voucherOrder);
        // 8. 返回订单id
        return Result.ok(orderId);
    }
}
```

但是以上代码还是存在问题，问题的原因在于当前方法被spring的事务控制，如果你在方法内部加锁，可能会导致当前方法事务还没有提交，但是锁已经释放也会导致问题，所以我们选择将当前方法整体包裹起来，确保事务不会出现问题。

把 synchronized 添加到seckillVoucher 方法中，就能保证事务的特性，同时也控制了锁的粒度

```java
Long userId = UserHolder.getUser().getId();
synchronized (userId.toString().intern()) {
    createVoucherOrder(voucherId);
}
```

但是以上做法依然有问题，调用的方法其实是this.的方式调用的，并不是通过service的代理对象调用，所以事务想要生效，还得利用代理对象来操作事务：

```java
Long userId = UserHolder.getUser().getId();
synchronized (userId.toString().intern()) {
    // 获取代理对象 (事务)
    IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
    return proxy.createVoucherOrder(voucherId);
}
```

同时需要导入依赖，并配置启动类：

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
</dependency>
```

```java
// 暴露代理对象，不暴露的话获取不到代理对象
@EnableAspectJAutoProxy(exposeProxy = true)
```



## 3.8、集群环境下的并发问题

通过加锁可以解决在单机情况下的一人一单安全问题，但是在集群模式下就不行了。

1. 我们将服务启动两份，端口分别为8081和8082：

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221030113349625.png" alt="image-20221030113349625" style="zoom:80%;" />

2. 修改nginx的conf目录下的nginx.conf文件，配置反向代理和负载均衡：

   ```properties
   upstream backend {
       server 127.0.0.1:8081 max_fails=5 fail_timeout=10s weight=1;
       #server 127.0.0.1:8082 max_fails=5 fail_timeout=10s weight=1;
   }
   ```

   

**有关锁失效原因分析**

由于现在我们部署了多个tomcat，每个tomcat都有一个属于自己的jvm，那么假设在服务器A的tomcat内部，有两个线程，这两个线程由于使用的是同一份代码，那么他们的锁对象是同一个，是可以实现互斥的，但是如果现在是服务器B的tomcat内部，又有两个线程，但是他们的锁对象写的虽然和服务器A一样，但是锁对象却不是同一个，所以线程3和线程4可以实现互斥，但是却无法和线程1和线程2实现互斥，这就是 集群环境下，syn锁失效的原因，在这种情况下，我们就需要使用分布式锁来解决这个问题。

![1653374044740](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653374044740.png)



# 4、分布式锁



## 4.1、基本原理和实现方式对比

分布式锁：满足分布式系统或集群模式下多进程可见并且互斥的锁。

分布式锁的核心思想就是让大家都使用同一把锁，只要大家使用的是同一把锁，那么我们就能锁住线程，不让线程进行，让程序串行执行，这就是分布式锁的核心思路

![1653374296906](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653374296906.png)

那么分布式锁他应该满足一些什么样的条件呢？

- 可见性：多个线程都能看到相同的结果。注意：这个地方说的可见性并不是并发编程中指的内存可见性，只是说多个进程之间都能感知到变化的意思

- 互斥：互斥是分布式锁的最基本的条件，使得程序串行执行
- 高可用：程序不易崩溃，时时刻刻都保证较高的可用性
- 高性能：由于加锁本身就让性能降低，所有对于分布式锁本身需要他就较高的加锁性能和释放锁性能
- 安全性：安全也是程序中必不可少的一环



常见的分布式锁有三种：

Mysql：mysql本身就带有锁机制，但是由于mysql性能本身一般，所以采用分布式锁的情况下，其实使用mysql作为分布式锁比较少见

Redis：redis作为分布式锁是非常常见的一种使用方式，现在企业级开发中基本都使用redis或者zookeeper作为分布式锁，利用setnx这个方法，如果插入key成功，则表示获得到了锁，如果有人插入成功，其他人插入失败则表示无法获得到锁，利用这套逻辑来实现分布式锁

Zookeeper：zookeeper也是企业级开发中较好的一个实现分布式锁的方案

![1653382219377](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653382219377.png)



## 4.2、Redis分布式锁的实现核心思路

实现分布式锁时需要实现的两个基本方法：

- 获取锁

  - 互斥：确保只能有一个线程获取锁

    ```sh
    # 添加锁，NX是互斥，EX是设置超时时间
    set lock thread1 NX EX 10
    ```

  - 非阻塞：尝试一次，成功返回true，失败返回false

- 释放锁：

  * 手动释放
  * 超时释放：获取锁时添加一个超时时间

![1653382669900](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653382669900.png)

核心思路：

我们利用redis 的setNx 方法，当有多个线程进入时，我们就利用该方法，第一个线程进入时，redis 中就有这个key 了。如果返回结果是1，则表示他抢到了锁，那么他去执行业务，然后再删除锁，退出锁逻辑，没有抢到锁的线程，等待一定时间后重试即可

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653382830810.png" alt="1653382830810" style="zoom:80%;" />



## 4.3、实现分布式锁版本一

- 加锁逻辑

**锁的基本接口**

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1656079017728.png" alt="1656079017728" style="zoom: 67%;" />

**SimpleRedisLock**

利用setnx方法进行加锁，同时增加过期时间，防止死锁，此方法可以保证加锁和增加过期时间具有原子性

```java
public class SimpleRedisLock implements ILock{

    private String name;
    private StringRedisTemplate stringRedisTemplate;

    public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {
        this.name = name;
        this.stringRedisTemplate = stringRedisTemplate;
    }

    private static final String KEY_PREFIX = "lock:";

    /**
     * 获取锁
     */
    @Override
    public boolean tryLock(long timeoutSec) {
        // 获取线程标识
        long threadId = Thread.currentThread().getId();
        // 获取锁
        Boolean success = stringRedisTemplate.opsForValue()
                .setIfAbsent(KEY_PREFIX + name, threadId + "", timeoutSec, TimeUnit.SECONDS);
        return Boolean.TRUE.equals(success);
    }
    
    /**
     * 释放锁
     */
    @Override
    public void unlock() {
        stringRedisTemplate.delete(KEY_PREFIX + name);
    }
}
```



**修改业务代码**

```java
@Override
public Result seckillVoucher(Long voucherId) {
    // 1. 查询优惠券
    SeckillVoucher voucher = seckillVoucherService.getById(voucherId);
    // 2. 判断秒杀是否已经开始
    if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
        // 尚未开始
        return Result.fail("秒杀尚未开始！");
    }
    // 3. 判断秒杀是否已经结束
    if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
        return Result.fail("秒杀已经结束！");
    }
    // 4. 判断库存是否充足
    if (voucher.getStock() < 1) {
        // 库存不足
        return Result.fail("库存不足！");
    }

    Long userId = UserHolder.getUser().getId();
    // 创建锁对象
    SimpleRedisLock lock = new SimpleRedisLock("order:" + userId, stringRedisTemplate);
    // 获取锁
    boolean isLock = lock.tryLock(1200L);
    // 判断是否获取锁成功
    if (!isLock) {
        // 获取锁失败，返回错误或重试
        return Result.fail("不允许重复下单！");
    }
    try {
        // 获取代理对象 (事务)
        IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
        return proxy.createVoucherOrder(voucherId);
    }finally {
        // 释放锁
        lock.unlock();
    }
}
```



## 4.4、Redis分布式锁误删情况说明

持有锁的线程在锁的内部出现了阻塞，导致他的锁自动释放，这时其他线程，线程2来尝试获得锁，就拿到了这把锁，然后线程2在持有锁执行过程中，线程1反应过来，继续执行，而线程1执行过程中，走到了删除锁逻辑，此时就会把本应该属于线程2的锁进行删除，这就是误删别人锁的情况。

解决方案就是在每个线程释放锁的时候，去判断一下当前这把锁是否属于自己，如果属于自己，则不进行锁的删除，假设还是上边的情况，线程1卡顿，锁自动释放，线程2进入到锁的内部执行逻辑，此时线程1反应过来，然后删除锁，但是线程1，一看当前这把锁不是属于自己，于是不进行删除锁逻辑，当线程2走到删除锁逻辑时，如果没有卡过自动释放锁的时间点，则判断当前这把锁是属于自己的，于是删除这把锁。

![1653385920025](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653385920025.png)



## 4.5、解决Redis分布式锁误删问题

需求：修改之前的分布式锁实现，满足在获取锁时存入线程标示（可以用UUID表示）

在释放锁时先获取锁中的线程标示，判断是否与当前线程标示一致

* 如果一致则释放锁
* 如果不一致则不释放锁

核心逻辑：在存入锁时，放入自己线程的标识，在删除锁时，判断当前这把锁的标识是不是自己存入的，如果是，则进行删除，如果不是，则不进行删除。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653387398820.png" alt="1653387398820" style="zoom:80%;" />

具体代码如下：

```java
public class SimpleRedisLock implements ILock{

    private String name;
    private StringRedisTemplate stringRedisTemplate;

    public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {
        this.name = name;
        this.stringRedisTemplate = stringRedisTemplate;
    }

    private static final String KEY_PREFIX = "lock:";
    private static final String ID_PREFIX = UUID.randomUUID().toString(true) + "-";

    @Override
    public boolean tryLock(long timeoutSec) {
        // 获取线程标识
        String threadId = ID_PREFIX + Thread.currentThread().getId();
        // 获取锁
        Boolean success = stringRedisTemplate.opsForValue()
                .setIfAbsent(KEY_PREFIX + name, threadId, timeoutSec, TimeUnit.SECONDS);
        return Boolean.TRUE.equals(success);
    }

    @Override
    public void unlock() {
        // 获取线程标识
        String threadId = ID_PREFIX + Thread.currentThread().getId();
        String id = stringRedisTemplate.opsForValue().get(KEY_PREFIX + name);
        // 判断标识是否一致
        if (id.equals(threadId)) {
            // 释放锁
            stringRedisTemplate.delete(KEY_PREFIX + name);
        }
    }
}
```



在我们修改完此处代码后，我们重启工程，然后启动两个线程，第一个线程持有锁后，手动释放锁，第二个线程此时进入到锁内部，再放行第一个线程，此时第一个线程由于锁的value值并非是自己，所以不能释放锁，也就无法删除别人的锁，此时第二个线程能够正确释放锁，通过这个案例初步说明我们解决了锁误删的问题。



## 4.6、分布式锁的原子性问题

线程1现在持有锁之后，在执行业务逻辑过程中，他正准备删除锁，而且已经走到了条件判断的过程中，比如他已经拿到了当前这把锁确实是属于他自己的，正准备删除锁，但是此时他的锁到期了，那么此时线程2进来，但是线程1他会接着往后执行，当他卡顿结束后，他直接就会执行删除锁那行代码，相当于条件判断并没有起到作用，这就是删锁时的原子性问题，之所以有这个问题，是因为线程1的拿锁，比锁，删锁，实际上并不是原子性的，我们要防止刚才的情况发生。

![1653387764938](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653387764938.png)



## 4.7、Lua脚本解决多条命令原子性问题

Redis提供了Lua脚本功能，在一个脚本中编写多条Redis命令，确保多条命令执行时的原子性。Lua是一种编程语言，它的基本语法可以参考网站：https://www.runoob.com/lua/lua-tutorial.html，这里重点介绍Redis提供的调用函数。

我们可以使用lua去操作redis，又能保证他的原子性，这样就可以实现拿锁比锁删锁是一个原子性动作了，作为Java程序员这一块并不作一个简单要求，并不需要大家过于精通，只需要知道他有什么作用即可。

这里重点介绍Redis提供的调用函数，语法如下：

```lua
redis.call('命令名称', 'key', '其它参数', ...)
```

例如，我们要执行set name jack，则脚本是这样：

```lua
# 执行 set name jack
redis.call('set', 'name', 'jack')
```

例如，我们要先执行set name Rose，再执行get name，则脚本如下：

```lua
# 先执行 set name jack
redis.call('set', 'name', 'Rose')
# 再执行 get name
local name = redis.call('get', 'name')
# 返回
return name
```

写好脚本以后，需要用Redis命令来调用脚本，调用脚本的常见命令如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653392181413.png" alt="1653392181413" style="zoom: 67%;" />

例如，我们要执行 redis.call('set', 'name', 'jack') 这个脚本，语法如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653392218531.png" alt="1653392218531" style="zoom:80%;" />

如果脚本中的key、value不想写死，可以作为参数传递。key类型参数会放入KEYS数组，其它参数会放入ARGV数组，在脚本中可以从KEYS和ARGV数组获取这些参数：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653392438917.png" alt="1653392438917" style="zoom:80%;" />

接下来我们来回一下释放锁的逻辑：

释放锁的业务流程是这样的

1. 获取锁中的线程标示
2. 判断是否与指定的标示（当前线程标示）一致
3. 如果一致则释放锁（删除）
4. 如果不一致则什么都不做

如果用Lua脚本来表示则是这样的：

最终我们操作redis的拿锁比锁删锁的lua脚本就会变成这样

```lua
-- 这里的 KEYS[1] 就是锁的key，这里的ARGV[1] 就是当前线程标示
-- 比较线程标识与锁中的标识是否一致
if(redis.call('get', KEYS[1]) == ARGV[1]) then
    -- 一致则释放锁 del
    return redis.call('del', KEYS[1])
end
-- 不一致，则返回0
return 0
```



## 4.8、利用Java代码调用Lua脚本改造分布式锁

lua脚本本身并不需要大家花费太多时间去研究，只需要知道如何调用，大致是什么意思即可，所以在笔记中并不会详细的去解释这些lua表达式的含义。

我们的RedisTemplate中，可以利用execute方法去执行lua脚本，参数对应关系就如下图：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1653393304844.png" alt="1653393304844" style="zoom:80%;" />

**Java代码**

```java
private static final DefaultRedisScript<Long> UNLOCK_SCRIPT;
    static {
        UNLOCK_SCRIPT = new DefaultRedisScript<>();
        UNLOCK_SCRIPT.setLocation(new ClassPathResource("unlock.lua"));
        UNLOCK_SCRIPT.setResultType(Long.class);
    }

public void unlock() {
    // 调用lua脚本
    stringRedisTemplate.execute(
            UNLOCK_SCRIPT,
            Collections.singletonList(KEY_PREFIX + name),
            ID_PREFIX + Thread.currentThread().getId());
}
```



总结：

基于Redis的分布式锁实现思路：

* 利用set nx ex获取锁，并设置过期时间，保存线程标示
* 释放锁时先判断线程标示是否与自己一致，一致则删除锁

特性：

* 利用set nx满足互斥性
* 利用set ex保证故障时锁依然能释放，避免死锁，提高安全性
* 利用Redis集群保证高可用和高并发特性



我们一路走来，利用添加过期时间，防止死锁问题的发生，但是有了过期时间之后，可能出现误删别人锁的问题，这个问题我们开始是利用删之前 通过拿锁，比锁，删锁这个逻辑来解决的，也就是删之前判断一下当前这把锁是否是属于自己的，但是现在还有原子性问题，也就是我们没法保证拿锁比锁删锁是一个原子性的动作，最后通过lua表达式来解决这个问题

但是目前还剩下一个问题锁不住，什么是锁不住呢，你想一想，如果当过期时间到了之后，我们可以给他续期一下，比如续个30s，是不是后边的问题都不会发生了，那么续期问题怎么解决呢，可以依赖于我们接下来要学习redission。



**测试逻辑**

第一个线程进来，得到了锁，手动删除锁，模拟锁超时了，其他线程会执行lua来抢锁，当第一个线程利用lua删除锁时，lua能保证他不能删除他的锁，第二个线程删除锁时，利用lua同样可以保证不会删除别人的锁，同时还能保证原子性。



# 5、分布式锁-redission



## 5.1、redission功能介绍

基于setnx实现的分布式锁存在下面的问题：

**重入问题**：重入问题是指 获得锁的线程可以再次进入到相同的锁的代码块中，可重入锁的意义在于防止死锁，比如HashTable这样的代码中，他的方法都是使用synchronized修饰的，假如他在一个方法内，调用另一个方法，那么此时如果是不可重入的，不就死锁了吗？所以可重入锁他的主要意义是防止死锁，我们的synchronized和Lock锁都是可重入的。