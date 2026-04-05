促销系统 - 架构师学习笔记
    
    


    
        
            
                促销系统
            
            
            
                
                    促销系统是电商系统中提升用户购买转化率和客单价的重要模块，通过各种促销活动吸引用户消费。促销系统需要支持多种促销类型、复杂的计算规则、高并发处理以及灵活的配置管理。
                
                
                促销系统功能
                
                
                    
                        
                            
                            优惠券管理
                        
                        
                            优惠券创建和配置
                            优惠券发放和领取
                            优惠券使用规则
                            优惠券统计分析
                        
                    
                    
                    
                        
                            
                            活动管理
                        
                        
                            满减活动
                            折扣活动
                            秒杀活动
                            拼团活动
                        
                    
                    
                    
                        
                            
                            价格计算
                        
                        
                            促销规则引擎
                            价格计算逻辑
                            优惠叠加规则
                            实时价格展示
                        
                    
                    
                    
                        
                            
                            配置管理
                        
                        
                            促销规则配置
                            活动时间管理
                            用户参与限制
                            黑白名单控制
                        
                    
                
                
                促销类型设计
                
                
                    优惠券类型
                    
                        
                            满减券
                            
                                满足指定金额后减免部分金额
                            
                            例如：满100减20
                        
                        
                            折扣券
                            
                                按指定比例折扣商品价格
                            
                            例如：9折优惠
                        
                        
                            无门槛券
                            
                                无消费金额限制直接减免
                            
                            例如：无门槛减10元
                        
                    
                    
                    活动类型
                    
                        
                            秒杀活动
                            
                                限时限量特价销售
                            
                        
                        
                            拼团活动
                            
                                多人成团享受优惠价
                            
                        
                        
                            预售活动
                            
                                提前支付定金锁定商品
                            
                        
                        
                            会员专享
                            
                                特定会员等级享受优惠
                            
                        
                    
                
                
                促销规则引擎
                
                
                    规则设计
                    
                        
                            
                                
                                    规则类型
                                    条件
                                    动作
                                    优先级
                                
                            
                            
                                
                                    商品范围
                                    指定商品、品类、品牌
                                    适用优惠
                                    1
                                
                                
                                    用户条件
                                    会员等级、新老用户
                                    差异化优惠
                                    2
                                
                                
                                    时间条件
                                    活动时间、星期几
                                    限时优惠
                                    3
                                
                                
                                    金额条件
                                    满减、满赠
                                    阶梯优惠
                                    4
                                
                            
                        
                    
                    
                    规则引擎示例
                    
// 促销规则引擎示例
public class PromotionEngine {
    
    public PromotionResult calculatePromotion(Cart cart, List promotions) {
        PromotionResult result = new PromotionResult();
        List applicablePromotions = new ArrayList();
        
        // 1. 筛选适用的促销活动
        for (CartItem item : cart.getItems()) {
            for (Promotion promotion : promotions) {
                if (promotion.isApplicable(item)) {
                    applicablePromotions.add(promotion);
                }
            }
        }
        
        // 2. 按优先级排序
        applicablePromotions.sort(Comparator.comparing(Promotion::getPriority));
        
        // 3. 应用促销规则
        BigDecimal totalDiscount = BigDecimal.ZERO;
        for (Promotion promotion : applicablePromotions) {
            PromotionResult promotionResult = promotion.apply(cart);
            totalDiscount = totalDiscount.add(promotionResult.getDiscount());
        }
        
        result.setTotalDiscount(totalDiscount);
        result.setApplicablePromotions(applicablePromotions);
        return result;
    }
}
                
                
                数据模型设计
                
                
                    核心数据表
                    
                        
                            
                                
                                    表名
                                    字段
                                    说明
                                
                            
                            
                                
                                    coupons
                                    coupon_id, name, type, discount, condition, start_time, end_time
                                    优惠券主表
                                
                                
                                    user_coupons
                                    user_coupon_id, user_id, coupon_id, status, receive_time, use_time
                                    用户优惠券表
                                
                                
                                    promotions
                                    promotion_id, name, type, rule, start_time, end_time
                                    促销活动表
                                
                                
                                    promotion_rules
                                    rule_id, promotion_id, condition, action
                                    促销规则表
                                
                            
                        
                    
                
                
                高并发处理
                
                
                    
                        
                            
                            秒杀场景优化
                        
                        
                            库存预热到缓存
                            限流控制
                            异步下单
                            排队机制
                        
                    
                    
                    
                        
                            
                            防刷策略
                        
                        
                            频率限制
                            用户行为分析
                            设备指纹识别
                            黑名单机制
                        
                    
                
                
                优惠叠加规则
                
                
                    叠加策略
                    
                        
                            互斥策略
                            
                                优惠活动不能同时使用
                            
                        
                        
                            并行策略
                            
                                优惠活动可以同时享受
                            
                        
                        
                            优先级策略
                            
                                按优先级顺序应用优惠
                            
                        
                    
                    
                    叠加规则示例
                    
