<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Melodik Matematik Uygulaması</title>
    <!-- Tailwind CSS Yükleniyor -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&display=swap" rel="stylesheet">
    <!-- Tone.js Ses Kütüphanesi -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    <style>
        /* Ana Renkler (Duolingo Esintisi: Canlı Yeşil ve Mavi) */
        :root {
            --color-primary: #58CC02; /* Canlı Yeşil - İlerleme ve Doğru Cevap */
            --color-secondary: #1CB0F6; /* Canlı Mavi - Arkaplan ve Menü */
            --color-dark: #3c3c3c; /* Koyu Gri Metin */
            --color-progress-bg: #e5e5e5; /* İlerleme Çubuğu Arkaplanı */
            --color-progress: #58CC02; /* İlerleme Çubuğu Ön Alanı */
        }

        body {
            font-family: 'Inter', sans-serif;
            background-color: var(--color-secondary);
        }

        .primary-bg { background-color: var(--color-primary); }

        /* Duolingo benzeri canlı buton stilizasyonu */
        .btn-action {
            background-color: var(--color-primary);
            color: white;
            border-bottom: 5px solid #47A800; /* Daha koyu yeşil alt çizgi */
            transition: all 0.1s ease-in-out;
        }

        .btn-action:hover {
            background-color: #63d602;
            border-bottom: 3px solid #47A800;
            transform: translateY(-1px);
        }

        .btn-action:active {
            transform: translateY(2px);
            border-bottom: 2px solid #47A800;
        }
        
        .btn-unit {
            background-color: white;
            color: var(--color-dark);
            border: 2px solid #ddd;
            transition: all 0.2s;
        }
        .btn-unit:hover {
            border-color: var(--color-secondary);
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }

        /* Ses Oynatma İkonu Stilizasyonu */
        .audio-icon-btn {
            background-color: #ffffff;
            color: var(--color-secondary);
            transition: background-color 0.2s;
            border: 2px solid var(--color-secondary);
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .audio-icon-btn:hover {
            background-color: #e0f6ff;
        }

        /* İlerleme Çubuğu */
        .progress-bar {
            height: 12px;
            background-color: var(--color-progress-bg);
            border-radius: 9999px;
            overflow: hidden;
        }

        .progress-fill {
            height: 100%;
            background-color: var(--color-progress);
            transition: width 0.5s ease-out;
        }
        
        /* Loading Indicator Stili */
        #loading-indicator {
            background-color: rgba(255, 255, 255, 0.9);
            z-index: 10;
        }

    </style>
