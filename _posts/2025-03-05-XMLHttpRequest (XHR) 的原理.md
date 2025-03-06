---
layout:     post   				    # 使用的布局（不需要改）
title:      XMLHttpRequest (XHR) 的原理 				# 标题 
subtitle:   #副标题
date:       2025-03-25 				# 时间
author:     BY chenqingyun					# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - XHR
---
## **📌 XMLHttpRequest (XHR) 的原理**
`XMLHttpRequest`（简称 XHR）是浏览器提供的 **AJAX (Asynchronous JavaScript and XML) API**，用于在 **不刷新页面的情况下与服务器通信**，实现 **异步 HTTP 请求**。

虽然 `fetch` 现在更流行，但 `XHR` 仍然是底层的核心技术，许多库（如 `axios`）都基于 `XHR` 进行封装。

---

## **📌 XHR 运行流程**
### **1️⃣ 创建 XHR 对象**
```js
const xhr = new XMLHttpRequest();
```
浏览器会创建一个 `XMLHttpRequest` 实例，提供 `open`、`send` 等方法。

---

### **2️⃣ 初始化请求 (`open` 方法)**
```js
xhr.open("GET", "https://api.example.com/data", true);
```
- **第 1 个参数**：请求方法（`GET`、`POST`、`PUT` 等）
- **第 2 个参数**：请求 URL
- **第 3 个参数**：是否异步（`true`=异步，`false`=同步，通常使用 `true`）

---

### **3️⃣ 设置请求头**
```js
xhr.setRequestHeader("Content-Type", "application/json");
```
如果请求需要额外的 HTTP 头部（如 `Authorization`），可以用 `setRequestHeader` 添加。

---

### **4️⃣ 监听请求状态 (`onreadystatechange` / `onload`)**
```js
xhr.onreadystatechange = function () {
  if (xhr.readyState === 4 && xhr.status === 200) {
    console.log("Response:", xhr.responseText);
  }
};
```
- **`readyState` 代表请求的不同阶段：**
  | 状态值 | 含义 |
  |--------|----------------|
  | `0` | **未初始化**（还未调用 `open`） |
  | `1` | **已打开**（调用 `open` 但未发送） |
  | `2` | **已发送**（调用 `send`） |
  | `3` | **交互中**（部分数据接收中） |
  | `4` | **请求完成**（数据已接收完毕） |

- **`status` 代表 HTTP 状态码：**
  | 状态码 | 含义 |
  |--------|----------------|
  | `200` | 成功 |
  | `400` | 请求错误 |
  | `401` | 未授权 |
  | `403` | 禁止访问 |
  | `404` | 资源未找到 |
  | `500` | 服务器错误 |

---

### **5️⃣ 发送请求**
```js
xhr.send();
```
如果是 `POST` 请求：
```js
xhr.send(JSON.stringify({ name: "Alice" }));
```
- `GET` 请求通常不需要 `send` 传数据
- `POST` 需要 `send` 发送 `body`

---

### **6️⃣ 解析返回数据**
```js
xhr.onload = function () {
  if (xhr.status >= 200 && xhr.status < 300) {
    console.log("成功:", JSON.parse(xhr.responseText));
  } else {
    console.error("请求失败:", xhr.status);
  }
};
```
- **`responseText`**：返回的文本数据（字符串）
- **`response`**：可解析为 `JSON` 或 `Blob`

---

## **📌 XHR 的拦截原理**
你可以**劫持 XHR**，拦截所有 AJAX 请求。

### **✅ 劫持 `XMLHttpRequest.prototype.open`**
```js
const originalOpen = XMLHttpRequest.prototype.open;
XMLHttpRequest.prototype.open = function (method, url, ...args) {
  console.log("Intercepted XHR:", method, url);
  return originalOpen.apply(this, [method, url, ...args]);
};
```
- 这样所有 `XHR` 请求都会被拦截，并可以修改 URL 或其他参数。

### **✅ 劫持 `XMLHttpRequest.prototype.send`**
```js
const originalSend = XMLHttpRequest.prototype.send;
XMLHttpRequest.prototype.send = function (body) {
  console.log("Request Data:", body);
  return originalSend.apply(this, arguments);
};
```
- 这样可以捕获 `POST` 请求的 `body` 数据。

---

## **📌 `XMLHttpRequest` 的优缺点**
### **✅ 优点**
- **兼容性好**（支持所有现代浏览器）
- **支持同步和异步**（`fetch` 只支持异步）
- **支持 `abort()` 取消请求**

### **❌ 缺点**
- **回调地狱**（多个请求嵌套时不易管理，`fetch` 解决了这个问题）
- **不支持 `Promise`**（`fetch` 默认支持 `Promise`）
- **CORS 限制**（如果服务器不允许跨域，浏览器会阻止请求）

---

## **📌 `XHR` vs `fetch`**
| 特性 | `XMLHttpRequest` | `fetch` |
|------|-----------------|---------|
| **兼容性** | **广泛支持** | 现代浏览器支持 |
| **Promise 支持** | ❌ 需要手动封装 | ✅ 天生支持 |
| **简洁性** | 复杂 | 更简洁 |
| **拦截方式** | `prototype` 劫持 | `window.fetch = ...` |
| **支持 `abort()`** | ✅ 有 `abort()` | ❌ 需 `AbortController` |
| **进度监听** | ✅ `onprogress` | ❌ 需 `ReadableStream` |
| **默认不发送 Cookie** | ❌ 需要手动 `withCredentials` | ✅ `credentials: "include"` |

---

## **📌 总结**
1. `XMLHttpRequest` 是浏览器提供的 **底层 AJAX API**。
2. **请求流程**：`open()` → `setRequestHeader()` → `send()` → `onreadystatechange/onload` 处理返回数据。
3. **可以劫持 `open/send`** 来拦截请求数据，适用于监控或修改请求。
4. `fetch` 是 `XHR` 的现代替代方案，但 `XHR` 仍然在老旧代码和某些特定场景下有用。

🚀 **现在你已经掌握了 `XMLHttpRequest` 的原理和拦截方式！**
