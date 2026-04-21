# qzcli 任务查找和删除指南

## 问题背景

批量提交任务后，qzcli 本地追踪（`qzcli list`）可能只显示部分任务，但实际上所有任务都已提交到启智平台。本地追踪不完整的原因可能是：
- Cookie 过期
- 提交时网络问题
- qzcli 自动追踪机制失败

## 解决方案：使用 qzcli delete 命令查找任务

`qzcli delete` 命令会直接从启智平台 API 查询任务，不依赖本地追踪数据。

### 1. 按名称查找任务

```bash
qzcli delete --name "VASP-Mn" \
  --workspace "ws-6e6ba362-e98e-45b2-9c5a-311998e93d65" \
  --limit 200
```

**参数说明：**
- `--name`: 任务名称模糊匹配（包含匹配）
- `--workspace`: 工作空间 ID
- `--limit`: 查询数量限制（默认50，最大建议200）

**输出示例：**
```
将删除以下 103 个任务：

    1. [HPC] VASP-MnPTC_Mono                          queueing        hpc-job-40c151bf-9a99-4490-97e5-44aa7aecfa56
    2. [HPC] VASP-MnPIC_Mono_1                        queueing        hpc-job-041c2149-3ca4-417d-8f76-3f488a6f0443
    ...
   70. [HPC] VASP-MnOHNPc_Mono_1                      running         hpc-job-6c4e1f0f-8177-4c07-9dbd-281ee51988a5
   ...
   83. [HPC] VASP-MnHTTX_Mono_79                      succeeded       hpc-job-69803725-fdc4-48e0-9d57-2317bfc8b26a
```

### 2. 按状态过滤（可选）

```bash
# 只查找排队中的任务
qzcli delete --name "VASP-Mn" \
  --workspace "ws-6e6ba362-e98e-45b2-9c5a-311998e93d65" \
  --status "queueing" \
  --limit 200

# 只查找失败的任务
qzcli delete --name "VASP-Mn" \
  --workspace "ws-6e6ba362-e98e-45b2-9c5a-311998e93d65" \
  --status "failed" \
  --limit 200
```

**任务状态：**
- `queueing`: 排队中
- `running`: 运行中
- `succeeded`: 成功完成
- `failed`: 失败

### 3. 统计任务状态

```bash
qzcli delete --name "VASP-Mn" \
  --workspace "ws-6e6ba362-e98e-45b2-9c5a-311998e93d65" \
  --limit 200 2>&1 | \
  grep -oE "(queueing|running|succeeded|failed)" | \
  sort | uniq -c
```

**输出示例：**
```
      1 failed
      1 running
     21 succeeded
```

### 4. 确认删除任务

**注意：删除操作不可逆，请谨慎操作！**

```bash
# 交互式删除（会提示确认）
qzcli delete --name "VASP-Mn" \
  --workspace "ws-6e6ba362-e98e-45b2-9c5a-311998e93d65" \
  --limit 200

# 自动确认删除（跳过确认提示）
qzcli delete --name "VASP-Mn" \
  --workspace "ws-6e6ba362-e98e-45b2-9c5a-311998e93d65" \
  --limit 200 \
  --yes
```

## 常见场景

### 场景1：删除所有排队中的任务

```bash
# 先查看
qzcli delete --name "VASP-" \
  --workspace "ws-6e6ba362-e98e-45b2-9c5a-311998e93d65" \
  --status "queueing" \
  --limit 200

# 确认后删除
qzcli delete --name "VASP-" \
  --workspace "ws-6e6ba362-e98e-45b2-9c5a-311998e93d65" \
  --status "queueing" \
  --limit 200 \
  --yes
```

### 场景2：删除失败的任务

```bash
qzcli delete --name "VASP-" \
  --workspace "ws-6e6ba362-e98e-45b2-9c5a-311998e93d65" \
  --status "failed" \
  --limit 200 \
  --yes
```

### 场景3：删除特定前缀的所有任务

```bash
# 删除所有 VASP-Mn 开头的任务
qzcli delete --name "VASP-Mn" \
  --workspace "ws-6e6ba362-e98e-45b2-9c5a-311998e93d65" \
  --limit 200 \
  --yes
```

## 注意事项

1. **删除前先查看**：不要直接加 `--yes`，先运行一次查看要删除的任务列表
2. **不要删除运行中的任务**：除非确实需要停止，否则不要删除 `running` 状态的任务
3. **已完成的任务可以保留**：`succeeded` 状态的任务不影响资源，可以保留用于查看日志
4. **Cookie 过期**：如果提示认证失败，先运行 `qzcli login` 重新登录

## 工作空间 ID

当前项目的工作空间 ID：
```
ws-6e6ba362-e98e-45b2-9c5a-311998e93d65
```

## 相关命令

```bash
# 查看本地追踪的任务
qzcli list --limit 200

# 查看运行中任务的资源使用情况
qzcli perf -w "ws-6e6ba362-e98e-45b2-9c5a-311998e93d65"

# 重新登录获取 cookie
qzcli login

# 追踪单个任务
qzcli track <job_id>
```

## 故障排查

### 问题1：qzcli list 只显示少量任务

**原因**：本地追踪数据不完整

**解决**：使用 `qzcli delete --name` 从 API 查询，不依赖本地数据

### 问题2：提示 Cookie 过期

**解决**：
```bash
qzcli login
# 输入用户名和密码
```

### 问题3：找不到任务

**检查**：
1. 确认任务名称是否正确（支持模糊匹配）
2. 确认工作空间 ID 是否正确
3. 增加 `--limit` 参数（默认只查50个）

---

最后更新：2026-04-18
