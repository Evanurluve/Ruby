<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>Typography Spatial Wave - Transparent Export</title>
    <style>
        :root {
            --bg-color: #0b0b0b;
            --panel-bg: #141414;
            --text-color: #ffffff;
            --border-color: #262626;
            --fixed-line-height: 1.2; 
        }

        * { box-sizing: border-box; margin: 0; padding: 0; }

        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            font-family: 'SF Mono', 'Monaco', 'Inconsolata', 'Roboto Mono', monospace;
            display: flex;
            height: 100vh;
            overflow: hidden;
        }

        /* 左侧预览区域 */
        #captureArea {
            flex: 1;
            display: flex;
            justify-content: center;
            align-items: center;
            background-color: var(--bg-color); /* 默认有背景，方便看效果 */
            padding: 40px;
            position: relative;
        }

        .text-grid {
            display: grid;
            line-height: var(--fixed-line-height); 
        }

        .row {
            display: flex;
            justify-content: space-between; 
            margin: 0 -0.05em;
        }

        .word-group {
             display: flex;
             letter-spacing: -0.02em; 
        }

        .char {
            font-weight: var(--base-weight, 400);
            font-size: var(--font-size, 60px); 
            transition: font-weight 0.05s ease-out;
            display: inline-block;
            text-align: center;
            min-width: 0.55em;
            /* 使用用户自定义的颜色作为基准色 */
            color: var(--text-color);
        }

        /* 右侧控制面板 */
        .control-pane {
            width: 380px;
            background-color: var(--panel-bg);
            border-left: 1px solid var(--border-color);
            display: flex;
            flex-direction: column;
            padding: 20px;
            overflow-y: auto;
            z-index: 10;
        }

        .panel-title {
            font-size: 14px; font-weight: 600; letter-spacing: 1px; margin-bottom: 20px;
            display: flex; align-items: center; justify-content: space-between;
            border-bottom: 1px solid var(--border-color); padding-bottom: 12px;
        }

        .control-group { margin-bottom: 16px; }
        .control-group label { display: block; font-size: 12px; color: #a3a3a3; margin-bottom: 6px; }
        .control-group input[type="text"],
        .control-group input[type="color"] { 
            width: 100%; background: #1f1f1f; border: 1px solid var(--border-color); color: #fff; padding: 6px; border-radius: 4px; font-size: 13px; 
        }
        .control-group input[type="color"] { height: 36px; cursor: pointer; padding: 2px; }
        .control-group input[type="range"] { accent-color: #fff; cursor: pointer; width: 100%; }
        .range-value { float: right; font-family: monospace; color: #fff; font-size: 11px;}

        .export-btn {
            background-color: #333;
            color: white;
            border: none;
            padding: 8px 12px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 12px;
        }
        .export-btn:hover { background-color: #444; }

    </style>
</head>
<body>

    <!-- 左侧预览区域 -->
    <div id="captureArea">
        <div class="text-grid" id="textGrid"></div>
    </div>

    <!-- 右侧控制面板 -->
    <div class="control-pane">
        <div class="panel-title">
            ⚙ 排版与透明导出
            <button class="export-btn" id="exportPngBtn">导出透明PNG</button>
        </div>

        <div class="control-group">
            <label>字体颜色选择</label>
            <input type="color" id="textColorPicker" value="#ffffff">
        </div>

        <div class="control-group">
            <label>左侧文本</label>
            <input type="text" id="inputLeft" value="JOEYI">
        </div>
         <div class="control-group">
            <label>右侧文本</label>
            <input type="text" id="inputRight" value="WANAGA">
        </div>

        <div class="control-group">
            <label>全局相位位移 (Phase) <span class="range-value" id="val-phase">4.50</span></label>
            <input type="range" id="phaseShift" min="0" max="10" step="0.05" value="4.5">
        </div>

        <div class="control-group">
            <label>行际相位偏差 (Row Offset) <span class="range-value" id="val-row">0.70</span></label>
            <input type="range" id="rowOffset" min="0" max="2" step="0.02" value="0.7">
        </div>

        <div class="control-group">
            <label>字际相位偏差 (Char Offset) <span class="range-value" id="val-char">0.20</span></label>
            <input type="range" id="charOffset" min="0" max="1" step="0.01" value="0.2">
        </div>

        <div class="control-group">
            <label>基准字号 (Font Size) <span class="range-value" id="val-size">60px</span></label>
            <input type="range" id="fontSize" min="30" max="120" step="1" value="60">
        </div>
    </div>

    <!-- 引入 html2canvas 库 -->
    <script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>

    <script>
        const textGrid = document.getElementById('textGrid');
        const inputLeft = document.getElementById('inputLeft');
        const inputRight = document.getElementById('inputRight');
        const textColorPicker = document.getElementById('textColorPicker');
        const phaseShiftInput = document.getElementById('phaseShift');
        const rowOffsetInput = document.getElementById('rowOffset');
        const charOffsetInput = document.getElementById('charOffset');
        const fontSizeInput = document.getElementById('fontSize');
        const captureArea = document.getElementById('captureArea');
        const exportBtn = document.getElementById('exportPngBtn');

        const rowCount = 14; 

        function renderRow(text, rowIndex, globalPhase, charOffsetVal) {
            const wordGroupEl = document.createElement('div');
            wordGroupEl.className = 'word-group';

            for (let c = 0; c < text.length; c++) {
                const charSpan = document.createElement('span');
                charSpan.className = 'char';
                charSpan.innerText = text[c];

                const angle = globalPhase + (rowIndex * parseFloat(rowOffsetInput.value)) + (c * charOffsetVal);
                const sineValue = (Math.sin(angle) + 1) / 2; 
                const weight = Math.round(100 + sineValue * 800); 

                charSpan.style.fontVariationSettings = `'wght' ${weight}`;
                // 通过透明度营造层次感
                charSpan.style.opacity = 0.15 + sineValue * 0.85;
                
                wordGroupEl.appendChild(charSpan);
            }
            return wordGroupEl;
        }

        function render() {
            const leftText = inputLeft.value;
            const rightText = inputRight.value;
            const phase = parseFloat(phaseShiftInput.value);
            const charOff = parseFloat(charOffsetInput.value);
            const size = parseInt(fontSizeInput.value);
            const textColor = textColorPicker.value;

            // 动态设置颜色和样式变量
            document.documentElement.style.setProperty('--text-color', textColor);
            textGrid.style.setProperty('--font-size', size + 'px');

            document.getElementById('val-phase').innerText = phase.toFixed(2);
            document.getElementById('val-row').innerText = rowOffsetInput.value;
            document.getElementById('val-char').innerText = charOff.toFixed(2);
            document.getElementById('val-size').innerText = size + 'px';

            textGrid.innerHTML = '';

            for (let r = 0; r < rowCount; r++) {
                const rowEl = document.createElement('div');
                rowEl.className = 'row';

                const leftGroup = renderRow(leftText, r, phase, charOff);
                rowEl.appendChild(leftGroup);

                const rightGroup = renderRow(rightText, r, phase, -charOff); 
                rowEl.appendChild(rightGroup);

                textGrid.appendChild(rowEl);
            }
        }

        // 监听所有输入变动
        [inputLeft, inputRight, textColorPicker, phaseShiftInput, rowOffsetInput, charOffsetInput, fontSizeInput].forEach(element => {
            element.addEventListener('input', render);
        });

        // --- 透明背景导出功能 ---
        exportBtn.addEventListener('click', () => {
            // 1. 导出前临时把网页左侧的背景设为透明（null）
            captureArea.style.backgroundColor = 'transparent';
            captureArea.style.overflow = 'hidden';
            
            html2canvas(captureArea, {
                backgroundColor: null, // 关键：设置为null开启透明底
                scale: 2, // 2倍高清缩放
                logging: false,
            }).then(canvas => {
                const link = document.createElement('a');
                link.download = 'typography_transparent.png';
                link.href = canvas.toDataURL('image/png');
                link.click();
                
                // 2. 导出完成后，把网页左侧的背景色恢复回来，方便你继续观看
                captureArea.style.backgroundColor = 'var(--bg-color)';
                captureArea.style.overflow = '';
            });
        });

        // 初始化渲染
        render();
    </script>
</body>
</html>
