\n\n    
    
    业务模块 - B2B分布式项目
    
    
    
        .feature-card {
            transition: all 0.3s ease;
        }
        .feature-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 25px rgba(0,0,0,0.15);
        }
        .flow-node {
            transition: all 0.3s ease;
        }
        .flow-node:hover {
            transform: scale(1.05);
        }
    \n\n    
        
            
                合同中心
            
            Business Modules - 核心业务功能实现
            
            
                
                    
                    核心定位：合同中心作为业务模块的核心，提供合同全生命周期管理，支持多种交收方式、支付方式、签章、付款、配货、开票等功能，同时处理复杂的资金组合方式。
                
            

            
                功能概览
            
            
            
                
                    
                    合同中心
                    合同全生命周期
                
                
                    
                    交收方式
                    组织交收、自由交收
                
                
                    
                    支付方式
                    多种支付方式
                
                
                    
                    合同签章
                    电子签章
                
                
                    
                    合同配货
                    对接仓单物流
                
                
                    
                    合同开票
                    对接开票服务
                
                
                    
                    资金管理
                    预付款、手续费、货款、短溢货款、开票保证金
                
            

            
                核心功能实现
            
            
            
                
                    
                        
                            
                        
                        合同中心基础功能
                    
                    
                        提供合同的创建、查询、修改、删除等基础功能，支持合同全生命周期管理。
                    
                    
                        @Service
public class ContractService {
    
    @Autowired
    private ContractRepository contractRepository;
    
    @Autowired
    private ContractItemRepository contractItemRepository;
    
    @Autowired
    private IdGenerator idGenerator;
    
    public Contract createContract(CreateContractRequest request) {
        String contractId = idGenerator.generate();
        
        Contract contract = Contract.builder()
            .contractId(contractId)
            .contractNo(generateContractNo())
            .buyerId(request.getBuyerId())
            .sellerId(request.getSellerId())
            .contractType(request.getContractType())
            .settlementType(request.getSettlementType())
            .paymentType(request.getPaymentType())
            .totalAmount(request.getTotalAmount())
            .status(ContractStatus.DRAFT)
            .createTime(new Date())
            .build();
        
        List items = request.getItems().stream()
            .map(item -> ContractItem.builder()
                .contractId(contractId)
                .productId(item.getProductId())
                .productName(item.getProductName())
                .quantity(item.getQuantity())
                .price(item.getPrice())
                .amount(item.getAmount())
                .build())
            .collect(Collectors.toList());
        
        contractRepository.save(contract);
        contractItemRepository.saveAll(items);
        
        return contract;
    }
    
    public Contract getContract(String contractId) {
        return contractRepository.findByContractId(contractId);
    }
    
    public void updateContract(String contractId, UpdateContractRequest request) {
        Contract contract = contractRepository.findByContractId(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.DRAFT) {
            throw new BusinessException("只有草稿状态的合同可以修改");
        }
        
        if (request.getTotalAmount() != null) {
            contract.setTotalAmount(request.getTotalAmount());
        }
        if (request.getSettlementType() != null) {
            contract.setSettlementType(request.getSettlementType());
        }
        if (request.getPaymentType() != null) {
            contract.setPaymentType(request.getPaymentType());
        }
        if (request.getRemark() != null) {
            contract.setRemark(request.getRemark());
        }
        
        contractRepository.save(contract);
    }
    
    public void submitContract(String contractId) {
        Contract contract = contractRepository.findByContractId(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.DRAFT) {
            throw new BusinessException("只有草稿状态的合同可以提交");
        }
        
        contract.setStatus(ContractStatus.PENDING);
        contract.setSubmitTime(new Date());
        contractRepository.save(contract);
    }
    
    public void approveContract(String contractId, Long approverId, String approvalOpinion) {
        Contract contract = contractRepository.findByContractId(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.PENDING) {
            throw new BusinessException("只有待审批状态的合同可以审批");
        }
        
        contract.setStatus(ContractStatus.APPROVED);
        contract.setApproverId(approverId);
        contract.setApprovalOpinion(approvalOpinion);
        contract.setApprovalTime(new Date());
        contractRepository.save(contract);
    }
    
