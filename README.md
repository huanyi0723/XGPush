��Ѷ�Ÿ� ����
- 1 ��д��Ӧ�����ơ��͡�Ӧ�ð�����������Ҫ��APPһ�£���ѡ�񡰲���ϵͳ���͡����ࡱ�������������Ӧ�á���
Ӧ�ô����ɹ��󣬵����Ӧ�����á����ɿ���
���� com.qq.xgdemo
AccessId 2100101929
AccessKey A45VE55D5WKX
- 2 Androidmanifest.xml������������ã�����ο����ذ���Demo�޸ģ�������YOUR_ACCESS_ID��YOUR_ACCESS_KEY
�滻ΪAPP��Ӧ��accessId��accessKey,��ȷ������Ҫ�����ã�������ܵ��·���������ʹ�á� 
<application
   <!-- APP��Ŀ����������... -->

   <!-- �����롿 �Ÿ�receiver�㲥���� -->
    <receiver
        android:name="com.tencent.android.tpush.XGPushReceiver"
        android:process=":xg_service_v2" >
        <intent-filter android:priority="0x7fffffff" >
            <!-- �����롿 �Ÿ�SDK���ڲ��㲥 -->
            <action android:name="com.tencent.android.tpush.action.SDK" />
            <action android:name="com.tencent.android.tpush.action.INTERNAL_PUSH_MESSAGE" />
            <!-- �����롿 ϵͳ�㲥�������������л� -->
            <action android:name="android.intent.action.USER_PRESENT" />
            <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
            
            <!-- ����ѡ�� һЩ���õ�ϵͳ�㲥����ǿ�Ÿ�service�ĸ�����ᣬ�������Ҫѡ�񡣵�Ȼ����Ҳ�������APP�Զ����һЩ�㲥������service -->
            <action android:name="android.bluetooth.adapter.action.STATE_CHANGED" />
            <action android:name="android.intent.action.ACTION_POWER_CONNECTED" />
            <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED" />
        </intent-filter>
    </receiver>

    <!-- �����롿 (2.30�����ϰ�����)չʾ֪ͨ��activity -->
    <!-- ��ע�⡿ ������򿪵�activity������ģʽΪSingleTop��SingleTask��SingleInstance�������֪ͨ���쳣�Բ��б��8�㴦��-->
     <activity
         android:name="com.tencent.android.tpush.XGPushActivity"
         android:exported="true" >
         <intent-filter>
            <!-- ��ʹ��AndroidStudio��������android:name="android.intent.action"-->
             <action android:name="" />
         </intent-filter>
    </activity>

    <!-- �����롿 �Ÿ�service -->
    <service
        android:name="com.tencent.android.tpush.service.XGPushService"
        android:exported="true"
        android:persistent="true"
        android:process=":xg_service_v2" />
 
    <!-- �����롿 ֪ͨservice����ѡ����������ߵִ��� -->
    <service
        android:name="com.tencent.android.tpush.rpc.XGRemoteService"
        android:exported="true" >
        <intent-filter>
            <action android:name="Ӧ�ð���.PUSH_ACTION" />
        </intent-filter>
    </service>

    <!-- �����롿 �뽫YOUR_ACCESS_ID�޸�ΪAPP��AccessId����21����ͷ��10λ���֣��м�û�ո� -->
    <meta-data 
        android:name="XG_V2_ACCESS_ID"
        android:value="YOUR_ACCESS_ID" />
    <!-- �����롿 �뽫YOUR_ACCESS_KEY�޸�ΪAPP��AccessKey����A����ͷ��12λ�ַ������м�û�ո� -->
    <meta-data 
        android:name="XG_V2_ACCESS_KEY" 
        android:value="YOUR_ACCESS_KEY" />
</application>

<!-- �����롿 �Ÿ�SDK����Ȩ�� -->
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
<!-- ����ѡ�� �Ÿ�SDK����Ȩ�� -->
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BATTERY_STATS" />

- 3 �򿪹��̵���Activity������onCreate(Bundle savedInstanceState)���ط����ڣ�������´��룬����Ÿ�����������APPע����̡� 

// ����logcat���������debug������ʱ��ر�
// XGPushConfig.enableDebug(this, true);
// �����Ҫ֪��ע���Ƿ�ɹ�����ʹ��registerPush(getApplicationContext(), XGIOperateCallback)��callback�汾
// �����Ҫ���˺ţ���ʹ��registerPush(getApplicationContext(),account)�汾
// ����ɲο���ϸ�Ŀ���ָ��
// ���ݵĲ���ΪApplicationContext
Context context = getApplicationContext();
XGPushManager.registerPush(context);	

// 2.36����������֮ǰ�İ汾��Ҫ��������2�д���
Intent service = new Intent(context, XGPushService.class);
context.startService(service);


// �������õ�API��
// ���˺ţ�������ע�᣺registerPush(context,account)��registerPush(context,account, XGIOperateCallback)������accountΪAPP�˺ţ�����Ϊ�����ַ�����qq��openid���������������ҵ��һ��Ҫע���ն����̨����һ�¡�
// ȡ�����˺ţ���������registerPush(context,"*")����account="*"Ϊȡ���󶨣����󣬸���Ը��˺ŵ����ͽ�ʧЧ
// ��ע�ᣨ���ٽ�����Ϣ����unregisterPush(context)
// ���ñ�ǩ��setTag(context, tagName)
// ɾ����ǩ��deleteTag(context, tagName)

4 �����logcat�е�TPush��ǩ�����������Ƶ������˵���Ѿ�ע��ɹ���������token��
04-27 13:50:29.922: W/TPush(8523): +++ register push sucess. token:f6e890d82afbddde48538ed574fe4efc9306da0a

5 ����˴�������
