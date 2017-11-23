<!--
author: 冷火-王胜 
date: 2017-2-20
title: Permission组件解析
tags: Android 组件
category: Android
status: publish 
-->
Android 6.0之后，对于一些敏感的权限，需要我们手动请求，本文介绍一个权限组件Permisssion，主要是用于Android6.0以后（也就是API>23）的敏感权限请求，如果项目中的targetSdkVersion大于等于23，并且某个activity要需要一个或者多个危险权限的话可以用此组件进行判断是否拥有权限以及请求权限。
####组件的使用
1.在module的build.gradle 中增加: compile 'com.neteaseyx.permission:permission-lib:1.0.0'，
2.创建PermissionManager对象，如果是在fragment中使用传入参数是传入 mFragment.getActivity()

     mPermissionManager = new PermissionManager(this);//创建PermissionManager对象
3.明确所需权限，可以是多个权限（哪些是危险权限都可以查得到），这些权限都是用Manifest.permission.xxx声明

     final String[] permissions = {
                Manifest.permission.CAMERA,
                Manifest.permission.WRITE_EXTERNAL_STORAGE,
                Manifest.permission.ACCESS_FINE_LOCATION
        };
4.在需要请求权限的地方使用PermissionManager.requestPermissions 方法请求权限，第一个参数是请求码，第二个参数是要请求的权限，第三个参数是请求结果的回调，PermissionCallback里面有四个回调方法，onGranted_Third是回调所有允许的权限，onExplanation是回调拒绝但是没有勾选不再询问的权限，onNormalDenied_Second是回调所有拒绝的权限，onNoMoreAskDenied_First是回调所有已经拒绝，而且勾选了不再询问的权限

    mPermissionManager.requestPermissions(PERMISSION_REQ_CODE, permissions, new PermissionCallback() {

                    @Override
                    public void onGranted_Third(String... permissions) {    //回调所有允许的权限
                        for (String permission : permissions) {
                            addTextToResult(permission + "\n  -已批准\n", permission);
                        }
                    }

                    @Override
                    public void onExplanation(String... permissions) {  //拒绝但没有勾选不再询问，会通过这个方法回调
                        mPermissionManager.showRequestDialog(MainActivity.this, permissions, PERMISSION_REQ_CODE);
                    }

                    @Override
                    public void onNormalDenied_Second(String... permissions) { //回调所有拒绝的权限
                        for (String permission : permissions) {
                            addTextToResult(permission + "\n  -已拒绝\n", permission);
                        }
                    }

                    @Override
                    public void onNoMoreAskDenied_First(String... permissions) { //拒绝，而且勾选了不再询问
                        for (String permission : permissions) {
                            addTextToResult(permission + "\n  -已永久拒绝\n", permission);
                        }
                    }
                });
5.在 Activity 或者 Fragment 的 onRequestPermissionsResult 方法中调用 PermissionManager.onRequestPermissionResult, 否则不会有任何结果回调

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        mPermissionManager.onRequestPermissionResult(requestCode, permissions, grantResults);
    }
####组件内部实现
让我们看看组件内部是怎么实现的，首先是PermissionManager.requestPermissions 方法请求权限，它先判断是否所有的请求都被批准了，如果都被批准了，直接回调onGranted_Third(permissions),回调所有允许的权限，如果不是，就遍历权限集合，判断是否要用户选择，将需要用户选择的权限添加到permissionsToExplainList集合里，并转换成数组，如果数组不为空那么就弹出相应请求框

注意：这里callback.onExplanation(permissionsToExplain);一般情况是用户上一次选择了拒绝（没有勾选不再询问），这次又重新请求，通常在此处还需要再次弹出请求对话框，而且这里不会自动出现权限请求弹窗, 需要在用户处理完之前提到的 "解释" 之后, 手动通过 PermissionManager.showRequestDialog 显示弹窗


     /**
     * 请求权限
     *
     * @param requestCode 本次请求的唯一标识（不要过大，最好小于999）
     * @param permissions 本次需要请求的权限，可以多个
     * @param callback    本次请求在用户选择完毕后的回调
     */
    public void requestPermissions(int requestCode, String[] permissions, PermissionCallback callback) {
        mCallbackMap.put(requestCode, callback);
        if (!isPermissionsGranted(permissions)) {//是否所有的请求都被批准了，如果没有被批准
            List<String> permissionsToExplainList = new ArrayList<>(3);
            for (String permission : permissions) {
                if (ActivityCompat.shouldShowRequestPermissionRationale(mActivity, permission)) {//判断是否应该请求，里面做了是否是危险权限的判断
                    permissionsToExplainList.add(permission);  //把应该请求的权限添加到permissionsToExplainList集合
                }
            }
            String[] permissionsToExplain = permissionsToExplainList.toArray(new String[permissionsToExplainList.size()]);//集合转化为数组
            if (permissionsToExplain.length != 0) {
                callback.onExplanation(permissionsToExplain);
            } else {
                showRequestDialog(mActivity, permissions, requestCode);
            }
        } else {
            callback.onGranted_Third(permissions);//所有的请求都被批准
        }
    }

在执行完 PermissionManager.requestPermissions 后, 如果有权限请求需要用户选择, 系统会弹出相应请求框
如果没有权限需要用户选择(所有权限或是已经允许, 或是已经拒绝, 或是永久拒绝), 直接回调相应结果
接下来我们来看看回调结果

    * @param requestCode  请求的唯一标识
     * @param permissions  请求的权限
     * @param grantResults 请求的结果
     */
    public void onRequestPermissionResult(int requestCode, String[] permissions, int[] grantResults) {
        PermissionCallback callback = mCallbackMap.get(requestCode);
        if (callback != null) {
            List<String> grantedList = new ArrayList<>(3);//用户同意的权限
            List<String> normalDeniedList = new ArrayList<>(3);//用户拒绝的权限，但是没有勾选不再询问
            List<String> noMoreAskDeniedList = new ArrayList<>(3);//用户拒绝的权限，而且勾选了不再询问
            for (int i = 0; i < permissions.length; i++) {
                if (grantResults[i] == PackageManager.PERMISSION_GRANTED) {
                    grantedList.add(permissions[i]);
                } else {
                    if (!ActivityCompat.shouldShowRequestPermissionRationale(mActivity, permissions[i])) {
                        noMoreAskDeniedList.add(permissions[i]);
                    } else {
                        normalDeniedList.add(permissions[i]);
                    }
                }
            }
            callback.onNoMoreAskDenied_First(noMoreAskDeniedList.toArray(new String[noMoreAskDeniedList.size()]));
            callback.onNormalDenied_Second(normalDeniedList.toArray(new String[normalDeniedList.size()]));
            callback.onGranted_Third(grantedList.toArray(new String[grantedList.size()]));
        }
    }
首先定义了三个数组，以个是表示用户同意的权限，一个是用户拒绝的权限，但是没有勾选不再询问，一个是用户拒绝的权限而且勾选了不再询问，然后开始遍历权限数组，将用户选择之后的权限分成三份，放在对应的结合里，然后调用回调，传到相应的activity中，在activity中的callback里就可以知道哪些权限是用户拒绝或者同意了的，还可以对相应的权限进行操作。

注意：PermissionCallback里的各种结果的回调有个先后顺序:
* onExplanation 如果有权限 "上一次选择了拒绝, 但没有勾选不再询问", 最开始会通过此方法回调
* onNoMoreAskDenied_First 然后回调所有 "不再询问 且 勾选拒绝" 的权限
* onNormalDenied_Second 然后回调所有 "拒绝" 的权限
* onGranted_Third 最后回调所有 "允许" 的权限