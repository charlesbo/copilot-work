# 任务交接文档

## 上次更新：2024-01-15

## 当前状态
正在实现用户认证系统，已完成 3/7 个子任务。

## 刚才做了什么
- 完成了登录 API 的基本实现
- 正在调试 token 刷新逻辑，问题出在 refresh token 的过期时间计算

## 未解决的问题
1. refresh token 过期时间使用绝对时间还是滑动窗口？（倾向滑动窗口）
2. 并发刷新时的 race condition 还没处理

## 下一步
1. 修复 token 刷新的过期时间计算（参考 src/auth/refresh.ts:42）
2. 添加并发刷新的互斥锁
3. 编写 token 刷新的集成测试

## 关键文件
- src/auth/jwt.ts — token 生成/验证
- src/auth/refresh.ts — token 刷新（进行中）
- src/routes/auth.ts — 认证路由
- tests/auth/ — 所有认证测试

## 注意事项
- jsonwebtoken 锁定 v8.5.1，不要升级到 v9（有 breaking change）
- bcrypt cost factor 用 12，不要改（已做过性能测试，见 snapshots/）
