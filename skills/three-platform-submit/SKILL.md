---
name: three-platform-submit
description: 当用户要求使用统一 Excel 在 VIPON、Dealmoon、Koupon 顺序提交商品推广、让三平台与金山文档共用同一 Google Chrome 登录会话，或维护和打包对应 Windows 工具时使用；不把加载技能视为真实提交、清空在线表格、修改线上记录或保存账号凭据的授权。
---

# 三平台一键提交

本技能用于操作、诊断和维护三平台一键提交工具。加载本技能不等于用户已经授权本轮真实提交、更新既有推广、清空金山文档或对外发布；根据用户当前请求区分演练、单平台提交和正式全流程。

本技能只包含流程说明，不自带三平台工具源码或 Windows 安装包。执行或维护前，先定位用户提供的工具根目录，并确认其中有 `dashboard.py`、`package_windows.py` 和 `platforms/`。如果这些文件不存在，只能提供操作指导或请用户提供完整工具包，不能声称本技能已经包含或运行了该工具。

## 核心架构

- VIPON、Dealmoon、Koupon 和金山文档必须共用 VIPON launcher 启动的同一个 Google Chrome 进程、同一个 profile 和同一个 CDP 调试端口，默认端口为 `9224`。
- Windows 共享 profile 为 `%LOCALAPPDATA%\三平台一键提交\runtime\profiles\vipon\`。保留 `vipon` 目录名是为了兼容已有登录状态。
- VIPON launcher 拥有 Chrome 进程和 profile；Dealmoon、Koupon、金山文档只通过 CDP 新建各自标签页。各流程结束时只关闭自己创建的标签页，不关闭共享 Chrome、context 或其他标签页。
- 端口不可达时可调用 VIPON launcher 启动该 Chrome；端口已被非目标 Chrome 服务占用、返回无效响应或无法验证时必须明确失败。共享模式不得回退到 Edge、Brave、系统默认浏览器、Playwright Chromium 或第二个 Chrome。
- `DEALMOON_PROFILE_DIR`、Dealmoon `profileDir`/`.browser-profile` 和金山文档独立 profile 只用于用户明确要求的 standalone 模式，控制板共享模式不读取它们。

## 输入与授权

开始前确认：

1. 本轮使用的 `.xlsx` 文件及工作表。
2. 平台范围：VIPON、Dealmoon、Koupon 中的一个、多个或全部。
3. 模式：正式提交或演练。演练不得点击最终提交。
4. 是否允许更新已存在的 VIPON 推广。读取当前任务中仍有效的明确授权和最新限制；不要把其他任务、其他批次或模糊历史描述当成本轮新增授权。
5. VIPON 完成后是否按现有控制板流程清空并重写指定金山文档。该动作会替换在线表格当前内容，必须在用户授权范围内。

账号、密码、验证码、cookie、token 和浏览器 profile 只能留在用户本机受控配置或浏览器中，不写入技能、仓库、日志、报告、Excel 或聊天回复。

## 运行前检查

- Windows 10/11、Python 3.9+、Node.js 20+ 和 Google Chrome 可用。
- Python 依赖至少包括 `openpyxl`、`websocket-client` 和 Playwright；Dealmoon 的 `node_modules` 中必须有 `playwright-core`、`xlsx` 及其运行文件。
- 即使只运行 Dealmoon 或 Koupon，也要检查 `websocket-client`，因为控制板可能需要通过 VIPON launcher 启动共享 Chrome。
- Excel/WPS 已关闭，源文件存在且第一个工作表非空。统一模板使用 `1:`、`2:` 等编号块，并包含商品名、原价、Deal 价、折扣、日期和链接等实际所需字段。
- Koupon 配置含公司名称以及邮箱、电话或微信中的至少一种联系方式。
- 先探测 CDP `/json/version`。连接拒绝或超时表示端点暂时不可达，此时允许尝试 VIPON launcher；有效 HTTP 响应但不是目标 Chrome 表示端口冲突，不能启动另一个浏览器覆盖它。

若用户只是要求诊断，保持只读，不运行真实提交、不写回 Excel、不清空在线表格。

## 执行流程

1. 为本次运行创建时间戳目录，并为每个平台复制独立 Excel 工作副本。不要让多个平台同时写同一文件。
2. 默认顺序为 VIPON、Dealmoon、Koupon。执行 Dealmoon 或 Koupon 前再次确认共享 Chrome 可连接。
3. VIPON 使用共享 profile 和端口运行。演练只做预检；正式模式只有在用户允许时才启用更新既有推广。
4. Dealmoon 以 `--reuse-browser --debug-port 9224` 连接共享 Chrome。页面级 header、时区和初始化脚本只能作用于 Dealmoon 自有标签页，不能污染共享 context 的其他页面。
5. Koupon 同样以 `--reuse-browser --debug-port 9224` 连接默认 context，并只关闭自己创建的页面。
6. 一个外部平台失败时记录真实原因并按控制板既有策略继续后续平台；共享浏览器本身不可用时，该平台不得启动备用浏览器。
7. 出现登录、验证码或人工确认点时暂停并让用户在当前共享 Chrome 完成。出现安全警告、checkpoint 或异常行为提示时停止，不绕过、不切换账号、浏览器、代理或 IP 重试。

结果不确定时，先检查平台页面、运行记录和输出文件再决定是否重试；不得把超时直接当成未提交并盲目重发。

## 类别与重复记录

- 优先保留平台 API 返回的完整有效类别。只有类别缺字段时，才可依据商品详情标题和明确的类别词补全。
- 类别 ID 与组冲突、叶子类别有歧义或没有足够证据时，提交前明确失败或交给人工选择，不发送空类别，不随意猜测。
- 平台报告已存在时，区分“跳过重复”“用户授权更新”和“结果未知”。不要把历史批次里更新过的记录当成本轮默认更新清单。

## 结果与金山文档

- 配置保存在 `%APPDATA%\三平台一键提交\`；运行记录和浏览器资料保存在 `%LOCALAPPDATA%\三平台一键提交\runtime\`。
- 每次任务的工作副本、日志和平台结果位于 `runtime\runs\时间戳\` 下的对应子目录。
- VIPON 成功结果可按工具既有规则回写源 Excel 的上贴链接；写回前必须备份源文件。Excel 被占用或保存失败时保留原文件并报告，不能用生成结果覆盖损坏的源文件。
- 正式 VIPON 流程生成同步载荷后，控制板可清空并重写已授权的金山文档。同步前确认登录和编辑权限；没有权限时停止，不清空。同步结束只关闭临时标签页。
- 状态必须区分成功、重复、失败、已停止、结果未知和金山文档同步失败；不要用“已完成”概括部分失败。

## Windows 打包

维护源码或向另一台 Windows 电脑交付时，在已定位的三平台工具根目录中优先使用 `package_windows.py` 或 `PACKAGE_WINDOWS.bat` 生成 UTF-8 ZIP，并执行以下检查：

- 排除 `runtime`、`profiles`、`logs`、`.browser-profile*`、虚拟环境、缓存、输出目录和旧 ZIP。
- 排除 VIPON、Koupon、Dealmoon 的可变 `config.json`、任何 private/auth/cookie/storage-state 文件以及密钥和证书。
- 排除所有用户工作簿，只允许经过审查的统一模板和平台模板。
- 包含 `*.example.json`，让 Windows 首次安装在配置不存在时创建本机配置；升级不得覆盖用户现有配置。
- 生成 `PACKAGE_MANIFEST.json`，逐文件核对大小和 SHA-256，确认 manifest、ZIP 成员和当前源码一致，并验证中文路径使用 UTF-8 标记。
- 打包前后扫描用户提供过的账号和密码值，确认它们没有进入归档；检查时不要把秘密打印到终端或回复。

交付说明应要求完整解压到普通可写目录，先运行 `首次安装环境.bat`，再运行 `START.bat`。不要让用户直接在 ZIP 内运行。

## 维护代码时

若当前任务包含修改源码，先确认用户提供的三平台工具根目录，再阅读现有实现并保持这些相对路径边界：

- 控制板：`dashboard.py`
- VIPON 与金山文档：`platforms/vipon/`
- Dealmoon：`platforms/dealmoon/`
- Koupon：`platforms/koupon/`
- Windows 打包：`package_windows.py`

至少运行与改动相关的平台测试、Python 编译检查和 Dealmoon 的 Node 检查。共享浏览器改动还应在隔离的临时 Chrome/profile 和非生产 CDP 端口上验证：两个平台能同时创建标签页，任一客户端退出后共享 Chrome、原标签和另一平台标签仍存在。不要用用户真实 profile、真实 `9224` 会话或生产帖子做测试。

## 明确不做

- 不因加载本技能自动提交、更新、重投、清空在线表格或创建定时任务。
- 不把账号、密码、验证码或登录资料提交到 Git、ZIP、Excel、日志或聊天。
- 不修改用户源 Excel，除非当前流程明确包含经过备份的 VIPON 链接回写。
- 不为了“继续运行”而切换浏览器或绕过平台安全控制。
- 不把演练、页面填写、登录成功或同步载荷生成写成真实发布成功。
