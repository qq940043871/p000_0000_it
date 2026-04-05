\n\n    
    
    下载功能 - 视频监控项目
    
    
    
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
        .progress-bar {
            width: 100%;
            height: 8px;
            background: #e2e8f0;
            border-radius: 4px;
            overflow: hidden;
            margin: 8px 0;
        }
        .progress-fill {
            height: 100%;
            background: #10b981;
            transition: width 0.3s ease;
        }
        .progress-text {
            font-size: 12px;
            color: #64748b;
        }
        .task-progress {
            margin: 8px 0;
        }
    \n\n    
        
            
                下载功能
            
            Download - 视频文件导出与下载
            
            
                
                    
                    核心功能：支持录像文件下载、格式转换、断点续传、批量下载、下载任务管理等功能，支持 PC 和手机同时访问。
                
            

            
                功能概览
            
            
            
                
                    
                    单文件下载
                    单个录像文件下载
                
                
                    
                    批量下载
                    多文件打包下载
                
                
                    
                    格式转换
                    MP4/AVI/MKV 转换
                
                
                    
                    任务管理
                    下载任务队列管理
                
            

            
                下载流程
            

            
                
                    
                    
                        
                            1
                            
                                用户选择录像
                                按时间范围、摄像头选择需要下载的录像
                            
                        
                        
                            2
                            
                                服务器处理
                                生成下载链接，支持格式转换、加密
                            
                        
                        
                            3
                            
                                客户端下载
                                支持浏览器直接下载、断点续传
                            
                        
                        
                            4
                            
                                任务完成
                                下载完成通知、任务状态更新
                            
                        
                    
                
            

            
                核心代码示例
            

            
                
                    
                        下载服务
                        DownloadService.java
                    
                    
/**
 * 下载服务 - 处理录像文件下载
 */
@Service
@Slf4j
public class DownloadService {

    @Autowired
    private RecordingRepository recordingRepository;
    
    @Autowired
    private AsyncTaskExecutor asyncTaskExecutor;
    
    // 存储下载任务进度
    private final Map&lt;String, DownloadProgress&gt; progressMap = new ConcurrentHashMap&lt;&gt;();

    /**
     * 单文件下载
     */
    public DownloadTask createDownloadTask(DownloadRequest request) {
        Long recordingId = request.getRecordingId();
        String format = request.getFormat();
        String taskId = UUID.randomUUID().toString();

        // 创建下载任务
        DownloadTask task = DownloadTask.builder()
            .taskId(taskId)
            .recordingId(recordingId)
            .format(format)
            .status(DownloadStatus.PENDING)
            .build();

        // 初始化进度信息
        progressMap.put(taskId, new DownloadProgress(0, 0));

        // 异步处理下载
        asyncTaskExecutor.execute(() -&gt; {
            try {
                processDownloadTask(task);
            } catch (Exception e) {
                task.setStatus(DownloadStatus.FAILED);
                task.setErrorMessage(e.getMessage());
                log.error("Download task failed: {}", e.getMessage(), e);
            }
        });

        return task;
    }

    /**
     * 批量下载
     */
    public DownloadTask createBatchDownloadTask(BatchDownloadRequest request) {
        List&lt;Long&gt; recordingIds = request.getRecordingIds();
        String taskId = UUID.randomUUID().toString();

        DownloadTask task = DownloadTask.builder()
            .taskId(taskId)
            .recordingIds(recordingIds)
            .status(DownloadStatus.PENDING)
            .isBatch(true)
            .build();

        // 初始化进度信息
        progressMap.put(taskId, new DownloadProgress(0, 0));

        asyncTaskExecutor.execute(() -&gt; {
            try {
                processBatchDownloadTask(task);
            } catch (Exception e) {
                task.setStatus(DownloadStatus.FAILED);
                task.setErrorMessage(e.getMessage());
                log.error("Batch download task failed: {}", e.getMessage(), e);
            }
        });

        return task;
    }