    public void rejectContract(String contractId, Long approverId, String rejectReason) {
        Contract contract = contractRepository.findByContractId(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.PENDING) {
            throw new BusinessException("只有待审批状态的合同可以审批");
        }
        
        contract.setStatus(ContractStatus.REJECTED);
        contract.setApproverId(approverId);
        contract.setRejectReason(rejectReason);
        contract.setApprovalTime(new Date());
        contractRepository.save(contract);
    }
    
    public List queryContracts(ContractQueryRequest request) {
        return contractRepository.queryByCondition(request);
    }
    
    private String generateContractNo() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
        String dateStr = sdf.format(new Date());
        String sequence = String.format("%06d", contractRepository.countByCreateTimeLike(dateStr + "%") + 1);
        return "HT" + dateStr + sequence;
    }
}

public enum ContractStatus {
    DRAFT("草稿"),
    PENDING("待审批"),
    APPROVED("已审批"),
    REJECTED("已拒绝"),
    SIGNED("已签章"),
    PAID("已付款"),
    ALLOCATED("已配货"),
    SHIPPED("已发货"),
    DELIVERED("已收货"),
    INVOICED("已开票"),
    COMPLETED("已完成"),
    CANCELLED("已取消");
    
    private final String desc;
    
    ContractStatus(String desc) {
        this.desc = desc;
    }
}

public enum SettlementType {
    ORGANIZED("组织交收"),
    FREE("自由交收");
    
    private final String desc;
    
    SettlementType(String desc) {
        this.desc = desc;
    }
}

@Entity
@Table(name = "contract")
public class Contract {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String contractId;
    
    private String contractNo;
    
    private Long buyerId;
    
    private Long sellerId;
    
    private String contractType;
    
    @Enumerated(EnumType.STRING)
    private SettlementType settlementType;
    
    private String paymentType;
    
    private BigDecimal totalAmount;
    
    @Enumerated(EnumType.STRING)
    private ContractStatus status;
    
    private Date createTime;
    
    private Date submitTime;
    
    private Long approverId;
    
    private String approvalOpinion;
    
    private String rejectReason;
    
    private Date approvalTime;
    
    private String remark;
}

@Entity
@Table(name = "contract_item")
public class ContractItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String contractId;
    
    private Long productId;
    
    private String productName;
    
    private BigDecimal quantity;
    
    private BigDecimal price;
    
    private BigDecimal amount;
}
                    
                

                
                    
                        
                            
                        
                        多种交收方式流程
                    
                    
                        支持组织交收和自由交收两种交收方式，每种方式都有完整的流程处理。
                    
                    
                    
                        组织交收流程
                        
                            
                                
                                合同审批
                            
                            
                            
                                
                                合同签章
                            
                            
                            
                                
                                支付货款
                            
                            
                            
                                
                                仓单过户
                            
                            
                            
                                
                                交收完成
                            
                        
                    
                    
                    
                        自由交收流程
                        
                            
                                
                                合同审批
                            
                            
                            
                                
                                合同签章
                            
                            
                            
                                
                                自行发货
                            
                            
                            
                                
                                物流配送
                            
                            
                            
                                
                                确认收货
                            
                            
                            
                                
                                交收完成
                            
                        
                    
                    
                    
                        @Service
public class SettlementService {
    
    @Autowired
    private ContractService contractService;
    
    @Autowired
    private WarehouseReceiptService warehouseReceiptService;
    
    @Autowired
    private LogisticsService logisticsService;
    
