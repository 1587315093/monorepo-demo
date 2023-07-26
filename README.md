## MonoRepo 实践

使用 `pnpm` 的 `workspace` 实现

#### 1. 初始化

> 这边 node 版本 16 pnpm 版本 7

创建 monorepo-demo 文件夹，进入文件开始初始化

```bash
# 1. 建文件夹
mkdir monorepo-demo && cd monorepo-demo

# 2. 初始化package.json
pnpm init -w

# 3. 创建 pnpm-workspace.yaml
touch  pnpm-workspace.yaml
```

在 pnpm-workspace.yaml 下写入

表示所有位于 packages 目录下的子目录都是工作区中的包。

```yaml
packages:
  - "packages/*"
```

一个主包，多个子包

主包 `packages` 包含多个子包，`node_modules` 在根目录下，子包复用 `node_modules`

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

注意: 在根目录下要加 `-w`,表示向项目的根目录安装依赖项，`-w` 是`--workspace-root` 的缩写

```bash
# monprepo-demo/
pnpm add react -w
```

可以看到主包的 `package.json` 中 多了一个 `react` 的依赖,

```json
"dependencies": {
	"react": "^18.2.0"
}
```

去 `package1` 与 `package2` 与里建个 `ts` 文件看看能不能引入到 `react`

可以看到 `package1` 与 `package2` 都能用到 `react` 的包

`package1`

![image](https://github.com/1587315093/monorepo-demo/assets/77056991/945d2d04-01bc-4462-81c0-b4ef7d61a4fd)

`package2`

![image](https://github.com/1587315093/monorepo-demo/assets/77056991/a26f083b-4920-4255-b7df-1bff7b6c91b7)

再试试到 `package1` `下装依赖，package2` 能不能用到吧，子包则不需要加 `-w`

```bash
# package/package1 下
cd package/package1

pnpm add axios
```

然后 子包 `package1` 的 `package.json` 下多了以下配置，而且依赖在 `package1` 的 `node_modules` 下

```json
  "dependencies": {
    "axios": "^1.4.0"
  }
```

引入看看

`package1` 正常引入

![image](https://github.com/1587315093/monorepo-demo/assets/77056991/32a4a121-9db5-4d9d-9118-0d73a6934be3)

而 `package2` 引入报错~ 因为是 `package1` 的私包

![image](https://github.com/1587315093/monorepo-demo/assets/77056991/260541f0-479e-45ed-8c9c-86753e6c8a32)

#### 3. 项目

使用 vite 创个 react 项目

```bash
cd packages

pnpm create vite

# 然后填文件名和模板，这边填的是 package3文件名 然后React+Ts模板

```

然后 `pnpm dev` 也能正常启动

![image](https://github.com/1587315093/monorepo-demo/assets/77056991/ef94f9d0-ad5f-4227-b109-cc715604be40)

但是初始化的一些依赖，下到了私包中，需要手动移到主包的依赖表下

![image](https://github.com/1587315093/monorepo-demo/assets/77056991/3378075c-b7ea-4ef6-af9f-47981a8b1ccd)

移动完后把 `packages3` 的 `node_modules` 删掉，然后在根目录下 `pnpm install`

```bash
# cd 到 monorepo-demo/
pnpm install

# cd 到 packages3/
rm -rf node_modules

# 然后启动packages3的项目
pnpm dev
```

一顿命令框框输完后，`packages3` 项目还是可以正常启动的

但是每次都要进入到子包才能启动本地开发？太麻烦了

可以在主包的 `packages.json` 里加一个脚本叫 `dev:p3` 代表这启动 `packages3` 的 `dev` 服务器

`"--filter"` 是一个参数，用于指定 `pnpm` 命令中要操作的子包的名称或路径

```diff
	"scripts": {
+		"dev:p3": "pnpm --filter=packages3 dev",
		"prepare": "husky install",
		"commitmsg": "echo \"Commit message validation failed. Please make sure your commit message follows the conventional commit format.\" && exit 1"
	}
```

然后在主包根目录下来使用 `dev:p3` 去启动 `packages3` 了，太方便辣

```bahs
pnpm dev:p3
```

给 `packages1` 和 `packages2` 也初始化下看看，之前有过这两个文件夹，`vite` 创项目要创文件夹，我直接把原来两个的删了

```bash
rm -rf packages1 && rm -rf packages2
```

然后再创

```bash
cd packages　
# 老样子，但是文件名叫packages1 与　packages2
pnpm create vite
```

还是一样的把两个包的依赖删掉，因为都是用的主包的依赖，没差别

主要是为了试试多个项目启动命令

```diff
	"scripts": {
+		"dev:p1": "pnpm --filter=packages1 dev",
+		"dev:p2": "pnpm --filter=packages2 dev",
		"dev:p3": "pnpm --filter=packages3 dev",
		"prepare": "husky install",
		"commitmsg": "echo \"Commit message validation failed. Please make sure your commit message follows the conventional commit format.\" && exit 1"
	},
```

这样 `pnpm dev:p1` 或者 `pnpm dev:p2`　就可以启动子包 `packages1` 与　 `packages2` 　了
