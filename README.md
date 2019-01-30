# RecorderManager
因为在项目中经常需要使用音视频录制，所以写了一个公共库RecorderManager，欢迎大家使用。
## 一.效果展示
仿微信界面视频录制
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190129161436569.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsMjAxOGdvZA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190129161453953.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsMjAxOGdvZA==,size_16,color_FFFFFF,t_70)
2.音频录制界面比较简单，就不放图了
## 二.引用
1.Add it in your root build.gradle at the end of repositories
```
allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
```
2.Add the dependency

```
dependencies {
	        implementation 'com.github.MingYueChunQiu:RecorderManager:0.2'
	}
```
## 三.使用
### 1.音频录制
采用默认配置录制
```
mRecorderManager.recordAudio(mFilePath);
```
自定义配置参数录制

```
mRecorderManager.recordAudio(new RecorderOption.Builder()
                    .setAudioSource(MediaRecorder.AudioSource.MIC)
                    .setOutputFormat(MediaRecorder.OutputFormat.MPEG_4)
                    .setAudioEncoder(MediaRecorder.AudioEncoder.AAC)
                    .setAudioSamplingRate(44100)
                    .setBitRate(96000)
                    .setFilePath(path)
                    .build();
```
### 2.视频录制
#### (1).可以直接使用RecordVideoActivity，实现了仿微信风格的录制界面
```
                startActivity(new Intent(MainActivity.this, RecordVideoActivity.class));
```
通过在Intent中传入下列参数来设置路径和最长时间

```
 	//录制视频文件路径
    public static final String EXTRA_RECORD_VIDEO_FILE_PATH = PREFIX_EXTRA + "record_video_file_path";
    //录制视频最大时长
    public static final String EXTRA_RECORD_VIDEO_MAX_DURATION = PREFIX_EXTRA + "record_video_max_duration";
```
RecordVideoActivity里已经配置好了默认参数，可以直接使用，然后在onActivityResult里拿到视频路径的返回值

```
@Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode == Activity.RESULT_OK && requestCode == 0) {
            Log.e("onActivityResult", "onActivityResult: " + " " + data.getStringExtra(EXTRA_RECORD_VIDEO_FILE_PATH));
        }
    }
```

#### (2).如果想要界面一些控件的样式，可以继承RecordVideoActivity，里面提供了几个protected方法，可以拿到界面的一些控件

```
/**
     * 获取计时控件
     *
     * @return 返回计时AppCompatTextView
     */
    protected AppCompatTextView getTimingView() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getTimingView();
    }

    /**
     * 获取圆形进度按钮
     *
     * @return 返回进度CircleProgressButton
     */
    protected CircleProgressButton getCircleProgressButton() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getCircleProgressButton();
    }

    /**
     * 获取播放控件
     *
     * @return 返回播放AppCompatImageView
     */
    protected AppCompatImageView getPlayView() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getPlayView();
    }

    /**
     * 获取取消控件
     *
     * @return 返回取消AppCompatImageView
     */
    protected AppCompatImageView getCancelView() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getCancelView();
    }

    /**
     * 获取确认控件
     *
     * @return 返回确认AppCompatImageView
     */
    protected AppCompatImageView getConfirmView() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getConfirmView();
    }

    /**
     * 获取返回控件
     *
     * @return 返回返回AppCompatImageView
     */
    protected AppCompatImageView getBackView() {
        return mRecordVideoFg == null ? null : mRecordVideoFg.getBackView();
    }
```
#### (3).同时提供了对应的RecordVideoFragment，实现与RecordVideoActivity同样的功能，实际RecordVideoActivity就是包裹了一个RecordVideoFragment
1.创建RecordVideoFragment

