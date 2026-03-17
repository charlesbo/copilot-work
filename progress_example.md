# 进度日志

## 2024-01-15 Session 1

### 子任务：实现 JWT 模块
- ✅ 创建了 src/auth/jwt.ts，实现了 sign/verify 函数
- ✅ 使用 RS256 算法（选择理由：公私钥分离更安全）
- ✅ 单元测试通过（12/12）
- ⚠️ 发现 jsonwebtoken 库 v9 有 breaking change，锁定 v8.5.1
- 📁 关键文件：src/auth/jwt.ts, tests/auth/jwt.test.ts

#### 子任务完成检查清单
- [ ] 代码改动已保存
- [ ] 测试已通过（记录结果到 progress.md）
- [ ] 关键决策已记录理由
- [ ] 新增/修改的文件列表已记录
- [ ] 发现的问题/TODO 已记录
- [ ] git commit 已提交（带有清晰的 commit message）

### 子任务：实现登录 API
- ✅ POST /api/login 端点完成
- ✅ 密码使用 bcrypt 哈希，cost factor = 12
- ❌ 限流中间件还没做（移到下一个子任务）
- 📁 关键文件：src/routes/auth.ts, src/middleware/auth.ts

#### 子任务完成检查清单
- [ ] 代码改动已保存
- [ ] 测试已通过（记录结果到 progress.md）
- [ ] 关键决策已记录理由
- [ ] 新增/修改的文件列表已记录
- [ ] 发现的问题/TODO 已记录
- [ ] git commit 已提交（带有清晰的 commit message）
---

## 2024-01-16 Session 2
...
