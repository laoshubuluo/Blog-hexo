title: Android-Code-获取栈顶的activity
description: Android开发技术点：获取栈顶的activity
date: 2016/05/02 14:12:12 
categories: Android
tags: [Android,Code,Activity]
---
根据业务逻辑要求，需要判断当前的activity是否是之前栈顶的activity，也就是需要判断是否是从后台返回的activity。

<!--more-->

引用代码如下： 

    @Override
    protected void onResume() {
        super.onResume();
        //只有后台还原（还原时栈顶为当前activity），才执行
        if (getTopActivity(this).contains(this.getClass().getSimpleName())) {
        }
    }

判断代码如下： 

    /**
     * 获取栈顶的activity
     */
    String getTopActivity(Activity context) {
        String shortClassName = "";
        ActivityManager manager = (ActivityManager) context.getSystemService(ACTIVITY_SERVICE);
        List<ActivityManager.RunningTaskInfo> runningTaskInfos = manager.getRunningTasks(1);
        if (runningTaskInfos != null) {
            ActivityManager.RunningTaskInfo info = runningTaskInfos.get(0);
            shortClassName = info.topActivity.getShortClassName(); //类名
        }
        return shortClassName;
    }