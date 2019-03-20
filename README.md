# InksLibrary
应用方法：
1. aar 应用
apply plugin: 'com.android.application'
android {
   repositories {
        flatDir {
            dirs 'libs'
        }
    }
}
dependencies {
    implementation(name: 'inkslibrary', ext: 'aar')
}
2.直接导入mode
dependencies {
implementation project(':inkslibrary')
}
3.远程添加
build.gradle (Project)中
添加   maven { url 'https://jitpack.io' }
allprojects {
    repositories {
        jcenter()
        maven { url 'https://jitpack.io' }
    }
}
build.gradle (app)中
添加   implementation 'com.github.inksnow:Inkslibrary:1.0.2'
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
     ......
    implementation 'com.github.inksnow:Inkslibrary:1.0.2'
}

AndroidManifest.xml
权限：（蓝牙，蓝牙需要位置，存储，网络，安装，卸载）
<uses-feature android:required="false" android:name="android.hardware.bluetooth_le"/>
<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"
    tools:ignore="ProtectedPermissions" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.CHANGE_CONFIGURATION"
    tools:ignore="ProtectedPermissions" />

下载新版本activity
设置configChanges这个值就可以避免Activity生命周期被回调，下面是这个值的详细说明：
configChanges属性可以设置多个值，中间使用竖线分割； 
orientation 屏幕在纵向和横向间旋转 
keyboardHidden 键盘显示或隐藏 
screenSize 屏幕大小改变了 
fontScale 用户变更了首选的字体大小 
locale 用户选择了不同的语言设定 
keyboard 键盘类型变更，例如手机从12键盘切换到全键盘 
touchscreen或navigation 键盘或导航方式变化，一般不会发生这样的事件

android:theme="@style/StyleTransparentActivity" 设置activity透明

<activity android:name="com.inks.inkslibrary.UI.LibUpActivity"
    android:theme="@style/StyleTransparentActivity"
    android:configChanges="orientation|keyboardHidden|screenSize">
</activity>

蓝牙连接服务

<service
    android:name="com.inks.inkslibrary.Service.BTServiceForMore"
    android:enabled="true" />

检测版本更新服务
<service
    android:name="com.inks.inkslibrary.Service.QueryVersionService"
    android:enabled="true" />
Android7.0以上需要访问文件权限
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="com.inks.inklibrary.fileProvider"
    android:grantUriPermissions="true"
    android:exported="false">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>

file_paths.xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <external-path path="." name="external_storage_root" />
</paths>

检查更新使用方法：

启动服务：
private QueryVersionService queryVersionService;
private final ServiceConnection queryVersionConn = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName componentName, IBinder service) {
        queryVersionService = ((QueryVersionService.LocalBinder) service).getService();
    }
    @Override
    public void onServiceDisconnected(ComponentName componentName) {
    }
};
//在onCreate中启动服务
Intent QServiceIntent = new Intent(this, QueryVersionService.class);
bindService(QServiceIntent, queryVersionConn, BIND_AUTO_CREATE);




检查更新：
启动服务成功后可以在任何时候直接调用checkUpdate（）方法
第一个参数：获取版本信息地址
第二个参数：服务器版本与现在版本不相等后是否直接弹出新版本信息
queryVersionService.checkUpdate("http://192.168.1.45/apps/api/getInfoForD.php",true);
服务返回必须为json格式，包含以下字段：
private String appVersion;//服务器上的APP版本
private String versionMgsEN;//更新内容英文
private String versionMgsCH;//更新内容中文
private String mustUp;//是否必须升级
private String downUri;//APK下载路径

服务器getInfoForD.php 参考

