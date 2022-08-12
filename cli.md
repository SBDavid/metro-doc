# 入口

- 1. react-native cli.js
    - 它调用了 @react-native-community/cli
    - cli.run();
- 2. @react-native-community/cli
    - 运行命令的地方： setupAndRun()
    - 其实就是运行 command 中的 func
    - command 位于 @react-native-community/cli/commands/bundle/
- 3. ramBundle 命令
    - 内部使用 bundle 命令
    - 使用 ramBundle 的output: metro/src/shared/output/RamBundle
- 4. bundle 命令
    - 内部使用了 buildBundle 模块
- 5. buildBundle 模块
    - 这个模块主要逻辑是
    - const bundle = await output.build(server, requestOpts);
    - await output.save(bundle, args, logger.info);
    - 这里的 output 主要是 
        - metro/src/shared/output/RamBundle
        - metro/src/shared/output/bundle
- 6. 如果是 RamBundle 
    - output.build：使用了 server 中的 getRamBundleInfo 方法
    - 在 getRamBundleInfo 方法中
        - 解析依赖关系 this._bundler.buildGraph
        - 完成代码的转换 getRamBundleInfo
- 7. getRamBundleInfo 模块
    - 连接 pre 与 graph
    - 通过 getAppendScripts 添加启动代码，sourcemap



# 本次修改
## baseJSBundle
如果是打业务包，则不需要preModules
```js
if (!options.baseBundle) {
    preModules = [];
}
```

## 添加完添加启动代码，sourcemap，后重新给予模块id
```js
// 添加完添加启动代码，sourcemap，后重新给予模块id
  const postModule = getAppendScripts(
    entryPoint,
    [...preModules, ...modules],
    graph.importBundleNames,
    {
      asyncRequireModulePath: options.asyncRequireModulePath,
      createModuleId: options.createModuleId,
      getRunModuleStatement: options.getRunModuleStatement,
      inlineSourceMap: options.inlineSourceMap,
      projectRoot: options.projectRoot,
      runBeforeMainModule: options.runBeforeMainModule,
      runModule: options.runModule,
      sourceMapUrl: options.sourceMapUrl,
      sourceUrl: options.sourceUrl,
      baseBundle: options.baseBundle,
    },
  );
  postModule.forEach((module: Module<>) => options.createModuleId(module.path));

  const postCode = processModules(
    postModule,
    processModulesOptions,
  )
    .map(([_, code]) => code)
    .join('\n');
```

# metro 如何发布
## yarn 安装
yarn install

## 编译 prepublish
npm run prepublish

## 发布
npm publish

## 删除临时文件
npm run postpublish

## 修改cli项目的版本，并发布
"@xmly/metro": "0.3.4-alpha1",

"@xmly/cli": "0.3.9-alpha1",
npm run prebuild
npm run build

## 修改xm-react-native的package.json
git push


