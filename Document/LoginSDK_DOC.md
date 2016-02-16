### **修改记录**
---
| 版本号 | 时间 | 修改人 | 修改内容 |
| :--: | :--: | :--: | :--: |
| 1.2.0.0 | 2016-01-06 |   Y.J.Zhou   | 发布新登录sdk版本 |
| 1.2.0.1 | 2016-01-15 |   Y.J.Zhou   | 为配置增加redirect_url接口 |

<p>
<p>

###**目录**
---
&nbsp;&nbsp;&nbsp;&nbsp;[1、导入库与资源](#导入库与资源)  
&nbsp;&nbsp;&nbsp;&nbsp;[2、接入登录SDK](#接入登录SDK)  
&nbsp;&nbsp;&nbsp;&nbsp;[3、接入服务端接口](#接入服务端接口)  
&nbsp;&nbsp;[附录一、登录SDK授权步骤](#附录一、登录SDK授权步骤)  
&nbsp;&nbsp;[附录二、登录接口结果码对照表](#附录二、登录接口结果码对照表)  
&nbsp;&nbsp;[附录三、自定义SDK界面风格](#附录三、自定义SDK界面风格)  


###**导入库与资源**
---

1. 将 \res\下的文件复制到工程相应的资源目录中去。
2. 将 m4399LoginSDK.jar 文件复制到工程的 \libs 目录下
3. 在工程AndroidManifest.xml 中注册 OpeHostActivity ：
```xml
< activity
   android :name= "cn.m4399.operate.controller.OpeHostActivity"
   android :configChanges= "orientation|screenSize|keyboardHidden"
   android :launchMode= "singleTask" />
```
4. 在工程AndroidManifest.xml 中加入权限
```xml
< uses-permission android :name= "android.permission.INTERNET" />
< uses-permission android :name= "android.permission.READ_PHONE_STATE" />
< uses-permission android :name= "android.permission.ACCESS_NETWORK_STATE" />
```

###**接入登录SDK**
---

#### **1.  初始化**
建议在 **Appliction** 类的 onCreate 方法中进行初始化。


```java
public class AppDemo extends Application{

    final String AUTH_CALLBACK = "http://ptlogin.3304399.net/resource/images/ptlogin_mask.png";
    @Override
    public void onCreate() {   
        super.onCreate();
        // 获取sdk操作类
        OperateCenter opeCenter = OperateCenter.getInstance();
        // sdk配置信息类
        OperateConfig opeConfig = new OperateConfig.Builder()      
                // 登陆界面横竖屏配置（ 当使用游戏盒授权时，登陆强制为竖屏 ）（ 必填 ）
                .setOrientation(SCREEN_ORIENTATION) 
                // app在用户中心分配的 client_id （ 必填 ）第一次接入的 APP 可自行向 4399用户中心（厦门）申请
                .setClientID("testNet") 
                // app在用户中心分配的 client_id 所对应的 redirect_url
                .setRedirectUrl(AUTH_CALLBACK) 
                // app在游戏盒分配的 id （非必填 默认自动填充为clientid ）
                .setGameID("testNet")
                // 是否全屏显示登录界面 （ 选填 ）
                .setFullScreen(false)
                .build();
                
        // 二选一 进行SDK初始化
        // 进行sdk初始化（接口一）
        mOpeCenter.init(getApplicationContext(), opeConfig);
                
        // 进行sdk初始化（接口二）（有效防止进程被杀而导致的游戏盒无法授权登录 推荐使用）
        mOpeCenter.init(getApplicationContext(), opeConfig,  new OperateCenter.ValidateListener(){
               @Override
               public void onValidateFinished(SDKResult result) {
                    Toast.makeText(getApplicationContext(), result.toString(), Toast.LENGTH_LONG).show();
                }
        });
    }
}

```

```java
class SDKResult {
        // 登录接口结果码，参照（附录二）来确定是哪种登录方式
        public void getResultCode();
        // 登录接口结果
        public void getResultMsg();
        // Web 授权登录模式 AuthCode
        public void getAuthCode();
        // 游戏盒授权登录模式 refresh_token
        public void getRefreshToken();
        // 游戏盒授权登录模式 uid
        public String geUID();
}
```
#### **2.  注册**
```java
// 注册接口一经调用，无论原先是否已经登录一律清除原有登录信息，重新进行注册并且登录步骤。
mOpeCenter.register(this, opeConfig, new OperateCenter.ValidateListener(){
                @Override
                public void onValidateFinished(SDKResult result) {
                    Toast.makeText(getApplicationContext(), result.toString(), Toast.LENGTH_LONG).show();
                }
        });
```
#### **3.  登录**
```java
// 注册接口一经调用，无论原先是否已经登录一律清除原有登录信息，重新进行注册并且登录步骤。
mOpeCenter.login(this, opeConfig, new OperateCenter.ValidateListener(){
                @Override
                public void onValidateFinished(SDKResult result) {
                    Toast.makeText(getApplicationContext(), result.toString(), Toast.LENGTH_LONG).show();
                }
        });
```

#### **4. 记录网页登录历史用户名**

```java
        mOpeCenter.recordAccountName(String accountName);
```

#### **5. 调用结果回调**

```java
    public interface ValidateListener {
        /**
         * @param result   用户验证结果
         */
        public void onValidateFinished(SDKResult result);
    }
```

<p>

###**接入服务端接口**
---

####**1. 服务端接口与参数**
<p>
web授权登录
> **POST** https://ptlogin.4399.com/oauth2/token.do
<p>

>| 参数名 | 内容 |
| :--: | :--: |
| grant_type=AUTHORIZATION_CODE | （固定字段） |
| client_id | 用户中心分配的 client_id |
| redirect_url | 用户中心分配的 redirect_url |
| client_secret | 用户中心分配的 secret |
| code | Web登录获取到的 **AuthCode** |

<p>
游戏盒授权登录
> **POST** https://ptlogin.4399.com/oauth2/token.do
<p>

>| 参数名 | 内容 |
| :--: | :--: |
| grant_type=REFRESH_TOKEN | （固定字段） |
| client_id| 用户中心分配的 client_id |
| client_secret| 用户中心分配的 secret |
| UID | SDK 返回的 UID |
| Refresh_token | 游戏盒授权登录获取到的 **Refresh_token** |

####**2.  服务端成功响应**
<p>

> | 参数名 | 内容 |
| :--: | :--: |
| ACCESS_TOKEN | ACCESS_TOKEN |
| EXPIRED_AT | accesstoken过期时间点|
| EXPIRES_IN | accesstoken有效期|
| REFRESH_TOKEN| REFRESH_TOKEN|
| SCOPE | 权限范围，现在都是basic|
| UID| 用户身份识别码 |
| USERNAME| 用户名 |
| DISPLAY_NAME | 用于显示的名字|
| EXT_NICK | 第三方账号昵称 |
| BOUND_PHONE | 绑定手机号 |
| ACCOUNT_TYPE | 账号类型，4399，qq，weixin，weibo |
| EXT_TOKEN_EXPIRED_AT | 第三方账号token过期时间点 |

####**3.  服务端失败响应**
<p>
```json
{"error" : "invalid_request" , "error_description" : "Code unauthorized"}
```
<p>

###**附录一、登录SDK授权步骤**
---

<p>
<img src="/Resource/loginsdk_architecture.png" alt="登录 SDK 授权步骤" />
<p>
<p>

###**附录二、登录接口结果码对照表**
---

#### 登录授权成功
>| Result_code | Result_msg |
| :--: | :--: |
| 0x000| Web 登录返回 **AuthCode** 码 |
| 0x001| Web 注册返回 **AuthCode** 码 |
| 0x002| 游戏盒授权返回 **Refresh_Code** 与 **UID** |

#### 登录授权失败
>| Result_code | Result_msg |
| :--: | :--: |
| 0x101| 网页登录网络异常 |
| 0x102| 网页注册网络异常 |
| 0x103| 游戏盒授权无结果 |
| 0x104| 用户取消登录 |

###**附录三、自定义SDK界面风格**
---

<img src="/Resource/title_change_guider.png" alt="登录SDK的头部" />

登录SDK的头部一共有三张 9-patch 背景图片：
图**红色**部分背景图为：
>导航背景图：m4399loginsdk_9patch_title_bg.9.png

图**蓝色**部分按钮背景图：
> 按钮正常态：m4399loginsdk_9patch_title_btn_normal.9.png

> 按钮选定态：m4399loginsdk_9patch_title_btn_active.9.png

接入时可以对这些图片进行替换以达到更改背景的目的。
