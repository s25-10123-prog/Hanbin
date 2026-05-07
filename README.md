# Hanbin
```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>타자 연습 마스터</title>
    <style>
        /* 기본 스타일 설정 */
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #f0f2f5;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        .game-container {
            width: 90%;
            max-width: 600px;
            background: white;
            padding: 40px;
            border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            text-align: center;
            position: relative;
            overflow: hidden;
        }

        h1 { color: #333; margin-bottom: 5px; }
        p.subtitle { color: #888; margin-bottom: 30px; font-size: 0.9rem; }

        /* 단어 표시창 */
        #word-target {
            font-size: 3.5rem;
            font-weight: bold;
            margin-bottom: 25px;
            color: #ddd; /* 시작 전에는 흐리게 */
            min-height: 5rem;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        #word-target.active { color: #333; }
        #word-target.correct { color: #4caf50; }
        #word-target.incorrect { color: #f44336; }

        /* 입력창 */
        #input-field {
            width: 100%;
            padding: 15px;
            font-size: 1.5rem;
            border: 2px solid #ddd;
            border-radius: 10px;
            box-sizing: border-box;
            text-align: center;
            transition: border-color 0.3s;
        }

        #input-field:focus {
            outline: none;
            border-color: #2196f3;
        }

        #input-field:disabled {
            background-color: #f9f9f9;
            cursor: not-allowed;
        }

        /* 상태 바 */
        .stats {
            display: flex;
            justify-content: space-between;
            margin-top: 30px;
            padding-top: 20px;
            border-top: 1px solid #eee;
        }

        .stat-item { flex: 1; }
        .stat-label { font-size: 0.8rem; color: #999; margin-bottom: 5px; }
        .stat-value { font-size: 1.2rem; font-weight: bold; color: #2196f3; }

        /* 시작 오버레이 */
        #overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.98);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 100;
        }

        .start-btn {
            padding: 15px 40px;
            font-size: 1.5rem;
            background-color: #2196f3;
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            font-weight: bold;
            box-shadow: 0 4px 15px rgba(33, 150, 243, 0.3);
            transition: transform 0.2s, background-color 0.2s;
        }

        .start-btn:hover {
            background-color: #1976d2;
            transform: scale(1.05);
        }

        .lang-toggle {
            margin-bottom: 20px;
        }

        .lang-btn {
            padding: 8px 20px;
            border: 1px solid #ddd;
            background: white;
            cursor: pointer;
            border-radius: 20px;
            margin: 0 5px;
        }

        .lang-btn.active {
            background: #333;
            color: white;
            border-color: #333;
        }

        /* 결과 모달 */
        #result-modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.7);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 200;
        }

        .modal-content {
            background: white;
            padding: 30px;
            border-radius: 15px;
            width: 300px;
            text-align: center;
        }

        .result-stats {
            text-align: left;
            margin: 20px 0;
            background: #f8f9fa;
            padding: 15px;
            border-radius: 10px;
        }

        .close-btn {
            background: #333;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            width: 100%;
        }
    </style>
</head>
<body>

    <div class="game-container">
        <!-- 시작 오버레이 -->
        <div id="overlay">
            <div class="lang-toggle">
                <button id="btn-ko" class="lang-btn active" onclick="changeLang('ko')">한글</button>
                <button id="btn-en" class="lang-btn" onclick="changeLang('en')">English</button>
            </div>
            <button class="start-btn" onclick="startGame()">시작하기</button>
        </div>

        <h1>타자 연습</h1>
        <p class="subtitle">제한 시간 내에 많은 단어를 입력하세요!</p>

        <div id="word-target">준비</div>

        <input type="text" id="input-field" placeholder="여기에 입력하세요" autocomplete="off" disabled>

        <div class="stats">
            <div class="stat-item">
                <div class="stat-label">속도 (WPM)</div>
                <div id="wpm-val" class="stat-value">0</div>
            </div>
            <div class="stat-item">
                <div class="stat-label">정확도</div>
                <div id="acc-val" class="stat-value">100%</div>
            </div>
            <div class="stat-item">
                <div class="stat-label">시간</div>
                <div id="timer-val" class="stat-value">60</div>
            </div>
        </div>
    </div>

    <!-- 결과 창 -->
    <div id="result-modal">
        <div class="modal-content">
            <h2>게임 종료!</h2>
            <div class="result-stats">
                <p>타수: <span id="res-wpm" style="float:right; font-weight:bold; color:#2196f3;">0</span></p>
                <p>정확도: <span id="res-acc" style="float:right; font-weight:bold; color:#4caf50;">0%</span></p>
            </div>
            <button class="close-btn" onclick="closeResult()">다시 시작</button>
        </div>
    </div>

    <script>
        const words = {
            ko: ["사과", "바나나", "컴퓨터", "대한민국", "나무", "구름", "하늘", "바다", "성공", "노력", "우주", "지구", "사랑", "행복", "미래", "오늘", "내일", "학교", "공부", "운동", "음악", "영화", "친구", "가족", "열정", "희망", "풍경", "여행", "시계", "창의"],
            en: ["apple", "banana", "computer", "korea", "tree", "cloud", "sky", "ocean", "success", "effort", "space", "earth", "love", "happy", "future", "today", "tomorrow", "school", "study", "exercise", "music", "movie", "friend", "family", "passion", "hope", "scenery", "travel", "watch", "creative"]
        };

        let currentLang = 'ko';
        let timeLeft = 60;
        let timer = null;
        let isPlaying = false;
        let currentTarget = "";
        let correctWords = 0;
        let totalChars = 0;
        let correctChars = 0;

        const overlay = document.getElementById('overlay');
        const wordDisplay = document.getElementById('word-target');
        const inputField = document.getElementById('input-field');
        const timerVal = document.getElementById('timer-val');
        const wpmVal = document.getElementById('wpm-val');
        const accVal = document.getElementById('acc-val');
        const resultModal = document.getElementById('result-modal');

        function changeLang(lang) {
            currentLang = lang;
            document.getElementById('btn-ko').classList.toggle('active', lang === 'ko');
            document.getElementById('btn-en').classList.toggle('active', lang === 'en');
        }

        function startGame() {
            // 초기화
            isPlaying = true;
            timeLeft = 60;
            correctWords = 0;
            totalChars = 0;
            correctChars = 0;
            inputField.value = "";
            inputField.disabled = false;
            
            // 오버레이 숨기기
            overlay.style.display = 'none';
            wordDisplay.classList.add('active');
            
            // 첫 단어 세팅 및 포커스
            setNewWord();
            inputField.focus();
            
            // 타이머 시작
            timer = setInterval(updateTimer, 1000);
        }

        function setNewWord() {
            const list = words[currentLang];
            currentTarget = list[Math.floor(Math.random() * list.length)];
            wordDisplay.innerText = currentTarget;
            wordDisplay.className = 'active';
            inputField.value = "";
        }

        function updateTimer() {
            timeLeft--;
            timerVal.innerText = timeLeft;

            // WPM 계산
            const timePassed = (60 - timeLeft) / 60;
            const wpm = Math.round(correctWords / (timePassed || 1));
            wpmVal.innerText = wpm;

            if (timeLeft <= 0) {
                endGame();
            }
        }

        function endGame() {
            clearInterval(timer);
            isPlaying = false;
            inputField.disabled = true;

            const finalAcc = totalChars === 0 ? 0 : Math.round((correctChars / totalChars) * 100);
            document.getElementById('res-wpm').innerText = wpmVal.innerText + " WPM";
            document.getElementById('res-acc').innerText = finalAcc + "%";
            
            resultModal.style.display = 'flex';
        }

        function closeResult() {
            resultModal.style.display = 'none';
            overlay.style.display = 'flex';
            timerVal.innerText = "60";
            wpmVal.innerText = "0";
            accVal.innerText = "100%";
        }

        inputField.addEventListener('input', (e) => {
            if (!isPlaying) return;

            const val = e.target.value.trim();

            // 색상 피드백
            if (currentTarget.startsWith(val)) {
                wordDisplay.className = 'correct';
            } else {
                wordDisplay.className = 'incorrect';
            }

            // 단어 완성 시
            if (val === currentTarget) {
                correctWords++;
                correctChars += currentTarget.length;
                totalChars += currentTarget.length;
                
                // 정확도 갱신
                const acc = Math.round((correctChars / totalChars) * 100);
                accVal.innerText = acc + "%";
                
                setNewWord();
            }
        });

        // 오타 감지 (정확도 계산용)
        inputField.addEventListener('keydown', (e) => {
            if (!isPlaying) return;
            if (e.key.length === 1 && e.key !== 'Enter') {
                totalChars++;
            }
        });
    </script>
</body>
</html>

```
