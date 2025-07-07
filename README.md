<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>8분할 프레젠테이션 타이머</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Inter 폰트 로드 */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@700&display=swap');

        body {
            font-family: 'Inter', sans-serif;
            overflow: hidden; /* 스크롤바 방지 */
            background-color: #000; /* 전체 배경색을 검정으로 변경 */
        }

        /* 메인 컨테이너의 16:9 비율 유지 및 화면 중앙 정렬 */
        .aspect-ratio-container {
            width: 100vw; /* 뷰포트 너비의 100% */
            height: 100vh; /* 뷰포트 높이의 100% */
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .timer-grid-wrapper {
            position: relative;
            width: 100%;
            padding-bottom: 56.25%; /* 16:9 비율 (9 / 16 * 100%) */
            max-width: calc(100vh * (16 / 9)); /* 높이에 맞춰 최대 너비 설정 */
            max-height: calc(100vw * (9 / 16)); /* 너비에 맞춰 최대 높이 설정 */
            overflow: hidden;
        }

        .timer-grid {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: grid;
            grid-template-columns: repeat(2, 1fr); /* 2열 */
            grid-template-rows: repeat(4, 1fr); /* 4행 */
            gap: 0.75rem; /* 그리드 간격 증가 */
            padding: 0.75rem; /* 전체 패딩 증가 */
            box-sizing: border-box;
        }

        .timer-box {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            border-radius: 1.25rem; /* 둥근 모서리 증가 */
            padding: 0.75rem;
            background-color: #1a1a1a; /* 각 타이머 박스의 배경색을 어두운 회색으로 변경 */
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.5); /* 그림자 진하게 */
            transition: transform 0.2s ease-in-out;
        }

        .timer-box:hover {
            transform: translateY(-8px); /* 호버 효과 증가 */
        }

        .timer-display {
            font-size: 8vw; /* 뷰포트 너비에 비례하여 글자 크기 더 크게 조정 */
            font-weight: 900; /* 더 진하게 */
            margin-bottom: 0.75rem; /* 마진 증가 */
            line-height: 1; /* 줄 간격 조절 */
            text-shadow: 3px 3px 6px rgba(0, 0, 0, 0.4); /* 텍스트 그림자 진하게 */
        }

        /* 숫자 색상: 노란색 */
        .text-yellow-timer {
            color: #fcd34d; /* Tailwind yellow-300 */
        }

        /* 숫자 색상: 초록색 */
        .text-green-timer {
            color: #86efac; /* Tailwind green-300 */
        }

        .timer-controls {
            display: flex;
            gap: 0.4rem; /* 버튼 간격 더 작게 */
        }

        .timer-button {
            padding: 0.35rem 0.8rem; /* 버튼 패딩 더 작게 */
            font-size: 0.8rem; /* 버튼 글자 크기 더 작게 */
            font-weight: bold;
            border-radius: 0.4rem; /* 둥근 모서리 */
            cursor: pointer;
            transition: background-color 0.2s, transform 0.1s, opacity 0.2s;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.15);
            color: #fff; /* 버튼 글자색은 흰색으로 통일 */
        }

        .timer-button:active {
            transform: translateY(1px);
        }

        .timer-button:disabled {
            opacity: 0.5;
            cursor: not-allowed;
            box-shadow: none;
        }

        /* 버튼 색상 */
        .timer-button.start-btn {
            background-color: #3b82f6; /* Tailwind blue-500 */
        }
        .timer-button.start-btn:hover:not(:disabled) {
            background-color: #2563eb; /* Tailwind blue-600 */
        }

        .timer-button.stop-btn {
            background-color: #ef4444; /* Tailwind red-500 */
        }
        .timer-button.stop-btn:hover:not(:disabled) {
            background-color: #dc2626; /* Tailwind red-600 */
        }

        .timer-button.reset-btn {
            background-color: #6b7280; /* Tailwind gray-500 */
            color: #fff;
        }
        .timer-button.reset-btn:hover:not(:disabled) {
            background-color: #4b5563; /* Tailwind gray-600 */
        }

        /* 미디어 쿼리: 화면 크기에 따라 폰트 크기 및 간격 조정 */
        @media (max-width: 768px) {
            .timer-display {
                font-size: 12vw; /* 모바일에서 더 크게 */
            }
            .timer-button {
                font-size: 0.7rem;
                padding: 0.3rem 0.7rem;
            }
        }
        @media (max-width: 480px) {
            .timer-display {
                font-size: 18vw; /* 아주 작은 화면에서 더 크게 */
            }
            .timer-button {
                font-size: 0.6rem;
                padding: 0.25rem 0.5rem;
            }
        }
    </style>
