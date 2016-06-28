### 1、IDE工具：
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
### 3、资源文件 Resources

- **命名** 遵循前缀表明类型的习惯，形如`type_foo_bar.xml`。例如：`fragment_contact_details.xml`,`view_primary_button.xml`,`activity_main.xml`.

**组织布局文件** 若果你不确定如何排版一个布局文件，遵循一下规则可能会有帮助。
- 考虑使用[Designtime attributes 设计时布局属性](http://tools.android.com/tips/layout-designtime-attributes)，Android Studio已经提供支持，而不是硬编码`android:text`
(译者注：墙内也可以参考stormzhang的这篇博客[链接](http://stormzhang.com/devtools/2015/01/11/android-studio-tips1/))。

作为一个经验法则,`android:layout_****`属性应该在 layout XML 中定义,同时其它属性`android:****` 应放在 style XML中。此规则也有例外，不过大体工作的很好。这个思想整体是保持layout属性(positioning, margin, sizing) 和content属性在布局文件中，同时将所有的外观细节属性（colors, padding, font）放在style文件中。


例外有以下这些:

- `android:id` 明显应该在layout文件中
- layout文件中`android:orientation`对于一个`LinearLayout`布局通常更有意义
- `android:text` 由于是定义内容，应该放在layout文件中
- 有时候将`android:layout_width` 和 `android:layout_height`属性放到一个style中作为一个通用的风格中更有意义，但是默认情况下这些应该放到layout文件中。

**使用styles** 几乎每个项目都需要适当的使用style文件，因为对于一个视图来说有一个重复的外观是很常见的。
在应用中对于大多数文本内容，最起码你应该有一个通用的style文件，例如：

```xml
<style name="ContentText">
	<item name="android:textSize">@dimen/font_normal</item>
	<item name="android:textColor">@color/basic_black</item>
</style>
```

应用到TextView 中:

```xml
<TextView
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:text="@string/price"
	style="@style/ContentText"
	/>
```


你或许需要为按钮控件做同样的事情，不要停止在那里。将一组相关的和重复`android:****`的属性放到一个通用的style中。


**将一个大的style文件分割成多个文件** 你可以有多个`styles.xml` 文件。Android SDK支持其它文件，`styles`这个文件名称并没有作用，起作用的是在文件
里xml的`<style>`标签。因此你可以有多个style文件`styles.xml`,`style_home.xml`,`style_item_details.xml`,`styles_forms.xml`。
不用于资源文件路径需要为系统构建起的有意义，在`res/values`目录下的文件可以任意命名。


**`colors.xml`是一个调色板** 在你的`colors.xml`文件中应该只是映射颜色的名称一个RGBA值，而没有其它的。不要使用它为不同的按钮来定义RGBA值。

*不要这样做*

```xml
<resources>
	<color name="button_foreground">#FFFFFF</color>
	<color name="button_background">#2A91BD</color>
	<color name="comment_background_inactive">#5F5F5F</color>
	<color name="comment_background_active">#939393</color>
	<color name="comment_foreground">#FFFFFF</color>
	<color name="comment_foreground_important">#FF9D2F</color>
	...
	<color name="comment_shadow">#323232</color>
```


使用这种格式，你会非常容易的开始重复定义RGBA值，这使如果需要改变基本色变的很复杂。同时，这些定义是跟一些环境关联起来的，如`button`或者`comment`,
应该放到一个按钮风格中，而不是在`color.xml`文件中。


***相反，这样做:***

```xml
<resources>

	<!-- grayscale -->
	<color name="white"     >#FFFFFF</color>
	<color name="gray_light">#DBDBDB</color>
	<color name="gray"      >#939393</color>
	<color name="gray_dark" >#5F5F5F</color>
	<color name="black"     >#323232</color>

	<!-- basic colors -->
	<color name="green">#27D34D</color>
	<color name="blue">#2A91BD</color>
	<color name="orange">#FF9D2F</color>
	<color name="red">#FF432F</color>

</resources>
```

向应用设计者那里要这个调色板，名称不需要跟"green", "blue", 等等相同。
"brand_primary", "brand_secondary", "brand_negative" 这样的名字也是完全可以接受的。
像这样规范的颜色很容易修改或重构，会使应用一共使用了多少种不同的颜色变得非常清晰。
通常一个具有审美价值的UI来说，减少使用颜色的种类是非常重要的。


**像对待colors.xml一样对待dimens.xml文件** 与定义颜色调色板一样，你同时也应该定义一个空隙间隔和字体大小的“调色板”。
一个好的例子，如下所示：

```xml
<resources>

	<!-- font sizes -->
	<dimen name="font_larger">22sp</dimen>
	<dimen name="font_large">18sp</dimen>
	<dimen name="font_normal">15sp</dimen>
	<dimen name="font_small">12sp</dimen>

	<!-- typical spacing between two views -->
	<dimen name="spacing_huge">40dp</dimen>
	<dimen name="spacing_large">24dp</dimen>
	<dimen name="spacing_normal">14dp</dimen>
	<dimen name="spacing_small">10dp</dimen>
	<dimen name="spacing_tiny">4dp</dimen>

	<!-- typical sizes of views -->
	<dimen name="button_height_tall">60dp</dimen>
	<dimen name="button_height_normal">40dp</dimen>
	<dimen name="button_height_short">32dp</dimen>

</resources>
```
	
布局时在写 margins 和 paddings 时，你应该使用`spacing_****`尺寸格式来布局，而不是像对待String字符串一样直接写值。
这样写会非常有感觉，会使组织和改变风格或布局是非常容易。

**避免深层次的视图结构** 有时候为了摆放一个视图，你可能尝试添加另一个LinearLayout。你可能使用这种方法解决：

```xml
<LinearLayout
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical"
	>

	<RelativeLayout
		...
		>

		<LinearLayout
			...
			>

			<LinearLayout
				...
				>

				<LinearLayout
					...
					>
				</LinearLayout>

			</LinearLayout>

		</LinearLayout>

	</RelativeLayout>

</LinearLayout>
```


即使你没有非常明确的在一个layout布局文件中这样使用，如果你在Java文件中从一个view inflate 到其他views当中，也是可能会发生的。


可能会导致一系列的问题。你可能会遇到性能问题，因为处理起需要处理一个复杂的UI树结构。
还可能会导致以下更严重的问题[StackOverflowError](http://stackoverflow.com/questions/2762924/java-lang-stackoverflow-error-suspected-too-many-views).


因此尽量保持你的视图tree：学习如何使用[RelativeLayout](https://developer.android.com/guide/topics/ui/layout/relative.html),
如何 [optimize 你的布局](http://developer.android.com/training/improving-layouts/optimizing-layout.html) 和如何使用
[`<merge>` 标签](http://stackoverflow.com/questions/8834898/what-is-the-purpose-of-androids-merge-tag-in-xml-layouts).


**小心关于WebViews的问题.** 如果你必须显示一个web视图，
比如说对于一个新闻文章，避免做客户端处理HTML的工作，
最好让后端工程师协助，让他返回一个 "*纯*" HTML。
[WebViews 也能导致内存泄露](http://stackoverflow.com/questions/3130654/memory-leak-in-webview)
当保持引他们的Activity，而不是被绑定到ApplicationContext中的时候。
当使用简单的文字或按钮时，避免使用WebView，这时使用TextView或Buttons更好。

