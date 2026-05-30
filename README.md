<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>中文点读课 - 这是什么颜色</title>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #f8f9fa;
            --card: #ffffff;
            --text-dark: #2c3e50;
            --btn-next: #ff7043;
            --blue-q: #0026e6; /* 还原截图中问题的标准深蓝色 */
            --blue-q-en: #4a90e2; /* 英文问题的浅蓝色 */
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
            position: fixed; inset: 0; background: linear-gradient(135deg, #0026e6, #4a90e2);
            z-index: 1000; color: white; display: flex;
            flex-direction: column; align-items: center; justify-content: center;
            cursor: pointer; text-align: center; padding: 20px;
        }

        .progress { font-size: 1.3rem; color: #7f8c8d; font-weight: bold; margin-bottom: 12px; }

        /* 主体卡片排版 */
        .card {
            background: var(--card); border-radius: 25px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.06);
            width: 100%; max-width: 600px;
            padding: 30px; text-align: left;
            position: relative; display: flex; flex-direction: column;
            min-height: 500px; justify-content: space-between;
        }

        /* 白色卡片模式下的特殊黑底背景（完美还原白色截图效果） */
        .card.white-mode-card {
            background: #000000 !important;
        }

        /* 内容布局 */
        .main-content {
            display: flex; width: 100%; align-items: center; gap: 25px; flex-grow: 1;
        }

        .text-section { flex: 1; display: flex; flex-direction: column; justify-content: center; }
        
        /* 核心改进：100% 准确的超大颜色展示实体卡片 */
        .color-card-area {
            width: 180px; height: 180px; display: flex; align-items: center; justify-content: center;
        }
        
        .color-solid-plate {
            width: 100%; height: 100%; border-radius: 20px;
            box-shadow: 0 6px 15px rgba(0,0,0,0.08);
            border: 2px solid rgba(0,0,0,0.05);
            transition: all 0.3s ease;
        }

        .clickable-area { cursor: pointer; transition: transform 0.1s; user-select: none; }
        .clickable-area:active { transform: scale(0.98); opacity: 0.8; }

        /* 问句区域样式 */
        .question-box { margin-bottom: 30px; }
        .q-zh { font-size: 2.2rem; color: var(--blue-q); font-weight: bold; margin: 0 0 6px 0; }
        .q-en { font-size: 1.4rem; color: var(--blue-q-en); font-weight: normal; margin: 0; }

        /* 白色卡片模式下的问句样式 */
        .white-mode-text .q-zh { color: #4a90e2 !important; }
        .white-mode-text .q-en { color: #ffffff !important; }

        /* 答句区域样式 */
        .answer-box {
            display: none; /* 初始隐藏 */
            border-top: 2px dashed #f0f3f6; padding-top: 25px;
        }
        .white-mode-text .answer-box { border-top: 2px dashed #333333; }
        
        .a-zh { font-size: 2.3rem; font-weight: bold; margin: 0 0 6px 0; }
        .a-en { font-size: 1.5rem; font-weight: bold; margin: 0; }

        /* 底部导航控制栏 */
        .nav-controls { width: 100%; display: flex; gap: 20px; margin-top: 25px; }
        .nav-btn {
            flex: 1; padding: 15px; border-radius: 50px; border: none;
            background: var(--text-dark); color: white; font-size: 1.1rem;
            font-weight: bold; cursor: pointer; text-align: center;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
        }
        .nav-btn:active { opacity: 0.9; }

        .audio-tip { font-size: 0.9rem; color: #cbd5e1; margin-left: 8px; vertical-align: middle; }
    </style>
</head>
<body>

    <div id="shield" onclick="boot()">
        <div style="font-size: 5.5rem; margin-bottom: 10px;">🎨</div>
        <h2>点击开始“这是什么颜色”点读练习</h2>
        <p style="font-size: 1.1rem; opacity: 0.9;">慢速逐字发音 · 每一个句子都能点读</p>
    </div>

    <div class="progress" id="prog-info">1 / 11</div>
    
    <div class="card" id="main-card">
        <div class="main-content" id="text-container">
            <div class="text-section">
                <div class="question-box clickable-area" onclick="clickQuestion()">
                    <p class="q-zh" id="q-zh-text"></p>
                    <p class="q-en" id="q-en-text"></p>
                </div>
                
                <div class="answer-box clickable-area" id="ans-section" onclick="clickAnswer()">
                    <p class="a-zh" id="a-zh-text"></p>
                    <p class="a-en" id="a-en-text"></p>
                </div>
            </div>
            
            <div class="color-card-area">
                <div class="color-solid-plate" id="color-plate"></div>
            </div>
        </div>

        <div class="nav-controls">
            <button class="nav-btn" onclick="prev()">上一个</button>
            <button class="nav-btn" style="background: var(--btn-next);" onclick="next()">下一个</button>
        </div>
    </div>

    <script>
        // 严格还原截图顺序及配色的标准 11 种颜色数据库
        const colorData = [
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是红色", aEn: "This is red.", hex: "#D62828", isWhiteMode: false }, // 纯正红
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是橙色", aEn: "This is orange.", hex: "#E65C00", isWhiteMode: false }, // 纯正橙
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是黄色", aEn: "This is yellow.", hex: "#FFCC00", isWhiteMode: false }, // 纯正黄
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是绿色", aEn: "This is green.", hex: "#2D6A4F", isWhiteMode: false }, // 纯正绿
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是蓝色", aEn: "This is blue.", hex: "#0026E6", isWhiteMode: false }, // 纯正蓝
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是紫色", aEn: "This is purple.", hex: "#8A2BE2", isWhiteMode: false }, // 纯正紫
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是粉色", aEn: "This is pink.", hex: "#FF1493", isWhiteMode: false }, // 纯正粉
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是白色", aEn: "This is white.", hex: "#FFFFFF", isWhiteMode: true }, // 白色（开启截图里的黑底卡片模式）
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是黑色", aEn: "This is black.", hex: "#000000", isWhiteMode: false }, // 纯正黑
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是灰色", aEn: "This is grey.", hex: "#808080", isWhiteMode: false }, // 纯正灰
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是棕色", aEn: "This is brown.", hex: "#6E3A0F", isWhiteMode: false } // 纯正棕
        ];

        let index = 0;
        const synth = window.speechSynthesis;

        function boot() {
            synth.speak(new SpeechSynthesisUtterance("")); // 激活移动端发声环境
            document.getElementById('shield').style.display = 'none';
            render();
        }

        function render() {
            const item = colorData[index];
            const cardNode = document.getElementById('main-card');
            const textContainer = document.getElementById('text-container');
            
            // 实时更新顶部页码进度
            document.getElementById('prog-info').textContent = `${index + 1} / ${colorData.length}`;
            
            // 改变图卡区域的纯色色块，颜色绝对纯正
            document.getElementById('color-plate').style.backgroundColor = item.hex;
            
            // 1. 处理特殊模式：如果当前是白色，完美还原截图里的“黑底白字”风格
            if (item.isWhiteMode) {
                cardNode.classList.add('white-mode-card');
                textContainer.classList.add('white-mode-text');
            } else {
                cardNode.classList.remove('white-mode-card');
                textContainer.classList.remove('white-mode-text');
            }
            
            // 2. 写入问句文本
            document.getElementById('q-zh-text').innerHTML = item.qZh + '<span class="audio-tip">🔊</span>';
            document.getElementById('q-en-text').textContent = item.qEn;
            
            // 3. 写入答句文本并调整字体颜色（与卡片实体颜色完全保持一致）
            const azhNode = document.getElementById('a-zh-text');
            const aenNode = document.getElementById('a-en-text');
            azhNode.innerHTML = item.aZh + '<span class="audio-tip">🔊</span>';
            aenNode.textContent = item.aEn;
            
            // 设置文字颜色（白色卡片模式文字显纯白，其余跟色块同步）
            const textStyleColor = item.isWhiteMode ? "#FFFFFF" : item.hex;
            azhNode.style.color = textStyleColor;
            aenNode.style.color = textStyleColor;

            // 切换页面时，先将回答区隐藏
            document.getElementById('ans-section').style.display = 'none';

            // 自动慢速清晰播放蓝色的提问
            speakDual(item.qZh, item.qEn);
        }

        // 点击提问：发声，并在1.4秒延迟（给孩子反应留白）后，自动向下浮现回答并清晰发声
        function clickQuestion() {
            const item = colorData[index];
            speakDual(item.qZh, item.qEn);
            
            setTimeout(() => {
                document.getElementById('ans-section').style.display = 'block';
                speakDual(item.aZh, item.aEn);
            }, 1400);
        }

        // 点击回答：独立、高保真重复点读
        function clickAnswer() {
            const item = colorData[index];
            speakDual(item.aZh, item.aEn);
        }

        // 核心技术：逐字慢速、确保“字字听清”的双语发声引擎
        function speakDual(zhText, enText) {
            synth.cancel(); // 强行斩断当前音频线，防止点击过快时产生重音混乱

            // 将中文句子分解成单字，中间加入微小的连字符空隙，确保发音嘴型到位、吐字颗粒清晰
            const clearZhText = zhText.split("").join(" ");

            // 1. 中文朗读配置（针对学生磨耳朵特别优化过语速）
            const msgZh = new SpeechSynthesisUtterance(clearZhText);
            msgZh.lang = 'zh-CN';
            msgZh.rate = 0.75; // 极清慢速
            msgZh.pitch = 1.0;
            
            const voices = synth.getVoices();
            const zhVoice = voices.find(v => v.name.includes('Xiaoxiao') || v.lang === 'zh-CN');
            if (zhVoice) msgZh.voice = zhVoice;

            // 2. 英文朗读配置
            const msgEn = new SpeechSynthesisUtterance(enText);
            msgEn.lang = 'en-US';
            msgEn.rate = 0.8; // 保证语调标准自然且清晰
            const enVoice = voices.find(v => v.lang.includes('en-US'));
            if (enVoice) msgEn.voice = enVoice;

            // 队列依序播放
            synth.speak(msgZh);
            synth.speak(msgEn);
        }

        function next() {
            index = (index + 1) % colorData.length;
            render();
        }

        function prev() {
            index = (index - 1 + colorData.length) % colorData.length;
            render();
        }

        // 监听浏览器语音引擎加载
        window.speechSynthesis.onvoiceschanged = () => synth.getVoices();
    </script>
</body>
</html>