// 优惠叠加处理示例
public class PromotionStackingService {
    
    public List resolvePromotionConflicts(List promotions) {
        // 1. 按优先级排序
        promotions.sort(Comparator.comparing(Promotion::getPriority));
        
        // 2. 处理互斥关系
        List resolvedPromotions = new ArrayList();
        Set appliedCategories = new HashSet();
        
        for (Promotion promotion : promotions) {
            // 检查是否与已应用的促销互斥
            if (!isMutuallyExclusive(promotion, resolvedPromotions)) {
                // 检查品类冲突
                if (!hasCategoryConflict(promotion, appliedCategories)) {
                    resolvedPromotions.add(promotion);
                    appliedCategories.addAll(promotion.getCategories());
                }
            }
        }
        
        return resolvedPromotions;
    }
    
    private boolean isMutuallyExclusive(Promotion newPromotion, List appliedPromotions) {
        for (Promotion applied : appliedPromotions) {
            if (newPromotion.getMutuallyExclusivePromotions().contains(applied.getId())) {
                return true;
            }
        }
        return false;
    }
}
                
                
                实时价格计算
                
                
                    计算流程
                    
                        
                            1. 获取商品
                            商品信息
                        
                        
                            2. 查询促销
                            适用活动
                        
                        
                            3. 应用规则
                            计算优惠
                        
                        
                            4. 叠加处理
                            优惠组合
                        
                        
                            5. 返回价格
                            最终价格
                        
                    
                    
                    价格计算示例
                    
// 实时价格计算示例
@Service
public class PriceCalculationService {
    
    public ProductPrice calculatePrice(String productId, String userId) {
        // 1. 获取商品信息
        Product product = productRepository.findById(productId);
        
        // 2. 获取用户可用优惠券
        List userCoupons = couponService.getUserAvailableCoupons(userId);
        
        // 3. 获取当前适用的促销活动
        List activePromotions = promotionService.getActivePromotions();
        
        // 4. 计算基础价格
        BigDecimal basePrice = product.getPrice();
        
        // 5. 应用促销优惠
        BigDecimal promotionDiscount = applyPromotions(basePrice, activePromotions, product);
        
        // 6. 应用优惠券
        Coupon bestCoupon = selectBestCoupon(basePrice.subtract(promotionDiscount), userCoupons);
        BigDecimal couponDiscount = calculateCouponDiscount(basePrice.subtract(promotionDiscount), bestCoupon);
        
        // 7. 计算最终价格
        BigDecimal finalPrice = basePrice.subtract(promotionDiscount).subtract(couponDiscount);
        
        return ProductPrice.builder()
            .originalPrice(basePrice)
            .promotionDiscount(promotionDiscount)
            .couponDiscount(couponDiscount)
            .finalPrice(finalPrice)
            .build();
    }
}
                
                
                监控与统计
                
                
                    
                        
                            
                            关键指标
                        
                        
                            优惠券使用率
                            活动参与率
                            促销转化率
                            客单价提升幅度
                        
                    
                    
                    
                        
                            
                            运营分析
                        
                        
                            活动效果评估
                            用户行为分析
                            ROI计算
                            趋势预测
                        
                    
                
                
                促销系统最佳实践
                
                    
                        灵活的促销规则引擎设计
                        完善的优惠叠加处理机制
                        高效的高并发处理方案
                        全面的防刷和限流策略
                        准确的实时价格计算
                        详细的统计分析功能
                        健全的监控和告警体系
