<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>整數的分數倍 - 表徵互動教學</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* 允許網頁捲動以適應不同載具 */
        html, body {
            height: auto;
            overflow-y: auto;
            scroll-behavior: smooth;
        }

        .modal-overlay { background-color: rgba(0, 0, 0, 0.5); backdrop-filter: blur(4px); }
        
        /* 分數切分線：紅色虛線 */
        .fraction-split-line { stroke: #ef4444; stroke-width: 3; stroke-dasharray: 6,4; pointer-events: none; }
        
        /* 未選取的底色：淡藍色 */
        .base-unit-fill { fill: #e0f2fe; transition: all 0.2s; }

        .btn-split {
            @apply w-12 h-12 flex items-center justify-center rounded-xl border-2 border-slate-200 font-bold transition-all text-lg bg-white;
        }
        .btn-split-active {
            @apply bg-red-500 border-red-500 text-white shadow-md transform scale-105;
        }

        /* 標準分數顯示樣式 (分子在上分母在下) */
        .frac-display {
            display: inline-flex;
            flex-direction: column;
            vertical-align: middle;
            text-align: center;
            line-height: 1.1;
            padding: 0 0.1em;
        }
        .frac-display .top { 
            border-bottom: 2px solid currentColor; 
            padding: 0 0.2em;
        }
        .frac-display .bottom { 
            padding: 0 0.2em;
        }

        /* 讓畫布在小螢幕上可以橫向滾動 */
        .svg-container {
            width: 100%;
            overflow-x: auto;
            display: flex;
            justify-content: center;
            padding: 20px 0;
        }
    </style>
</head>
<body class="bg-slate-50 min-h-screen font-sans text-slate-900 pb-20">

    <!-- 頂部導覽列 -->
    <header class="sticky top-0 z-40 bg-white/80 backdrop-blur-md shadow-sm p-4 flex justify-between items-center border-b-2 border-slate-200">
        <div class="flex items-center space-x-3">
            <button onclick="openSettings()" class="p-2 bg-blue-100 text-blue-600 rounded-lg hover:bg-blue-200 transition-colors flex items-center gap-2 font-bold text-sm">
                <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73-.73l.22-.39a2 2 0 0 0-.73-2.73l-.15-.08a2 2 0 0 1-1-1.74v-.5a2 2 0 0 1 1-1.74l.15-.09a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-2 0l-.43-.25a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z"/><circle cx="12" cy="12" r="3"/></svg>
                更換題目
            </button>
            <h1 class="font-black text-base tracking-tight hidden sm:block">整數的分數倍</h1>
        </div>
        
        <div class="flex items-center gap-3">
            <div id="status-tag" class="bg-purple-100 text-purple-700 px-3 py-1 rounded-full text-xs font-bold border border-purple-200 flex items-center gap-1">
                目標：<span id="target-frac-mini"></span> 倍
            </div>
            <button onclick="resetApp()" class="p-2 bg-slate-100 text-slate-600 rounded-full hover:bg-slate-200 transition-colors">
                <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"/><path d="M3 3v5h5"/></svg>
            </button>
        </div>
    </header>

    <main class="max-w-5xl mx-auto p-4 flex flex-col gap-6">
        
        <!-- 任務引導區 -->
        <section class="bg-white rounded-2xl shadow-sm p-6 border border-slate-200">
            <h2 class="text-[10px] font-black text-slate-400 mb-2 uppercase tracking-widest">任務引導</h2>
            <div id="guide-text" class="text-2xl sm:text-3xl text-slate-800 font-bold flex items-center flex-wrap gap-2">
                <!-- 由 JS 動態生成 -->
            </div>
        </section>

        <!-- 主操作區 -->
        <div class="flex flex-col lg:flex-row gap-6">
            
            <!-- 控制面板 -->
            <aside class="lg:w-80 flex flex-col gap-4">
                <div class="bg-white rounded-2xl shadow-sm p-6 border border-slate-200">
                    <h2 class="text-[10px] font-black text-slate-400 mb-6 uppercase tracking-widest">互動操作</h2>
                    
                    <div class="space-y-8">
                        <div>
                            <label class="block text-sm font-bold text-slate-700 mb-4">1. 選擇平分成幾份：</label>
                            <div id="split-buttons" class="grid grid-cols-4 gap-2"></div>
                        </div>

                        <div class="p-5 bg-purple-50 rounded-2xl border border-purple-100">
                            <label class="block text-sm font-bold text-purple-800 mb-3">2. 點擊圖形填色：</label>
                            <div class="flex items-center justify-between">
                                <span class="text-xs text-purple-600 font-bold">目前選取</span>
                                <div class="flex items-baseline gap-1">
                                    <span id="selected-count-display" class="text-4xl font-black text-purple-600">0</span>
                                    <span class="text-sm text-purple-600 font-bold">份</span>
                                </div>
                            </div>
                        </div>
                    </div>

                    <button onclick="checkAnswer()" class="w-full mt-8 py-4 bg-slate-900 hover:bg-black text-white font-black rounded-xl transition-all shadow-lg active:scale-95">
                        檢查操作結果
                    </button>
                </div>
            </aside>

            <!-- 畫布區 -->
            <section class="flex-1 bg-white rounded-3xl shadow-sm border border-slate-200 p-4 flex flex-col items-center min-h-[500px]">
                <div class="svg-container">
                    <svg id="math-svg" width="650" height="380" viewBox="0 0 650 380" class="drop-shadow-sm"></svg>
                </div>

                <!-- 算式聯覺提示 -->
                <div id="calculation-preview" class="mt-4 px-8 py-5 bg-slate-50 border-2 border-slate-200 text-slate-900 rounded-2xl font-bold text-lg opacity-0 transition-all duration-500 flex flex-wrap justify-center items-center gap-4">
                </div>
            </section>
        </div>
    </main>

    <!-- 結果與設定 Modal (保持原樣) -->
    <div id="result-modal" class="fixed inset-0 modal-overlay z-[100] hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-3xl shadow-2xl max-w-sm w-full p-8 text-center transform transition-all scale-95 opacity-0" id="result-content">
            <div id="result-icon" class="text-6xl mb-4"></div>
            <h3 id="result-title" class="text-2xl font-black mb-2"></h3>
            <p id="result-text" class="text-slate-600 font-bold mb-8"></p>
            <button onclick="closeResult()" class="w-full py-4 bg-slate-900 text-white font-black rounded-xl hover:bg-black transition-colors">確定</button>
        </div>
    </div>

    <div id="settings-modal" class="fixed inset-0 modal-overlay z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white rounded-3xl shadow-2xl max-w-sm w-full p-8 transform transition-all scale-95 opacity-0" id="modal-content">
            <h3 class="text-xl font-bold text-slate-800 mb-8 text-center tracking-tight">自訂任務數值</h3>
            <div class="flex items-center justify-center gap-6 mb-10">
                <div class="flex flex-col items-center">
                    <label class="text-[10px] font-black text-slate-400 mb-2 uppercase tracking-tighter">被乘數</label>
                    <input type="number" id="input-total" value="10" class="w-16 h-16 text-center text-2xl font-black border-2 border-slate-100 rounded-xl focus:border-blue-500 outline-none">
                </div>
                <div class="text-2xl font-black text-slate-300 mt-6">×</div>
                <div class="flex flex-col items-center gap-2">
                    <input type="number" id="input-num" value="2" class="w-14 h-14 text-center text-xl font-black border-2 border-slate-100 rounded-xl focus:border-purple-500 outline-none text-purple-600">
                    <div class="w-16 h-1 bg-slate-800 rounded-full"></div>
                    <input type="number" id="input-den" value="5" class="w-14 h-14 text-center text-xl font-black border-2 border-slate-100 rounded-xl focus:border-emerald-500 outline-none text-emerald-600">
                </div>
            </div>
            <button onclick="applySettings()" class="w-full py-4 bg-slate-900 text-white font-black rounded-xl hover:bg-black">更新題目</button>
        </div>
    </div>

    <script>
        let config = { total: 10, targetNum: 2, targetDen: 5 };
        let state = { userSplits: 1, selectedIndices: [] };

        const svgNS = "http://www.w3.org/2000/svg";
        const svg = document.getElementById('math-svg');

        // 產生垂直分數 HTML 結構的輔助函式
        function getVerticalFracHTML(num, den, isSmall = false) {
            const sizeClass = isSmall ? "text-xs" : "text-2xl sm:text-3xl";
            return `
                <div class="frac-display ${sizeClass}">
                    <span class="top">${num}</span>
                    <span class="bottom">${den}</span>
                </div>
            `;
        }

        function init() {
            createSplitButtons();
            updateUI();
            render();
        }

        function createSplitButtons() {
            const container = document.getElementById('split-buttons');
            if (!container) return;
            container.innerHTML = '';
            for (let i = 1; i <= 12; i++) {
                const btn = document.createElement('button');
                btn.innerText = i;
                btn.className = `btn-split ${state.userSplits === i ? 'btn-split-active' : ''}`;
                btn.onclick = () => {
                    state.userSplits = i;
                    state.selectedIndices = [];
                    createSplitButtons();
                    updateUI();
                    render();
                };
                container.appendChild(btn);
            }
        }

        function updateUI() {
            // 更新任務引導區
            const guide = document.getElementById('guide-text');
            if (guide) {
                guide.innerHTML = `
                    <span>請問</span> 
                    <span class="text-blue-600 underline">${config.total}</span> 
                    <span>的</span> 
                    <span class="text-purple-600 mx-1">${getVerticalFracHTML(config.targetNum, config.targetDen)}</span> 
                    <span>倍是多少？</span>
                `;
            }
            
            // 更新迷你狀態標籤
            const miniFrac = document.getElementById('target-frac-mini');
            if (miniFrac) {
                miniFrac.innerHTML = getVerticalFracHTML(config.targetNum, config.targetDen, true);
            }

            const countDisplay = document.getElementById('selected-count-display');
            if (countDisplay) countDisplay.innerText = state.selectedIndices.length;
            
            updateCalcPreview();
        }

        function render() {
            if (!svg) return;
            svg.innerHTML = '';
            const w = 600, h = 320, x0 = 25, y0 = 10;
            const gap = 6; 

            const unitW = (w - (config.total - 1) * gap) / config.total;
            for(let i=0; i < config.total; i++) {
                const xPos = x0 + i * (unitW + gap);
                const uRect = drawRect(xPos, y0, unitW, h, "#f0f9ff", "#000000", 2);
                uRect.classList.add("base-unit-fill");
                svg.appendChild(uRect);
            }

            const rowH = h / state.userSplits;
            for(let i=0; i < state.userSplits; i++) {
                const interactRect = drawRect(x0, y0 + i*rowH, w, rowH, "transparent", "none", 0);
                interactRect.setAttribute("class", "cursor-pointer hover:fill-blue-500/5 transition-colors");
                
                if(state.selectedIndices.includes(i)) {
                    const fillRect = drawRect(x0, y0 + i*rowH, w, rowH, "#a855f7", "none", 0);
                    fillRect.setAttribute("style", "pointer-events: none; fill-opacity: 0.6;");
                    svg.appendChild(fillRect);
                }

                interactRect.onclick = () => {
                    const pos = state.selectedIndices.indexOf(i);
                    if(pos === -1) state.selectedIndices.push(i);
                    else state.selectedIndices.splice(pos, 1);
                    updateUI();
                    render();
                };
                svg.appendChild(interactRect);
            }

            for(let i=1; i < state.userSplits; i++) {
                const line = drawLine(x0, y0 + i*rowH, x0 + w, y0 + i*rowH, "fraction-split-line");
                svg.appendChild(line);
            }
        }

        // 修改算式提示區的分數格式
        function updateCalcPreview() {
            const preview = document.getElementById('calculation-preview');
            if (!preview) return;
            if(state.selectedIndices.length > 0) {
                preview.classList.replace('opacity-0', 'opacity-100');
                const perPartHTML = getVerticalFracHTML(config.total, state.userSplits, true);
                const totalResultHTML = getVerticalFracHTML(config.total * state.selectedIndices.length, state.userSplits, true);
                preview.innerHTML = `
                    <div class="flex items-center gap-1">每一份是 ${perPartHTML} 格</div>
                    <div class="w-px h-6 bg-slate-300 mx-2"></div>
                    <div class="flex items-center gap-1">選了 ${state.selectedIndices.length} 份 = 共 ${totalResultHTML} 格</div>
                `;
            } else {
                preview.classList.replace('opacity-100', 'opacity-0');
            }
        }

        function checkAnswer() {
            const targetValue = (config.total * config.targetNum) / config.targetDen;
            const currentUserValue = (config.total / state.userSplits) * state.selectedIndices.length;
            if (state.selectedIndices.length === 0) {
                showModal("🤔", "尚未選取", "請點擊圖形區域來選取平分後的份數。");
                return;
            }
            if (Math.abs(currentUserValue - targetValue) < 0.01) {
                showModal("🎉", "正確！", `太棒了！這正是 ${config.total} 的 ${config.targetNum}/${config.targetDen} 倍。`);
            } else {
                showModal("❌", "再試試看", "選取的範圍與目標不符，檢查看看平分的份數或選取的數量。");
            }
        }

        function showModal(emoji, t, txt) {
            const modal = document.getElementById('result-modal');
            const content = document.getElementById('result-content');
            if (!modal || !content) return;
            document.getElementById('result-icon').innerText = emoji;
            document.getElementById('result-title').innerText = t;
            document.getElementById('result-text').innerText = txt;
            modal.classList.remove('hidden');
            setTimeout(() => content.classList.remove('scale-95', 'opacity-0'), 10);
        }

        function closeResult() {
            const modal = document.getElementById('result-modal');
            const content = document.getElementById('result-content');
            if (!content || !modal) return;
            content.classList.add('scale-95', 'opacity-0');
            setTimeout(() => modal.classList.add('hidden'), 200);
        }

        function drawRect(x, y, w, h, fill, stroke, sw) {
            const r = document.createElementNS(svgNS, "rect");
            r.setAttribute("x", x); r.setAttribute("y", y);
            r.setAttribute("width", w); r.setAttribute("height", h);
            r.setAttribute("fill", fill);
            if(stroke !== "none") {
                r.setAttribute("stroke", stroke);
                r.setAttribute("stroke-width", sw);
            }
            return r;
        }

        function drawLine(x1, y1, x2, y2, className) {
            const l = document.createElementNS(svgNS, "line");
            l.setAttribute("x1", x1); l.setAttribute("y1", y1);
            l.setAttribute("x2", x2); l.setAttribute("y2", y2);
            l.setAttribute("class", className);
            return l;
        }

        function resetApp() { state.userSplits = 1; state.selectedIndices = []; init(); }
        function openSettings() {
            const modal = document.getElementById('settings-modal');
            const content = document.getElementById('modal-content');
            if (!modal || !content) return;
            modal.classList.remove('hidden');
            setTimeout(() => content.classList.remove('scale-95', 'opacity-0'), 10);
        }
        function applySettings() {
            config.total = parseInt(document.getElementById('input-total').value) || 10;
            config.targetNum = parseInt(document.getElementById('input-num').value) || 2;
            config.targetDen = parseInt(document.getElementById('input-den').value) || 5;
            const content = document.getElementById('modal-content');
            if (content) content.classList.add('scale-95', 'opacity-0');
            setTimeout(() => { 
                const modal = document.getElementById('settings-modal');
                if (modal) modal.classList.add('hidden'); 
                resetApp(); 
            }, 200);
        }

        window.onload = init;
    </script>
</body>
</html>
