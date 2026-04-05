\n\n    
    
    直播功能 - 视频监控项目
    
    
    
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
        
            
                直播功能
            
            Live Streaming - 实时视频流传输与播放
            
            
                
                    
                    核心功能：支持多路摄像头同时预览，多协议（WebRTC/HLS/FLV）自适应播放，PC 和手机同时访问，分屏显示、云台控制、截图等功能。
                
            

            
                功能概览
            
            
            
                
                    
                    多路预览
                    1/4/9/16 分屏显示
                
                
                    
                    云台控制
                    PTZ 上下左右缩放
                
                
                    
                    实时截图
                    Canvas 截图保存
                
                
                    
                    多端播放
                    PC/手机自适应
                
            

            
                直播流架构
            

            
                
                    
                        
                            摄像头 (RTSP)
                        
                        
                        
                            FFmpeg 转码
                        
                        
                        
                            ZLMediaKit
                        
                    
                    
                        
                    
                    
                        
                            
                            WebRTC
                            &lt; 500ms 延迟
                            PC 端优先
                        
                        
                            
                            FLV.js
                            1-2s 延迟
                            PC 端降级
                        
                        
                            
                            HLS
                            3-5s 延迟
                            手机端优先
                        
                    
                
            

            
                核心代码示例
            

            
                
                    
                        分屏播放器
                        MultiScreenPlayer.vue
                    
                    
// 分屏播放器组件 - 支持 1/4/9/16 分屏
&lt;template&gt;
  &lt;div class="multi-screen-player"&gt;
    &lt;div :class="screenClass"&gt;
      &lt;div 
        v-for="(stream, index) in displayStreams" 
        :key="index"
        class="screen-item"
      &gt;
        &lt;VideoPlayer 
          :streamInfo="stream"
          @screenshot=handleScreenshot"
        /&gt;
        &lt;div class="stream-info"&gt;
          &lt;span&gt;{{ stream.name }}&lt;/span&gt;
          &lt;button @click="toggleFullscreen(index)"&gt;
            &lt;i class="fas fa-expand"&gt;&lt;/i&gt;
          &lt;/button&gt;
        &lt;/div&gt;
      &lt;/div&gt;
    &lt;/div&gt;
    
    &lt;div class="screen-controls"&gt;
      &lt;button 
        v-for="mode in screenModes"
        :key="mode.count"
        :class=['btn', { active: currentMode === mode.count }]
        @click="currentMode = mode.count"
      &gt;
        {{ mode.label }}
      &lt;/button&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/template&gt;

&lt;script&gt;
import { computed, ref } from 'vue';
import VideoPlayer from './VideoPlayer.vue';

export default {
  name: 'MultiScreenPlayer',
  components: { VideoPlayer },
  props: {
    streams: Array
  },
  setup(props) {
    const currentMode = ref(4);
    const screenModes = [
      { count: 1, label: '1屏' },
      { count: 4, label: '4屏' },
      { count: 9, label: '9屏' },
      { count: 16, label: '16屏' }
    ];

    const screenClass = computed(() =&gt; {
      return `grid grid-cols-${Math.sqrt(currentMode.value)} gap-2`;
    });

    const displayStreams = computed(() =&gt; {
      return props.streams.slice(0, currentMode.value);
    });

    const handleScreenshot = (stream) =&gt; {
      // 保存截图到本地
    };

    const toggleFullscreen = (index) =&gt; {
      currentMode.value = 1;
    };

    return {
      currentMode,
      screenModes,
      screenClass,
      displayStreams,
      handleScreenshot,
      toggleFullscreen
    };
  }
};
&lt;/script&gt;
                    
                

                
                    
                        云台控制服务
                        PTZControlService.java
                    
                    
/**
 * 云台控制服务 - 通过 ONVIF 协议控制摄像头 PTZ
 */
@Service
@Slf4j
public class PTZControlService {

