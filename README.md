# AutoTrackDemo
##### 简介
埋点工具，利用Android Transform API 在编译期间动态修改代码,将埋点代码插入特定方法中。
参考的库[：https://github.com/JieYuShi/Luffy](https://github.com/JieYuShi/Luffy)

##### 结构
Demo项目分为两块：app和plugin；
plugin是实现埋点插桩的库；
app是接入的demo；

##### 编译时注意
1.plugin打包出来的插件放在本地maven库(目录snapshotRepo下面)，没有放到远端maven仓库,app应用也是引用本地，所以如果改动了plugin的代码，需要在重新打包插件上传本地maven库：

```
gradlew uploadArchives
```

##### app引用
1.项目的build.gradle下：

```
buildscript {
    ...
    repositories {
        ...
        maven {
            url uri('./snapshotRepo')
        }
    }
    dependencies {
        ...
        classpath 'com.chzy:autotrack-gradle-plugin:1.0.3-SNAPSHOT'
    }
}
```
2.app的build.gradle下：

```
apply plugin: 'com.chzy.autotrack'

AutoTrackConfig {
    // 是否打印日志,可选,默认false
    isDebug = false
    // 是否打开SDK的日志全埋点采集,可选,默认true
    isOpenLogTrack = true
    // 自定义track类
    // 你的埋点入口类的路径。（埋点入口类可参考Demo里AutoTrackHelper的实现）
    trackClass = 'com/demo/cc/autotrack/AutoTrackHelper'
    // com.demo.cc 换成你的包名
    include = ["com.demo.cc","butterknife.internal"]
    
    exclude = []
    // 支持自定义配置,可选,默认空
    matchData = []
}

```

编译通过之后在app\build\intermediates\transforms\AutoTrack\debug下面可以看到插桩后的代码。
比如：

```
public class MainActivity extends AppCompatActivity {
    private AppBarConfiguration appBarConfiguration;
    private ActivityMainBinding binding;

    public MainActivity() {
    }

    protected void onCreate(Bundle savedInstanceState) {
        ...
        this.binding.fab.setOnClickListener(new OnClickListener() {
            @AutoDataInstrumented
            public void onClick(View view) {
                //这一行是插桩插入的代码
                AutoTrackHelper.trackViewOnClick(view);
                Snackbar.make(view, "Replace with your own action", 0).setAction("Action", (OnClickListener)null).show();
            }
        });
    }

    @AutoDataInstrumented
    public boolean onOptionsItemSelected(MenuItem item) {
        //这一行是插桩插入的代码
        AutoTrackHelper.trackMenuItem(this, item);
        int id = item.getItemId();
        return id == 2131230788 ? true : super.onOptionsItemSelected(item);
    }

```
Logcat可以过滤日志 **[自动埋点]** 打印埋点代码输出的信息，验证是否埋点成功。