```
/**
     * 获取录制视频Fragment实例（使用默认配置项）
     *
     * @param filePath 存储文件路径
     * @return 返回RecordVideoFragment
     */
    public static RecordVideoFragment newInstance(String filePath) {
        return newInstance(new RecordVideoOption.Builder()
                .setRecorderOption(new RecorderOption.Builder().buildDefaultVideoBean(filePath))
                .build());
    }

    /**
     * 获取录制视频Fragment实例
     *
     * @param option 录制配置信息对象
     * @return 返回RecordVideoFragment
     */
    public static RecordVideoFragment newInstance(RecordVideoOption option) {
        RecordVideoFragment fragment = new RecordVideoFragment();
        fragment.mOption = option;
        if (fragment.mOption == null) {
            fragment.mOption = new RecordVideoOption();
        }
        if (fragment.mOption.getRecorderOption() == null && fragment.getContext() != null) {
            File file = fragment.getContext().getExternalFilesDir(Environment.DIRECTORY_MOVIES);
            if (file != null) {
                fragment.mOption.setRecorderOption(new RecorderOption.Builder().buildDefaultVideoBean(
                        file.getAbsolutePath() +
                                File.separator + System.currentTimeMillis() + ".mp4"));
            }
        }
        return fragment;
    }
```
2.然后添加RecordVideoFragment到自己想要的地方就可以了
3.可以设置OnRecordVideoListener，拿到各个事件的回调

```
public class RecordVideoOption：
 	private RecorderOption option;//录制配置信息
    private int maxDuration;//最大录制时间
    private OnRecordVideoListener listener;//录制视频监听器

	/**
     * 录制视频监听器
     */
    public interface OnRecordVideoListener {
        /**
         * 当完成视频录制时回调
         *
         * @param option 录制配置信息对象
         */
        void onCompleteRecordVideo(RecorderOption option);

        /**
         * 当点击返回按钮时回调
         */
        void onClickBack();
    }
```
#### (4).如果想自定义自己的界面，可以直接使用RecorderManager类
1.创建RecorderManager实例
```
public class RecorderManager implements RecorderManagerable

/**
     * 创建录制管理类实例（使用默认录制类）
     *
     * @return 返回录制管理类实例
     */
    public static RecorderManagerable newInstance() {
        return new RecorderManager(new RecorderHelper());
    }

    /**
     * 创建录制管理类实例
     *
     * @param recorderable 实际录制类
     * @return 返回录制管理类实例
     */
    public static RecorderManagerable newInstance(Recorderable recorderable) {
        return new RecorderManager(recorderable);
    }
```
它们返回的都是RecorderManagerable 接口类型，RecorderManager 是默认的实现类，RecorderManager 内持有一个真正进行操作的Recorderable。

Recorderable是一个接口类型，由实现Recorderable的子类来进行录制操作，默认提供的是RecorderHelper，RecorderHelper实现了Recorderable。

```
public interface Recorderable {

    boolean recordAudio(String path);

    boolean recordAudio(RecorderOption bean);

    boolean recordVideo(Camera camera, Surface surface, String path);

    boolean recordVideo(Camera camera, Surface surface, RecorderOption bean);

    void release();

    MediaRecorder getMediaRecorder();
}
```
2.拿到后创建相机对象

```
if (mCamera == null) {
            mCamera = mManager.initCamera();
        }
        try {
            mCamera.setPreviewDisplay(svVideoRef.get().getHolder());
            mCamera.startPreview();
            mCamera.unlock();
        } catch (IOException e) {
            e.printStackTrace();
        }
```
3.录制

```
isRecording = mManager.recordVideo(mCamera, svVideoRef.get().getHolder().getSurface(), mOption.getRecorderOption());
```
4.释放

```
			mManager.release();
            mManager.releaseCamera(mCamera);
            mCamera = null;
```
## 四.总结
目前来说，大体流程就是这样，更详细的信息请到Github上查看， 后期将添加摄像头切换、闪光灯等更多功能，敬请关注，github地址为 https://github.com/MingYueChunQiu/RecorderManager ，码云地址为 https://gitee.com/MingYueChunQiu/RecorderManager ，如果它能对你有所帮助，请帮忙点个star，有什么建议或意见欢迎反馈。
