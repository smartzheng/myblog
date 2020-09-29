+++
title= "bilibili开源库ijkPalyer的集成和使用"
date= "2017-12-08"
categories= [ "Android" ]
+++

最近在我的个人项目中集成了bilibili视频播放库ijkPalyer，网上的很多教程所写的是之前版本的使用方法，官方文档也没有写的太清楚，所以在此做一个简单的介绍。
个人项目：一个基于MVP，dagger2，RXJava，Retrofit，Glide，集成bilibili的ijkPlayer，用Bmob实现后台服务，遵循Material Design风格的APP，数据主要来自豆瓣。欢迎star。
地址：https://github.com/smartzheng/LoveDouBan  

1.引入
之前的版本似乎需要进行源码编译，现在直接引入即可。ijkPalyer地址：https://github.com/Bilibili/ijkplayer
最简单的方式，直接在app下的gradle添加：
```
# required
allprojects {
    repositories {
        jcenter()
    }
}

dependencies {
    # required, enough for most devices.
    compile 'tv.danmaku.ijk.media:ijkplayer-java:0.8.4'
    compile 'tv.danmaku.ijk.media:ijkplayer-armv7a:0.8.4'

    # Other ABIs: optional
    compile 'tv.danmaku.ijk.media:ijkplayer-armv5:0.8.4'
    compile 'tv.danmaku.ijk.media:ijkplayer-arm64:0.8.4'
    compile 'tv.danmaku.ijk.media:ijkplayer-x86:0.8.4'
    compile 'tv.danmaku.ijk.media:ijkplayer-x86_64:0.8.4'

    # ExoPlayer as IMediaPlayer: optional, experimental
    compile 'tv.danmaku.ijk.media:ijkplayer-exo:0.8.4'
}
```
如果对源码有兴趣，可以将项目下的以下指出的文件夹（ijkplayer-example可根据需求进行选择，下面会进行介绍）引入到app中：
![](http://upload-images.jianshu.io/upload_images/2983970-622faf359ab4502a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其中tools直接拷贝，其他文件夹以module的形式引入，再添加为项目依赖即可。

ijkPlayer的定制性比较强，在官方地址下有一个sample，可以将其直接以项目的形式引入至自己的项目中进行使用，其中包括了不少官方写好的工具类和控件。
当然，为了减小apk大小，直接引入一个项目不是好选择，所以我选择复制粘贴需要的代码，如下：
![](http://upload-images.jianshu.io/upload_images/2983970-edf8ac6b8486b906.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上图中的wiget目录可全部拷贝入自己的项目中，拷贝完成之后会缺少很多资源文件。所以需要再根据报错信息分别将res目录下的相关资源和声明复制到对应的目录或文件中：
![](http://upload-images.jianshu.io/upload_images/2983970-4e4ca21654d4639d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
layout直接拷贝，values复制内容至自己项目对应的values文件中去，并注意删除重复和未使用的声明。 

至此，引入完毕。
2.简单使用
1）初始化：官方提供以下两种方式进行初始化操作，在应用初始化或者使用视频播放的activity初始化时调用以下方法。

        //第一：默认初始化
        Bmob.initialize(this, "e8f177acd86e0fde391b2af19243fc1b");
        // 注:自v3.5.2开始，数据sdk内部缝合了统计sdk，开发者无需额外集成，传渠道参数即可，不传默认没开启数据统计功能
        //Bmob.initialize(this, "Your Application ID","bmob");

        //第二：自v3.4.7版本开始,设置BmobConfig,允许设置请求超时时间、文件分片上传时每片的大小、文件的过期时间(单位为秒)，
        //BmobConfig config =new BmobConfig.Builder(this)
        ////设置appkey
        //.setApplicationId("Your Application ID")
        ////请求超时时间（单位为秒）：默认15s
        //.setConnectTimeout(30)
        ////文件分片上传时每片的大小（单位字节），默认512*1024
        //.setUploadBlockSize(1024*1024)
        ////文件的过期时间(单位为秒)：默认1800s
        //.setFileExpiration(2500)
        //.build();
        //Bmob.initialize(config);

2）布局：
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- The main content view -->
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/colorPrimary">

        <com.zs.douban.customview.media.IjkVideoView
            android:id="@+id/video_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="center">

        </com.zs.douban.customview.media.IjkVideoView>

        <TextView
            android:id="@+id/toast_text_view"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="left|center_vertical"
            android:background="@color/colorPrimary"
            android:padding="16dp"
            android:textSize="16sp"
            android:visibility="gone"/>

        <TableLayout
            android:visibility="invisible"
            android:id="@+id/hud_view"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="right|center_vertical"
            android:background="@color/colorPrimary"
            android:padding="8dp"/>

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:minHeight="?attr/actionBarSize"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
    </FrameLayout>

    <!-- The navigation drawer -->
    <FrameLayout
        android:id="@+id/right_drawer"
        android:layout_width="240dp"
        android:layout_height="match_parent"
        android:layout_gravity="right"
        android:background="@color/colorPrimary"/>
</android.support.v4.widget.DrawerLayout>
```

以上是我项目下的布局内容，可以根据需求自定义，但是需要引入com.zs.douban.customview.media.IjkVideoView，以及一个TableLayout，播放视频时TableLayout会显示对应的播放信息，如果不需要，可以设为invisible，但是如果不加入，会报错，可能需要修改源码才能避免。

3）配置和播放
    
    private void initPlayer() {
        IjkMediaPlayer.loadLibrariesOnce(null);
        IjkMediaPlayer.native_profileBegin("libijkplayer.so");
        mVideoView = (IjkVideoView) findViewById(R.id.video_view);
        mVideoView.setHudView(mHudView);//没有TableLayout此处会报空指针
        AndroidMediaController controller = new AndroidMediaController(this, false);
        mVideoView.setMediaController(controller);
    }

    private void play() {
        String url = getIntent().getStringExtra("url");//设置播放地址
        mVideoView.setVideoURI(Uri.parse(url));
        mVideoView.start();
        setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
    }
4)全屏
默认情况下，播放为竖屏小窗口。全屏需要在IjkViewView源码中进行修改，添加以下代码：
```
   public IRenderView getmRenderView() {
        return mRenderView;
    }
    public int getmVideoWidth() {
        return mVideoWidth;
    }
    public int getmVideoHeight() {
        return mVideoHeight;
    }
```
在播放视频的activity中增加以下方法（具体可以查看我的项目下的MoviePlayActivity）：
```
//横竖屏检测切换
    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        //重新获取屏幕宽高
        initScreenInfo();
        if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {//切换为横屏
            LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) mVideoView.getLayoutParams();
            lp.height = screenHeight;
            lp.width = screenWidth;
            mVideoView.setLayoutParams(lp);
        } else {
            LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) mVideoView.getLayoutParams();
            lp.height = screenWidth * 9 / 16;
            lp.width = screenWidth;
            mVideoView.setLayoutParams(lp);
        }
        setScreenRate(currentSize);
    }
    //初始化屏幕配置信息
    private void initScreenInfo() {
        WindowManager wm = this.getWindowManager();
        screenWidth = wm.getDefaultDisplay().getWidth();
        screenHeight = wm.getDefaultDisplay().getHeight();
    }

    //设置视频播放比例
    public void setScreenRate(int rate) {
        int width = 0;
        int height = 0;
        if (getRequestedOrientation() == ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE) {// 横屏
            if (rate == SIZE_DEFAULT) {
                width = mVideoView.getmVideoWidth();
                height = mVideoView.getmVideoHeight();
            } else if (rate == SIZE_4_3) {
                width = screenHeight / 3 * 4;
                height = screenHeight;
            } else if (rate == SIZE_16_9) {
                width = screenHeight / 9 * 16;
                height = screenHeight;
            }
        } else { //竖屏
            if (rate == SIZE_DEFAULT) {
                width = mVideoView.getmVideoWidth();
                height = mVideoView.getmVideoHeight();
            } else if (rate == SIZE_4_3) {
                width = screenWidth;
                height = screenWidth * 3 / 4;
            } else if (rate == SIZE_16_9) {
                width = screenWidth;
                height = screenWidth * 9 / 16;
            }
        }
        if (width > 0 && height > 0) {
            FrameLayout.LayoutParams lp = (FrameLayout.LayoutParams) mVideoView.getmRenderView().getView().getLayoutParams();
            lp.width = width;
            lp.height = height;
            mVideoView.getmRenderView().getView().setLayoutParams(lp);
        }
    }

    //手动切换
    private void fullChangeScreen() {
        if (getRequestedOrientation() == ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE) {// 切换为竖屏
            setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
        } else {
            setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
        }
    }
