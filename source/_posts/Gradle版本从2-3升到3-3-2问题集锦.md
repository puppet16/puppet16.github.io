---
title: Gradle版本从2.3升到3.3.2问题集锦
date: 2020-05-31 21:39:36
tags: Gradle 
summary: Gradle版本从2.3升到3.3.2问题集锦
---

<!-- toc -->

# 一、Gradle版本升级步骤
**1. 在gradle-wrapper.properties文件中修改distributionUrl的版本.**

**2. 在工程里的build.gradle文件中的dependencies项里修改gradle版本，之后点 Try Again.**

# 二、错误集锦

## 1. Cannot set the value of read-only property 'outputFile' for ApkVariantOutputImpl_Decorated
**错误原因：** outputFile变成了可读字段  
**修改方案：**  
1. 将variant.outputs.each中的each改成all
2. output.outputFile = new File(outputFile.parent, fileName)改为 outputFileName  =fileName

**原代码：**

``` 
applicationVariants.all { variant ->
    variant.outputs.each { output ->
        def outputFile = output.outputFile
        if (outputFile != null && outputFile.name.endsWith('.apk')) {

            def fileName = ""
            if (checkQuDao()) {
                fileName = "QiQi_${defaultConfig.versionName}${variant.productFlavors[0].name}.apk"
            } else {
                fileName = "QiQi-debug.apk"
            }

            //    def fileName = "QiQi_${defaultConfig.versionName}${variant.productFlavors[0].name}.apk"
            output.outputFile = new File(outputFile.parent, fileName)
        }
    }
}
```

**修改后的代码：** 

```
applicationVariants.all { variant ->
    variant.outputs.**all** { output ->
        def outputFile = output.outputFile
        if (outputFile != null && outputFile.name.endsWith('.apk')) {

            def fileName = ""
            if (checkQuDao()) {
                fileName = "QiQi_${defaultConfig.versionName}${variant.productFlavors[0].name}.apk"
            } else {
                fileName = "QiQi-debug.apk"
            }

            //    def fileName = "QiQi_${defaultConfig.versionName}${variant.productFlavors[0].name}.apk"
            **outputFileName  =fileName**
        }
    }
}
```
## 2. 修改了错误1后再编译报如下错误：All flavors must now belong to a named flavor dimension

**错误原因：** gradle版本3.0之后添加了flavorDimensions管理  
**修改方案：** 在defaultConfig里面加入flavorDimensions，并在多渠道打包里的每一项里加上dimension。  
**修改后代码：**
```
defaultConfig {
    applicationId globalConfiguration.applicationId
    minSdkVersion globalConfiguration.AndridMinSdkVersion
    targetSdkVersion globalConfiguration.AndroidTargetSdkVersion
    multiDexEnabled true
    flavorDimensions "product"
}
```
```
productFlavors {

    xiaomi{
        dimension "product"
        ...
    }

    huawei{
        dimension "product"
        ...
    }

    oppo{
        dimension "product"
        ...
    }

}
```
>*参考文章[https://blog.csdn.net/chen_xi_hao/article/details/80526049](https://blog.csdn.net/chen_xi_hao/article/details/80526049)*

## 3. ERROR: The SourceSet 'instrumentTest' is not recognized by the Android Gradle Plugin. Perhaps you misspelled something?  
**错误原因：** gradle版本3.0之后已废弃instrumentTest  
**解决方案：** 将instrumentTest替换为androidTest  
**原代码：**
```
sourceSets {
main {
    manifest.srcFile 'AndroidManifest.xml'
    java.srcDirs = ['src']
    resources.srcDirs = ['src']
    aidl.srcDirs = ['src']
    renderscript.srcDirs = ['src']
    res.srcDirs = ['res']
    assets.srcDirs = ['assets']
}
instrumentTest.setRoot('tests')
```
**修改后代码：**
```
sourceSets {
    main {
        manifest.srcFile 'AndroidManifest.xml'
        java.srcDirs = ['src']
        resources.srcDirs = ['src']
        aidl.srcDirs = ['src']
        renderscript.srcDirs = ['src']
        res.srcDirs = ['res']
        assets.srcDirs = ['assets']
    }
    androidTest.setRoot('tests')
```

## 4. ERROR: The Android Gradle plugin supports only Kotlin Gradle plugin version 1.3.0 and higher.
**错误原因：** gradle只支持1.3.0及以上的Kotlin Gradle plugin版本
**修改方案：** 修改kotlin版本，修改工程下的build.gradle文件中的dependencies项里的kotlin版本

## 5. compile错误
**报了三个错误：**  
**1、WARNING: Configuration 'compile' is obsolete and has been replaced with 'implementation' and 'api'.**  
**2、WARNING: Configuration 'testCompile' is obsolete and has been replaced with 'testImplementation'.**  
**3、WARNING: Configuration 'androidTestCompile' is obsolete and has been replaced with 'androidTestImplementation'.**  
**错误原因：** compile已过时
**解决方案：** 使用implementation和api两个字段替换compile
**implementation和api区别：** 
* api 指令  
完全等同于compile指令，没区别，你将所有的compile改成api，完全没有错。  
* implement指令  
这个指令的特点就是，对于使用了该命令编译的依赖，对该项目有依赖的项目将无法访问到使用该命令编译的依赖中的任何程序，也就是将该依赖隐藏在内部，而不对外部公开，这样避免了不同module引用了不同版本的支持包报错的问题。  

>*参考文章[https://www.jianshu.com/p/f34c179bc9d0](https://www.jianshu.com/p/f34c179bc9d0)*

## 6. WARNING: API'variantOutput.getPackageApplication()'isobsolete and has been replaced with'variant.getPackageApplicationProvider()'.

**原代码：**
```
applicationVariants.all { variant ->
    variant.outputs.all { output ->
        def outputFile = output.outputFile
        if (outputFile != null && outputFile.name.endsWith('.apk')) {

            def fileName = ""
            if (checkQuDao()) {
                fileName = "QiQi_${defaultConfig.versionName}${variant.productFlavors[0].name}.apk"
            } else {
                fileName = "QiQi-debug.apk"
            }

            //    def fileName = "QiQi_${defaultConfig.versionName}${variant.productFlavors[0].name}.apk"
            outputFileName = fileName
        }
    }
}
```
**修改后的代码：**
```
applicationVariants.all { variant ->
    variant.outputs.all { output ->
        
            if (checkQuDao()) {
                outputFileName = "QiQi_${variant.versionName}${variant.productFlavors[0].name}.apk"
            } else {
                outputFileName = "QiQi-debug.apk"
            }
            //    def fileName = "QiQi_${defaultConfig.versionName}${variant.productFlavors[0].name}.apk"
    }
}
```