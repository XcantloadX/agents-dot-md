# AutoX Agent 自动编写笔记

> 文档源：https://autox-doc.vercel.app/ ｜ 适用：AutoX V7（需 >=7.1.5），手机与电脑同局域网

## 1. 能操控手机的三种外部能力（external_support）

| 方式 | 原理 | Agent 可用性 |
|---|---|---|
| VSCode 插件 `Autox.js` | 调试服务/WiFi+扫码，ADB-USB 也可，F5 run | 不适合 Agent（要手点） |
| **`autox-cli`（推荐）** | `POST http://<devip>:9317/api/v1?type=getip/runProject/saveProject` 推 zip | ✅ 有 bash 就行，已打通 |
| 应用广播 Intent | `adb shell am start -n ...OpenIntentActivity/RunIntentActivity -d file:///sdcard/xxx.js` | ✅ 备用，需 adb +（隐式要开“允许广播运行脚本”） |

`autox-cli@1.0.1` 源码要点（`src/tasks|runScript|runProject|save.ts`）：
- `getip()` 先 `POST?type=getip` 验连通；`--devip/-d` 传入否则交互问并缓存 `.deviceIpAddress`。
- `run file.js` = 包成 `project-<rand16>/file.js + project.json(main=file)` 再 `?type=runProject`。
- `run dir/` = `zip.directory(dir, project-xxx)` 直接推；`save dir/ --name` = `zip.directory(dir,'save') + ?type=saveProject&saveName=` 覆盖到脚本主目录，不支持单文件 save。
- 只回 `操作完成` = 上传成功，**不代表执行成功**，必须自建回调用验证。

```bash
npm i autox-cli -g
autox run path --devip <手机IP>
autox save ./proj --devip <手机IP> --name 存档名
curl -s -X POST 'http://<手机IP>:9317/api/v1?type=getip' -d '{}'
# => {"remoteHost":"<电脑IP>",...} 即连通
```

## 2. 引擎二选一（文件名即开关）

- **V6 Rhino（`.js`）**：全局函数，ES5+部分ES6，`require`，`threads` 多线程，`http.get` 同步阻塞。快写快测用它。
- **V7 Node（`.mjs/.cjs/.node.js`，`.mjs` 开 ESM）**：全 `import`（如 `import {showToast} from "toast"`），`Promise/async`，无多线程（`java` 模块转 Promise），可吃 npm + TS（`npm i -D autox-v7-api` + `npx install-autox-types`），UI 用 vue3/vue-core+htm。工程化/长维护用它。

## 3. Agent 写脚本前的必查前置权限（人先开好）

1. 无障碍：脚本头 `auto();` / `auto.waitFor()`，否则抛异常跳设置页。`fast` 模式开控件缓存。
2. 截图：`requestScreenCapture()`（弹一次“总是允许”）或 V7 `auto.takeScreenshot()`；`captureScreen()` 无权限直接 `SecurityException`。
3. 悬浮窗/后台/电池白名单/锁屏：`console.show()` 要悬浮窗；定时任务要自启动白名单；有锁屏密码别指望 `device.wakeUp()` 全自动。

## 4. 自动化三板斧（按优先级）

1. **控件优先（软件类）**：`text/desc/id/className + boundsInside` 链式定位，`findOne(timeout)/findOnce()/exists()/waitFor()`，动作 `click/longClick/setText/scrollForward`。`id` 在 QQ 列表/微信多版本下不可靠，改用 `textContains/desc`。androidx/support 控件（如 `androidx.recyclerview.widget.RecyclerView`）精确 `className()` 命中为 0，一律用 `classNameContains()`；再无结果用 `scrollable(true)` 兜底，多个容器取 `childCount` 最大者。
2. **坐标兜底（游戏/不可点控件）**：`click(x,y)/longClick/press/swipe/gesture`（7.0+）或 `RootAutomator.tap/swipe`（要 root），`setScreenMetrics(1080,1920)` 做分辨率放缩；`bounds().centerX/Y` + 坐标点击是常用桥接。
3. **图像/OCR（动态页）**：`findImage/matchTemplate`（`threshold~0.9`，`region` 缩范围），`findColor/findMultiColors/detectsColor`，`ocr`；`captureScreen()` 连调间隔 >16ms，注意 `recycle()`。

