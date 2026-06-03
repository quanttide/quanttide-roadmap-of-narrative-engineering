# Harness — LLM 集成测试方法论

Harness 不是框架、不是库、不是基础设施层。Harness 是写 LLM 集成测试之前自问的五个问题。

## 五问

### 1. 同一轮比较了吗？

LLM 输出数值不稳定。同一 prompt 两次调用可能返回不同的分数。

```
❌ assert score > 0.5
✅ assert score_match > score_mismatch
```

跨请求比较 LLM 输出不可信。A 和 B 必须在同一次 HTTP 请求内比较。

### 2. 正反两向都测了吗？

单方向的测试可能因为 LLM 的偶然性输出通过。

```
✅ test_campus_matches_campus
✅ test_campus_does_not_match_urban
✅ test_urban_matches_urban
✅ test_urban_does_not_match_campus
```

正反两个方向四组测试，任意一组失败说明分辨能力有问题。

### 3. 负样本控制了吗？

系统可能发生基线漂移——对所有输入都输出高分或低分。

```
✅ test_neutral_text_scores_low
✅ test_neutral_text_scores_low_urban
```

中性文本（超市日记、天气预报）不应在任何风格下获得高分。

### 4. 断言的是存在性还是具体数值？

LLM 输出格式可以通过 prompt 约束，但具体内容不可控。

```
❌ assert len(fix_strategies) >= 2
✅ assert len(fix_strategies) >= 1
❌ assert gap_len >= 15
✅ assert original != expected
```

优先断言存在性。数值边界只在"明显异常"时使用。

### 5. 失败了能不能追溯？

LLM 测试失败的原因可能是代码问题，也可能是 LLM 输出异常。

```
❌ assert 200 == resp.status_code       # 不知道是 502 还是逻辑错
✅ assert 200 == resp.status_code       # + 日志记录 LLM 请求和响应
```

每次 LLM 调用必须记录 prompt 摘要、response body、耗时。失败时自动输出完整调用链。
