# DeepSeek 用量面板(本地同步 + GitHub 网页)

在手机 / 电脑上,随时查看你的 **DeepSeek API 余额** 和 **用量统计**(每天用了多少 token、每个模型花了多少钱)。

## 这套东西是怎么工作的

```
你的电脑
  CC Switch 记账数据 ──┐
                       ├─→ 运行一次 sync.py
  DeepSeek 官方余额 ──┘        │ 只读你的数据,绝不改动
                               │ 生成 data/dashboard.json
                               ▼
  GitHub 仓库(只存数字,不含 API Key)
                               ▼
  GitHub Pages 专属网页 → 手机/电脑随时打开看
```

**安全说明:**
- 你的 DeepSeek API Key **永远只留在你这台电脑上**,脚本从 CC Switch 里读取、只用于查余额,不会上传。
- 上传到 GitHub 的只有一个 JSON 数字文件 + 网页文件,别人拿到也偷不了你的钱。
- 网页是公开的(免费托管要求),但内容只有用量数字,可以接受。

## 这个文件夹里的东西

| 文件 | 作用 |
|---|---|
| `sync.py` | 同步脚本(自动读用量 + 查余额 + 上传) |
| `sync.bat` | 双击就能跑同步脚本的入口 |
| `index.html` | 展示网页(会自动上传到仓库) |
| `config.json` | **需要你自己填**(GitHub 用户名/仓库/令牌),别上传 |
| `config.json.example` | 配置模板 |
| `README.md` | 本说明 |
| `setup-schedule.ps1` | (可选)设置每小时自动同步 |

---

## 一次性设置(约 10 分钟,以后就不用管了)

### 第 0 步:确认你有 GitHub 账号
没有的话先去 https://github.com 注册(免费,邮箱验证即可)。

### 第 1 步:在 GitHub 上建一个空仓库
1. 打开 https://github.com/new
2. **Repository name** 填:`deepseek-usage`
3. 下面保持默认(Public 就行,勾选 Public)
4. **先别勾选** "Add a README file" 等任何初始化选项,保持空仓库
5. 点绿色按钮 **Create repository** 创建

创建好后会显示仓库地址形如 `https://github.com/你的用户名/deepseek-usage`,记住你的**用户名**和**仓库名**。

### 第 2 步:生成一个"上传令牌"(GitHub Personal Access Token)
1. 打开 https://github.com/settings/tokens
2. 点右上角 **Generate new token** → **Generate new token (classic)**
3. **Note** 随便填,比如 `deepseek-usage-sync`
4. **Expiration** 选 `No expiration`(或你觉得合适的期限,过期后重新生成即可)
5. 在 **Select scopes** 里,勾选 **`repo`**(整个 "repo" 那一大格,它会自动带上子项)
6. 拉到最下面点 **Generate token**
7. 页面会显示一串字符(以 `ghp_` 开头),**只有这一次能看到**,复制保存好

> ⚠️ 这串令牌等于你 GitHub 的"上传钥匙",别发给别人、别提交到网页上。

### 第 3 步:填写本地配置
1. 进入本项目文件夹 `E:\Claude工作区\02_项目\deepseek-usage`
2. 把 `config.json.example` 复制一份,重命名为 `config.json`
3. 用记事本打开 `config.json`,把里面的三个值填上:
   - `github_owner`:你的 GitHub 用户名
   - `github_repo`:`deepseek-usage`(第 1 步建的仓库名)
   - `github_token`:第 2 步复制的那串 `ghp_...`

### 第 4 步:运行一次同步脚本
双击 **`sync.bat`**(或双击 `sync.py`)。

如果一切正常,会看到:
```
✅ 已上传到 GitHub 仓库: 你的用户名/deepseek-usage
🌐 你的专属网页: https://你的用户名.github.io/deepseek-usage/
```

第一次开启网页后,GitHub 需要 **1~3 分钟** 生成页面。等一会儿再打开网址。

### 第 5 步:(可选)设置每小时自动同步
这样你不用记着去双击,电脑开着时它自己每小时同步一次:
1. 右键以管理员身份打开 PowerShell(开始菜单搜 PowerShell → 右键 → 管理员)
2. 运行:
```powershell
powershell -ExecutionPolicy Bypass -File "E:\Claude工作区\02_项目\deepseek-usage\setup-schedule.ps1"
```
之后每次开机,计划任务会自动在后台每小时跑一次同步。

---

## 常见问题

**网页打开是 404 / 白屏?**
- 刚开启 Pages 要等 1~3 分钟,稍后再刷新。
- 确认网址拼写:`https://用户名.github.io/deepseek-usage/`(末尾的 `/` 也要)。

**上传失败 HTTP 401 / 403?**
- 多半是令牌没填对,或者生成令牌时没勾选 `repo`。重新做第 2 步。

**显示"找不到 CC Switch 数据库"?**
- 说明这台电脑上还没有 CC Switch 的数据。正常使用过 CC Switch 后就会生成。

**网页上的费用是美元还是人民币?**
- 余额是人民币(¥,DeepSeek 官方数据);用量费用是 CC Switch 按 models.dev 定价估算的美元($),两者单位不同,页面里都有标注。

**余额一直是旧的/提示取自上次记录?**
- 说明查询余额时网络或 Key 临时出问题,页面会显示上一次成功查询的余额。下次同步成功会自动更新。
