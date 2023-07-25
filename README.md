## MonoRepo 实践

使用 `pnpm` 的 `workspace` 实现

#### 1. 初始化

> 这边 node 版本 16 pnpm 版本 7

创建 monorepo-demo 文件夹，进入文件开始初始化

```bash

mkdir monorepo-demo && cd monorepo-demo
# 初始化package.json
pnpm init -w
# 创建 pnpm-workspace.yaml
touch  pnpm-workspace.yaml
```

在 pnpm-workspace.yaml 下写入

```yaml
packages:
  - "packages/*"
```

表示所有位于 packages 目录下的子目录都是工作区中的包。

一个主包，多个子包,主包 `packages` 包含多个子包，`node_modules` 在根目录下，子包复用 `node_modules`

根目录新建 `packages`，在里面创建 `packages1` 与 `packages2` 两个目录

```bash
cd packages && mkdir packages1 && mkdir packages2

```

然后进到两个目录下初始化子包的 `package.json`

```bash
cd packages1
pnpm init

cd packages2
pnpm init
```

最后得到一个这样的目录结构

```bash
monorepo-demo           # 根目录
├─ packages             #  主包
│  ├─ packages1         #  子包1
│  │  └─ package.json
│  └─ packages2         #  子包2
│     └─ package.json
├─ commitlint.config.js
├─ LICENSE
├─ package.json
├─ pnpm-lock.yaml
├─ pnpm-workspace.yaml  #  workspace文件
└─ README.md
```

#### 2. 使用

在根目录安装下 `react` 试试,

注意: 在根目录下要加 `-w`,表示向项目的根目录安装依赖项，`--workspace-root` 的缩写

```bash
# monprepo-demo/
pnpm add react -w
```

可以看到主包的 package.json 中 多了一个 react 的依赖,

```json
	"dependencies": {
		"react": "^18.2.0"
	}
```

去 package1 与 package2 与里建个 ts 文件看看能不能引入到 react

可以看到 package1 与 package2 都能用到 react 的包

package1

![image](https://github.com/1587315093/monorepo-demo/assets/77056991/945d2d04-01bc-4462-81c0-b4ef7d61a4fd)

package2

![image](https://github.com/1587315093/monorepo-demo/assets/77056991/a26f083b-4920-4255-b7df-1bff7b6c91b7)

再试试到 package1 下装依赖，package2 能不能用到吧,子包则不需要加 `-w`

```bash
# package/package1 下
cd package/package1

pnpm add axios
```

然后 子包 `package1` 的 `package.json` 下多了，而且依赖在 `package1` 的 `node_modules` 下

```json
  "dependencies": {
    "axios": "^1.4.0"
  }
```

引入看看

`package1` 正常引入

![image](https://github.com/1587315093/monorepo-demo/assets/77056991/32a4a121-9db5-4d9d-9118-0d73a6934be3)

`package2` 引入报错~ 因为是 `package1` 的私包

![image](https://github.com/1587315093/monorepo-demo/assets/77056991/260541f0-479e-45ed-8c9c-86753e6c8a32)


