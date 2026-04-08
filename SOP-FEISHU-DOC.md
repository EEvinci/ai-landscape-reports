# 飞书文档创建 SOP（权限分离版）

> 版本：v1.0
> 创建：2026-04-08
> 适用场景：主-子 Agent 协作创建飞书文档

---

## 一、问题背景

**现象：**
- 主 agent 调用 `feishu_doc create` 创建文档 → 文档创建成功，但用户没有访问权限
- 主 agent 调用 `feishu_perm add` 授权 → 权限 API 调用失败（`app secret invalid`）
- 子 agent 调用相同接口 → 权限授权成功

**根本原因（推断）：**
- OpenClaw 的飞书插件在主 agent session 和子 agent session 中，走的认证路径不同
- 主 agent session 的请求被平台标记为 `trusted requester identity unavailable`，跳过了权限写入步骤
- 子 agent session 走的是插件级认证（plugin-level tenant_access_token），权限操作完整执行

---

## 二、核心原则

```
主 agent：负责"内容写入"
子 agent：负责"权限授权"

两者分离，各司其职，不混在一起
```

---

## 三、操作流程

### Step 1：主 agent 创建文档

```python
# 主 agent 执行
feishu_doc(
  action="create",
  title="文档标题",
  content="# 文档内容..."  # 可选，写入内容
)
```

返回：
```json
{
  "document_id": "VBaHdvFepoZFTQx1t3Bcjv1hnFc",
  "title": "文档标题",
  "url": "https://feishu.cn/docx/VBaHdvFepoZFTQx1t3Bcjv1hnFc"
}
```

### Step 2：主 agent 写入内容（可选）

```python
# 主 agent 执行
feishu_doc(
  action="write",
  doc_token="VBaHdvFepoZFTQx1t3Bcjv1hnFc",
  content="# 文档内容..."
)
```

### Step 3：子 agent 负责权限授权（关键步骤）

```python
# 触发子 agent执行
feishu_perm(
  action="add",
  type="docx",
  member_type="openid",
  member_id="ou_36f398d4a7e8bc323453cda6d2c90a65",
  perm="full_access",
  token="VBaHdvFepoZFTQx1t3Bcjv1hnFc"
)
```

### Step 4：通知用户

```python
# 主 agent 发送飞书消息给用户
message(
  action="send",
  channel="feishu",
  message="📄 文档已创建并授权\n标题：xxx\n链接：https://feishu.cn/docx/xxx"
)
```

---

## 四、为什么权限必须由子 agent 执行

OpenClaw 平台的飞书插件认证机制：

```
飞书插件配置（平台级）
    ↓
所有飞书 API 请求携带 plugin-level tenant_access_token
    ↓
┌─────────────────────────────────────┐
│  主 agent session                    │
│  → 请求被标记为"trusted"             │
│  → 跳过权限写入步骤                   │
│  → 创建者有权限，但无法授权他人         │
└─────────────────────────────────────┘
         vs
┌─────────────────────────────────────┐
│  子 agent session                    │
│  → 请求走完整认证流程                  │
│  → permission API 完整执行             │
│  → 成功授权第三方用户                  │
└─────────────────────────────────────┘
```

**结论：权限操作必须在子 agent session 中执行，不能在主 agent session 中执行。**

---

## 五、子 agent 权限任务模板

当主 agent 完成文档创建和内容写入后，触发子 agent 执行权限授权：

```
## 任务
为飞书文档添加访问权限

## 输入参数
- doc_token: <文档token>
- doc_title: <文档标题>
- user_openid: ou_36f398d4a7e8bc323453cda6d2c90a65

## 执行步骤
1. 使用 feishu_perm 工具为用户添加 full_access 权限：
   feishu_perm(
     action="add",
     type="docx",
     member_type="openid",
     member_id="ou_36f398d4a7e8bc323453cda6d2c90a65",
     perm="full_access",
     token="<doc_token>"
   )
2. 确认权限添加成功
3. 回复：
[AGENT DONE]
doc_token: <doc_token>
perm_added: true
user: 李琛
[/AGENT DONE]
```

---

## 六、权限级别说明

| perm 值 | 说明 |
|---|---|
| `view` | 可查看 |
| `edit` | 可编辑 |
| `full_access` | 可管理（推荐） |
| `comment` | 可评论 |

推荐使用 `full_access`，确保用户有完整管理权限。

---

## 七、永久配置

用户 open_id：`ou_36f398d4a7e8bc323453cda6d2c90a65`
用户姓名：李琛

此配置固化在 AGENTS.md 和 SOP-FEISHU-DOC 中，每次创建文档后必须使用。

---

## 八、示例：创建 Batch F 飞书文档

**场景：** Topics 49-56（共8个）已完成，需要创建飞书文档汇总

**Step 1 - 主 agent 创建文档：**
```
文档标题：AI全景报告 Batch F（49-56）
```

**Step 2 - 主 agent 写入目录内容：**
```
写入：目录 + 各报告摘要 + GitHub 链接
```

**Step 3 - 子 agent 授权：**
```
feishu_perm add full_access to ou_36f398d4a7e8bc323453cda6d2c90a65
```

**Step 4 - 主 agent 通知用户：**
```
发送飞书消息，包含文档链接
```

---

## 九、常见问题

**Q：子 agent 创建的文档，主 agent 可以授权吗？**
A：可以，只要在子 agent session 中调用 `feishu_perm add` 即可。

**Q：主 agent 调用 feishu_perm 会报错吗？**
A：不会报错，但返回中会显示 `requester_permission_skipped_reason: trusted requester identity unavailable`，实际权限并未写入。

**Q：子 agent 授权后，主 agent 需要确认吗？**
A：不需要，权限写入是即时的，用户刷新飞书文档即可看到新权限。

**Q：批量创建文档时，可以多个子 agent 并行授权吗？**
A：可以，每个子 agent 独立执行一个文档的权限授权，互不干扰。

---

## 十、文件关联

- 主文档创建流程：SOP-FEISHU-DOC.md
- GitHub 推送 SOP：SOP-MULTI-AGENT.md
- 任务调度 SOP：SOP-MULTI-AGENT.md
