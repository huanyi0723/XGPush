腾讯信鸽 推送
- 1 填写“应用名称”和“应用包名”（必须要跟APP一致），选择“操作系统”和“分类”，最后点击“创建应用”。
应用创建成功后，点击“应用配置”即可看到
包名 com.qq.xgdemo
AccessId 2100101929
AccessKey A45VE55D5WKX
- 2 Androidmanifest.xml，添加以下配置（建议参考下载包的Demo修改），其中YOUR_ACCESS_ID和YOUR_ACCESS_KEY
替换为APP对应的accessId和accessKey,请确保按照要求配置，否则可能导致服务不能正常使用。 
<application
   <!-- APP项目的其它配置... -->

   <!-- 【必须】 信鸽receiver广播接收 -->
    <receiver
        android:name="com.tencent.android.tpush.XGPushReceiver"
        android:process=":xg_service_v2" >
        <intent-filter android:priority="0x7fffffff" >
            <!-- 【必须】 信鸽SDK的内部广播 -->
            <action android:name="com.tencent.android.tpush.action.SDK" />
            <action android:name="com.tencent.android.tpush.action.INTERNAL_PUSH_MESSAGE" />
            <!-- 【必须】 系统广播：开屏和网络切换 -->
            <action android:name="android.intent.action.USER_PRESENT" />
            <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
            
            <!-- 【可选】 一些常用的系统广播，增强信鸽service的复活机会，请根据需要选择。当然，你也可以添加APP自定义的一些广播让启动service -->
            <action android:name="android.bluetooth.adapter.action.STATE_CHANGED" />
            <action android:name="android.intent.action.ACTION_POWER_CONNECTED" />
            <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED" />
        </intent-filter>
    </receiver>

    <!-- 【必须】 (2.30及以上版新增)展示通知的activity -->
    <!-- 【注意】 如果被打开的activity是启动模式为SingleTop，SingleTask或SingleInstance，请根据通知的异常自查列表第8点处理-->
     <activity
         android:name="com.tencent.android.tpush.XGPushActivity"
         android:exported="true" >
         <intent-filter>
            <!-- 若使用AndroidStudio，请设置android:name="android.intent.action"-->
             <action android:name="" />
         </intent-filter>
    </activity>

    <!-- 【必须】 信鸽service -->
    <service
        android:name="com.tencent.android.tpush.service.XGPushService"
        android:exported="true"
        android:persistent="true"
        android:process=":xg_service_v2" />
 
    <!-- 【必须】 通知service，此选项有助于提高抵达率 -->
    <service
        android:name="com.tencent.android.tpush.rpc.XGRemoteService"
        android:exported="true" >
        <intent-filter>
            <action android:name="应用包名.PUSH_ACTION" />
        </intent-filter>
    </service>

    <!-- 【必须】 请将YOUR_ACCESS_ID修改为APP的AccessId，“21”开头的10位数字，中间没空格 -->
    <meta-data 
        android:name="XG_V2_ACCESS_ID"
        android:value="YOUR_ACCESS_ID" />
    <!-- 【必须】 请将YOUR_ACCESS_KEY修改为APP的AccessKey，“A”开头的12位字符串，中间没空格 -->
    <meta-data 
        android:name="XG_V2_ACCESS_KEY" 
        android:value="YOUR_ACCESS_KEY" />
</application>

<!-- 【必须】 信鸽SDK所需权限 -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<uses-permission android:name="android.permission.RESTART_PACKAGES" />
<uses-permission android:name="android.permission.BROADCAST_STICKY" />
<uses-permission android:name="android.permission.WRITE_SETTINGS" />
<uses-permission android:name="android.permission.RECEIVE_USER_PRESENT" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.KILL_BACKGROUND_PROCESSES" />
<uses-permission android:name="android.permission.GET_TASKS" />
<uses-permission android:name="android.permission.READ_LOGS" />
<uses-permission android:name="android.permission.VIBRATE" />
<!-- 【可选】 信鸽SDK所需权限 -->
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BATTERY_STATS" />

- 3 打开工程的主Activity，在其onCreate(Bundle savedInstanceState)重载方法内，添加以下代码，完成信鸽服务的启动与APP注册过程。 

// 开启logcat输出，方便debug，发布时请关闭
// XGPushConfig.enableDebug(this, true);
// 如果需要知道注册是否成功，请使用registerPush(getApplicationContext(), XGIOperateCallback)带callback版本
// 如果需要绑定账号，请使用registerPush(getApplicationContext(),account)版本
// 具体可参考详细的开发指南
// 传递的参数为ApplicationContext
Context context = getApplicationContext();
XGPushManager.registerPush(context);	

// 2.36（不包括）之前的版本需要调用以下2行代码
Intent service = new Intent(context, XGPushService.class);
context.startService(service);


// 其它常用的API：
// 绑定账号（别名）注册：registerPush(context,account)或registerPush(context,account, XGIOperateCallback)，其中account为APP账号，可以为任意字符串（qq、openid或任意第三方），业务方一定要注意终端与后台保持一致。
// 取消绑定账号（别名）：registerPush(context,"*")，即account="*"为取消绑定，解绑后，该针对该账号的推送将失效
// 反注册（不再接收消息）：unregisterPush(context)
// 设置标签：setTag(context, tagName)
// 删除标签：deleteTag(context, tagName)

4 如果在logcat中的TPush标签看到以下类似的输出，说明已经注册成功，并返回token。
04-27 13:50:29.922: W/TPush(8523): +++ register push sucess. token:f6e890d82afbddde48538ed574fe4efc9306da0a

5 服务端创建推送
