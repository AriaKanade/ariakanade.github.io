# 使用广播实现音乐播放器

**完成的功能：**

1. 音乐的播放与暂停
2. 使用RecyclerView的点击事件切换音乐
3. 音乐的上一首与下一首切换

## **展示图：**

![image-20210511120046567](..\..\image\image-20210511120046567.png) 

![image-20210511120722547](..\..\image\image-20210511120722547.png) 





## 功能说明：

1. 音乐资源文件存放在assets目录下，使用AssetManager类进行操作
2. 音乐的点击切换与上一首下一首切换主要通过广播机制来实现，具体请看功能分析
3. 音乐的播放由后台的Service控制，使用MediaPlayer类完成



## 核心代码说明：

![image-20210511122135220](C:\Users\admin\Documents\GitHub\ariakanade.github.io\image\image-20210511122135220.png) 

**项目结构如图，其中需要说明的点：**

- **entity下的Music是一个对应音乐信息的JavaBean**
- **utils下的Status是一个枚举类型，记录播放状态**
- **assets下存放音乐资源**

**MainActivity中的Action**

```java
    //Service接收的，从MainActivity发送的控制类型广播
	public static final String CONTROL_ACTION = "control_music_play";
	//Service接收的，从RecyclerView发送的切换歌曲广播
    public static final String CHANGE_ACTION = "change_by_click";
	//MainActivity接收的，从Service发送的歌曲播放状态与歌曲的切换广播
    public static final String UPDATE_ACTION = "update_by_service";
```

**Music.java**

```java
public class Music {
    private String author;	//歌手名

    private String title;	//歌曲名

    private String fileName; //文件名

    //constructor...
    //getter setter...
}
```

**MainActivity与Service中的dataInit方法**

```java
    public void dataInit(){
        try {
            assetManager = getAssets();
            for (String fileName:assetManager.list("")){
                if(fileName.contains(".mp3")){
                    String musicName = fileName.replace(".mp3","");
                    if(fileName.contains("-")){
                        list.add(new Music(musicName.split("-")[0],musicName.split("-")[1],fileName));
                    }else{
                        list.add(new Music(musicName,"无相关信息",fileName));
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

dataInit方法对一个List<Music>类型的list进行赋值，主要通过AssetManager的list方法，得到assets目录下的.mp3文件，之后通过字符串操作，初始化所有音乐的歌手歌曲名，这个list也用于RecyclerView的绑定，使用这种方法可以省去静态的list内容，根据assets目录动态的初始化RecyclerView，并进行播放

**MainActivity中注册的onClick方法**

```java
//在此之前先实现OnClickListener接口，并对按钮控件进行注册  
public void onClick(View v) {
        Intent intent = new Intent(CONTROL_ACTION);
        int code = -1;
        switch (v.getId()){
            case R.id.play:
                code = 1;
                break;
            case R.id.next:
                code = 2;
                break;
            case R.id.previous:
                code = 3;
                break;
        }
        intent.putExtra("code",code);
        sendBroadcast(intent);
    }
```

这个方法广播一个code，发送给Service，通过按钮的点击事件控制歌曲的播放，以及上一首下一首的切换

**Adapter中注册的onClick方法**

```java
   class ViewHolder extends RecyclerView.ViewHolder{

        TextView title;
        TextView author;

        public ViewHolder(@NonNull View itemView) {
            super(itemView);
            author = itemView.findViewById(R.id.author);
            title = itemView.findViewById(R.id.title);
            itemView.setOnClickListener(v -> {
                Intent intent = new Intent();
                intent.setAction(MainActivity.CHANGE_ACTION);
                intent.putExtra("current",this.getAdapterPosition());
                intent.putExtra("code",1);
                itemView.getContext().sendBroadcast(intent);
            });
        }
    }
```

这个onClick方法注册在Adapter的ViewHolder中，对RecyclerView中每一个itemView都注册了相应的点击方法，这个方法传递了一个current（点击歌曲的location），一个code（1控制直接播放）

**Service中的Receiver**

```java
//onCreate中注册接收器
{
   		IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(MainActivity.CONTROL_ACTION);
        intentFilter.addAction(MainActivity.CHANGE_ACTION);

        musicServiceReceiver = new MusicServiceReceiver();

        registerReceiver(musicServiceReceiver,intentFilter);
}

class MusicServiceReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context, Intent intent) {
            int receiveCurrent = intent.getIntExtra("current",-1);
            int code = intent.getIntExtra("code",-1);
            if(receiveCurrent != -1){
                current = receiveCurrent;
                prepareAndPlay(list.get(current).getFileName());
            }

            switch (code){
                case 1:
                    if(status == Status.PLAYING){
                        mediaPlayer.pause();
                        status = Status.PAUSING;
                    }else if(status == Status.PAUSING){
                        mediaPlayer.start();
                        status = Status.PLAYING;
                    }else{
                        prepareAndPlay(list.get(0).getFileName());
                        current = 0;
                        status = Status.PLAYING;
                    }
                    break;
                case 2:
                    if(status == Status.NOT_INITIALIZED){
                        Toast.makeText(getBaseContext(),"请先开始播放!",Toast.LENGTH_SHORT);
                    }
                    else{
                        current++;
                        if (current >= list.size()){
                            current = 0;
                        }
                        prepareAndPlay(list.get(current).getFileName());
                        status = Status.PLAYING;
                    }
                    break;
                case 3:
                    if(status == Status.NOT_INITIALIZED){
                        Toast.makeText(getBaseContext(),"请先开始播放!",Toast.LENGTH_SHORT);
                    }
                    else {
                        current--;
                        if (current < 0){
                            current = list.size() - 1;
                        }
                        prepareAndPlay(list.get(current).getFileName());
                        status = Status.PLAYING;
                    }
                    break;

            }
            Intent sendIntent = new Intent(MainActivity.UPDATE_ACTION);
            sendIntent.putExtra("update", status.getIndex());
            sendIntent.putExtra("current", current);
            sendBroadcast(sendIntent);
        }
    }

```

这个方法接收了Adapter与MainActivity中发送的广播，并对应不同的extra内容进行了不同的操作，其中，switch-case代码段控制音乐的开始暂停，上一首与下一首，receiveCurrent是点击切换的歌曲位置。最后向MainActivity发送广播

**MainActivity中的Receiver**

```java
//onCreate中注册接收器
{
     	IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(UPDATE_ACTION);

        mainBroadcastReceiver = new MainBroadcastReceiver();

        registerReceiver(mainBroadcastReceiver,intentFilter);
}

class MainBroadcastReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context, Intent intent) {
            int update = intent.getIntExtra("update", -1);
            int current = intent.getIntExtra("current", -1);
            if (current >= 0)
            {
                title.setText(list.get(current).getTitle());
                author.setText(list.get(current).getAuthor());
            }
            switch (update){
                case 1:
                    status = Status.PLAYING;
                    play.setImageResource(R.drawable.pause);
                    break;
                case 2:
                    status = Status.PAUSING;
                    play.setImageResource(R.drawable.play);
                    break;
            }
        }
  }
```

MainActivity中的接收器，改变当前播放曲目的名字与歌手，同时改变播放按钮是开始还是暂停，并记录播放状态status

**MusicService中的其他功能代码段**

```java
//播放方法，传入音乐文件名，播放对应音乐   
private void prepareAndPlay(String music)
    {
        try
        {
            AssetFileDescriptor afd = assetManager.openFd(music);
            mediaPlayer.reset();
            mediaPlayer.setDataSource(afd.getFileDescriptor(),
                    afd.getStartOffset(), afd.getLength());
            mediaPlayer.prepare();
            mediaPlayer.start();
            status = Status.PLAYING;
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
    }

//播放完成后自动切换
{
    mediaPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
            @Override
            public void onCompletion(MediaPlayer mp) {
                current++;
                if (current >= list.size()){
                    current = 0;
                }
                Intent sendIntent = new Intent(MainActivity.UPDATE_ACTION);
                sendIntent.putExtra("current", current);
                sendBroadcast(sendIntent);
                prepareAndPlay(list.get(current).getFileName());
            }
        });
}
```



**除此之外，Layout部分与其他未提到的代码部分可以参考源码**

**项目源码：https://github.com/AriaKanade/Android**