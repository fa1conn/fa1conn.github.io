---
layout:     post
title:      "安卓动态加载技术"
subtitle:   "dynamic technology for android"
date:       2019-11-13 12:00:00
author:     "fa1con"
header-img: ""
header-mask: 0.3
catalog:    true
tags:
    - android
    - java
---
# 动态加载技术
## 动态加载apk
java中用继承自ClassLoader的类加载器，然后通过defineClass从二进制流中加载Class

>android中派生出两个类DexClassLoader和PathClassLoader，本质上重载了findClass方法，Dalvik虚拟机识别的是dex文件

DexFile在加载类时，具体是调用成员方法loadClass或者loadClassBinaryName，loadClassBinaryName需要将包含名的类名中的“.”转换为“/”
```java
public Class loadClass(String name, ClassLoader loader) {
        String slashName = name.replace('.', '/');
        return loadClassBinaryName(slashName, loader);
}
```

**这两者的区别在于DexClassLoader需要提供一个可写的outpath路径，用来释放.apk包或者.jar包中的dex文件。换个说法来说，就是PathClassLoader不能主动从zip包中释放出dex，因此只支持直接操作dex格式文件，或者已经安装的apk（因为已经安装的apk在cache中存在缓存的dex文件）。而DexClassLoader可以支持.apk、.jar和.dex文件，并且会在指定的outpath路径释放出dex文件。**


然后看一下实例：
首先要写一个要被加载的apk
定义一个接口
IShowToast.java
```java
package com.dynamic;
import android.content.Context;
public interface IShowToast {
    int showToast(Context context);
}
```
还有一个具体实现的类
ShowToastImpl.java
```java
/**
 * 动态加载测试实现
 */

package com.dynamic;
import android.content.Context;
import android.widget.Toast;
import com.dynamic.IShowToast;
public class ShowToastImpl implements IShowToast {
    @Override
    public int showToast(Context context) {
        Toast.makeText(context, "我来自另一个dex文件", Toast.LENGTH_LONG).show();
        return 100;
    }
}
```
然后在android studio下编译成apk，这里名字为app-debug.apk

然后要写一个宿主apk，用它去加载前面的apk,前面加载的apk的包名最好和宿主apk包名保持一致（不知道会不会出问题）

