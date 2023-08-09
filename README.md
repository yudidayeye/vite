<!--
 * @Description: vite 源码
 * @Author: lijin
 * @Date: 2023-08-08 18:22:39
 * @LastEditTime: 2023-08-09 15:46:54
 * @LastEditors:
-->

# 调试 vite 源码

## 操作步骤

- 源码下载：https://github.com/vitejs/vite.git

- 构建，生成 sourcemap：依赖安装 -- 修改 `/packages/vite/rollup.config.ts` 的 sourcemap 设置为 `true` -- 构建（需调整 vite 版本为后 example 项目中 vite 的版本），生成 sourcemap 文件

  ```js
  // "/packages/vite/rollup.config.ts"
  function createNodeConfig(isProduction: boolean) {
  return defineConfig({
    ...sharedNodeOptions,
    input: {
      index: path.resolve(__dirname, 'src/node/index.ts'),
      cli: path.resolve(__dirname, 'src/node/cli.ts'),
      constants: path.resolve(__dirname, 'src/node/constants.ts'),
    },
    output: {
      ...sharedNodeOptions.output,
      // 修改
      sourcemap: true,
    },
    external: [
      'fsevents',
      'lightningcss',
      ...Object.keys(pkg.dependencies),
      ...(isProduction ? [] : Object.keys(pkg.devDependencies)),
    ],
    plugins: createNodePlugins(
      isProduction,
      !isProduction,
      // in production we use api-extractor for dts generation
      // in development we need to rely on the rollup ts plugin
      isProduction ? false : './dist/node',
    ),
  })
  }
  ```

- 创建 vite 项目：新建文件夹 `examples` -- `npm init vue@latest` 创建 `vue-project` vite项目 -- 依赖安装 -- 将上面生成的打包后的 `vite/dist` 文件夹替换项目中 `node_modules` 中的 `vite/dist` 文件夹 -- 在 `vite.config.js` 打断点

- 创建调试配置，调试

  ```json
  // ".vscode/launch.json"
  {
    "name": "Launch via NPM",
    "request": "launch",
    "runtimeArgs": ["run-script", "dev"],
    "console": "integratedTerminal",
    "runtimeExecutable": "npm",
    "resolveSourceMapLocations": ["${workspaceFolder}/**"],
    "skipFiles": ["<node_internals>/**"],
    "cwd": "${workspaceFolder}/examples/vue-project",
    "type": "node"
  }
  ```

- 解决源码无法访问问题：替代生成的 sourcemap 文件的 sources ，将 `examples/vue-project/node_modules/vite/dist/node/*` 中的 `.map` 文件的 `../../src/node` 替换为源码的绝对路径 `D:/04-learn-font/learn-package/vite/packages/vite/src/node` -- 重新调试，发现源码能正确定位
