### 一、背景
节约磁盘空间并提升安装速度 https://pnpm.io/zh/motivation

> pnpm 的出现，解决npm 和yarn弊端的问题，又节约安装的成本，是目前比较流行的管理工具，并且随着pnpm的产生，出现了一种新的架构方式mpmorepo
>
> 1. 什么是monorepo
>
>    在以往的开发过程中，通常都是一个仓库存放一个项目，比如现在有三个项目，需要远程创建三个项目远程仓库地址，如果想要三个项目进行下载依赖，就需要分别对三个项目进行分别下载，也就是下载升级三次。
>
>    那么monorepo的思想就是在一个仓库中，可以包含多个项目，每个项目都是独立的，也可以相互依赖，可以共享依赖包，实现安装一次，所有项目共同安装，每个项目也可以独立安装
>
> 2. 探索原理，有点类似js作用域
>
>    ```javascript
>    function monorepo() {
>      const scope = 6
>      function repo1() {
>        const part = 1
>        console.log(part + scope)
>      }
>      function repo2() {
>        const part = 2
>        console.log(part + scope)
>      }
>      function repo3() {
>        const part = 3
>        console.log(part + scope)
>      }
>      return {
>        repo1,
>        repo2,
>        repo3
>      }
>    }
>    const { repo1, repo2, repo3 } = monorepo()
>    repo1()
>    repo2()
>    repo3()
>    ```
>
>    类似于上面的代码，在monorepo总体的函数内部，都有公用的scope变量，每个函数里面又有自已私有的part属性变量，每个函数也使用到了公共变量scope,因为这个变量在每个函数里面并没有提供，所以函数会往上查找，最终找到所需要的上层scope依赖。
>
>    真正的monorepo项目也是使用类似的方式，将公共依赖放在公共可以访问的地方，然后每个独立的项目中可能还有一些私有的依赖，让一个或者多个项目共享公共依赖，就可以避免在多个项目开发的时候，导致这些公共依赖项重复安装的情况





### 二、环境

| 库     | 版本    |
| ------ | ------- |
| nodejs | v17.0.1 |
| npm    | 8.1.1   |
| pnpm   | 7.9.5   |

### 三、如何搭建一个monorepo项目

在nodejs环境下，全局安装pnpm

```bash
npm i pnpm -g
```

###### 1. 创建monorepo文件夹，作为项目的根目录,初始化项目,创建根目录的package.json文件

```bash
pnpm init
```

优化根目录的package.json文件，使其变成私有的包依赖，与要发布的npm包解耦，根目录层主要是管理项目

```json
// 优化前
{
  "name": "monorepo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
// 优化后
{
  "name": "monorepo",
  "version": "1.0.0",
  "description": "",
  // 增加private为true
  "private": true,
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```

###### 2. 创建工作空间配置文件

项目目录创建pnpm-workspace.yaml文件

```yaml
packages:
  # all packages in direct subdirs of packages/
  - 'packages/*'
  # all packages in subdirs of components/
  - 'components/**'
  # exclude packages that are inside test directories
  - '!**/test/**'
```

创建packages目录存放各个项目

```bash
cd packages
npm init vite@latest // 创建app1,app2，app3
```

创建公共依赖配置，创建的各个项目的依赖都是一样，都是基于vite创建的vue+ts的项目，将公共依赖配置到项目根目录，供创建的项目使用,

```json
{
  "name": "monorepo",
  "version": "1.0.0",
  "description": "",
  "private": true,
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "vue": "^3.2.37"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^3.0.3",
    "typescript": "^4.6.4",
    "vite": "^3.0.7",
    "vue-tsc": "^0.39.5"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```

每个新创建的应用项目可以修改成类似这样，将公共的依赖提取到了根目录

```json
{
  "name": "app3",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc --noEmit && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
  },
  "devDependencies": {
  }
}
```

至此，一个monorepo的项目大概初始化好了，可以使用先在根目录先用指令安装

```bash
pnpm install
```

> 注意：初次在根目录中使用pnpm install时会产生根目录的node_modules包依赖文件夹和pnpm-lock.yaml文件，此时，在packages文件夹下的应用项目在还没有启动服务之前是还没有node_modules包依赖文件夹的，这是pnpm比较聪明的地方

进入到packages里面的应用项目里面使用`npm run dev`启动项目，所有的项目都可以正常启动了



###### 3. 根目录package.json创建启动开发脚本和打包脚本

```json
{
  "name": "monorepo",
  "version": "1.0.0",
  "description": "",
  "private": true,
  "scripts": {
    "dev:app1": "vite packages/app1",
    "dev:app2": "vite packages/app2",
    "dev:app3": "vite packages/app3",
    "build:app1": "cd packages/app1 && npm run build",
    "build:app2": "cd packages/app2 && npm run build",
    "build:app3": "cd packages/app3 && npm run build",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "vue": "^3.2.37"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^3.0.3",
    "typescript": "^4.6.4",
    "vite": "^3.0.7",
    "vue-tsc": "^0.39.5"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```



###### 补充知识点

1. 局部安装项目依赖项，可进入到具体的项目里面使用`npm install <package-name>`或者在根目录使用`pnpm install <package-name> --filter <具体的应用名称package.json的name>`
2. 全局项目安装依赖，可在根目录使用`pnpm install <package-name> -W`安装，会安装到所有的项目中



###### 4. 总结

至此，一个个人对monorepo的思想感受和初始化就完成了，可以在项目中开箱即用pnpm+monorepo+vite+typescript



###### 



