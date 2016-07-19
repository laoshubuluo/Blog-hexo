title: Android-Code-Android6.0权限获取和检测
description: Android开发技术点：Android6.0权限检查及动态获取
date: 2016/07/19 16:22:12 
categories: Android
tags: [Android,Code,6.0,Permission]
---
Android6.0开始，Google增强了应用权限方面的控制，除了保留应用安装过程中的权限展示，还增加了应用在运行过程中，对特殊权限的控制，支持用户使用功能时才判断决定是否赋予相应权限。

<!--more-->

# 特别感谢 #
文章参考：
> http://my.oschina.net/u/990728/blog/549914
> http://www.open-open.com/lib/view/open1447236259272.html

如有侵权，请与我联系~我会尽快处理并删除侵权内容。

# 正文 #
动态检查是否存在指定权限

	LogUtil.i("aaaaaaaaaa 权限检查结果：" + selfPermissionGranted());
	private int targetSdkVersion;
	private String permission;

    public boolean selfPermissionGranted() {
        permission = Manifest.permission.INSTALL_SHORTCUT;
        try {
            final PackageInfo info = getPackageManager().getPackageInfo(getPackageName(), 0);
            targetSdkVersion = info.applicationInfo.targetSdkVersion;
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
        }

        // For Android < Android M, self permissions are always granted.
        boolean result = true;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (targetSdkVersion >= Build.VERSION_CODES.M) {
                // targetSdkVersion >= Android M, we can use Context#checkSelfPermission
                result = ContextCompat.checkSelfPermission(getApplicationContext(), permission) == PackageManager.PERMISSION_GRANTED;
            } else {
                // targetSdkVersion < Android M, we have to use PermissionChecker
                result = PermissionChecker.checkSelfPermission(getApplicationContext(), permission) == PermissionChecker.PERMISSION_GRANTED;
            }
        }
        return result;
    }


动态申请指定权限，及回调

	final public int REQUEST_PERMISSION_CODE = 123;
	ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.INSTALL_SHORTCUT}, REQUEST_PERMISSION_CODE);

	@Override
	public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
		        switch (requestCode) {
		            case REQUEST_PERMISSION_CODE:
		                if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
		                    LogUtil.i("aaaaaaaaaa 权限获取成功");
		                } else {
		                    LogUtil.i("aaaaaaaaaa 权限获取失败");
		                }
		            default:
		                super.onRequestPermissionsResult(requestCode, permissions, grantResults);
		        }
		    }

就这么简单= =、
