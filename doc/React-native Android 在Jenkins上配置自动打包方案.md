  #React-native Android 在Jenkins上配置自动打包方案 
       使用jenkins来实现自动化构建，可以简化开发测试的流程，原来debug包都不会做混淆，现在用了jenkins会自动的打混淆包，除了环境不一样，其他配置debug和release包都一样，这样就可以避免代码混淆带来的问题，早日发现早日治疗。推荐大家在废弃的电脑上搭建一个Jenkins。具体的搭建流程见[这里](http://my.oschina.net/u/930967/blog/299058)     

这里介绍一下我在项目中使用了React-native之后的jenkins配置，默认已经在jenkins上已经搭建好了普通Android的打包环境，如果想打带有React-native的apk，首先在React官方的[Generating Signed APK](https://facebook.github.io/react-native/docs/signed-apk-android.html#content)中有bundle命令来生成index.android.bundle

	$ react-native bundle --platform android --dev false --entry-file index.android.js \
	  --bundle-output android/app/src/main/assets/index.android.bundle \
	  --assets-dest android/app/src/main/res/
所以我们在Android打包之前首先要执行这个命令，明确了这个区别，就开始动手了。

- 首先在jenkins服务器上升级buildToolsVersion，compileSdkVersion 到23
![这里我全部升级完了](http://img.blog.csdn.net/20151128142442620)
如果下载慢，建议换成sdk.gdgshanghai.com的镜像
![sdk.gdgshanghai.com](http://img.blog.csdn.net/20151128142526224)

- 接下来在服务器上搭建react-native环境

不做重复的事情，很多文章已经写的很清楚，具体搭建步骤参考下面的博客http://www.race604.com/react-native-for-android-start/

		注意一点，很多同学搭建换成后，执行
		react-native init AwesomeProject
		会发现卡了半天还是没有成功，这个时候建议你使用
		npm config set registry=https://registry.npm.taobao.org
		将npm的源换成淘宝镜像
- 测试环境安装成功之后，开始配置我们的Jenkins

我们是通过jenkins Invoke Gradle script执行assembleQa --stacktrace task来打包的，
![Invoke Gradle script](http://img.blog.csdn.net/20151128143605795)
如果我们希望在执行这个task之前让React-native 生成index.android.bundle，那么我们可以在build.gradle写这样的task，但是这里我介绍一个更简单的方式，在jenkins中增加构建步骤->windows系统选择Execute windows batch command ||mac系统选择Excute shell
![选择Excute shell](http://img.blog.csdn.net/20151128143748789)

#关键的步骤来了！！！！！

我们按住Execute shell的右上角，将它拖动到 Invoke Gradle script的前面，这样就可以先执行Execute shell中的命令

- 接下来开始写Execute shell中的命令
	

```
react-native bundle --platform android -dev false --entry-file react-native/index.android.js \
 --bundle-output app/src/main/assets/index.android.bundle \
 --assets-dest app/src/main/res/
```
测试一下，失败，报react-native: command not found
google一下，React-native jenkins，发现没有任何相关的记录，菊花一紧，每当Stack Overflow上国外友人还没有踩过坑的时候，我就有一种使命感，难道我已经走在了全世界码农的前面？难道我就要这样走向人生颠覆了么，这样的幻觉让我像疯狗一样查资料誓要把这个问题解决，给全世界人民带来福利。

打了一针鸡血，发现jenkins的shell命令是跑在自己的Shell中的，这意味着什么？需要我们指定绝对路径，好吧，更换一成绝对路径。

```
which react-native
```
获取react-native的绝对路径,文件路径也一并换掉，整个命令变成了

```
/usr/local/bin/react-native bundle --platform android -dev false --entry-file /Users/jenkins/.jenkins/jobs/testAndroid-qa/workspace/react-native/index.android.js \
 --bundle-output /Users/jenkins/.jenkins/jobs/testAndroid-qa/workspace/app/src/main/assets/index.android.bundle \
 --assets-dest /Users/jenkins/.jenkins/jobs/testAndroid-qa/workspace/app/src/main/res/
```
再来构建->成功。

看一下我的构建历史
![这里写图片描述](http://img.blog.csdn.net/20151128150252868)

		不断的失败，
		不断的解决，
		我为全世界人民踩坑，
		我为全世界人民探路，
		我活在自我陶醉的程序员世界。