    public void startSettlement(String contractId) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.APPROVED && contract.getStatus() != ContractStatus.SIGNED) {
            throw new BusinessException("只有已审批或已签章的合同可以开始交收");
        }
        
        if (contract.getSettlementType() == SettlementType.ORGANIZED) {
            startOrganizedSettlement(contract);
        } else {
            startFreeSettlement(contract);
        }
    }
    
    private void startOrganizedSettlement(Contract contract) {
        contract.setStatus(ContractStatus.SIGNED);
        contractService.updateContractStatus(contract.getContractId(), ContractStatus.SIGNED);
    }
    
    private void startFreeSettlement(Contract contract) {
        contract.setStatus(ContractStatus.SIGNED);
        contractService.updateContractStatus(contract.getContractId(), ContractStatus.SIGNED);
    }
    
    public void allocateWarehouseReceipt(String contractId, List warehouseReceiptIds) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getSettlementType() != SettlementType.ORGANIZED) {
            throw new BusinessException("只有组织交收的合同可以分配仓单");
        }
        
        if (contract.getStatus() != ContractStatus.PAID) {
            throw new BusinessException("只有已付款的合同可以分配仓单");
        }
        
        for (String warehouseReceiptId : warehouseReceiptIds) {
            warehouseReceiptService.transfer(warehouseReceiptId, contract.getBuyerId());
        }
        
        contract.setStatus(ContractStatus.ALLOCATED);
        contractService.updateContractStatus(contractId, ContractStatus.ALLOCATED);
    }
    
    public void arrangeDelivery(String contractId, DeliveryRequest request) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getSettlementType() != SettlementType.FREE) {
            throw new BusinessException("只有自由交收的合同可以安排发货");
        }
        
        if (contract.getStatus() != ContractStatus.SIGNED) {
            throw new BusinessException("只有已签章的合同可以安排发货");
        }
        
        LogisticsOrder logisticsOrder = logisticsService.createOrder(
            request.getLogisticsCompany(),
            request.getSenderAddress(),
            request.getReceiverAddress(),
            request.getItems()
        );
        
        contract.setStatus(ContractStatus.SHIPPED);
        contract.setLogisticsOrderId(logisticsOrder.getOrderId());
        contractService.updateContractStatus(contractId, ContractStatus.SHIPPED);
    }
    
    public void confirmDelivery(String contractId) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.SHIPPED && contract.getStatus() != ContractStatus.ALLOCATED) {
            throw new BusinessException("只有已发货或已配货的合同可以确认收货");
        }
        
        contract.setStatus(ContractStatus.DELIVERED);
        contract.setDeliverTime(new Date());
        contractService.updateContractStatus(contractId, ContractStatus.DELIVERED);
    }
    
    public void completeSettlement(String contractId) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.DELIVERED) {
            throw new BusinessException("只有已收货的合同可以完成交收");
        }
        
        contract.setStatus(ContractStatus.COMPLETED);
        contract.setCompleteTime(new Date());
        contractService.updateContractStatus(contractId, ContractStatus.COMPLETED);
    }
}

@Data
public class DeliveryRequest {
    private String logisticsCompany;
    private String senderAddress;
    private String receiverAddress;
    private List items;
}

@Data
public class DeliveryItem {
    private Long productId;
    private String productName;
    private BigDecimal quantity;
}
                    
                

                
                    
                        
                            
                        
                        合同签章与付款
                    
                    
                        提供合同电子签章、合同付款等功能，支持多种支付方式。
                    
                    
                        @Service
public class ContractSignatureService {
    
    @Autowired
    private ContractService contractService;
    
    @Autowired
    private EsignService esignService;
    
    public SignResult signContract(String contractId, Long signerId, String signerRole) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.APPROVED) {
            throw new BusinessException("只有已审批的合同可以签章");
        }
        
        EsignRequest esignRequest = EsignRequest.builder()
            .contractId(contractId)
            .signerId(signerId)
            .signerRole(signerRole)
            .build();
        
        EsignResult esignResult = esignService.sign(esignRequest);
        
        if (esignResult.isSuccess()) {
            if ("buyer".equals(signerRole)) {
                contract.setBuyerSigned(true);
                contract.setBuyerSignTime(new Date());
            } else if ("seller".equals(signerRole)) {
                contract.setSellerSigned(true);
                contract.setSellerSignTime(new Date());
            }
            
            if (contract.isBuyerSigned() && contract.isSellerSigned()) {
                contract.setStatus(ContractStatus.SIGNED);
                contract.setSignTime(new Date());
            }
            
            contractService.updateContract(contract);
        }
        
        return SignResult.builder()
            .success(esignResult.isSuccess())
            .signUrl(esignResult.getSignUrl())
            .build();
    }
    
    public boolean isContractSigned(String contractId) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            return false;
        }
        return contract.isBuyerSigned() && contract.isSellerSigned();
    }
}

