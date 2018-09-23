>文章链接：[https://mp.weixin.qq.com/s/n6EXvfmpNPtWM1kEnGgwUA](https://mp.weixin.qq.com/s/n6EXvfmpNPtWM1kEnGgwUA)

摇一摇红包效果已经是老生常谈的了，利用手机的传感器识别摇一摇，同时过程中进行动画+震动+声音的效果。Ps：百度网页版「摇一摇」三个字，会有效果的，皮一哈！     
效果图：  

![](https://user-gold-cdn.xitu.io/2018/9/22/1660021404e7f9f1?w=159&h=216&f=gif&s=880485)  

摇一摇主要通过`SensorManager`监听手机，实现 `SensorEventListener`，在`onSensorChanged`去判断，根据加速度来判断摇晃的程度。
```
ShakeSensorListener shakeListener = new ShakeSensorListener();
SensorManager sensorManager = (SensorManager)getSystemService(Context.SENSOR_SERVICE);
        
private class ShakeSensorListener implements SensorEventListener {

    @Override
    public void onSensorChanged(SensorEvent event) {
         //避免一直摇
        if (isShake) {
            return;
        }
         // 开始动画
        anim.start();
        float[] values = event.values;
        /*
         * x : x轴方向的重力加速度，向右为正
         * y : y轴方向的重力加速度，向前为正
         * z : z轴方向的重力加速度，向上为正
         */
        float x = Math.abs(values[0]);
        float y = Math.abs(values[1]);
        float z = Math.abs(values[2]);
        //加速度超过19，摇一摇成功
        if (x > 19 || y > 19 || z > 19) {
            isShake = true;
            //播放声音
            playSound(MainActivity.this);
            //震动，注意权限
            vibrate( 500);
            //仿网络延迟操作，这里可以去请求服务器...
            new Handler().postDelayed(new Runnable() {
                @Override
                public void run() {
                    //弹框
                    showDialog();
                    //动画取消
                    anim.cancel();
                }
            },1000);
        }
    }

    @Override
    public void onAccuracyChanged(Sensor sensor, int accuracy) {
    }
}
```
如果手机一直在摇晃，会不停的调用onSensorChanged ，而我们只想要一次摇一摇的效果，所以加了`isShake` 字段去判断。 在一次摇一摇事件完成后置false，可以继续摇一摇。  

注册监听，同时别忘了取消注册。
```
@Override
protected void onResume() {
    //注册监听加速度传感器
    sensorManager.registerListener(shakeListener, sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER),
            SensorManager.SENSOR_DELAY_FASTEST);
    super.onResume();
}

@Override
protected void onPause() {
    //取消注册
    sensorManager.unregisterListener(shakeListener);
    super.onPause();
}
```

摇一摇过程可以执行动画效果。
```
ObjectAnimator anim = ObjectAnimator.ofFloat(imgHand,"rotation",0f,45f,-30f,0f);
anim.setDuration(500);
anim.setRepeatCount(ValueAnimator.INFINITE);
```

播放声音，这里放在raw 资源文件里的。  
```
private void playSound(Context context) {
    MediaPlayer player = MediaPlayer.create(context,R.raw.shake_sound);
    player.start();
}
```

震动效果，这里注意要在AndroidManifest 文件里添加权限 `<uses-permission android:name="android.permission.VIBRATE" />` 
```
private void vibrate(long milliseconds) {
    Vibrator vibrator = (Vibrator)getSystemService(Service.VIBRATOR_SERVICE);
    vibrator.vibrate(milliseconds);
}
```
一次摇一摇后，这里在弹框消失后可继续摇一摇。
```
private void showDialog() {
    final AlertDialog mAlertDialog = new AlertDialog.Builder(this).show();
    View view = LayoutInflater.from(this).inflate(R.layout.layout_dialog,null);
    mAlertDialog.setContentView(view);
    mAlertDialog.setOnCancelListener(new DialogInterface.OnCancelListener() {
        @Override
        public void onCancel(DialogInterface dialog) {
            //这里让弹框取消后，才可以执行下一次的摇一摇
            isShake = false;
            mAlertDialog.cancel();
        }
    });
    Window window = mAlertDialog.getWindow();
    window.setBackgroundDrawable(new ColorDrawable(0x00000000));
}
```
至此，一套摇一摇效果完成！

github地址：[https://github.com/taixiang/shake](https://github.com/taixiang/shake)

欢迎关注我的个人博客：[https://www.manjiexiang.cn/](https://www.manjiexiang.cn/)  

更多精彩欢迎关注微信号：春风十里不如认识你  
一起学习，一起进步，欢迎上车，有问题随时联系，一起解决！！！

![](https://user-gold-cdn.xitu.io/2018/8/12/1652cd77eaebeb98?w=900&h=540&f=jpeg&s=64949)    
