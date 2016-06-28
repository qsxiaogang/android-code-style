### 1、工具：
首推使用[Android Studio](https://developer.android.com/sdk/installing/studio.html),因为他是由谷歌开发，最接近Gradle，默认使用最新的工程结构，已经到beta阶段（目前已经有release 2.2了），它就是为Android开发定制的。
**并使用**[`Square's Java and Android projects code style settings`](https://github.com/square/java-code-styles)作为`code style`。

### 2、Gradle 配置

**常用结构** 参考[Google's guide on Gradle for Android](http://tools.android.com/tech-docs/new-build-system/user-guide)


**小任务** 除了(shell, Python, Perl, etc)这些脚本语言，你也可以使用Gradle 制作任务。
更多信息请参考[Gradle's documentation](http://www.gradle.org/docs/current/userguide/userguide_single.html#N10CBF)。


**密码** 在做版本release时你app的 `build.gradle`你需要定义 `signingConfigs`.此时你应该避免以下内容：


_不要做这个_ . 这会出现在版本控制中。

```groovy
signingConfigs {
	release {
		storeFile file("myapp.keystore")
		storePassword "password123"
		keyAlias "thekey"
		keyPassword "password789"
	}
}
```
	
而是，建立一个不加入版本控制系统的`gradle.properties`文件。

```
KEYSTORE_PASSWORD=password123
KEY_PASSWORD=password789
```


那个文件是gradle自动引入的，你可以在`buld.gradle`文件中使用，例如：

```groovy
signingConfigs {
	release {
		try {
			storeFile file("myapp.keystore")
			storePassword KEYSTORE_PASSWORD
			keyAlias "thekey"
			keyPassword KEY_PASSWORD
		}
		catch (ex) {
			throw new InvalidUserDataException("You should define KEYSTORE_PASSWORD and KEY_PASSWORD in gradle.properties.")
		}
	}
}
```
	

**使用 Maven 依赖方案代替使用导入jar包方案** 如果在你的项目中你明确使用了
jar文件，那么它们可能成为永久的版本，如`2.1.1`.下载jar包更新他们是很繁琐的，
这个问题Maven很好的解决了，这在Android Gradle构建中也是推荐的方法。你可
以指定版本的一个范围，如`2.1.+`,然后Maven会自动升级到制定的最新版本，例如：

```groovy
dependencies {
	compile 'com.netflix.rxjava:rxjava-core:0.19.+'
	compile 'com.netflix.rxjava:rxjava-android:0.19.+'
	compile 'com.fasterxml.jackson.core:jackson-databind:2.4.+'
	compile 'com.fasterxml.jackson.core:jackson-core:2.4.+'
	compile 'com.fasterxml.jackson.core:jackson-annotations:2.4.+'
	compile 'com.squareup.okhttp:okhttp:2.0.+'
	compile 'com.squareup.okhttp:okhttp-urlconnection:2.0.+'
}
```
### 3、