</head>
<body class="min-h-screen flex flex-col items-center justify-start py-8">

    <!-- Yükleniyor Göstergesi -->
    <div id="loading-indicator" class="fixed inset-0 flex flex-col items-center justify-center hidden">
        <div class="animate-spin rounded-full h-16 w-16 border-b-4 border-secondary-bg"></div>
        <p class="mt-4 text-lg font-semibold text-gray-700">Sorular hazırlanıyor...</p>
    </div>

    <!-- Ana Konteyner -->
    <div id="app-container" class="w-full max-w-lg mx-auto p-4 flex flex-col space-y-6">

        <!-- 1. Üst Alan: İlerleme, Başlık ve İptal -->
        <header id="header-bar" class="flex items-center space-x-4">
            <!-- Geri Dön Butonu -->
            <button id="back-button" onclick="showUnitSelection()" class="text-white hover:text-gray-200 transition hidden">
                <!-- SVG Sol Ok İkonu -->
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M10 19l-7-7m0 0l7-7m-7 7h18" />
                </svg>
            </button>
            
            <!-- Uygulama Adı -->
            <h1 id="main-title" class="text-3xl font-extrabold text-white flex-grow text-center">Melodik Matematik</h1>

            <!-- İlerleme Çubuğu (Varsayılan olarak gizli) -->
            <div id="progress-container" class="flex-grow hidden">
                <div class="progress-bar">
                    <div id="progress-fill" class="progress-fill" style="width: 0%;"></div>
                </div>
            </div>

            <!-- Can (Kalp İkonu - Varsayılan olarak gizli) -->
            <div id="life-container" class="text-white flex items-center text-lg font-bold hidden">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-red-500 mr-1" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M3.172 5.172a4 4 0 015.656 0L10 6.343l1.172-1.171a4 4 0 115.656 5.656L10 17.657l-6.828-6.829a4 4 0 010-5.656z" clip-rule="evenodd" />
                </svg>
                <span>5</span>
            </div>
        </header>

        <!-- 2. Ana İçerik Alanı -->
        <main id="main-content" class="bg-white rounded-xl shadow-lg p-6">
            
            <!-- ÜNİTE SEÇİM EKRANI -->
            <div id="unit-selection-screen">
                <div class="flex flex-col items-center mb-6">
                    <!-- Logo/İkon kaldırıldı, sadece başlık kaldı -->
                    <h2 class="text-2xl font-extrabold text-gray-800 text-center mb-2">Başla ve Öğren!</h2>
                    <p class="text-gray-600 text-center">Başlamak istediğin konuyu seçerek sesli egzersizlere başla.</p>
                </div>
                
                <div id="unit-list" class="space-y-4">
                    <!-- Üniteler JS ile buraya eklenecek -->
                </div>
            </div>

            <!-- SORU ALANI (Varsayılan olarak gizli) -->
            <div id="question-screen" class="hidden">
                <h2 class="text-2xl font-extrabold text-gray-800 mb-6 text-center" id="lesson-title"></h2>

                <div class="flex flex-col items-center space-y-8">
                    <!-- Soru Tipi Başlığı -->
                    <p class="text-xl font-extrabold text-center" style="color: var(--color-secondary);">Sadece DİNLE</p>
                    <p class="text-lg font-semibold text-gray-600 text-center">Sana söylenen matematik problemini çöz!</p>

                    <!-- Ses Oynatma Butonu -->
                    <button id="play-audio-btn" class="audio-icon-btn rounded-full shadow-lg w-28 h-28 flex-shrink-0">
                        <svg id="audio-icon" xmlns="http://www.w3.org/2000/svg" class="h-16 w-16" viewBox="0 0 20 20" fill="currentColor">
                            <path fill-rule="evenodd" d="M9.383 3.076A1 1 0 0110 4v12a1 1 0 01-1.707.707L4.586 13H2a1 1 0 01-1-1V8a1 1 0 011-1h2.586l3.707-3.707a1 1 0 011.09-.217zM14.657 5.343a1 1 0 010 1.414A5 5 0 0116 10a5 5 0 01-1.343 3.243 1 1 0 11-1.414-1.414A3 3 0 0014 10a3 3 0 00-.879-2.121 1 1 0 010-1.414zM16.486 3.514a1 1 0 010 1.414c3.241 3.241 3.241 8.491 0 11.732a1 1 0 11-1.414-1.414c2.675-2.676 2.675-7.018 0-9.692a1 1 0 011.414 0z" clip-rule="evenodd" />
                        </svg>
                        <div id="audio-spinner" class="hidden w-10 h-10 border-4 border-gray-300 border-t-4 border-t-blue-500 rounded-full animate-spin"></div>
                    </button>

                    <!-- Cevap Yazma Alanı (Açık Uçlu - Sayısal Değer Bekleniyor) -->
                    <textarea id="user-answer" rows="1" class="w-full p-4 text-center border-2 border-gray-300 rounded-xl focus:border-secondary-bg focus:ring-secondary-bg focus:ring-1 transition resize-none text-gray-800 text-3xl font-bold" placeholder="Cevabınızın sadece sayısal değerini girin..."></textarea>
                </div>
            </div>
        </main>

        <!-- 3. Alt Alan: Kontrol, Mesajlar -->
        <footer id="footer-area" class="mt-8 pt-6 border-t-2 border-gray-200 hidden">
            <div id="feedback-area" class="flex flex-col items-center space-y-4 mb-4">
                <div id="message-box" class="w-full p-3 rounded-xl text-white font-bold transition-all duration-300 shadow-md hidden">
                    <!-- Cevap mesajı buraya gelecek (Doğru/Yanlış) -->
                </div>
            </div>

            <button id="check-button" onclick="checkAnswer()" class="btn-action w-full py-4 rounded-xl text-2xl font-bold uppercase tracking-wider opacity-50 cursor-not-allowed" disabled>
                Kontrol Et
            </button>
        </footer>
    </div>

    <!-- Genel Modal (Duolingo Stili) -->
    <div id="custom-modal" class="fixed inset-0 bg-black bg-opacity-70 z-50 flex items-center justify-center hidden" onclick="hideMessage()">
        <div class="bg-white p-8 rounded-xl shadow-2xl w-full max-w-sm mx-4" onclick="event.stopPropagation()">
            <h3 class="text-xl font-bold text-gray-800 mb-4" id="modal-title">Uyarı</h3>
            <p class="text-gray-600 mb-6" id="modal-content">Bu bir deneme mesajıdır.</p>
            <button onclick="hideMessage()" class="w-full py-3 primary-bg text-white rounded-lg font-bold hover:opacity-90 transition">
                Tamam
            </button>
        </div>
    </div>

