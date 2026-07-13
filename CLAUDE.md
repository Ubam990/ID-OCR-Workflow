# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目性质

这不是代码仓库，而是 Claude Code 的会话工作目录。核心资产是 `memory/` 目录中的工作流记忆文件。

## 核心工作流：身份证+营业执照 OCR 提取

详见 `memory/id-extraction-workflow.md`，简要流程：

1. 用户提供身份证正反面 + 营业执照照片（三张）
2. 通过 https://ocr.wdku.net/ 在线 OCR 识别
   - 上传：`POST WebClient.UploadFile` → `/upload?convtype=ocr`
   - 合并提交：`POST` → `/index`（参数 ids, ts, paytype=free, core=all, saveformat=txt, lang=1,2, combine=1 等）
   - 轮询：`POST` → `/get_conv_convert_status`（需同时传 id 和 time）
   - 下载 ZIP 解压读取 `output_高精.txt`
3. 生成拼音（法人首字母大写，公司/地址全小写无空格）
4. 自动生成邮箱和密码
5. 输出到 `C:\Users\12905\Desktop\<姓名>.txt`

## 关键规则

- **工具直接执行，不询问权限**
- **OCR 结果不确定时必须暂停向用户确认**（准确性第一）
- 拼音：拼音转换工具 https://www.qqxiuzi.cn/zh/pinyin/ 常被拦截，可用人工知识替代
- PowerShell 5.1 环境：无 `-Form` 参数、无 `&&`/`||`、无三元运算符、编码用 UTF-8
- WebClient 上传文件：`$wc.UploadFile(url, "POST", $filePath)`

## 记忆文件

- `memory/id-extraction-workflow.md` — 完整提取工作流
- `memory/user-permissions.md` — 用户授权规则
- `memory/MEMORY.md` — 记忆索引
