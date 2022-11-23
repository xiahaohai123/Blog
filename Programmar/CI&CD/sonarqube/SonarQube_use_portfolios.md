# SonarQube 项目分类检视

## 相关别名

文件夹配置、视图配置

## 背景

一个公司内有多个部门，每个部门都有自己的大量项目。项目在 SonarQube 的首页上的显示会杂乱无章，部门 A 需要在整个公司的所有项目列表里面找出自己的项目，比较麻烦。

我们需要一个控制视图的办法，让各部门能专注于自己的项目，屏蔽掉其他项目。

## Portfolios

SonarQube 提供了一个文件夹系统，以方便我们控制视图。**注意，该功能只有企业版可以使用**。

该功能名为 **Portfolios**，意为文件夹列表，该列表内可以存在复数个 Portfolio。

### 如何操作

1. 使用 Admin 账户登录SonarQube。
2. 进入 Administration (中译: 配置) 页面。
3. 下拉配置选择 Portfolios (中译: 文件夹、视图)。
4. 创建一个 Portfolios, 填好名称后创建即可。
5. 创建出来的 Portfolios 会默认内置一个与 Portfolios 同名的文件夹实体。
6. 选择一个 Portfolio，在右侧编辑 Project selection mode。
7. 根据 Project selection mode，系统会选定一些项目纳入该目录。
8. 执行 Recompute，以让系统将项目纳入进 portfolio。

### Project selection mode

该配置将决定系统以什么策略将项目纳入 portfolio。目前 SonarQube 提供了以下策略:

1. 不选
2. 手动选择
3. 根据 tag (标签) 选择
4. 正则匹配项目名
5. 全部项目

其中根据标签选择比较适合一个公司多部门的项目管理，例如项目 a 可以标记为部门A，这样项目 a 就会自动纳入到部门A的目录。

- 这样创建新项目后如果要纳入文件夹管理时，直接打个标签就好了。
- 注意，打标签后也要执行 Recompute，让系统将项目纳入进 portfolio。
- 使用 tag 的另一个好处是，项目页的过滤器可以根据 tag 进行过滤。
- 如果有一天文件夹被手贱删除，那么新建一个文件夹纳管项目时只要选好 tag 就行。

## 参考文献

1. [Managing Portfolios](https://docs.sonarqube.org/latest/project-administration/managing-portfolios/)