@Service
public class ContractPaymentService {
    
    @Autowired
    private ContractService contractService;
    
    @Autowired
    private PaymentService paymentService;
    
    @Autowired
    private CapitalFlowService capitalFlowService;
    
    public PaymentResult payContract(String contractId, PaymentRequest request) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.SIGNED) {
            throw new BusinessException("只有已签章的合同可以付款");
        }
        
        BigDecimal payableAmount = calculatePayableAmount(contract);
        
        PaymentResult paymentResult = paymentService.pay(
            request.getPaymentType(),
            payableAmount,
            request.getExtra()
        );
        
        if (paymentResult.isSuccess()) {
            capitalFlowService.createFlow(
                contractId,
                "PAYMENT",
                payableAmount,
                "合同付款"
            );
            
            contract.setStatus(ContractStatus.PAID);
            contract.setPayTime(new Date());
            contractService.updateContractStatus(contractId, ContractStatus.PAID);
        }
        
        return paymentResult;
    }
    
    private BigDecimal calculatePayableAmount(Contract contract) {
        return contract.getTotalAmount();
    }
}

@Data
@Builder
public class SignResult {
    private boolean success;
    private String signUrl;
    private String errorMessage;
}

@Data
public class PaymentRequest {
    private String paymentType;
    private Map extra;
}

@Data
@Builder
public class PaymentResult {
    private boolean success;
    private String paymentId;
    private String paymentUrl;
    private String errorMessage;
}
                    
                

                
                    
                        
                            
                        
                        合同配货与对接仓单物流系统
                    
                    
                        提供合同配货功能，对接仓单系统和物流系统，支持货物调配和物流跟踪。
                    
                    
                        @Service
public class ContractAllocationService {
    
    @Autowired
    private ContractService contractService;
    
    @Autowired
    private WarehouseReceiptService warehouseReceiptService;
    
    @Autowired
    private LogisticsService logisticsService;
    
    @Autowired
    private ContractItemRepository contractItemRepository;
    
    public AllocationResult allocateGoods(String contractId, AllocationRequest request) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.PAID) {
            throw new BusinessException("只有已付款的合同可以配货");
        }
        
        List items = contractItemRepository.findByContractId(contractId);
        
        List allocationItems = new ArrayList();
        for (ContractItem item : items) {
            AllocationRequest.Item requestItem = request.getItems().stream()
                .filter(i -> i.getProductId().equals(item.getProductId()))
                .findFirst()
                .orElse(null);
            
            if (requestItem == null) {
                throw new BusinessException("商品 " + item.getProductName() + " 未分配");
            }
            
            if (requestItem.getQuantity().compareTo(item.getQuantity()) != 0) {
                throw new BusinessException("商品 " + item.getProductName() + " 数量不匹配");
            }
            
            WarehouseReceipt warehouseReceipt = warehouseReceiptService.getById(requestItem.getWarehouseReceiptId());
            if (warehouseReceipt == null) {
                throw new BusinessException("仓单不存在");
            }
            
            allocationItems.add(AllocationItem.builder()
                .productId(item.getProductId())
                .productName(item.getProductName())
                .quantity(item.getQuantity())
                .warehouseReceiptId(requestItem.getWarehouseReceiptId())
                .warehouseId(warehouseReceipt.getWarehouseId())
                .build());
        }
        
        ContractAllocation allocation = ContractAllocation.builder()
            .contractId(contractId)
            .items(allocationItems)
            .createTime(new Date())
            .build();
        
        contract.setStatus(ContractStatus.ALLOCATED);
        contract.setAllocationTime(new Date());
        contractService.updateContractStatus(contractId, ContractStatus.ALLOCATED);
        
        return AllocationResult.builder()
            .success(true)
            .allocationId(allocation.getId())
            .build();
    }
    
    public void createLogisticsOrder(String contractId, LogisticsOrderRequest request) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.ALLOCATED) {
            throw new BusinessException("只有已配货的合同可以创建物流单");
        }
        
        LogisticsOrder logisticsOrder = logisticsService.createOrder(
            request.getLogisticsCompany(),
            request.getSenderAddress(),
            request.getReceiverAddress(),
            request.getItems()
        );
        
        contract.setStatus(ContractStatus.SHIPPED);
        contract.setLogisticsOrderId(logisticsOrder.getOrderId());
        contract.setShipTime(new Date());
        contractService.updateContractStatus(contractId, ContractStatus.SHIPPED);
    }
    
    public LogisticsTraceResult getLogisticsTrace(String contractId) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getLogisticsOrderId() == null) {
            throw new BusinessException("物流单号不存在");
        }
        
        return logisticsService.getTrace(contract.getLogisticsOrderId());
    }
}

