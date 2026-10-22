<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>디지털 소외 계층 키오스크 체험 - Light Single View Simulator</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- FontAwesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;600;700&display=swap');
    
    * {
      touch-action: manipulation; /* 모바일 터치 지연 방지 및 모바일 드래그/줌 씹힘 방지 */
      -webkit-tap-highlight-color: transparent; /* 터치 시 하이라이트 제거 */
    }

    body {
      font-family: 'Noto Sans KR', sans-serif;
      user-select: none;
      -webkit-user-select: none;
      background-color: #ffffff;
      color: #0f172a;
    }

    /* 고령층 시야 특수 효과 - 약한 Blur & 작은 글씨 */
    .elderly-view {
      filter: blur(0.8px) contrast(98%) brightness(102%);
      font-size: 0.7rem !important; /* 모바일 맞춤 글씨 */
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

  <!-- 상단 헤더 & 컨트롤 바 -->
  <header class="bg-slate-100 border-b border-slate-200 px-3 py-1.5 sm:px-4 sm:py-2.5 flex justify-between items-center z-40 shrink-0 shadow-sm min-h-[40px]">
    <div class="flex items-center space-x-2 sm:space-x-3">
      <span class="bg-indigo-600 text-white text-[10px] sm:text-xs font-bold px-2 py-0.5 sm:px-2.5 sm:py-1 rounded-full flex items-center gap-1 shrink-0">
        <i class="fa-solid fa-eye-low-vision"></i> <span class="hidden xs:inline">디지털 약자</span> 체감
      </span>
      <h1 class="text-xs sm:text-base font-bold tracking-tight text-slate-800 hidden md:block">고령층 키오스크 장벽 체감 체험</h1>
    </div>

    <!-- 미션 가이드 & 컨트롤 -->
    <div class="flex items-center space-x-1.5 sm:space-x-2">
      <button onclick="showMissionModal()" class="text-[10px] sm:text-xs bg-white hover:bg-slate-50 active:bg-slate-100 text-slate-700 px-2 py-1 sm:px-3 sm:py-1.5 rounded-lg transition border border-slate-300 font-medium flex items-center gap-1 shadow-sm touch-manipulation">
        <i class="fa-solid fa-clipboard-list text-amber-500"></i>
        <span>미션</span>
        <span id="missionCheckBadge" class="bg-amber-100 text-amber-800 text-[9px] sm:text-[10px] font-bold px-1.5 py-0.2 rounded-full">0</span>
      </button>
      <button onclick="resetExperience()" class="text-[10px] sm:text-xs bg-rose-50 hover:bg-rose-100 active:bg-rose-200 text-rose-700 px-2 py-1 sm:px-3 sm:py-1.5 rounded-lg transition border border-rose-200 font-semibold touch-manipulation">
        <i class="fa-solid fa-rotate-right"></i> <span class="hidden sm:inline">초기화</span>
      </button>
    </div>
  </header>

  <!-- 실시간 심리 압박 정보 바 -->
  <div class="bg-amber-50 border-b border-amber-200 px-3 py-1 sm:px-4 sm:py-1.5 text-[10px] sm:text-xs text-amber-900 flex justify-between items-center z-30 shrink-0 min-h-[28px]">
    <div class="flex items-center space-x-2 sm:space-x-4 justify-between w-full sm:w-auto">
      <span class="flex items-center gap-1 text-rose-700 font-bold urgent-blink">
        <i class="fa-solid fa-triangle-exclamation text-rose-600"></i>
        <span>제한시간: <span id="timerDisplay" class="text-xs sm:text-sm font-extrabold underline">300</span>초</span>
      </span>
      <span class="text-amber-800 font-medium">
        <i class="fa-solid fa-users text-amber-600"></i>
        대기: <strong id="waitingCount" class="text-rose-600 text-xs sm:text-sm">3</strong>명
      </span>
    </div>
    <div class="text-[10px] text-slate-500 hidden md:block">
      * 디지털 약자가 실제로 느끼는 시인성·반응지연·용어 장벽 시뮬레이터
    </div>
  </div>

  <!-- 메인 키오스크 영역 -->
  <main class="flex-1 flex items-center justify-center p-1 sm:p-3 bg-slate-50 overflow-hidden relative">
    <div class="w-full max-w-xs sm:max-w-md h-full max-h-[720px] bg-white rounded-xl sm:rounded-2xl border-2 sm:border-4 border-slate-300 flex flex-col overflow-hidden relative shadow-xl">
      <!-- 키오스크 타이틀 바 -->
      <div class="bg-indigo-50 px-2.5 py-1.5 sm:px-3 sm:py-2 text-[10px] sm:text-xs font-bold text-indigo-900 border-b border-indigo-100 flex justify-between items-center shrink-0">
        <span class="flex items-center gap-1 truncate"><i class="fa-solid fa-display text-indigo-600"></i> 고령층 체감 시야</span>
        <span class="text-[9px] sm:text-[10px] bg-indigo-100 text-indigo-700 px-1.5 py-0.5 rounded-full font-semibold shrink-0">체험 전용</span>
      </div>

      <!-- 키오스크 화면 프레임 -->
      <div id="kioskContainer" class="flex-1 overflow-y-auto no-scrollbar relative flex flex-col bg-white elderly-view elderly-tremor">
        <!-- JS 렌더링 -->
      </div>
    </div>

    <!-- 터치 반응 지연 레이어 (pointer-events-none 기본 설정으로 터치 차단 막음) -->
    <div id="delayOverlay" class="hidden pointer-events-none fixed inset-0 bg-slate-900/20 z-50 cursor-wait flex items-center justify-center backdrop-blur-[2px]">
      <div class="bg-white text-slate-800 border border-slate-300 px-3 py-2 sm:px-5 sm:py-3 rounded-xl text-xs shadow-2xl animate-pulse flex items-center gap-2 sm:gap-2.5 font-bold">
        <i class="fa-solid fa-spinner fa-spin text-sm sm:text-base text-indigo-600"></i>
        <span>시스템 처리 지연 중...</span>
      </div>
    </div>
  </main>

  <!-- [1단계] 체험 시작 전 준비 안내 화면 -->
  <div id="startModal" class="fixed inset-0 bg-slate-900/80 backdrop-blur-md z-50 flex items-center justify-center p-3">
    <div class="bg-white border-2 border-indigo-500 rounded-2xl max-w-sm sm:max-w-lg w-full p-5 sm:p-7 text-slate-800 shadow-2xl relative max-h-[90vh] overflow-y-auto">
      <div class="text-center space-y-2 mb-4 border-b border-slate-100 pb-4">
        <div class="w-12 h-12 sm:w-14 sm:h-14 bg-indigo-100 text-indigo-600 rounded-full flex items-center justify-center mx-auto text-xl sm:text-2xl shadow-inner">
          <i class="fa-solid fa-hands-holding-circle"></i>
        </div>
        <h2 class="text-lg sm:text-2xl font-black text-slate-900">디지털 약자 키오스크 체험</h2>
        <p class="text-xs sm:text-sm text-slate-600 font-medium">고령층이 매장 키오스크 앞에서 겪는 어려움을 직접 체감해보세요.</p>
      </div>

      <div class="bg-indigo-50/70 p-3.5 sm:p-4 rounded-xl border border-indigo-100 space-y-2.5 mb-5 text-xs sm:text-sm">
        <h3 class="font-bold text-indigo-950 flex items-center gap-1.5 text-xs sm:text-sm">
          <i class="fa-solid fa-triangle-exclamation text-amber-500"></i> 체감되는 제약 사항
        </h3>
        <ul class="space-y-1.5 text-slate-700 pl-1">
          <li class="flex items-start gap-2">
            <i class="fa-solid fa-check text-indigo-600 mt-0.5 text-xs"></i>
            <span><strong>시각·인지 제약:</strong> 흐린 시야, 작은 글씨, 생소한 영어 표기법</span>
          </li>
          <li class="flex items-start gap-2">
            <i class="fa-solid fa-check text-indigo-600 mt-0.5 text-xs"></i>
            <span><strong>반응 지연 & 미세 손떨림:</strong> 화면 터치 시 동작 응답 지연 발생</span>
          </li>
          <li class="flex items-start gap-2">
            <i class="fa-solid fa-check text-indigo-600 mt-0.5 text-xs"></i>
            <span><strong>심리적 압박:</strong> 10초 후 사라지는 주문 미션 & 대기 인원 증가</span>
          </li>
        </ul>
      </div>

      <button onclick="openMissionFromStart()" class="w-full bg-indigo-600 hover:bg-indigo-700 active:bg-indigo-800 text-white font-extrabold py-3 sm:py-3.5 rounded-xl text-xs sm:text-sm transition shadow-lg flex items-center justify-center gap-2 touch-manipulation">
        <span>체험 시작하기</span>
        <i class="fa-solid fa-arrow-right"></i>
      </button>
    </div>
  </div>

  <!-- [2단계] 10초 후 사라지는 미션 팝업 -->
  <div id="missionModal" class="hidden fixed inset-0 bg-slate-900/70 backdrop-blur-sm z-50 flex items-center justify-center p-3">
    <div class="bg-white border-2 border-amber-400 rounded-2xl max-w-sm sm:max-w-lg w-full p-4 sm:p-6 text-slate-800 shadow-2xl relative max-h-[90vh] overflow-y-auto">
      <div class="flex justify-between items-start mb-2 sm:mb-4 border-b border-slate-100 pb-2 sm:pb-3">
        <div>
          <span class="bg-amber-100 text-amber-800 text-[10px] sm:text-xs font-bold px-2 py-0.5 sm:px-2.5 sm:py-1 rounded-full border border-amber-200">기억력 체험 미션</span>
          <h2 class="text-base sm:text-xl font-bold mt-1 text-slate-900">주문할 메뉴를 기억하세요!</h2>
        </div>
        <div class="text-right shrink-0">
          <span id="modalCountdown" class="text-2xl sm:text-3xl font-extrabold text-rose-600">10</span>
          <p class="text-[9px] sm:text-[10px] text-slate-400">초 뒤 사라짐</p>
        </div>
      </div>

      <div class="bg-slate-50 p-2.5 sm:p-4 rounded-xl border border-slate-200 space-y-1.5 sm:space-y-2 mb-3 sm:mb-4">
        <p class="text-[11px] sm:text-xs text-slate-600 font-medium">다음 조건에 정확히 맞춰 키오스크로 주문하세요:</p>
        <div class="bg-amber-50 p-2.5 sm:p-3 rounded-lg border border-amber-200 text-amber-950 text-xs sm:text-sm font-semibold space-y-1">
          <p>1. <span class="text-slate-900 font-bold underline">아이스 아메리카노</span> 1잔 선택</p>
          <p>2. 옵션: <span class="text-amber-800 underline">에스프레소 Extra Shot 2회</span> 추가</p>
          <p>3. 옵션: <span class="text-amber-800 underline">두유(Soy Milk)</span> 변경 & <span class="text-amber-800 underline">얼음 최소화</span></p>
          <p>4. 쿠폰: <span class="text-amber-800 underline">디저트 콤보 할인 쿠폰</span> 적용</p>
          <p>5. 수령: <span class="text-slate-900 font-bold underline">To-Go (포장 수령)</span> 선택</p>
        </div>
      </div>

      <p class="text-[10px] sm:text-xs text-rose-800 bg-rose-50 p-2 sm:p-3 rounded-lg border border-rose-200 flex items-start gap-1.5">
        <i class="fa-solid fa-brain text-rose-600 mt-0.5 shrink-0"></i>
        <span>어르신들이 겪는 순간 건망증과 시각 정보 손실을 체험하기 위해 10초 후 숨겨집니다.</span>
      </p>

      <button onclick="startSessionNow()" class="mt-3 sm:mt-4 w-full bg-amber-500 hover:bg-amber-600 active:bg-amber-700 text-slate-950 font-bold py-2.5 sm:py-3 rounded-xl text-xs sm:text-sm transition shadow-md touch-manipulation">
        지금 바로 시작하기
      </button>
    </div>
  </div>

  <!-- [3단계] 체험 결과 / 리포트 팝업 -->
  <div id="resultModal" class="hidden fixed inset-0 bg-slate-900/80 backdrop-blur-sm z-50 flex items-center justify-center p-3 overflow-y-auto">
    <div class="bg-white border border-slate-200 rounded-2xl max-w-sm sm:max-w-lg w-full p-4 sm:p-6 text-slate-800 shadow-2xl space-y-3 sm:space-y-5 my-auto max-h-[90vh] overflow-y-auto">
      <div id="resultHeader" class="text-center space-y-0.5 sm:space-y-1">
        <!-- JS 다이나믹 생성 -->
      </div>

      <div class="bg-slate-50 p-3 sm:p-4 rounded-xl border border-slate-200 space-y-2 sm:space-y-3">
        <h4 class="text-[10px] sm:text-xs font-bold text-slate-500 uppercase tracking-wider border-b border-slate-200 pb-1">체험 및 조작 분석 리포트</h4>
        
        <div class="grid grid-cols-3 gap-1.5 sm:gap-2 text-center">
          <div class="bg-white p-2 sm:p-2.5 rounded-lg border border-slate-200 shadow-sm">
            <span class="text-[9px] sm:text-[10px] text-slate-500 block">총 소요 시간</span>
            <span id="resTime" class="text-xs sm:text-base font-extrabold text-amber-600">0초</span>
          </div>
          <div class="bg-white p-2 sm:p-2.5 rounded-lg border border-slate-200 shadow-sm">
            <span class="text-[9px] sm:text-[10px] text-slate-500 block">미션 재확인</span>
            <span id="resCheckCount" class="text-xs sm:text-base font-extrabold text-indigo-600">0회</span>
          </div>
          <div class="bg-white p-2 sm:p-2.5 rounded-lg border border-slate-200 shadow-sm">
            <span class="text-[9px] sm:text-[10px] text-slate-500 block">실수/방황</span>
            <span id="resMistakes" class="text-xs sm:text-base font-extrabold text-rose-600">0회</span>
          </div>
        </div>

        <div class="bg-white p-2.5 sm:p-3 rounded-lg border border-slate-200 space-y-1">
          <span class="text-[10px] sm:text-[11px] font-bold text-slate-700 block border-b border-slate-100 pb-1">주문 조건 세부 검증 결과</span>
          <div id="missionMatchDetails" class="text-[10px] sm:text-xs space-y-0.5 sm:space-y-1 font-medium">
            <!-- JS 다이나믹 생성 -->
          </div>
        </div>

        <div class="space-y-1">
          <div class="flex justify-between text-[10px] sm:text-xs">
            <span class="text-slate-600 font-medium">디지털 피로도 및 인지 부담:</span>
            <span id="stressLevelText" class="font-bold text-rose-600">높음</span>
          </div>
          <div class="w-full bg-slate-200 h-2 rounded-full overflow-hidden">
            <div id="stressBar" class="bg-gradient-to-r from-amber-500 to-rose-600 h-full w-[80%]"></div>
          </div>
        </div>
      </div>

      <div class="bg-indigo-50 border border-indigo-200 p-3 sm:p-4 rounded-xl text-[11px] sm:text-xs space-y-1.5 text-indigo-950">
        <h4 class="font-bold text-indigo-900 flex items-center gap-1.5 text-xs sm:text-sm">
          <i class="fa-solid fa-heart-pulse text-indigo-600"></i> 함께 생각해요
        </h4>
        <p class="leading-relaxed text-slate-700">
          이 답답함은 <strong>고령층 및 디지털 약자가 매일 일상에서 느끼는 실제 무력감</strong>입니다.
        </p>
      </div>

      <button onclick="resetExperience()" class="w-full bg-indigo-600 hover:bg-indigo-700 active:bg-indigo-800 text-white font-bold py-2.5 sm:py-3 rounded-xl text-xs sm:text-sm transition shadow-lg touch-manipulation">
        다시 체험해보기
      </button>
    </div>
  </div>

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
      checkMissionCount: 0,
      startTime: null,
      timerInterval: null,
      isBusy: false // 클릭 처리 중 중복 처리 방지 플래그
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

    // 모바일 응답 지연 시뮬레이터 (지연 중 터치 이벤트 안정화)
    function handleAction(actionFn) {
      if (state.isBusy) return;
      state.isBusy = true;

      const overlay = document.getElementById('delayOverlay');
      if (overlay) overlay.classList.remove('hidden');

      // 400ms 지연 후 실행 (고령층 터치 지연 시뮬레이션)
      setTimeout(() => {
        try {
          actionFn();
        } catch (e) {
          console.error(e);
        } finally {
          if (overlay) overlay.classList.add('hidden');
          state.isBusy = false;
        }
      }, 400);
    }

    let missionCountdown = 10;
    let missionInterval = null;

    window.onload = () => {
      // 명확한 초기 화면 설정: 시작 모달만 오픈
      document.getElementById('startModal').classList.remove('hidden');
      document.getElementById('missionModal').classList.add('hidden');
      document.getElementById('resultModal').classList.add('hidden');
      renderKiosk();
    };

    function openMissionFromStart() {
      document.getElementById('startModal').classList.add('hidden');
      document.getElementById('missionModal').classList.remove('hidden');
      startMissionTimer();
    }

    function startMissionTimer() {
      missionCountdown = 10;
      const cdEl = document.getElementById('modalCountdown');
      if (cdEl) cdEl.innerText = missionCountdown;

      if (missionInterval) clearInterval(missionInterval);
      
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
      
      if (!state.timerInterval) {
        state.timerInterval = setInterval(() => {
          state.timeLeft--;
          const timerEl = document.getElementById('timerDisplay');
          if (timerEl) timerEl.innerText = state.timeLeft;
          
          if (state.timeLeft % 30 === 0) {
            state.waitingCount++;
            const waitEl = document.getElementById('waitingCount');
            if (waitEl) waitEl.innerText = state.waitingCount;
          }

          if (state.timeLeft <= 0) {
            clearInterval(state.timerInterval);
            finishExperience(true);
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

    // --- ACTIONS & NAVIGATION ---
    function selectCategory(cat) {
      state.selectedCategory = cat;
      renderKiosk();
    }

    function openCustomize(item) {
      state.selectedItem = item;
      state.customizations = { shots: 0, milk: 'REGULAR', ice: 'NORMAL' };
      state.step = 'CUSTOMIZE';
      renderKiosk();
    }

    function changeShot(delta) {
      state.customizations.shots = Math.max(0, state.customizations.shots + delta);
      renderKiosk();
    }

    function setMilk(milkType) {
      state.customizations.milk = milkType;
      renderKiosk();
    }

    function setIce(iceType) {
      state.customizations.ice = iceType;
      renderKiosk();
    }

    function addToCart() {
      state.cart = [{ item: state.selectedItem, custom: { ...state.customizations } }];
      state.step = 'CART';
      renderKiosk();
    }

    function toggleCoupon() {
      state.appliedCoupon = !state.appliedCoupon;
      renderKiosk();
    }

    function setTakeout(type) {
      state.takeoutType = type;
      renderKiosk();
    }

    function proceedToPayment() {
      if (state.cart.length === 0) {
        state.mistakes++;
        return;
      }
      state.step = 'PAYMENT';
      renderKiosk();
    }

    function finalizeOrder() {
      finishExperience(false);
    }

    function finishExperience(isTimeoutFail = false) {
      clearInterval(state.timerInterval);
      const totalTime = Math.round((Date.now() - (state.startTime || Date.now())) / 1000);

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
          <div class="text-emerald-500 text-3xl mb-0.5"><i class="fa-solid fa-circle-check"></i></div>
          <h3 class="text-lg font-extrabold text-slate-900">주문 미션 완벽 성공!</h3>
          <p class="text-[10px] text-slate-500">모든 옵션 조건을 정확하게 입력했습니다.</p>
        `;
      } else {
        header.innerHTML = `
          <div class="text-rose-500 text-3xl mb-0.5"><i class="fa-solid fa-circle-xmark"></i></div>
          <h3 class="text-lg font-extrabold text-slate-900">${isTimeoutFail ? '시간 초과 실패' : '주문 조건 불일치'}</h3>
          <p class="text-[10px] text-slate-500">어르신들이 복잡한 키오스크에서 느끼는 체감을 완료했습니다.</p>
        `;
      }

      document.getElementById('resTime').innerText = totalTime + "초";
      document.getElementById('resCheckCount').innerText = state.checkMissionCount + "회";
      document.getElementById('resMistakes').innerText = state.mistakes + "회";

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

    // --- UI RENDER BUILDER ---
    function renderKiosk() {
      const container = document.getElementById('kioskContainer');
      if (container) container.innerHTML = buildKioskHTML();
    }

    function buildKioskHTML() {
      const clickHandler = (fnStr) => `onclick="handleAction(() => { ${fnStr} })"`;

      if (state.step === 'MENU') {
        const items = menuData[state.selectedCategory] || [];
        return `
          <div class="flex flex-col h-full bg-white text-slate-800">
            <div class="bg-indigo-600 p-2 sm:p-3 text-center text-white shrink-0">
              <span class="text-[8px] sm:text-[9px] tracking-widest text-indigo-200 uppercase font-bold">POS DIRECT ORDER SYSTEM v4.2</span>
              <h2 class="text-[11px] sm:text-xs font-extrabold text-white mt-0.5">BEVERAGE & DESERT SYNERGY STORE</h2>
            </div>

            <div class="flex bg-slate-100 border-b border-slate-200 text-[10px] sm:text-[11px] shrink-0">
              <button ${clickHandler("selectCategory('BEVERAGE')")} class="flex-1 py-2 sm:py-2.5 px-1 font-bold border-b-2 text-center transition touch-manipulation ${state.selectedCategory === 'BEVERAGE' ? 'border-amber-500 text-amber-700 bg-white' : 'border-transparent text-slate-500'}">
                COFFEE
              </button>
              <button ${clickHandler("selectCategory('DESERT')")} class="flex-1 py-2 sm:py-2.5 px-1 font-bold border-b-2 text-center transition touch-manipulation ${state.selectedCategory === 'DESERT' ? 'border-amber-500 text-amber-700 bg-white' : 'border-transparent text-slate-500'}">
                DESSERT
              </button>
              <button ${clickHandler("selectCategory('SPECIAL_PROMO')")} class="flex-1 py-2 sm:py-2.5 px-1 font-bold border-b-2 text-center transition touch-manipulation ${state.selectedCategory === 'SPECIAL_PROMO' ? 'border-amber-500 text-amber-700 bg-white' : 'border-transparent text-slate-500'}">
                PROMO
              </button>
            </div>

            <div class="flex-1 p-2 sm:p-3 space-y-2 sm:space-y-2.5 overflow-y-auto bg-slate-50/50">
              <div class="text-[9px] sm:text-[10px] text-slate-400 font-semibold mb-1">SELECT ITEM CODE FOR CUSTOMIZATION:</div>
              ${items.map(item => `
                <div ${clickHandler(`openCustomize(${JSON.stringify(item).replace(/"/g, '&quot;')})`)} 
                     class="bg-white hover:bg-slate-50 active:bg-slate-100 p-2.5 sm:p-3 rounded-xl border border-slate-200 flex justify-between items-center cursor-pointer active:scale-98 transition shadow-sm touch-manipulation">
                  <div>
                    <div class="text-xs font-bold text-slate-900">${item.name}</div>
                    <div class="text-[9px] sm:text-[10px] text-slate-500 mt-0.5 font-medium">${item.desc}</div>
                  </div>
                  <div class="text-right shrink-0">
                    <span class="text-xs font-extrabold text-amber-600">₩${item.price.toLocaleString()}</span>
                    <span class="block text-[8px] sm:text-[9px] text-indigo-600 font-mono font-bold">SELECT ></span>
                  </div>
                </div>
              `).join('')}
            </div>

            <div class="bg-white p-2.5 sm:p-3 border-t border-slate-200 flex justify-between items-center shadow-lg shrink-0">
              <div>
                <span class="text-[9px] sm:text-[10px] text-slate-400 font-semibold block">CURRENT ORDER UNITS</span>
                <span class="text-xs font-extrabold text-indigo-700">${state.cart.length} ITEMS IN CART</span>
              </div>
              <button ${clickHandler("proceedToPayment()")} class="bg-amber-500 hover:bg-amber-600 active:bg-amber-700 text-slate-950 px-3 py-2 sm:px-4 sm:py-2.5 rounded-xl font-extrabold text-xs shadow-md transition touch-manipulation">
                ORDER CHECKOUT
              </button>
            </div>
          </div>
        `;
      }

      if (state.step === 'CUSTOMIZE') {
        const item = state.selectedItem;
        return `
          <div class="flex flex-col h-full bg-white text-slate-800 p-2.5 sm:p-3">
            <div class="border-b border-slate-200 pb-1.5 sm:pb-2 mb-2 sm:mb-3 shrink-0">
              <span class="text-[8px] sm:text-[9px] text-amber-600 font-bold uppercase">CUSTOMIZE SYNERGY MODIFIER</span>
              <h3 class="text-xs sm:text-sm font-bold text-slate-900 truncate">${item.name}</h3>
            </div>

            <div class="flex-1 space-y-2.5 sm:space-y-3.5 overflow-y-auto text-xs">
              <div class="bg-slate-50 p-2.5 sm:p-3 rounded-xl border border-slate-200 space-y-1 sm:space-y-1.5 shadow-sm">
                <div class="flex justify-between items-center">
                  <span class="font-bold text-slate-800 text-[10px] sm:text-xs">EXTRA ESPRESSO CONCENTRATE</span>
                  <span class="text-[9px] sm:text-[10px] text-amber-600 font-mono font-bold">+ ₩500 / Shot</span>
                </div>
                <div class="flex items-center space-x-2 sm:space-x-3 mt-1">
                  <button ${clickHandler("changeShot(-1)")} class="w-8 h-8 sm:w-9 sm:h-9 bg-white rounded-lg text-slate-800 font-bold text-base flex items-center justify-center border border-slate-300 shadow-sm active:bg-slate-100 touch-manipulation">-</button>
                  <span class="font-extrabold text-amber-600 text-xs sm:text-sm px-1 sm:px-2">${state.customizations.shots} SHOTS</span>
                  <button ${clickHandler("changeShot(1)")} class="w-8 h-8 sm:w-9 sm:h-9 bg-white rounded-lg text-slate-800 font-bold text-base flex items-center justify-center border border-slate-300 shadow-sm active:bg-slate-100 touch-manipulation">+</button>
                </div>
              </div>

              <div class="bg-slate-50 p-2.5 sm:p-3 rounded-xl border border-slate-200 space-y-1.5 sm:space-y-2 shadow-sm">
                <span class="font-bold text-slate-800 block text-[10px] sm:text-xs">LACTOSE / BASE ALTERNATIVE</span>
                <div class="grid grid-cols-2 gap-1.5 sm:gap-2 text-[10px] sm:text-[11px]">
                  <button ${clickHandler("setMilk('REGULAR')")} class="py-2 px-1 sm:px-2 rounded-lg border text-center font-bold transition touch-manipulation ${state.customizations.milk === 'REGULAR' ? 'bg-amber-500 text-white border-amber-500 shadow' : 'bg-white border-slate-300 text-slate-600'}">
                    ORIGINAL DAIRY
                  </button>
                  <button ${clickHandler("setMilk('SOY_SYNERGY')")} class="py-2 px-1 sm:px-2 rounded-lg border text-center font-bold transition touch-manipulation ${state.customizations.milk === 'SOY_SYNERGY' ? 'bg-amber-500 text-white border-amber-500 shadow' : 'bg-white border-slate-300 text-slate-600'}">
                    SOY BEAN EXTRACT
                  </button>
                </div>
              </div>

              <div class="bg-slate-50 p-2.5 sm:p-3 rounded-xl border border-slate-200 space-y-1.5 sm:space-y-2 shadow-sm">
                <span class="font-bold text-slate-800 block text-[10px] sm:text-xs">SOLID H2O RATIO CONTROL</span>
                <div class="grid grid-cols-2 gap-1.5 sm:gap-2 text-[10px] sm:text-[11px]">
                  <button ${clickHandler("setIce('NORMAL')")} class="py-2 px-1 sm:px-2 rounded-lg border text-center font-bold transition touch-manipulation ${state.customizations.ice === 'NORMAL' ? 'bg-amber-500 text-white border-amber-500 shadow' : 'bg-white border-slate-300 text-slate-600'}">
                    STANDARD ICE
                  </button>
                  <button ${clickHandler("setIce('ICE_MINIMIZE')")} class="py-2 px-1 sm:px-2 rounded-lg border text-center font-bold transition touch-manipulation ${state.customizations.ice === 'ICE_MINIMIZE' ? 'bg-amber-500 text-white border-amber-500 shadow' : 'bg-white border-slate-300 text-slate-600'}">
                    MINIMIZE ICE
                  </button>
                </div>
              </div>
            </div>

            <div class="pt-2 sm:pt-3 border-t border-slate-200 flex gap-2 shrink-0">
              <button ${clickHandler("state.step='MENU'; renderKiosk();")} class="w-1/3 bg-slate-200 hover:bg-slate-300 active:bg-slate-400 text-slate-700 py-2 sm:py-2.5 rounded-xl text-xs font-bold transition touch-manipulation">
                PREVIOUS
              </button>
              <button ${clickHandler("addToCart()")} class="w-2/3 bg-indigo-600 hover:bg-indigo-700 active:bg-indigo-800 text-white py-2 sm:py-2.5 rounded-xl text-xs font-extrabold shadow-md transition touch-manipulation">
                CONFIRM
              </button>
            </div>
          </div>
        `;
      }

      if (state.step === 'CART') {
        const totalAmount = state.cart.reduce((sum, c) => sum + c.item.price + (c.custom.shots * 500), 0);
        const finalPrice = state.appliedCoupon ? Math.max(0, totalAmount - 2000) : totalAmount;

        return `
          <div class="flex flex-col h-full bg-white text-slate-800 p-2.5 sm:p-3">
            <div class="border-b border-slate-200 pb-1.5 sm:pb-2 mb-2 shrink-0">
              <h3 class="text-xs sm:text-sm font-bold text-slate-900">CHECKOUT MATRIX & AUTHORIZATION</h3>
            </div>

            <div class="flex-1 overflow-y-auto space-y-2 sm:space-y-3">
              <div class="bg-slate-50 p-2.5 sm:p-3 rounded-xl border border-slate-200 space-y-1.5 sm:space-y-2 shadow-sm">
                <span class="text-[9px] sm:text-[10px] text-slate-500 font-bold uppercase">SELECTED BEVERAGE UNITS</span>
                ${state.cart.map(c => `
                  <div class="flex justify-between items-start text-xs border-b border-slate-200 pb-2">
                    <div>
                      <div class="font-bold text-slate-900">${c.item.name}</div>
                      <div class="text-[9px] sm:text-[10px] text-amber-700 font-medium">
                        Extra Shots: +${c.custom.shots} | Base: ${c.custom.milk} | Ice: ${c.custom.ice}
                      </div>
                    </div>
                    <span class="font-bold text-slate-900">₩${(c.item.price + c.custom.shots*500).toLocaleString()}</span>
                  </div>
                `).join('')}
              </div>

              <div class="bg-slate-50 p-2.5 sm:p-3 rounded-xl border border-slate-200 space-y-1 shadow-sm">
                <span class="text-[9px] sm:text-[10px] text-slate-500 font-bold block">DISCOUNT CODE AUTHORIZATION</span>
                <button ${clickHandler("toggleCoupon()")} class="w-full py-2 px-2 rounded-lg border text-[10px] sm:text-xs font-bold transition flex justify-between items-center touch-manipulation ${state.appliedCoupon ? 'bg-emerald-600 text-white border-emerald-600 shadow' : 'bg-white border-slate-300 text-slate-700'}">
                  <span>DESSERT COMBO AUTHORIZATION (-₩2,000)</span>
                  <i class="fa-solid ${state.appliedCoupon ? 'fa-check text-white' : 'fa-plus text-slate-400'}"></i>
                </button>
              </div>

              <div class="bg-slate-50 p-2.5 sm:p-3 rounded-xl border border-slate-200 space-y-1 shadow-sm">
                <span class="text-[9px] sm:text-[10px] text-slate-500 font-bold block">RECEIPT TYPE MODALITY</span>
                <div class="grid grid-cols-2 gap-1.5 sm:gap-2 text-[10px] sm:text-xs">
                  <button ${clickHandler("setTakeout('STORE')")} class="py-2 rounded-lg border font-bold transition touch-manipulation ${state.takeoutType === 'STORE' ? 'bg-indigo-600 text-white border-indigo-600 shadow' : 'bg-white border-slate-300 text-slate-700'}">
                    IN-STORE CONSUME
                  </button>
                  <button ${clickHandler("setTakeout('TOGO')")} class="py-2 rounded-lg border font-bold transition touch-manipulation ${state.takeoutType === 'TOGO' ? 'bg-indigo-600 text-white border-indigo-600 shadow' : 'bg-white border-slate-300 text-slate-700'}">
                    TO-GO PACKAGING
                  </button>
                </div>
              </div>
            </div>

            <div class="pt-2 sm:pt-3 border-t border-slate-200 space-y-1.5 shrink-0">
              <div class="flex justify-between items-center text-xs">
                <span class="text-slate-600 font-bold">TOTAL AMOUNT:</span>
                <span class="text-sm sm:text-base font-extrabold text-amber-600">₩${finalPrice.toLocaleString()}</span>
              </div>
              <button ${clickHandler("proceedToPayment()")} class="w-full bg-amber-500 hover:bg-amber-600 active:bg-amber-700 text-slate-950 py-2.5 rounded-xl font-extrabold text-xs shadow-md transition touch-manipulation">
                PROCEED TRANSACTION
              </button>
            </div>
          </div>
        `;
      }

      if (state.step === 'PAYMENT') {
        return `
          <div class="flex flex-col h-full bg-white text-slate-800 p-3 text-center justify-between">
            <div class="space-y-2 my-auto">
              <div class="text-indigo-600 text-3xl sm:text-4xl animate-pulse"><i class="fa-solid fa-credit-card"></i></div>
              <h3 class="text-xs sm:text-sm font-bold text-slate-900">POS IC-CARD TERMINAL</h3>
              <p class="text-[10px] sm:text-xs text-slate-500 max-w-xs mx-auto">
                Insert IC Chip Card into the designated authorization slot.
              </p>
            </div>

            <div class="space-y-1.5 shrink-0">
              <button ${clickHandler("finalizeOrder()")} class="w-full bg-emerald-600 hover:bg-emerald-700 active:bg-emerald-800 text-white py-2.5 rounded-xl font-extrabold text-xs shadow-md transition touch-manipulation">
                COMPLETE TRANSACTION DIRECT
              </button>
              <button ${clickHandler("state.step='MENU'; renderKiosk();")} class="w-full bg-slate-100 hover:bg-slate-200 active:bg-slate-300 text-slate-600 py-1.5 rounded-xl text-[10px] font-bold transition touch-manipulation">
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