<?php
$output = array();
$output["appVersion"] = '3.0.1'; 
$output["versionMgsEN"] = '1.语言选择增加韩语
\n2.添加电量显示\n3.添加状态指示\n4.添加体感平衡调节'; 
$output["versionMgsCH"] = '1.语言选择增加韩语
\n2.添加电量显示\n3.添加状态指示\n4.添加体感平衡调节'; 
$output["mustUp"] = "1"; 
$output["downUri"] = "http://192.168.1.45/apps/api/downForD.php"; 
foreach ( $output as $key => $value ) { 
  $output[$key]  = urlencode ( $value ); 
}
exit (urldecode (json_encode($output)));


downForD.php参考
 header('Content-Disposition:attachment;filename=' . basename('../apk/sparkForD.apk'));
 header('File-Name:'. basename('../apk/sparkForD.apk'));
 header('Content-Length:' . filesize('../apk/sparkForD.apk'));
 //读取文件并写入到输出缓冲
 readfile('../apk/sparkForD.apk');






注册检查更新结果广播：
（可接收，可不接收，根据需要做相应处理）
private final BroadcastReceiver ckeckVersionReceiver = new BroadcastReceiver() {
    @TargetApi(Build.VERSION_CODES.JELLY_BEAN_MR2)
    @Override
    public void onReceive(Context context, Intent intent) {
        final String action = intent.getAction();
        if (QueryVersionService.MGS.equals(action)) {

            switch (intent.getIntExtra("MGS", 0)) {
                case 404:
                    T.showShort(MainActivity.this,"网络错误");
                    break;
                case 201:
                    T.showShort(MainActivity.this,"已是最新版本");
                    break;
                case 200:
                    T.showShort(MainActivity.this,"发现新版本");
                    break;
            }
        }
    }
};
private static IntentFilter intentFilter() {
    final IntentFilter intentFilter = new IntentFilter();
    intentFilter.addAction(QueryVersionService.MGS);
    return intentFilter;
}
@Override
protected void onResume() {
    super.onResume();
    registerReceiver(ckeckVersionReceiver, intentFilter());
}

@Override
protected void onPause() {
    super.onPause();
    unregisterReceiver(ckeckVersionReceiver);
}






设置弹出新版本弹框样式及文字语言：（需要在调用）checkUpdate（）方法前设置

可设置参数及默认值如下：
使用LibUpUIData.setXXXX() 设置下列参数

//状态栏
private static boolean statuBar = true;
//宽（单位 dp）
private static int  width = 340;
//高
private static int  height = 200;
//圆角
private static int  radius = 15;
//滚动条持续时间（秒）
private static int  duration = 5;
//背景色
private static int[] colours = {0XFFaee5f5,0Xff00c6ff};
//渐变方向
private static GradientDrawable.Orientation orientation = GradientDrawable.Orientation.LEFT_RIGHT;
//字體大小(標題和按鈕、內容標題、內容 單位Dp)
private static int[] textSizes = {18,16,14};
//字體顏色(內容 、按鈕正常、按鈕無法點擊)
private static int[] textcolours = {0XFFffffff,0Xffffffff,0XFF777777};
//button padding(單位dp)
private static int buttonPadding = 10;
//分割线颜色
private static int dividingLineColour = 0X66666666;
//下载中progressBar颜色
private static int progressBarColour = 0XFFFF6666;
//使用语言 true 中文
private static boolean language = true;
private static String title ="版本更新";
private static String currentVersion ="当前版本：";
private static String  latestVersion = "最新版本：";
private static String  mgsTitle = "更新内容：";
private static String  no = "取消";
private static String  downYes = "下载";
private static String  loading = "正在下载    ";
private static String  failText = "下载失败，请检查网络重试";
private static String  failOk = "确定";
//权限
private static String  permissionTitle = "存储权限不可用";
private static String  permissionMgs = "请开启存储权限";
private static String  permissionOpen = "立即开启";

private static String authority = "com.inks.inklibrary.fileProvider";



蓝牙服务：
设置蓝牙读写UUID
SampleGattAttributes.setXXX()

