# ShowMeTheMoney Demo Backend

用于演示“带后端验证的付费 Skill”最小闭环：

1. `GET /execute?agent_did=...`
2. 后端先调用 StablePay `GET /api/v1/verify`
3. 若未购买则返回 `402 Payment Required`
4. StablePay 插件在 OpenClaw 侧接管支付（`/api/v1/pay`）
5. 支付后重试 `/execute`，后端返回 `200`

## 启动

```bash
cd /mnt/d/MyLab/StablePay/showmethemoney-skill/demo-backend
npm install
npm start
```

## 环境变量

- `PORT`：默认 `8787`
- `GATEWAY_BASE_URL`：默认 `http://127.0.0.1:28080`
- `STABLEPAY_API_KEY`：默认 `stablepay-dev-key`
- `SKILL_DID`：默认 `did:solana:REPLACE_WITH_SELLER_SOLANA_ADDRESS`（建议显式配置真实卖家 DID）
- `SKILL_NAME`：默认 `ShowMeTheMoney`
- `PRICE`：默认 `1.00`
- `CURRENCY`：默认 `USDC`
- `MESSAGE`：默认 `Pay to unlock ShowMeTheMoney`

推荐显式导出：

```bash
export GATEWAY_BASE_URL="http://127.0.0.1:28080"
export SKILL_DID="did:solana:<seller_solana_pubkey>"
export PRICE="1.00"
export CURRENCY="USDC"
```

## 路由说明

- `/healthz`
- `/execute?agent_did=...`：联调主入口（会触发 verify / 402）
- `/developer/revenue?skill_did=...`
- `/developer/sales?skill_did=...`
- `/agent/balance?agent_did=...`
- `/agent/transactions?agent_did=...`

## 与插件的协作方式

- 你在 `openclaw tui` 调用 `stablepay_execute_paid_skill_demo` 时，插件会先请求本服务 `/execute`。
- 收到 `402` 后，插件按 `ows-pay.md` 流程完成签名与支付。
- 支付成功后插件自动重试 `/execute`，直到后端看到 `purchased=true` 并返回 `200`。
