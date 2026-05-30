<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>中文点读课 - 游泳与买东西</title>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #f8f9fa;
            --card: #ffffff;
            --text-green: #2d6a4f;  /* 提问深绿色 */
            --text-blue: #0026e6;   /* 回答深蓝色 */
            --btn-next: #ff7043;
            --border-normal: #e0e0e0;
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        body {
            margin: 0; padding: 15px;
            background: var(--bg);
            font-family: 'Noto Sans SC', sans-serif;
            display: flex; flex-direction: column; align-items: center;
            justify-content: center; min-height: 100vh;
        }

        /* 启动遮罩层 */
        #shield {
            position: fixed; inset: 0; background: linear-gradient(135deg, #2d6a4f, #0026e6);
            z-index: 1000; color: white; display: flex;
            flex-direction: column; align-items: center; justify-content: center;
            cursor: pointer; text-align: center; padding: 20px;
        }

        .progress { font-size: 1.3rem; color: #7f8c8d; font-weight: bold; margin-bottom: 12px; }

        /* 主体点读卡片结构 */
        .card {
            background: var(--card); border-radius: 25px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.06);
            border: 3px solid var(--border-normal);
            width: 100%; max-width: 580px;
            padding: 35px; text-align: left;
            position: relative; display: flex; flex-direction: column;
            min-height: 460px; justify-content: space-between;
            transition: border-color 0.2s ease;
        }

        /* 内容水平排版：左边文字对白，右边实物图片 */
        .main-content {
            display: flex; width: 100%; align-items: center; gap: 20px; flex-grow: 1;
        }

        .text-section { flex: 1; display: flex; flex-direction: column; justify-content: center; }
        
        /* 右侧图片区域排版 */
        .image-section {
            width: 200px; height: 160px; display: flex; align-items: center; justify-content: center;
            overflow: hidden; border-radius: 12px; background: #ffffff;
            box-shadow: 0 4px 10px rgba(0,0,0,0.04);
        }
        
        .lesson-img {
            width: 100%; height: 100%; object-fit: cover;
        }

        .clickable-box { cursor: pointer; transition: transform 0.1s; user-select: none; padding: 10px 0; }
        .clickable-box:active { transform: scale(0.98); opacity: 0.85; }

        /* 对白文字样式优化 */
        .q-text { font-size: 2.2rem; color: var(--text-green); font-weight: bold; margin: 0 0 15px 0; }
        .a-text { font-size: 2.2rem; color: var(--text-blue); font-weight: bold; margin: 0; }

        /* 隐藏和显示的逻辑容器 */
        .answer-container {
            display: none; /* 初始隐藏，点击提问后浮现 */
            border-top: 2px dashed #f0f3f6; margin-top: 15px; padding-top: 15px;
        }

        /* 底部导航控制栏 */
        .nav-controls { width: 100%; display: flex; gap: 20px; margin-top: 30px; }
        .nav-btn {
            flex: 1; padding: 15px; border-radius: 50px; border: none;
            background: #2c3e50; color: white; font-size: 1.1rem;
            font-weight: bold; cursor: pointer; text-align: center;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
        }
        .nav-btn:active { opacity: 0.9; }

        .audio-icon { font-size: 1rem; color: #cbd5e1; margin-left: 10px; vertical-align: middle; }
    </style>
</head>
<body>

    <div id="shield" onclick="boot()">
        <div style="font-size: 5.5rem; margin-bottom: 15px;">🏊‍♂️</div>
        <h2>点击开始“看图识物与问答”点读练习</h2>
        <p style="font-size: 1.1rem; opacity: 0.95;">修复漏字问题 · 每一个字都清清楚楚</p>
    </div>

    <div class="progress" id="prog-info">1 / 6</div>
    
    <div class="card" id="lesson-card">
        <div class="main-content">
            <div class="text-section">
                <div class="clickable-box" onclick="clickQuestion()">
                    <p class="q-text" id="q-field"></p>
                </div>
                
                <div class="answer-container clickable-box" id="a-container" onclick="clickAnswer()">
                    <p class="a-text" id="a-field"></p>
                </div>
            </div>
            
            <div class="image-section">
                <img src="" alt="课件插图" class="lesson-img" id="content-img">
            </div>
        </div>

        <div class="nav-controls">
            <button class="nav-btn" onclick="prev()">上一个</button>
            <button class="nav-btn" style="background: var(--btn-next);" onclick="next()">下一个</button>
        </div>
    </div>

    <script>
        // 内置高保真准确图片URL，完全对应：游泳、游泳衣、游泳裤、红色裤子、十块钱
        const lessonData = [
            { 
                q: "你喜欢什么？", 
                a: "我喜欢游泳", 
                img: "https://images.unsplash.com/photo-1519315901367-f34ff9154487?w=500&q=80" // 正在游泳的人
            },
            { 
                q: "你要买什么？", 
                a: "我要买游泳衣", 
                img: "https://images.unsplash.com/photo-1551854838-212c50b4c184?w=500&q=80" // 专业的女士连体游泳衣
            },
            { 
                q: "你要买什么？", 
                a: "我要买游泳裤", 
                img: "https://images.unsplash.com/photo-1502163140606-888448ae8cfe?w=500&q=80" // 泳池边的男士运动游泳裤
            },
            { 
                q: "你喜欢什么颜色？", 
                a: "我喜欢红色", 
                img: "https://images.unsplash.com/photo-1594633312681-425c7b97ccd1?w=500&q=80", // 纯正红色的短裤
                hasBlueBorder: true // 第4张图特有的蓝色粗外框
            },
            { 
                q: "你看这件怎么样？", 
                a: "这件很好", 
                img: "https://images.unsplash.com/photo-1594633312681-425c7b97ccd1?w=500&q=80" // 红色衣服图卡展示
            },
            { 
                q: "游泳裤多少钱？", 
                a: "游泳裤十块钱", 
                img: "https://images.unsplash.com/photo-1509316975850-ff9c5deb0cd9?w=500&q=80" // 清晰的十块钱纸币面值
            }
        ];

        let currentIndex = 0;
        const speech = window.speechSynthesis;

        function boot() {
            speech.speak(new SpeechSynthesisUtterance("")); // 解锁移动端浏览器的音频限制
            document.getElementById('shield').style.display = 'none';
            renderCard();
        }

        function renderCard() {
            const data = lessonData[currentIndex];
            const cardEl = document.getElementById('lesson-card');
            
            // 1. 更新当前进度与右侧插图
            document.getElementById('prog-info').textContent = `${currentIndex + 1} / ${lessonData.length}`;
            document.getElementById('content-img').src = data.img;
            
            // 2. 还原第4张图特有的“蓝色粗边框”逻辑
            if (data.hasBlueBorder) {
                cardEl.style.borderColor = "var(--text-blue)";
                cardEl.style.borderWidth = "5px";
            } else {
                cardEl.style.borderColor = "var(--border-normal)";
                cardEl.style.borderWidth = "3px";
            }
            
            // 3. 渲染提问和回答文本
            document.getElementById('q-field').innerHTML = data.q + '<span class="audio-icon">🔊</span>';
            document.getElementById('a-field').innerHTML = data.a + '<span class="audio-icon">🔊</span>';
            document.getElementById('a-container').style.display = 'none';

            // 4. 新卡片载入时，默认自动清晰慢速朗读绿色提问
            speakWithNoMissingWords(data.q);
        }

        // 点击提问框：清晰发音，并在 1.3 秒后自动展开并朗读下方的蓝色答案
        function clickQuestion() {
            const data = lessonData[currentIndex];
            speakWithNoMissingWords(data.q);
            
            setTimeout(() => {
                document.getElementById('a-container').style.display = 'block';
                speakWithNoMissingWords(data.a);
            }, 1300);
        }

        // 点击回答框：支持学生独立重复、逐字听清发音
        function clickAnswer() {
            const data = lessonData[currentIndex];
            speakWithNoMissingWords(data.a);
        }

        // 核心技术改进：过滤末尾标点符号，确保最后一个汉字能被完美认读发音
        function speakWithNoMissingWords(text) {
            speech.cancel(); // 强行斩断上一句音频流

            // 1. 智能过滤句子末尾的标点符号，防止标点和最后一个汉字粘连导致不发音
            const cleanText = text.replace(/[？。！，,?!\s]$/g, '');

            // 2. 将汉字中间用空格彻底隔开，保证一字一顿
            const spacedText = cleanText.split("").join(" ");

            const utterance = new SpeechSynthesisUtterance(spacedText);
            utterance.lang = 'zh-CN';
            utterance.rate = 0.72;  // 针对儿童教学的极清慢速
            utterance.pitch = 1.0; 

            // 智能检索普通话女声引擎
            const voices = speech.getVoices();
            const preferredVoice = voices.find(v => v.name.includes('Xiaoxiao') || v.lang === 'zh-CN');
            if (preferredVoice) utterance.voice = preferredVoice;

            speech.speak(utterance);
        }

        function next() {
            currentIndex = (currentIndex + 1) % lessonData.length;
            renderCard();
        }

        function prev() {
            currentIndex = (currentIndex - 1 + lessonData.length) % lessonData.length;
            renderCard();
        }

        window.speechSynthesis.onvoiceschanged = () => speech.getVoices();
    </script>
</body>
</html>
