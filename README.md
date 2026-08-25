# -1.github.io
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>고령층 디지털 소외 체감 시뮬레이터</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Apple SD Gothic Neo', 'Malgun Gothic', sans-serif;
    }

    body {
      background-color: #f0f2f5;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 20px 10px;
      min-height: 100vh;
    }

    /* 상단 컨트롤 패널 */
    .control-panel {
      background: #ffffff;
      padding: 15px;
      border-radius: 12px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
      width: 100%;
      max-width: 360px;
      margin-bottom: 15px;
      text-align: center;
    }

    .control-panel h2 {
      font-size: 16px;
      margin-bottom: 10px;
      color: #333;
    }

    .toggle-group {
      display: flex;
      justify-content: space-around;
      gap: 10px;
    }

    .toggle-btn {
      flex: 1;
      padding: 8px 5px;
      font-size: 12px;
      border: 1px solid #ddd;
      border-radius: 6px;
      background: #f8f9fa;
      cursor: pointer;
      font-weight: bold;
      transition: all 0.2s;
    }

    .toggle-btn.active {
      background: #d9534f;
      color: white;
      border-color: #d9534f;
    }

    /* 스마트폰 프레임 */
    .phone-container {
      width: 100%;
      max-width: 360px;
      height: 580px;
      background: #ffffff;
      border: 10px solid #222;
      border-radius: 36px;
      position: relative;
      overflow: hidden;
      box-shadow: 0 10px 20px rgba(0,0,0,0.15);
    }

    /* 노화 필터 효과 */
    .phone-screen {
      width: 100%;
      height: 100%;
      overflow-y: auto;
      transition: filter 0.3s;
    }

    /* [시각 제약] 흐릿함 + 저대비 */
    .aging-vision {
      filter: blur(2.2px) contrast(65%) brightness(95%);
    }

    /* 화면 상단 미션 안내 */
    .mission-header {
      background: #2b3e50;
      color: #fff;
      padding: 12px;
      font-size: 12px;
      text-align: center;
      line-height: 1.4;
    }

    .mission-header span {
      color: #ffeb3b;
      font-weight: bold;
    }

    /* 미션 1: 키오스크 화면 */
    .kiosk-app {
      padding: 15px;
    }

    .app-title {
      font-size: 14px;
      font-weight: bold;
      margin-bottom: 10px;
      text-align: center;
      color: #444;
    }

    .menu-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 8px;
      margin-bottom: 15px;
    }

    .menu-item {
      border: 1px solid #eee;
      border-radius: 8px;
      padding: 8px;
      text-align: center;
      background: #fafafa;
    }

    /* 노년층이 보기 어려운 저대비/작은 글씨 UI 디자인 */
    .menu-item .title {
      font-size: 10px; /* 매우 작은 글씨 */
      color: #999; /* 낮은 대비 */
      margin-bottom: 4px;
    }

    .menu-item .price {
      font-size: 10px;
      color: #bbb;
    }

    .option-box {
      background: #f5f5f5;
      padding: 10px;
      border-radius: 8px;
      margin-top: 10px;
    }

    .option-title {
      font-size: 10px;
      color: #888;
      margin-bottom: 6px;
    }

    .option-btn {
      width: 100%;
      padding: 6px;
      margin: 3px 0;
      font-size: 10px;
      background: #e0e0e0;
      color: #777;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }

    .order-btn {
      width: 100%;
      padding: 12px;
      background: #ccc;
      color: #fff;
      font-size: 11px;
      border: none;
      border-radius: 6px;
      margin-top: 10px;
      cursor: pointer;
    }

    /* [신체 제약] 손떨림 효과 애니메이션 */
    @keyframes tremor {
      0% { transform: translate(0, 0); }
      25% { transform: translate(2px, -2px); }
      50% { transform: translate(-2px, 1px); }
      75% { transform: translate(1px, 2px); }
      100% { transform: translate(-1px, -1px); }
    }

    .hand-tremor .tremor-target {
      animation: tremor 0.15s infinite;
    }

    /* 완료 모달 */
    .modal {
      display: none;
      position: absolute;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.7);
      justify-content: center;
      align-items: center;
      padding: 20px;
    }

    .modal-content {
      background: white;
      padding: 20px;
      border-radius: 12px;
      text-align: center;
    }

    .modal-content h3 {
      color: #2e7d32;
      margin-bottom: 10px;
    }

    .modal-content p {
      font-size: 12px;
      color: #555;
      line-height: 1.5;
      margin-bottom: 15px;
    }

    .modal-content button {
      padding: 8px 16px;
      background: #2196F3;
      color: white;
      border: none;
      border-radius: 4px;
    }
  </style>
