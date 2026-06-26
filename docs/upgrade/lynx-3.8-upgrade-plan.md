# Sparkling Lynx 3.7 → 3.8 升级计划

> 目标仓库：`fulisadawang/sparkling`  
> 源仓库：`tiktok/sparkling`  
> 计划分支：`upgrade/lynx-3.8-plan`  
> 生成日期：2026-06-27  
> 目标：把 Sparkling 当前锁定的 Lynx 3.7 运行时 / 类型 / iOS Pod / Android Maven 依赖升级到 Lynx 3.8，并保留可回滚路径。

---

## 1. 结论先行

这次升级不应该只改一个版本号。Sparkling 现在同时存在四类 Lynx 版本入口：

1. Android Maven 运行时：`org.lynxsdk.lynx:*`，当前锁在 `3.7.0`。
2. iOS CocoaPods 运行时：`Lynx / LynxBase / LynxServiceAPI / LynxDevtool / PrimJS / XElement / LynxService`，当前锁在 `3.7.0`。
3. 前端类型包：`@lynx-js/types`，当前锁在 `3.7.0` 或 `^3.7.0`。
4. 编译链 / ReactLynx 工具链：`@lynx-js/react`、`@lynx-js/rspeedy`、`@lynx-js/react-rsbuild-plugin`、`@lynx-js/qrcode-rsbuild-plugin`。

建议分两步升级：

- **第一阶段：运行时 + 类型升级到 3.8.0**。这是本次主目标，改动范围可控。
- **第二阶段：再评估 Rspeedy / ReactLynx 工具链是否需要同步小版本升级**。这些包不是 `3.x` 语义版本，不要盲目把它们和 Lynx Runtime 版本强绑定。

---

## 2. 当前版本盘点

### 2.1 Root workspace

`package.json` 当前：

```json
{
  "name": "sparkling",
  "version": "2.1.0-rc.12",
  "engines": {
    "node": "^22 || ^24",
    "pnpm": ">=10.0.0"
  },
  "packageManager": "pnpm@10.26.0"
}
```

这说明升级验证应该在 Node 22/24 + pnpm 10 环境下做，不建议拿 Node 18/20 直接跑结论。

### 2.2 Playground 前端依赖

文件：`packages/playground/package.json`

当前关键版本：

```json
"dependencies": {
  "@lynx-js/react": "^0.116.2"
},
"devDependencies": {
  "@lynx-js/qrcode-rsbuild-plugin": "^0.4.4",
  "@lynx-js/react-rsbuild-plugin": "^0.12.7",
  "@lynx-js/rspeedy": "^0.13.3",
  "@lynx-js/types": "^3.7.0"
}
```

第一阶段只强制升级：

```diff
- "@lynx-js/types": "^3.7.0"
+ "@lynx-js/types": "^3.8.0"
```

`@lynx-js/react / rspeedy / react-rsbuild-plugin` 先不强行改，等 `pnpm outdated` 和构建结果确认。

### 2.3 Template 前端依赖

文件：`template/sparkling-app-template/package.json`

当前关键版本：

```json
"dependencies": {
  "@lynx-js/react": "^0.116.2"
},
"devDependencies": {
  "@lynx-js/qrcode-rsbuild-plugin": "^0.4.4",
  "@lynx-js/react-rsbuild-plugin": "^0.12.7",
  "@lynx-js/rspeedy": "^0.13.3",
  "@lynx-js/types": "^3.7.0"
}
```

第一阶段只强制升级：

```diff
- "@lynx-js/types": "^3.7.0"
+ "@lynx-js/types": "^3.8.0"
```

### 2.4 Sparkling 内部 TS 包

文件：`packages/sparkling-types/package.json`

```diff
- "@lynx-js/types": "3.7.0"
+ "@lynx-js/types": "3.8.0"
```

文件：`packages/sparkling-method/package.json`

```diff
- "@lynx-js/types": "3.7.0"
+ "@lynx-js/types": "3.8.0"
```

说明：内部库建议使用精确版本 `3.8.0`，避免发布产物在 CI 和本地出现类型差异。

### 2.5 Android Runtime

文件：

- `packages/playground/android/gradle/libs.versions.toml`
- `template/sparkling-app-template/android/gradle/libs.versions.toml`

当前：

```toml
lynxSdk="3.7.0"
primjs="3.7.0"
```

目标：

