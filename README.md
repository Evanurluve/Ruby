<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>Typography Spatial Wave Generator</title>
    <style>
        :root {
            --bg-color: #0b0b0b;
            --panel-bg: #141414;
            --text-main: #ffffff;
            --border-color: #262626;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-main);
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            display: flex;
            height: 100vh;
            overflow: hidden;
        }

        /* 左侧预览区域 */
        .preview-pane {
            flex: 1;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            padding: 40px;
            position: relative;
        }

        .text-grid {
            display: flex;
            flex-direction: column;
            gap: var(--line-gap, 8px);
        }

        .row {
            display: flex;
            gap: var(--col-gap, 10px);
            white-space: nowrap;
        }

        .char {
            /* 使用支持字重轴(wght)的变体字体 */
            font-family: 'Inter', system-ui, sans-serif;
            font-weight: var(--base-weight, 400);
            font-size: var(--font-size, 48px);
            transition: font-weight 0.05s ease-out;
            display: inline-block;
        }

        /* 右侧控制面板 */
        .control-pane {
            width: 380px;
            background-color: var(--panel-bg);
            border-left: 1px solid var(--border-color);
            display: flex;
            flex-direction: column;
            padding: 24px;
            overflow-y: auto;
        }

        .panel-title {
            font-size: 14px;
            font-weight: 600;
            letter-spacing: 1px;
            margin-bottom: 24px;
            display: flex;
            align-items: center;
            gap: 8px;
            border-bottom: 1px solid var(--border-color);
            padding-bottom: 12px;
        }

        .control-group {
            margin-bottom: 20px;
        }

        .control-group label {
            display: block;
            font-size: 12px;
            color: #a3a3a3;
            margin-bottom: 8px;
        }

        .control-group input[type="text"],
        .control-group select {
            width: 100%;
            background: #1f1f1f;
            border: 1px solid var(--border-color);
            color: #fff;
            padding: 10px;
            border-radius: 6px;
            font-size: 13px;
        }

        .control-group input[type="range"] {
            width: 100%;
            accent-color: #fff;
            cursor: pointer;
        }

        .range-value {
            float: right;
            font-family: monospace;
            color: #fff;
        }
    </style>
</head>
<body>

    <!-- 左侧波形排版展示 -->
    <div class="preview-pane">
        <div class="text-grid" id="textGrid"></div>
    </div>

    <!-- 右侧控制面板 -->
    <div class="control-pane">
        <div class="panel-title">⚙ 排版参数微调</div>

        <div class="control-group">
            <label>显示文本</label>
            <input type="text" id="inputText" value="BREATHE OUT">
        </div>

        <div class="control-group">
            <label>全局相位位移 (Phase Shift) <span class="range-value" id="val-phase">4.50</span></label>
            <input type="range" id="phaseShift" min="0" max="10" step="0.1" value="4.5">
        </div>

        <div class="control-group">
            <label>行际相位偏差 (Row Offset) <span class="range-value" id="val-row">0.70</span></label>
            <input type="range" id="rowOffset" min="0" max="2" step="0.05" value="0.7">
        </div>

        <div class="control-group">
            <label>字际相位偏差 (Char Offset) <span class="range-value" id="val-char">0.20</span></label>
            <input type="range" id="charOffset" min="0" max="1" step="0.02" value="0.2">
        </div>

        <div class="control-group">
            <label>基准字号 (Font Size) <span class="range-value" id="val-size">48px</span></label>
            <input type="range" id="fontSize" min="20" max="100" step="1" value="48">
        </div>

        <div class="control-group">
            <label>横向间距比例 (Col Spacing) <span class="range-value" id="val-colgap">10px</span></label>
            <input type="range" id="colSpacing" min="0" max="30" step="1" value="10">
        </div>

        <div class="control-group">
            <label>纵向行距比例 (Line Height) <span class="range-value" id="val-linegap">8px</span></label>
            <input type="range" id="lineGap" min="0" max="30" step="1" value="8">
        </div>
    </div>

    <script>
        const textGrid = document.getElementById('textGrid');
        const inputText = document.getElementById('inputText');
        const phaseShiftInput = document.getElementById('phaseShift');
        const rowOffsetInput = document.getElementById('rowOffset');
        const charOffsetInput = document.getElementById('charOffset');
        const fontSizeInput = document.getElementById('fontSize');
        const colSpacingInput = document.getElementById('colSpacing');
        const lineGapInput = document.getElementById('lineGap');

        const rowCount = 14; // 重复显示的行数

        function render() {
            const text = inputText.value;
            const phase = parseFloat(phaseShiftInput.value);
            const rowOff = parseFloat(rowOffsetInput.value);
            const charOff = parseFloat(charOffsetInput.value);
            const size = parseInt(fontSizeInput.value);
            const colGap = parseInt(colSpacingInput.value);
            const lineGap = parseInt(lineGapInput.value);

            // 更新CSS变量控制整体样式
            textGrid.style.setProperty('--font-size', size + 'px');
            textGrid.style.setProperty('--col-gap', colGap + 'px');
            textGrid.style.setProperty('--line-gap', lineGap + 'px');

            // 更新数值显示
            document.getElementById('val-phase').innerText = phase.toFixed(2);
            document.getElementById('val-row').innerText = rowOff.toFixed(2);
            document.getElementById('val-char').innerText = charOff.toFixed(2);
            document.getElementById('val-size').innerText = size + 'px';
            document.getElementById('val-colgap').innerText = colGap + 'px';
            document.getElementById('val-linegap').innerText = lineGap + 'px';

            textGrid.innerHTML = '';

            // 动态生成网格与波形字重
            for (let r = 0; r < rowCount; r++) {
                const rowEl = document.createElement('div');
                rowEl.className = 'row';

                for (let c = 0; c < text.length; c++) {
                    const charSpan = document.createElement('span');
                    charSpan.className = 'char';
                    charSpan.innerText = text[c];

                    // 核心数学波形算法：通过正弦函数计算当前字符的字重 (100 - 900)
                    const angle = phase + (r * rowOff) + (c * charOff);
                    const sineValue = (Math.sin(angle) + 1) / 2; // 归一化到 0 ~ 1
                    const weight = Math.round(100 + sineValue * 800); // 映射到变体字体字重区间

                    charSpan.style.fontVariationSettings = `'wght' ${weight}`;
                    // 渐变透明度让视觉更有层次感
                    charSpan.style.opacity = 0.2 + sineValue * 0.8;

                    rowEl.appendChild(charSpan);
                }
                textGrid.appendChild(rowEl);
            }
        }

        // 监听所有输入控件的变动
        [inputText, phaseShiftInput, rowOffsetInput, charOffsetInput, fontSizeInput, colSpacingInput, lineGapInput].forEach(element => {
            element.addEventListener('input', render);
        });

        // 初始化渲染
        render();
    </script>
</body>
</html>