@Data
public class AllocationRequest {
    private List items;
    
    @Data
    public static class Item {
        private Long productId;
        private BigDecimal quantity;
        private String warehouseReceiptId;
    }
}

@Data
@Builder
public class AllocationResult {
    private boolean success;
    private String allocationId;
    private String errorMessage;
}

@Data
@Builder
public class AllocationItem {
    private Long productId;
    private String productName;
    private BigDecimal quantity;
    private String warehouseReceiptId;
    private Long warehouseId;
}

@Data
public class LogisticsOrderRequest {
    private String logisticsCompany;
    private String senderAddress;
    private String receiverAddress;
    private List items;
}

@Data
public class LogisticsItem {
    private Long productId;
    private String productName;
    private BigDecimal quantity;
}

@Data
@Builder
public class LogisticsTraceResult {
    private String orderId;
    private List traces;
}

@Data
@Builder
public class TraceNode {
    private Date time;
    private String location;
    private String description;
}
                    
                

                
                    
                        
                            
                        
                        合同开票与对接开票服务
                    
                    
                        提供合同开票功能，对接开票服务，支持发票申请、开具、查询等功能。
                    
                    
                        @Service
public class ContractInvoiceService {
    
    @Autowired
    private ContractService contractService;
    
    @Autowired
    private InvoiceService invoiceService;
    
    public InvoiceResult applyInvoice(String contractId, InvoiceRequest request) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getStatus() != ContractStatus.DELIVERED && contract.getStatus() != ContractStatus.COMPLETED) {
            throw new BusinessException("只有已收货或已完成的合同可以开票");
        }
        
        BigDecimal invoiceAmount = calculateInvoiceAmount(contract, request);
        
        InvoiceApplication application = InvoiceApplication.builder()
            .contractId(contractId)
            .invoiceType(request.getInvoiceType())
            .invoiceAmount(invoiceAmount)
            .buyerName(request.getBuyerName())
            .buyerTaxNo(request.getBuyerTaxNo())
            .buyerAddress(request.getBuyerAddress())
            .buyerPhone(request.getBuyerPhone())
            .buyerBankName(request.getBuyerBankName())
            .buyerBankAccount(request.getBuyerBankAccount())
            .items(request.getItems())
            .status(InvoiceApplicationStatus.PENDING)
            .createTime(new Date())
            .build();
        
        InvoiceResult invoiceResult = invoiceService.applyInvoice(application);
        
        if (invoiceResult.isSuccess()) {
            application.setInvoiceId(invoiceResult.getInvoiceId());
            application.setStatus(InvoiceApplicationStatus.APPLIED);
            
            contract.setStatus(ContractStatus.INVOICED);
            contract.setInvoiceTime(new Date());
            contractService.updateContractStatus(contractId, ContractStatus.INVOICED);
        }
        
        return invoiceResult;
    }
    
    public InvoiceResult queryInvoice(String contractId) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        return invoiceService.queryInvoice(contract.getInvoiceId());
    }
    
    public void downloadInvoice(String contractId, String filePath) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (contract.getInvoiceId() == null) {
            throw new BusinessException("发票不存在");
        }
        
        invoiceService.downloadInvoice(contract.getInvoiceId(), filePath);
    }
    
    private BigDecimal calculateInvoiceAmount(Contract contract, InvoiceRequest request) {
        if (request.getInvoiceAmount() != null) {
            return request.getInvoiceAmount();
        }
        return contract.getTotalAmount();
    }
}

public enum InvoiceApplicationStatus {
    PENDING("待申请"),
    APPLIED("已申请"),
    ISSUED("已开具"),
    FAILED("申请失败");
    
