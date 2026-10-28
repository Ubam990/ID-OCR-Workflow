---
name: image-ocr
description: 从图片中提取文字（OCR）。当用户提供图片并要求识别/提取其中文字时使用，适用于身份证、营业执照、截图、文档照片等任意图片。
---

# 图片文字提取（OCR）

## 适用场景

用户提供一张或多张图片，需要提取图片中的文字内容。图片类型不限（身份证、营业执照、截图、文档照片、票据等）。

## OCR 服务

使用在线 OCR 工具 **https://ocr.wdku.net/**，通过 `curl.exe` 模拟网页上传请求完成。

## 操作步骤

### 1. 上传图片

用 `curl.exe -F` 逐张上传：

```bash
curl.exe -s -X POST -F "file=@<文件路径>" "https://ocr.wdku.net/upload?convtype=ocr"
```

返回 `{"errno":0,"data":{"id":"<文件ID>","time":<时间戳>}}`，**记录每张图片的 `id` 和 `time`**。

### 2. 合并提交转换

所有图片**合并为一个请求**：

```bash
curl.exe -s -X POST -d "ids=<ID1>,<ID2>&ts=<time1>,<time2>&paytype=free&core=all&saveformat=txt&lang=1,2" "https://ocr.wdku.net/index"
```

- `ids` / `ts` 按顺序一一对应，逗号分隔
- **不要**带多余参数（如 `resolution`），否则报错
- 返回 `{"errno":0,"id":"<转换ID>","time":<转换时间>}`

### 3. 轮询状态

先 `sleep 20` 等服务器处理，再查询：

```bash
sleep 20
curl.exe -s "https://ocr.wdku.net/get_conv_convert_status?id=<转换ID>&time=<转换时间>"
```

- `status=0` → 转换中，继续轮询（每次间隔 ≥10 秒）
- `status=1` → 完成

### 4. 下载并读取

用返回的下载 URL 下载 ZIP，解压后读取 `output_高精.txt`（或 `*_高精.txt`）文件，得到提取出的文字。

## 关键规则

- **必须用 `curl.exe -F` 上传**，不能用 PowerShell 的 `Invoke-WebRequest`（缺少 `-Form` 支持）
- **所有图片必须合并为一个转换任务提交**，不能分开发送
- 单张图片也走「上传 → 提交 → 轮询 → 下载」完整流程
- PowerShell 5.1 环境：无 `&&`/`||`、无三元运算符、编码用 UTF-8
- **OCR 结果不确定时必须暂停向用户确认，绝不猜测**（形近字、数字易混、模糊遮挡等）
