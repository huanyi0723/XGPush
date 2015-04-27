腾讯信鸽 推送
//开始界面
	if(UserState.getInstance().isLoggedOn()){
							String temp = UserState.getInstance().getUser().account_id + "";
							if(temp.length()<=1){
								temp = "0"+temp;
							}
							XGUtils.getInstances(getActivity()).registerXG(temp);
						}else{
							XGUtils.getInstances(getActivity()).registerXG();
						}

//推送工具类
public class XGUtils {
	
	private static final int REGISTER_XG_FAILED = 1 << 3;
	
	private static final int UNREGISTER_XG_FAILED = 1 << 4;
	
	private static XGUtils sXgUtils;
	
	private Context mContext;
	
	
	private XGUtils(Context context){
		Log.d("blocker:l99", "XGUtil init...");
		this.mContext = context.getApplicationContext();
		XGPushConfig.enableDebug(this.mContext, true);
		enhancePushService();
	}
	
	public static XGUtils getInstances(Context context) {
		if (sXgUtils == null) {
			synchronized (XGUtils.class) {
				if (sXgUtils == null) {
					sXgUtils = new XGUtils(context);
				}
			}
		}
		return sXgUtils;
	}
	
	private int tryCount = 0;
	
	private Handler xgHandler = new Handler(new Handler.Callback() {
		
		@Override
		public boolean handleMessage(Message msg) {
			Log.d("blocker", "xgHandler#handleMessage...");
			switch (msg.what) {
			case REGISTER_XG_FAILED:
				String accountId = (String) msg.obj;
				
				if (tryCount < 5) {
					if(accountId.equals("-1")){
						registerXG();
					}else{
						registerXG(accountId);
					}
				}
				
				break;
			case UNREGISTER_XG_FAILED:
				unRegisterXG();
				break;
			default:
				break;
			}
			return true;
		}
	});
	
	
	public void registerXG(final String accountId) {
		XGPushManager.registerPush(mContext, accountId, new XGIOperateCallback() {
			
			@Override
			public void onSuccess(Object data, int flag) {
				Log.d("blocker", "register push success...");
				Context context = mContext.getApplicationContext();
				Intent service = new Intent(context, XGPushService.class);
				context.startService(service);
				//设置标签
				XGTagUtil.setTag(mContext);
			}
			
			@Override
			public void onFail(Object data, int errCode, String msg) {
				Log.d("blocker", "register push failed...  errorCode: " + errCode + "\n"
						+ "msg: " + msg);
				tryCount++;
				Message message = Message.obtain();
				message.obj = accountId;
				message.what = REGISTER_XG_FAILED;
				xgHandler.sendMessage(message);
			}
		});
	}
	
	public void registerXG(){
		XGPushManager.registerPush(mContext, new XGIOperateCallback() {
			
			@Override
			public void onSuccess(Object data, int flag) {
				Log.d("blocker", "register push success...");
				Context context = mContext.getApplicationContext();
				Intent service = new Intent(context, XGPushService.class);
				context.startService(service);
				//设置标签
				XGTagUtil.setTag(mContext);
			}
			
			@Override
			public void onFail(Object data, int errCode, String msg) {
				Log.d("blocker", "register push failed...  errorCode: " + errCode + "\n"
						+ "msg: " + msg);
				tryCount++;
				Message message = Message.obtain();
				message.obj = "-1";
				message.what = REGISTER_XG_FAILED;
				xgHandler.sendMessage(message);
			}
		});
	}
	
	public void unRegisterXG() {
		tryCount = 0;
		XGPushManager.unregisterPush(mContext, new XGIOperateCallback() {
			
			@Override
			public void onSuccess(Object data, int flag) {
				Log.d("blocker", "unRegisterXG#onSuccess...");
				if(UserState.getInstance().isLoggedOn()){
					String temp = UserState.getInstance().getUser().account_id + "";
					if(temp.length()<=1){
						temp = "0"+temp;
					}
					XGUtils.getInstances(mContext).registerXG(temp);
				}else{
					XGUtils.getInstances(mContext).registerXG();
				}
			}
			
			@Override
			public void onFail(Object data, int errCode, String msg) {
				Log.d("blocker", "unregister push failed...  errorCode: " + errCode + "\nmsg: " + msg);
				xgHandler.sendEmptyMessage(UNREGISTER_XG_FAILED);
			}
		});
	}
	
	private void enhancePushService() {
		enableComponentIfNeeded(mContext, XGPushService.class.getName());
		enableComponentIfNeeded(mContext, XGPushReceiver.class.getName());
		enableComponentIfNeeded(mContext, XGPushActivity.class.getName());
		//enableComponentIfNeeded(mContext, FTPushReceiver.class.getName());
	}
	
	
	private static void enableComponentIfNeeded(Context context, String componentName) {
		PackageManager pmManager = context.getPackageManager();
		if (pmManager != null) {
			ComponentName cnComponentName = new ComponentName(
					context.getPackageName(), componentName);
			int status = pmManager.getComponentEnabledSetting(cnComponentName);
			if (status != PackageManager.COMPONENT_ENABLED_STATE_ENABLED) {
				pmManager.setComponentEnabledSetting(cnComponentName,
						PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
						PackageManager.DONT_KILL_APP);
			}
		}
	}
	
	
	public static class XgMsgUtils {
		
		public static void saveNotify(/*String nickName, String title, */String content, int msgCount) {
//			ConfigWrapper.put("nf_nick_name", nickName);
//			ConfigWrapper.put("nf_title", title);
			ConfigWrapper.put("nf_content", content);
			ConfigWrapper.put("nf_count", msgCount);
			ConfigWrapper.commit();
		}
		
		public static int getNotifyCount() {
			return ConfigWrapper.get("nf_count", 0);
		}
		
		public static void clearNotify() {
//			ConfigWrapper.remove("nf_nick_name");
//			ConfigWrapper.remove("nf_title");
			ConfigWrapper.remove("nf_content");
			ConfigWrapper.remove("nf_count");
			ConfigWrapper.commit();
		}
	}
}


退出账号
XGUtils.getInstances(this).unRegisterXG();

注册界面
XGUtils.getInstances(getActivity()).unRegisterXG();