    private final String desc;
    
    InvoiceApplicationStatus(String desc) {
        this.desc = desc;
    }
}

@Data
public class InvoiceRequest {
    private String invoiceType;
    private BigDecimal invoiceAmount;
    private String buyerName;
    private String buyerTaxNo;
    private String buyerAddress;
    private String buyerPhone;
    private String buyerBankName;
    private String buyerBankAccount;
    private List items;
}

@Data
public class InvoiceItem {
    private String productName;
    private String specification;
    private String unit;
    private BigDecimal quantity;
    private BigDecimal price;
    private BigDecimal amount;
    private String taxRate;
}

@Data
@Builder
public class InvoiceResult {
    private boolean success;
    private String invoiceId;
    private String invoiceNo;
    private String invoiceUrl;
    private String errorMessage;
}
                    
                

                
                    
                        
                            
                        
                        复杂资金组合方式
                    
                    
                        处理复杂的资金组合方式，包括预付款、手续费、货款、多发货少发货时的短溢货款、开票保证金等。
                    
                    
                    
                        
                            
                                
                                    资金类型
                                    说明
                                    计算方式
                                
                            
                            
                                
                                    预付款
                                    合同签订前支付的定金
                                    合同金额 × 预付比例
                                
                                
                                    手续费
                                    交易服务费用
                                    合同金额 × 手续费率
                                
                                
                                    货款
                                    货物实际金额
                                    实际发货数量 × 单价
                                
                                
                                    短溢货款
                                    多发货或少发货的差额
                                    （实际数量 - 合同数量）× 单价
                                
                                
                                    开票保证金
                                    开票前冻结的保证金
                                    合同金额 × 保证金比例
                                
                            
                        
                    
                    
                    
                        @Service
public class CapitalService {
    
    @Autowired
    private ContractService contractService;
    
    @Autowired
    private CapitalFlowRepository capitalFlowRepository;
    
    public CapitalCalculationResult calculateCapital(String contractId) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        BigDecimal contractAmount = contract.getTotalAmount();
        
        BigDecimal prepayment = calculatePrepayment(contract);
        BigDecimal serviceFee = calculateServiceFee(contract);
        BigDecimal goodsPayment = calculateGoodsPayment(contract);
        BigDecimal shortOverPayment = calculateShortOverPayment(contract);
        BigDecimal invoiceDeposit = calculateInvoiceDeposit(contract);
        
        BigDecimal totalPayable = prepayment.add(serviceFee).add(goodsPayment)
            .add(shortOverPayment.compareTo(BigDecimal.ZERO) > 0 ? shortOverPayment : BigDecimal.ZERO);
        
