支付系统 - 架构师学习笔记
    
    


    
        
            
                支付系统
            
            
            
                
                    支付系统是电商系统的核心模块之一，负责处理用户支付请求、与第三方支付平台交互、资金结算等关键业务。支付系统需要保证交易的安全性、可靠性和一致性，是电商系统的资金入口。
                
                
                支付系统功能
                
                
                    
                        
                            
                            支付渠道对接
                        
                        
                            支付宝、微信支付等第三方支付平台
                            银联、网联等银行卡支付
                            Apple Pay、Google Pay等移动支付
                            企业账户支付
                        
                    
                    
                    
                        
                            
                            交易处理
                        
                        
                            支付请求处理
                            支付结果通知
                            交易状态查询
                            退款处理
                        
                    
                    
                    
                        
                            
                            资金结算
                        
                        
                            交易对账
                            资金分账
                            手续费计算
                            结算报表
                        
                    
                    
                    
                        
                            
                            风险控制
                        
                        
                            支付风控
                            反欺诈检测
                            交易限额
                            黑名单管理
                        
                    
                
                
                支付流程设计
                
                
                    支付流程
                    
                        
                            1. 创建支付
                            生成支付订单
                        
                        
                            2. 跳转支付
                            跳转到支付页面
                        
                        
                            3. 用户支付
                            用户完成支付
                        
                        
                            4. 结果通知
                            接收支付结果
                        
                        
                            5. 处理结果
                            更新订单状态
                        
                    
                    
                    支付状态机
                    
                        
                            
                                
                                    状态
                                    说明
                                    可流转状态
                                
                            
                            
                                
                                    待支付
                                    支付订单已创建
                                    支付中, 已取消
                                
                                
                                    支付中
                                    用户正在支付
                                    支付成功, 支付失败, 已取消
                                
                                
                                    支付成功
                                    支付完成
                                    已退款
                                
                                
                                    支付失败
                                    支付未完成
                                    待支付
                                
                            
                        
                    
                
                
                支付渠道对接
                
                
                    
                        
                            
                            支付宝支付
                        
                        
// 支付宝支付示例
AlipayClient alipayClient = new DefaultAlipayClient(
    "https://openapi.alipay.com/gateway.do",
    appId, privateKey, "json", charset, alipayPublicKey, "RSA2");

AlipayTradePagePayRequest request = new AlipayTradePagePayRequest();
request.setBizContent("{" +
    "\"out_trade_no\":\"20150320010101001\"," +
    "\"total_amount\":88.88," +
    "\"subject\":\"Iphone6 16G\"," +
    "\"product_code\":\"FAST_INSTANT_TRADE_PAY\"}");
    
String form = alipayClient.pageExecute(request).getBody();
                    
                    
                    
                        
                            
                            微信支付
                        
                        
// 微信支付示例
WXPay wxPay = new WXPay(config);
Map data = new HashMap();
data.put("body", "腾讯充值中心-QQ会员充值");
data.put("out_trade_no", "2016090910595900000012");
data.put("total_fee", "1");
data.put("spbill_create_ip", "123.12.12.123");
data.put("notify_url", "http://www.example.com/wxpay/notify");
data.put("trade_type", "NATIVE");

Map resp = wxPay.unifiedOrder(data);
                    
                
                
                数据模型设计
                
                
                    核心数据表
                    
                        
                            
                                
                                    表名
                                    字段
                                    说明
                                
                            
                            
                                
                                    payment_orders
                                    payment_id, order_id, amount, status, channel, create_time
                                    支付订单表
                                
                                
                                    payment_records
                                    record_id, payment_id, transaction_id, status, create_time
                                    支付记录表
                                
                                
                                    refund_orders
                                    refund_id, payment_id, amount, status, create_time
                                    退款订单表
                                
                                
                                    payment_channels
                                    channel_id, channel_name, config, status
                                    支付渠道配置表
                                
                            
                        
                    
                
                
                分布式事务
                
                
                    事务处理方案
                    
                        
                            TCC模式
                            
                                Try-Confirm-Cancel三阶段提交
                            
                            
                                Try阶段：检查并预留资源
                                Confirm阶段：确认执行
                                Cancel阶段：回滚操作
                            
                        
                        
                            Saga模式
                            
                                长事务解决方案
                            
                            
                                正向操作序列
                                补偿操作序列
                                事务协调器
                            
                        
                    
                    
                    支付事务示例
                    
