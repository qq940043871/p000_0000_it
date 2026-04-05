\n\n    
    
    认证模块 - B2B分布式项目
    
    
    
        .feature-card {
            transition: all 0.3s ease;
        }
        .feature-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 25px rgba(0,0,0,0.15);
        }
    \n\n    
        
            
                认证模块
            
            Authentication Module - 统一身份认证与权限管理
            
            
                
                    
                    核心定位：认证模块提供统一的身份认证、权限管理、多方式登录、账户安全等功能，支持手机号对应多账户、多种登录方式、账户锁定、多终端登录配置等特性。
                
            

            
                功能概览
            
            
            
                
                    
                    用户权限加载
                    菜单权限、接口权限
                
                
                    
                    多账户支持
                    手机号对应多账户
                
                
                    
                    多种登录方式
                    验证码、密码、第三方
                
                
                    
                    账户锁定
                    锁定、注销、失败锁定
                
                
                    
                    多终端管理
                    多终端登录配置
                
                
                    
                    单设备登录
                    单设备限制
                
            

            
                核心功能实现
            
            
            
                
                    
                        
                            
                        
                        用户信息与权限加载
                    
                    
                        根据不同用户加载用户信息、菜单权限和接口权限。
                    
                    
                        @Service
public class UserPermissionService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private RoleRepository roleRepository;
    
    @Autowired
    private MenuRepository menuRepository;
    
    @Autowired
    private ApiPermissionRepository apiPermissionRepository;
    
    public UserInfoDTO loadUserInfo(Long userId) {
        User user = userRepository.findById(userId).orElse(null);
        if (user == null) {
            throw new BusinessException("用户不存在");
        }
        
        List roles = roleRepository.findByUserId(userId);
        List menus = menuRepository.findByRoles(roles.stream().map(Role::getId).collect(Collectors.toList()));
        List apiPermissions = apiPermissionRepository.findByRoles(roles.stream().map(Role::getId).collect(Collectors.toList()));
        
        return UserInfoDTO.builder()
            .userId(user.getId())
            .username(user.getUsername())
            .nickname(user.getNickname())
            .avatar(user.getAvatar())
            .roles(roles.stream().map(Role::getCode).collect(Collectors.toList()))
            .menus(buildMenuTree(menus))
            .apiPermissions(apiPermissions.stream()
                .map(ApiPermission::getPermission)
                .collect(Collectors.toSet()))
            .build();
    }
    
    private List buildMenuTree(List menus) {
        Map> menuMap = menus.stream()
            .collect(Collectors.groupingBy(Menu::getParentId));
        
        return buildMenuTree(menuMap, 0L);
    }
    
    private List buildMenuTree(Map> menuMap, Long parentId) {
        List children = menuMap.getOrDefault(parentId, Collections.emptyList());
        return children.stream()
            .map(menu -> MenuDTO.builder()
                .id(menu.getId())
                .name(menu.getName())
                .path(menu.getPath())
                .icon(menu.getIcon())
                .children(buildMenuTree(menuMap, menu.getId()))
                .build())
            .collect(Collectors.toList());
    }
}

@Data
@Builder
public class UserInfoDTO {
    private Long userId;
    private String username;
    private String nickname;
    private String avatar;
    private List roles;
    private List menus;
    private Set apiPermissions;
}

@Data
@Builder
public class MenuDTO {
    private Long id;
    private String name;
    private String path;
    private String icon;
    private List children;
}
                    
                

                
                    
                        
                            
                        
                        多账户支持
                    
                    
                        支持一个手机号对应多个账户，用户可以选择登录哪个账户。
                    
                    
                        @Service
