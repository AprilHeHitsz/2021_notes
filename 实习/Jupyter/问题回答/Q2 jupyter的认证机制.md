# jupyter的认证机制

## PAM方式

与操作系统用户层共享同一认证管理，user和对应的password都与其相同。
默认采用这种方式进行认证。在login时，如果没有自定义Login_handler的话，就是使用这种形式。直接将配置好的username传上去。（？这里还不是太清楚，具体看代码）

![login1](D:\2020_Sources\2020秋\实习\0210\login1.png)

 

![auth](D:\2020_Sources\2020秋\实习\0210\auth.png)

LocalAuthenticator 是一种特殊的身份验证器，它能够管理本地系统上的用户。当您尝试向hub添加新用户时，LocalAuthenticator 将检查该用户是否已经存在。如果在配置文件中将配置值 create _ system _ users 设置为 True，则 LocalAuthenticator 拥有将用户添加到系统的权限。

![local](D:\2020_Sources\2020秋\实习\0210\local.png)

![pam1](D:\2020_Sources\2020秋\实习\0210\pam1.png)

![pam2](D:\2020_Sources\2020秋\实习\0210\pam2.png)

## OAuth方式

需要重写login handler。
依赖于令牌（token）而不是用户名和密码，支持 Auth0, Azure AD, Bitbucket, CILogon, GitHub, GitLab, Globus, Google, MediaWiki, Okpy, OpenShift。

如果重写了，将会使用config里面指定的loginhandler进行进一步认证。

![login2](D:\2020_Sources\2020秋\实习\0210\login2.png)

## 自定义
使用自定义的Authenticator子类，就可以通过其他方式启用身份验证。



# 认证（JupyterHub）

## cookie和cookie secrets（用于加密cookie）

(app.init_secrets())在初始化时加载cookie_secret，可从config、env、file中加载。如果三个里面都没有，那新建一个secret_cookie,然后存储到file里面。

*还需要看一下 这三个来源在哪里*

设置cookie：在LoginHandler里面，当用户登录了的时候（self.current_user!=None），使用set_login_cookie方法设置user的cookie。如下。

（这里的用户登录是指通过http request里的token或cookie通过身份验证。）

![cookie1](D:\2020_Sources\2020秋\实习\0210\cookie1.png)

![cookie2](D:\2020_Sources\2020秋\实习\0210\cookie2.png)

问题：这三个cookie分别在验证什么的时候会用到？





## Oauth

### 基本原理

JupyterHubAuthServer()的作用是，登陆的时候，将会重定向到Server上，并附上client的ID和URI。

然后，Authorization Server要向用户确认是否给client授权。方法是让用户向Authorization Server提供**用户名和密码**。

接着，Authorization Server收到用户的授权后，会重定向到client的URL，并附带**Authorization code**。

之后，client使用Authorization code和URI向Authorization Server请求**token**。

最后，Authorization Server验证Authorization code和URI之后，向client发送token。

### 具体实现（源码）

- init_oath()：make_privider函数实例化一个JupyterHubRequestValidator(),提供给JupyterHubOAuthServer()

  ![init_oauth1](D:\2020_Sources\2020秋\实习\0210\init_oauth1.png)

  ![init_oauth2](D:\2020_Sources\2020秋\实习\0210\init_oauth2.png)

- JupyterHubRequestValidator():继承于RequestValidator(),实现了如下方法：

  - `authenticate_client`(*request*, args*,** *kwargs*)：根据request.client_id到数据库里寻找对应的oath_client，比较oath_client和client_id，返回对应的true/false。

  - `authenticate_client_id`(*client_id*, *request*,*args,** *kwargs*)：将request.client 设置为与给定 client _ id 相关联的客户机对象。

  - `confirm_redirect_uri`(*client_id*, *code*, *redirect_uri*, *client*, *request*, args*, *kwargs*)：确保这个 authorization code表示的授权过程以这个“ redirect _ uri”开始。

  - `get_default_redirect_uri`(*client_id*, *request*, args*, *kwargs*)：获取client的默认重定向 URI。

  - `get_default_scopes`(*client_id*, *request*, args*, *kwargs*)：获取客户端的默认scopes。

  - ...

    ![init_oauth3](D:\2020_Sources\2020秋\实习\0210\init_oauth3.png)

  总之是一些与授权代码（authorization code）、访问权限的刷新、保存令牌等有关的一些方法。

  ![init_oauth4](D:\2020_Sources\2020秋\实习\0210\init_oauth4.png)

- JupyterHubOAuthServer()继承于WebApplication,实现了如下方法：

  - add_client(self,client_id,client_secret, redirect_uri, descrption):向数据库中添加一个client。
  - fetch_by_client_id(self,client):根据client_id获取client。

![init_oauth5](D:\2020_Sources\2020秋\实习\0210\init_oauth5.png)

![init_oauth6](D:\2020_Sources\2020秋\实习\0210\init_oauth6.png)

- init_users()
  - 首先将从config里面获取的的admin_user添加到数据库中。
  - 然后将白名单里面的所有user规范化后添加到数据库里面。
  - 最后将数据库中存在的所有用户通知给authenticator。
  
  
  
  ![init_user1](D:\2020_Sources\2020秋\实习\0210\init_user1.png)
  
  ![init_user2](D:\2020_Sources\2020秋\实习\0210\init_user2.png)
  
  ![init_user3](D:\2020_Sources\2020秋\实习\0210\init_user3.png)
  
  ![init_user4](D:\2020_Sources\2020秋\实习\0210\init_user4.png)
  
