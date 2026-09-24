# BMS v2 — Setup Guide

> 本目录 = BMS v2（`mgd-bms-v2`），从 v1（`~/Desktop/mgd-bms-github/`）复制改造而来。
> **v1 继续正常使用，不受影响。** v2 完工后再把 v1 数据迁过来。

## 已完成（本地骨架）
- [x] 从 v1 复制 `index.html`（980 KB，单文件）
- [x] `firebaseConfig` 换成 v2 项目占位（`projectId: mgd-bms-v2`）
- [x] 加安全开关 `V2_CLOUD_READY` —— **apiKey 未填前，云同步全关**，绝不碰 v1 数据
- [x] 访问密码改为 `MGD0088`
- [x] 标题加 v2 标识
- [x] 10 个 script 块语法校验通过；零 v1 残留引用

## 待办

### ① 建 Firebase 项目（Tony 操作，用 Google 账号）
1. 打开 https://console.firebase.google.com/ → **Add project**
2. 项目名填 `mgd-bms-v2`（Project ID 会显示为 `mgd-bms-v2`，若被占用加后缀，记得回来同步改）
3. 建好后：左侧 **Build → Firestore Database → Create database**
   - 位置选 **asia-southeast1 (Singapore)**
   - 模式选 **Start in production mode**（和 v1 一致）
4. 左侧 **Project settings（齿轮）→ General → Your apps → 添加 Web 应用**（图标 `</>`）
   - 昵称随意，如 `bms-v2-web`
   - 复制生成的 `firebaseConfig` 对象

### ② 把 config 贴进 index.html
打开 `~/Desktop/mgd-bms-v2/index.html`，找到约 **7090 行**的：

```js
var firebaseConfig = {
  apiKey: "__PASTE_V2_API_KEY__",              // ← 换成真实 apiKey
  authDomain: "mgd-bms-v2.firebaseapp.com",     // ← 核对
  projectId: "mgd-bms-v2",                      // ← 核对
  storageBucket: "mgd-bms-v2.firebasestorage.app", // ← 核对
  messagingSenderId: "__PASTE_SENDER_ID__",     // ← 换成真实值
  appId: "__PASTE_APP_ID__"                     // ← 换成真实值
};
```

**只要 `apiKey` 换掉，`V2_CLOUD_READY` 会自动变 `true`，云同步自动开启。**

### ③ Firestore 安全规则（与 v1 相同，允许匿名读写）
Firestore → Rules，参考 v1 项目 `mgd-bms` 的规则（test mode 或允许读写）。

### ④ 建 GitHub 仓库 + 上线
```sh
cd ~/Desktop/mgd-bms-v2
git init
git add index.html SETUP.md
git commit -m "BMS v2: initial shell copied from v1 (cloud isolated, password MGD0088)"
gh repo create Tony80-del/mgd-bms-v2-online --public --source=. --push
# 再到 GitHub 仓库 Settings → Pages → Source: main / (root) 开启 Pages
```
线上地址将是：`https://tony80-del.github.io/mgd-bms-v2-online/`

### ⑤ 数据迁移（v2 完工后，一次性）
v1 云数据：Firestore 项目 `mgd-bms` → 文档 `bms_data/master`
v2 云数据：Firestore 项目 `mgd-bms-v2` → 文档 `bms_data/master`（同名结构）

因为 **v2 沿用 v1 的数据结构（id / data key 不变）**，迁移 = 把 v1 `master.data` 整个 payload 写进 v2 `master.data`，然后核对条数一致。

## 关键安全点
- v2 的云操作绑定的是 `firebaseConfig.projectId` —— 已锁定 `mgd-bms-v2`
- `V2_CLOUD_READY` 为 false 时：`saveData` / `listenToCloud` / `syncFromCloud` 全部直接 return
- **任何时候都不要把 v2 的 config 换回 `mgd-bms`**