    public void controlPTZ(PTZControlRequest request) {
        String cameraIp = request.getCameraIp();
        PTZCommand command = request.getCommand();
        float speed = request.getSpeed();

        switch (command) {
            case UP:
                moveUp(cameraIp, speed);
                break;
            case DOWN:
                moveDown(cameraIp, speed);
                break;
            case LEFT:
                moveLeft(cameraIp, speed);
                break;
            case RIGHT:
                moveRight(cameraIp, speed);
                break;
            case ZOOM_IN:
                zoomIn(cameraIp, speed);
                break;
            case ZOOM_OUT:
                zoomOut(cameraIp, speed);
                break;
            case STOP:
                stop(cameraIp);
                break;
        }
    }

    private void moveUp(String ip, float speed) {
        // 使用 ONVIF 协议发送向上移动指令
        sendOnvifCommand(ip, "ContinuousMove", 
            "PanTilt", "x=0,y=" + speed);
    }

    private void zoomIn(String ip, float speed) {
        // 使用 ONVIF 协议发送缩放指令
        sendOnvifCommand(ip, "ContinuousMove", 
            "Zoom", "x=" + speed);
    }
}
                    
                
            

            
                性能优化
            
            
            
                
                    FFmpeg 转码优化
                
                
                
                    
                        FFmpeg 推流命令
                        StreamService.java
                    
                    
// 构建高性能 FFmpeg 推流命令
private String buildFFmpegCommand(String rtspUrl, String rtmpUrl, String resolution, int bitrate) {
    return String.format(
        "ffmpeg -re -rtsp_transport tcp -i %s " +
        "-c:v libx264 -preset ultrafast -tune zerolatency " +
        "-s %s -b:v %dk -g 25 -sc_threshold 0 " +
        "-c:a aac -b:a 64k -ar 44100 " +
        "-f flv %s",
        rtspUrl, resolution, bitrate, rtmpUrl
    );
}
                    
                
                
                
                    ZLMediaKit 配置优化
                
                
                
                    
                        连接复用
                        
                            
                                
                            
                            启用连接复用，减少服务器资源占用
                        
                    
                    
                        缓存优化
                        
                            
                                
                            
                            合理设置缓存大小，平衡延迟和稳定性
                        
                    
                    
                        线程池配置
                        
                            
                                
                            
                            根据服务器CPU核心数配置线程池
                        
                    
                
                
                
                    服务器性能指标
                
                
                
                    
                        50-100
                        普通4C8G服务器支持路数
                    
                    
                        1-2Mbps
                        单路视频推荐码率
                    
                    
                        720p
                        推荐视频分辨率
                    
                
            
            
            
                多端访问实现
            
            
            
                
                    
                        设备检测与协议适配
                        StreamPlayer.vue
                    
                    
// 设备检测与协议自适应
const detectDeviceAndSelectProtocol = () =&gt; {
    const userAgent = navigator.userAgent;
    const isMobile = /Android|iPhone|iPad|iPod/i.test(userAgent);
    
    if (!isMobile) {
        // PC 端优先使用 WebRTC
        if (checkWebRTCSupport()) {
            return { protocol: 'webrtc', url: streamInfo.webrtcUrl };
        } else {
            // 降级使用 FLV
            return { protocol: 'flv', url: streamInfo.flvUrl };
        }
    } else {
        // 手机端使用 HLS
        return { protocol: 'hls', url: streamInfo.hlsUrl };
    }
};
                    
                
                
                
                    
                        
                            PC 端
                        
                        
                            WebRTC：延迟 &lt; 500ms
                            FLV.js：延迟 1-2s（降级方案）
                            支持 16 路分屏
                            支持云台控制
                        
                    
                    
                        
                            手机端
                        
                        
                            HLS：延迟 3-5s（广泛兼容）
                            自适应分辨率
                            支持 4 路分屏
                            触摸控制
                        
                    
                
            
            
            
                核心优势
            
            
            
                
                    
                    低延迟
                    WebRTC 延迟 &lt; 500ms
                
                
                    
                    多分屏
                    1/4/9/16 分屏自由切换
                
                
                    
                    PTZ控制
                    ONVIF 标准云台控制
                
                
                    
                    实时截图
                    Canvas 快速截图
                
                
                    
                    多端兼容
                    PC/手机自动适配
                
                
                    
                    协议降级
                    WebRTC → FLV → HLS
                
                
                    
                    高性能
                    普通服务器支持 50-100 路
                
                
                    
                    易扩展
                    支持集群部署
                
                
                    
                    安全可靠
                    RTSP 密码认证
