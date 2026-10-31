# qx-apple-cert-block

[![GitHub stars](https://img.shields.io/github/stars/yebuwudong/qx-apple-cert-block)](https://github.com/yebuwudong/qx-apple-cert-block)

**Apple 证书屏蔽验证 · Quantumult X 版**

精准屏蔽 Apple p12 证书验证，让已过期证书能够继续使用。

移植自 [salem-2007/apple-cert-block](https://github.com/salem-2007/apple-cert-block) 的 Loon 插件 `AntiRevoke.plugin`。

---

## 一键订阅（推荐）

```
https://raw.githubusercontent.com/yebuwudong/qx-apple-cert-block/main/apple-cert-block.list
```

**导入方式：** Quantumult X → 分流 → 引用 → 右上角 `+` → 粘贴上面的链接 → 标签随意填 → 保存 → 开启右上角开关。

无需配置 MITM，无需证书，导入即生效。

> 如果想用 CDN 加速，把前缀换成 `https://cdn.jsdelivr.net/gh/yebuwudong/qx-apple-cert-block@main/` 即可，文件名不变。

---

## 原理

现代系统在建立 HTTPS 连接时，除了校验证书是否由可信 CA 签发，还会在线查询证书的**当前状态**：

- **OCSP**：实时查询证书是否被吊销
- **CRL**：下载证书吊销列表
- **Apple 自有验证接口**：`valid.apple.com` / `ppq.apple.com` 等

证书被吊销或过期后，这些接口会返回"状态异常"，系统随即中断 TLS 握手，导致过期 p12 证书无法使用。

屏蔽上述验证域名后，验证请求发不出去，系统降级为"无法获知状态"的直接信任，证书得以继续使用。

本规则使用 `HOST, xxx, reject`，QX 在 DNS 层直接返回 `127.0.0.1`，连接根本不会建立，因此**不需要开 MITM 也不需要装根证书**。

---

## 文件说明

| 文件 | 用途 |
| --- | --- |
| `apple-cert-block.list` | 分流规则远程资源，一条链接直接用 |
| `apple-cert-block.qx.conf` | 完整配置片段，可手动贴进 QX 配置文件 |
| `mitm/apple-cert-block-hostname.txt` | MITM 主机名列表（本方案不需要，留作其他用途） |
| `LICENSE` | Apache-2.0，继承自上游 |

---

## 屏蔽的域名（17 条）

| 分类 | 域名 |
| --- | --- |
| Apple OCSP | `ocsp.apple.com` `ocsp2.apple.com` |
| Apple 验证 | `valid.apple.com` `ppq.apple.com` `ppq-ext.v.aaplimg.com` |
| Apple CRL | `crl.apple.com` `certs.apple.com` |
| DigiCert | `ocsp.digicert.com` `ocsp.digicert.cn` `crl3.digicert.com` `crl4.digicert.com` |
| Entrust | `ocsp.entrust.net` `crl.entrust.net` |
| Sectigo / USERTrust | `ocsp.sectigo.com` `crl.sectigo.com` `ocsp.usertrust.com` `crl.usertrust.com` |

---

## 与 Loon 原版的差异

| Loon | Quantumult X |
| --- | --- |
| `[Rule]` | `[filter_local]` |
| `DOMAIN, xxx, REJECT` | `HOST, xxx, reject` |
| `[MITM] hostname = %APPEND% ...` | 本方案省去（`reject` 在 DNS 层生效，不走 MITM） |

原版 Loon 插件里 `[MITM]` 只列了三个 Apple 域名且带 `%APPEND%`，实际作用是占位而非解密。QX 侧 `reject` 已足够，故不提供 MITM 段，减少对正常流量的干扰。

---

## 注意事项

- 仅影响证书**状态查询**，不影响证书链校验与正常 TLS 加密。
- 不同 iOS/iPadOS 版本对验证失败的降级策略存在差异，不保证所有场景生效。
- 长期使用可能触发 Apple 的备用验证机制或部分服务提示异常。
- 仅供学习与技术研究，请勿用于商业绕过。

---

## 致谢

- 原插件：[salem-2007/apple-cert-block](https://github.com/salem-2007/apple-cert-block)
- 协议：Apache-2.0