```
播放时调用setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE)即可。

5)播放进度，需自行加入SeekBar：
```
seekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
    ....
    @Override
    public void onStopTrackingTouch(SeekBar seekBar) {
         video.seekTo(seekBar.getProgress()*video.getDuration()/100);
         ...
    }
});
//视频开始播放时使用handle.sendMessageDelayed更新时间显示
private void refreshTime(){
    int totalSeconds = video.getCurrentPosition() / 1000;
    int seconds = totalSeconds % 60;
    int minutes = (totalSeconds / 60) % 60;
    int hours = totalSeconds / 3600;
    String ti=hours > 0 ? String.format("%02d:%02d:%02d", hours, minutes, seconds):String.format("%02d:%02d", minutes, seconds);
    time.setText(ti);
}
```

4、最后，需要根据需求在返回键或者activity销毁时取消后台播放：
```
    @Override
    public void onBackPressed() {
        mBackPressed = true;

        super.onBackPressed();
    }

    @Override
    protected void onStop() {
        super.onStop();

        if (mBackPressed || !mVideoView.isBackgroundPlayEnabled()) {
            mVideoView.stopPlayback();
            mVideoView.release(true);
            mVideoView.stopBackgroundPlay();
        } else {
            mVideoView.enterBackground();
        }
        IjkMediaPlayer.native_profileEnd();
    }
