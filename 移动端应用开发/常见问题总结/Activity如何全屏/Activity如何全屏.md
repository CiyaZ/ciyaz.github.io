# Activity如何全屏

使用Android Studio新建的工程，Activity默认继承AppCompatActivity，它提供了更美观的界面，和新特性支持，但是因此也只能使用AppCompat的theme。原来的内置theme有全屏的，但是新的AppCompat的theme却没有提供，实际上我们继承一个AppCompat的theme加上全屏属性，也是一样能工作的。

styles.xml
```xml
<!-- Base application theme with full screen -->
<style name="AppTheme.FullScreen" parent="Theme.AppCompat.Light.DarkActionBar">
  <!-- Customize your theme here. -->
  <item name="colorPrimary">@color/colorPrimary</item>
  <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
  <item name="colorAccent">@color/colorAccent</item>
  <item name="android:windowFullscreen">true</item>
  <item name="windowNoTitle">true</item>
  <item name="windowActionBar">false</item>
</style>
```
