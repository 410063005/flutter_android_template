# 这个项目有什么作用

这是 add-to-app 的模板工程，用于已有 Android 项目快速集成 Flutter。这个模板工程包含如下特性：

+ 可一键切换集成方式
+ 支持 armeabi 架构
+ 完全基于官方集成方案

# 背景

[Integrate a Flutter module into your Android project](https://flutter.dev/docs/development/add-to-app/android/project-setup) 提到了如下两种集成方式：

+ Option A - Depend on the Android Archive (AAR)，称之为产物集成
+ Option B - Depend on the module’s source code，称之为源码集成

## 一、兼容产物集成和源码集成

不过实际 Android 项目集成时 Flutter 往往**不是两种集成方式二选一，而是需要同时兼容**。这么做不需要太多理由，只举个简单例子：我们组共8个开发，但只有3人在 Flutter 开发上投入较多精力，另外5人并没有太多时间精力来投入。所以有3个人需要 Option B，因为他们需要开发 Flutter 代码。而另外5个人只需要 Option A，因为他们并不关心 Flutter 代码如何开发和编译，而只关心 Android 项目能否整体编译运行，这种情况下要求这些同事折腾 Flutter 环境显然不合理。

同时兼容 Option A 和 Option B 并不困难，照着 [Integrate a Flutter module into your Android project](https://flutter.dev/docs/development/add-to-app/android/project-setup) 中的指引操作即可。运气好的话，20分钟能搞定。运气不好的话，就难说。所以，准备好一个模板项目，需要时直接 copy 下，省得折腾，岂不更好？

## 二、armeabi 兼容问题

Android 项目集成 Flutter 时一个常见问题是 **armeabi 架构兼容问题**。简单来说，就是 App 运行时找不到 `libflutter.so`。Flutter 官方引擎不提供 armeabi 平台的 `libflutter.so`，因为 Android 官方不再支持该平台。

> 2018年6月发布的 NDK r17 及之后的版本不再支持 ARMv5 (即 armeabi)，否则编译时会提示 ABIs [armeabi] are not supported for platform

![](https://blog-1251688504.cos.ap-shanghai.myqcloud.com/2020/06/29/15933987080765.jpg)

Flutter 拥抱变化是好的，但问题是很多 Android 应用只支持 armeabi 架构。该怎么办？如果你在网上搜相关资料，八成能看到美团的这篇 [Flutter原理与美团的实践](https://juejin.im/post/5b6d59476fb9a04fe91aa778)。它提到的做法是：

> 修改android-arm、android-arm-profile和android-arm-release下的flutter.jar，将其中的lib/armeabi-v7a/libflutter.so移动到lib/armeabi/libflutter.so

```sh
cd $FLUTTER_ROOT/bin/cache/artifacts/engine
for arch in android-arm android-arm-profile android-arm-release; do
  pushd $arch
  cp flutter.jar flutter-armeabi-v7a.jar # 备份
  unzip flutter.jar lib/armeabi-v7a/libflutter.so
  mv lib/armeabi-v7a lib/armeabi
  zip -d flutter.jar lib/armeabi-v7a/libflutter.so
  zip flutter.jar lib/armeabi/libflutter.so
  popd
done
```

如果你按照这种方法来做，绝对是被美团误导了。这个修改 Flutter SDK 的做法非常不好。只写两点理由：

+ 很多时候，修改 Flutter SDK 是不可行的。比如，可能没权限修改CI机器上的 Flutter SDK
+ 即使可以修改 Flutter SDK，较大的开发团队中如何保证修改的一致性也是问题

不必修改 Flutter SDK，只需要简单配置 Android 工程的构建脚本，同样也能解决 armeabi 架构兼容问题。

## 三、完全基于官方集成方案

完全基于 Flutter 官方集成方案很重要。官方提供了一种特别的 profile buildType，用于性能分析。

![](https://blog-1251688504.cos.ap-shanghai.myqcloud.com/2020/06/29/15934000408409.jpg)

非官方的集成方案很容易漏掉上图中的这个 buildType，导致性能分析不便。