```diff
- lynxSdk="3.7.0"
- primjs="3.7.0"
+ lynxSdk="3.8.0"
+ primjs="3.8.0"
```

受影响 Android artifacts：

```toml
lynx = { group = "org.lynxsdk.lynx", name = "lynx", version.ref = "lynxSdk" }
lynx-devtool = { module = "org.lynxsdk.lynx:lynx-devtool", version.ref = "lynxSdk" }
lynx-jssdk = { group = "org.lynxsdk.lynx", name = "lynx-jssdk", version.ref = "lynxSdk" }
lynx-trace = { group = "org.lynxsdk.lynx", name = "lynx-trace", version.ref = "lynxSdk" }
primjs = { group = "org.lynxsdk.lynx", name = "primjs", version.ref = "primjs" }
lynx-service-image = { group = "org.lynxsdk.lynx", name = "lynx-service-image", version.ref = "lynxSdk" }
lynx-service-log = { group = "org.lynxsdk.lynx", name = "lynx-service-log", version.ref = "lynxSdk" }
lynx-service-http = { group = "org.lynxsdk.lynx", name = "lynx-service-http", version.ref = "lynxSdk" }
lynx-service-devtool = { group = "org.lynxsdk.lynx", name = "lynx-service-devtool", version.ref = "lynxSdk" }
lynx-processor = { group = "org.lynxsdk.lynx", name = "lynx-processor", version.ref = "lynxSdk" }
lynx-xelement = { group = "org.lynxsdk.lynx", name = "xelement", version.ref = "lynxSdk" }
lynx-xelement-input = { group = "org.lynxsdk.lynx", name = "xelement-input", version.ref = "lynxSdk" }
```

第一阶段不要新增额外 Maven 包，避免把“升级 Runtime”变成“引入新能力”。如果后面要使用 markdown、svg、title-bar-view、refresh 等新元素，再单独做能力引入。

### 2.6 iOS Runtime

文件：`packages/playground/ios/Podfile`

当前：

```ruby
pod 'Lynx', '3.7.0'
pod 'LynxBase', '3.7.0'
pod 'LynxServiceAPI', '3.7.0'
pod 'LynxDevtool', '3.7.0'
pod 'PrimJS', '3.7.0'
pod 'XElement', '3.7.0'
pod 'LynxService', '3.7.0'
```

目标：

```diff
- pod 'Lynx', '3.7.0'
- pod 'LynxBase', '3.7.0'
- pod 'LynxServiceAPI', '3.7.0'
- pod 'LynxDevtool', '3.7.0'
- pod 'PrimJS', '3.7.0'
- pod 'XElement', '3.7.0'
- pod 'LynxService', '3.7.0'
+ pod 'Lynx', '3.8.0'
+ pod 'LynxBase', '3.8.0'
+ pod 'LynxServiceAPI', '3.8.0'
+ pod 'LynxDevtool', '3.8.0'
+ pod 'PrimJS', '3.8.0'
+ pod 'XElement', '3.8.0'
+ pod 'LynxService', '3.8.0'
```

文件：`template/sparkling-app-template/ios/Podfile`

当前：

```ruby
pod 'Lynx', '3.7.0'
pod 'PrimJS', '3.7.0'
pod 'LynxService', '3.7.0'
```

目标：

```diff
- pod 'Lynx', '3.7.0'
- pod 'PrimJS', '3.7.0'
- pod 'LynxService', '3.7.0'
+ pod 'Lynx', '3.8.0'
+ pod 'PrimJS', '3.8.0'
+ pod 'LynxService', '3.8.0'
```

### 2.7 iOS DebugTool podspec

文件：`packages/sparkling-debug-tool/ios/Sparkling-DebugTool.podspec`

当前：

```ruby
s.dependency 'Lynx', '~> 3.7.0'
s.dependency 'LynxService/Devtool', '~> 3.7.0'
s.dependency 'LynxDevtool/Framework', '~> 3.7.0'
```

目标：

```diff
- s.dependency 'Lynx', '~> 3.7.0'
- s.dependency 'LynxService/Devtool', '~> 3.7.0'
- s.dependency 'LynxDevtool/Framework', '~> 3.7.0'
+ s.dependency 'Lynx', '~> 3.8.0'
+ s.dependency 'LynxService/Devtool', '~> 3.8.0'
+ s.dependency 'LynxDevtool/Framework', '~> 3.8.0'
```

---

## 3. 3.8 能力变化需要关注什么

