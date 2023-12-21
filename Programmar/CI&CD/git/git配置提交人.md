# git 配置提交人


在 Git 中设置用户名和邮箱是为了标识每次提交的作者信息。这信息会被包含在每个 Git 提交中，以追踪是谁做出了什么样的更改。

```shell
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

这两个配置项会存储在 ~/.gitconfig 文件中（--global 标志表示这些配置是全局的，适用于您系统上的所有 Git 仓库）。您可以根据每个特定的仓库需要在该仓库的目录下使用相同的命令，去掉 --global 选项，这样配置将只在当前仓库中生效。

windows 下 gitconfig 在 `C:\Users\电脑登录用户名` 下