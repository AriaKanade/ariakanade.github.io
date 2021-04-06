# tab切换效果展示

#### 展示图:

#### ![image-20210406182256261](..\..\image\image-20210406182256261.png)



#### ![image-20210406182405868](..\..\image\image-20210406182405868.png)

#### 功能说明:

整个app由一个toolbar,一个标题栏和一个LayoutFrame容器组成,四个fragment对应四个图标.

#### 功能分析:

**XXXFragment:**

整段代码为自动生成,其他几个fragment与这个相同

```java
package com.example.myapplication;

import android.os.Bundle;

import androidx.fragment.app.Fragment;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;


public class SettingsFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_settings, container, false);
    }
}
```

**MainActivity:**

MainActivity实现了OnClickListener的接口,主要为了实现按钮点击的监听,Fragment之间用FragmentManager与FragmentTransaction控制.

特殊的地方是使用了一个ActiveButtonId的变量存储当前激活的按钮,从而实现更改按钮图标

```java
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentManager;
import androidx.fragment.app.FragmentTransaction;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.FrameLayout;
import android.widget.ImageButton;
import android.widget.LinearLayout;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {


    int ActiveButtonId;

    LinearLayout Mes;
    LinearLayout Fra;
    LinearLayout Setting;
    LinearLayout Address;

    FrameLayout MainFrameLayout;

    FragmentManager fm;
    FragmentTransaction ft;

    Fragment MesFragment = new MesFragment();
    Fragment FraFragment = new FraFragment();
    Fragment AddressFragment = new AddressFragment();
    Fragment SettingFragment = new SettingsFragment();



    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ActiveButtonId = R.id.MesButton;
        Mes = findViewById(R.id.Mes);
        Fra = findViewById(R.id.Fra);
        Setting = findViewById(R.id.Setting);
        Address = findViewById(R.id.Address);


        MainFrameLayout = findViewById(R.id.MainFragment);
        initFragment();

        Mes.setOnClickListener(this);
        Fra.setOnClickListener(this);
        Setting.setOnClickListener(this);
        Address.setOnClickListener(this);
    }


    @Override
    public void onClick(View v) {
        try {
            switch (v.getId()){
                case R.id.Mes:
                    replaceFragment(MesFragment);
                    changeButtonToInactive();
                    changeButtonToActive(R.id.MesButton);
                    ActiveButtonId = R.id.MesButton;
                    break;
                case R.id.Fra:
                    replaceFragment(FraFragment);
                    changeButtonToInactive();
                    changeButtonToActive(R.id.FraButton);
                    ActiveButtonId = R.id.FraButton;
                    break;
                case R.id.Setting:
                    replaceFragment(SettingFragment);
                    changeButtonToInactive();
                    changeButtonToActive(R.id.SettingButton);
                    ActiveButtonId = R.id.SettingButton;
                    break;
                case R.id.Address:
                    replaceFragment(AddressFragment);
                    changeButtonToInactive();
                    changeButtonToActive(R.id.AddressButton);
                    ActiveButtonId = R.id.AddressButton;
                    break;
                default:
                    break;
            }
        }catch (Exception e){
            e.printStackTrace();
        }

    }
    
    
	//更换Fragment的方法
    public void replaceFragment(Fragment fragment){
        fm = getSupportFragmentManager();
        ft= fm.beginTransaction();
        ft.replace(R.id.MainFragment,fragment).commit();
    }

    //更换button图标
    public void changeButtonToActive(int ButtonId){
       ImageButton button =  findViewById(ButtonId);
       switch (ButtonId){
           case R.id.MesButton:
               button.setImageDrawable(getResources().getDrawable(R.drawable.tab_weixin_pressed));
               break;
           case R.id.FraButton:
               button.setImageDrawable(getResources().getDrawable(R.drawable.tab_find_frd_pressed));
               break;
           case R.id.SettingButton:
               button.setImageDrawable(getResources().getDrawable(R.drawable.tab_settings_pressed));
               break;
           case R.id.AddressButton:
               button.setImageDrawable(getResources().getDrawable(R.drawable.tab_address_pressed));
               break;
           default:
               break;
       }

    }

    //将图标更换为未激活
    public void changeButtonToInactive(){
        ImageButton button = findViewById(ActiveButtonId);

        switch (ActiveButtonId){
            case R.id.MesButton:
                button.setImageDrawable(getResources().getDrawable(R.drawable.tab_weixin_normal));
                break;
            case R.id.FraButton:
                button.setImageDrawable(getResources().getDrawable(R.drawable.tab_find_frd_normal));
                break;
            case R.id.SettingButton:
                button.setImageDrawable(getResources().getDrawable(R.drawable.tab_settings_normal));
                break;
            case R.id.AddressButton:
                button.setImageDrawable(getResources().getDrawable(R.drawable.tab_address_normal));
                break;
            default:
                break;
        }

    }

    //初始化界面,默认是微信Fragment
    public void initFragment(){
        replaceFragment(MesFragment);
        changeButtonToActive(R.id.MesButton);
    }
}
```

**Layout部分:**

使用gravity对齐文字,使用weight调整控件之间的大小格式,具体代码可以参考项目源码