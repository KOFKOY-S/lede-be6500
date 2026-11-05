# LEDE 云编译套件（JDCloud BE6500 / 赵云 RE-CS-06）

用 GitHub Actions 在线编译 LEDE 固件，产物直接在 Actions 页面下载。

## 编译内容

| 项 | 值 |
|---|---|
| 源码 | coolsnowwolf/lede（master） |
| 目标 | `qualcommbe` / `ipq53xx` |
| 设备 | `jdcloud_re-cs-06`（BE6500，128G） |
| 内核 | 6.18 |
| 无线 | ath12k + `ath12k-firmware-*-ddwrt`（DD-WRT 签名固件） |

## 使用步骤

1. **创建仓库**
   - 在 GitHub 新建一个仓库（建议 **Public**，否则 Actions 分钟数受限）
   - 名称随意，例如 `lede-be6500`

2. **上传本目录全部内容**（保持目录结构）
   ```
   .github/workflows/build.yml
   config.seed
   ```
   - 网页端可直接拖拽整个 `lede-build` 文件夹
   - 注意 `.github` 是隐藏目录，确保上传成功

3. **触发编译**
   - 仓库页面 → **Actions** → 左侧 `Build LEDE for JDCloud BE6500` → **Run workflow**
   - 或直接 push 一次（`on: push` 已配置）

4. **等待完成**
   - 首次约 **2–5 小时**（GitHub 单任务上限 6 小时）
   - 状态变绿后，进入该次运行页面，底部 **Artifacts** 下载 `lede-jdcloud-be6500`

5. **产物说明**
   ```
   lede-qualcommbe-ipq53xx-jdcloud_re-cs-06-squashfs-sysupgrade.bin
   lede-qualcommbe-ipq53xx-jdcloud_re-cs-06-squashfs-factory.bin（若有）
   *.manifest / sha256sums
   ```

## 刷机

1. 断电 → 按住 reset → 插电，等指示灯变蓝（进入 U-Boot）
2. 电脑设静态 IP `192.168.3.10/24`，浏览器打开 `http://192.168.3.3`
3. 在 **firmware** 页面上传 `*sysupgrade.bin`（**不要**再刷 GPT）
4. 等待完成，绿灯即成功

## 风险提示

- **Wi-Fi 不一定能起来**：LEDE 用 ath12k + DD-WRT 签名固件，理论上比 GWRT 更容易通过安全启动，但不保证；若失败，日志会显示 `qcom-wcss-secure-pil ... error -5`
- **ART / factory 已备份**：刷机不会动 p17/p25
- **可回滚**：QWRT 镜像和分区表备份都在本机 `router-backup`
- 刷机前建议先看一次编译日志里 `jdcloud_re-cs-06` 的 image 生成情况

## 如果编译失败

把 Actions 里失败的步骤日志（最后 100 行）发我，常见原因是：
- feeds 拉取超时（重跑即可）
- 缺依赖包（按报错补 `apt-get install`）
- 6 小时超时（可拆成两步：先 `make download`，再编译，或改用自建 runner）
