\n\n    
    
    聊天接待 - 客服系统
    
    
    
        .feature-card {
            transition: all 0.3s ease;
        }
        .feature-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 25px rgba(0,0,0,0.15);
        }
        .code-block {
            background: #1e293b;
            color: #e2e8f0;
            font-family: 'Consolas', 'Monaco', monospace;
            font-size: 13px;
            line-height: 1.6;
            overflow-x: auto;
        }
    \n\n    
        
            
                聊天接待
            
            Chat Reception - 实时客户沟通与接待
            
            
                
                    
                    核心功能：支持微信登录、WebSocket实时通信、敏感词过滤、RAG智能回答、聊天记录保存、消息转发。
                
            

            
                功能概览
            
            
            
                
                    
                    微信登录
                    OAuth2.0微信授权
                
                
                    
                    实时通信
                    WebSocket双向通信
                
                
                    
                    智能回答
                    RAG知识库检索
                
                
                    
                    记录保存
                    聊天记录持久化
                
                
                    
                    敏感过滤
                    敏感词自动过滤
                
                
                    
                    消息转发
                    客服间消息转接
                
                
                    
                    文件发送
                    图片文件支持
                
                
                    
                    消息提醒
                    实时消息通知
                
            

            
                通信流程
            

            
                
                    
                    
                        
                            1
                            
                                微信登录
                                用户通过微信OAuth2.0授权登录，获取用户信息
                            
                        
                        
                            2
                            
                                建立连接
                                客户端与服务器建立WebSocket连接，进行身份认证
                            
                        
                        
                            3
                            
                                智能分流
                                先尝试RAG智能回答，无法回答时转接人工客服
                            
                        
                        
                            4
                            
                                实时通信
                                用户与客服进行实时消息收发，消息经过敏感词过滤
                            
                        
                        
                            5
                            
                                记录保存
                                所有聊天消息保存到Redis，支持会话记录查询
                            
                        
                    
                
            

            
                核心代码示例
            

            
                
                    
                        微信登录服务
                        WeChatAuthService.java
                    
                    
/**
 * 微信登录服务 - OAuth2.0认证
 */
@Service
@Slf4j
public class WeChatAuthService {

    @Autowired
    private RedisTemplate&lt;String, Object&gt; redisTemplate;

    public WeChatUser login(String code) throws Exception {
        // 1. 使用code换取access_token
        String accessTokenUrl = String.format(
            "https://api.weixin.qq.com/sns/oauth2/access_token?appid=%s&secret=%s&code=%s&grant_type=authorization_code",
            wechatConfig.getAppId(),
            wechatConfig.getAppSecret(),
            code
        );
        
        String response = restTemplate.getForObject(accessTokenUrl, String.class);
        JSONObject json = JSON.parseObject(response);
        
        String accessToken = json.getString("access_token");
        String openId = json.getString("openid");
        
        // 2. 获取用户信息
        String userInfoUrl = String.format(
            "https://api.weixin.qq.com/sns/userinfo?access_token=%s&openid=%s",
            accessToken,
            openId
        );
        
        String userInfoResponse = restTemplate.getForObject(userInfoUrl, String.class);
        WeChatUser user = JSON.parseObject(userInfoResponse, WeChatUser.class);
        
        // 3. 保存用户到Redis
        String userKey = "wechat_user:" + openId;
        redisTemplate.opsForValue().set(userKey, user, 7, TimeUnit.DAYS);
        
        // 4. 生成会话token
        String sessionToken = UUID.randomUUID().toString();
        redisTemplate.opsForValue().set("session:" + sessionToken, openId, 24, TimeUnit.HOURS);
        
        user.setSessionToken(sessionToken);
        return user;
    }
}
                    
                

                
                    
                        聊天客户端
                        ChatClient.vue
                    
                    
