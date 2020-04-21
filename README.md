# Android-Activity-to-Foreground
Let activity to foreground from background

### Activity class

    // define
    private static MainActivity mainActivity;
    private MainActivityReceiver mMainActivityReceiver;
    public final static String thisAction = TAG + ".returnHere";
    
    // onCreate
    if (mainActivity == null){
        mainActivity = MainActivity.this;
        mMainActivityReceiver = new MainActivityReceiver();
        registerReceiver(mMainActivityReceiver, new IntentFilter(thisAction));
    }    
    
    // define boardcast receiver for this Activity
    public class MainActivityReceiver extends BroadcastReceiver {
        private final String TAG = MainActivityReceiver.class.getCanonicalName();

        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public void onReceive(Context context, Intent intent) {
            try {
                Log.e(TAG, "onReceive: action = " + intent.getAction());
//                unregisterReceiver(mMainActivityReceiver);

                Intent i = new Intent(mainActivity, MainActivity.class);
                i.addFlags(Intent.FLAG_ACTIVITY_REORDER_TO_FRONT);// * onResume activity avoid re onCreate it
                startActivity(i);
            } catch (Exception e) {
                Log.e(TAG, "onReceive: exception = " + e.toString());
            }
        }
    }
    
### Service

    // add in the status to do let activity to foreground when it isn't in foreground. 
    new Handler().postDelayed(new Runnable() {
    @Override
    public void run() {
        if (isAppIsInBackground(context)) {
            Log.d(TAG, "onReceive: App is in background");

            try {

                Intent intent = new Intent();
                intent.setAction(MainActivity.thisAction);
                intent.putExtra("data","Notice me senpai!");
                context.sendBroadcast(intent);

            }catch (Exception e){
                Log.e(TAG, "run: Exception = " + e.toString());
            }
        }
        else
            Log.d(TAG, "onReceive: App is not in Background");
    }
    },5000);
    
    private boolean isAppIsInBackground(Context context) {
        boolean isInBackground = true;
        ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
        if (Build.VERSION.SDK_INT > Build.VERSION_CODES.KITKAT_WATCH) {
            List<ActivityManager.RunningAppProcessInfo> runningProcesses = am.getRunningAppProcesses();
            for (ActivityManager.RunningAppProcessInfo processInfo : runningProcesses) {
                if (processInfo.importance == ActivityManager.RunningAppProcessInfo.IMPORTANCE_FOREGROUND) {
                    for (String activeProcess : processInfo.pkgList) {
                        if (activeProcess.equals(context.getPackageName())) {
                            isInBackground = false;
                        }
                    }
                }
            }
        } else {
            List<ActivityManager.RunningTaskInfo> taskInfo = am.getRunningTasks(1);
            ComponentName componentInfo = taskInfo.get(0).topActivity;
            if (componentInfo.getPackageName().equals(context.getPackageName())) {
                isInBackground = false;
            }
        }

        return isInBackground;
    }
