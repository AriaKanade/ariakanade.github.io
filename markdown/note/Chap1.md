# 第一节——基础UI部分

**项目源码：https://github.com/AriaKanade/Android**

**官方文档：https://developer.android.google.cn/docs?hl=zh_cn**

[TOC]

## Activity的初步使用

- #### 常用的布局模式

  - LinearLayout

  使用线性布局方式，可以根据需求确定布局的排布方式（Orientation）：垂直布局（Vertical），水平布局（Horizontal）

  - ConstraintLayout

  使用约束布局方式，可以根据不同的控件确定对应的约束条件，做到不需要嵌套布局而将控件的相应大小与位置进行约束，主要使用约束工具（Magic Stick）来控制对应的约束条件。

- #### 向Empty Activity中添加控件

  - 首先添加一个空Activity，框选生成对应的
  - 在res/layout文件夹下找到新创建的Activity对应的xml文件
  - 打开设计（Design）模式，使用工具板（Palatte）拖取控件到窗口中，默认创建的布局模式是ConstraintLayout，可以打开编码（Code）模式，将标签替换成LinearLayout，就创建了一个线性布局对应的Activity**（*不一定使用拖取方式创建控件，可以通过编码的方式手写控件，个人习惯是拖取创建控件，编码编写控件属性*）**
  - 编写控件属性，两种方式：编码方式，或者使用右侧的属性（Attributes）栏，选择需要编辑或者添加的属性进行编辑

- #### 添加后台控制代码

  - Empty Activity默认生成的代码段如下，可以看出每个创建的Activity都是继承自AppCompatActivity，且必须重写其中的onCreate方法，这个方法就是Activity被创建时会调用的方法

    ```java
    package com.example.demo1;
    
    import androidx.appcompat.app.AppCompatActivity;
    
    import android.os.Bundle;
    
    public class MainActivity extends AppCompatActivity {
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main2);
        }
    }
    ```

    其中setContentView方法的参数为layout中对应的xml的id，最勇士绑定后台代码段与前台的xml文件，可以通过修改其中的参数改变所要显示的不同的Layout

  - 获取相应控件id，控件在创建时有一个**非常重要的属性——id**，每个控件的id是全局唯一的，因此在创建控件时我们需要手动指定一个唯一的id，命名需要具有一定的可读性和可理解性，如创建一个登录按钮：

    ```xml
        <Button
            android:id="@+id/LoginBtn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Button" />
    ```

    之后，我们可以通过**R文件**在后台得到对应的控件id，通过**findViewById静态方法**获取控件

    ```java
    Button b = findViewById(R.id.LoginBtn);
    ```

    - **关于R文件**，官方文档的说明如下：

      `编译应用时，`aapt` 会生成 `R` 类，其中包含 `res/` 目录中所有资源的资源 ID。每个资源类型都有对应的 `R` 子类（例如，`R.drawable` 对应所有可绘制对象资源），而该类型的每个资源都有对应的静态整型数（例如，`R.drawable.icon`）。该整型数就是可用来检索资源的资源 ID。`

  - 获取了控件id之后，编写对控件的控制代码，如控件的点击事件，TextView的内容绑定等等

    - click的编写方式：实现一个OnClickListener的接口，为相应的控件注册对应的click事件，再编写OnClick方法处理点击事件，代码示例如下

      ```java
      public class MainActivity extends AppCompatActivity implements View.OnClickListener {
      
      
          LinearLayout Mes;
          LinearLayout Fra;
          LinearLayout Setting;
          LinearLayout Address;
      
      
      
          @Override
          protected void onCreate(Bundle savedInstanceState) {
       		//...
              Mes.setOnClickListener(this);
              Fra.setOnClickListener(this);
              Setting.setOnClickListener(this);
              Address.setOnClickListener(this);
              //...
          }
          
          public void onClick(View v){
              //编写点击事件...
          }
      ```

      

  

## LinearLayout的布局

- #### 常用的基础属性

  ```xml
  <!--
  	LinearLayout的排布方式
  	horizontal:水平布局（默认）
  	vertical:垂直布局
  -->
  android:orientation="horizontal/vertical"
  <!--
  	Layout或者普通控件的高与宽
  	match_parent:父布局决定控件大小
  	wrap_content:由控件内容决定大小
  	xxdp:xx是数字，定宽或者定高
  -->
  android:layout_width="match_parent/wrap_content/xxdp"
  android:layout_height="match_parent/wrap_content/xxdp"
  
  ```

  此外还有background，textSize等等基础属性，需要使用时可以参考官方文档，不一一叙述