public static void setREAD2(UUID READ2) {
    SampleGattAttributes.READ2 = READ2;
}
public static void setWRITE2(UUID WRITE2) {
    SampleGattAttributes.WRITE2 = WRITE2;
}
public static void setREAD(UUID READ) {
    SampleGattAttributes.READ = READ;
}
public static void setWRITE(UUID WRITE) {
    SampleGattAttributes.WRITE = WRITE;
}

默认：
public static UUID READ2 = UUID.fromString("00001002-0000-1000-8000-00805f9b34fb");
public static UUID WRITE2 = UUID.fromString("00001001-0000-1000-8000-00805f9b34fb");
public static UUID READ = UUID.fromString("00004a5b-0000-1000-8000-00805f9b34fb");
public static UUID WRITE = UUID.fromString("00004a5b-0000-1000-8000-00805f9b34fb");


绑定蓝牙服务：
private BTServiceForMore btServiceForMore;

private final ServiceConnection mServiceConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName componentName, IBinder service) {
        btServiceForMore = ((BTServiceForMore.LocalBinder) service).getService();
        if (!btServiceForMore.initialize()) {
            Log.e("MainApplication", "Unable to initialize Bluetooth");
        } else {
            Log.e("MainApplication", "服务绑定成功");
        }
    }
    @Override
    public void onServiceDisconnected(ComponentName componentName) {
        Log.e("MainApplication", "服务解除绑定");
    }
};

Intent gattServiceIntent = new Intent(this, BTServiceForMore.class);
bindService(gattServiceIntent, mServiceConnection, BIND_AUTO_CREATE);

连接及接收数据广播监听：


private final BroadcastReceiver mGattUpdateReceiver = new BroadcastReceiver() {
    @TargetApi(Build.VERSION_CODES.JELLY_BEAN_MR2)
    @Override
    public void onReceive(Context context, Intent intent) {
        final String action = intent.getAction();
        final String MAC = intent.getStringExtra("MAC");
        final String NAME = intent.getStringExtra("NAME");
        L.i(MAC);
        L.i(NAME+"");
        L.i(action);
        if (BTServiceForMore.ACTION_GATT_CONNECTED.equals(action)) {
            //连接成功
            linkTure(NAME,MAC);

        } else if (BTServiceForMore.ACTION_GATT_DISCONNECTED.equals(action)) {
            //断开连接
            linkFlast(NAME,MAC);

        }  else if (BTServiceForMore.ACTION_DATA_AVAILABLE.equals(action)) {
            //收到数据
             linkData(intent.getStringExtra(BTServiceForMore.ALL_DATA),MAC);
        }else if("LINK_EXCESS_FOUR".equals(action)){
            linkExcessFour();
        }
    }
};



registerReceiver(mGattUpdateReceiver, makeGattUpdateIntentFilter());

    private static IntentFilter makeGattUpdateIntentFilter() {
        final IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(BTServiceForMore.ACTION_GATT_CONNECTED);
        intentFilter.addAction(BTServiceForMore.ACTION_GATT_DISCONNECTED);
        intentFilter.addAction(BTServiceForMore.ACTION_DATA_AVAILABLE);
        intentFilter.addAction("LINK_EXCESS_FOUR");

        return intentFilter;
    }


服务的一些方法：
连接
public boolean connect(final String address)
发送数据
public void writeCharacteristic(String mac, byte[] bytes)
public void writeCharacteristic(BluetoothGattCharacteristic characteristic, BluetoothGatt gatt, byte[] bytes)
发送给全部连接的设备
public void writeAllCharacteristic(byte[] bytes) 
获取已连接的蓝牙MAC
public ArrayList<String> getLinkBt()
获取已连接的蓝牙MAC 和Name
public ArrayList<BTBean> getBTBean()

public void close(String mac)
public void closeAll() 
public boolean disconnect(String mac)

