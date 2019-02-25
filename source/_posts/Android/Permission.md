---
title: Permission
categories:
  - Android
  - 新特性
comments: true
date: 2017-12-29 10:47:24
updated: 2017-12-29 10:47:24
tags:
keywords:
description:
---

# 6.0 动态权限申请，自己封装使用的一个 Activity

<!-- more -->


### 配置方式
- Activity 设置透明背景
- `android:theme="@android:style/Theme.Translucent.NoTitleBar"`
- 暂时仅设置每次申请一种权限

### 使用方式
```java
    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.M) {
            PermissionActivity.startNewActivity(context, Manifest.permission.WRITE_EXTERNAL_STORAGE);
    }
```

### Activity 源码
```java

public class PermissionActivity extends Activity {

    public static final String PERMISSION = "PERMISSION";

    private static final int REQUEST_Code = 2017;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.layout_permission);
        isImmersive = false;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            setTranslucentStatus(true);
        }
        //为状态栏着色
        SystemBarTintManager tintManager = new SystemBarTintManager(this);
        tintManager.setStatusBarTintEnabled(true);
        tintManager.setStatusBarTintColor(Color.TRANSPARENT);

        KLog.e(Build.FINGERPRINT);
        findViewById(R.id.background).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                finish();
            }
        });
        String mPermission = getIntent().getStringExtra(PERMISSION);
        if (!TextUtil.isValidate(mPermission))
            return;
        checkPermission(mPermission, REQUEST_Code);
    }

    private void checkPermission(String permission, int requestCode) {
        switch (PermissionChecker.checkSelfPermission(PermissionActivity.this, permission)) {
            case PermissionChecker.PERMISSION_GRANTED:
                KLog.e(permission + " 已申请");

                switch (permission) {
                    case Manifest.permission.ACCESS_COARSE_LOCATION:
                        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {//这个需要版本号大于19
                            AppOpsManager appOpsManager = (AppOpsManager) getSystemService(Context.APP_OPS_SERVICE);
                            int checkOp = appOpsManager.checkOp(AppOpsManager.OPSTR_COARSE_LOCATION, android.os.Process.myUid(), getPackageName());

                            checkAppOpsManager(permission, checkOp);
                        }
                        break;
                    case Manifest.permission.ACCESS_FINE_LOCATION:
                        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {//这个需要版本号大于19
                            AppOpsManager appOpsManager = (AppOpsManager) getSystemService(Context.APP_OPS_SERVICE);
                            int checkOp = appOpsManager.checkOp(AppOpsManager.OPSTR_FINE_LOCATION, android.os.Process.myUid(), getPackageName());

                            checkAppOpsManager(permission, checkOp);
                        }
                        break;
                    case Manifest.permission.READ_CONTACTS:
                        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {//这个需要版本号大于19

                            AppOpsManager appOpsManager = (AppOpsManager) getSystemService(Context.APP_OPS_SERVICE);

                            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                                int checkOp = appOpsManager.checkOp(AppOpsManager.OPSTR_READ_CONTACTS, android.os.Process.myUid(), getPackageName());

                                checkAppOpsManager(permission, checkOp);
                            }
                        }
                        break;
                    default:
                        break;

                }
                checkAppOpsManagerCompat(permission);

                finish();
//            Intent intent = new Intent("android.content.pm.action.REQUEST_PERMISSIONS");
//            intent.putExtra("android.content.pm.extra.REQUEST_PERMISSIONS_NAMES", permission);
//            startActivity(intent);
                break;
            case PermissionChecker.PERMISSION_DENIED:
                checkShowRequestPermissionRationale("未申请", permission, requestCode);
                break;
            case PermissionChecker.PERMISSION_DENIED_APP_OP:
                checkShowRequestPermissionRationale("APP_OP未申请", permission, requestCode);
                break;
            default:
                KLog.e(PermissionChecker.checkSelfPermission(PermissionActivity.this, permission));

                checkShowRequestPermissionRationale("error", permission, requestCode);

                ActivityCompat.requestPermissions(PermissionActivity.this, new String[]{permission}, requestCode);
        }

    }

    void checkShowRequestPermissionRationale(String starte, final String permission, final int requestCode) {
        KLog.e(permission + ":" + starte);
        if (shouldShowRequestPermissionRationale(PermissionActivity.this, permission)) {
            KLog.e(starte + ":自定义弹窗");
            Dialogbox_permission.newInstance(PermissionActivity.this)
                    .setTitle("获取权限")
                    .setContent(getString(permission))
                    .setCancel("残忍拒绝")
                    .setOk("重新授权")
                    .setOnCallback(
                            new Dialogbox_permission.OnCallback() {
                                @Override
                                public void callback(Dialogbox_permission.DialogObject dialogObject) {
                                    ActivityCompat.requestPermissions(PermissionActivity.this, new String[]{permission}, requestCode);
                                    dialogObject.dialog.cancel();
                                }
                            }
                    )
                    .setCancelOnCallback(new Dialogbox_permission.OnCallback() {
                        @Override
                        public void callback(Dialogbox_permission.DialogObject dialogObject) {
                            finish();
                        }
                    }).show();
        } else {
            KLog.e(starte + ":开始请求");
            ActivityCompat.requestPermissions(PermissionActivity.this, new String[]{permission}, requestCode);
        }

        checkAppOpsManagerCompat(permission);
    }


    void checkAppOpsManager(String permission, int checkOp) {
        switch (checkOp) {
            case AppOpsManager.MODE_ALLOWED:
                KLog.e(permission + " MODE_ALLOWED");
                break;
            case AppOpsManager.MODE_DEFAULT:
                KLog.e(permission + " MODE_DEFAULT");
                break;
            case AppOpsManager.MODE_ERRORED:
                KLog.e(permission + " MODE_ERRORED");
                break;
            case AppOpsManager.MODE_IGNORED:
                KLog.e(permission + " MODE_IGNORED");
                break;
            default:
                KLog.e(permission + " MODE_DEFAULT: " + checkOp);
        }
    }

    void checkAppOpsManagerCompat(String permission) {
        String permissionToOp = AppOpsManagerCompat.permissionToOp(permission);
        if (permissionToOp == null) {
            KLog.e(permission + " permissionToOp 为 null 仍需要授权？需要 SDK >= 23");
        } else {
            KLog.e(permission + " permissionToOp 为 " + permissionToOp);
            int noteOp = AppOpsManagerCompat.noteOp(PermissionActivity.this, permissionToOp, android.os.Process.myUid(), getBaseContext().getPackageName());
            switch (noteOp) {
                case AppOpsManagerCompat.MODE_ALLOWED:
                    KLog.e(permission + " AppOpsManagerCompat.MODE_ALLOWED");
                    KLog.e("-----------------------");
                    break;
                case AppOpsManagerCompat.MODE_DEFAULT:
                    KLog.e(permission + " AppOpsManagerCompat.MODE_DEFAULT");
                    KLog.e("xxxxxxxxxxxxxxxxxxxxxxx");
                    break;
                case AppOpsManagerCompat.MODE_IGNORED:
                    KLog.e("iiiiiiiiiiiiiiiiiiiiiii");
                    KLog.e(permission + " AppOpsManagerCompat.MODE_IGNORED");//小米在系统取消授权后，会调到这里，请求的回调都是成功，然而事实不是，考虑跳通知界面
                    break;
                default:
                    KLog.e(permission + " AppOpsManagerCompat.MODE " + noteOp);
            }
        }
    }

    @Override
    public void onRequestPermissionsResult(final int requestCode, @NonNull final String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        if (permissions == null || permissions.length < 1) {
            return;
        }

        KLog.e("onRequestPermissionsResult:" + permissions[0]);

        switch (requestCode) {
            case REQUEST_Code:
                if (verifyPermissions(grantResults)) {
                    KLog.e(Arrays.toString(permissions) + " 申请成功");
                    finish();

                } else {
                    if (PermissionActivity.shouldShowRequestPermissionRationale(PermissionActivity.this, permissions)) {

                        Dialogbox_permission.newInstance(PermissionActivity.this)
                                .setTitle("获取权限")
                                .setContent(getString(permissions[0]))
                                .setCancel("残忍拒绝")
                                .setOk("重新授权")
                                .setOnCallback(new Dialogbox_permission.OnCallback() {
                                    @Override
                                    public void callback(Dialogbox_permission.DialogObject dialogObject) {
                                        ActivityCompat.requestPermissions(PermissionActivity.this, permissions, requestCode);
                                        dialogObject.dialog.cancel();
                                    }
                                })
                                .setCancelOnCallback(new Dialogbox_permission.OnCallback() {
                                    @Override
                                    public void callback(Dialogbox_permission.DialogObject dialogObject) {
                                        finish();
                                    }
                                }).show();

                    } else {

                        Dialogbox_permission.newInstance(PermissionActivity.this)
                                .setTitle("获取权限")
                                .setContent(getString2(permissions[0]))
                                .setCancel("我已了解")
                                .setOk("前往设置")
                                .setOnCallback(new Dialogbox_permission.OnCallback() {
                                    @Override
                                    public void callback(Dialogbox_permission.DialogObject dialogObject) {
                                        startActivity(PermissionsPageManager.getIntent(PermissionActivity.this));
                                        dialogObject.dialog.cancel();
                                    }
                                })
                                .setCancelOnCallback(new Dialogbox_permission.OnCallback() {
                                    @Override
                                    public void callback(Dialogbox_permission.DialogObject dialogObject) {
                                        finish();
                                    }
                                })
                                .show();

                    }
                }
                break;
            default:
                break;
        }

    }


    public static boolean verifyPermissions(int... grantResults) {
        if (grantResults.length == 0) {
            return false;
        }
        for (int result : grantResults) {
            if (result != PackageManager.PERMISSION_GRANTED) {
                return false;
            }
        }
        return true;
    }

    /**
     * Checks given permissions are needed to show rationale.
     *
     * @param activity    activity
     * @param permissions permission list
     * @return returns true if one of the permission is needed to show rationale.
     */
    public static boolean shouldShowRequestPermissionRationale(Activity activity, String... permissions) {
        for (String permission : permissions) {
            if (ActivityCompat.shouldShowRequestPermissionRationale(activity, permission)) {
                return true;
            }
        }
        return false;
    }

    /**
     * Checks given permissions are needed to show rationale.
     *
     * @param fragment    fragment
     * @param permissions permission list
     * @return returns true if one of the permission is needed to show rationale.
     */
    public static boolean shouldShowRequestPermissionRationale(android.support.v4.app.Fragment fragment, String... permissions) {
        for (String permission : permissions) {
            if (fragment.shouldShowRequestPermissionRationale(permission)) {
                return true;
            }
        }
        return false;
    }

    private Spanned getString(String permission) {
        if (!TextUtil.isValidate(permission)) {
            return Html.fromHtml("完善App功能需要该权限");
        }
        switch (permission) {
            case Manifest.permission.ACCESS_COARSE_LOCATION:
            case Manifest.permission.ACCESS_FINE_LOCATION:
                return Html.fromHtml("我们需要获取<font color='#0378d8'>位置信息</font>权限，以便为您提供相应服务。否则您将无法正常使用该功能");
            case Manifest.permission.CAMERA:
                return Html.fromHtml("我们需要获取<font color='#0378d8'>相机</font>权限，以便为您提供相应服务。否则您将无法正常使用该功能");
            case Manifest.permission.WRITE_EXTERNAL_STORAGE:
                return Html.fromHtml("我们需要获取<font color='#0378d8'>SD卡</font>权限，以便为您提供相应服务。否则您将无法正常更新或使用XXX应用");
            case Manifest.permission.READ_PHONE_STATE:
                return Html.fromHtml("我们需要获取<font color='#0378d8'>读取设备信息</font>权限，以便为您提供相应服务。否则您将无法正常更新或使用XXX应用");
            case Manifest.permission.READ_CONTACTS:
                return Html.fromHtml("我们需要获取<font color='#0378d8'>读取联系人</font>权限，以便为您提供相应服务。否则您将无法正常使用该功能");
            case Manifest.permission.RECORD_AUDIO:
                return Html.fromHtml("我们需要获取<font color='#0378d8'>麦克风</font>权限，以便为您提供相应服务。否则您将无法正常使用该功能");
            default:
                return Html.fromHtml("完善App功能需要该权限");
        }
    }

    private Spanned getString2(String permission) {
        if (!TextUtil.isValidate(permission)) {
            return Html.fromHtml("权限缺少将对体验造成较大影响");
        }
        switch (permission) {
            case Manifest.permission.ACCESS_COARSE_LOCATION:
            case Manifest.permission.ACCESS_FINE_LOCATION:
                return Html.fromHtml("由于无法获取<font color='#0378d8'>位置信息</font>权限，请手动开启该该权限，以便获取更好的体验<br><br>设置路径：系统设置->XXX应用->权限");
            case Manifest.permission.CAMERA:
                return Html.fromHtml("由于无法获取<font color='#0378d8'>相机</font>权限，请手动开启该该权限，以便获取更好的体验<br><br>设置路径：系统设置->XXX应用->权限");
            case Manifest.permission.WRITE_EXTERNAL_STORAGE:
                return Html.fromHtml("由于无法获取<font color='#0378d8'>SD卡</font>权限，请手动开启该该权限，以便获取更好的体验<br><br>设置路径：系统设置->XXX应用->权限");
            case Manifest.permission.READ_PHONE_STATE:
                return Html.fromHtml("由于无法获取<font color='#0378d8'>读取设备信息</font>权限，请手动开启该该权限，以便获取更好的体验<br><br>设置路径：系统设置->XXX应用->权限");
            case Manifest.permission.READ_CONTACTS:
                return Html.fromHtml("由于无法获取<font color='#0378d8'>读取联系人</font>权限，请手动开启该该权限，以便获取更好的体验<br><br>设置路径：系统设置->XXX应用->权限");
            case Manifest.permission.RECORD_AUDIO:
                return Html.fromHtml("由于无法获取<font color='#0378d8'>麦克风</font>权限，请手动开启该该权限，以便获取更好的体验<br><br>设置路径：系统设置->XXX应用->权限");
            default:
                return Html.fromHtml("权限缺少将对体验造成较大影响");
        }
    }

    public static void startNewActivity(Context context, String permission) {
        Intent intent = new Intent(context, PermissionActivity.class);
        intent.putExtra(PERMISSION, permission);
        context.startActivity(intent);
    }
}

```


