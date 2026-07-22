# Bug修复：UZI-Skill --school F 模式下 F派评委评分全为0

## 现象
运行 `python run.py 002497.SZ --depth lite --school F --no-browser` 后：
- panel.json 中 F派 21位活跃评委 score=0, signal=bearish
- 其他派别评委全部 skip（正常，school lock 行为）
- 自检报 critical: "panel 分数异常 (avg=0.0)"，拦截报告生成
- 临时绕过: `UZI_SKIP_REVIEW=1` 可跳过自检生成报告，但评分不可用

## 根因分析
school lock 模式下，评分逻辑未正确将 features 传入 investor_evaluator。
F派评委拿到空 features → 默认 0 分 → 默认 bearish 信号。

## 修复方向
1. 查 `skills/deep-analysis/scripts/lib/pipeline/score.py` 和 `lib/pipeline/score_fns.py` 中 school lock 相关分支
2. 确认 features 是否正确传递给 F派评委的 scoring 函数
3. 查 investor_evaluator 的调用链，确认 school lock 时 features 的传递路径
4. 可能需要查 `lib/pipeline/collect.py` 和 `lib/pipeline/run.py` 中 school lock 的处理逻辑
5. 检查 `investor_db` 或 `investor_knowledge` 模块中 school 过滤逻辑

## 验证方法
```bash
cd /root/UZI-Skill && /root/uzi-venv/bin/python run.py 002497.SZ --depth lite --school F --no-browser
```
检查 .cache/002497.SZ/panel.json 中 F派评委 score 不为 0。

## 注意事项
- 用 `/root/uzi-venv/bin/python` 运行（Python 3.11，系统 Python 3.12 会 SIGSEGV）
- 不要修改 .cache/ 目录下的缓存数据
- 只修评分逻辑，不要碰数据采集和报告生成代码
- 代码署名 @author Jiane
