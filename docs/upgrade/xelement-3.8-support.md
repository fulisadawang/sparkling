# Lynx 3.8 XElement 支持矩阵与 Sparkling 接入说明

> 目标分支：`upgrade/lynx-3.8-plan`  
> 目标版本：Lynx Runtime / PrimJS / `@lynx-js/types` `3.8.0`

---

## 1. 先给结论

这次 Sparkling 的 XElement 接入不把所有 `develop` 分支上已经存在的实验组件一次性塞进来，而是按 **Lynx 3.8 类型变更 + 当前 Sparkling 已有依赖方式** 做最小可控升级：

- Android 升级 `lynxSdk` / `primjs` 到 `3.8.0`。
- Android 保留并升级已有的 `xelement` / `xelement-input`。
- Android 新增 `xelement-refresh` 显式依赖，因为 `@lynx-js/types` 3.8.0 明确新增 `refresh` element types。
- iOS Playground 升级 `XElement` Pod 到 `3.8.0`。
- iOS Template 原来没有显式接入 `XElement`，这次补上 `pod 'XElement', '3.8.0'`，避免模板项目 Android 可用、iOS 不可用。
- 不把 `viewpager` / `scroll-coordinator` / `webview` / `markdown` / `blur-view` 作为 3.8 接入目标，因为官方 types changelog 显示这些属于 3.9 或 4.0 后续能力。

---

## 2. 3.8 官方类型层新增能力

`@lynx-js/types` 3.8.0 相关变化：

- memory event
- image typings 更新
- `title-bar-view` element types
- `refresh` element types
- `requestResourcePrefetch` 新增 `font` 类型
- CSS 新增 `-x-auto-font-size-line-ranges`
- `frame` 新增 `auto-width` / `auto-height`

其中和 XElement 强相关的是：

| 能力 | 3.8 是否接入 | 说明 |
| --- | --- | --- |
| `refresh` / `refresh-header` | 是 | 3.8 明确新增 element types，需要 Android `xelement-refresh` 与 iOS `XElement`。 |
| `title-bar-view` | 只升级 types/runtime，不额外加 Android xelement 包 | 类型层 3.8 新增，但它偏窗口拖拽区域，类型标注主要面向 ClayWindows / ClayMacOS；移动端 Sparkling 不作为本轮重点。 |
| `image` typings | 是 | 通过 `@lynx-js/types` 和 Lynx Runtime 3.8 继承。 |
| `frame auto-width/auto-height` | 是 | 通过 `@lynx-js/types` 和 Lynx Runtime 3.8 继承。 |

---

## 3. 这次实际接入的 Android artifacts

当前 Sparkling 原本只在 Android version catalog 里声明：

```toml
lynx-xelement = { group = "org.lynxsdk.lynx", name = "xelement", version.ref = "lynxSdk" }
lynx-xelement-input = { group = "org.lynxsdk.lynx", name = "xelement-input", version.ref = "lynxSdk" }
```

本次新增：

```toml
lynx-xelement-refresh = { group = "org.lynxsdk.lynx", name = "xelement-refresh", version.ref = "lynxSdk" }
```

原因：Lynx 官方源码中 Android XElement refresh 模块的 artifact 名称就是 `xelement-refresh`，并且 3.8 types 明确新增 `refresh`。

### Android Playground

文件：`packages/playground/android/app/build.gradle.kts`

```kotlin
implementation(libs.lynx.xelement)
implementation(libs.lynx.xelement.input)
implementation(libs.lynx.xelement.refresh)
```

### Android Template

文件：`template/sparkling-app-template/android/app/build.gradle.kts`

模板之前没有显式接入 XElement，这会导致模板用户天然缺少 input/refresh 相关 runtime。现在补齐：

```kotlin
implementation(libs.lynx.xelement)
implementation(libs.lynx.xelement.input)
implementation(libs.lynx.xelement.refresh)
```

---

## 4. 这次实际接入的 iOS Pods

### iOS Playground

