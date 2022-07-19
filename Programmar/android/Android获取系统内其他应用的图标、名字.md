---
title: Android获取系统内其他应用的图标、名字
date: 2021-04-04 18:33:51.526
updated: 2021-04-04 18:35:49.073
url: http://summersea.top:8090/archives/android-huo-qu-xi-tong-nei-qi-ta-ying-yong-de-tu-biao--ming-zi
categories: 
tags: 安卓 | android
---

最近毕设想实现一个获取其他应用信息的功能，但是翻了很久网上的博客，也没翻出个所以然。
哪天空闲的时候突然灵机一动，想起了v2rayNG的app不是有这个功能吗，直接上github找[v2rayNG](https://github.com/2dust/v2rayNG)的开源代码学习学习。

翻了翻人家的实现，发现这个软件自己做了个获取应用信息的类：
```kotlin
object AppManagerUtil {
    fun loadNetworkAppList(ctx: Context): ArrayList<AppInfo> {
        val packageManager = ctx.packageManager
        val packages = packageManager.getInstalledPackages(PackageManager.GET_PERMISSIONS)
        val apps = ArrayList<AppInfo>()

        for (pkg in packages) {
            if (!pkg.hasInternetPermission && pkg.packageName != "android") continue

            val applicationInfo = pkg.applicationInfo

            val appName = applicationInfo.loadLabel(packageManager).toString()
            val appIcon = applicationInfo.loadIcon(packageManager)
            val isSystemApp = (applicationInfo.flags and ApplicationInfo.FLAG_SYSTEM) > 0

            val appInfo = AppInfo(appName, pkg.packageName, appIcon, isSystemApp, 0)
            apps.add(appInfo)
        }

        return apps
    }

    fun rxLoadNetworkAppList(ctx: Context): Observable<ArrayList<AppInfo>> = Observable.unsafeCreate {
        it.onNext(loadNetworkAppList(ctx))
    }

    val PackageInfo.hasInternetPermission: Boolean
        get() {
            val permissions = requestedPermissions
            return permissions?.any { it == Manifest.permission.INTERNET } ?: false
        }
}
```
<br/>
不过是用kotlin写的，目前我不会这个，得改成java来用。
<br/>再看看权限配置

 	<!-- https://developer.android.com/about/versions/11/privacy/package-visibility -->
    <uses-permission android:name="android.permission.QUERY_ALL_PACKAGES"
        tools:ignore="QueryAllPackagesPermission" />


信息足够，开始借鉴，我需要的只是loadNetwrokApplist的功能，再去掉筛选不使用网络功能app的代码即可，其他的观察者模式设计我不需要。

```java
    /**
     * 获取系统内的app列表
     * @param context 上下文
     * @return app信息列表
     */
    public static ArrayList<AppInfo> getExternalAppList(Context context) {
        PackageManager packageManager = context.getPackageManager();
        List<PackageInfo> installedPackages = packageManager.getInstalledPackages(PackageManager.GET_PERMISSIONS);
        ArrayList<AppInfo> appList = new ArrayList<>();

        for (PackageInfo installedPackage : installedPackages) {
            // 跳过系统应用
            if (installedPackage.packageName.startsWith("com.android")
                || installedPackage.packageName.startsWith("android")) {
                continue;
            }
            ApplicationInfo applicationInfo = installedPackage.applicationInfo;

            // 获取app信息
            String appName = applicationInfo.loadLabel(packageManager).toString();
            Drawable appIcon = applicationInfo.loadIcon(packageManager);
            boolean isSystemApp = (applicationInfo.flags & ApplicationInfo.FLAG_SYSTEM) > 0;

            AppInfo appInfo = new AppInfo(appName, installedPackage.packageName, appIcon, isSystemApp);
            appList.add(appInfo);
        }
        return appList;
    }
```
最终配合其他代码可以看到效果：
![QQ截图20210404183132.jpg](http://summersea.top:8090/upload/2021/04/QQ%E6%88%AA%E5%9B%BE20210404183132-e7442060d18d41e59d993355b5deeaab.jpg)