</head>
<body>
    <div class="aspect-ratio-container">
        <div class="timer-grid-wrapper">
            <div id="timerGrid" class="timer-grid">
                <!-- 타이머가 여기에 동적으로 생성됩니다 -->
            </div>
        </div>
    </div>

    <script>
        // 타이머 데이터 배열
        const timers = [
            { id: 'timer1', initialTime: 3 * 60, currentTime: 3 * 60, interval: null, running: false, color: 'yellow' },
            { id: 'timer2', initialTime: 3 * 60, currentTime: 3 * 60, interval: null, running: false, color: 'green' },
            { id: 'timer3', initialTime: 4 * 60, currentTime: 4 * 60, interval: null, running: false, color: 'yellow' },
            { id: 'timer4', initialTime: 4 * 60, currentTime: 4 * 60, interval: null, running: false, color: 'green' },
            { id: 'timer5', initialTime: 2 * 60, currentTime: 2 * 60, interval: null, running: false, color: 'yellow' },
            { id: 'timer6', initialTime: 2 * 60, currentTime: 2 * 60, interval: null, running: false, color: 'green' },
            { id: 'timer7', initialTime: 3 * 60, currentTime: 3 * 60, interval: null, running: false, color: 'yellow' },
            { id: 'timer8', initialTime: 3 * 60, currentTime: 3 * 60, interval: null, running: false, color: 'green' }
        ];

        // HTML 요소 참조
        const timerGrid = document.getElementById('timerGrid');

        /**
         * 시간을 MM:SS 형식으로 포맷합니다.
         * @param {number} seconds - 총 초 단위 시간
         * @returns {string} 포맷된 시간 문자열
         */
        function formatTime(seconds) {
            const minutes = Math.floor(seconds / 60);
            const remainingSeconds = seconds % 60;
            return `${String(minutes).padStart(2, '0')}:${String(remainingSeconds).padStart(2, '0')}`;
        }

        /**
         * 특정 타이머의 디스플레이를 업데이트합니다.
         * @param {number} index - 타이머 배열의 인덱스
         */
        function updateDisplay(index) {
            const timer = timers[index];
            const displayElement = document.getElementById(`${timer.id}-display`);
            if (displayElement) {
                displayElement.textContent = formatTime(timer.currentTime);
                // 시간이 0이 되면 정지
                if (timer.currentTime <= 0 && timer.running) {
                    stopTimer(index);
                }
            }
        }

        /**
         * 특정 타이머를 시작합니다.
         * @param {number} index - 타이머 배열의 인덱스
         */
        function startTimer(index) {
            const timer = timers[index];
            const startButton = document.getElementById(`${timer.id}-start`);
            const stopButton = document.getElementById(`${timer.id}-stop`);

            if (!timer.running && timer.currentTime > 0) {
                timer.running = true;
                startButton.disabled = true; // 시작 버튼 비활성화
                stopButton.disabled = false; // 정지 버튼 활성화

                timer.interval = setInterval(() => {
                    timer.currentTime--;
                    updateDisplay(index);
                    if (timer.currentTime <= 0) {
                        stopTimer(index); // 시간이 0이 되면 자동으로 정지
                    }
                }, 1000);
            }
        }

        /**
         * 특정 타이머를 정지합니다.
         * @param {number} index - 타이머 배열의 인덱스
         */
        function stopTimer(index) {
            const timer = timers[index];
            const startButton = document.getElementById(`${timer.id}-start`);
            const stopButton = document.getElementById(`${timer.id}-stop`);

            if (timer.running) {
                clearInterval(timer.interval);
                timer.running = false;
                startButton.disabled = false; // 시작 버튼 활성화
                stopButton.disabled = true; // 정지 버튼 비활성화
            }
        }

        /**
         * 특정 타이머를 초기화합니다.
         * @param {number} index - 타이머 배열의 인덱스
         */
        function resetTimer(index) {
            const timer = timers[index];
            stopTimer(index); // 정지 후 초기화
            timer.currentTime = timer.initialTime;
            updateDisplay(index);

            // 초기화 후 시작 버튼 활성화, 정지 버튼 비활성화
            const startButton = document.getElementById(`${timer.id}-start`);
            const stopButton = document.getElementById(`${timer.id}-stop`);
            startButton.disabled = false;
            stopButton.disabled = true;
        }

        /**
         * 모든 타이머를 HTML에 생성하고 초기화합니다.
         */
        function createTimers() {
            timers.forEach((timer, index) => {
                const timerBox = document.createElement('div');
                timerBox.id = timer.id;
                // 타이머 박스 배경색은 CSS의 .timer-box에서 설정
                timerBox.classList.add('timer-box');

                timerBox.innerHTML = `
                    <div id="${timer.id}-display" class="timer-display ${timer.color === 'yellow' ? 'text-yellow-timer' : 'text-green-timer'}"></div>
                    <div class="timer-controls">
                        <button id="${timer.id}-start" class="timer-button start-btn">시작</button>
                        <button id="${timer.id}-stop" class="timer-button stop-btn" disabled>정지</button>
                        <button id="${timer.id}-reset" class="timer-button reset-btn">초기화</button>
                    </div>
                `;
                timerGrid.appendChild(timerBox);

                // 이벤트 리스너 추가
                document.getElementById(`${timer.id}-start`).addEventListener('click', () => startTimer(index));
                document.getElementById(`${timer.id}-stop`).addEventListener('click', () => stopTimer(index));
                document.getElementById(`${timer.id}-reset`).addEventListener('click', () => resetTimer(index));

                // 초기 디스플레이 업데이트
                updateDisplay(index);
            });
        }

        // 페이지 로드 시 타이머 생성 및 초기화
        window.onload = createTimers;
    </script>
</body>
</html>
