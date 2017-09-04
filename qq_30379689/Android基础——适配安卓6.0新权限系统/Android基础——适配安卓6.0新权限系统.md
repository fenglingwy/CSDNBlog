#适配安卓6.0新权限系统


----------


>本篇文章内容：

>* 安卓6.0新权限系统分类
>* 申请单一或多个权限
>* 解决用户多次拒绝权限或点击不再提示

##**安卓6.0新权限系统分类**

在安卓6.0版本以后，新的权限系统出现了，为了更好的保护用户的安全，新的权限系统需要开发者在代码中手动申请，所以为了适配6.0权限系统，我们不得不学习权限系统

安卓6.0新权限系统分类有两种

1. 普通权限（normal）：这个权限类型并不直接威胁到用户的隐私，可以直接在manifest清单里注册，系统会帮我们默认授权的
2. 危险权限 （dangerous）：这个可以直接给app访问用户一些敏感的数据，不仅需要在manifest清单里注册，同时在使用的时候，需要向系统请求授权

危险权限的特点

* 危险权限是按组分配的，只要同个组的某个权限被同意后，组中的其他权限也会被默认同意

普通权限列表图

![](http://img.blog.csdn.net/20170226120024646?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

危险权限分组图

![](http://img.blog.csdn.net/20170226120052493?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


##**申请单一或多个权限**

申请权限很人性化，类似我们的签证办手续一样，其申请步骤有


1. 申明该权限
2. 检查是否已经有该权限
3. 如果没有则进行申请权限
4. 接收申请成功或者失败回调

① 要使用权限时，别忘了要在manifest中申请

```
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

② 申请单一权限
```
private static final int Location_Permission = 0x01;

private void requestPermission() {
	//1. 检查是否已经有该权限
    if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION)
            != PackageManager.PERMISSION_GRANTED) {   
        //2. 权限没有开启，请求权限
        ActivityCompat.requestPermissions(this,
                new String[]{Manifest.permission.ACCESS_COARSE_LOCATION}, Location_Permission);
    }else{
        //权限已经开启，做相应事情
	}
}

//3. 接收申请成功或者失败回调
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    if (requestCode == Location_Permission) {
        if (grantResults.length>0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            //权限被用户同意,做相应的事情
        } else {
            //权限被用户拒绝，做相应的事情
        }
    }
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
}
```

③ 申请多个权限

如要你要同时申请多个权限，那么可以在requestPermissions传进要申请的权限即可
```
ActivityCompat.requestPermissions(this,new String[]{Manifest.permission.ACCESS_COARSE_LOCATION,
					Manifest.permission.CAMERA}, Location_Permission);
```

④ 判断安卓6.0系统

为了代码的严谨性，在安卓6.0以下我们就不用手动申请了，这里就需要判断一下安卓的版本

```
if (Build.VERSION.SDK_INT >= 23) {
    requestPermission();
}
```


##**解决用户多次拒绝权限或点击不再提示**

很多时候用户不小心点击拒绝，或者害怕手机信息被盗而点拒绝，当第二次进入程序时，我们要进行相对应的处理，这里先看下面这个方法的说明

* shouldShowRequestPermissionRationale()

1. 第一次请求权限时，用户拒绝了，调用shouldShowRequestPermissionRationale()后返回true，应该显示一些为什么需要这个权限的说明
2. 用户在第一次拒绝某个权限后，下次再次申请时，授权的dialog中将会出现“不再提醒”选项，一旦选中勾选了，那么下次申请将不会提示用户
3. 第二次请求权限时，用户拒绝了，并选择了“不在提醒”的选项，调用shouldShowRequestPermissionRationale()后返回false

知道了这个方法的原理后，那么代码就很快就可以写出来了，下面就直接贴上完整代码

```
public class PermissionActivity extends AppCompatActivity {

    private static final int Location_Permission = 0x01;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_permission);

        if (Build.VERSION.SDK_INT >= 23) {
            requestPermission();
        }
    }

    private void requestPermission() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
	        //询问用户是否拒绝过，如果没有，该方法返回false，如果被拒绝过，该方法返回true
            if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.ACCESS_COARSE_LOCATION)) {
                // 用户拒绝过这个权限了，应该提示用户，为什么需要这个权限
                new AlertDialog.Builder(this)
                        .setTitle("友好提醒")
                        .setMessage("没有定位权限将不能搜索附近蓝牙，请把定位权限赐给我吧！")
                        .setPositiveButton("赏你", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                dialog.cancel();
                                // 用户同意继续申请
                            }
                        })
                        .setNegativeButton("不给", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                dialog.cancel();
                                // 用户拒绝申请
                            }
                        }).show();
            } else {
                // 申请授权。
                ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.ACCESS_COARSE_LOCATION}, Location_Permission);
            }
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == Location_Permission) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                //权限被用户同意,做相应的事情
            } else {
                //权限被用户拒绝，做相应事情
            }
        }
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }
}
```

实现效果图

![](http://img.blog.csdn.net/20170226121548219?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

[代码下载](http://download.csdn.net/detail/qq_30379689/9764347)