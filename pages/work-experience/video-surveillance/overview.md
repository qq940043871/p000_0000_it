\n\n    
    
    视频监控项目 - 架构师学习笔记
    
    
    
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
        
            
                视频监控项目
            
            Video Surveillance System - FFmpeg + ZLM4J 技术方案
            
            
                
                    
                    技术方案：采用 FFmpeg + ZLMediaKit (ZLM4J) 流媒体服务器，支持 RTSP/RTMP/HLS/WebRTC 多协议，实现 PC 和手机同时访问，普通服务器支持大并发多路拉流。
                
            

            
                功能概览
            
            
            
                
                    
                    实时直播
                    多路摄像头同时预览
                
                
                    
                    录像回放
                    按时间轴检索回放
                
                
                    
                    视频下载
                    录像文件下载
                
                
                    
                    多端适配
                    PC 和手机同时访问
                
            

            
                系统架构
            
            
            
                
                    
                        
                        摄像头
                        RTSP/ONVIF
                    
                    
                        
                    
                    
                        
                        流媒体服务器
                        ZLMediaKit
                    
                    
                        
                    
                    
                        
                        客户端
                        PC/手机
                    
                
            

            
                核心技术栈
            

            
                
                    
                        后端技术
                    
                    
                        
                            
                                
                            
                            
                                Spring Boot
                                业务服务层
                            
                        
                        
                            
                                
                            
                            
                                FFmpeg
                                视频转码、拉流推流
                            
                        
                        
                            
                                
                            
                            
                                ZLMediaKit (ZLM4J)
                                高性能流媒体服务器
                            
                        
                    
                

                
                    
                        前端技术
                    
                    
                        
                            
                                
                            
                            
                                WebRTC
                                超低延迟直播预览
                            
                        
                        
                            
                                
                            
                            
                                HLS / FLV.js
                                多端兼容播放
                            
                        
                        
                            
                                
                            
                            
                                响应式设计
                                PC 和手机自适应
                            
                        
                    
                
            

            
                数据流设计
            

            
                直播流数据流
                
// 摄像头 → ZLMediaKit → 客户端
1. FFmpeg 从摄像头拉取 RTSP 流
2. FFmpeg 转码为 H.264/H.265 + AAC
3. 推流 到 ZLMediaKit (RTMP 协议)
4. ZLMediaKit 分发多种协议流
   ├─ WebRTC (PC 低延迟)
   ├─ HLS (手机兼容)
   └─ FLV (PC 低延迟)
5. 客户端 按需拉取对应协议流
                
            

            
                核心代码示例
            

            
                
                    
                        ZLM4J 推流服务
                        StreamPushService.java
                    
                    
/**
 * ZLM4J 推流服务 - 使用 FFmpeg 拉取 RTSP 并推送到 ZLMediaKit
 */
@Service
@Slf4j
public class StreamPushService {

    @Autowired
    private ZLMediaKitClient zlMediaKitClient;

    /**
     * 开始推流
     */
    public StreamPushResult startPush(StreamPushRequest request) {
        String app = "live";
        String streamId = UUID.randomUUID().toString();

        // 1. 在 ZLMediaKit 上注册流
        zlMediaKitClient.registerStream(app, streamId);

        // 2. 构建 FFmpeg 推流命令
        String ffmpegCmd = buildFFmpegCommand(
            request.getRtspUrl(),
            request.getRtmpUrl(),
            request.getResolution(),
            request.getBitrate()
        );

        // 3. 启动 FFmpeg 进程
        Process process = Runtime.getRuntime().exec(ffmpegCmd);

        // 4. 监控推流状态
        monitorStream(process, app, streamId);

        return StreamPushResult.builder()
            .app(app)
            .streamId(streamId)
            .webrtcUrl(zlMediaKitClient.getWebRtcUrl(app, streamId))
            .hlsUrl(zlMediaKitClient.getHlsUrl(app, streamId))
            .flvUrl(zlMediaKitClient.getFlvUrl(app, streamId))
            .build();
    }

    /**
     * 构建 FFmpeg 推流命令
     */
    private String buildFFmpegCommand(String rtspUrl, String rtmpUrl, 
            String resolution, int bitrate) {
        return String.format(
            "ffmpeg -re -rtsp_transport tcp -i %s " +
            "-c:v libx264 -preset ultrafast -tune zerolatency " +
            "-s %s -b:v %dk -g 25 -sc_threshold 0 " +
            "-c:a aac -b:a 64k -ar 44100 " +
            "-f flv %s",
            rtspUrl, resolution, bitrate, rtmpUrl
        );
    }
}
                    
                

                
                    
                        前端播放器 (多端兼容)
                        VideoPlayer.vue
                    
                    