public class MultiAccountService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private UserAccountRepository userAccountRepository;
    
    public List getAccountsByPhone(String phone) {
        List accounts = userAccountRepository.findByPhone(phone);
        
        return accounts.stream()
            .map(account -> UserAccountDTO.builder()
                .accountId(account.getId())
                .userId(account.getUserId())
                .username(account.getUsername())
                .nickname(account.getNickname())
                .avatar(account.getAvatar())
                .accountType(account.getAccountType())
                .lastLoginTime(account.getLastLoginTime())
                .build())
            .collect(Collectors.toList());
    }
    
    public LoginResponse selectAccount(String phone, Long accountId) {
        UserAccount account = userAccountRepository.findById(accountId).orElse(null);
        if (account == null) {
            throw new BusinessException("账户不存在");
        }
        
        if (!account.getPhone().equals(phone)) {
            throw new BusinessException("账户与手机号不匹配");
        }
        
        User user = userRepository.findById(account.getUserId()).orElse(null);
        if (user == null) {
            throw new BusinessException("用户不存在");
        }
        
        return LoginResponse.builder()
            .userId(user.getId())
            .accountId(accountId)
            .username(account.getUsername())
            .nickname(account.getNickname())
            .build();
    }
    
    public void bindAccount(String phone, Long userId, String accountType) {
        UserAccount account = UserAccount.builder()
            .phone(phone)
            .userId(userId)
            .username(generateUsername())
            .accountType(accountType)
            .createTime(new Date())
            .build();
        
        userAccountRepository.save(account);
    }
    
    private String generateUsername() {
        return "user_" + System.currentTimeMillis();
    }
}

@Entity
@Table(name = "user_account")
public class UserAccount {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String phone;
    
    private Long userId;
    
    private String username;
    
    private String nickname;
    
    private String avatar;
    
    private String accountType;
    
    private Date lastLoginTime;
    
    private Date createTime;
}

@Data
@Builder
public class UserAccountDTO {
    private Long accountId;
    private Long userId;
    private String username;
    private String nickname;
    private String avatar;
    private String accountType;
    private Date lastLoginTime;
}
                    
                

                
                    
                        
                            
                        
                        多种登录方式
                    
                    
                        支持手机号+验证码、账户+密码、邮箱+密码、第三方登录（微信等）。
                    
                    
                        @Service
public class LoginService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private UserAccountRepository userAccountRepository;
    
    @Autowired
    private SmsService smsService;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Autowired
    private JwtService jwtService;
    
    @Autowired
    private ThirdPartyLoginService thirdPartyLoginService;
    
    public LoginResponse loginByPhoneCode(PhoneCodeLoginRequest request) {
        boolean verified = smsService.verifyCode(request.getPhone(), request.getCode(), "login");
        
        if (!verified) {
            throw new BusinessException("验证码错误");
        }
        
        List accounts = userAccountRepository.findByPhone(request.getPhone());
        
        if (accounts.isEmpty()) {
            throw new BusinessException("手机号未注册");
        }
        
        if (accounts.size() > 1) {
            return LoginResponse.builder()
                .needSelectAccount(true)
                .accounts(accounts.stream()
                    .map(this::convertToDTO)
                    .collect(Collectors.toList()))
                .build();
        }
        
        UserAccount account = accounts.get(0);
        return doLogin(account.getUserId(), account.getId(), request.getDeviceType());
    }
    
    public LoginResponse loginByPassword(PasswordLoginRequest request) {
        User user = userRepository.findByUsername(request.getUsername());
        
        if (user == null) {
            throw new BusinessException("用户名或密码错误");
        }
        
        if (!passwordEncoder.matches(request.getPassword(), user.getPassword())) {
            throw new BusinessException("用户名或密码错误");
        }
        
        UserAccount account = userAccountRepository.findByUserId(user.getId()).stream()
            .findFirst()
            .orElse(null);
        
        if (account == null) {
            throw new BusinessException("账户不存在");
        }
        
        return doLogin(user.getId(), account.getId(), request.getDeviceType());
    }
    
    public LoginResponse loginByEmail(EmailLoginRequest request) {
        User user = userRepository.findByEmail(request.getEmail());
        
        if (user == null) {
            throw new BusinessException("邮箱或密码错误");
        }
        
        if (!passwordEncoder.matches(request.getPassword(), user.getPassword())) {
            throw new BusinessException("邮箱或密码错误");
        }
        
        UserAccount account = userAccountRepository.findByUserId(user.getId()).stream()
            .findFirst()
            .orElse(null);
        
        if (account == null) {
            throw new BusinessException("账户不存在");
        }
        
        return doLogin(user.getId(), account.getId(), request.getDeviceType());
    }
    
    public LoginResponse loginByThirdParty(ThirdPartyLoginRequest request) {
        ThirdPartyUser thirdPartyUser = thirdPartyLoginService.getUserInfo(
            request.getProvider(),
            request.getCode()
        );
        
        User user = userRepository.findByThirdPartyId(
            request.getProvider(),
            thirdPartyUser.getOpenId()
        );
        
        if (user == null) {
            user = createUserFromThirdParty(thirdPartyUser, request.getProvider());
        }
        
        UserAccount account = userAccountRepository.findByUserId(user.getId()).stream()
            .findFirst()
            .orElse(null);
        
        if (account == null) {
            account = createAccountForUser(user, request.getProvider());
        }
        
        return doLogin(user.getId(), account.getId(), request.getDeviceType());
    }
    
    private LoginResponse doLogin(Long userId, Long accountId, String deviceType) {
        User user = userRepository.findById(userId).orElse(null);
        UserAccount account = userAccountRepository.findById(accountId).orElse(null);
        
        String accessToken = jwtService.generateAccessToken(user);
        String refreshToken = jwtService.generateRefreshToken(user);
        
        account.setLastLoginTime(new Date());
        userAccountRepository.save(account);
        
        return LoginResponse.builder()
            .accessToken(accessToken)
            .refreshToken(refreshToken)
            .userId(userId)
            .accountId(accountId)
            .username(account.getUsername())
            .nickname(account.getNickname())
            .avatar(account.getAvatar())
            .build();
    }
    
    private UserAccountDTO convertToDTO(UserAccount account) {
        return UserAccountDTO.builder()
            .accountId(account.getId())
            .userId(account.getUserId())
            .username(account.getUsername())
            .nickname(account.getNickname())
            .avatar(account.getAvatar())
            .accountType(account.getAccountType())
            .build();
    }
}