说明：每次连接后如果连接失败status==133，会再重连2次
在进行BLE开发过程中可能会遇到操作失败等情况,这个时候可能需要断开与BLE的连接或者清理相关资源.在BluetoothGatt类中有两个相关的方法 
1. disconnect() 
2. close() 
那么这个两个方法有什么区别,又该如何使用呢.
disconnect()方法: 如果调用了该方法之后可以调用connect()方法进行重连,这样还可以继续进行断开前的操作.
close()方法: 一但调用了该方法, 如果你想再次连接,必须调用BluetoothDevice的connectGatt()方法. 因为close()方法将释放BluetootheGatt的所有资源.

需要注意的问题: 
当你需要手动断开时,调用disconnect()方法，此时断开成功后会回调onConnectionStateChange方法,在这个方法中再调用close方法释放资源。 
如果在disconnect后立即调用close，会导致无法回调onConnectionStateChange方法。


其他工具类：
APKVersionCodeUtils
getVersionCode获取当前本地apk的版本
getVerName获取版本号名称

AppUtil
openAppByPackageName通过指定的包名启动应用
unInstall卸载指定应用的包名
checkApplication判断该包名的应用是否安装

AudioUtil
getMediaMaxVolume获取多媒体最大音量
getMediaVolume获取多媒体音量
getCallMaxVolume获取通话最大音量
getSystemMaxVolume获取系统音量最大值
getSystemVolume获取系统音量
getAlermMaxVolume获取提示音量最大值
setMediaVolume设置多媒体音量
setCallVolume设置通话音量
setSpeakerStatus关闭/打开扬声器播放

ClickUtil
isFastDoubleClick()判断两次点击的间隔，如果小于1s，则认为是多次无效点击（任意两个view，固定时长1s） 
isFastDoubleClick(long diff)判断两次点击的间隔，如果小于diff，则认为是多次无效点击（任意两个view，自定义间隔时长）
isFastDoubleClick(int buttonId) 判断两次点击的间隔，如果小于1s，则认为是多次无效点击（同一个view，固定时长1s） 
isFastDoubleClick(int buttonId, long diff)判断两次点击的间隔，如果小于diff，则认为是多次无效点击（同一按钮，自定义间隔时长）




DensityUtils常用单位转换的辅助类
dp2pxdp转px
sp2px
px2dp
px2sp

GetResId
 获取布局id
 GetResId.getId(this, "layout", "activity_main")
获取控件id
GetResId.getId(this, "id", "up")
getStringId获取 string 值
getDrawableId获取 drawable 布局文件 或者 图片的
getStyleId获取 style
getAnimId获取 anim
getColorId获取color
getResourceId

Install
install(String apkPath)静默安装，需要手机ROOT。
copyApkFromAssets 从assets复制文件到内存
isRoot()判断当前手机是否有ROOT权限

KeyBoardUtils
openKeybord打开软键盘
closeKeybord关闭软键盘

L Log简化

NetWorkGetUtils
isNetWorkAvailableOfGet检查互联网地址是否可以访问-使用get请求

NumberUtil
intToByte1
intToByte2
intToByte4int整数转换为4字节的byte数组
longToByte8long整数转换为8字节的byte数组

ProgressDialogUtil
showPd
dismissPd



ScreenUtils
getScreenWidth获得屏幕宽度
getScreenHeight获得屏幕高度
getStatusHeight获得状态栏的高度
snapShotWithStatusBar获取当前屏幕截图，包含状态栏
snapShotWithoutStatusBar获取当前屏幕截图，不包含状态栏

SDCardUtilsSD卡相关的辅助类
isSDCardEnable判断SDCard是否可用
getSDCardPath获取SD卡路径
getSDCardAllSize获取SD卡的剩余容量 单位byte
getFreeBytes获取指定路径所在空间的剩余可用容量字节数，单位byte
getRootDirectoryPath获取系统存储路径

SharedPrefencesUtils

StringAndByteUtils
bytesToHexString
printHexByte
stringToBytes
judgeFlag//00000000 ~  11111111
RemoveSpaces

T  Toast统一管理类
isShow
showShort

ViewPagerAutoPlayTimeUtilViewPager自动轮播时间设置
setViewPagerScrollSpeed
