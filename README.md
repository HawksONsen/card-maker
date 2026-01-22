<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>AIなんでもカードメーカー V8.7</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Cinzel:wght@700;900&family=Noto+Sans+JP:wght@400;700;900&display=swap');
        body { font-family: 'Noto Sans JP', sans-serif; background: #0a0a0c; color: #e6edf3; overflow-x: hidden; }
        
        /* カード本体のデザイン */
        .card-base { 
            position: relative; width: 340px; min-height: 540px; padding: 20px; 
            border-radius: 2rem; background: #1a1b1e; 
            box-sizing: border-box; overflow: hidden;
            display: flex; flex-direction: column;
            box-shadow: 0 20px 50px rgba(0,0,0,0.8);
            border: 2px solid #30363d;
            margin: 0 auto;
        }

        /* レアリティ別ベースカラー */
        .card-N { background: linear-gradient(135deg, #1a1b1e, #2d333b); }
        .card-R { background: linear-gradient(135deg, #0d1b2a, #1b263b); border-color: #3b82f6; }
        .card-SR { background: linear-gradient(135deg, #1d1135, #240046); border-color: #a855f7; }
        .card-SSR { background: linear-gradient(135deg, #2d2006, #432818); border-color: #eab308; }
        .card-UR { background: linear-gradient(135deg, #2d0b0b, #4a0404); border-color: #ef4444; }
        .card-LR { background: linear-gradient(135deg, #1a1a1a, #2c3e50, #000); border-color: #ffffff; }

        /* 豪華な装飾枠（全周フレーム） */
        .ornament-frame {
            position: absolute; inset: 6px;
            pointer-events: none; z-index: 10;
            border: 2px solid transparent;
            border-radius: 1.8rem;
        }
        .card-SSR .ornament-frame { border: 4px solid #eab308; box-shadow: inset 0 0 15px #eab30866; border-style: double; }
        .card-UR .ornament-frame { border: 4px solid #ef4444; box-shadow: inset 0 0 20px #ef444466; border-style: double; }
        .card-LR .ornament-frame { 
            border: 5px solid; 
            border-image: linear-gradient(45deg, #ffd700, #fff, #ffd700, #ff8c00) 1;
            box-shadow: 0 0 30px rgba(255,255,255,0.3);
        }

        /* 画像エリア */
        .img-container { 
            position: relative; width: 100%; height: 215px; 
            border-radius: 1rem; overflow: hidden; margin-bottom: 12px;
            border: 1px solid rgba(255,255,255,0.1);
            z-index: 5;
        }
        .img-main { width: 100%; height: 100%; object-fit: cover; }

        /* テキストプレート */
        .content-plate {
            background: rgba(0, 0, 0, 0.6);
            backdrop-filter: blur(8px);
            border-radius: 1rem;
            padding: 10px;
            border: 1px solid rgba(255,255,255,0.1);
            z-index: 5;
        }

        /* ステータス */
        .stat-box { background: rgba(255,255,255,0.07); border-radius: 0.5rem; padding: 4px; text-align: center; border: 1px solid rgba(255,255,255,0.1); }
        .stat-label { font-size: 8px; font-weight: 900; color: #9ca3af; margin-bottom: 1px; }
        .stat-value { font-size: 11px; font-weight: 900; color: #fff; }

        .loader { width: 48px; height: 48px; border: 5px solid #FFF; border-bottom-color: #3b82f6; border-radius: 50%; display: inline-block; animation: rotation 1s linear infinite; }
        @keyframes rotation { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        
        .step-container { display: none; }
        .step-container.active { display: block; animation: fadeIn 0.4s ease-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        .config-panel {
            position: fixed; top: 1rem; right: 1rem; z-index: 200;
        }
    </style>
</head>
<body class="p-4">

<!-- APIキー設定ボタン -->
<div class="config-panel">
    <button onclick="$('keyModal').classList.remove('hidden')" class="bg-slate-800/80 p-3 rounded-full border border-slate-700 hover:bg-slate-700 transition-all shadow-xl">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-blue-400" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M11.49 3.17c-.38-1.56-2.6-1.56-2.98 0a1.532 1.532 0 01-2.286.948c-1.372-.836-2.942.734-2.106 2.106.54.886.061 2.042-.947 2.287-1.561.379-1.561 2.6 0 2.978a1.532 1.532 0 01.947 2.287c-.836 1.372.734 2.942 2.106 2.106a1.532 1.532 0 012.287.947c.379 1.561 2.6 1.561 2.978 0a1.533 1.533 0 012.287-.947c1.372.836 2.942-.734 2.106-2.106a1.533 1.533 0 01.947-2.287c1.561-.379 1.561-2.6 0-2.978a1.532 1.532 0 01-.947-2.287c.836-1.372-.734-2.942-2.106-2.106a1.532 1.532 0 01-2.287-.947zM10 13a3 3 0 100-6 3 3 0 000 6z" clip-rule="evenodd" />
        </svg>
    </button>
</div>

<!-- APIキー入力モーダル -->
<div id="keyModal" class="fixed inset-0 bg-black/90 z-[300] hidden flex items-center justify-center p-6">
    <div class="bg-slate-900 border border-slate-800 p-8 rounded-[2.5rem] w-full max-w-sm space-y-4 shadow-2xl">
        <h3 class="font-black text-xl text-blue-500 italic">API設定</h3>
        <p class="text-xs text-slate-400">Gemini APIキーを入力してください。<br>キーはブラウザにのみ保存されます。</p>
        <input type="password" id="apiKeyInput" placeholder="AIキーを入力..." class="w-full bg-slate-950 p-4 rounded-2xl border border-slate-800 text-sm text-white focus:ring-2 focus:ring-blue-500 outline-none">
        <div class="flex gap-2">
            <button onclick="saveApiKey()" class="flex-1 py-4 bg-blue-600 rounded-2xl font-black">保存する</button>
            <button onclick="$('keyModal').classList.add('hidden')" class="flex-1 py-4 bg-slate-800 rounded-2xl font-bold">閉じる</button>
        </div>
        <p class="text-[10px] text-center text-slate-600">Google AI Studioで無料で取得可能です</p>
    </div>
</div>

<div class="max-w-2xl mx-auto pt-10 pb-20">
    <div class="mb-10 text-center">
        <h1 class="text-4xl font-black text-blue-500 tracking-tighter italic drop-shadow-lg">AIカードメーカー <span class="text-white">V8.7</span></h1>
        <p class="text-[10px] text-slate-500 tracking-[0.4em] uppercase mt-2">Professional Card Forge Edition</p>
    </div>

    <!-- ステップ1: タイプ選択 -->
    <div id="step1" class="step-container active space-y-4">
        <h2 class="text-lg font-bold text-center text-slate-300">魂の形態を選択</h2>
        <div class="grid grid-cols-1 gap-4">
            <button onclick="setCardType('ファイター/サポーター')" class="p-8 bg-slate-900/50 border border-slate-800 rounded-[2.5rem] hover:bg-blue-900/20 hover:border-blue-500 transition-all text-left group shadow-lg">
                <span class="block font-black text-2xl text-blue-100 group-hover:text-blue-400">ユニット</span>
                <span class="text-xs text-slate-500 font-bold mt-1 block">キャラクターやクリーチャーのカード</span>
            </button>
            <button onclick="setCardType('スキル/装備/アイテム')" class="p-8 bg-slate-900/50 border border-slate-800 rounded-[2.5rem] hover:bg-blue-900/20 hover:border-blue-500 transition-all text-left group shadow-lg">
                <span class="block font-black text-2xl text-blue-100 group-hover:text-blue-400">サポート</span>
                <span class="text-xs text-slate-500 font-bold mt-1 block">魔法・武器・道具などのカード</span>
            </button>
        </div>
    </div>

    <!-- ステップ2: 画像生成 -->
    <div id="step2" class="step-container space-y-6">
        <h2 class="text-lg font-bold text-center text-slate-300">イラストの錬成</h2>
        <div id="visualInputArea" class="space-y-4">
            <div class="bg-slate-950 p-10 rounded-[2.5rem] border-2 border-slate-800 border-dashed text-center hover:border-blue-500/50 transition-all">
                <input type="file" id="imageInput" accept="image/*" class="w-full text-sm text-slate-500 cursor-pointer">
                <p class="text-[10px] text-slate-600 mt-2 font-bold uppercase">PNG, JPG 対応</p>
            </div>
            <textarea id="visualComment" placeholder="イラストのテーマを入力（例：氷の魔法を操る少女、荒廃した未来の都市など）" class="w-full bg-slate-900 p-6 rounded-[1.5rem] border border-slate-800 h-32 resize-none focus:ring-2 focus:ring-blue-500 outline-none text-sm text-white placeholder-slate-600"></textarea>
            <button onclick="generateVisual()" class="w-full py-5 bg-blue-600 rounded-[1.5rem] font-black text-xl shadow-xl shadow-blue-600/20 active:scale-95 transition-all">イラストを錬成</button>
        </div>
        <div id="visualPreviewArea" class="hidden space-y-6 text-center">
            <div class="flex justify-center">
                <img id="tempImage" class="w-80 h-80 object-cover rounded-[2rem] border-2 border-slate-700 shadow-2xl">
            </div>
            <div class="flex gap-3 max-w-sm mx-auto">
                <button onclick="resetVisualStep()" class="flex-1 py-4 bg-slate-800 rounded-[1.2rem] font-bold hover:bg-slate-700 transition-all">やり直す</button>
                <button onclick="changeStep(3)" class="flex-1 py-4 bg-blue-600 rounded-[1.2rem] font-black hover:bg-blue-500 transition-all">次へ</button>
            </div>
        </div>
    </div>

    <!-- ステップ3: テキスト設定 -->
    <div id="step3" class="step-container space-y-6">
        <h2 class="text-lg font-bold text-center text-slate-300">言霊の解析</h2>
        <div id="textInputArea" class="space-y-4">
            <div class="flex gap-4 border-b border-slate-800 text-xs font-black mb-4">
                <button id="tabAuto" onclick="switchTextMode('auto')" class="tab-btn active pb-3 px-2">AIにおまかせ</button>
                <button id="tabCustom" onclick="switchTextMode('custom')" class="tab-btn pb-3 px-2">自分で命名</button>
            </div>

            <div id="customInputs" class="hidden space-y-3 p-6 bg-slate-900/30 rounded-[1.5rem] border border-slate-800">
                <div class="grid grid-cols-2 gap-3">
                    <input type="text" id="inputTitle" placeholder="二つ名" class="bg-slate-950 p-4 rounded-xl border border-slate-800 text-sm text-white">
                    <input type="text" id="inputName" placeholder="名前" class="bg-slate-950 p-4 rounded-xl border border-slate-800 text-sm text-white">
                </div>
                <input type="text" id="inputSkill" placeholder="スキル名" class="w-full bg-slate-950 p-4 rounded-xl border border-slate-800 text-sm text-white">
            </div>

            <textarea id="textComment" placeholder="詳細な設定（性格、能力、武器など）" class="w-full bg-slate-900 p-6 rounded-[1.5rem] border border-slate-800 h-28 resize-none outline-none focus:ring-2 focus:ring-blue-500 text-sm text-white placeholder-slate-600"></textarea>
            <button onclick="generateText()" class="w-full py-5 bg-blue-600 rounded-[1.5rem] font-black text-xl shadow-xl active:scale-95 transition-all">テキストを刻む</button>
        </div>

        <div id="textPreviewArea" class="hidden space-y-6 bg-slate-900 p-8 rounded-[2.5rem] border border-slate-800 shadow-2xl">
            <div class="space-y-4 text-left">
                <div class="border-l-4 border-blue-500 pl-4">
                    <p id="previewName" class="font-black text-2xl text-white tracking-tight"></p>
                    <p id="previewSkill" class="text-sm font-bold text-blue-400 mt-2"></p>
                </div>
                <p id="previewFlavor" class="text-xs text-slate-400 italic leading-relaxed"></p>
            </div>
            <div class="flex gap-3">
                <button onclick="resetTextStep()" class="flex-1 py-4 bg-slate-800 rounded-[1.2rem] font-bold">修正</button>
                <button onclick="ascendCard()" class="flex-1 py-4 bg-gradient-to-r from-blue-600 to-indigo-600 rounded-[1.2rem] font-black text-white">カードを完成させる</button>
            </div>
        </div>
    </div>

    <!-- ステップ4: 完了 -->
    <div id="step4" class="step-container space-y-8 text-center">
        <div class="space-y-2">
            <h2 class="text-2xl font-black text-blue-500 italic">FORGE COMPLETE</h2>
            <p class="text-[10px] text-slate-500 font-bold uppercase tracking-widest">カードが正常に錬成されました</p>
        </div>
        
        <div class="flex justify-center py-4">
             <div id="cardTarget" class="scale-100 sm:scale-110"></div>
        </div>

        <div class="grid grid-cols-2 gap-3 max-w-sm mx-auto pt-4">
            <button onclick="downloadCard()" class="col-span-2 py-6 bg-green-600 hover:bg-green-500 rounded-[1.5rem] font-black text-xl shadow-xl shadow-green-900/20 transition-all active:scale-95">画像を保存</button>
            <button onclick="location.reload()" class="col-span-2 py-4 bg-slate-900 border border-slate-800 rounded-[1.2rem] font-bold text-slate-400 hover:text-white transition-all">最初から作る</button>
        </div>
    </div>

    <!-- ローダー -->
    <div id="loadingOverlay" class="fixed inset-0 bg-black/95 z-[1000] hidden flex flex-col items-center justify-center p-6 text-center">
        <div class="loader mb-8 shadow-2xl shadow-blue-500/50"></div>
        <p id="loadingMsg" class="font-black text-blue-500 animate-pulse text-lg italic tracking-tighter">AIが思考中...</p>
    </div>
</div>

<script>
    let currentApiKey = localStorage.getItem('gemini_api_key') || "";
    let appState = {
        cardType: '',
        originalBase64: '',
        processedBase64: '',
        textData: null,
        finalData: null,
        textMode: 'auto'
    };

    const $ = id => document.getElementById(id);

    function saveApiKey() {
        const key = $('apiKeyInput').value.trim();
        if(!key) return alert("キーを入力してください");
        currentApiKey = key;
        localStorage.setItem('gemini_api_key', key);
        $('keyModal').classList.add('hidden');
        alert("設定を保存しました");
    }

    function checkApiKey() {
        if(!currentApiKey) {
            $('keyModal').classList.remove('hidden');
            return false;
        }
        return true;
    }

    function changeStep(step) {
        document.querySelectorAll('.step-container').forEach(el => el.classList.remove('active'));
        $(`step${step}`).classList.add('active');
        window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    function setCardType(type) {
        appState.cardType = type;
        changeStep(2);
    }

    function switchTextMode(mode) {
        appState.textMode = mode;
        $('tabAuto').classList.toggle('active', mode === 'auto');
        $('tabCustom').classList.toggle('active', mode === 'custom');
        $('customInputs').classList.toggle('hidden', mode === 'auto');
    }

    async function generateVisual() {
        if(!checkApiKey()) return;
        const file = $('imageInput').files[0];
        if(!file && !appState.originalBase64) return alert("画像を選択してください");
        
        $('loadingOverlay').classList.remove('hidden');
        $('loadingMsg').innerText = "イラストを魔力錬成中...";

        const runGen = async (base64) => {
            try {
                const comment = $('visualComment').value;
                const prompt = `TCG Card Illustration of a ${appState.cardType}. Theme: ${comment || 'Fantasy'}. 
                Masterpiece, cinematic lighting, vivid colors. NO UI, NO TEXT.`;

                const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${currentApiKey}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: prompt }, { inlineData: { mimeType: "image/png", data: base64 } }] }],
                        generationConfig: { responseModalities: ['TEXT', 'IMAGE'] }
                    })
                });
                
                if(!response.ok) throw new Error();
                
                const data = await response.json();
                appState.processedBase64 = data.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data || base64;
                $('tempImage').src = `data:image/png;base64,${appState.processedBase64}`;
                $('visualInputArea').classList.add('hidden');
                $('visualPreviewArea').classList.remove('hidden');
            } catch (e) { alert("APIエラーが発生しました。キーが正しいか確認してください。"); }
            finally { $('loadingOverlay').classList.add('hidden'); }
        };

        if(file) {
            const reader = new FileReader();
            reader.onload = e => {
                appState.originalBase64 = e.target.result.split(',')[1];
                runGen(appState.originalBase64);
            };
            reader.readAsDataURL(file);
        } else { runGen(appState.originalBase64); }
    }

    async function generateText() {
        if(!checkApiKey()) return;
        $('loadingOverlay').classList.remove('hidden');
        $('loadingMsg').innerText = "言霊を解析中...";
        try {
            const comment = $('textComment').value;
            const cName = $('inputName').value;
            const cTitle = $('inputTitle').value;
            const cSkill = $('inputSkill').value;

            const prompt = `Create TCG card text in Japanese.
            ${appState.textMode === 'custom' ? `Locked values: Name=${cName}, Title=${cTitle}, Skill=${cSkill}` : ''}
            Context: ${comment}, Type: ${appState.cardType}.`;

            const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${currentApiKey}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    contents: [{ parts: [{ text: prompt }, { inlineData: { mimeType: "image/png", data: appState.processedBase64 } }] }],
                    generationConfig: { 
                        responseMimeType: "application/json",
                        responseSchema: {
                            type: "OBJECT",
                            properties: {
                                name: { type: "STRING" }, title: { type: "STRING" }, skill: { type: "STRING" }, flavor: { type: "STRING" }
                            },
                            required: ["name", "title", "skill", "flavor"]
                        }
                    }
                })
            });
            const data = await response.json();
            appState.textData = JSON.parse(data.candidates[0].content.parts[0].text);
            $('previewName').innerText = `${appState.textData.title} ${appState.textData.name}`;
            $('previewSkill').innerText = appState.textData.skill;
            $('previewFlavor').innerText = appState.textData.flavor;
            $('textInputArea').classList.add('hidden');
            $('textPreviewArea').classList.remove('hidden');
        } catch (e) { alert("テキスト生成エラー"); }
        finally { $('loadingOverlay').classList.add('hidden'); }
    }

    async function ascendCard() {
        if(!checkApiKey()) return;
        $('loadingOverlay').classList.remove('hidden');
        $('loadingMsg').innerText = "運命の昇格鑑定...";
        try {
            const statsRes = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${currentApiKey}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    contents: [{ parts: [{ text: "Analyze 6 stats (100-300) and element (icon only)." }, { inlineData: { mimeType: "image/png", data: appState.processedBase64 } }] }],
                    generationConfig: { 
                        responseMimeType: "application/json",
                        responseSchema: {
                            type: "OBJECT",
                            properties: {
                                stats: { type: "ARRAY", items: { type: "NUMBER" }, minItems: 6, maxItems: 6 },
                                element: { type: "STRING", enum: ["🔥", "💧", "🍃", "⚡", "🌑", "☀️"] }
                            }
                        }
                    }
                })
            });
            const statsData = JSON.parse((await statsRes.json()).candidates[0].content.parts[0].text);
            
            const total = statsData.stats.reduce((a, b) => a + b, 0);
            let rarity = "N";
            let multiplier = 1.0;

            if(total > 1500) { rarity = "LR"; multiplier = 3.5; }
            else if(total > 1200) { rarity = "UR"; multiplier = 2.8; }
            else if(total > 900) { rarity = "SSR"; multiplier = 2.2; }
            else if(total > 600) { rarity = "SR"; multiplier = 1.6; }
            else if(total > 300) { rarity = "R"; multiplier = 1.2; }
            
            statsData.stats = statsData.stats.map(v => Math.floor(v * multiplier));

            if(["SSR", "UR", "LR"].includes(rarity)) {
                const effectPrompt = `Enhance illustration with divine ${rarity} effects.`;
                const effectRes = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${currentApiKey}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: effectPrompt }, { inlineData: { mimeType: "image/png", data: appState.processedBase64 } }] }],
                        generationConfig: { responseModalities: ['TEXT', 'IMAGE'] }
                    })
                });
                const effectData = await effectRes.json();
                const newImg = effectData.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;
                if(newImg) appState.processedBase64 = newImg;
            }

            appState.finalData = { ...appState.textData, ...statsData, rarity };
            renderFinalCard();
            changeStep(4);
        } catch (e) { alert("昇格プロセスエラー"); }
        finally { $('loadingOverlay').classList.add('hidden'); }
    }

    function renderFinalCard() {
        const d = appState.finalData;
        const html = `
            <div id="cardFrame" class="card-base card-${d.rarity}">
                <div class="ornament-frame"></div>
                <div class="flex justify-between items-center px-1 mb-2 z-10">
                    <span class="text-[9px] font-black text-blue-400 uppercase tracking-widest">${appState.cardType}</span>
                    <span class="text-yellow-400 italic text-base font-black font-serif">${d.rarity}</span>
                </div>
                <div class="flex items-center gap-2 mb-3 px-1 z-10">
                    <div class="text-3xl drop-shadow-lg">${d.element}</div>
                    <div class="flex-grow">
                        <p class="text-[7px] text-blue-300 font-bold uppercase leading-none mb-1 opacity-70">${d.title}</p>
                        <h2 class="text-lg font-black text-white leading-tight">${d.name}</h2>
                    </div>
                </div>
                <div class="img-container">
                    <img src="data:image/png;base64,${appState.processedBase64}" class="img-main">
                    <div class="absolute inset-0 bg-gradient-to-t from-black/50 to-transparent"></div>
                </div>
                <div class="content-plate mt-1 flex-grow flex flex-col">
                    <div class="grid grid-cols-3 gap-1.5 mb-3">
                        ${['HP','ATK','DEF','SPD','HIT','LUK'].map((label, idx) => `
                            <div class="stat-box"><p class="stat-label">${label}</p><p class="stat-value">${d.stats[idx]}</p></div>
                        `).join('')}
                    </div>
                    <div class="bg-blue-600/20 border-l-2 border-blue-400 p-3 rounded-r-lg mb-2">
                        <p class="text-[10px] font-black text-blue-50 leading-snug">${d.skill}</p>
                    </div>
                    <p class="text-[8px] text-slate-300 italic px-1 leading-relaxed opacity-90 line-clamp-3">${d.flavor}</p>
                </div>
            </div>
        `;
        $('cardTarget').innerHTML = html;
    }

    function resetVisualStep() { $('visualInputArea').classList.remove('hidden'); $('visualPreviewArea').classList.add('hidden'); }
    function resetTextStep() { $('textInputArea').classList.remove('hidden'); $('textPreviewArea').classList.add('hidden'); }

    function downloadCard() {
        const target = $('cardTarget').firstElementChild;
        html2canvas(target, { scale: 3, backgroundColor: null, useCORS: true }).then(canvas => {
            const a = document.createElement('a');
            a.download = `AI_CARD_${appState.finalData.rarity}.png`;
            a.href = canvas.toDataURL();
            a.click();
        });
    }
</script>
</body>
</html>
