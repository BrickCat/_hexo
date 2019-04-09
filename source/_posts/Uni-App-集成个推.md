---
title: Uni-App 集成个推
date: 2019-04-09 17:30:46
tags: [App, Uni-App, 个推, 推送]
categories: [App,推送]
---

### 1、申请UniPush插件使用权限
- 打开`manifest.json`->`App SDK配置`->`推送`->`DCloud Push`->`配置` （需要实名制认证）
- 打开`manifest.json`->`App模块权限配置`->`打包模块配置`->`Push(消息推送)`
- 包名在云打包时可以获取。
### 2、Java后台集成个推
1.添加Jar包
在`pom.xml`中添加如下：
```xml
<!--个推-->
<dependency>
    <groupId>com.gexin.platform</groupId>
    <artifactId>gexin-rp-sdk-http</artifactId>
    <version>4.1.0.1</version>
</dependency>
```
```xml
<repositories>
    <!--个推私服-->
    <repository>
        <id>getui-nexus</id>
        <url>http://mvn.gt.igexin.com/nexus/content/repositories/releases/</url>
    </repository>
</repositories>
```

2. 消息推送工具类

消息格式：`{"title": "xxx","content": "xxx","payload": "xxx"} `

**AppPushUtil.java**
```java
package cn.chinaunicom.sdsi.app.attendance.util;

import cn.chinaunicom.sdsi.app.attendance.config.AppPushConstant;
import com.alibaba.fastjson.JSONObject;
import com.gexin.rp.sdk.base.IPushResult;
import com.gexin.rp.sdk.base.impl.SingleMessage;
import com.gexin.rp.sdk.base.impl.Target;
import com.gexin.rp.sdk.base.payload.APNPayload;
import com.gexin.rp.sdk.http.IGtPush;
import com.gexin.rp.sdk.template.TransmissionTemplate;
import org.apache.commons.lang3.StringUtils;

import java.util.Map;
import java.util.Set;

/**
 * @author 毛子坤
 * @Title: ${file_name}
 * @Package ${package_name}
 * @Description: ${todo}
 * @date 2019/4/915:55
 */
public class AppPushUtil {
    private static String appId = "t0OtmULeTI64zCeJ6Zctp7";
    private static String appKey = "WKD8OZLMTp7osmkjIIBkt8";
    private static String masterSecret = "0S82U6rQ4Q9FwHYPjx5bI7";
    private static String url  = "http://sdk.open.api.igexin.com/apiex.htm";

    public static void doPush(String clientId, String title, String content, JSONObject payload){
        JSONObject jsonObject = new JSONObject();
        jsonObject.put(AppPushConstant.TITLE, title);
        jsonObject.put(AppPushConstant.CONTENT, content);
        jsonObject.put(AppPushConstant.PAYLOAD, payload.toJSONString());

        pushMessage(clientId,jsonObject);
    }
    /**
     * 推送透传消息
     *
     * @param clientId 终端Id
     *          
     */
    private static IPushResult pushMessage(String clientId, JSONObject json) {
        if (StringUtils.isBlank(clientId)) {
            return null;
        }

        IGtPush push = new IGtPush(url, appKey, masterSecret);

        TransmissionTemplate template = getTemplate(json);

        //定义消息推送方式为，单推
        SingleMessage message = new SingleMessage();
        // 设置推送消息的内容
        message.setData(template);
        // 设置推送目标
        Target target = new Target();
        target.setAppId(appId);
        // 设置cid
        target.setClientId(clientId);
        // 获得推送结果
        IPushResult result = push.pushMessageToSingle(message, target);
        /*
         * 1. 失败：{result=sign_error}
         * 2. 成功：{result=ok, taskId=OSS-0212_1b7578259b74972b2bba556bb12a9f9a, status=successed_online}
         * 3. 异常
         */
        return result;
    }

    /**
     * 透传消息的模板
     * @return
     */
    public static TransmissionTemplate getTemplate(JSONObject json) {
        TransmissionTemplate template = new TransmissionTemplate();
        template.setAppId(appId);
        template.setAppkey(appKey);
        // 透传消息设置，1为强制启动应用，客户端接收到消息后就会立即启动应用；2为等待应用启动
        template.setTransmissionType(1);
        System.out.println(json.toJSONString());
        APNPayload payload = new APNPayload();

        String title = json.getString(AppPushConstant.TITLE);
        String body = json.getString(AppPushConstant.CONTENT);
        if (!StringUtils.isEmpty(title) && !StringUtils.isEmpty(body)) {
            payload.setAlertMsg(getDictionaryAlertMsg(title, body));
        }

        // 在已有数字基础上加1显示，设置为-1时，在已有数字上减1显示，设置为数字时，显示指定数字
        payload.setAutoBadge("+1");
        payload.setContentAvailable(1);
        payload.setSound("default");
        payload.setCategory("$由客户端定义");

        Set<Map.Entry<String, Object>> entrySet = json.entrySet();
        for (Map.Entry<String, Object> entry : entrySet) {
            payload.addCustomMsg(entry.getKey(), entry.getValue());
        }

        template.setAPNInfo(payload);

        return template;
    }

    private static APNPayload.DictionaryAlertMsg getDictionaryAlertMsg(
            String title, String body) {
        APNPayload.DictionaryAlertMsg alertMsg = new APNPayload.DictionaryAlertMsg();
        alertMsg.setBody(body);
        alertMsg.setActionLocKey("ActionLockey");
        alertMsg.setLocKey("LocKey");
        alertMsg.addLocArg("loc-args");
        alertMsg.setLaunchImage("launch-image");
        // iOS8.2以上版本支持
        alertMsg.setTitle(title);
        alertMsg.setTitleLocKey("TitleLocKey");
        alertMsg.addTitleLocArg("TitleLocArg");
        return alertMsg;
    }

}
```
消息体属性名统一配置：

