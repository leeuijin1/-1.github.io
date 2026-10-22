<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>디지털 소외 계층 키오스크 체험 - Light Single View Simulator</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- FontAwesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;600;700&display=swap');
    body {
      font-family: 'Noto Sans KR', sans-serif;
      user-select: none;
      background-color: #ffffff;
      color: #0f172a;
    }

    /* 고령층 시야 특수 효과 - 약한 Blur & 작은 글씨 (밝은 화면 고정) */
    .elderly-view {
      filter: blur(0.8px) contrast(98%) brightness(102%);
      font-size: 0.72rem !important; /* 모바일 맞춤 글씨 */
      transform-origin: center center;
    }

    /* 미세 손떨림 효과 (부드러운 진동) */
    @keyframes tremor {
      0% { transform: translate(0, 0) rotate(0deg); }
      20% { transform: translate(-0.5px, 0.4px) rotate(-0.05deg); }
      40% { transform: translate(0.6px, -0.3px) rotate(0.08deg); }
      60% { transform: translate(-0.4px, -0.5px) rotate(-0.03deg); }
      80% { transform: translate(0.5px, 0.3px) rotate(0.05deg); }
      100% { transform: translate(0, 0) rotate(0deg); }
    }

    .elderly-tremor {
      animation: tremor 0.25s infinite ease-in-out;
    }

    /* 긴급 붉은 깜빡임 효과 */
    @keyframes pulse-red {
      0%, 100% { opacity: 1; color: #dc2626; }
      50% { opacity: 0.4; color: #991b1b; }
    }
    .urgent-blink {
      animation: pulse-red 0.8s infinite;
    }

    /* 스크롤바 감추기 */
    .no-scrollbar::-webkit-scrollbar {
      display: none;
    }
    .no-scrollbar {
      -ms-overflow-style: none;
      scrollbar-width: none;
    }
  </style>
</head>
<body class="h-[100dvh] w-screen overflow-hidden flex flex-col bg-white text-slate-800">

  <!-- 상단 헤더 & 컨트롤 바 (모바일 세로형 가독성 최적화) -->
  <header class="bg-slate-100 border-b border-slate-200 px-2 sm:px-4 py-2 flex justify-between items-center z-50 shrink-0 shadow-sm">
    <div class="flex items-center space-x-1.5">
      <span class="bg-indigo-600 text-white text-[10px] sm:text-xs font-bold px-2 py-0.5 sm:py-1 rounded-full flex items-center gap-1 shrink-0">
        <i class="fa-solid fa-eye-low-vision"></i> 약자 시뮬레이터
      </span>
      <h1 class="text-xs sm:text-base font-bold tracking-tight text-slate-800 truncate hidden xs:block">고령층 키오스크 체감</h1>
    </div>

    <!-- 미션 가이드 & 컨트롤 -->
    <div class="flex items-center space-x-1.5 sm:space-x-2">
      <button onclick="showMissionModal()" class="text-[11px] sm:text-xs bg-white hover:bg-slate-50 text-slate-700 px-2 py-1 rounded-lg transition border border-slate-300 font-medium flex items-center gap-1 shadow-sm">
        <i class="fa-solid fa-clipboard-list text-amber-500"></i>
        <span>미션</span>
        <span id="missionCheckBadge" class="bg-amber-100 text-amber-800 text-[10px] font-bold px-1.5 py-0.2 rounded-full">0</span>
      </button>
      <button id="audioToggleBtn" onclick="toggleAudio()" class="text-[11px] sm:text-xs bg-white hover:bg-slate-50 text-slate-700 px-2 py-1 rounded-lg transition border border-slate-300 font-medium flex items-center gap-1 shadow-sm">
        <i class="fa-solid fa-volume-high text-indigo-600" id="audioIcon"></i>
        <span id="audioStatusText" class="hidden sm:inline">소음 OFF</span>
      </button>
      <button onclick="resetExperience()" class="text-[11px] sm:text-xs bg-rose-50 hover:bg-rose-100 text-rose-700 px-2 py-1 rounded-lg transition border border-rose-200 font-semibold">
        <i class="fa-solid fa-rotate-right"></i>
      </button>
    </div>
  </header>

  <!-- 실시간 심리 압박 정보 바 (모바일 핏) -->
  <div class="bg-amber-50 border-b border-amber-200 px-2.5 py-1 text-[11px] sm:text-xs text-amber-900 flex justify-between items-center z-40 shrink-0">
    <div class="flex items-center justify-between w-full sm:w-auto sm:justify-start sm:space-x-4">
      <span class="flex items-center gap-1 text-rose-700 font-bold urgent-blink">
        <i class="fa-solid fa-triangle-exclamation text-rose-600"></i>
        <span>제한시간: <span id="timerDisplay" class="text-xs sm:text-sm font-extrabold underline">300</span>초</span>
      </span>
      <span class="text-amber-800 font-medium">
        <i class="fa-solid fa-users text-amber-600"></i>
        대기: <strong id="waitingCount" class="text-rose-600 text-xs sm:text-sm">3</strong>명
      </span>
    </div>
  </div>

  <!-- 메인 단일 키오스크 영역 (모바일 디바이스 맞춤 flex-1) -->
  <main class="flex-1 flex items-center justify-center p-1.5 sm:p-4 bg-slate-50 overflow-hidden relative">
    <div class="w-full max-w-sm sm:max-w-md h-full bg-white rounded-xl sm:rounded-2xl border-2 sm:border-4 border-slate-300 flex flex-col overflow-hidden relative shadow-lg">
      <!-- 키오스크 타이틀 바 -->
      <div class="bg-indigo-50 px-2.5 py-1.5 text-[10px] sm:text-xs font-bold text-indigo-900 border-b border-indigo-100 flex justify-between items-center shrink-0">
        <span class="flex items-center gap-1 truncate"><i class="fa-solid fa-display text-indigo-600"></i> 고령층 체감 시야</span>
        <span class="text-[9px] bg-indigo-100 text-indigo-700 px-1.5 py-0.5 rounded font-semibold shrink-0">체험 전용</span>
      </div>

      <!-- 키오스크 화면 프레임 (모바일 스크롤 가능) -->
      <div id="kioskContainer" class="flex-1 overflow-y-auto no-scrollbar relative flex flex-col bg-white elderly-view elderly-tremor">
        <!-- JS 렌더링 -->
      </div>
    </div>

    <!-- 터치 반응 지연 레이어 -->
    <div id="delayOverlay" class="hidden absolute inset-0 bg-slate-900/20 z-50 pointer-events-auto cursor-wait flex items-center justify-center backdrop-blur-[2px]">
      <div class="bg-white text-slate-800 border border-slate-300 px-4 py-2.5 rounded-xl text-xs shadow-2xl animate-pulse flex items-center gap-2 font-bold">
        <i class="fa-solid fa-spinner fa-spin text-sm text-indigo-600"></i>
        <span>시스템 처리 지연 중...</span>
      </div>
    </div>
  </main>

  <!-- 10초 후 사라지는 건망증 체험 미션 팝업 -->
  <div id="missionModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center p-3">
    <div class="bg-white border-2 border-amber-400 rounded-xl max-w-md w-full p-4 sm:p-5 text-slate-800 shadow-2xl relative max-h-[90vh] overflow-y-auto">
      <div class="flex justify-between items-start mb-2.5 border-b border-slate-100 pb-2">
        <div>
          <span class="bg-amber-100 text-amber-800 text-[10px] font-bold px-2 py-0.5 rounded-full border border-amber-200">기억력 체험 미션</span>
          <h2 class="text-base sm:text-lg font-bold mt-1 text-slate-900">주문할 메뉴를 기억하세요!</h2>
        </div>
        <div class="text-right shrink-0">
          <span id="modalCountdown" class="text-2xl font-extrabold text-rose-600">10</span>
          <p class="text-[9px] text-slate-400">초 뒤 사라짐</p>
        </div>
      </div>

      <div class="bg-slate-50 p-3 rounded-lg border border-slate-200 space-y-1.5 mb-3">
        <p class="text-[11px] text-slate-600 font-medium">다음 조건에 정확히 맞춰 주문하세요:</p>
        <div class="bg-amber-50 p-2.5 rounded-md border border-amber-200 text-amber-950 text-xs font-semibold space-y-1">
          <p>1. <span class="text-slate-900 font-bold underline">아이스 아메리카노</span> 1잔 선택</p>
          <p>2. 옵션: <span class="text-amber-800 underline">에스프레소 Extra Shot 2회</span> 추가</p>
          <p>3. 옵션: <span class="text-amber-800 underline">두유(Soy Milk)</span> 변경 & <span class="text-amber-800 underline">얼음 최소화</span></p>
          <p>4. 쿠폰: <span class="text-amber-800 underline">디저트 콤보 할인 쿠폰</span> 적용</p>
          <p>5. 수령: <span class="text-slate-900 font-bold underline">To-Go (포장)</span> 선택</p>
        </div>
      </div>

      <p class="text-[10px] text-rose-800 bg-rose-50 p-2.5 rounded-md border border-rose-200 flex items-start gap-1.5 leading-tight">
        <i class="fa-solid fa-brain text-rose-600 mt-0.5 shrink-0"></i>
        <span>어르신들이 겪는 순간 건망증 체험을 위해 10초 후 숨겨집니다.</span>
      </p>

      <button onclick="startSessionNow()" class="mt-3 w-full bg-amber-500 hover:bg-amber-600 text-slate-950 font-bold py-2.5 rounded-lg text-xs transition shadow-md">
        지금 바로 시작하기
      </button>
    </div>
  </div>

  <!-- 체험 결과 / 리포트 팝업 -->
  <div id="resultModal" class="hidden fixed inset-0 bg-slate-900/70 backdrop-blur-sm z-50 flex items-center justify-center p-3 overflow-y-auto">
    <div class="bg-white border border-slate-200 rounded-xl max-w-md w-full p-4 sm:p-5 text-slate-800 shadow-2xl space-y-3.5 my-auto max-h-[92vh] overflow-y-auto">
      <div id="resultHeader" class="text-center space-y-0.5">
        <!-- JS로 다이나믹 생성 -->
      </div>

      <div class="bg-slate-50 p-3 rounded-lg border border-slate-200 space-y-2.5">
        <h4 class="text-[10px] font-bold text-slate-500 uppercase tracking-wider border-b border-slate-200 pb-1">체험 및 조작 분석 리포트</h4>
        
        <div class="grid grid-cols-3 gap-1.5 text-center">
          <div class="bg-white p-2 rounded border border-slate-200 shadow-sm">
            <span class="text-[9px] text-slate-500 block">소요 시간</span>
            <span id="resTime" class="text-sm font-extrabold text-amber-600">0초</span>
          </div>
          <div class="bg-white p-2 rounded border border-slate-200 shadow-sm">
            <span class="text-[9px] text-slate-500 block">미션 재확인</span>
            <span id="resCheckCount" class="text-sm font-extrabold text-indigo-600">0회</span>
          </div>
          <div class="bg-white p-2 rounded border border-slate-200 shadow-sm">
            <span class="text-[9px] text-slate-500 block">실수/방황</span>
            <span id="resMistakes" class="text-sm font-extrabold text-rose-600">0회</span>
          </div>
        </div>

        <!-- 주문 세부 조건 세부 일치 결과 검증 -->
        <div class="bg-white p-2.5 rounded border border-slate-200 space-y-1">
          <span class="text-[10px] font-bold text-slate-700 block border-b border-slate-100 pb-0.5">주문 세부 조건 검증</span>
          <div id="missionMatchDetails" class="text-[11px] space-y-0.5 font-medium">
            <!-- JS 다이나믹 생성 -->
          </div>
        </div>

        <div class="space-y-0.5">
          <div class="flex justify-between text-[11px]">
            <span class="text-slate-600 font-medium">인지 부담 지수:</span>
            <span id="stressLevelText" class="font-bold text-rose-600">높음</span>
          </div>
          <div class="w-full bg-slate-200 h-2 rounded-full overflow-hidden">
            <div id="stressBar" class="bg-gradient-to-r from-amber-500 to-rose-600 h-full w-[80%]"></div>
          </div>
        </div>
      </div>

      <!-- 공감 메시지 및 사회적 의의 -->
      <div class="bg-indigo-50 border border-indigo-200 p-3 rounded-lg text-[11px] space-y-1 text-indigo-950">
        <h4 class="font-bold text-indigo-900 flex items-center gap-1 text-xs">
          <i class="fa-solid fa-heart-pulse text-indigo-600"></i> 함께 생각해요
        </h4>
        <p class="leading-snug text-slate-700">
          이 답답함은 **고령층이 일상에서 느끼는 실제 무력감**입니다.
        </p>
        <ul class="list-disc list-inside space-y-0.5 text-slate-600 pl-0.5 text-[10px]">
          <li><strong>쉬운 우리말 표기</strong> 도입</li>
          <li><strong>고대비·큰 글씨 모드</strong> 필수 제공</li>
          <li>타이머 없는 <strong>충분한 조작 시간 보장</strong></li>
        </ul>
      </div>

      <button onclick="resetExperience()" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2.5 rounded-lg text-xs transition shadow-md">
        다시 체험해보기
      </button>
    </div>
  </div>


  <!-- JavaScript Logic -->
  <script>
    // --- APP STATE ---
    const state = {
      step: 'MENU',
      selectedCategory: 'BEVERAGE',
      selectedItem: null,
      customizations: {
        shots: 0,
        milk: 'REGULAR',
        ice: 'NORMAL'
      },
      cart: [],
      appliedCoupon: false,
      takeoutType: null,
      
      // 통계 & 추적 데이터
      timeLeft: 300,
      waitingCount: 3,
      mistakes: 0,
      checkMissionCount: 0, // 미션/주문 재확인 횟수
      startTime: null,
      timerInterval: null,
      audioActive: false,
      audioCtx: null,
      noiseNode: null
    };

    // 난해한 외래어 카테고리 & 메뉴 데이터
    const menuData = {
      BEVERAGE: [
        { id: 'm1', name: 'Espresso d\'Illusion Water', desc: '아이스 아메리카노 (기본형)', price: 4500, target: true },
        { id: 'm2', name: 'Velvet Lactose Latte System', desc: '카페 라떼', price: 5000, target: false },
        { id: 'm3', name: 'Versailles Caramel Extract Dark Brew', desc: '카라멜 마키아토', price: 5800, target: false }
      ],
      DESERT: [
        { id: 'd1', name: 'Glucose Layered Pastry Portion', desc: '조각 케이크', price: 6500, target: false },
        { id: 'd2', name: 'Artisan Dough Bakery Unit', desc: '크루아상', price: 3800, target: false }
      ],
      SPECIAL_PROMO: [
        { id: 'p1', name: 'Seasonal Synergy Package Option', desc: '시즌 세트', price: 9900, target: false }
      ]
    };

    // --- INITIALIZATION & TIMER ---
    let missionCountdown = 10;
    let missionInterval = null;

    // 터치 반응 지연 시뮬레이션 함수 (handleAction 추가)
    function handleAction(callback) {
      const overlay = document.getElementById('delayOverlay');
      if (overlay) overlay.classList.remove('hidden');
      
      // 0.6초 지연 후 동작 실행 (어르신의 터치 지연/반응 속도 체험)
      setTimeout(() => {
        if (overlay) overlay.classList.add('hidden');
        if (typeof callback === 'function') {
          callback();
        }
        renderKiosk();
      }, 600);
    }

    // 카테고리 선택
    function selectCategory(cat) {
      state.selectedCategory = cat;
    }

    // 옵션 설정 창 열기
    function openCustomize(item) {
      state.selectedItem = item;
      state.customizations = { shots: 0, milk: 'REGULAR', ice: 'NORMAL' };
      state.step = 'CUSTOMIZE';
    }

    // 샷 수량 변경
    function changeShot(delta) {
      state.customizations.shots = Math.max(0, state.customizations.shots + delta);
    }

    // 우유 옵션 변경
    function setMilk(milkType) {
      state.customizations.milk = milkType;
    }

    // 얼음 옵션 변경
    function setIce(iceType) {
      state.customizations.ice = iceType;
    }

    // 장바구니에 담기
    function addToCart() {
      if (state.selectedItem) {
        state.cart = [{ item: state.selectedItem, custom: { ...state.customizations } }];
        state.step = 'CART';
      }
    }

    // 쿠폰 적용 토글
    function toggleCoupon() {
      state.appliedCoupon = !state.appliedCoupon;
    }

    // 수령 방식 설정
    function setTakeout(type) {
      state.takeoutType = type;
    }

    // 결제 단계 이동 및 최종 주문 처리
    function proceedToPayment() {
      if (state.step === 'CART') {
        state.step = 'PAYMENT';
      } else if (state.step === 'MENU') {
        if (state.cart.length > 0) {
          state.step = 'CART';
        } else {
          state.mistakes++;
        }
      }
    }

    function finalizeOrder() {
      finishExperience(false);
    }

    // 소음 토글 기능
    function toggleAudio() {
      state.audioActive = !state.audioActive;
      const btnText = document.getElementById('audioStatusText');
      const btnIcon = document.getElementById('audioIcon');
      
      if (state.audioActive) {
        if (btnText) btnText.innerText = "소음 ON";
        if (btnIcon) btnIcon.className = "fa-solid fa-volume-high text-rose-600";
      } else {
        if (btnText) btnText.innerText = "소음 OFF";
        if (btnIcon) btnIcon.className = "fa-solid fa-volume-xmark text-slate-400";
      }
    }

    window.onload = () => {
      startMissionTimer();
      renderKiosk();
    };

    function startMissionTimer() {
      const cdEl = document.getElementById('modalCountdown');
      missionInterval = setInterval(() => {
        missionCountdown--;
        if (cdEl) cdEl.innerText = missionCountdown;
        if (missionCountdown <= 0) {
          clearInterval(missionInterval);
          startSessionNow();
        }
      }, 1000);
    }

    function startSessionNow() {
      clearInterval(missionInterval);
      document.getElementById('missionModal').classList.add('hidden');
      state.startTime = Date.now();
      
      // 300초 메인 타이머 시작
      if (!state.timerInterval) {
        state.timerInterval = setInterval(() => {
          state.timeLeft--;
          document.getElementById('timerDisplay').innerText = state.timeLeft;
          
          // 대기 인원 증가 심리 압박 (30초마다 1명씩 증가)
          if (state.timeLeft % 30 === 0) {
            state.waitingCount++;
            document.getElementById('waitingCount').innerText = state.waitingCount;
          }

          if (state.timeLeft <= 0) {
            clearInterval(state.timerInterval);
            finishExperience(false); // 시간 초과 실패
          }
        }, 1000);
      }
    }

    function showMissionModal() {
      state.checkMissionCount++;
      const badge = document.getElementById('missionCheckBadge');
      if (badge) badge.innerText = state.checkMissionCount;
      document.getElementById('missionModal').classList.remove('hidden');
    }

    // --- AUDIOWAVE BACKGROUND NOISE (시끄러운 카페 주변 소음 가상 생성) ---
    function finishExperience(isTimeoutFail = false) {
      clearInterval(state.timerInterval);
      const totalTime = Math.round((Date.now() - state.startTime) / 1000);

      // 세부 주문 조건 검증
      const cartItem = state.cart.length > 0 ? state.cart[0] : null;
      const checks = {
        itemMatch: cartItem && cartItem.item.id === 'm1',
        shotMatch: cartItem && cartItem.custom.shots === 2,
        milkMatch: cartItem && cartItem.custom.milk === 'SOY_SYNERGY',
        iceMatch: cartItem && cartItem.custom.ice === 'ICE_MINIMIZE',
        couponMatch: state.appliedCoupon === true,
        togoMatch: state.takeoutType === 'TOGO'
      };

      const isAllMatched = Object.values(checks).every(Boolean);

      const resModal = document.getElementById('resultModal');
      const header = document.getElementById('resultHeader');

      if (!isTimeoutFail && isAllMatched) {
        header.innerHTML = `
          <div class="text-emerald-500 text-4xl mb-1"><i class="fa-solid fa-circle-check"></i></div>
          <h3 class="text-xl font-extrabold text-slate-900">주문 미션 완벽 성공!</h3>
          <p class="text-xs text-slate-500">모든 복잡한 옵션 조건을 정확하게 확인하여 주문했습니다.</p>
        `;
      } else {
        header.innerHTML = `
          <div class="text-rose-500 text-4xl mb-1"><i class="fa-solid fa-circle-xmark"></i></div>
          <h3 class="text-xl font-extrabold text-slate-900">${isTimeoutFail ? '시간 초과로 주문 실패' : '주문 세부 조건 불일치'}</h3>
          <p class="text-xs text-slate-500">어르신들이 복잡한 메뉴와 옵션 선택에서 겪는 어려움을 체험하셨습니다.</p>
        `;
      }

      // 통계 값 바인딩
      document.getElementById('resTime').innerText = totalTime + "초";
      document.getElementById('resCheckCount').innerText = state.checkMissionCount + "회";
      document.getElementById('resMistakes').innerText = state.mistakes + "회";

      // 주문 세부 결과 브레이크다운 생성
      const detailsContainer = document.getElementById('missionMatchDetails');
      detailsContainer.innerHTML = `
        <div class="flex justify-between items-center ${checks.itemMatch ? 'text-emerald-700' : 'text-rose-600'}">
          <span>1. 메뉴 (아이스 아메리카노):</span>
          <span class="font-bold">${checks.itemMatch ? '✓ 일치' : '✗ 불일치'}</span>
        </div>
        <div class="flex justify-between items-center ${checks.shotMatch ? 'text-emerald-700' : 'text-rose-600'}">
          <span>2. 옵션 (샷 2회 추가):</span>
          <span class="font-bold">${checks.shotMatch ? '✓ 일치' : '✗ 불일치'}</span>
        </div>
        <div class="flex justify-between items-center ${checks.milkMatch ? 'text-emerald-700' : 'text-rose-600'}">
          <span>3. 옵션 (두유 / Soy Milk):</span>
          <span class="font-bold">${checks.milkMatch ? '✓ 일치' : '✗ 불일치'}</span>
        </div>
        <div class="flex justify-between items-center ${checks.iceMatch ? 'text-emerald-700' : 'text-rose-600'}">
          <span>4. 옵션 (얼음 최소화):</span>
          <span class="font-bold">${checks.iceMatch ? '✓ 일치' : '✗ 불일치'}</span>
        </div>
        <div class="flex justify-between items-center ${checks.couponMatch ? 'text-emerald-700' : 'text-rose-600'}">
          <span>5. 쿠폰 (디저트 콤보 할인):</span>
          <span class="font-bold">${checks.couponMatch ? '✓ 일치' : '✗ 불일치'}</span>
        </div>
        <div class="flex justify-between items-center ${checks.togoMatch ? 'text-emerald-700' : 'text-rose-600'}">
          <span>6. 수령 방식 (To-Go 포장):</span>
          <span class="font-bold">${checks.togoMatch ? '✓ 일치' : '✗ 불일치'}</span>
        </div>
      `;

      resModal.classList.remove('hidden');
    }

    function resetExperience() {
      location.reload();
    }

    // --- RENDER KIOSK SINGLE VIEW (BRIGHT LIGHT THEME) ---
    function renderKiosk() {
      document.getElementById('kioskContainer').innerHTML = buildKioskHTML();
    }

    function buildKioskHTML() {
      const clickHandler = (fnStr) => `onclick="handleAction(() => { ${fnStr} })"`;

      // 1. 카테고리/메뉴 선택 화면
      if (state.step === 'MENU') {
        const items = menuData[state.selectedCategory] || [];
        return `
          <div class="flex flex-col h-full bg-white text-slate-800">
            <!-- 밝은 상단 브랜드 헤더 -->
            <div class="bg-indigo-600 p-3 text-center text-white">
              <span class="text-[9px] tracking-widest text-indigo-200 uppercase font-bold">POS DIRECT ORDER SYSTEM v4.2</span>
              <h2 class="text-xs font-extrabold text-white mt-0.5">BEVERAGE & DESERT SYNERGY STORE</h2>
            </div>

            <!-- 불친절한 복잡 서브 카테고리 (밝은 라이트 스타일) -->
            <div class="flex bg-slate-100 border-b border-slate-200 text-[11px]">
              <button ${clickHandler("selectCategory('BEVERAGE')")} class="flex-1 py-2.5 px-1 font-bold border-b-2 text-center transition ${state.selectedCategory === 'BEVERAGE' ? 'border-amber-500 text-amber-700 bg-white' : 'border-transparent text-slate-500'}">
                COFFEE & INFUSION
              </button>
              <button ${clickHandler("selectCategory('DESERT')")} class="flex-1 py-2.5 px-1 font-bold border-b-2 text-center transition ${state.selectedCategory === 'DESERT' ? 'border-amber-500 text-amber-700 bg-white' : 'border-transparent text-slate-500'}">
                BAKERY DESSERT
              </button>
              <button ${clickHandler("selectCategory('SPECIAL_PROMO')")} class="flex-1 py-2.5 px-1 font-bold border-b-2 text-center transition ${state.selectedCategory === 'SPECIAL_PROMO' ? 'border-amber-500 text-amber-700 bg-white' : 'border-transparent text-slate-500'}">
                PACKAGE PROMO
              </button>
            </div>

            <!-- 메뉴 리스트 (밝고 선명하게) -->
            <div class="flex-1 p-3 space-y-2.5 overflow-y-auto bg-slate-50/50">
              <div class="text-[10px] text-slate-400 font-semibold mb-1">SELECT ITEM CODE FOR CUSTOMIZATION:</div>
              ${items.map(item => `
                <div ${clickHandler(`openCustomize(${JSON.stringify(item).replace(/"/g, '&quot;')})`)} 
                     class="bg-white hover:bg-slate-50 p-3 rounded-xl border border-slate-200 flex justify-between items-center cursor-pointer active:scale-98 transition shadow-sm">
                  <div>
                    <div class="text-xs font-bold text-slate-900">${item.name}</div>
                    <div class="text-[10px] text-slate-500 mt-0.5 font-medium">${item.desc}</div>
                  </div>
                  <div class="text-right">
                    <span class="text-xs font-extrabold text-amber-600">₩${item.price.toLocaleString()}</span>
                    <span class="block text-[9px] text-indigo-600 font-mono font-bold">SELECT ></span>
                  </div>
                </div>
              `).join('')}
            </div>

            <!-- 장바구니 요약 하단 바 (밝은 스타일) -->
            <div class="bg-white p-3 border-t border-slate-200 flex justify-between items-center shadow-lg">
              <div>
                <span class="text-[10px] text-slate-400 font-semibold block">CURRENT ORDER UNITS</span>
                <span class="text-xs font-extrabold text-indigo-700">${state.cart.length} ITEMS IN CART</span>
              </div>
              <button ${clickHandler("proceedToPayment()")} class="bg-amber-500 hover:bg-amber-600 text-slate-950 px-4 py-2.5 rounded-xl font-extrabold text-xs shadow-md transition">
                ORDER PROCESS CHECKOUT
              </button>
            </div>
          </div>
        `;
      }

      // 2. 옵션 커스텀 화면 (밝은 테마)
      if (state.step === 'CUSTOMIZE') {
        const item = state.selectedItem;
        return `
          <div class="flex flex-col h-full bg-white text-slate-800 p-3">
            <div class="border-b border-slate-200 pb-2 mb-3">
              <span class="text-[9px] text-amber-600 font-bold uppercase">CUSTOMIZE SYNERGY MODIFIER</span>
              <h3 class="text-sm font-bold text-slate-900">${item.name}</h3>
            </div>

            <div class="flex-1 space-y-3.5 overflow-y-auto text-xs">
              <!-- 에스프레소 샷 추가 -->
              <div class="bg-slate-50 p-3 rounded-xl border border-slate-200 space-y-1.5 shadow-sm">
                <div class="flex justify-between items-center">
                  <span class="font-bold text-slate-800">EXTRA ESPRESSO CONCENTRATE</span>
                  <span class="text-[10px] text-amber-600 font-mono font-bold">+ ₩500 / Shot</span>
                </div>
                <div class="flex items-center space-x-3 mt-1">
                  <button ${clickHandler("changeShot(-1)")} class="w-8 h-8 bg-white rounded-lg text-slate-800 font-bold text-base flex items-center justify-center border border-slate-300 shadow-sm">-</button>
                  <span class="font-extrabold text-amber-600 text-sm px-2">${state.customizations.shots} SHOTS</span>
                  <button ${clickHandler("changeShot(1)")} class="w-8 h-8 bg-white rounded-lg text-slate-800 font-bold text-base flex items-center justify-center border border-slate-300 shadow-sm">+</button>
                </div>
              </div>

              <!-- 유제품 옵션 -->
              <div class="bg-slate-50 p-3 rounded-xl border border-slate-200 space-y-2 shadow-sm">
                <span class="font-bold text-slate-800 block">LACTOSE / BASE ALTERNATIVE</span>
                <div class="grid grid-cols-2 gap-2 text-[11px]">
                  <button ${clickHandler("setMilk('REGULAR')")} class="py-2 px-2 rounded-lg border text-center font-bold transition ${state.customizations.milk === 'REGULAR' ? 'bg-amber-500 text-white border-amber-500 shadow' : 'bg-white border-slate-300 text-slate-600'}">
                    ORIGINAL DAIRY
                  </button>
                  <button ${clickHandler("setMilk('SOY_SYNERGY')")} class="py-2 px-2 rounded-lg border text-center font-bold transition ${state.customizations.milk === 'SOY_SYNERGY' ? 'bg-amber-500 text-white border-amber-500 shadow' : 'bg-white border-slate-300 text-slate-600'}">
                    SOY BEAN EXTRACT
                  </button>
                </div>
              </div>

              <!-- 얼음 옵션 -->
              <div class="bg-slate-50 p-3 rounded-xl border border-slate-200 space-y-2 shadow-sm">
                <span class="font-bold text-slate-800 block">SOLID H2O RATIO CONTROL</span>
                <div class="grid grid-cols-2 gap-2 text-[11px]">
                  <button ${clickHandler("setIce('NORMAL')")} class="py-2 px-2 rounded-lg border text-center font-bold transition ${state.customizations.ice === 'NORMAL' ? 'bg-amber-500 text-white border-amber-500 shadow' : 'bg-white border-slate-300 text-slate-600'}">
                    STANDARD ICE
                  </button>
                  <button ${clickHandler("setIce('ICE_MINIMIZE')")} class="py-2 px-2 rounded-lg border text-center font-bold transition ${state.customizations.ice === 'ICE_MINIMIZE' ? 'bg-amber-500 text-white border-amber-500 shadow' : 'bg-white border-slate-300 text-slate-600'}">
                    MINIMIZE ICE UNITS
                  </button>
                </div>
              </div>
            </div>

            <!-- 하단 확인 / 취소 -->
            <div class="pt-3 border-t border-slate-200 flex gap-2">
              <button ${clickHandler("state.step='MENU'")} class="w-1/3 bg-slate-200 hover:bg-slate-300 text-slate-700 py-2.5 rounded-xl text-xs font-bold transition">
                PREVIOUS STEP
              </button>
              <button ${clickHandler("addToCart()")} class="w-2/3 bg-indigo-600 hover:bg-indigo-700 text-white py-2.5 rounded-xl text-xs font-extrabold shadow-md transition">
                CONFIRM SELECTION
              </button>
            </div>
          </div>
        `;
      }

      // 3. 장바구니 및 수령 방식/쿠폰 적용 화면 (밝은 테마)
      if (state.step === 'CART') {
        const totalAmount = state.cart.reduce((sum, c) => sum + c.item.price + (c.custom.shots * 500), 0);
        const finalPrice = state.appliedCoupon ? Math.max(0, totalAmount - 2000) : totalAmount;

        return `
          <div class="flex flex-col h-full bg-white text-slate-800 p-3">
            <div class="border-b border-slate-200 pb-2 mb-2">
              <h3 class="text-sm font-bold text-slate-900">CHECKOUT MATRIX & AUTHORIZATION</h3>
              <p class="text-[10px] text-slate-500">주문 내역과 인증 쿠폰을 조합하세요.</p>
            </div>

            <div class="flex-1 overflow-y-auto space-y-3">
              <!-- 장바구니 리스트 -->
              <div class="bg-slate-50 p-3 rounded-xl border border-slate-200 space-y-2 shadow-sm">
                <span class="text-[10px] text-slate-500 font-bold uppercase">SELECTED BEVERAGE UNITS</span>
                ${state.cart.map(c => `
                  <div class="flex justify-between items-start text-xs border-b border-slate-200 pb-2">
                    <div>
                      <div class="font-bold text-slate-900">${c.item.name}</div>
                      <div class="text-[10px] text-amber-700 font-medium">
                        Extra Shots: +${c.custom.shots} | Base: ${c.custom.milk} | Ice: ${c.custom.ice}
                      </div>
                    </div>
                    <span class="font-bold text-slate-900">₩${(c.item.price + c.custom.shots*500).toLocaleString()}</span>
                  </div>
                `).join('')}
              </div>

              <!-- 쿠폰 적용 버튼 -->
              <div class="bg-slate-50 p-3 rounded-xl border border-slate-200 space-y-1.5 shadow-sm">
                <span class="text-[10px] text-slate-500 font-bold block">DISCOUNT AUTHORIZATION CODE</span>
                <button ${clickHandler("toggleCoupon()")} class="w-full py-2.5 px-3 rounded-lg border text-xs font-bold transition flex justify-between items-center ${state.appliedCoupon ? 'bg-emerald-600 text-white border-emerald-600 shadow' : 'bg-white border-slate-300 text-slate-700'}">
                  <span>DESSERT COMBO AUTHORIZATION (-₩2,000)</span>
                  <i class="fa-solid ${state.appliedCoupon ? 'fa-check text-white' : 'fa-plus text-slate-400'}"></i>
                </button>
              </div>

              <!-- 수령 방식 선택 -->
              <div class="bg-slate-50 p-3 rounded-xl border border-slate-200 space-y-1.5 shadow-sm">
                <span class="text-[10px] text-slate-500 font-bold block">RECEIPT TYPE MODALITY</span>
                <div class="grid grid-cols-2 gap-2 text-xs">
                  <button ${clickHandler("setTakeout('STORE')")} class="py-2.5 rounded-lg border font-bold transition ${state.takeoutType === 'STORE' ? 'bg-indigo-600 text-white border-indigo-600 shadow' : 'bg-white border-slate-300 text-slate-700'}">
                    IN-STORE CONSUME
                  </button>
                  <button ${clickHandler("setTakeout('TOGO')")} class="py-2.5 rounded-lg border font-bold transition ${state.takeoutType === 'TOGO' ? 'bg-indigo-600 text-white border-indigo-600 shadow' : 'bg-white border-slate-300 text-slate-700'}">
                    TO-GO PACKAGING
                  </button>
                </div>
              </div>
            </div>

            <!-- 결제 금액 및 최종 진행 -->
            <div class="pt-3 border-t border-slate-200 space-y-2">
              <div class="flex justify-between items-center text-xs">
                <span class="text-slate-600 font-bold">FINAL TRANSACTION AMOUNT:</span>
                <span class="text-base font-extrabold text-amber-600">₩${finalPrice.toLocaleString()}</span>
              </div>
              <button ${clickHandler("proceedToPayment()")} class="w-full bg-amber-500 hover:bg-amber-600 text-slate-950 py-3 rounded-xl font-extrabold text-xs shadow-md transition">
                PROCEED TO TERMINAL TRANSACTION
              </button>
            </div>
          </div>
        `;
      }

      // 4. 결제 진행 단계 (밝은 라이트 테마)
      if (state.step === 'PAYMENT') {
        return `
          <div class="flex flex-col h-full bg-white text-slate-800 p-4 text-center justify-between">
            <div class="space-y-3 my-auto">
              <div class="text-indigo-600 text-4xl animate-pulse"><i class="fa-solid fa-credit-card"></i></div>
              <h3 class="text-sm font-bold text-slate-900">POS IC-CARD CONTACT TERMINAL</h3>
              <p class="text-[11px] text-slate-500 max-w-xs mx-auto">
                Insert IC Chip Card into the designated authorization slot at the bottom right corner.
              </p>
              <div class="bg-amber-50 p-3 rounded-xl border border-amber-200 text-[10px] text-amber-900 text-left space-y-1">
                <div>* Do not extract card during Transaction Authorization process.</div>
                <div>* IC칩 결제 모듈 승인 지연 시 반환 버튼을 누르지 마십시오.</div>
              </div>
            </div>

            <div class="space-y-2">
              <button ${clickHandler("finalizeOrder()")} class="w-full bg-emerald-600 hover:bg-emerald-700 text-white py-3 rounded-xl font-extrabold text-xs shadow-md transition">
                COMPLETE TRANSACTION DIRECT AUTHORIZE
              </button>
              <button ${clickHandler("state.step='MENU'")} class="w-full bg-slate-100 hover:bg-slate-200 text-slate-600 py-2 rounded-xl text-[10px] font-bold transition">
                RETURN TO INITIAL PROCESS
              </button>
            </div>
          </div>
        `;
      }

      return '';
    }
  </script>
</body>
</html>