<script>
    // Firebase ve Gemini API için gerekli global değişkenler
    const apiKey = ""; // API anahtarı burada olmalı.
    const TOTAL_QUESTIONS_PER_UNIT = 50; // Her ünite için 50 soru

    // Ünite Tanımları
    const UNITS = [
        { id: 1, name: "Ünite 1: Temel Toplama İşlemleri", type: "addition", color: "green" },
        { id: 2, name: "Ünite 2: Temel Çıkarma İşlemleri", type: "subtraction", color: "blue" },
        { id: 3, name: "Ünite 3: Ritmik Sayma Alıştırması", type: "rhythmic", color: "orange" },
        { id: 4, name: "Ünite 4: Basit Örüntü Tamamlama", type: "pattern", color: "purple" }
    ];

    // Durum Değişkenleri
    let synth;
    let currentQuestionIndex = 0;
    let isWaitingForResponse = false;
    let questions = []; // Seçili ünitenin soruları
    let currentUnit = null; // Şu anki aktif ünite objesi

    // DOM Elementleri
    const loadingIndicator = document.getElementById('loading-indicator');
    const unitSelectionScreen = document.getElementById('unit-selection-screen');
    const questionScreen = document.getElementById('question-screen');
    const unitListContainer = document.getElementById('unit-list');
    const headerBar = document.getElementById('header-bar');
    const mainTitle = document.getElementById('main-title');
    const backButton = document.getElementById('back-button');
    const progressContainer = document.getElementById('progress-container');
    const lifeContainer = document.getElementById('life-container');
    const progressFill = document.getElementById('progress-fill');
    const lessonTitle = document.getElementById('lesson-title');
    const userAnswerInput = document.getElementById('user-answer');
    const checkButton = document.getElementById('check-button');
    const playAudioBtn = document.getElementById('play-audio-btn');
    const audioIcon = document.getElementById('audio-icon');
    const audioSpinner = document.getElementById('audio-spinner');
    const messageBox = document.getElementById('message-box');
    const modal = document.getElementById('custom-modal');
    const modalTitle = document.getElementById('modal-title');
    const modalContent = document.getElementById('modal-content');
    const footerArea = document.getElementById('footer-area');

    // --- SES EFECTİ TANIMLAMALARI (Tone.js) ---
    function playCorrectSound() {
        if (!synth) return;
        if (Tone.context.state !== 'running') {
            Tone.context.resume();
        }

        const now = Tone.now();
        const duration = "8n"; 
        synth.triggerAttackRelease("C5", duration, now);
        synth.triggerAttackRelease("E5", duration, now + 0.1);
        synth.triggerAttackRelease("G5", duration, now + 0.2);
    }
    // --- SES EFECTİ TANIMLAMALARI SONU ---

    // --- UI/Ekran Yönetimi ---

    /**
     * Ana menüye (Ünite Seçimine) döner.
     */
    function showUnitSelection() {
        // Durumu sıfırla
        currentUnit = null;
        questions = [];
        currentQuestionIndex = 0;

        // UI'yı ayarla
        unitSelectionScreen.classList.remove('hidden');
        questionScreen.classList.add('hidden');
        footerArea.classList.add('hidden');
        
        // Header'ı ayarla
        backButton.classList.add('hidden');
        progressContainer.classList.add('hidden');
        lifeContainer.classList.add('hidden');
        mainTitle.classList.remove('hidden'); // Başlığı göster
        mainTitle.textContent = "Melodik Matematik";

        renderUnitList();
    }
    
    /**
     * Ünite listesini DOM'a çizer.
     */
    function renderUnitList() {
        unitListContainer.innerHTML = '';
        UNITS.forEach(unit => {
            const button = document.createElement('button');
            button.className = `btn-unit w-full py-4 px-6 rounded-xl text-xl font-bold text-left shadow-md flex items-center`;
            button.innerHTML = `
                <div class="rounded-full w-8 h-8 flex items-center justify-center mr-4" style="background-color: ${unit.color === 'green' ? '#47A800' : unit.color === 'blue' ? '#1CB0F6' : unit.color === 'orange' ? '#FFC700' : '#8A2BE2'}; color: white;">
                    ${unit.id}
                </div>
                <span>${unit.name}</span>
            `;
            button.onclick = () => startUnit(unit);
            unitListContainer.appendChild(button);
        });
    }

    /**
     * Belirtilen üniteyi başlatır.
     */
    async function startUnit(unit) {
        currentUnit = unit;
        questions = generateQuestionsForUnit(unit.type, TOTAL_QUESTIONS_PER_UNIT);
        currentQuestionIndex = 0;

        // UI'yı soru ekranına geçir
        unitSelectionScreen.classList.add('hidden');
        questionScreen.classList.remove('hidden');
        footerArea.classList.remove('hidden');
        
        // Header'ı ayarla
        mainTitle.classList.add('hidden'); // Başlığı gizle
        backButton.classList.remove('hidden');
        progressContainer.classList.remove('hidden');
        lifeContainer.classList.remove('hidden');
        
        lessonTitle.textContent = unit.name;

        // İlk soruyu yükle
        loadingIndicator.classList.remove('hidden');
        
        // İlk sorunun sesini çek
        await fetchAudioFromApi(questions[0].audioText)
            .then(url => {
                questions[0].audioUrl = url;
            })
            .catch(e => {
                console.error("İlk ses yüklenirken hata oluştu, uygulama kullanıma hazır.", e);
            });
            
        loadingIndicator.classList.add('hidden');
        
        updateUI();
    }
    
    /**
     * İlerleme çubuğunu günceller ve bir sonraki soruya hazırlar.
     */
    function updateUI() {
        if (!currentUnit) return; 

        const progress = (currentQuestionIndex / TOTAL_QUESTIONS_PER_UNIT) * 100;
        progressFill.style.width = `${progress}%`;

        if (currentQuestionIndex < TOTAL_QUESTIONS_PER_UNIT) {
            userAnswerInput.value = '';
            checkButton.disabled = true;
            checkButton.classList.add('opacity-50', 'cursor-not-allowed');
            messageBox.classList.add('hidden');
            messageBox.innerHTML = '';
            playAudioBtn.focus();
        } else {
            // Ünite Bitti
            lessonComplete();
        }
        
        // UI güncellendikten sonra bir sonraki soruların seslerini önceden yükle
        preloadNextAudio(); 
    }

    /**
     * Üniteyi tamamlar ve bitiş mesajını gösterir.
     */
    function lessonComplete() {
        showMessage('Tebrikler!', `${currentUnit.name} ünitesini tamamladın! Harikasın!`, 'success');
        checkButton.textContent = 'Menüye Dön';
        checkButton.classList.remove('opacity-50', 'cursor-not-allowed');
        checkButton.disabled = false;
        checkButton.onclick = () => showUnitSelection();
    }


    // --- Soru Üretme Fonksiyonu ---

    /**
     * Belirtilen ünite tipi için soru listesi üretir.
     */
    function generateQuestionsForUnit(type, count) {
        const questionList = [];
        let idCounter = 1;
        
        for (let i = 0; i < count; i++) {
            let text;
            let correctAnswer;

            switch (type) {
                case 'addition':
                    // Toplama (Toplam 10'u geçmez)
                    const numA1 = Math.floor(Math.random() * 9) + 1;
                    const numA2 = Math.floor(Math.random() * (10 - numA1)) + 1;
                    text = `${numA1} artı ${numA2} kaç eder? Cevabını yaz.`;
                    correctAnswer = numA1 + numA2;
                    break;
                    
                case 'subtraction':
                    // Çıkarma (Sonuç negatif olmaz, 10'a kadar)
                    const numS1 = Math.floor(Math.random() * 9) + 1;
                    let numS2 = Math.floor(Math.random() * numS1);
                    numS2 = (numS2 === 0 && numS1 > 0) ? 1 : numS2; // Sıfırdan çıkarma yapmamak için
                    text = `${numS1}'den ${numS2} çıkarsa kaç kalır? Sadece sayıyı yaz.`;
                    correctAnswer = numS1 - numS2;
                    break;
                    
                case 'rhythmic':
                    // Ritmik Sayma (İkişer ve Beşer - 30'a kadar)
                    const step = (i % 2 === 0) ? 2 : 5; // İkişer ve Beşer dönüşümlü
                    const maxStart = step === 2 ? 28 : 25;
                    const start = Math.floor(Math.random() * (maxStart - step)) + 1;
                    const nextNumber = start + step;
                    text = `${start}'dan başlayarak ${step}'şer sayarsan, sonra gelen sayı nedir?`;
                    correctAnswer = nextNumber;
                    break;

                case 'pattern':
                    // Basit Örüntü Tamamlama (Artan veya azalan 1-2-3 adımlı)
                    const patternStep = Math.floor(Math.random() * 3) + 1; // 1, 2 veya 3 artış
                    const isIncreasing = Math.random() < 0.7; // %70 artan
                    let sequence = [];
                    let initialValue = Math.floor(Math.random() * 5) + 1;
                    
                    if (isIncreasing) {
                        // Artan örüntü
                        for(let j = 0; j < 3; j++) {
                            sequence.push(initialValue + j * patternStep);
                        }
                        correctAnswer = sequence[2] + patternStep;
                    } else {
                        // Azalan örüntü
                        // Başlangıç değerini örüntüden sonra negatif olmaması için ayarla
                        initialValue = Math.floor(Math.random() * (15 - 3 * patternStep)) + (3 * patternStep) + 1; 
                        for(let j = 0; j < 3; j++) {
                            sequence.push(initialValue - j * patternStep);
                        }
                        correctAnswer = sequence[2] - patternStep;
                        
                        // Azalan örüntünün sonucu asla negatif olmamalı
                        if (correctAnswer < 0) { 
                            i--; // Bu soruyu atla ve tekrar dene
                            continue;
                        }
                    }
                    // Örüntü sorusunda metni biraz daha açıklayıcı yap
                    text = `${sequence.join(', ')} şeklinde devam eden örüntüdeki bir sonraki sayı nedir? Sadece sayıyı yaz.`;
                    break;
            }

            questionList.push({ id: idCounter++, audioText: text, correctAnswer: correctAnswer, type: type, audioUrl: null, isPreloading: false });
        }
        
        questionList.sort(() => Math.random() - 0.5); 
        return questionList;
    }


    // --- Ses ve Mesaj Fonksiyonları (TTS - Text to Speech) ---
    
    /**
     * Ses oynatma butonunun durumunu yükleme/hazır olarak günceller.
     */
    function setAudioButtonLoading(isLoading) {
        if (isLoading) {
            audioIcon.classList.add('hidden');
            audioSpinner.classList.remove('hidden');
            playAudioBtn.disabled = true;
            playAudioBtn.classList.add('animate-pulse');
        } else {
            audioIcon.classList.remove('hidden');
            audioSpinner.classList.add('hidden');
            playAudioBtn.disabled = false;
            playAudioBtn.classList.remove('animate-pulse');
        }
    }


    /**
     * TTS API'sini kullanarak metni sese dönüştürür ve oynatır.
     * Ses önbellekte varsa anında oynatır.
     */
    async function playAudio() {
        if (isWaitingForResponse || currentQuestionIndex >= questions.length || !currentUnit) return;
        
        const currentQuestion = questions[currentQuestionIndex];
        
        if (currentQuestion.audioUrl) {
            // Önbellekte varsa anında oynat
            try {
                const audio = new Audio(currentQuestion.audioUrl);
                await audio.play(); 
            } catch (e) {
                showMessage(
                    'Ses Oynatma Engellendi', 
                    'Lütfen sesi duymak için hoparlör butonuna TEKRAR basın.', 
                    'info'
                );
            }
            return;
        }

        // Önbellekte yoksa, API çağrısı yap
        await fetchAndPlayAudio(currentQuestion, currentQuestionIndex);
    }

    /**
     * Ses dosyasını API'den alır ve oynatır. Ayrıca başarılı olursa question objesine kaydeder.
     */
    async function fetchAndPlayAudio(question, index) {
        isWaitingForResponse = true;
        setAudioButtonLoading(true);

        try {
            const audioUrl = await fetchAudioFromApi(question.audioText);
            
            questions[index].audioUrl = audioUrl;

            const audio = new Audio(audioUrl);
            try {
                await audio.play();
            } catch (e) {
                if (e.name === "NotAllowedError" || e.name === "AbortError") {
                    showMessage(
                        'Ses Oynatma Engellendi', 
                        'Tarayıcı, medyayı otomatik oynatmayı engelledi. Lütfen sesi duymak için hoparlör butonuna TEKRAR basın.', 
                        'info'
                    );
                } else {
                    throw e; 
                }
            }

        } catch (error) {
            console.error("Ses API Hatası:", error);
            showMessage('Hata', 'Ses servisine bağlanılamadı. Lütfen tekrar deneyin.', 'error');
        } finally {
            isWaitingForResponse = false;
            setAudioButtonLoading(false);
        }
    }

    /**
     * TTS API'sinden ses dosyasını çekip Blob URL'si döner.
     */
    async function fetchAudioFromApi(text) {
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${apiKey}`;
        
        const payload = {
            contents: [{ parts: [{ text: text }] }],
            generationConfig: {
                responseModalities: ["AUDIO"],
                speechConfig: {
                    voiceConfig: {
                        prebuiltVoiceConfig: { voiceName: "Kore" }
                    },
                    languageCode: "tr-TR"
                }
            },
            model: "gemini-2.5-flash-preview-tts"
        };

        const response = await fetchWithRetry(apiUrl, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
        });

        const result = await response.json();
        const part = result?.candidates?.[0]?.content?.parts?.[0];
        const audioData = part?.inlineData?.data;
        const mimeType = part?.inlineData?.mimeType;

        if (audioData && mimeType && mimeType.startsWith("audio/")) {
            const sampleRateMatch = mimeType.match(/rate=(\d+)/);
            const sampleRate = sampleRateMatch ? parseInt(sampleRateMatch[1], 10) : 16000;
            
            const pcmData = base64ToArrayBuffer(audioData);
            const pcm16 = new Int16Array(pcmData);
            const wavBlob = pcmToWav(pcm16, sampleRate);
            return URL.createObjectURL(wavBlob); // Blob URL döndürülür
        } else {
            throw new Error("API'den geçerli ses verisi alınamadı.");
        }
    }

    /**
     * Arka planda sonraki 5 sorunun sesini önbelleğe yükler.
     */
    function preloadNextAudio() {
        const preloadCount = 5;
        for (let i = 1; i <= preloadCount; i++) {
            const index = currentQuestionIndex + i;
            if (index < questions.length && !questions[index].audioUrl && !questions[index].isPreloading) {
                questions[index].isPreloading = true;
                
                fetchAudioFromApi(questions[index].audioText)
                    .then(url => {
                        // Kullanıcı üniteyi terk ettiyse (questions array'i sıfırlandıysa) bu işlemi yapma
                        if (index < questions.length && questions[index]) {
                            questions[index].audioUrl = url;
                            questions[index].isPreloading = false;
                        }
                    })
                    .catch(e => {
                        // Kullanıcı üniteyi terk ettiyse (questions array'i sıfırlandıysa) bu işlemi yapma
                        if (index < questions.length && questions[index]) {
                            questions[index].isPreloading = false;
                        }
                        console.error(`Soru ${index + 1} sesi önbelleğe alınırken hata:`, e);
                    });
            }
        }
    }


    /**
     * Genel amaçlı modal mesajı gösterir (alert yerine).
     */
    function showMessage(title, content, type = 'info') {
        modalTitle.textContent = title;
        modalContent.textContent = content;

        let titleColor = 'text-gray-800';
        if (type === 'success') titleColor = 'text-green-600';
        if (type === 'error') titleColor = 'text-red-600';
        modalTitle.className = `text-xl font-bold mb-4 ${titleColor}`;

        modal.classList.remove('hidden');
    }

    /**
     * Genel amaçlı modal mesajını gizler.
     */
    function hideMessage() {
        modal.classList.add('hidden');
    }

    // --- Cevap Kontrol Fonksiyonu ---

    /**
     * Cevabı kontrol eder.
     */
    async function checkAnswer() {
        if (checkButton.disabled || isWaitingForResponse || !currentUnit) return;

        const answer = userAnswerInput.value.trim();
        const numericAnswer = parseInt(answer);

        if (isNaN(numericAnswer)) {
            showMessage('Lütfen Sayı Girin', 'Lütfen cevabınızı sadece sayısal bir değer olarak yazın.', 'info');
            return;
        }

        isWaitingForResponse = true;
        checkButton.disabled = true;
        checkButton.classList.add('animate-pulse');

        const currentQuestion = questions[currentQuestionIndex];
        const isCorrect = numericAnswer === currentQuestion.correctAnswer;

        await new Promise(resolve => setTimeout(resolve, 1000));
        
        messageBox.classList.remove('hidden');
        messageBox.classList.remove('bg-red-500', 'bg-green-600');
        
        if (isCorrect) {
            playCorrectSound(); 

            messageBox.classList.add('bg-green-600');
            messageBox.innerHTML = `Doğru! Cevap ${currentQuestion.correctAnswer} olacaktı. Süpersin! 🎉`;
            currentQuestionIndex++;
        } else {
            messageBox.classList.add('bg-red-500');
            messageBox.innerHTML = `Yanlış cevap. Tekrar dene! Cevap ${currentQuestion.correctAnswer} olacaktı.`;
        }

        isWaitingForResponse = false;
        checkButton.classList.remove('animate-pulse');
        checkButton.disabled = false;
        
        updateUI(); 
    }

    // --- Olay Dinleyicileri ---

    // Cevap alanındaki değişiklikleri dinle ve butonu etkinleştir.
    userAnswerInput.addEventListener('input', (e) => {
        const textValue = e.target.value.trim();
        const isValidNumber = textValue.length > 0 && !isNaN(parseInt(textValue));
        
        if (isValidNumber) {
            checkButton.disabled = false;
            checkButton.classList.remove('opacity-50', 'cursor-not-allowed');
        } else {
            checkButton.disabled = true;
            checkButton.classList.add('opacity-50', 'cursor-not-allowed');
        }
    });
    
    // Enter tuşuna basıldığında kontrol etme
    userAnswerInput.addEventListener('keypress', (e) => {
        if (e.key === 'Enter' && !checkButton.disabled) {
            e.preventDefault(); 
            checkAnswer();
        }
    });


    // Ses oynatma butonuna tıklama
    playAudioBtn.addEventListener('click', () => {
        playAudio();
    });

    // --- Yardımcı Fonksiyonlar (API ve Ses İşleme) ---

    /**
     * Base64'ü ArrayBuffer'a dönüştürür.
     */
    function base64ToArrayBuffer(base64) {
        const binaryString = atob(base64);
        const len = binaryString.length;
        const bytes = new Uint8Array(len);
        for (let i = 0; i < len; i++) {
            bytes[i] = binaryString.charCodeAt(i);
        }
        return bytes.buffer;
    }

    /**
     * PCM ses verisini WAV dosyasına dönüştürür.
     */
    function pcmToWav(pcmData, sampleRate) {
        const numChannels = 1;
        const bitsPerSample = 16;
        const byteRate = sampleRate * numChannels * (bitsPerSample / 8);
        const blockAlign = numChannels * (bitsPerSample / 8);
        const dataSize = pcmData.byteLength;
        const buffer = new ArrayBuffer(44 + dataSize);
        const view = new DataView(buffer);
        let offset = 0;

        function writeString(s) {
            for (let i = 0; i < s.length; i++) {
                view.setUint8(offset + i, s.charCodeAt(i));
            }
            offset += s.length;
        }

        function writeUint32(i) {
            view.setUint32(offset, i, true);
            offset += 4;
        }

        function writeUint16(i) {
            view.setUint16(offset, i, true);
            offset += 2;
        }

        // RIFF chunk
        writeString('RIFF');
        writeUint32(36 + dataSize); // ChunkSize
        writeString('WAVE');

        // fmt chunk
        writeString('fmt ');
        writeUint32(16); // Subchunk1Size
        writeUint16(1);  // AudioFormat (1 for PCM)
        writeUint16(numChannels); // NumChannels
        writeUint32(sampleRate); // SampleRate
        writeUint32(byteRate); // ByteRate
        writeUint16(blockAlign); // BlockAlign
        writeUint16(bitsPerSample); // BitsPerSample

        // data chunk
        writeString('data');
        writeUint32(dataSize); // Subchunk2Size

        // Write PCM data
        const pcmBytes = new Uint8Array(pcmData.buffer);
        for (let i = 0; i < dataSize; i++) {
            view.setUint8(offset + i, pcmBytes[i]);
        }

        return new Blob([view], { type: 'audio/wav' });
    }

    /**
     * Üstel geri çekilme ile API çağrısı yapar.
     */
    async function fetchWithRetry(url, options, retries = 3) {
        for (let i = 0; i < retries; i++) {
            try {
                const response = await fetch(url, options);
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                return response;
            } catch (error) {
                if (i === retries - 1) {
                    throw error;
                }
                const delay = Math.pow(2, i) * 1000;
                await new Promise(resolve => setTimeout(resolve, delay));
            }
        }
    }


    // Uygulamayı Başlat
    window.onload = function() {
        // Tone.js Synth objesini başlat
        synth = new Tone.PolySynth(Tone.Synth, {
            oscillator: { type: "triangle" }, 
            envelope: {
                attack: 0.01, decay: 0.1, sustain: 0.05, release: 0.5
            }
        }).toDestination();
        
        // İlk olarak ünite seçim ekranını göster
        showUnitSelection();
    };

</script>
</body>
</html>