Lynx `@lynx-js/types` 3.8.0 相比 3.7.0 的核心变化包括：

- 新增 memory event。
- 更新 image typings。
- 新增 `title-bar-view` 元素类型。
- 新增 `refresh` 元素类型。
- `requestResourcePrefetch` 新增 `font` 类型。
- 新增 CSS 属性 `-x-auto-font-size-line-ranges`。
- `frame` 元素新增 `auto-width` / `auto-height`。

这些大多是“新增能力 / typings 增强”，不是必须大规模改业务代码。但升级后要重点验证：

1. 已有 image 页面是否类型和运行时行为一致。
2. 已有 frame 页面是否没有被 `auto-width / auto-height` 默认行为影响。
3. 使用下拉刷新、标题栏、资源预取的页面是否可以开始按 3.8 类型补齐。
4. 如果项目未来要引入 `refresh` 或 `title-bar-view`，需要单独补 runtime feature demo。

---

## 4. 必须删除 / 收敛的历史 workaround

两个 iOS Podfile 里都有一段 LynxServiceAPI 3.7 的 HEADER_SEARCH_PATHS workaround。

当前注释已经说明：

```ruby
# LynxServiceAPI 3.7.0 ships a podspec whose HEADER_SEARCH_PATHS only
# contains the CI builder's absolute path. service_api/service_api.cc
# does `#include "service_api/service_api.h"`, which needs the pod's
# own root in the search paths to resolve locally.
# This has been fixed upstream in Lynx 3.8.0 — remove this workaround
# once Lynx 3.8 stable is released.
```

升级到 3.8 后应该移除这段 workaround。

处理方式：

### 4.1 `packages/playground/ios/Podfile`

保留 `Local.xcconfig` 注入逻辑，只删除 LynxServiceAPI patch 段：

```ruby
post_install do |installer|
  local_xcconfig = File.expand_path('Local.xcconfig', __dir__)
  if File.exist?(local_xcconfig)
    Dir.glob(File.join(__dir__, 'Pods/Target Support Files/Pods-SparklingGo/Pods-SparklingGo.*.xcconfig')) do |path|
      content = File.read(path)
      include_line = '#include? "../../../Local.xcconfig"'
      unless content.include?(include_line)
        File.open(path, 'a') { |f| f.puts "\n#{include_line}" }
      end
    end
  end
end
```

### 4.2 `template/sparkling-app-template/ios/Podfile`

这里 `post_install` 只有 LynxServiceAPI workaround，升级后可以删除整个 `post_install` 块。

---

## 5. 建议提交拆分

不要一次性全改，建议拆 5 个 commit：

### Commit 1：docs: add Lynx 3.8 upgrade plan

只新增本文档。

### Commit 2：chore(android): bump Lynx runtime to 3.8.0

修改：

- `packages/playground/android/gradle/libs.versions.toml`
- `template/sparkling-app-template/android/gradle/libs.versions.toml`

只改：

```toml
lynxSdk="3.8.0"
primjs="3.8.0"
```

### Commit 3：chore(ios): bump Lynx pods to 3.8.0

修改：

- `packages/playground/ios/Podfile`
- `template/sparkling-app-template/ios/Podfile`
- `packages/sparkling-debug-tool/ios/Sparkling-DebugTool.podspec`

同时移除 3.7 专用 workaround。

### Commit 4：chore(types): bump @lynx-js/types to 3.8.0

修改：

- `packages/playground/package.json`
- `template/sparkling-app-template/package.json`
- `packages/sparkling-types/package.json`
- `packages/sparkling-method/package.json`
- `pnpm-lock.yaml`

### Commit 5：test: add 3.8 smoke verification notes / fixtures

如果发现 3.8 行为变化，再补最小 demo 或测试，不做无关重构。

---

## 6. 推荐本地执行命令

### 6.1 准备分支

```bash
git clone https://github.com/fulisadawang/sparkling.git
cd sparkling
git checkout -b upgrade/lynx-3.8
pnpm install
```

### 6.2 修改版本号

推荐先用搜索确认所有 3.7 入口：

```bash
rg "3\.7\.0|\^3\.7\.0|~> 3\.7\.0" .
```

只改和 Lynx 相关的版本，不要误改业务版本、示例版本或文档里历史说明。

### 6.3 更新 lockfile

```bash
pnpm install --lockfile-only
pnpm install
```

如果 `@lynx-js/types@3.8.0` 不存在，停止升级，不要用 3.9/4.0 顶替；先确认官方包发布状态。

### 6.4 TS / 前端构建验证

```bash
pnpm -r build
pnpm coverage:ts
pnpm --filter sparkling-playground test
pnpm --filter sparkling-app-template test
```

重点看：

- `sparkling-types` 是否能通过 `tsc`。
- `sparkling-method` 是否能通过 `tsc`。
- `packages/playground` 是否能过 `rspeedy build`。
- `template/sparkling-app-template` 是否能过 `sparkling-app-cli build --copy`。

### 6.5 Android 验证

```bash
cd packages/playground/android
./gradlew --refresh-dependencies
./gradlew assembleDebug
```

再验证 template：

```bash
cd ../../../template/sparkling-app-template/android
./gradlew --refresh-dependencies
./gradlew assembleDebug
```

重点看：

- `org.lynxsdk.lynx:lynx:3.8.0` 是否能解析。
- `org.lynxsdk.lynx:primjs:3.8.0` 是否能解析。
- `xelement` / `xelement-input` 3.8.0 是否存在。
- `lynx-service-*` 3.8.0 是否存在。
- ProGuard / R8 是否因为 3.8 新增类出现 keep rule 问题。

### 6.6 iOS 验证

```bash
cd packages/playground/ios
bundle exec pod repo update || pod repo update
bundle exec pod install || pod install
xcodebuild \
  -workspace SparklingGo.xcworkspace \
  -scheme SparklingGoInHouse \
  -configuration Debug \
  -sdk iphonesimulator \
  build
