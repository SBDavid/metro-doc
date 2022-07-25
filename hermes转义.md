# 把js文件翻译成hbc

## 位置
1. cli项目
2. packages/cli

## 添加打包命令变量
1. 文件：cli/src/commands/bundle/bundleCommandLineArgs.ts
2. CommandLineArgs 增加类型
3. 增加command的参数配置
name: '--useJsBundle [boolean]',
name: '--useHermesBundle [boolean]',

## buildBundle.js 
这个文件负责创建server，js文案打包，图片资源打包
我们自己添加了压缩ZIP的逻辑
- 1. 文件: cli/src/commands/bundle/buildBundle.js 
- 2. 在 buildBundle 中，为参数设定默认值
```js
if (args.useJsBundle === undefined) {
  args.useJsBundle = true;
}
if (args.useHermesBundle === undefined) {
  args.useHermesBundle = false;
}
```
- 3. 调用命令进行转义
```js
const {exec} = require('child_process');

在 save js bundle 后进行转义
await output.save(bundle, args, logger.info);
if (args.useHermesBundle) {
  await new Promise((resolve, reject) => {
    let dir = path.dirname(args.bundleOutput);
    let out = path.join(dir, args.bundleName + '.bundle.hbc');
    let src = path.join(dir, args.bundleName + '.bundle');
    exec(
      `./node_modules/hermes-engine/osx-bin/hermesc -emit-binary -out ${out} ${src}`,
      (err, stdout) => {
        if (err) {
          console.error(err);
          reject(err);
          return;
        }
        console.log(stdout);
        resolve(stdout);
      },
    );
  });
}
```

## archiveZip.js 
在这里修改config文件，并增加hbc文件到压缩包

- 1. 压缩文件
```js
config.useHermesBundle = options.useHermesBundle;
config.useJsBundle = options.useJsBundle;
```

- 2. 添加转义后的文件hbc
```js
if (config.useJsBundle) {
  archive.append(fs.createReadStream(options.bundleOutput), {
    name: options.bundleName + '.bundle',
  });
}
if (config.useHermesBundle) {
  archive.append(fs.createReadStream(options.bundleOutput), {
    name: options.bundleName + '.bundle.hbc',
  });
}
```