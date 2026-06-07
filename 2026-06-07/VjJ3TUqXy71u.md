**主要问题**
- 严重：`openai-code-review-sdk/src/main/java/ruics/com/sdk/utils/WechatUtils.java:24` 将 `APP_ID`、`APP_SECRET` 以及默认 `touser`/`template_id` 写死在源码里，生产凭据和接收人会随仓库暴露，且无法按环境隔离；应改为环境变量或安全配置读取。
- 严重：`openai-code-review-sdk/src/test/java/ruics/com/sdk/test/ApiTest.java:59` 新增测试会真实调用微信接口且没有任何断言，跑单测就会产生外部副作用、依赖网络和真实凭据，CI 很容易误发消息并变得不稳定；应改为 mock，或显式标记为手工集成测试。
- 中等：`openai-code-review-sdk/src/main/java/ruics/com/sdk/utils/WechatUtils.java:63` `sendPostRequest` 没有设置超时、没有检查 HTTP 状态和微信返回的 `errcode`，异常还被直接吞掉；发送失败时主流程仍会看起来“成功”，后续排障会很困难。
- 中等：`openai-code-review-sdk/src/main/java/ruics/com/sdk/OpenAiCodeReview.java:54` 通知里的 `branch` 和 `review` 被硬编码为 `big-market`、`feat: 新加功能`，与当前真实分支和评审结果无关，合并后所有消息都会误导接收人。
- 中等：`openai-code-review-sdk/src/main/java/ruics/com/sdk/utils/WechatUtils.java:26` token 缓存固定为 60 秒，没有使用微信返回的 `expires_in`；这会频繁刷新 token，增加接口限流风险。

**结论**
- 这次改动里，前两项建议作为合并阻断项处理；其余几项会明显影响可用性和运维性，也建议一并修正。
- 如果你需要，我可以继续按这些问题给出一版可落地的修复方案或直接整理成 review comment 模板。