- #### Layout嵌套

  ![image-20210409112409337](..\..\image\image-20210409112409337.png) 

  这是项目中toolbar的内容，具体效果如下：

  ![image-20210409113129948](..\..\image\image-20210409113129948.png) 

  在这个嵌套中有非常重要的两个属性，分别是控件的gravity属性与Layout的weight属性

  ```xml
  android:gravity="center" //内容对齐处在中心，另外有center_vertical/center_horizental等值可以取
  
  android:layout_weight="1" //在同一个Layout下，决定不同区域所占该Layout的比值，数字越小占比越高
  
  ```

  这个切换栏最外层是一个水平线性布局，里面嵌套了四个垂直线性布局，在这之中就使用了这个两个属性完成了上图的效果

- #### 使用include标签完成类似组件化引入的效果

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
  
      <include layout="@layout/top"></include>
  
      <FrameLayout
          android:id="@+id/MainFragment"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:layout_weight="1"/>
  
      <include layout="@layout/toolbar"></include>
  
  </LinearLayout>
  ```

  代码段中的@layout/top是标题栏，@layout/toolbar则是刚才的切换栏，FrameLayout则是一个容器，获取之后会用到的fragment，整个文件类似于一个组件加载的模式



## Fragment的基础应用

- #### 什么是Fragment

  官方文档定义如下：` Fragment表示应用界面中可重复使用的一部分。Fragment 定义和管理自己的布局，具有自己的生命周期，并且可以处理自己的输入事件。Fragment 不能独立存在，而是必须由 Activity 或另一个 Fragment 托管。Fragment 的视图层次结构会成为宿主的视图层次结构的一部分，或附加到宿主的视图层次结构`

  简而言之，Fragment（碎片）是在应用中可以重复使用的，类似于子Activity，有自己的布局，生命周期的一类组件

- #### 使用Fragment

  - 创建一个Fragment，勾选创建对应的Layout
  - 添加控件等内容，编辑控件属性
  - 编写后台控制代码
  - 在需要使用Fragment的Activity中添加Fragment

  总体而言，Fragment的Layout内容与Activity非常相似，但控制代码不一样，内容如下

  ```java
  package com.example.myapplication;
  
  import android.os.Bundle;
  
  import androidx.fragment.app.Fragment;
  
  import android.view.LayoutInflater;
  import android.view.View;
  import android.view.ViewGroup;
  
  public class FraFragment extends Fragment {
  
      @Override
      public View onCreateView(LayoutInflater inflater, ViewGroup container,
                               Bundle savedInstanceState) {
          // Inflate the layout for this fragment
          return inflater.inflate(R.layout.fragment_fra, container, false);
      }
  }
  ```

  可以看到，创建的Fragment继承了Fragment类，需要重写onCreateView这个方法

  inflate方法通过获取container（fragment的容器），inflater（需要展现的内容）获得fragment渲染后产生的View，并把它放到容器中

- #### 使用FragmentTransaction与FragmentManager管理Fragment

  - 这个demo的MainActivity中有一个FrameLayout，相当于一个Fragment的容器，而使用FragmentTransaction与FragmentManager的目的就是管理这个容器中不同的Fragment的切换效果

  - 获得FragmentTransaction与FragmentManager实例的方法

    ```java
    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();
    ```

    在这之后我们就可以使用这两个实例对Fragment进行操作

  - 切换Fragment的方法：

    ```java
       public void replaceFragment(Fragment fragment){
       		FragmentManager fm = getSupportFragmentManager();
    		FragmentTransaction ft = fm.beginTransaction();
            ft.replace(R.id.MainFragment,fragment).commit();
        }
    ```

    FragmentTransaction下有两个方法可以实现切换效果：add与replace两个方法，两个方法接受的参数相同，第一个是容器的id，第二个是需要切换的fragment，在demo中，这个模块接收需要展现的fragment，把容器中的fragment替换成相应的内容，具体的实现参考源码内容