```

再验证 template：

```bash
cd ../../../template/sparkling-app-template/ios
bundle exec pod repo update || pod repo update
bundle exec pod install || pod install
xcodebuild \
  -workspace SparklingGo.xcworkspace \
  -scheme SparklingGoInHouse \
  -configuration Debug \
  -sdk iphonesimulator \
  build
```

重点看：

- Pod 是否能解析 `Lynx 3.8.0`、`PrimJS 3.8.0`、`LynxService 3.8.0`。
- 移除 LynxServiceAPI workaround 后是否仍能编译。
- `LynxDevtool` / `DebugRouter` 是否仍能在 InHouse target 正常链接。
- Release target 是否没有意外带入 devtool 符号。

---

## 7. 功能回归清单

### 7.1 Sparkling 核心链路

- App 冷启动进入 Lynx 首页。
- App 从 native 打开 Lynx 页面。
- Lynx 页面内路由跳转。
- Lynx 页面返回 native。
- 多页面 nav chain demo。
- Scheme builder / scheme presets demo。
- GlobalProps 注入和读取。
- initData 注入和读取。

### 7.2 Method / Bridge

- `sparkling-method` iOS module 注册。
- `sparkling-method` Android module wrapper 注册。
- Promise / callback 返回。
- 错误码透传。
- method pipe 在 LynxView 生命周期销毁后不回调 crash。

### 7.3 Media / Storage / Navigation

- media choose。
- media upload。
- media download。
- storage get / set / remove。
- navigation push / pop / replace。
- navigation 链路跨页面参数传递。

### 7.4 UI Runtime

- image 加载、失败、占位、gif / webp。
- input / textarea 输入、焦点、键盘弹起。
- scroll-view 滚动。
- list 大数据滚动。
- frame 嵌套页面。
- xelement / xelement-input。

### 7.5 Debug / Devtool

- Debug panel 可以打开。
- Console log 可以输出。
- QRCode dev server 连接。
- InHouse target 带 devtool。
- Release target 不带 DebugRouter / LynxDevtool。

---

## 8. 风险点和处理策略

### 风险 1：Android 某些 3.8 Maven artifact 未发布

表现：Gradle resolve 失败。

处理：

1. 不要把某个 artifact 偷偷降回 3.7，避免运行时混版。
2. 先确认 `lynxSdk`、`primjs`、`xelement`、`xelement-input` 是否全部有 3.8.0。
3. 如果只有部分 3.8 缺失，本次升级暂停，保留 3.7。

### 风险 2：iOS Pod 3.8 缺少某个 subspec

表现：`pod install` 报 subspec 不存在。

处理：

1. 保持版本统一。
2. 先查官方 podspec。
3. 不要把 `Lynx` 升 3.8、`LynxService` 留 3.7。

### 风险 3：LynxServiceAPI workaround 删除后仍有 header 问题

表现：iOS 编译找不到 `service_api/service_api.h`。

处理：

1. 先确认 Pod cache 是否是 3.8。
2. 清理缓存后再装：

```bash
rm -rf Pods Podfile.lock ~/Library/Caches/CocoaPods
pod repo update
pod install
```

3. 如果 3.8 仍失败，说明官方 podspec 可能未完全修复；临时保留 workaround，但注释必须改成 `TODO: remove after verified fixed in 3.8.x`。

### 风险 4：`@lynx-js/types` 3.8 暴露出历史代码类型错误

表现：TS 编译失败。

处理：

1. 优先修真实类型错误。
2. 不要用 `skipLibCheck` 掩盖项目类型错误。
3. 如果是第三方类型声明问题，单独建 `types/lynx-3.8-compat.d.ts` 做最小 shim。

### 风险 5：Rspeedy / ReactLynx 工具链和 3.8 types 不兼容

表现：`rspeedy build` 或 ReactLynx 编译失败。

处理：

1. 先确认是否只是 types 问题。
2. 再跑：

```bash
pnpm outdated "@lynx-js/*"
```

3. 如果需要同步升级，单独提交：

```bash
pnpm update @lynx-js/react @lynx-js/rspeedy @lynx-js/react-rsbuild-plugin @lynx-js/qrcode-rsbuild-plugin --latest
```

4. 不要和 Runtime 版本升级混在同一个 commit。

---

## 9. 回滚方案

如果升级失败，按平台回滚：

### 9.1 Android 回滚

```diff
- lynxSdk="3.8.0"
- primjs="3.8.0"
+ lynxSdk="3.7.0"
+ primjs="3.7.0"
```

然后：

```bash
./gradlew --refresh-dependencies assembleDebug
```

### 9.2 iOS 回滚

把所有 Lynx / PrimJS / XElement / LynxService 相关 Pod 从 `3.8.0` 改回 `3.7.0`，并恢复 LynxServiceAPI workaround。

### 9.3 JS 类型回滚

```diff
- "@lynx-js/types": "^3.8.0"
+ "@lynx-js/types": "^3.7.0"
```

内部包：

```diff
- "@lynx-js/types": "3.8.0"
+ "@lynx-js/types": "3.7.0"
```

然后：

```bash
pnpm install --lockfile-only
pnpm -r build
```

---

## 10. 最小可接受完成标准

这次升级只有同时满足下面条件才算完成：

- Android playground 可以 `assembleDebug`。
- Android template 可以 `assembleDebug`。
- iOS playground `SparklingGoInHouse` 可以编译。
- iOS template `SparklingGoInHouse` 可以编译。
- `pnpm -r build` 通过。
- `pnpm coverage:ts` 通过。
- playground 首页、nav-chain、media、storage、scheme、debug-panel 至少完成一轮真机或模拟器 smoke test。
- 版本不混用：不能出现 Lynx 3.8 + PrimJS 3.7 或 Android 3.8 + iOS 3.7 的长期状态。

---

## 11. 后续可做但不放进本次升级的事情

下面这些都不是 3.7 → 3.8 的必要条件，建议后置：

1. 引入 `refresh` 组件 demo。
2. 引入 `title-bar-view` demo。
3. 基于 memory event 做内存压力降级策略。
4. 对 image 新 typings 做更严格的封装。
5. 升级到 Lynx 3.9 / 4.0。
6. 重构 Sparkling navigation 协议。
7. 重构 BottomSheet / Transition Runtime。

---

## 12. 建议最终 PR 描述

```md
## Summary

Upgrade Sparkling's Lynx runtime and typings from 3.7.0 to 3.8.0.

## Changes

- Bump Android `org.lynxsdk.lynx:*` runtime versions to 3.8.0.
- Bump iOS Lynx / PrimJS / LynxService pods to 3.8.0.
- Bump `@lynx-js/types` to 3.8.0.
- Remove LynxServiceAPI 3.7 header search path workaround after validating 3.8.
- Keep ReactLynx / Rspeedy toolchain upgrades separate unless required by build validation.

## Validation

- [ ] `pnpm -r build`
- [ ] `pnpm coverage:ts`
- [ ] Android playground `assembleDebug`
- [ ] Android template `assembleDebug`
- [ ] iOS playground `SparklingGoInHouse` build
- [ ] iOS template `SparklingGoInHouse` build
- [ ] Playground smoke test: home / nav / media / storage / scheme / debug panel
```
