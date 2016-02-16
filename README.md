### 优雅的项目配置--常用库和版本管理

原文地址：http://blog.csdn.net/caroline_wendy/article/details/50552549

　　随着Android开发的成熟, 模块越来越多, 为了开发稳定的程序, 引入的库也随之增加, 如何确保所有项目使用相同的编译版本he库版本呢?
当然, Gradle的参数配置可以帮我们实现这些.<br/>
####主要:
* (1) 常用库的展示与配置.
* (2) 统一管理项目和库的版本.
* (3) 设置项目的私有参数.

####1、常用库
编程三剑客, RxJava+Retrofit+Dagger<br/>
常用: ButterKnife依赖注解, Glide/Picasso图片处理<br/>
使用根项目(rootProject)的参数管理子项目的版本<br/>
```Java
apply plugin: 'me.tatarka.retrolambda'      // Lambda表达式
apply plugin: 'com.android.application'     // Android应用
apply plugin: 'com.neenbedankt.android-apt' // 编译时类
apply plugin: 'com.android.databinding'     // 数据绑定

def cfg = rootProject.ext.configuration // 配置
def libs = rootProject.ext.libraries // 库

android {
    compileSdkVersion cfg.compileVersion
    buildToolsVersion cfg.buildToolsVersion

    defaultConfig {
        applicationId cfg.package
        minSdkVersion cfg.minSdk
        targetSdkVersion cfg.targetSdk
        versionCode cfg.version_code
        versionName cfg.version_name

        buildConfigField "String", "MARVEL_PUBLIC_KEY", "\"${marvel_public_key}\""
        buildConfigField "String", "MARVEL_PRIVATE_KEY", "\"${marvel_private_key}\""
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    // 注释冲突
    packagingOptions {
        exclude 'META-INF/services/javax.annotation.processing.Processor'
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'

    // Android
    compile "com.android.support:design:${libs.supportVersion}"
    compile "com.android.support:appcompat-v7:${libs.supportVersion}"
    compile "com.android.support:cardview-v7:${libs.supportVersion}"
    compile "com.android.support:recyclerview-v7:${libs.supportVersion}"
    compile "com.android.support:palette-v7:${libs.supportVersion}"

    // Retrofit
    compile "com.squareup.retrofit:retrofit:${libs.retrofit}"
    compile "com.squareup.retrofit:converter-gson:${libs.retrofit}"
    compile "com.squareup.retrofit:adapter-rxjava:${libs.retrofit}"

    // ReactiveX
    compile "io.reactivex:rxjava:${libs.rxandroid}"
    compile "io.reactivex:rxandroid:${libs.rxandroid}"

    // Dagger
    compile "com.google.dagger:dagger:${libs.dagger}"
    apt "com.google.dagger:dagger-compiler:${libs.dagger}"
    compile "org.glassfish:javax.annotation:${libs.javax_annotation}"

    // Others
    compile "com.jakewharton:butterknife:${libs.butterknife}" // 资源注入
    compile "com.github.bumptech.glide:glide:${libs.glide}" // 图片处理
    compile "jp.wasabeef:recyclerview-animators:${libs.recycler_animators}" // Recycler动画
    compile "de.hdodenhof:circleimageview:${libs.circleimageview}" // 头像视图
}
```
项目版本:
```Java
def cfg = rootProject.ext.configuration
cfg.compileVersion
```
库版本:
```Java
def libs = rootProject.ext.libraries
${libs.retrofit}
```Java
####2、参数管理
buildConfigField管理私有参数, 配置在gradle.properties里面.
```Java
android {
    defaultConfig {
        buildConfigField "String", "MARVEL_PUBLIC_KEY", "\"${marvel_public_key}\""
        buildConfigField "String", "MARVEL_PRIVATE_KEY", "\"${marvel_private_key}\""
    }
}
```
设置参数的类型\变量名\位置三个部分.
```Java
marvel_public_key   = 74129ef99c9fd5f7692608f17abb88f9
marvel_private_key  = 281eb4f077e191f7863a11620fa1865f2940ebeb
```
未指定路径, 默认是配置在gradle.properties中<br/>
两个地方可以配置参数, 一个是项目的build.gradle, 一个是gradle.properties<br/>
项目中使用BuildConfig.xxx引入参数<br/>
```Java
MarvelSigningIterceptor signingIterceptor = new MarvelSigningIterceptor(
BuildConfig.MARVEL_PUBLIC_KEY, BuildConfig.MARVEL_PRIVATE_KEY);
```
####3. 版本管理
版本管理配置在项目的build.gradle中, 包含两个部分, 一个是项目的版本, 一个是库的版本. 把常用参数设置成为变量. 子项目使用rootProject.ext.xxx的形式引入.
```Java
ext {
    configuration = [
            package          : "me.chunyu.spike.springrainnews",
            buildToolsVersion: "23.0.1",
            compileVersion   : 23,
            minSdk           : 14,
            targetSdk        : 23,
            version_code     : 1,
            version_name     : "0.0.1",
    ]

    libraries = [
            supportVersion    : "23.1.1",
            retrofit          : "2.0.0-beta2",
            rxandroid         : "1.1.0",
            dagger            : "2.0",
            javax_annotation  : "10.0-b28",
            butterknife       : "7.0.1",
            glide             : "3.6.1",
            recycler_animators: "2.1.0",
            circleimageview   : "2.0.0"
    ]
}

buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:2.0.0-alpha5'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
        classpath 'me.tatarka:gradle-retrolambda:3.2.4'
        classpath 'com.android.databinding:dataBinder:1.0-rc4'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
通过这样的方式管理Android项目, 可以便捷的更改版本号, 所有模块统一.