    /**
     * 获取下载进度
     */
    public DownloadProgress getDownloadProgress(String taskId) {
        return progressMap.getOrDefault(taskId, new DownloadProgress(0, 0));
    }

    private void processDownloadTask(DownloadTask task) throws Exception {
        task.setStatus(DownloadStatus.PROCESSING);
        
        // 获取录像信息
        Recording recording = recordingRepository.findById(task.getRecordingId())
            .orElseThrow(() -&gt; new NotFoundException("录像不存在"));

        // 处理格式转换（如果需要）
        String outputPath = recording.getFilePath();
        if (!"mp4".equals(task.getFormat())) {
            outputPath = convertFormat(recording.getFilePath(), task.getFormat(), task.getTaskId());
        }

        // 生成下载链接
        String downloadUrl = generateDownloadUrl(outputPath, task.getTaskId());
        task.setDownloadUrl(downloadUrl);
        task.setStatus(DownloadStatus.COMPLETED);
        
        // 更新进度为100%
        progressMap.put(task.getTaskId(), new DownloadProgress(100, 100));
    }

    private void processBatchDownloadTask(DownloadTask task) throws Exception {
        task.setStatus(DownloadStatus.PROCESSING);
        
        // 打包多个录像文件
        String zipPath = createZipFile(task.getRecordingIds(), task.getTaskId());
        
        // 生成下载链接
        String downloadUrl = generateDownloadUrl(zipPath, task.getTaskId());
        task.setDownloadUrl(downloadUrl);
        task.setStatus(DownloadStatus.COMPLETED);
        
        // 更新进度为100%
        progressMap.put(task.getTaskId(), new DownloadProgress(100, 100));
    }

    private String convertFormat(String inputPath, String format, String taskId) throws Exception {
        String outputPath = inputPath.replace(".mp4", "." + format);
        
        // 使用 FFmpeg 进行格式转换
        String ffmpegCmd = String.format(
            "ffmpeg -i %s -c copy %s",
            inputPath, outputPath
        );
        
        // 执行命令并监控进度
        Process process = Runtime.getRuntime().exec(ffmpegCmd);
        
        // 简单的进度模拟，实际项目中可以解析FFmpeg输出获取真实进度
        for (int i = 10; i &lt; 100; i += 10) {
            Thread.sleep(500);
            progressMap.put(taskId, new DownloadProgress(i, 100));
        }
        
        process.waitFor();
        return outputPath;
    }

    private String createZipFile(List&lt;Long&gt; recordingIds, String taskId) throws Exception {
        String zipPath = "/tmp/batch_" + taskId + ".zip";
        
        // 模拟打包过程，实际项目中使用ZipOutputStream
        for (int i = 0; i &lt; recordingIds.size(); i++) {
            int progress = (i + 1) * 100 / recordingIds.size();
            progressMap.put(taskId, new DownloadProgress(progress, 100));
            Thread.sleep(300);
        }
        
        return zipPath;
    }

    private String generateDownloadUrl(String filePath, String taskId) {
        // 生成带有签名的下载链接，有效期 24 小时
        return "/api/downloads/" + taskId + "?token=" + generateToken(taskId);
    }
    
