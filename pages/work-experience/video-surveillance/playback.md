\n\n    
    
    回放功能 - 视频监控项目
    
    
    
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
        
            
                回放功能
            
            Playback - 历史视频检索与播放
            
            
                
                    
                    核心功能：支持按时间轴检索历史录像，倍速播放、快进快退、截图、时间轴拖拽定位、告警录像标记等功能。
                
            

            
                功能概览
            
            
            
                
                    
                    时间轴检索
                    按日期时间快速定位
                
                
                    
                    倍速播放
                    0.5x/1x/2x/4x/8x
                
                
                    
                    快进快退
                    拖拽时间轴定位
                
                
                    
                    告警标记
                    告警录像高亮显示
                
            

            
                录像存储架构
            

            
                
                    
                        
                            摄像头 (RTSP)
                        
                        
                        
                            FFmpeg 录制
                        
                        
                        
                            MP4 文件
                        
                    
                    
                        
                    
                    
                        
                            
                            MySQL
                            录像元数据（时间、设备、路径）
                        
                        
                            
                            NAS / OSS
                            MP4 视频文件存储
                        
                    
                
            

            
                核心代码示例
            

            
                
                    
                        录像检索服务
                        RecordingSearchService.java
                    
                    
/**
 * 录像检索服务 - 按时间轴查询历史录像
 */
@Service
@Slf4j
public class RecordingSearchService {

    @Autowired
    private RecordingRepository recordingRepository;

    /**
     * 按时间范围检索录像
     */
    public Page&lt;RecordingVO&gt; searchRecordings(RecordingSearchRequest request) {
        LocalDateTime startTime = request.getStartTime();
        LocalDateTime endTime = request.getEndTime();
        Long cameraId = request.getCameraId();

        return recordingRepository.findByCameraIdAndStartTimeBetween(
            cameraId, startTime, endTime,
            PageRequest.of(request.getPage(), request.getSize())
        ).map(this::toVO);
    }

    /**
     * 获取录像播放地址
     */
    public String getPlaybackUrl(Long recordingId) {
        Recording recording = recordingRepository.findById(recordingId)
            .orElseThrow(() -&gt; new NotFoundException("录像不存在"));

        // 检查是本地存储还是 OSS
        if (recording.getStorageType() == StorageType.OSS) {
            // 生成 OSS 预签名 URL
            return ossService.generatePresignedUrl(recording.getFilePath(), 3600);
        } else {
            // 本地文件通过 Nginx 或应用服务提供
            return "/api/recordings/play/" + recordingId;
        }
    }

    /**
     * 获取时间轴上的录像片段
     */
    public List&lt;TimelineSegment&gt; getTimelineSegments(
            Long cameraId, LocalDateTime day) {
        LocalDateTime startOfDay = day.toLocalDate().atStartOfDay();
        LocalDateTime endOfDay = startOfDay.plusDays(1);

        List&lt;Recording&gt; recordings = recordingRepository
            .findByCameraIdAndStartTimeBetweenOrderByStartTimeAsc(
                cameraId, startOfDay, endOfDay
            );

        return recordings.stream().map(r -&gt; TimelineSegment.builder()
            .recordingId(r.getId())
            .startTime(r.getStartTime())
            .endTime(r.getEndTime())
            .hasAlarm(r.getHasAlarm())
            .build()
        ).collect(Collectors.toList());
    }
}
                    
                

                
                    
                        回放播放器
                        PlaybackPlayer.vue
                    
                    
// 回放播放器组件 - 支持倍速、快进、截图
&lt;template&gt;
  &lt;div class="playback-player"&gt;
    &lt;video 
      ref="videoRef"
      :src="playbackUrl"
      controls
      class="w-full"
      @timeupdate="onTimeUpdate"
      @loadedmetadata="onLoadedMetadata"
    &gt;&lt;/video&gt;

    &lt;div class="controls-bar"&gt;
      &lt;div class="speed-control"&gt;
        &lt;button
          v-for="speed in playbackSpeeds"
          :key="speed"
          :class=['speed-btn', { active: currentSpeed === speed }]
          @click="setPlaybackSpeed(speed)"
        &gt;
          {{ speed }}x
        &lt;/button&gt;
      &lt;/div&gt;

      &lt;div class="action-buttons"&gt;
        &lt;button @click="skip(-30)"&gt;
          &lt;i class="fas fa-backward"&gt;&lt;/i&gt; -30s
        &lt;/button&gt;
        &lt;button @click="skip(30)"&gt;
          +30s &lt;i class="fas fa-forward"&gt;&lt;/i&gt;
        &lt;/button&gt;
        &lt;button @click="takeScreenshot"&gt;
          &lt;i class="fas fa-camera"&gt;&lt;/i&gt; 截图
        &lt;/button&gt;
      &lt;/div&gt;
    &lt;/div&gt;

    &lt;div class="timeline"&gt;
      &lt;div 
        v-for="segment in timelineSegments"
        :key="segment.recordingId"
        :class=['segment', { alarm: segment.hasAlarm }]
        @click="seekToSegment(segment)"
      &gt;&lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/template&gt;

&lt;script&gt;
import { ref } from 'vue';

export default {
  name: 'PlaybackPlayer',
  props: {
    playbackUrl: String,
    timelineSegments: Array
  },
  setup(props) {
    const videoRef = ref(null);
    const currentSpeed = ref(1);
    const playbackSpeeds = [0.5, 1, 2, 4, 8];

    const setPlaybackSpeed = (speed) =&gt; {
      currentSpeed.value = speed;
      if (videoRef.value) {
        videoRef.value.playbackRate = speed;
      }
    };

    const skip = (seconds) =&gt; {
      if (videoRef.value) {
        videoRef.value.currentTime += seconds;
      }
    };

    const takeScreenshot = () =&gt; {
      const canvas = document.createElement('canvas');
      canvas.width = videoRef.value.videoWidth;
      canvas.height = videoRef.value.videoHeight;
      canvas.getContext('2d').drawImage(videoRef.value, 0, 0);
      
      const link = document.createElement('a');
      link.download = `screenshot-${Date.now()}.png`;
      link.href = canvas.toDataURL('image/png');
      link.click();
    };

    const seekToSegment = (segment) =&gt; {
      // 跳转到指定录像片段
    };

    return {
      videoRef,
      currentSpeed,
      playbackSpeeds,
      setPlaybackSpeed,
      skip,
      takeScreenshot,
      seekToSegment
    };
  }
};
&lt;/script&gt;
                    
                
            

            
                核心优势
            
            
            
                
                    
                    快速检索
                    时间轴可视化快速定位
                
                
                    
                    多倍速
                    0.5x - 8x 灵活切换
                
                
                    
                    快捷截图
                    Canvas 一键截图保存
                
                
                    
                    告警标记
                    告警录像高亮显示
                
                
                    
                    预签名URL
                    OSS 安全下载
                
                
                    
                    灵活存储
                    本地 NAS / OSS 支持