```

补充Activity所有源码：
```
public class MoviePlayActivity extends AppCompatActivity {
    @InjectView(R.id.video_view)
    IjkVideoView mVideoView;
    @InjectView(R.id.toast_text_view)
    TextView mToastTextView;
    @InjectView(R.id.hud_view)
    TableLayout mHudView;
    @InjectView(R.id.toolbar)
    Toolbar mToolbar;
    @InjectView(R.id.right_drawer)
    FrameLayout mRightDrawer;
    @InjectView(R.id.drawer_layout)
    DrawerLayout mDrawerLayout;
    private boolean mBackPressed;
    private static final int SIZE_DEFAULT = 0;
    private static final int SIZE_4_3 = 1;
    private static final int SIZE_16_9 = 2;
    private int currentSize = SIZE_16_9;
    private int screenHeight;
    private int screenWidth;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_movie_play);
        ButterKnife.inject(this);
        initToolBar();
        initPlayer();
        play();
    }

    /**
     * 初始化ToolBar
     */
    private void initToolBar() {
        mToolbar.setTitleTextColor(Color.WHITE);
        setSupportActionBar(mToolbar);
        final ActionBar actionBar = getSupportActionBar();
        if (actionBar != null) {
//            actionBar.setHomeAsUpIndicator(R.drawable.ic_menu);
            actionBar.setDisplayHomeAsUpEnabled(true);
            actionBar.setTitle(getIntent().getStringExtra("title"));
        }
    }

    //
    private void initPlayer() {
        IjkMediaPlayer.loadLibrariesOnce(null);
        IjkMediaPlayer.native_profileBegin("libijkplayer.so");
        mVideoView = (IjkVideoView) findViewById(R.id.video_view);
        mVideoView.setHudView(mHudView);
        AndroidMediaController controller = new AndroidMediaController(this, false);
        mVideoView.setMediaController(controller);
    }

    private void play() {
        String url = getIntent().getStringExtra("url");
        mVideoView.setVideoURI(Uri.parse(url));
        mVideoView.start();
        setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
    }

    @Override
    public void onBackPressed() {
        mBackPressed = true;

        super.onBackPressed();
    }

    @Override
    protected void onStop() {
        super.onStop();

        if (mBackPressed || !mVideoView.isBackgroundPlayEnabled()) {
            mVideoView.stopPlayback();
            mVideoView.release(true);
            mVideoView.stopBackgroundPlay();
        } else {
            mVideoView.enterBackground();
        }
        IjkMediaPlayer.native_profileEnd();
    }

    //横竖屏检测切换
    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        //重新获取屏幕宽高
        initScreenInfo();
        if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {//切换为横屏
            LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) mVideoView.getLayoutParams();
            lp.height = screenHeight;
            lp.width = screenWidth;
            mVideoView.setLayoutParams(lp);
        } else {
            LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) mVideoView.getLayoutParams();
            lp.height = screenWidth * 9 / 16;
            lp.width = screenWidth;
            mVideoView.setLayoutParams(lp);
        }
        setScreenRate(currentSize);
    }
    //初始化屏幕配置信息
    private void initScreenInfo() {
        WindowManager wm = this.getWindowManager();
        screenWidth = wm.getDefaultDisplay().getWidth();
        screenHeight = wm.getDefaultDisplay().getHeight();
    }

    //设置视频播放比例
    public void setScreenRate(int rate) {
        int width = 0;
        int height = 0;
        if (getRequestedOrientation() == ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE) {// 横屏
            if (rate == SIZE_DEFAULT) {
                width = mVideoView.getmVideoWidth();
                height = mVideoView.getmVideoHeight();
            } else if (rate == SIZE_4_3) {
                width = screenHeight / 3 * 4;
                height = screenHeight;
            } else if (rate == SIZE_16_9) {
                width = screenHeight / 9 * 16;
                height = screenHeight;
            }
        } else { //竖屏
            if (rate == SIZE_DEFAULT) {
                width = mVideoView.getmVideoWidth();
                height = mVideoView.getmVideoHeight();
            } else if (rate == SIZE_4_3) {
                width = screenWidth;
                height = screenWidth * 3 / 4;
            } else if (rate == SIZE_16_9) {
                width = screenWidth;
                height = screenWidth * 9 / 16;
            }
        }
        if (width > 0 && height > 0) {
            FrameLayout.LayoutParams lp = (FrameLayout.LayoutParams) mVideoView.getmRenderView().getView().getLayoutParams();
            lp.width = width;
            lp.height = height;
            mVideoView.getmRenderView().getView().setLayoutParams(lp);
        }
    }

    //手动切换
    private void fullChangeScreen() {
        if (getRequestedOrientation() == ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE) {// 切换为竖屏
            setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
        } else {
            setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
        }
    }
}
```



































