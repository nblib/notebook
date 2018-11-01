### 安装idea插件
1. 下载插件:
```
http://plugins.jetbrains.com/plugin/1347-scala
```
2. 安装插件:`  idea->plugins->Install from disk`
3. 重启idea
### 安装scala
1. idea 新建project->scala
2. Scala SDK 为空,点击后方create
3. 点击Download,选择需要的版本
### 配置maven项目
1. 新建maven项目
2. 删掉 java,resources,test等无用文件夹
3. 新建main->scala文件夹
4. 右击项目名->Add Framework Support->选择scala
5. 右击scala文件夹->make dir as -> source root
6. finish