# 第二节——RecyclerView使用

**项目源码：https://github.com/AriaKanade/Android**

**官方文档：https://developer.android.google.cn/docs?hl=zh_cn**

- [第二节——RecyclerView使用](#第二节recyclerview使用)
  - [列表展示控件——RecyclerView](#列表展示控件recyclerview)
      - [RecyclerView控件编写流程](#recyclerview控件编写流程)
  - [创建JavaBean](#创建javabean)
  - [Item样式](#item样式)
  - [Adapter的编写](#adapter的编写)
  - [后台控制代码编写](#后台控制代码编写)
  - [其他RecyclerView进阶内容](#其他recyclerview进阶内容)

## 列表展示控件——RecyclerView

**RecyclerView**是android开发中的列表展示控件，它作为ListView（Legacy）的升级替代，功能更为强大，可以支持下拉刷新，循环查看列表等功能

每一个RecyclerView的UI都能分成两个部分，RecyclerView的整体展现部分，以及列表中显示的单个内容（item）的布局模式，而后者则是一个单独的xml文件，可以进行自定义布局，前者是一个控件，而且可自定义性很高

#### RecyclerView控件编写流程

- 创建每个item中的JavaBean，用来保存数据模型

- 编写UI部分，调整Item的样式

- 编写Adapter，绑定数据与控件

- 编写相对应的后台控制代码，决定布局模式，填充数据内容

  



## 创建JavaBean

**JavaBean**是java中一种特殊的类，类必须是具体的和公共的，并且具有无参数的构造器，JavaBean 通过提供符合一致性设计模式的公共方法将内部域暴露成员属性，set和get方法获取。这是java中的一种设计规范，并常用于各种数据模型的编写

一个JavaBean的范例如下：

```java
package com.example.myapplication;

public class Userdata {

    private int imageId;

    private String userName;

    private String message;

    public int getImageId() {
        return imageId;
    }

    public void setImageId(int imageId) {
        this.imageId = imageId;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}

//其中无参构造函数省略
```

创建JavaBean的目的是定义即将使用的RecyclerView中，每一个View对应的数据模型。例子中的数据模型为微信聊天窗口中的内容——imageId（头像）、userName（用户名），message（最近消息）





## Item样式

- #### 首先创建一个RecyclerView，并且生成对应的xml

- #### 编写对应的布局，创建控件并修改UI

这个Item的样式是一个模板，之后向RecyclerView中添加数据时，就是使用这个模板，并且向其中填充数据





## Adapter的编写

adapter顾名思义，是一个适配器，它是RecyclerView的编写过程中最重要的一个部分，它将数据与布局中的UI控件联系起来，进行绑定，同时RecyclerView的adapter继承自一个**RecyclerView.Adapter<>**的泛型，其中的限定类型为XXX.ViewHolder（XXX是当前类）

```java
public class UserAdapter extends RecyclerView.Adapter<UserAdapter.ViewHolder> {
    
}
```

其中需要编写一个静态内部类，重写三个方法

- #### 静态类ViewHolder

```java
  static class ViewHolder extends RecyclerView.ViewHolder{

        ImageView imageView;
        TextView textViewName;
        TextView textViewMessage;

        public ViewHolder(@NonNull View itemView) {
            super(itemView);
            imageView = (ImageView)itemView.findViewById(R.id.user_image);
            textViewMessage = (TextView)itemView.findViewById(R.id.text_message);
            textViewName = (TextView)itemView.findViewById(R.id.text_username);
        }
    }
```

这个静态类即泛型中的类，需要继承**RecyclerView.ViewHolder**类，主要作用是静态获取单个itemView中的控件，优化加载时的性能，避免每次刷新或下滑时重新获取控件

- #### onCreateViewHolder方法

```java
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.activity_recycle_view,parent,false);
        ViewHolder viewHolder = new ViewHolder(view);
        return viewHolder;
    }
```

这个方法得到了itemView对应的Layout，并创建了一个对应的ViewHolder

- #### onBindViewHolder方法

```java
@Override 
public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        Userdata userdata = list.get(position);
        holder.imageView.setImageResource(userdata.getImageId());
        holder.textViewMessage.setText(userdata.getMessage());
        holder.textViewName.setText(userdata.getUserName());
    }
```

这个方法使用刚刚创建的ViewHolder，将数据绑定到对应的控件中

- #### getItemCount方法

```java
@Override
    public int getItemCount() {
        return list.size();
    }
```

显而易见，获取item的个数



**此外，我们还需要一个list存放数据内容，一个带参构造器初始化这个Adapter**

```java
  private List<Userdata> list;

    public UserAdapter(List<Userdata> list) {
        this.list = list;
    }
```



## 后台控制代码编写

在项目实例中，我们在Fragment中添加了对应的RecyclerView，模拟出无法点击的微信聊天界面的展示内容，Fragment的源码内容如下

```java
package com.example.myapplication;

import android.os.Bundle;

import androidx.fragment.app.Fragment;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import java.util.ArrayList;
import java.util.List;


public class MesFragment extends Fragment {

    private List<Userdata> list = new ArrayList<>();

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        View view = inflater.inflate(R.layout.fragment_mes, container, false);
        init();
        LinearLayoutManager linearLayoutManager = new LinearLayoutManager(view.getContext());
        RecyclerView recyclerView = view.findViewById(R.id.MesListView);
        recyclerView.setLayoutManager(linearLayoutManager);
        UserAdapter userAdapter = new UserAdapter(list);
        recyclerView.setAdapter(userAdapter);
        return view;
    }


    private void init() {
        if(list.size() != 0){
            list.clear();
        }
        Userdata u1 = new Userdata();
        u1.setImageId(R.drawable.tab_weixin_normal);
        u1.setMessage("你好！");
        u1.setUserName("小明");
        list.add(u1);
        Userdata u2 = new Userdata();
        u2.setImageId(R.drawable.tab_weixin_pressed);
        u2.setMessage("你好！");
        u2.setUserName("小红");
        list.add(u2);
    }
}
```

代码段中添加的list用来存放数据，init方法则是初始化数据，demo中举例随便添加了几条，常见的初始化方式还是需要循环控制，而其中的if判断则是fragment中必须要有的，不然在fragment的切换过程中，每切到当前fragment，init方法便会执行一次，数据就会被重复添加

其中，与Adapter联系最紧密的是这一段代码

```java
        LinearLayoutManager linearLayoutManager = new LinearLayoutManager(view.getContext());
        RecyclerView recyclerView = view.findViewById(R.id.MesListView);
        recyclerView.setLayoutManager(linearLayoutManager);
        UserAdapter userAdapter = new UserAdapter(list);
        recyclerView.setAdapter(userAdapter);
```

LinearLayoutManager、setLayoutManger()的作用是设置RecyclerView的布局方式，不能省略，省略后将无法显示内容，提示无对应的布局方式

setAdapter方法设置RecyclerView对应的adapter，成功完成绑定，最后显示如下图：

![image-20210419113735406](..\..\image\image-20210419113735406.png) 



## 其他RecyclerView进阶内容

**参考博客：https://www.jianshu.com/p/3eb81f50f4db**