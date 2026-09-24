# src-hunting — 公开 SRC 授权漏洞扫描工具链

> ⚠️ **重要声明：本项目严格限定于授权目标。严禁对未授权系统发起任何扫描。**

## 工具链

| 工具 | 版本 | 用途 |
|------|------|------|
| [nuclei](https://github.com/projectdiscovery/nuclei) | v3.11.1 | 基于模板的漏洞扫描引擎 |
| [nuclei-templates](https://github.com/projectdiscovery/nuclei-templates) | 本地已缓存 | 漏洞检测模板库（~29k 条） rightfully |
| [httpx](https://github.com/projectdiscovery/httpx) | v1.12.0 | 探针：URL/站点存活检测 |
| [subfinder](https://github.com/projectdiscovery/subfinder) | v2.16.0 | 子域名枚举 |
| [katana](https://github.com/projectdiscovery/katana) | 可选 | 爬虫：页面发现与指纹识别 |
| [xray](https://github.com/raysec/xray) | 可选 | 漏洞扫描（黑盒） |

> 本机安装目录为 `~/bin/`（nuclei、httpx、subfinder 已就绪）。
> 若需用 Go 源码方式安装 katana 等：先安装 Go 运行时，再执行：
> ```bash
> go install -v github.com/projectdiscovery/katana/cmd/katana@latest
> ```

## 工作流程

```
子域名枚举 → 存活探测 → 指纹识别 → 漏洞扫描 → 报告输出
subfinder   → httpx    → katana    → nuclei    → 结果归档
```

### 1. 子域名枚举

```bash
# 对授权目标枚举子域名
subfinder -d target.com -o results/subdomains.txt
```

### 2. 存活探测

```bash
# 批量探测子域名存活并收口 URL
httpx -l results/subdomains.txt -o results/alive.txt -silent
```

### 3. 漏洞扫描（仅授权目标）

```bash
# 使用 nuclei 对存活目标扫描
nuclei -l results/alive.txt -t ~/nuclei-templates -o results/nuclei-report.json -silent

# 按需加载特定模板
nuclei -l results/alive.txt -t ~/nuclei-templates/http/exposures/ -o results/exposures.json

# 同步最新模板
nuclei -ut
```

### 4. 可选：爬虫发现

```bash
# 深度爬取授权目标
katana -u https://target.com -d 3 -json -o results/katana-output.json
```

## 授权扫描注意事项

1. **必须有书面授权**：仅在获得目标 SRC 官方授权后方可扫描，任何未授权扫描均违反法律。
2. **遵守扫描规则**：遵循目标 SRC 的扫描频率、时段、接口限制等规定。
3. **最小化影响**：避免对生产环境造成性能影响；优先使用 `-rate-limit` 和 `-burst` 参数控制并发。
4. **数据安全**：扫描结果仅保存在本地，不上传任何发现数据。
5. **漏洞报告**：发现的漏洞通过 SRC 官方渠道提交，不通过其他途径披露。
6. **模板更新**：定期执行 `nuclei -ut` 更新模板库，保持检测能力。
7. **日志留存**：保留扫描记录以备内部审计。

## 免责声明

本项目仅用于合法授权的网络安全测试。使用者须自行承担因不当使用产生的全部责任。作者不对任何直接或间接损失负责。

## 状态

- [x] Git 仓库初始化
- [x] nuclei + nuclei-templates 本地可用
- [x] GitHub 远程推送（https://github.com/kxh2009/src-hunting）
- [x] httpx / subfinder 二进制安装（~/bin/，无需 Go 运行时）
