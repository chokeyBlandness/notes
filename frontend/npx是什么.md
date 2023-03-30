# npx是什么

npx 是 npm5.2.0版本新增的一个工具包，定义为npm包的执行者，相比 npm，npx 会自动安装依赖包并执行某个命令。npm是node的包管理器。

假如我们要用 create-react-app 脚手架创建一个 react 项目，常规的做法是先安装 create-react-app，然后才能使用 create-react-app 执行命令进行项目创建。

```shell
npm i -g create-react-app
create-react-app my-react-app
```

有了 npx 后，我们可以省略安装 create-react-app 这一步。

```shell
npx create-react-app my-react-app
```

npx 会在当前目录下的`./node_modules/.bin`里去查找是否有可执行的命令，没有找到的话再从全局里查找是否有安装对应的模块，全局也没有的话就会自动下载对应的模块，如上面的 create-react-app，npx 会将 create-react-app 下载到一个临时目录，用完即删，不会占用本地资源。

### npx命令参数

| 命令参数          | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| --no-install      | 告诉npx不要自动下载，也就意味着如果本地没有该模块则无法执行后续的命令。 |
| --ignore-existing | 告诉npx忽略本地已经存在的模块，每次都去执行下载操作，也就是每次都会下载安装临时模块并在用完后删除。 |