public enum LoginType {
    PHONE_CODE,
    USERNAME_PASSWORD,
    EMAIL_PASSWORD,
    WECHAT,
    QQ,
    WEIBO
}

@Data
@Builder
public class LoginResponse {
    private boolean needSelectAccount;
    private List accounts;
    private String accessToken;
    private String refreshToken;
    private Long userId;
    private Long accountId;
    private String username;
    private String nickname;
    private String avatar;
}
                    
                

                
                    
                        
                            
                        
                        账户锁定与注销
                    
                    
                        支持账户锁定、注销、登录失败多次后锁定。
                    
                    
                        @Service
public class AccountSecurityService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private UserAccountRepository userAccountRepository;
    
    @Autowired
    private LoginFailRecordRepository loginFailRecordRepository;
    
    @Autowired
    private RedisTemplate redisTemplate;
    
    private static final String LOCK_KEY_PREFIX = "account:lock:";
    private static final String FAIL_COUNT_KEY_PREFIX = "login:fail:";
    
    public void lockAccount(Long accountId, String reason, int lockMinutes) {
        UserAccount account = userAccountRepository.findById(accountId).orElse(null);
        if (account == null) {
            throw new BusinessException("账户不存在");
        }
        
        account.setLocked(true);
        account.setLockReason(reason);
        account.setLockTime(new Date());
        account.setUnlockTime(new Date(System.currentTimeMillis() + lockMinutes * 60 * 1000));
        userAccountRepository.save(account);
        
        String lockKey = LOCK_KEY_PREFIX + accountId;
        redisTemplate.opsForValue().set(lockKey, reason, lockMinutes, TimeUnit.MINUTES);
    }
    
    public void unlockAccount(Long accountId) {
        UserAccount account = userAccountRepository.findById(accountId).orElse(null);
        if (account == null) {
            throw new BusinessException("账户不存在");
        }
        
        account.setLocked(false);
        account.setLockReason(null);
        account.setLockTime(null);
        account.setUnlockTime(null);
        userAccountRepository.save(account);
        
        String lockKey = LOCK_KEY_PREFIX + accountId;
        redisTemplate.delete(lockKey);
    }
    
    public boolean isAccountLocked(Long accountId) {
        UserAccount account = userAccountRepository.findById(accountId).orElse(null);
        if (account == null) {
            return false;
        }
        
        if (account.isLocked()) {
            if (account.getUnlockTime() != null && 
                account.getUnlockTime().before(new Date())) {
                unlockAccount(accountId);
                return false;
            }
            return true;
        }
        
        String lockKey = LOCK_KEY_PREFIX + accountId;
        return Boolean.TRUE.equals(redisTemplate.hasKey(lockKey));
    }
    
    public void recordLoginFail(String username, String ip) {
        String failKey = FAIL_COUNT_KEY_PREFIX + username;
        Long failCount = redisTemplate.opsForValue().increment(failKey);
        
        if (failCount == 1) {
            redisTemplate.expire(failKey, 1, TimeUnit.HOURS);
        }
        
        LoginFailRecord record = LoginFailRecord.builder()
            .username(username)
            .ip(ip)
            .failTime(new Date())
            .build();
        loginFailRecordRepository.save(record);
        
        if (failCount >= 5) {
            User user = userRepository.findByUsername(username);
            if (user != null) {
                UserAccount account = userAccountRepository.findByUserId(user.getId()).stream()
                    .findFirst()
                    .orElse(null);
                if (account != null) {
                    lockAccount(account.getId(), "登录失败次数过多", 30);
                }
            }
        }
    }
    
    public void clearLoginFail(String username) {
        String failKey = FAIL_COUNT_KEY_PREFIX + username;
        redisTemplate.delete(failKey);
    }
    
    public void deactivateAccount(Long accountId, String reason) {
        UserAccount account = userAccountRepository.findById(accountId).orElse(null);
        if (account == null) {
            throw new BusinessException("账户不存在");
        }
        
        account.setActive(false);
        account.setDeactivateReason(reason);
        account.setDeactivateTime(new Date());
        userAccountRepository.save(account);
    }
    
    public void reactivateAccount(Long accountId) {
        UserAccount account = userAccountRepository.findById(accountId).orElse(null);
        if (account == null) {
            throw new BusinessException("账户不存在");
        }
        
        account.setActive(true);
        account.setDeactivateReason(null);
        account.setDeactivateTime(null);
        userAccountRepository.save(account);
    }
}