// 基于TCC的支付处理
@TccTransaction(confirmMethod = "confirmPayment", cancelMethod = "cancelPayment")
public void preparePayment(PaymentRequest request) {
    // 1. 创建支付订单
    PaymentOrder paymentOrder = new PaymentOrder();
    paymentOrder.setOrderId(request.getOrderId());
    paymentOrder.setAmount(request.getAmount());
    paymentOrder.setStatus(PaymentStatus.PREPARING);
    paymentOrderMapper.insert(paymentOrder);
    
    // 2. 冻结用户账户资金
    accountService.freezeBalance(request.getUserId(), request.getAmount());
    
    // 3. 预占商品库存
    inventoryService.reserveStock(request.getProductId(), request.getQuantity());
}

public void confirmPayment(PaymentRequest request) {
    // 1. 更新支付订单状态
    paymentOrderMapper.updateStatus(request.getPaymentId(), PaymentStatus.SUCCESS);
    
    // 2. 扣减用户账户资金
    accountService.deductBalance(request.getUserId(), request.getAmount());
    
    // 3. 扣减商品库存
    inventoryService.deductStock(request.getProductId(), request.getQuantity());
}

public void cancelPayment(PaymentRequest request) {
    // 1. 更新支付订单状态
    paymentOrderMapper.updateStatus(request.getPaymentId(), PaymentStatus.CANCELLED);
    
    // 2. 解冻用户账户资金
    accountService.unfreezeBalance(request.getUserId(), request.getAmount());
    
    // 3. 释放商品库存
    inventoryService.releaseStock(request.getProductId(), request.getQuantity());
}
                
                
                安全与风控
                
                
                    
                        
                            
                            支付安全
                        
                        
                            数据加密传输（HTTPS）
                            敏感信息加密存储
                            签名验证
                            防重放攻击
                            防篡改校验
                        
                    
                    
                    
                        
                            
                            风控策略
                        
                        
                            用户行为分析
                            交易频率限制
                            异常IP检测
                            黑名单机制
                            多维度验证
                        
                    
                
                
                对账与结算
                
                
                    对账流程
                    
                        
                            数据准备
                            获取交易数据
                        
                        
                            数据比对
                            系统间数据核对
                        
                        
                            差异处理
                            处理不一致数据
                        
                        
                            结果确认
                            生成对账报告
                        
                    
                    
                    对账示例
                    
// 对账任务示例
@Component
public class ReconciliationTask {
    
    @Scheduled(cron = "0 0 2 * * ?") // 每天凌晨2点执行
    public void reconcile() {
        // 1. 获取系统内交易数据
        List localRecords = paymentRecordMapper
            .selectByDate(LocalDate.now().minusDays(1));
        
        // 2. 获取支付宝交易数据
        List alipayRecords = alipayService
            .downloadBill(LocalDate.now().minusDays(1));
        
        // 3. 数据比对
        ReconciliationResult result = reconciliationService
            .compare(localRecords, alipayRecords);
        
        // 4. 处理差异
        if (!result.isConsistent()) {
            handleDiscrepancy(result.getDiscrepancies());
        }
        
        // 5. 记录对账结果
        reconciliationResultMapper.insert(result);
    }
}
                
                
                监控与运维
                
                
                    
                        
                            
                            关键指标
                        
                        
                            支付成功率
                            支付平均耗时
                            退款率
                            对账一致率
                            异常交易数
                        
                    
                    
                    
                        
                            
                            运维保障
                        
                        
                            多活部署
                            灾备切换
                            容量规划
                            应急预案
                            故障演练
                        
                    
                
                
                支付系统最佳实践
                
                    
                        标准化支付接口设计
                        完善的异常处理机制
                        健壮的分布式事务方案
                        全面的安全防护措施
                        精确的对账和结算流程
                        详细的监控和告警体系
                        充分的压力测试和容量评估