// 聊天客户端 - WebSocket实时通信
&lt;template&gt;
  &lt;div class="chat-client"&gt;
    &lt;div class="chat-messages"&gt;
      &lt;div 
        v-for="msg in messages"
        :key="msg.id"
        :class=['message', msg.type]
      &gt;
        &lt;div class="message-content"&gt;
          {{ msg.content }}
        &lt;/div&gt;
        &lt;div class="message-time"&gt;
          {{ formatTime(msg.timestamp) }}
        &lt;/div&gt;
      &lt;/div&gt;
    &lt;/div&gt;
    
    &lt;div class="chat-input"&gt;
      &lt;input
        v-model="inputMessage"
        @keyup.enter="sendMessage"
        placeholder="请输入消息..."
        class="flex-1"
      &gt;
      &lt;button @click="sendMessage"&gt;
        &lt;i class="fas fa-paper-plane"&gt;&lt;/i&gt;
      &lt;/button&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/template&gt;

&lt;script&gt;
import { ref, onMounted, onUnmounted } from 'vue';

export default {
  name: 'ChatClient',
  props: {
    sessionToken: String
  },
  setup(props) {
    const messages = ref([]);
    const inputMessage = ref('');
    let ws = null;

    const connect = () =&gt; {
      const wsUrl = `ws://localhost:8080/ws?token=${props.sessionToken}`;
      ws = new WebSocket(wsUrl);
      
      ws.onopen = () =&gt; {
        console.log('WebSocket连接成功');
      };
      
      ws.onmessage = (event) =&gt; {
        const msg = JSON.parse(event.data);
        messages.value.push(msg);
      };
      
      ws.onclose = () =&gt; {
        console.log('WebSocket连接关闭');
        setTimeout(connect, 3000);
      };
    };

    const sendMessage = () =&gt; {
      if (!inputMessage.value.trim()) return;
      
      const msg = {
        type: 'USER',
        content: inputMessage.value,
        timestamp: new Date().toISOString()
      };
      
      ws.send(JSON.stringify(msg));
      messages.value.push(msg);
      inputMessage.value = '';
    };

    const formatTime = (timestamp) =&gt; {
      return new Date(timestamp).toLocaleTimeString();
    };

    onMounted(() =&gt; {
      connect();
    });

    onUnmounted(() =&gt; {
      if (ws) {
        ws.close();
      }
    });

    return {
      messages,
      inputMessage,
      sendMessage,
      formatTime
    };
  }
};
&lt;/script&gt;
                    
                

                
                    
                        敏感词过滤服务
                        SensitiveWordService.java
                    
                    
/**
 * 敏感词过滤服务 - 支持内置和第三方
 */
@Service
@Slf4j
public class SensitiveWordService {

    @Autowired
    private RedisTemplate&lt;String, Object&gt; redisTemplate;
    
    private TrieNode root;

    @PostConstruct
    public void init() {
        // 从Redis加载敏感词库
        Set&lt;String&gt; words = (Set&lt;String&gt;) redisTemplate.opsForSet().members("sensitive_words");
        // 构建Trie树
        buildTrie(words);
    }

    public String filter(String content) {
        // 1. 内置敏感词过滤
        String filtered = filterByTrie(content);
        
        // 2. 第三方敏感词检测（可选）
        if (thirdPartyConfig.isEnabled()) {
            filtered = filterByThirdParty(filtered);
        }
        
        return filtered;
    }

    private String filterByTrie(String content) {
        // 使用Trie树进行敏感词匹配
        StringBuilder result = new StringBuilder();
        int n = content.length();
        int p = 0;
        
        while (p &lt; n) {
            TrieNode node = root;
            int q = p;
            boolean found = false;
            
            while (q &lt; n &amp;&amp; node.children.containsKey(content.charAt(q))) {
                node = node.children.get(content.charAt(q));
                if (node.isEnd) {
                    found = true;
                    // 替换为***
                    result.append("***");
                    p = q + 1;
                    break;
                }
                q++;
            }
            
            if (!found) {
                result.append(content.charAt(p));
                p++;
            }
        }
        
        return result.toString();
    }

    private String filterByThirdParty(String content) {
        // 调用第三方敏感词检测API
        // 实际项目中对接阿里云、腾讯云等敏感词检测服务
        return content;
    }
}
                    
                
            

            
                核心优势
            
            
            
                
                    
                    微信登录
                    OAuth2.0安全认证
                
                
                    
                    实时通信
                    WebSocket低延迟
                
                
                    
                    智能分流
                    先智能后人工
                
                
                    
                    敏感过滤
                    内置+第三方双重
                
                
                    
                    记录持久化
                    Redis全量存储
                
                
                    
                    断线重连
                    自动重连机制
