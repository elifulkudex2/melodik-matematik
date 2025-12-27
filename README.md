<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Melodik Matematik | Eğlenceli Öğrenme</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&display=swap" rel="stylesheet">
    <!-- Müzikal geri bildirimler için Tone.js kütüphanesi -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    <style>
        :root {
            --color-primary: #58CC02;
            --color-secondary: #1CB0F6;
            --color-dark: #3c3c3c;
            --color-progress-bg: #e5e5e5;
        }
        body { 
            font-family: 'Inter', sans-serif; 
            background-color: var(--color-secondary); 
            user-select: none;
            touch-action: manipulation;
        }
        .btn-action {
            background-color: var(--color-primary);
            color: white;
            border-bottom: 5px solid #47A800;
            transition: all 0.1s ease-in-out;
        }
        .btn-action:hover:not(:disabled) { background-color: #63d602; border-bottom: 3px solid #47A800; transform: translateY(-1px); }
        .btn-action:active:not(:disabled) { transform: translateY(2px); border-bottom: 2px solid #47A800; }
        .btn-unit {
            background-color: white;
            color: var(--color-dark);
            border: 2px solid #ddd;
            transition: all 0.2s;
        }
        .btn-unit:hover { border-color: var(--color-secondary); box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1); }
        .progress-bar { height: 12px; background-color: var(--color-progress-bg); border-radius: 9999px; overflow: hidden; }
        .progress-fill { height: 100%; background-color: var(--color-primary); transition: width 0.5s ease-out; }
        .music-card {
            background: linear-gradient(135deg, #ffffff 0%, #f0f9ff 100%);
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1);
        }
        /* Yükleme simgesi animasyonu */
        .spinner {
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-top: 4px solid white;
            border-radius: 50%;
            width: 24px;
            height: 24px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="min-h-screen flex flex-col items-center justify-start py-8 px-4">

    <div id="app-container" class="w-full max-w-lg mx-auto flex flex-col space-y-6">
        <!-- Üst Bilgi Alanı -->
        <header class="flex items-center space-x-4">
            <button id="back-button" onclick="showUnitSelection()" class="text-white hover:opacity-80 transition hidden">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="3">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M15 19l-7-7 7-7" />
                </svg>
            </button>
            <h1 id="main-title" class="text-3xl font-extrabold text-white flex-grow text-center tracking-tight">Melodik Matematik</h1>
            <div id="progress-container" class="flex-grow hidden">
                <div class="progress-bar"><div id="progress-fill" class="progress-fill" style="width: 0%;"></div></div>
            </div>
        </header>

        <!-- Ana İçerik Kartı -->
        <main id="main-content" class="bg-white rounded-3xl shadow-2xl p-6 md:p-8 min-h-[450px] flex flex-col">
            
            <!-- Ünite Seçim Ekranı -->
            <div id="unit-selection-screen" class="space-y-6">
                <div class="text-center space-y-2">
                    <h2 class="text-2xl font-black text-gray-800">Hoş Geldin!</h2>
                    <p class="text-gray-500 font-medium text-sm md:text-base">Öğrenmek istediğin üniteyi seçerek başla.</p>
                </div>
                <div id="unit-list" class="space-y-4"></div>
            </div>

            <!-- Şarkı ve Eğitim Ekranı -->
            <div id="song-screen" class="hidden flex flex-col items-center space-y-6 flex-grow">
                <div class="music-card p-6 rounded-2xl border-2 border-blue-100 w-full flex-grow flex flex-col justify-center">
                    <span id="song-badge" class="bg-blue-100 text-blue-600 px-3 py-1 rounded-full text-xs font-bold w-fit mb-4 mx-auto">EĞİTİM ŞARKISI</span>
                    <h2 id="song-unit-title" class="text-2xl font-black text-gray-800 mb-4 text-center"></h2>
                    <div id="song-lyrics" class="italic text-gray-600 leading-relaxed text-base md:text-lg whitespace-pre-line mb-8 text-center bg-white p-4 rounded-xl shadow-inner border border-gray-50"></div>
                    
                    <button id="play-song-btn" onclick="handleSongPlay()" class="btn-action w-full py-4 rounded-2xl flex items-center justify-center space-x-3 shadow-md">
                        <span id="song-btn-icon">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="currentColor" viewBox="0 0 20 20">
                                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM9.555 7.168A1 1 0 008 8v4a1 1 0 001.555.832l3-2a1 1 0 000-1.664l-3-2z" clip-rule="evenodd" />
                            </svg>
                        </span>
                        <span id="song-btn-text" class="font-bold text-lg">Şarkıyı Dinle</span>
                    </button>
                </div>

                <button id="start-questions-btn" onclick="startQuestions()" class="btn-action w-full py-5 rounded-2xl text-xl font-black hidden animate-bounce shadow-lg">
                    HADİ BAŞLAYALIM!
                </button>
            </div>

            <!-- Soru Ekranı -->
            <div id="question-screen" class="hidden flex flex-col items-center justify-center space-y-8 flex-grow">
                <div class="text-center">
                    <p class="text-sm font-bold text-blue-500 uppercase tracking-widest mb-2">Dikkatle Dinle</p>
                    <h3 class="text-gray-400 text-sm italic">Soruyu duymak için butona bas</h3>
                </div>
                
                <button id="play-audio-btn" onclick="playQuestionAudio()" class="bg-white border-4 border-blue-400 text-blue-500 rounded-full shadow-xl w-32 h-32 md:w-36 md:h-36 flex items-center justify-center hover:scale-105 active:scale-95 transition-all">
                    <div id="audio-icon-wrapper">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 md:h-20 md:w-20" viewBox="0 0 20 20" fill="currentColor">
                            <path d="M9.383 3.076A1 1 0 0110 4v12a1 1 0 01-1.707.707L4.586 13H2a1 1 0 01-1-1V8a1 1 0 011-1h2.586l3.707-3.707a1 1 0 011.09-.217zM14.657 5.343a1 1 0 010 1.414A5 5 0 0116 10a5 5 0 01-1.343 3.243 1 1 0 11-1.414-1.414A3 3 0 0014 10a3 3 0 00-.879-2.121 1 1 0 010-1.414z" />
                        </svg>
                    </div>
                    <div id="audio-loading-spinner" class="hidden spinner !border-blue-500 !border-t-transparent !w-12 !h-12"></div>
                </button>

                <div class="w-full space-y-2">
                    <label class="text-xs font-bold text-gray-400 ml-2 uppercase">Cevabın</label>
                    <input type="number" id="user-answer" class="w-full p-4 md:p-5 text-center border-4 border-gray-100 rounded-3xl focus:border-blue-400 outline-none text-4xl md:text-5xl font-black text-gray-800 transition-colors" placeholder="?" inputmode="numeric">
                </div>
            </div>
        </main>

        <!-- Alt Kontrol ve Mesaj Alanı -->
        <footer id="footer-area" class="hidden space-y-4">
            <div id="message-box" class="p-5 rounded-2xl text-white font-bold text-center hidden transform transition-all duration-300 shadow-lg"></div>
            <button id="check-button" onclick="handleCheck()" class="btn-action w-full py-5 rounded-2xl text-2xl font-black uppercase opacity-50 cursor-not-allowed" disabled>Kontrol Et</button>
        </footer>
    </div>

<script>
    /**
     * YAPILANDIRMA
     * API anahtarınızı aşağıdaki değişkene eklemeyi unutmayın.
     */
    const apiKey = "";
    const TOTAL_QUESTIONS = 5; 
    let synth, currentUnit, questions = [], currentQuestionIndex = 0, isBusy = false;

    const UNITS = [
        { 
            id: 1, name: "Ünite 1: Toplama", type: "addition", color: "#47A800",
            song: "Bir elmaya bir ekle, iki olur heyecanla.\nSayılar dans eder, artar hepsi yan yana.\nToplamaktır adı, çoğalır her şey dünyada!\nHaydi şimdi sen de katıl, bu sihirli toplama!"
        },
        { 
            id: 2, name: "Ünite 2: Çıkarma", type: "subtraction", color: "#1CB0F6",
            song: "Beş kuş vardı dalda, biri uçtu uzağa.\nEksilir sayılar, veda eder arkadaşına.\nÇıkarma yapıyoruz, kalanları say sırayla.\nMatematik eğlence, her şey senin aklında!"
        },
        { 
            id: 3, name: "Ünite 3: Ritmik Sayma", type: "rhythmic", color: "#FFC700",
            song: "İki, dört, altı, sekiz; adım adım gideriz.\nBeşer beşer zıplayıp, bulutlara değeriz.\nRitmik saymak çok kolay, biz bu işi severiz.\nSayıların müziğiyle, her engeli geçeriz!"
        },
        { 
            id: 4, name: "Ünite 4: Örüntüler", type: "pattern", color: "#8A2BE2",
            song: "Bir kırmızı, bir mavi; takip eder birbirini.\nSayılar bir sıra tutmuş, bozma sakın düzeni.\nÖrüntüde gizlidir, matematiğin en güzeli.\nBak bakalım hangisi, tamamlar bu gizemi?"
        }
    ];

    // -- EKRAN YÖNETİMİ --

    function showUnitSelection() {
        currentUnit = null;
        questions = [];
        toggleScreen('unit-selection-screen');
        document.getElementById('back-button').classList.add('hidden');
        document.getElementById('progress-container').classList.add('hidden');
        document.getElementById('main-title').classList.remove('hidden');
        document.getElementById('footer-area').classList.add('hidden');
        renderUnitList();
    }

    function renderUnitList() {
        const list = document.getElementById('unit-list');
        list.innerHTML = '';
        UNITS.forEach(u => {
            const btn = document.createElement('button');
            btn.className = 'btn-unit w-full p-4 md:p-5 rounded-2xl text-lg md:text-xl font-bold flex items-center shadow-sm hover:translate-x-1 transition-transform';
            btn.innerHTML = `<div class="w-10 h-10 rounded-full flex items-center justify-center mr-4 text-white font-black shrink-0" style="background:${u.color}">${u.id}</div><span class="truncate">${u.name}</span>`;
            btn.onclick = () => loadSongScreen(u);
            list.appendChild(btn);
        });
    }

    function loadSongScreen(unit) {
        currentUnit = unit;
        toggleScreen('song-screen');
        document.getElementById('main-title').classList.add('hidden');
        document.getElementById('back-button').classList.remove('hidden');
        document.getElementById('song-unit-title').textContent = unit.name;
        document.getElementById('song-lyrics').textContent = unit.song;
        document.getElementById('start-questions-btn').classList.add('hidden');
    }

    // -- SES VE TTS (METİN OKUMA) İŞLEMLERİ --

    async function handleSongPlay() {
        if (isBusy) return;
        isBusy = true;
        const btnText = document.getElementById('song-btn-text');
        const btnIcon = document.getElementById('song-btn-icon');
        
        btnText.textContent = "Seslendiriliyor...";
        btnIcon.innerHTML = `<div class="spinner"></div>`;
        
        try {
            const url = await fetchTTS(currentUnit.song, "Say it like a very cheerful children's song narrator:");
            const audio = new Audio(url);
            audio.onended = () => {
                isBusy = false;
                btnText.textContent = "Tekrar Dinle";
                btnIcon.innerHTML = `<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" /></svg>`;
                document.getElementById('start-questions-btn').classList.remove('hidden');
            };
            await audio.play();
        } catch (e) {
            isBusy = false;
            btnText.textContent = "Şarkıyı Dinle";
            document.getElementById('start-questions-btn').classList.remove('hidden');
        }
    }

    async function playQuestionAudio() {
        if (isBusy || currentQuestionIndex >= TOTAL_QUESTIONS) return;
        isBusy = true;
        setAudioLoading(true);

        try {
            const q = questions[currentQuestionIndex];
            const url = await fetchTTS(q.text);
            const audio = new Audio(url);
            audio.onended = () => { isBusy = false; setAudioLoading(false); };
            await audio.play();
        } catch (e) {
            isBusy = false;
            setAudioLoading(false);
            console.error(e);
        }
    }

    async function fetchTTS(text, style = "") {
        if (!apiKey) {
            alert("Lütfen kodun içindeki apiKey değişkenine geçerli bir Gemini API anahtarı ekleyin!");
            throw new Error("API key missing");
        }
        const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${apiKey}`;
        const res = await fetch(url, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                contents: [{ parts: [{ text: style + " " + text }] }],
                generationConfig: { 
                    responseModalities: ["AUDIO"], 
                    speechConfig: { voiceConfig: { prebuiltVoiceConfig: { voiceName: "Kore" } } } 
                }
            })
        });
        const data = await res.json();
        const base64 = data.candidates[0].content.parts[0].inlineData.data;
        return pcmToWavUrl(base64);
    }

    // -- OYUN MANTIĞI --

    function startQuestions() {
        questions = generateQuestions(currentUnit.type, TOTAL_QUESTIONS);
        currentQuestionIndex = 0;
        toggleScreen('question-screen');
        document.getElementById('footer-area').classList.remove('hidden');
        document.getElementById('progress-container').classList.remove('hidden');
        updateQuestionUI();
    }

    function updateQuestionUI() {
        const progress = (currentQuestionIndex / TOTAL_QUESTIONS) * 100;
        document.getElementById('progress-fill').style.width = `${progress}%`;
        document.getElementById('user-answer').value = '';
        document.getElementById('message-box').classList.add('hidden');
        setCheckButtonState(false);
        
        if (currentQuestionIndex >= TOTAL_QUESTIONS) {
            finishUnit();
        }
    }

    function handleCheck() {
        if (isBusy) return;
        const input = document.getElementById('user-answer');
        const userVal = parseInt(input.value);
        const correctVal = questions[currentQuestionIndex].answer;
        const msgBox = document.getElementById('message-box');
        
        msgBox.classList.remove('hidden', 'bg-green-500', 'bg-red-500');
        
        if (userVal === correctVal) {
            msgBox.textContent = "Mükemmel! Doğru Cevap! 🌟";
            msgBox.classList.add('bg-green-500', 'block');
            playTone(true);
            currentQuestionIndex++;
            setTimeout(updateQuestionUI, 1500);
        } else {
            msgBox.textContent = "Bir daha dene, yapabilirsin! 🔄";
            msgBox.classList.add('bg-red-500', 'block');
            playTone(false);
            input.focus();
        }
    }

    function finishUnit() {
        const container = document.getElementById('question-screen');
        container.innerHTML = `
            <div class="text-center space-y-8 py-10">
                <div class="text-6xl mb-4">🏆</div>
                <h2 class="text-4xl font-black text-green-500">BİTTİ!</h2>
                <p class="text-xl text-gray-500 font-medium">${currentUnit.name} ünitesini başarıyla tamamladın.</p>
                <button onclick="location.reload()" class="btn-action px-12 py-5 rounded-2xl font-bold text-xl shadow-xl">YENİ ÜNİTEYE GEÇ</button>
            </div>
        `;
        document.getElementById('footer-area').classList.add('hidden');
        document.getElementById('progress-fill').style.width = "100%";
    }

    // -- YARDIMCI ARAÇLAR --

    function toggleScreen(screenId) {
        ['unit-selection-screen', 'song-screen', 'question-screen'].forEach(id => {
            document.getElementById(id).classList.toggle('hidden', id !== screenId);
        });
    }

    function setAudioLoading(loading) {
        document.getElementById('audio-icon-wrapper').classList.toggle('hidden', loading);
        document.getElementById('audio-loading-spinner').classList.toggle('hidden', !loading);
    }

    function setCheckButtonState(enabled) {
        const btn = document.getElementById('check-button');
        btn.disabled = !enabled;
        btn.classList.toggle('opacity-50', !enabled);
        btn.classList.toggle('cursor-not-allowed', !enabled);
    }

    function playTone(correct) {
        const now = Tone.now();
        if (correct) {
            synth.triggerAttackRelease("C5", "8n", now);
            synth.triggerAttackRelease("E5", "8n", now + 0.1);
            synth.triggerAttackRelease("G5", "8n", now + 0.2);
        } else {
            synth.triggerAttackRelease("F3", "4n", now);
        }
    }

    function generateQuestions(type, count) {
        let qs = [];
        for (let i = 0; i < count; i++) {
            let a = Math.floor(Math.random() * 8) + 2, b = Math.floor(Math.random() * 8) + 1;
            if (type === 'addition') qs.push({ text: `${a} artı ${b} kaç eder?`, answer: a + b });
            else if (type === 'subtraction') {
                if (a < b) [a, b] = [b, a];
                qs.push({ text: `${a} eksi ${b} kaçtır?`, answer: a - b });
            }
            else if (type === 'rhythmic') {
                let step = i % 2 === 0 ? 2 : 5, s = Math.floor(Math.random() * 5);
                qs.push({ text: `${s}, ${s + step}... Bu ritmik saymada sırada hangi sayı var?`, answer: s + step * 2 });
            }
            else {
                let s = Math.floor(Math.random() * 4), step = 2;
                qs.push({ text: `${s}, ${s + step}, ${s + step * 2}... Bu örüntüyü hangi sayı tamamlar?`, answer: s + step * 3 });
            }
        }
        return qs;
    }

    function pcmToWavUrl(base64) {
        const binary = atob(base64);
        const pcm = new Int16Array(new Uint8Array(binary.length).map((_, i) => binary.charCodeAt(i)).buffer);
        const header = new ArrayBuffer(44);
        const d = new DataView(header);
        const writeString = (o, s) => s.split('').forEach((c, i) => d.setUint8(o + i, c.charCodeAt(0)));
        writeString(0, 'RIFF'); d.setUint32(4, 36 + pcm.byteLength, true); writeString(8, 'WAVE');
        writeString(12, 'fmt '); d.setUint32(16, 16, true); d.setUint16(20, 1, true); d.setUint16(22, 1, true);
        d.setUint32(24, 16000, true); d.setUint32(28, 32000, true); d.setUint16(32, 2, true); d.setUint16(34, 16, true);
        writeString(36, 'data'); d.setUint32(40, pcm.byteLength, true);
        return URL.createObjectURL(new Blob([header, pcm], { type: 'audio/wav' }));
    }

    // -- OLAY DİNLEYİCİLERİ --

    document.getElementById('user-answer').oninput = (e) => setCheckButtonState(e.target.value.length > 0);
    document.getElementById('user-answer').onkeydown = (e) => { if(e.key === 'Enter' && e.target.value) handleCheck(); };

    window.onload = () => {
        // Tone.js seslerini başlat
        synth = new Tone.PolySynth().toDestination();
        showUnitSelection();
    };
</script>
</body>
</html>