</head>
<body>

  <div class="control-panel">
    <h2>노년층 신체 제약 모드 설정</h2>
    <div class="toggle-group">
      <button class="toggle-btn active" id="visionBtn" onclick="toggleVision()">시각 제약 ON</button>
      <button class="toggle-btn active" id="tremorBtn" onclick="toggleTremor()">손떨림 ON</button>
    </div>
  </div>

  <div class="phone-container">
    <div id="phoneScreen" class="phone-screen aging-vision hand-tremor">
      
      <div class="mission-header">
        [미션] <span>'디카페인 카페라떼'</span>를 찾아서<br>
        <span>'두유 변경'</span> 옵션을 선택하여 주문하세요!
      </div>

      <div class="kiosk-app">
        <div class="app-title">카페 무인 주문 시스템</div>
        
        <div class="menu-grid">
          <div class="menu-item tremor-target" onclick="wrongSelect('아메리카노')">
            <div class="title">아메리카노(HOT)</div>
            <div class="price">3,000원</div>
          </div>
          <div class="menu-item tremor-target" onclick="wrongSelect('카페라떼')">
            <div class="title">카페라떼(ICE)</div>
            <div class="price">3,500원</div>
          </div>
          <div class="menu-item tremor-target" onclick="selectRightMenu()">
            <div class="title">디카페인 라떼</div>
            <div class="price">4,000원</div>
          </div>
          <div class="menu-item tremor-target" onclick="wrongSelect('바닐라라떼')">
            <div class="title">바닐라라떼(ICE)</div>
            <div class="price">4,500원</div>
          </div>
        </div>

        <div class="option-box" id="optionBox" style="opacity: 0.4; pointer-events: none;">
          <div class="option-title">세부 커스텀 옵션 선택</div>
          <button class="option-btn tremor-target" onclick="wrongOption('우유 유지를')">기본 우유 선택</button>
          <button class="option-btn tremor-target" id="soyBtn" onclick="selectSoyOption()">두유로 변경 (+500원)</button>
          <button class="option-btn tremor-target" onclick="wrongOption('저지방 우유를')">저지방 우유 변경</button>
        </div>

        <button class="order-btn tremor-target" id="payBtn" onclick="completeMission()" disabled>결제하기</button>
      </div>

      <div class="modal" id="successModal">
        <div class="modal-content">
          <h3>미션 성공!</h3>
          <p>
            흐릿한 화면과 흔들리는 손으로 원하는 메뉴와 옵션을 찾는 것이 얼마나 답답하셨나요?<br><br>
            노년층은 매일 이러한 기술적·신체적 장벽을 겪고 있습니다.
          </p>
          <button onclick="resetMission()">다시 시도하기</button>
        </div>
      </div>

    </div>
  </div>

  <script>
    let isMenuSelected = false;
    let isOptionSelected = false;

    // 시각 제약 토글
    function toggleVision() {
      const screen = document.getElementById('phoneScreen');
      const btn = document.getElementById('visionBtn');
      screen.classList.toggle('aging-vision');
      btn.classList.toggle('active');
      btn.innerText = screen.classList.contains('aging-vision') ? '시각 제약 ON' : '시각 제약 OFF';
    }

    // 손떨림 제약 토글
    function toggleTremor() {
      const screen = document.getElementById('phoneScreen');
      const btn = document.getElementById('tremorBtn');
      screen.classList.toggle('hand-tremor');
      btn.classList.toggle('active');
      btn.innerText = screen.classList.contains('hand-tremor') ? '손떨림 ON' : '손떨림 OFF';
    }

    // 메뉴 선택 로직
    function selectRightMenu() {
      isMenuSelected = true;
      const optionBox = document.getElementById('optionBox');
      optionBox.style.opacity = '1';
      optionBox.style.pointerEvents = 'auto';
      alert("디카페인 라떼가 선택되었습니다. 옵션을 선택하세요.");
    }

    function wrongSelect(menuName) {
      alert("잘못 선택하셨습니다. '" + menuName + "' 대신 '디카페인 라떼'를 찾아보세요.");
    }

    // 옵션 선택 로직
    function selectSoyOption() {
      if(!isMenuSelected) return;
      isOptionSelected = true;
      const soyBtn = document.getElementById('soyBtn');
      const payBtn = document.getElementById('payBtn');
      soyBtn.style.background = '#4CAF50';
      soyBtn.style.color = '#fff';
      payBtn.style.background = '#2196F3';
      payBtn.disabled = false;
      alert("두유 변경 옵션이 선택되었습니다. 하단의 결제하기 버튼을 누르세요.");
    }

    function wrongOption(optName) {
      alert("잘못 선택하셨습니다. " + optName + " 아닌 '두유로 변경'을 선택해야 합니다.");
    }

    // 미션 완료
    function completeMission() {
      if(isMenuSelected && isOptionSelected) {
        document.getElementById('successModal').style.display = 'flex';
      }
    }

    // 리셋
    function resetMission() {
      isMenuSelected = false;
      isOptionSelected = false;
      document.getElementById('successModal').style.display = 'none';
      document.getElementById('optionBox').style.opacity = '0.4';
      document.getElementById('optionBox').style.pointerEvents = 'none';
      
      const soyBtn = document.getElementById('soyBtn');
      const payBtn = document.getElementById('payBtn');
      soyBtn.style.background = '#e0e0e0';
      soyBtn.style.color = '#777';
      payBtn.style.background = '#ccc';
      payBtn.disabled = true;
    }
  </script>
</body>
</html>
