硬件驱动操作系统 - 架构师学习笔记
    
    
    
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            background: linear-gradient(135deg, #0f172a 0%, #1e293b 50%, #334155 100%);
            color: #e2e8f0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            min-height: 100vh;
            overflow-x: hidden;
        }
        
        .container {
            max-width: 1400px;
            margin: 0 auto;
            padding: 20px;
        }
        
        /* 主板样式 */
        .motherboard {
            background: linear-gradient(145deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
            border-radius: 20px;
            padding: 40px;
            box-shadow: 
                0 25px 50px rgba(0,0,0,0.5),
                inset 0 1px 0 rgba(255,255,255,0.1),
                inset 0 -1px 0 rgba(0,0,0,0.3);
            position: relative;
            overflow: hidden;
            border: 2px solid #2d3748;
        }
        
        /* 主板纹理 */
        .motherboard::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-image: 
                repeating-linear-gradient(45deg, transparent, transparent 30px, rgba(255,255,255,0.02) 30px, rgba(255,255,255,0.02) 60px),
                repeating-linear-gradient(-45deg, transparent, transparent 30px, rgba(255,255,255,0.02) 30px, rgba(255,255,255,0.02) 60px);
            z-index: 0;
            pointer-events: none;
        }
        
        /* 硬件组件基础样式 */
        .component {
            background: linear-gradient(145deg, #2d3748 0%, #1a202c 100%);
            border: 2px solid #4a5568;
            border-radius: 8px;
            padding: 15px;
            text-align: center;
            position: relative;
            z-index: 1;
            transition: all 0.3s ease;
            box-shadow: 
                0 4px 6px rgba(0,0,0,0.3),
                inset 0 1px 0 rgba(255,255,255,0.1);
        }
        
        .component:hover {
            transform: translateY(-3px);
            box-shadow: 
                0 8px 12px rgba(0,0,0,0.4),
                inset 0 1px 0 rgba(255,255,255,0.1);
            border-color: #718096;
        }
        
        .component.active {
            background: linear-gradient(145deg, #10b981 0%, #059669 100%);
            border-color: #34d399;
            box-shadow: 
                0 0 20px rgba(16, 185, 129, 0.6),
                inset 0 1px 0 rgba(255,255,255,0.2);
            transform: scale(1.02);
        }
        
        .component.processing {
            background: linear-gradient(145deg, #3b82f6 0%, #2563eb 100%);
            border-color: #60a5fa;
            box-shadow: 
                0 0 20px rgba(59, 130, 246, 0.6),
                inset 0 1px 0 rgba(255,255,255,0.2);
        }
        
        /* CPU样式 */
        .cpu {
            background: linear-gradient(145deg, #f59e0b 0%, #d97706 100%);
            border-color: #fbbf24;
            grid-column: span 2;
        }
        
        /* 内存样式 */
        .memory {
            background: linear-gradient(145deg, #3b82f6 0%, #2563eb 100%);
            border-color: #60a5fa;
        }
        
        /* 芯片组样式 */
        .chipset {
            background: linear-gradient(145deg, #8b5cf6 0%, #7c3aed 100%);
            border-color: #a78bfa;
        }
        
        /* BIOS样式 */
        .bios {
            background: linear-gradient(145deg, #ef4444 0%, #dc2626 100%);
            border-color: #f87171;
        }
        
        /* 电源样式 */
        .power {
            background: linear-gradient(145deg, #22c55e 0%, #16a34a 100%);
            border-color: #4ade80;
        }
        
        /* 时钟样式 */
        .clock {
            background: linear-gradient(145deg, #ec4899 0%, #db2777 100%);
            border-color: #f472b6;
        }
        
        /* 组件图标 */
        .component i {
            font-size: 2rem;
            margin-bottom: 8px;
            display: block;
        }
        
        .component h3 {
            font-size: 0.9rem;
            font-weight: 600;
            margin-bottom: 4px;
            color: #e2e8f0;
        }
        
        .component p {
            font-size: 0.75rem;
            color: #a0aec0;
        }
        
        /* 总线样式 */
        .bus {
            position: absolute;
            background: linear-gradient(to right, #4a5568, #718096, #4a5568);
            z-index: 0;
            border-radius: 2px;
        }
        
        .bus.horizontal {
            height: 4px;
            width: 100%;
        }
        
        .bus.vertical {
            width: 4px;
            height: 100%;
        }
        
        /* 数据流动画 */
        .data-flow {
            position: absolute;
            width: 12px;
            height: 12px;
            background: #10b981;
            border-radius: 50%;
            box-shadow: 0 0 10px rgba(16, 185, 129, 0.8);
            z-index: 2;
            opacity: 0;
        }
        
        .data-flow.active {
            animation: flow 1s linear infinite;
        }
        
        @keyframes flow {
            0% {
                opacity: 1;
                transform: translateX(0);
            }
            100% {
                opacity: 0;
                transform: translateX(100px);
            }
        }
        
        /* LED指示灯 */
        .led {
            position: absolute;
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: #ef4444;
            box-shadow: 0 0 8px rgba(239, 68, 68, 0.8);
            top: 8px;
            right: 8px;
            transition: all 0.3s ease;
        }
        
        .led.on {
            background: #10b981;
            box-shadow: 0 0 8px rgba(16, 185, 129, 0.8);
        }
        
        /* 控制面板 */
        .control-panel {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 12px;
            padding: 20px;
            margin-bottom: 20px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
        
        .btn {
            background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 8px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 6px rgba(59, 130, 246, 0.3);
            margin: 5px;
        }
        
        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 12px rgba(59, 130, 246, 0.4);
        }
        
        .btn:active {
            transform: translateY(0);
        }
        
        .btn.power-btn {
            background: linear-gradient(135deg, #22c55e 0%, #16a34a 100%);
            box-shadow: 0 4px 6px rgba(34, 197, 94, 0.3);
        }
        
        .btn.power-btn:hover {
            box-shadow: 0 6px 12px rgba(34, 197, 94, 0.4);
        }
        
        .btn.app-btn {
            background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);
            box-shadow: 0 4px 6px rgba(245, 158, 11, 0.3);
        }
        
        .btn.app-btn:hover {
            box-shadow: 0 6px 12px rgba(245, 158, 11, 0.4);
        }
        
        /* 终端样式 */
        .terminal {
            background: #0f172a;
            border-radius: 8px;
            padding: 15px;
            font-family: 'Courier New', monospace;
            font-size: 14px;
            color: #10b981;
            overflow-y: auto;
            max-height: 200px;
            border: 1px solid #334155;
            margin-top: 15px;
        }
        
        .terminal-line {
            margin-bottom: 5px;
            animation: fadeIn 0.3s ease;
        }
        
        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(5px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        
        /* 进度条 */
        .progress-bar {
            width: 100%;
            height: 6px;
            background: #374151;
            border-radius: 3px;
            margin: 8px 0;
            overflow: hidden;
        }
        
        .progress-fill {
            height: 100%;
            background: linear-gradient(to right, #10b981, #059669);
            width: 0%;
            transition: width 0.5s ease;
            box-shadow: 0 0 10px rgba(16, 185, 129, 0.6);
        }
        
        /* 主板布局网格 */
        .motherboard-grid {
            display: grid;
            grid-template-columns: repeat(6, 1fr);
            grid-template-rows: repeat(4, 1fr);
            gap: 15px;
            position: relative;
            min-height: 500px;
        }
        
        /* 标题样式 */
        h1, h2, h3 {
            background: linear-gradient(135deg, #60a5fa 0%, #3b82f6 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }
        
        /* 信息面板 */
        .info-panel {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 12px;
            padding: 20px;
            margin-bottom: 20px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
        
        .step-indicator {
            display: flex;
            justify-content: center;
            margin-bottom: 15px;
            gap: 10px;
        }
        
        .step {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            background: #d1d5db;
            transition: all 0.3s ease;
        }
        
        .step.active {
            background: #10b981;
            transform: scale(1.3);
            box-shadow: 0 0 10px rgba(16, 185, 129, 0.6);
        }
        
        /* 接口样式 */
        .interface {
            background: linear-gradient(145deg, #4a5568 0%, #2d3748 100%);
            border: 2px solid #718096;
            border-radius: 4px;
            padding: 8px;
            text-align: center;
            font-size: 0.7rem;
            color: #a0aec0;
        }
        
        /* 响应式设计 */
        @media (max-width: 1024px) {
            .motherboard-grid {
                grid-template-columns: repeat(3, 1fr);
                grid-template-rows: repeat(6, 1fr);
            }
            
            .cpu {
                grid-column: span 1;
            }
        }
        
        @media (max-width: 768px) {
            .motherboard-grid {
                grid-template-columns: repeat(2, 1fr);
                grid-template-rows: repeat(8, 1fr);
            }
            
            .motherboard {
                padding: 20px;
            }
            
            .component {
                padding: 10px;
            }
            
            .component i {
                font-size: 1.5rem;
            }
        }
        
        /* 状态指示器 */
        .status-badge {
            display: inline-block;
            padding: 4px 12px;
            border-radius: 12px;
            font-size: 0.75rem;
            font-weight: 600;
            margin-left: 8px;
        }
        
        .status-badge.offline {
            background: #ef4444;
            color: white;
        }
        
        .status-badge.online {
            background: #10b981;
            color: white;
        }
        
        .status-badge.processing {
            background: #3b82f6;
            color: white;
        }
    


    
        
            
                硬件驱动操作系统
            
            真实主板布局模拟 - 硬件工作原理可视化
        
        
        
        
            
                系统控制面板
            
            
                
                    电源启动
                
                
                    启动应用
                
                
                    重置系统
                
                
                    速度: 正常
                
            
        
        
        
        
            
                
                
                
                
                
                
            
            
            
                系统待机
                等待用户操作
            
            
            
                [系统] 等待电源启动...
            
        
        
        
        
            
                
                
                    
                    
                    电源供应
                    ATX 24Pin
                    
                        
                    
                
                
                
                
                    
                    
                    BIOS芯片
                    UEFI固件
                    
                        
                    
                
                
                
                
                    
                    
                    时钟发生器
                    4.0 GHz
                    
                        
                    
                
                
                
                
                    
                    
                    北桥芯片
                    内存控制器
                    
                        
                    
                
                
                
                
                    
                    
                    CPU
                    8核心处理器
                    
                        
                    
                    
                        状态: 待机
                    
                
                
                
                
                    
                    
                    内存插槽1
                    DDR4 8GB
                    
                        
                    
                
                
                
                
                    
                    
                    内存插槽2
                    DDR4 8GB
                    
                        
                    
                
                
                
                
                    
                    
                    南桥芯片
                    I/O控制器
                    
                        
                    
                
                
                
                
                    
                    
                    存储控制器
                    SATA/NVMe
                    
                        
                    
                
                
                
                
                    
                    
                    显卡
                    PCIe x16
                    
                        
                    
                
                
                
                
                    
                    
                    网卡
                    PCIe x1
                    
                        
                    
                
                
                
                
                    
                    
                    USB控制器
                    USB 3.2
                    
                        
                    
                
                
                
                
                    
                    
                    操作系统内核
                    Linux Kernel
                    
                        
                    
                    
                        内存: 0%
                    
                
                
                
                
                    
                    
                    应用程序
                    用户进程
                    
                        
                    
                
                
                
                
                    
                    
                    显示输出
                    显示器
                    
                        
                    
                
            
            
            
            
            
        
        
        
        
            工作流程说明
            
            
                
                    
                        电源启动流程
                    
                    
                        1. 电源供应器提供电力
                        2. BIOS芯片进行硬件自检
                        3. 时钟发生器启动系统时钟
                        4. 北桥芯片初始化内存控制器
                        5. CPU开始执行引导程序
                        6. 操作系统内核加载到内存
                    
                
                
                
                    
                        应用启动流程
                    
                    
                        1. 用户双击应用程序图标
                        2. 操作系统创建新进程
                        3. 分配内存空间
                        4. 加载程序代码到内存
                        5. CPU开始执行程序指令
                        6. 显示输出到屏幕
                    
                
            
        
    
    
    
        // 系统状态
        let systemState = {
            isPoweredOn: false,
            isAnimating: false,
            currentStep: 0,
            animationSpeed: 1000,
            memoryUsage: 0
        };
        
        // 电源启动步骤
        const powerOnSteps = [
            {
                title: "电源供应",
                description: "电源供应器开始供电，主板各组件获得电力",
                component: "powerSupply",
                duration: 1500,
                terminal: "[电源] ATX电源供应器启动，提供+12V、+5V、+3.3V电压"
            },
            {
                title: "BIOS自检",
                description: "BIOS芯片进行硬件自检和初始化",
                component: "biosChip",
                duration: 2000,
                terminal: "[BIOS] 系统自检开始... 检测CPU、内存、存储设备"
            },
            {
                title: "时钟启动",
                description: "时钟发生器开始产生系统时钟信号",
                component: "clockGenerator",
                duration: 1000,
                terminal: "[时钟] 系统时钟启动，频率: 4.0 GHz"
            },
            {
                title: "芯片组初始化",
                description: "北桥和南桥芯片组初始化",
                component: "northbridge",
                duration: 1500,
                terminal: "[芯片组] 北桥芯片初始化内存控制器，南桥芯片初始化I/O控制器"
            },
            {
                title: "CPU启动",
                description: "CPU开始执行引导程序",
                component: "cpu",
                duration: 2000,
                terminal: "[CPU] 处理器启动，开始执行引导程序"
            },
            {
                title: "系统加载",
                description: "操作系统内核加载到内存",
                component: "osKernel",
                duration: 2500,
                terminal: "[系统] 操作系统内核加载完成，系统启动成功"
            }
        ];
        
        // 应用启动步骤
        const appLaunchSteps = [
            {
                title: "用户操作",
                description: "用户双击应用程序图标",
                component: "application",
                duration: 1000,
                terminal: "[用户] 双击应用程序图标"
            },
            {
                title: "进程创建",
                description: "操作系统创建新进程",
                component: "osKernel",
                duration: 1500,
                terminal: "[系统] 创建新进程，分配进程ID"
            },
            {
                title: "内存分配",
                description: "为应用程序分配内存空间",
                component: "memory1",
                duration: 1500,
                terminal: "[内存] 分配内存空间，加载程序代码"
            },
            {
                title: "程序加载",
                description: "应用程序代码加载到内存",
                component: "storageController",
                duration: 2000,
                terminal: "[存储] 从磁盘读取程序文件到内存"
            },
            {
                title: "CPU执行",
                description: "CPU开始执行应用程序指令",
                component: "cpu",
                duration: 2000,
                terminal: "[CPU] 开始执行应用程序指令"
            },
            {
                title: "显示输出",
                description: "应用程序界面显示到屏幕",
                component: "displayOutput",
                duration: 1500,
                terminal: "[显示] 应用程序界面渲染完成"
            }
        ];
        
        // 初始化
        document.addEventListener('DOMContentLoaded', function() {
            updateStepDisplay();
            startClockAnimation();
        });
        
        // 时钟动画
        function startClockAnimation() {
            const clockProgress = document.getElementById('clockProgress');
            
            setInterval(() => {
                if (systemState.isPoweredOn) {
                    clockProgress.style.width = '100%';
                    setTimeout(() => {
                        clockProgress.style.width = '0%';
                    }, 500);
                }
            }, systemState.animationSpeed);
        }
        
        // 电源启动
        function startPowerOn() {
            if (systemState.isAnimating) return;
            
            systemState.isAnimating = true;
            systemState.isPoweredOn = true;
            systemState.currentStep = 0;
            
            clearTerminal();
            addTerminalLine("[系统] 开始电源启动流程...");
            
            // 逐步执行电源启动步骤
            let delay = 0;
            powerOnSteps.forEach((step, index) => {
                setTimeout(() => {
                    executeStep(step, index, powerOnSteps);
                }, delay);
                delay += step.duration;
            });
            
            // 启动完成
            setTimeout(() => {
                systemState.isAnimating = false;
                addTerminalLine("[系统] 电源启动完成，系统正常运行");
                activateComponent('powerLed');
            }, delay);
        }
        
        // 应用启动
        function startAppLaunch() {
            if (!systemState.isPoweredOn) {
                addTerminalLine("[错误] 系统未启动，请先启动电源");
                return;
            }
            
            if (systemState.isAnimating) return;
            
            systemState.isAnimating = true;
            systemState.currentStep = 0;
            
            addTerminalLine("[系统] 开始应用程序启动流程...");
            
            // 逐步执行应用启动步骤
            let delay = 0;
            appLaunchSteps.forEach((step, index) => {
                setTimeout(() => {
                    executeStep(step, index, appLaunchSteps);
                }, delay);
                delay += step.duration;
            });
            
            // 启动完成
            setTimeout(() => {
                systemState.isAnimating = false;
                addTerminalLine("[系统] 应用程序启动完成");
            }, delay);
        }
        
        // 执行步骤
        function executeStep(step, index, steps) {
            systemState.currentStep = index;
            updateStepDisplay();
            
            // 更新标题和描述
            document.getElementById('currentStepTitle').textContent = step.title;
            document.getElementById('currentStepDescription').textContent = step.description;
            
            // 激活组件
            activateComponent(step.component);
            
            // 更新进度条
            updateProgress(step.component, 100);
            
            // 添加终端信息
            addTerminalLine(step.terminal);
            
            // 激活LED
            const ledId = step.component.replace('Component', 'Led');
            const led = document.getElementById(ledId);
            if (led) {
                led.classList.add('on');
            }
            
            // 闪烁CPU
            if (step.component === 'cpu') {
                flashCpu();
            }
        }
        
        // 激活组件
        function activateComponent(componentId) {
            const component = document.getElementById(componentId);
            if (component) {
                component.classList.add('active');
                setTimeout(() => {
                    component.classList.remove('active');
                }, 1000);
            }
        }
        
        // 更新进度条
        function updateProgress(componentId, value) {
            const progressId = componentId + 'Progress';
            const progress = document.getElementById(progressId);
            if (progress) {
                progress.style.width = value + '%';
            }
        }
        
        // CPU闪烁
        function flashCpu() {
            const cpu = document.getElementById('cpu');
            const cpuLed = document.getElementById('cpuLed');
            
            if (cpu && cpuLed) {
                cpu.classList.add('processing');
                cpuLed.classList.add('on');
                
                setTimeout(() => {
                    cpu.classList.remove('processing');
                    cpuLed.classList.remove('on');
                }, 2000);
            }
        }
        
        // 更新步骤显示
        function updateStepDisplay() {
            const steps = document.querySelectorAll('.step');
            steps.forEach((step, index) => {
                if (index  {
                component.classList.remove('active', 'processing');
            });
            
            // 重置所有LED
            document.querySelectorAll('.led').forEach(led => {
                led.classList.remove('on');
            });
            
            // 重置所有进度条
            document.querySelectorAll('.progress-fill').forEach(progress => {
                progress.style.width = '0%';
            });
            
            // 重置步骤指示器
            updateStepDisplay();
            
            // 重置终端
            clearTerminal();
            addTerminalLine("[系统] 系统已重置，等待电源启动...");
            
            // 重置标题
            document.getElementById('currentStepTitle').textContent = "系统待机";
            document.getElementById('currentStepDescription').textContent = "等待用户操作";
        }
        
        // 切换速度
        function toggleSpeed() {
            const speedDisplay = document.getElementById('speedDisplay');
            
            if (systemState.animationSpeed === 1000) {
                systemState.animationSpeed = 500;
                speedDisplay.textContent = "快速";
            } else if (systemState.animationSpeed === 500) {
                systemState.animationSpeed = 200;
                speedDisplay.textContent = "极速";
            } else {
                systemState.animationSpeed = 1000;
                speedDisplay.textContent = "正常";
            }
        }
        
        // 键盘快捷键
        document.addEventListener('keydown', function(event) {
            if (event.code === 'Space') {
                event.preventDefault();
                if (!systemState.isPoweredOn) {
                    startPowerOn();
                } else {
                    startAppLaunch();
                }
            } else if (event.code === 'KeyR') {
                resetSystem();
            }
        });
