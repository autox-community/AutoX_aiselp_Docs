# Termux Api

> v7.1.7 新增

termux 模块提供了与 [termuxApp](https://f-droid.org/packages/com.termux/) 交互的能力，

首先你需要在`termux`中编辑`~/.termux/termux.properties`, 添加以下一行配置以允许运行命令

```
allow-external-apps = true
```

你还需要为该应用授予`com.termux.permission.RUN_COMMAND`权限，
可以选择在"应用详情"中手动允许或者通过调用该模块相关方法

> 此模块不是全局可用，需使用以下代码导入

```js
let termux = require("termux");
```

示例

```js
let { requestPermission, bash, sh } = require("termux");

bash('echo "Hello from Termux bash!"', (result) => {
  console.log(result);
});
```

类型

```ts
export interface Result {
  stdout: string;
  stdout_original_length: number;
  stderr: string;
  stderr_original_length: number;
  exitCode: number;
  errCode: number | null;
  errmsg: string | null;
}
export type Callback = {
  (result: Result): void;
};

export declare function createShortcutCommand(
  options: CommandOptions
): ShortcutCommandFunction;
export declare function runCommand(
  options: CommandOptions,
  callback?: Callback
): void;
export declare const bash: ShortcutCommandFunction;
export declare const sh: ShortcutCommandFunction;
export declare function checkPermission(): boolean;
export declare function requestPermission(
  callback?: (r: boolean) => void
): void;
```

## runCommand([options](#commandoptions), callback?)

向`termux`发送一个运行命令的请求，命令运行在`termux`中，而不是该应用，可选传入一个回调函数接收结果，如果没有传入`callback`，则视为放弃结果，该函数不会阻塞线程。

如果没有权限则会直接抛出错误

## createShortcutCommand([options](#commandoptions))

创建一个运行命令的快捷函数，传入的选项作为默认值，返回一个[快捷函数](#shortcutcommandfunction)

## checkPermission()

检查是否拥有权限

## requestPermission()

向用户请求权限，可传入回调函数获得结果

```js
let { requestPermission, bash, sh } = require("termux");
requestPermission((s) => {
  log(s); //true or false
});
```

## bash, sh

这两个函数是[快捷函数](#shortcutcommandfunction)，但是必须传入参数，没有参数会抛出错误，

传入的参数将附加到`bash -c ...`或`sh -c ...`，传入多个参数将使用空格进行拼接

## CommandOptions

类型

```ts
export interface CommandOptions {
  //可执行文件的路径，必填
  path: string;
  //参数
  arguments?: string[];
  //工作目录
  workdir?: string;
  //是否在后台运行，默认为false
  background?: boolean;
  //当在前台运行时有效
  sessionAction?: "0" | "1" | "2" | "3";
  //前台运行时显示的label
  label?: string;
  //前台运行时的description
  description?: string;
}
```

## ShortcutCommandFunction

[createShortcutCommand](#createshortcutcommandoptions)所返回的函数，
可传入额外的选项附加到默认选项

```ts
type ShortcutCommandFunction = {
  //args 将作为额外参数添加到默认选项
  (args?: string | string[], callback?: Callback): void;
  //第二个参数是字符串则作为 workdir 覆盖默认值
  (args?: string | string[], workdir?: string, callback?: Callback): void;
  //第二个参数是对象则作为新的选项覆盖原有的相关属性
  (
    args?: string | string[],
    options?: CommandOptions,
    callback?: Callback
  ): void;
};
```