- init_groups()
  
  - 将预先定义的组和用户添加到数据库中。
  
  ![init_group](D:\2020_Sources\2020秋\实习\0210\init_group.png)



# jupyterhub 实现 用户身份验证

## 基础知识

### 认证和授权的区别：

- Authentication：你是谁，关于ID、密码等
- Authorization：你能做什么，关于admin等

### Cookie

- 保存在客户端，存放的是关于用户的信息

- 应用案例

  - 页面1和页面2是同一个域名，但是只在页面1登录了。希望在进入页面2的时候服务器能直接知道我是哪一个用户，而不需要重新登录。于是可以在这个域名下设置一个cookie，当用户在页面1进行了登录之后，服务器就会为该用户在这个域名下设置一个cookie，保存到客户端，于是用户在访问页面2的时候，会把cookie也发送过去，客户端由cookie可以知道这是那个用户，就不用再进行登录了。
  - 保存已经登陆过的用户信息，可以自动填写一些基本信息
  - 保存session或者token，用来发送给服务器，以便于让后端知道用户当前的状态（HTTP是无状态的）
  - 用来记录、分析用户行为，比如在某个页面查看过哪些内容，以便于个性化推荐

- Cookie的设置和使用

  

### Session

- 保存在服务器端，客户端存储的只有sessionID，写在cookie文件里。session认证就是把user的信息存储到session里面。
- 应用案例







### Token

- 令牌，最简单的就是，uid(用户唯一的身份标识)、time(当前时间的时间戳)、sign(签名，由token的前几位+以哈希算法压缩成一定长的十六进制字符串，可以防止恶意第三方拼接token请求服务器)。如果是OAuth Token这种机制，就是提供认证（让服务器知道你是谁）和授权（使App拥有访问某用户信息的权利）。

- 应用案例

  - 当用户首次登录成功（注册也是一种可以适用的场景）之后, 服务器端就会**生成一个 token 值**，这个值，会在**服务器保存token**值(保存在数据库中)，再将这个token值**返回给客户端**.

    客户端拿到 token 值之后,进行**本地保存**。（SP存储是大家能够比较支持和易于理解操作的存储）

    当客户端再次发送网络请求(一般不是登录请求)的时候,就会将这个 token 值**附带到参数**中发送给服务器.

    服务器接收到客户端的请求之后,会取出token值与保存在本地(数据库)中的token值做**对比**: 如果两个 token 值相同， 说明用户登录成功过,当前用户处于登录状态.如果没有这个 token 值, 则说明没有登录成功.如果 token 值不同: 说明原来的登录信息已经失效,让用户重新登录.

  

  

  

### Cookie验证和Token验证的区别

- Cookie验证：利用cookie存储sessionID进行验证，或者利用cookie存储Token进行验证。

  - 有状态，验证记录或者会话需要一直在server（session）或client（cookie）端保持。server需要保持对数据库活动会话的追踪。也就不便于应用的扩展。

- Token验证：Token存储在Authorization Header中进行验证。

  - 无状态，服务器端不需要保留用户的认证信息或者会话信息。基于token认证机制的应用不需要考虑用户在哪一台服务器登录了，有利于应用的扩展。

    每个请求都带上server要求验证的token，token放在Authorization header中发送过去。也可以在其他字段发送，例如post body，query parameter。

    server端不需要保持对token的记录，它只需要在登录成功后生成或者sign token，token发送过来之后对其进行验证即可。如果使用了第三方服务，则是由第三方进行token的签发，而server只需要验证token的有效性即可。

    可以把token放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP Header 的 Authorization字段中：` Authorization: Bearer Token`。

  ![image-20210224182155739](C:\Users\hyp19\AppData\Roaming\Typora\typora-user-images\image-20210224182155739.png)

  （来自csdn，需要进一步验证）

## 具体流程

### 代码解析

![2-get_current_user()](D:\2020_Sources\2020秋\实习\截图\2-27\2-27\2-get_current_user().png)

- **self.get_current_user_token**:(首先使用Token验证方式，查看是否在请求中含有一个JWT(?大概是通过这个东西？)，获取Authorization中的Token, 然后在数据库中寻找是否存在对应的正确的token。再根据orm_token.user到数据库中寻找user。验证成功则返回用户，否则（数据库中找不到正确的token）返回none。（还需要详细看一下，可能不准确。）

  ![2-get_current_user_token()](D:\2020_Sources\2020秋\实习\截图\2-27\2-27\2-get_current_user_token().png)

- **self.get_current_user_cookie**:使用self.hub.cookie_name()获取cookie_name,然后再数据库里寻找相应的用户。如果成功则返回用户，否则返回None。

  ![2-get_current_user_cookie()](D:\2020_Sources\2020秋\实习\截图\2-27\2-27\2-get_current_user_cookie().png)	



- ## ![2-_user_for_cookie](D:\2020_Sources\2020秋\实习\截图\2-27\2-27\2-_user_for_cookie.png)

  ## 登录以及后续处理流程：

  当访问/hub/时，proxy将·会把请求转发给hub处理，hub首先会检查用户是否登录，如果没有登录，则先重定向到登录页面，登录成功之后，若配置中redirect_to_server为真，则重定向到/hub/spawn/页面，此时就由spawner来处理这个流程。