文件：`packages/playground/ios/Podfile`

```ruby
pod 'XElement', '3.8.0'
```

Playground 原来就有 `XElement 3.7.0`，这次升级为 3.8.0。

### iOS Template

文件：`template/sparkling-app-template/ios/Podfile`

```ruby
pod 'XElement', '3.8.0'
```

Template 原来没有 `XElement`，这次补上。这样模板项目使用 `input`、`textarea`、`svg`、`refresh` 这类元素时，iOS 不会天然缺失 XElement runtime。

---

## 5. 3.8 可以认为稳定接入的元素范围

以 3.8 types changelog 和 Sparkling 当前实际依赖为边界，本轮支持范围建议定义为：

| 元素 / 能力 | 前端 types | Android runtime | iOS runtime | 本轮状态 |
| --- | --- | --- | --- | --- |
| `input` | 已存在 | `xelement` + `xelement-input` | `XElement` | 支持 |
| `textarea` | 已存在 | `xelement` + `xelement-input` | `XElement` | 支持 |
| `svg` | 3.7 已引入类型 | `xelement` umbrella 覆盖；必要时可显式补 `xelement-svg` | `XElement` | 保持支持，不新增显式依赖 |
| `overlay` | 3.6 已引入类型 | `xelement` umbrella 覆盖 | `XElement` | 保持支持，不新增显式依赖 |
| `refresh` | 3.8 新增 | 显式新增 `xelement-refresh` | `XElement` | 新增支持 |
| `refresh-header` | 3.8 新增 | 显式新增 `xelement-refresh` | `XElement` | 新增支持 |
| `title-bar-view` | 3.8 新增 | 不额外接 Android 包 | 不额外接 iOS 包 | types/runtime 升级，但移动端不作为本轮重点 |

---

## 6. 暂不纳入 3.8 的能力

下面这些在当前 `develop` 源码里能看到，但不要混入本次 3.8 升级：

| 能力 | 原因 |
| --- | --- |
| `viewpager` / `viewpager-item` | 官方 types changelog 显示 `viewpager` 是 3.9 引入。 |
| `scroll-coordinator` 系列 | 官方 types changelog 显示 `scroll-coordinator` 是 3.9 引入。 |
| `webview` | 官方 types changelog 显示 `webview` 是 4.0 引入。 |
| `markdown` | 官方 types changelog 显示 `markdown` 是 4.0 引入。 |
| `blur-view` | 官方 types changelog 显示 `blur-view` 是 4.0 引入。 |
| `video` | 当前 develop 有 video 相关类型和 artifact，但 3.8 changelog 未声明为 3.8 能力，本轮不引入。 |

---

## 7. 推荐 smoke test 页面

升级后建议在 Playground 或 template 里补一个最小页面，验证 3.8 XElement runtime 是否完整：

```tsx
export function XElementSmoke() {
  return (
    <page>
      <view className="xelement-smoke">
        <input placeholder="input smoke" />
        <textarea placeholder="textarea smoke" />
        <refresh enable-refresh bindstartrefresh={() => {}}>
          <refresh-header />
          <scroll-view>
            <text>refresh smoke</text>
          </scroll-view>
        </refresh>
      </view>
    </page>
  );
}
```

验证点：

- Android / iOS 都能启动页面。
- `input` 可以 focus、输入、失焦。
- `textarea` 可以多行输入。
- `refresh` 下拉能触发 `bindstartrefresh`。
- 调用 `finishRefresh` 后刷新态可以结束。

---

## 8. 后续如果要继续扩展

后续可以单独开 PR 做：

1. `xelement-svg` 显式依赖化：如果发现 umbrella `xelement` 的传递依赖在 Gradle 发布包里不稳定，就把 `xelement-svg` 也显式加入 catalog。
2. 3.9 升级：再接 `viewpager`、`scroll-coordinator`。
3. 4.0 升级：再接 `webview`、`markdown`、`blur-view`。
4. 为每个 XElement 补独立 demo 和端能力差异表。