@Entity
@Table(name = "login_fail_record")
public class LoginFailRecord {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    
    private String ip;
    
    private Date failTime;
}
                    
                

                
                    
                        
                            
                        
                        多终端登录配置
                    
                    
                        支持配置账户是否可以多终端登录，只能一种设备登录一次等。
                    
                    
                        @Service
public class MultiDeviceService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private UserAccountRepository userAccountRepository;
    
    @Autowired
    private UserSessionRepository userSessionRepository;
    
    @Autowired
    private RedisTemplate redisTemplate;
    
    @Autowired
    private JwtService jwtService;
    
    private static final String SESSION_KEY_PREFIX = "user:session:";
    private static final String DEVICE_KEY_PREFIX = "user:device:";
    
    public DeviceConfigDTO getDeviceConfig(Long accountId) {
        UserAccount account = userAccountRepository.findById(accountId).orElse(null);
        if (account == null) {
            throw new BusinessException("账户不存在");
        }
        
        return DeviceConfigDTO.builder()
            .accountId(accountId)
            .allowMultiDevice(account.isAllowMultiDevice())
            .maxDeviceCount(account.getMaxDeviceCount())
            .singleDeviceType(account.isSingleDeviceType())
            .deviceTypes(account.getAllowedDeviceTypes())
            .build();
    }
    
    public void updateDeviceConfig(Long accountId, DeviceConfigUpdateRequest request) {
        UserAccount account = userAccountRepository.findById(accountId).orElse(null);
        if (account == null) {
            throw new BusinessException("账户不存在");
        }
        
        account.setAllowMultiDevice(request.isAllowMultiDevice());
        account.setMaxDeviceCount(request.getMaxDeviceCount());
        account.setSingleDeviceType(request.isSingleDeviceType());
        account.setAllowedDeviceTypes(request.getDeviceTypes());
        userAccountRepository.save(account);
    }
    
    public boolean checkDeviceLogin(Long userId, Long accountId, String deviceId, String deviceType) {
        UserAccount account = userAccountRepository.findById(accountId).orElse(null);
        if (account == null) {
            throw new BusinessException("账户不存在");
        }
        
        if (!account.isAllowMultiDevice()) {
            String sessionKey = SESSION_KEY_PREFIX + accountId;
            String existingDevice = redisTemplate.opsForValue().get(sessionKey);
            
            if (existingDevice != null && !existingDevice.equals(deviceId)) {
                kickOutDevice(userId, accountId, existingDevice);
            }
            
            redisTemplate.opsForValue().set(sessionKey, deviceId, 7, TimeUnit.DAYS);
            return true;
        }
        
        if (account.isSingleDeviceType()) {
            String deviceKey = DEVICE_KEY_PREFIX + accountId + ":" + deviceType;
            String existingDevice = redisTemplate.opsForValue().get(deviceKey);
            
            if (existingDevice != null && !existingDevice.equals(deviceId)) {
                kickOutDevice(userId, accountId, existingDevice);
            }
            
            redisTemplate.opsForValue().set(deviceKey, deviceId, 7, TimeUnit.DAYS);
        }
        
        if (account.getMaxDeviceCount() != null && account.getMaxDeviceCount() > 0) {
            String sessionPattern = SESSION_KEY_PREFIX + accountId + ":*";
            Set keys = redisTemplate.keys(sessionPattern);
            
            if (keys != null && keys.size() >= account.getMaxDeviceCount()) {
                throw new BusinessException("已达到最大设备登录数");
            }
        }
        
        String sessionKey = SESSION_KEY_PREFIX + accountId + ":" + deviceId;
        redisTemplate.opsForValue().set(sessionKey, deviceType, 7, TimeUnit.DAYS);
        
        saveUserSession(userId, accountId, deviceId, deviceType);
        
        return true;
    }
    
    public void logoutDevice(Long userId, Long accountId, String deviceId) {
        String sessionKey = SESSION_KEY_PREFIX + accountId + ":" + deviceId;
        redisTemplate.delete(sessionKey);
        
        userSessionRepository.findByAccountIdAndDeviceId(accountId, deviceId)
            .ifPresent(session -> {
                session.setActive(false);
                session.setLogoutTime(new Date());
                userSessionRepository.save(session);
            });
    }
    
    public void kickOutDevice(Long userId, Long accountId, String deviceId) {
        logoutDevice(userId, accountId, deviceId);
        
        String kickMessage = "您的账号已在其他设备登录，您已被踢下线";
        notifyUser(userId, deviceId, kickMessage);
    }
    
    public List getActiveSessions(Long accountId) {
        String sessionPattern = SESSION_KEY_PREFIX + accountId + ":*";
        Set keys = redisTemplate.keys(sessionPattern);
        
        if (keys == null || keys.isEmpty()) {
            return Collections.emptyList();
        }
        
        return userSessionRepository.findByAccountIdAndActive(accountId, true).stream()
            .map(this::convertToDTO)
            .collect(Collectors.toList());
    }
    
    private void saveUserSession(Long userId, Long accountId, String deviceId, String deviceType) {
        UserSession session = UserSession.builder()
            .userId(userId)
            .accountId(accountId)
            .deviceId(deviceId)
            .deviceType(deviceType)
            .loginTime(new Date())
            .active(true)
            .build();
        
        userSessionRepository.save(session);
    }
    
    private void notifyUser(Long userId, String deviceId, String message) {
    }
    
    private UserSessionDTO convertToDTO(UserSession session) {
        return UserSessionDTO.builder()
            .sessionId(session.getId())
            .deviceId(session.getDeviceId())
            .deviceType(session.getDeviceType())
            .loginTime(session.getLoginTime())
            .active(session.isActive())
            .build();
    }
}

@Data
@Builder
public class DeviceConfigDTO {
    private Long accountId;
    private boolean allowMultiDevice;
    private Integer maxDeviceCount;
    private boolean singleDeviceType;
    private List deviceTypes;
}

@Data
@Builder
public class UserSessionDTO {
    private Long sessionId;
    private String deviceId;
    private String deviceType;
    private Date loginTime;
    private boolean active;
}

public enum DeviceType {
    WEB,
    IOS,
    ANDROID,
    WINDOWS,
    MAC
}

@Entity
@Table(name = "user_session")
public class UserSession {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private Long userId;
    
    private Long accountId;
    
    private String deviceId;
    
    private String deviceType;
    
    private Date loginTime;
    
    private Date logoutTime;
    
    private boolean active;
}
                    
                
            

            
                核心优势
            
            
            
                
                    
                    灵活认证
                    多种登录方式任选
                
                
                    
                    多账户支持
                    一号多账，灵活切换
                
                
                    
                    安全可靠
                    多重安全防护机制
                
                
                    
                    设备管理
                    灵活的多终端配置
                
                
                    
                    权限精细
                    菜单权限与接口权限
                
                
                    
                    异常监控
                    登录失败自动锁定
