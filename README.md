# 基于jshERP的进销存接口、UI与性能测试项目

独立的软件测试实践项目。针对开源jshERP验证登录、采购入库、跨仓调拨、销售出库、库存与必要收付款结算，练习测试设计、自动化执行和缺陷分析。**本仓库只包含ERP测试，不包含AI工作台。**

技术栈：Python、Pytest、Requests、Playwright、Locust、Allure、MySQL。被测ERP固定v3.5 / `7ca0f9d68fef761d98b00854d09e975eba291465`；上游Java/Vue业务系统不是自行开发成果。

## 测什么

- 19条基础接口用例：登录与会话、仓库等基础资料、库存、异常和权限。
- A仓采购100件→调拨30件至B仓→B仓销售10件，核对单据数量及明细、A仓70/B仓20/总库存90。
- 3条进阶结算用例：采购付款、销售分次收款、欠款与账户流水；重复付款单号及空明细拒绝。属于ERP内部记账，不是外部支付网关测试。
- 3条Playwright流程：登录退出、页面采购销售、只读新增按钮限制。失败保留截图和Trace。
- Locust库存查询10/30/50/100虚拟用户逐级负载，核对HTTP与业务成功并采集P50、P95、吞吐量、错误率和资源观察。

## 目录

|目录|用途|
|---|---|
|beginner/tests|简化后的接口与UI测试|
|beginner/helpers|请求、登录、测试数据与查询方法|
|beginner/performance|Python Locust任务、分档运行和报告|
|beginner/postman|Collection、空白Environment与手工步骤|
|beginner/docs|上手路线、用例、缺陷与中文测试报告|
|scripts|必要的本机部署、启动、身份准备和安全清理辅助|

## 启动与运行

Windows历史验证环境使用Python3.13.2、PowerShell7和Git。首次部署见[部署说明](docs/环境部署与排障.md)，命令为`python scripts/bootstrap.py`。该过程会下载工具并构建固定ERP，只用于本机隔离环境；本次仓库拆分没有从空机器重新部署验收。已有数据不得清空或重置。

已有环境，在仓库根目录执行：

```powershell
$env:PYTHONIOENCODING='utf-8'
.venv/Scripts/python.exe scripts/start_local.py
cd beginner
../.venv/Scripts/python.exe -m pytest tests/test_login.py::test_correct_login
../.venv/Scripts/python.exe -m pytest tests/test_inventory.py::test_stock_chain
../.venv/Scripts/python.exe -m pytest tests/test_settlement.py::test_purchase_payment_transfer_sale_collection
../.venv/Scripts/python.exe -m pytest tests/test_ui.py --headed
../.venv/Scripts/python.exe performance/run.py --users 10 30 50 100 --duration 90 --warmup 30 --spawn-rate 5
```

不将性能测试与接口、UI、清理或构建同时运行。前端为127.0.0.1:3000，后端为127.0.0.1:9999/jshERP-boot，禁止对公网加压。

接口与UI统一报告：

```powershell
../.venv/Scripts/python.exe -m pytest tests -m 'not ui' --alluredir=artifacts/allure-api
../.venv/Scripts/python.exe -m pytest tests/test_ui.py --alluredir=artifacts/allure-ui
$env:JAVA_HOME=(Resolve-Path ../tools/jdk8u472-b08).Path
../tools/allure-2.32.0/bin/allure.bat generate artifacts/allure-api artifacts/allure-ui -o artifacts/allure-report
../tools/allure-2.32.0/bin/allure.bat open artifacts/allure-report
```

停止ERP：从beginner执行`../.venv/Scripts/python.exe ../scripts/stop_local.py`。

## 已实际执行的结果

|范围|历史实测结果|
|---|---|
|基础接口19条|17通过、2失败，产品缺陷继续暴露|
|UI 3条|修正测试脚本后3通过|
|收付款3条|3通过，另有2条定向清理验证通过|
|逐级负载|四档均执行，48209次请求/11次失败；100用户P95约715ms、吞吐约175次/秒，低于50用户档|

以上来自不同轮次，不代表本次仓库拆分重跑了所有测试。详见[接口/UI报告](beginner/docs/测试报告.md)、[结算链路](beginner/docs/进阶收付款链路.md)、[逐级负载报告](beginner/docs/逐级负载验证.md)。记录了业务500及Redis连接异常，未确认根因，不宣传100用户稳定通过或最大容量。

已知缺陷及默认用例范围见[缺陷记录](beginner/docs/缺陷记录.md)。不通过跳过或弱化断言制造全绿。

## Postman、证据与学习

按[Postman说明](beginner/postman/README.md)导入Collection和Environment示例，验证码人工输入，不关闭校验。Collection已生成并检查，尚未在Postman中执行。

阅读[快速上手与80分钟路线](beginner/docs/快速上手.md)及[业务与测试用例](beginner/docs/业务与测试用例.md)。数据库、随机密码、Token、工具、上游源码、原始Trace和运行产物不上传。历史原始证据保留在原本机，报告中的artifacts链接可能只在该本机可用。来源与成果见[NOTICE](NOTICE.md)。