    /**
     * 大文件下载支持
     */
    public void downloadLargeFile(String filePath, HttpServletResponse response) throws Exception {
        File file = new File(filePath);
        
        // 设置响应头，支持断点续传
        response.setHeader("Content-Disposition", "attachment; filename=" + URLEncoder.encode(file.getName(), "UTF-8"));
        response.setHeader("Content-Type", "application/octet-stream");
        response.setHeader("Content-Length", String.valueOf(file.length()));
        response.setHeader("Accept-Ranges", "bytes");
        
        // 处理断点续传
        String range = response.getHeader("Range");
        if (range != null) {
            long start = 0, end = file.length() - 1;
            // 解析Range头
            if (range.startsWith("bytes=")) {
                String[] parts = range.substring(6).split("-");
                if (!parts[0].isEmpty()) {
                    start = Long.parseLong(parts[0]);
                }
                if (parts.length &gt; 1 &amp;&amp; !parts[1].isEmpty()) {
                    end = Long.parseLong(parts[1]);
                }
            }
            
            response.setStatus(HttpServletResponse.SC_PARTIAL_CONTENT);
            response.setHeader("Content-Range", "bytes " + start + "-" + end + "/" + file.length());
            
            // 分块读取文件并发送
            try (FileInputStream fis = new FileInputStream(file);
                 BufferedInputStream bis = new BufferedInputStream(fis);
                 OutputStream os = response.getOutputStream()) {
                
                // 跳过已经下载的部分
                bis.skip(start);
                
                // 分块传输
                byte[] buffer = new byte[8192];
                long remaining = end - start + 1;
                int bytesRead;
                
                while (remaining &gt; 0 &amp;&amp; (bytesRead = bis.read(buffer, 0, (int) Math.min(buffer.length, remaining))) != -1) {
                    os.write(buffer, 0, bytesRead);
                    os.flush();
                    remaining -= bytesRead;
                }
            }
        } else {
            // 正常下载
            Files.copy(file.toPath(), response.getOutputStream());
        }
    }
}
                    
                

                
                    
                        下载管理组件
                        DownloadManager.vue
                    
                    
// 下载管理组件 - 支持单文件和批量下载
&lt;template&gt;
  &lt;div class="download-manager"&gt;
    &lt;div class="download-form"&gt;
      &lt;h3 class="font-semibold mb-4"&gt;创建下载任务&lt;/h3&gt;
      
      &lt;div class="form-group"&gt;
        &lt;label class="block mb-2"&gt;选择录像&lt;/label&gt;
        &lt;select 
          v-model="selectedRecordingId"
          class="w-full p-2 border rounded"
        &gt;
          &lt;option value=""&gt;请选择录像&lt;/option&gt;
          &lt;option 
            v-for="recording in recordings"
            :key="recording.id"
            :value="recording.id"
          &gt;
            {{ recording.name }} - {{ formatTime(recording.startTime) }}
          &lt;/option&gt;
        &lt;/select&gt;
      &lt;/div&gt;

      &lt;div class="form-group"&gt;
        &lt;label class="block mb-2"&gt;格式&lt;/label&gt;
        &lt;select 
          v-model="format"
          class="w-full p-2 border rounded"
        &gt;
          &lt;option value="mp4"&gt;MP4 (推荐)&lt;/option&gt;
          &lt;option value="avi"&gt;AVI&lt;/option&gt;
          &lt;option value="mkv"&gt;MKV&lt;/option&gt;
        &lt;/select&gt;
      &lt;/div&gt;

      &lt;button 
        class="btn-primary mt-4"
        @click="createDownloadTask"
      &gt;
        开始下载
      &lt;/button&gt;
    &lt;/div&gt;

    &lt;div class="download-tasks"&gt;
      &lt;h3 class="font-semibold mb-4"&gt;下载任务&lt;/h3&gt;
      
      &lt;div 
        v-for="task in downloadTasks"
        :key="task.taskId"
        :class=['task-item', task.status]
      &gt;
        &lt;div class="task-info"&gt;
          &lt;span class="task-name"&gt;{{ getTaskName(task) }}&lt;/spanspan class="task-status"&gt;{{ getStatusText(task.status) }}&lt;/spandiv&gt;
        
        &lt;div class="task-progress" v-if="task.status === 'PROCESSING'"&gt;
          &lt;div class="progress-bar"&gt;
            &lt;div 
              class="progress-fill"
              :style="{ width: taskProgress[task.taskId]?.progress + '%' }"
            &gt;&lt;/div&gt;
          &lt;/div&gt;
          &lt;span class="progress-text"&gt;
            {{ taskProgress[task.taskId]?.progress || 0 }}%
          &lt;/spandiv&gt;
        
        &lt;div class="task-actions"&gt;
          &lt;button 
            v-if="task.status === 'COMPLETED' && task.downloadUrl"
            @click="downloadFile(task.downloadUrl)"
          &gt;
            下载文件
          &lt;/button&gt;
        &lt;/div&gt;
      &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