```java

package com.dynamic;
import android.content.Intent;
import android.content.pm.ActivityInfo;
import android.content.pm.PackageManager;
import android.content.pm.ResolveInfo;
import android.nfc.Tag;
import android.os.Environment;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;
import com.dynamic.IShowToast;
import java.io.File;
import java.io.IOException;
import java.util.List;
import dalvik.system.DexClassLoader;
import dalvik.system.PathClassLoader;
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
    private IShowToast lib;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button showBannerBtn = (Button) findViewById(R.id.button);
        showBannerBtn.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
                loadDexClass();
            }
        });
    }
    /**
     * 加载dex文件中的class，并调用其中的showToast方法
     */
    private void loadDexClass() {
        String dexPath = Environment.getExternalStorageDirectory().toString() + File.separator + "app-debug.apk";
        Log.i(TAG,dexPath);
        File dexOutputDir = getDir("dex", 0);
        DexClassLoader cl = new DexClassLoader(dexPath,dexOutputDir.getAbsolutePath(),null,getClassLoader());
        try {
            //该name就是internalPath路径下的dex文件里面的ShowToastImpl这个类的包名+类名
            Class<?> clz = cl.loadClass("com.dynamic.ShowToastImpl");
            IShowToast impl= (IShowToast) clz.newInstance();//通过该方法得到IShowToast类
            if (impl!=null)
                impl.showToast(this);//调用打开弹窗
        } catch (Exception e) {
            e.printStackTrace();
            Log.i(TAG,"加载app-debug.apk有问题");
        }
    }
}
```
**最后是权限问题，非常重要**：
AndroidManifest.xml里添加
```xml
<!--往SDCard写入数据权限-->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<!--从SDCard读取数据权限-->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

然后就是app-debug.apk需要adb push到模拟器的`/storage/emulated/0/`中
[demo链接](https://github.com/fa1conn/Dynamic-Load-Apk-for-Android)


[https://blog.csdn.net/jiangwei0910410003/article/details/17679823](https://blog.csdn.net/jiangwei0910410003/article/details/17679823)


[Android动态加载Dex过程](https://blog.csdn.net/a2923790861/article/details/80539862)
[类加载器用法与不同](https://blog.csdn.net/jiangwei0910410003/article/details/41384667)


## 动态加载资源（图片之类的）
需要解决的问题：将插件apk中的资源添加到宿主apk中，需要采用反射机制
为什么需要采用反射机制？
调用AssetManager中的addAssetPath方法，我们可以将一个apk中的资源加载到Resources中，由于addAssetPath是隐藏api无法直接调用，所以需要采用反射,所以将apk路径传递给assetManager，资源就加载到AssetManager中，然后再通过AssetManager创建一个新的Resources对象，这个对象就是我们可以使用的apk中的资源
```java

    public static Resources getSkinResource(Context context, String skinFilePath) {
        if (mSkinResource != null) {
            return mSkinResource;
        }
        try {
            AssetManager assetManager = AssetManager.class.newInstance();
            Method addAssetPath = assetManager.getClass().getMethod("addAssetPath", String.class);
            addAssetPath.setAccessible(true);
            addAssetPath.invoke(assetManager, skinFilePath);
            Resources superRes = context.getResources();
            mSkinResource = new Resources(assetManager, superRes.getDisplayMetrics(), superRes.getConfiguration());
            return mSkinResource;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
```
然后获取apk包名，这里自己手动写包名也可以
```java
    public static String getPackageName(Context context, String apkFilePath) {
        PackageManager pm = context.getPackageManager();
        PackageInfo info = pm.getPackageArchiveInfo(apkFilePath,
                PackageManager.GET_ACTIVITIES);
        return info.packageName;
    }

```
然后是获取资源id,`context.getResources()`是当前apk的资源对象，通过getResourceEntryName(orginalResourceId)的方法拿到对应资源的id（**注意：加载资源的名称和宿主app的名称保持一致，不然会无法编译app，名字不一样的情况暂时不知道怎么处理**），type要和加载资源类型保持一致，比如drawable目录下的type是SkinManager.DRAWABLE，mipmap目录下的type是SkinManager.MIPMAP
```java

    /**
     * 根据原项目中的图片id获取皮肤文件中的图片id
     *
     * @param context
     * @param skinResource
     * @param orginalResourceId
     * @param skinPackageName   皮肤包名
     * @param type              资源类型(mipmap,drawable,string,color)
     * @return
     */
    public static int getSkinResorceId(Context context, Resources skinResource, int orginalResourceId, String skinPackageName, String type) {
        //根据原资源文件id获取资源文件对应的名称
        String resourceEntryName = context.getResources().getResourceEntryName(orginalResourceId);
        Log.e("TEST", "entryName " + resourceEntryName);
        return skinResource.getIdentifier(resourceEntryName, type, skinPackageName);
    }
```
**切记，授予app读写sd卡权限，不然获取包名的时候会返回空指针**
AndroidManifest.xml里添加
```xml
<!--往SDCard写入数据权限-->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<!--从SDCard读取数据权限-->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```
[demo链接](https://github.com/fa1conn/Dynamic-Load-Resource-for-Android)

[Android动态加载插件资源](http://ibat.xyz/2017/04/18/Android%E5%8A%A8%E6%80%81%E5%8A%A0%E8%BD%BD%E6%8F%92%E4%BB%B6%E8%B5%84%E6%BA%90/#Demo)
[Android中插件开发篇之----应用换肤原理解析](https://blog.csdn.net/jiangwei0910410003/article/details/47679843)