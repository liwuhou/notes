# Login 模块
该类是一个用于在小程序环境下的登录模块，用于管理用户登录状态和获取用户信息等功能。

### Usage

```js
import Login, { isQyEnv } from 'libs/login/index'

;(async () => {
  const isLoginSuccess = await Login.login() // boolean
})()

Page({
  data: { 
    userInfo: null, 
    isQy: isQyEnv(), // 是否企微环境，也可通过 Login.isQy 获取
  },
  async onLoad(option) {
    // 可以直接获取用户信息，不用管是否已经登录，模块内会做好处理
    const userInfo = await Login.getUserInfo()
    this.setData({ userInfo })

    // #### 获取 KoToken ####
    try {
      const res = await Login.askForKoToken({
        appId, // 不传店铺 appId 的话，模块也会尝试获取当前的店铺 appId
        isNeedRegister: true, // 新用户的时候，是否需要跳转到 h5 静默注册，默认为 true
        option, // 当 isNeedRegister 为 true 时，必填，否则方法直接抛错！
        maxRetry: 5, // 由于跳转注册不一定会成功，这里能限制尝试跳转注册的次数， 默认为 5
        onBeforeRegister: async () => {
          // 跳转静默注册前的回调方法，  方法执行完后会立即跳转，需要注意的是届时调用侧的页面的后续逻辑会被中断。
          wx.showToast({title: '啊喔，你是尊贵的新用户'})
          //  可通过 webStorage 保存一些上下文或是上报
          await api.reportError(/** balabala */)
        }
      }) 
      // 有默认参数，其实也可以直接就这样
      const { code, data: koToken, message } = await Login.askForKoToken({ option })

      //  这里只管岁月静好地使用返回的 koToken
      if (code === 0) {
        this.setData({ koToken })
      } else {
        wx.showToast({ title: message, icon: 'fail' })
      }
    } catch(e) {
      /**
       * 走到这里几种可能性
       * 1. appId 传值有错误，抛锚警告开发者正确传入
       * 2. isNeedRegister 传了 true,  但 option 没有传，抛错警告开发者传入
       */
      // FIXME
    }
  }
})
```

###  核心 Api

**login** 
登录方法，具有一定重试策略和一个并发缓存队列，集成了异常上报，业务调用可以放心使用。

无入参，接受 `Pomise` 化调用
返回参数：Boolean

**getLoginData()**
获取登录凭证信息，未登录时会自动登录。

无入参，接受 `Pomise` 化调用

返回参数: Object object

| 参数名     | 类型   | 说明            |
| ---------- | ------ | --------------- |
| `union_id` | String | 用户 `union_id` |
| `token`    | String | 用户登录凭证    | 

**getUserInfo()**
获取当前用户信息，当未登录前调用时，会尝试进行登录并一并返回用户信息

无参数，接受 `Promise` 化调用

返回参数: Object object

| 参数名               | 所属平台         | 类型   | 说明                                                |
| -------------------- | ---------------- | ------ | --------------------------------------------------- |
| `nickname`           | `wechat`,`wecom` | String | 用户呢称                                            |
| `headimgurl`         | `wechat`,`wecom` | String | 用户头像链接                                        |
| `app_id`             | `wechat`,`wecom` | String | 店铺 id                                             |
| `self_app_id`        | `wechat`,`wecom` | String |                                                     |
| `universal_union_id` | `wechat`,`wecom` | String | 用户 union id，`注: 在企微中，这个值其实是 open id` |
| `universal_open_id`  | `wechat`         | String | 用户 open id                                        |
| `is_bind_pohone`     | `wechat`         | String | 用户是否绑定手机号                                  |
| `phone_number`       | `wechat`         | String | 用户绑定了的手机号                                  |
| `is_verified`        | `wechat`         | Number | 是否实名认证                                        |
| `corp_id`            | `wecom`          | String | 企业 ID                                             |
| `corp_name`          | `wecom`          | String | 企业名称                                            |

**askForKoToken()**
获取店铺 koToken，具有一定容灾能力，对新用户会进行跳转注册，使用方式可参考 `Usage` 中的 `获取 koToken 例子`。


 入参：Object object
| 参数名             | 类型    | 必填 | 说明                                                                             |
| ------------------ | ------- | ---- | -------------------------------------------------------------------------------- |
| `appId`            | String  | 是   | 当前获取 `koToken` 的店铺 ai                                                     |
| `isNeedRegister`   | String  | 否   | 当前用户为店铺新用户时，是否需求跳转到 h5 注册，默认为 `true`                    |
| `option`           | Object  | 否   | 当前页面的 `query` 参数对象，当 `isNeedRegister` 为 `true` 时必填                |
| `onBeforeRegister` | Boolean | 否   | 用户跳转 h5 注册前调用，需要注意的是，注册可能会失败，该函数可能会被重复执行数次 |
| `maxRetry`         | Number  | 否   | 因为跳转 h5 注册有可能会失败，这个参数会限制重试的次数                           |

成功更新或者注册成功会返回 包含 `{data: koToken}`的对象数据，服务请求失败或者超过重试限制会返回错误状态，调用层需自行判断错误并做处理。

返回参数：Object object

| 参数名    | 类型           | 说明      |
| --------- | -------------- | --------- |
| `code`    | Number         | 状态码    |
| `data`    | String or Null | `koToken` |
| `message` | String         | 异常信息  |

 `code` 状态值列表

| `code` | 说明                                             |
| ------ | ------------------------------------------------ |
| 0      | 正常，在 `data`中返回 `koToken`                  |
| -1     | 请求网络服务出现异常, `message` 返回错误原因描述 |

**getCurrentAppId()**
获取当前店铺的店铺 id

 返回参数：String

**isQy**

类型：Boolean
当前环境是否为企微环境

  
###  模块成员方法

**getDefaultAppId()**
获取当前店铺的 appId。

返回参数：String