&lt;/template&gt;

&lt;script&gt;
import { ref, onMounted, onUnmounted } from 'vue';

export default {
  name: 'DownloadManager',
  setup() {
    const selectedRecordingId = ref('');
    const format = ref('mp4');
    const recordings = ref([]);
    const downloadTasks = ref([]);
    const taskProgress = ref({});
    let progressInterval = null;

    const loadRecordings = async () =&gt; {
      // 加载可下载的录像列表
      const response = await fetch('/api/recordings');
      recordings.value = await response.json();
    };

    const loadDownloadTasks = async () =&gt; {
      // 加载下载任务列表
      const response = await fetch('/api/downloads');
      downloadTasks.value = await response.json();
    };

    const getTaskProgress = async (taskId) =&gt; {
      // 获取单个任务的进度
      try {
        const response = await fetch(`/api/downloads/progress/${taskId}`);
        const progress = await response.json();
        taskProgress.value[taskId] = progress;
      } catch (error) {
        console.error('获取进度失败:', error);
      }
    };

    const updateAllProgress = () =&gt; {
      // 更新所有处理中任务的进度
      downloadTasks.value.forEach(task =&gt; {
        if (task.status === 'PROCESSING') {
          getTaskProgress(task.taskId);
        }
      });
    };

    const createDownloadTask = async () =&gt; {
      if (!selectedRecordingId.value) {
        alert('请选择录像');
        return;
      }

      const response = await fetch('/api/downloads', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          recordingId: selectedRecordingId.value,
          format: format.value
        })
      });

      const task = await response.json();
      downloadTasks.value.unshift(task);
      taskProgress.value[task.taskId] = { progress: 0, total: 100 };
    };

    const downloadFile = (url) =&gt; {
      // 打开下载链接
      window.open(url, '_blank');
    };

    const formatTime = (time) =&gt; {
      return new Date(time).toLocaleString();
    };

    const getStatusText = (status) =&gt; {
      const statusMap = {
        'PENDING': '等待中',
        'PROCESSING': '处理中',
        'COMPLETED': '已完成',
        'FAILED': '失败'
      };
      return statusMap[status] || status;
    };

    const getTaskName = (task) =&gt; {
      if (task.isBatch) {
        return `批量下载 (${task.recordingIds.length}个文件)`;
      } else {
        const recording = recordings.value.find(r =&gt; r.id === task.recordingId);
        return recording ? recording.name : '录像文件';
      }
    };

    onMounted(() =&gt; {
      loadRecordings();
      loadDownloadTasks();
      
      // 每2秒更新一次进度
      progressInterval = setInterval(updateAllProgress, 2000);
    });

    onUnmounted(() =&gt; {
      // 清除定时器
      if (progressInterval) {
        clearInterval(progressInterval);
      }
    });

    return {
      selectedRecordingId,
      format,
      recordings,
      downloadTasks,
      taskProgress,
      createDownloadTask,
      downloadFile,
      formatTime,
      getStatusText,
      getTaskName
    };
  }
};
&lt;/script&gt;
                    
                
            

            
                核心优势
            
            
            
                
                    
                    多格式支持
                    MP4/AVI/MKV 格式转换
                
                
                    
                    批量打包
                    多文件 Zip 打包下载
                
                
                    
                    断点续传
                    支持断点续传下载
                
                
                    
                    安全下载
                    签名链接，有效期控制
                
                
                    
                    任务管理
                    下载任务队列管理
                
                
                    
                    多端支持
                    PC/手机自适应
                
                
                    
                    进度查询
                    实时显示下载进度
                
                
                    
                    大文件支持
                    分块传输，高效稳定
                
                
                    
                    高性能
                    异步处理，不阻塞主线程
