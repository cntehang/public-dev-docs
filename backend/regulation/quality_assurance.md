# quality_assurance

代码检查规则，放到单独的 repo，所有项目公用

## 添加方式

先 cd 到项目的`build_scripts`目录下面，然后执行下面命令：

```shell
git submodule add https://github.com/cntehang/quality_assurance.git
```

这样就把代码检查工具添加到你的目录下了，即可像其他的一样正常使用

假如执行上面命令的过程中遇到报错：`'build_scripts/quality_assurance' already exists and is not a valid git repo`

尝试在`build_scripts`目录下运行如下命令：

```shell
rm -rf quality_assurance/
git submodule add -f https://github.com/cntehang/quality_assurance.git
```

## 更新方式：

新 clone 下来的后台项目是不会默认 clone 子 module 的，所以`quality_assurance`还没有 clone，所以需要在项目的根目录执行下面的命令

```shell
git submodule update --init --recursive
```

执行完之后，`quality_assurance`就会 clone 到本地项目目录下，如此即可正常使用了

后面如果`quality_assurance`有更新，执行同样的命令即可.