// 多端兼容视频播放器 - 自动选择最优播放协议
&lt;template&gt;
  &lt;div&gt;
    &lt;video id="videoPlayer" autoplay muted playsinline 
           class="w-full h-full"&gt;&lt;/video&gt;
  &lt;/div&gt;
&lt;/template&gt;

&lt;script&gt;
import { onMounted, ref } from 'vue';
import Hls from 'hls.js';
import FlvJs from 'flv.js';

export default {
  name: 'VideoPlayer',
  props: {
    streamInfo: Object
  },
  setup(props) {
    const videoElement = ref(null);
    let player = null;

    const isMobile = () =&gt; {
      return /Android|iPhone|iPad|iPod/i.test(navigator.userAgent);
    };

    const initPlayer = () =&gt; {
      const { webrtcUrl, hlsUrl, flvUrl } = props.streamInfo;
      
      // 手机端优先使用 HLS
      if (isMobile()) {
        playHLS(hlsUrl);
      } else {
        // PC 端优先 WebRTC，降级 FLV，再降级 HLS
        if (webrtcUrl) {
          playWebRTC(webrtcUrl);
        } else if (flvUrl &amp;&amp; FlvJs.isSupported()) {
          playFLV(flvUrl);
        } else {
          playHLS(hlsUrl);
        }
      }
    };

    const playHLS = (url) =&gt; {
      if (Hls.isSupported()) {
        player = new Hls();
        player.loadSource(url);
        player.attachMedia(videoElement.value);
      } else if (videoElement.value.canPlayType('application/vnd.apple.mpegurl')) {
        videoElement.value.src = url;
      }
    };

    const playFLV = (url) =&gt; {
      player = FlvJs.createPlayer({
        type: 'flv',
        url: url,
        isLive: true,
        cors: true
      });
      player.attachMediaElement(videoElement.value);
      player.load();
    };

    onMounted(() =&gt; {
      initPlayer();
    });

    return { videoElement };
  }
};
&lt;/script&gt;
                    
                
            

            
                性能优化方案
            

            
                
                    
                        服务器端优化
                    
                    
                        
                            
                            ZLMediaKit 高效转发：单路 RTSP 输入，多路协议输出
                        
                        
                            
                            GOP 缓存：缓存关键帧，新用户秒开
                        
                        
                            
                            自适应码率：根据网络自动切换清晰度
                        
                        
                            
                            连接复用：相同流复用同一转码进程
                        
                    
                
                
                    
                        FFmpeg 转码优化
                    
                    
                        
                            
                            ultrafast 预设：降低编码复杂度，提升转码速度
                        
                        
                            
                            zerolatency tune：优化实时性，降低延迟
                        
                        
                            
                            硬件加速：支持 NVIDIA NVENC / Intel QSV
                        
                        
                            
                            合理 GOP：GOP=25，平衡画质与延迟
                        
                    
                
            

            
                性能指标
            

            
                
                    
                        
                            指标项
                            普通服务器 (4C8G)
                            说明
                        
                    
                    
                        
                            支持摄像头路数
                            50-100 路
                            取决于码率和分辨率
                        
                        
                            并发观看人数
                            1000+ 人
                            ZLMediaKit 高效转发
                        
                        
                            WebRTC 延迟
                            &lt; 500ms
                            PC 端低延迟预览
                        
                        
                            HLS 延迟
                            3-5s
                            手机端兼容播放
                        
                        
                            CPU 使用率
                            60-80%
                            100 路 720P 转码
                        
                        
                            内存占用
                            5-6G
                            包含 GOP 缓存
                        
                    
                
            

            
                核心优势
            
            
            
                
                    
                    开源方案
                    FFmpeg + ZLMediaKit 完全开源
                
                
                    
                    高性能
                    单服务器支持 100+ 路摄像头
                
                
                    
                    多端兼容
                    PC 和手机同时访问
                
                
                    
                    多协议支持
                    RTSP/RTMP/HLS/WebRTC
                
                
                    
                    低延迟
                    WebRTC 延迟 &lt; 500ms
                
                
                    
                    可扩展
                    支持集群和负载均衡