常用模块速查：`app/device/keys/shell`（启动按压），`files/storages`（路径 `./1.png` 相对项目根），`http/websocket/axios/mqtt`（联网），`images/colors/canvas`（视觉），`engines/events/threads/timers`（多脚本调度），`floaty/ui/dialogs`（交互），`ocr/keyboard/media/sensors/shizuku` 按需。

## 5. Agent 闭环：写→推→验→调（核心）

`autox-cli` 无日志回传，所以每次 `run` 必须自带“探针”：

```js
// v6 回调用模板（手机 -> 电脑 <电脑IP>:<回调端口>）
try {
  auto(); // 按需
  var r = http.get("http://<电脑IP>:<回调端口>/ping?from=autox&t=" + Date.now());
  toast("回调:" + r.statusCode);
} catch (e) { toast("失败:" + e); console.log(e); }
```

- **结果回传**：`http.get/postJson` 把 `JSON.stringify({ok, data, err, stack})` 打到本机监听器；大图用 `images.toBase64(img,"jpg",50)` 分段 post。
- **布局回传**：`className("TextView").find().slice(0,50).map(w=>({t:w.text(), d:w.desc(), b:w.bounds()}))` post 回来再决定选择器，替代人眼看布局分析。
- **截图回传**：`captureScreen()` → `toBase64` → post，本机解码看；`requestScreenCapture` 失败直接 `exit()` 别空转。
- **收尾**：调完 `engines.stopAll()` 清残留；长循环必加 `sleep` + 超时 `exit()`，`RootAutomator` 在 `events.on('exit')` 里 `ra.exit()`。
- **推前先 `node --check` 过语法**，省一次手机往返；回传 JSON 把摘要字段（`mode/count/stopped`）放 `items` 前面（序列化按插入序，大包被服务端截断时头部仍有结论）。自建回调服务调大单包上限（如 200KB）或分包，大列表（~100 条即 50-70KB）否则必被截断。

标准 SOP：需求 → 选 `.js/.mjs` → 写最小探针（含回调）→ `autox run --devip` → 看本机 HIT/回传 JSON → 补选择器/坐标 → dump 校验 → 沉淀正式脚本。

## 6. 通用工作习惯建议

- 先写连通性探针（toast/回调用），跑通再写正式脚本；单文件用 `run 文件`，多文件项目用 `run 目录/`（配 `project.json` 指定 `main`，V7 项目设 `useNodejs`）。
- 设备 IP 变化时先重扫：同网段扫 `:9317` + `?type=getip`，再跑 `autox run`。示例中的 `<回调端口>/ping` 等占位符不要填真实值入库。

## 7. 已知坑

- `操作完成` ≠ 脚本 OK；无回调一律视为未验证。
- V6/V7 API 不互通，抄文档先看清 `rhino/` 还是 `nodejs/`；V7 文档在线缺失，以 `autox-v7-api` 类型为准。
- 明文 HTTP 无鉴权，仅放可信局域网；`Tap/Swipe` 大写版异步无返回，配 `sleep`。
- 断连分层诊断：`ping` 通但 `:9317` 拒绝 = 手机在、AutoX 服务没起（重开应用/调试服务）；`502`/`timeout(28)` = 网络瞬断，先 `getip` 重验再 `run`，固定加一次重试。USB 共享网络下手机 IP 即网关 IP（`ipconfig getpacket en0` 看 `router`），先认网关再扫网段。
- `node.id()` 无 id 时可能返回 `"null"` 字符串，过滤；id 字面含义不可信（实测某 `tv_time` 实为薪资行），以 `id→字段` 实测映射为准、文本正则只做兜底；`[...]` 文本不一定是状态标签，以带状态 id 的节点为准。