        return CapitalCalculationResult.builder()
            .contractAmount(contractAmount)
            .prepayment(prepayment)
            .serviceFee(serviceFee)
            .goodsPayment(goodsPayment)
            .shortOverPayment(shortOverPayment)
            .invoiceDeposit(invoiceDeposit)
            .totalPayable(totalPayable)
            .build();
    }
    
    private BigDecimal calculatePrepayment(Contract contract) {
        BigDecimal prepaymentRate = contract.getPrepaymentRate() != null ? 
            contract.getPrepaymentRate() : new BigDecimal("0.3");
        return contract.getTotalAmount().multiply(prepaymentRate);
    }
    
    private BigDecimal calculateServiceFee(Contract contract) {
        BigDecimal serviceFeeRate = contract.getServiceFeeRate() != null ? 
            contract.getServiceFeeRate() : new BigDecimal("0.005");
        return contract.getTotalAmount().multiply(serviceFeeRate);
    }
    
    private BigDecimal calculateGoodsPayment(Contract contract) {
        List items = contractItemRepository.findByContractId(contract.getContractId());
        BigDecimal actualQuantity = items.stream()
            .map(ContractItem::getActualQuantity)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        BigDecimal contractQuantity = items.stream()
            .map(ContractItem::getQuantity)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        if (actualQuantity.compareTo(BigDecimal.ZERO) == 0) {
            return contract.getTotalAmount();
        }
        
        return items.stream()
            .map(item -> item.getActualQuantity() != null ? 
                item.getActualQuantity().multiply(item.getPrice()) : item.getAmount())
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    private BigDecimal calculateShortOverPayment(Contract contract) {
        List items = contractItemRepository.findByContractId(contract.getContractId());
        
        return items.stream()
            .map(item -> {
                BigDecimal actualQty = item.getActualQuantity() != null ? item.getActualQuantity() : item.getQuantity();
                BigDecimal diff = actualQty.subtract(item.getQuantity());
                return diff.multiply(item.getPrice());
            })
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    private BigDecimal calculateInvoiceDeposit(Contract contract) {
        BigDecimal invoiceDepositRate = contract.getInvoiceDepositRate() != null ? 
            contract.getInvoiceDepositRate() : new BigDecimal("0.1");
        return contract.getTotalAmount().multiply(invoiceDepositRate);
    }
    
    public void createCapitalFlow(String contractId, String flowType, BigDecimal amount, String remark) {
        CapitalFlow flow = CapitalFlow.builder()
            .contractId(contractId)
            .flowType(flowType)
            .amount(amount)
            .remark(remark)
            .createTime(new Date())
            .build();
        
        capitalFlowRepository.save(flow);
    }
    
    public List getCapitalFlows(String contractId) {
        return capitalFlowRepository.findByContractIdOrderByCreateTimeDesc(contractId);
    }
    
    public void freezeInvoiceDeposit(String contractId) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        CapitalCalculationResult calculation = calculateCapital(contractId);
        
        createCapitalFlow(contractId, "INVOICE_DEPOSIT_FREEZE", 
            calculation.getInvoiceDeposit(), "开票保证金冻结");
        
        contract.setInvoiceDepositFrozen(true);
        contractService.updateContract(contract);
    }
    
    public void unfreezeInvoiceDeposit(String contractId) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        if (!contract.isInvoiceDepositFrozen()) {
            throw new BusinessException("开票保证金未冻结");
        }
        
        CapitalCalculationResult calculation = calculateCapital(contractId);
        
        createCapitalFlow(contractId, "INVOICE_DEPOSIT_UNFREEZE", 
            calculation.getInvoiceDeposit().negate(), "开票保证金解冻");
        
        contract.setInvoiceDepositFrozen(false);
        contractService.updateContract(contract);
    }
    
    public void settleShortOverPayment(String contractId, BigDecimal settleAmount) {
        Contract contract = contractService.getContract(contractId);
        if (contract == null) {
            throw new BusinessException("合同不存在");
        }
        
        CapitalCalculationResult calculation = calculateCapital(contractId);
        
        String flowType = settleAmount.compareTo(BigDecimal.ZERO) >= 0 ? 
            "SHORT_OVER_PAYMENT_RECEIVE" : "SHORT_OVER_PAYMENT_PAY";
        
        createCapitalFlow(contractId, flowType, settleAmount.abs(), 
            settleAmount.compareTo(BigDecimal.ZERO) >= 0 ? "短溢货款收取" : "短溢货款支付");
        
        contract.setShortOverPaymentSettled(true);
        contractService.updateContract(contract);
    }
}

@Data
@Builder
public class CapitalCalculationResult {
    private BigDecimal contractAmount;
    private BigDecimal prepayment;
    private BigDecimal serviceFee;
    private BigDecimal goodsPayment;
    private BigDecimal shortOverPayment;
    private BigDecimal invoiceDeposit;
    private BigDecimal totalPayable;
}

@Entity
@Table(name = "capital_flow")
public class CapitalFlow {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String contractId;
    
    private String flowType;
    
    private BigDecimal amount;
    
    private String remark;
    
    private Date createTime;
}
                    
                
            

            
                核心优势
            
            
            
                
                    
                    全生命周期
                    合同从创建到完成全流程
                
                
                    
                    灵活交收
                    组织交收和自由交收
                
                
                    
                    资金灵活
                    多种资金组合方式
                
                
                    
                    电子签章
                    安全便捷的电子签章
                
                
                    
                    系统集成
                    对接仓单、物流、开票系统
                
                
                    
                    资金安全
                    完善的资金管理机制
                
            

        
    \n\n