**AppPushConstant.java**
```java
public class AppPushConstant {
    public static final String TITLE = "title";
    public static final String CONTENT = "content";
    public static final String PAYLOAD = "payload";
}
```

参数接收体：

**AppPushBody.java**

```java
package cn.chinaunicom.sdsi.app.attendance.bo;

import cn.chinaunicom.sdsi.framework.request.BaseRequest;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import org.hibernate.validator.constraints.Length;

import java.io.Serializable;

/**
 * @author 毛子坤
 * @Title: ${file_name}
 * @Package ${package_name}
 * @Description: ${todo}
 * @date 2019/4/916:25
 */

@ApiModel(value="推送对象")
public class AppPushBody implements Serializable {
    private static final long serialVersionUID = 1L;

    @ApiModelProperty(value = "终端Id，多个[,]号隔开",required = true)
    private String clientIds;

    @Length(message = "标题最多30个字符", max = 30)
    @ApiModelProperty(value = "标题",required = true)
    private String title;

    @Length(message = "内容最多800个字符",max = 800)
    @ApiModelProperty(value = "内容", required = true)
    private String content;

    @ApiModelProperty(value = "参数Id", required = true)
    private String detailId;

    public String getClientIds() {
        return clientIds;
    }

    public void setClientIds(String clientIds) {
        this.clientIds = clientIds;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public String getDetailId() {
        return detailId;
    }

    public void setDetailId(String detailId) {
        this.detailId = detailId;
    }
}

```

swagger接口调用：

**AppPushController.java**

```java
package cn.chinaunicom.sdsi.app.attendance.controller;

import cn.chinaunicom.sdsi.app.attendance.bo.AppPushBody;
import cn.chinaunicom.sdsi.app.attendance.util.AppPushUtil;
import cn.chinaunicom.sdsi.framework.base.BaseController;
import cn.chinaunicom.sdsi.framework.enums.ResponseEnum;
import cn.chinaunicom.sdsi.framework.exception.BusinessException;
import cn.chinaunicom.sdsi.framework.response.BaseResponse;
import cn.chinaunicom.sdsi.system.role.bo.SysRoleQueryRequestBody;
import cn.chinaunicom.sdsi.system.role.vo.SysRolePermissionVO;
import com.alibaba.fastjson.JSONObject;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author 毛子坤
 * @Title: ${file_name}
 * @Package ${package_name}
 * @Description: ${todo}
 * @date 2019/4/916:22
 */
@Api(
        tags = {"推送管理接口"}
)
@RestController
@RequestMapping("/push")
public class AppPushController extends BaseController {
    private static final Logger log = LoggerFactory.getLogger(AppPushController.class);

    @ApiOperation(
            value = "向一个或多个终端发推送",
            notes = "根据clientId向终端发推送"
    )
    @PostMapping("/doPush")
    public BaseResponse<Object> findPermissionsByRole(@RequestBody AppPushBody appPushBody) throws BusinessException {

        String clientIds[] = appPushBody.getClientIds().split(",");

        for (String clientId : clientIds){
            JSONObject playload = new JSONObject();
            playload.put("detailId",appPushBody.getDetailId());
            AppPushUtil.doPush(clientId,appPushBody.getTitle(),appPushBody.getContent(),playload);
        }

        return this.ok(null);
    }
}

```

### 3、swagger调用
```shell
curl -X POST "http://localhost:8081/push/doPush" -H "accept: */*" -H "Content-Type: application/json" -d "{ \"clientIds\": \"ec68aa86daf96d885d15128aa5c0224a\", \"content\": \"11111111\", \"detailId\": \"111111\", \"length\": 0, \"start\": 0, \"title\": \"222222222222\"}"
```

### 4、前端监听推送和点击事件

[官方Demo](https://ask.dcloud.net.cn/file/download/file_name-dW5pLXB1c2guemlw__url-Ly9pbWctY2RuLXFpbml1LmRjbG91ZC5uZXQuY24vdXBsb2Fkcy9hcnRpY2xlLzIwMTkwMzIyL2IxMjQyZmU0M2FhZWFhNTQ0YTJkM2I5OGFiYWZjZjU0)

`App.vue`

```
<script>
	export default {
		onLaunch: function() {
			console.log('App Launch')
			// #ifdef APP-PLUS
			const _self = this;
			const _handlePush = function(message) {
				console.log(message)
				//根据条件跳转
// 				uni.navigateTo({
// 					url: '../fire/detail?detailID='+message.payload.id
// 				});
			};
			
			plus.push.addEventListener('click', function(message) {
				_handlePush(message);
			});
			plus.push.addEventListener('receive', function(message) {
				_handlePush(message)
			});
			// #endif
		},
		onShow: function() {
			console.log('App Show')
			var info = plus.push.getClientInfo();
			console.log( JSON.stringify( info ) );
		},
		onHide: function() {
			console.log('App Hide')
		},
		methods: {
		}
	}
</script>

<style>
	/*每个页面公共css */
</style>

```
