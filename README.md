<!DOCTYPE html>
<html lang="ko">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>분당·수지 교구 검색기</title>
  <script src="https://t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js"></script>
  <style>
    :root {
      --primary: #4f46e5;
      --primary-light: #eef2ff;
      --surface: #ffffff;
      --text: #1e293b;
      --text-secondary: #64748b;
      --border: #e2e8f0;
      --radius: 16px;
    }

    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      font-family: 'Pretendard', 'Noto Sans KR', -apple-system, sans-serif;
      background: #f1f5f9;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    .container {
      background: var(--surface);
      border-radius: var(--radius);
      padding: 40px 32px;
      max-width: 600px;
      width: 100%;
      box-shadow: 0 20px 50px rgba(0, 0, 0, 0.08);
      border: 1px solid var(--border);
    }

    h1 {
      font-size: 28px;
      font-weight: 800;
      color: var(--text);
      text-align: center;
      margin-bottom: 8px;
    }

    .subtitle {
      text-align: center;
      color: var(--text-secondary);
      margin-bottom: 32px;
      font-size: 14px;
    }

    .btn-area {
      text-align: center;
      margin-bottom: 24px;
    }

    .search-btn {
      width: 100%;
      background: var(--primary);
      color: white;
      border: none;
      padding: 18px 32px;
      border-radius: 12px;
      font-size: 18px;
      font-weight: 700;
      cursor: pointer;
      transition: all 0.2s;
      letter-spacing: -0.3px;
    }

    .search-btn:hover {
      background: #4338ca;
    }

    .search-btn:active {
      transform: scale(0.98);
    }

    .result-card {
      background: #fafbff;
      border-radius: 12px;
      padding: 24px;
      border: 1px solid #e0e7ff;
      display: none;
    }

    .result-card.show {
      display: block;
    }

    .result-title-row {
      display: flex;
      align-items: center;
      gap: 10px;
      margin-bottom: 16px;
    }

    .result-icon {
      width: 40px;
      height: 40px;
      background: var(--primary-light);
      border-radius: 10px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 20px;
    }

    .result-district {
      font-size: 22px;
      font-weight: 800;
      color: var(--text);
    }

    .result-pastor {
      font-size: 16px;
      font-weight: 600;
      color: var(--primary);
    }

    .result-detail {
      font-size: 13px;
      color: var(--text-secondary);
      line-height: 1.8;
      background: #fff;
      border-radius: 10px;
      padding: 16px;
      border: 1px solid var(--border);
    }

    .badge {
      display: inline-block;
      background: var(--primary-light);
      color: var(--primary);
      padding: 3px 10px;
      border-radius: 20px;
      font-size: 12px;
      font-weight: 700;
      margin: 2px;
    }

    .badge-success {
      background: #d1fae5;
      color: #065f46;
    }

    .modal {
      display: none;
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.5);
      z-index: 9999;
      align-items: center;
      justify-content: center;
    }

    .modal.show {
      display: flex;
    }

    .modal-content {
      background: white;
      border-radius: 20px;
      padding: 32px;
      max-width: 400px;
      width: 90%;
      text-align: center;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
    }

    .modal h3 {
      font-size: 18px;
      margin-bottom: 6px;
    }

    .modal p {
      font-size: 13px;
      color: var(--text-secondary);
      margin-bottom: 20px;
    }

    .modal-buttons {
      display: flex;
      flex-direction: column;
      gap: 8px;
    }

    .modal-btn {
      padding: 12px 20px;
      border-radius: 10px;
      border: none;
      font-size: 14px;
      font-weight: 700;
      cursor: pointer;
    }

    .modal-btn.primary {
      background: var(--primary);
      color: white;
    }

    .modal-btn.secondary {
      background: #f1f5f9;
      color: var(--text);
    }

    .modal-btn:hover {
      opacity: 0.9;
    }
  </style>
</head>

<body>
  <div class="container">
    <h1>🏠 분당·수지 교구 검색</h1>
    <p class="subtitle">주소 검색 → 건물명 분석 → 담당 지구·목사 확인</p>

    <div class="btn-area">
      <button id="searchBtn" class="search-btn">🔍 주소 검색 열기</button>
    </div>

    <div id="resultCard" class="result-card">
      <div class="result-title-row">
        <div class="result-icon">📍</div>
        <div>
          <div class="result-district" id="resultTitle"></div>
          <div class="result-pastor" id="resultPastor"></div>
        </div>
      </div>
      <div class="result-detail" id="resultDetail"></div>
    </div>
  </div>

  <!-- 이매촌 모달 -->
  <div id="imaeModal" class="modal">
    <div class="modal-content">
      <h3>🏠 이매촌 목장 유형 선택</h3>
      <p>이매촌은 목장 유형에 따라 지구가 다릅니다</p>
      <div class="modal-buttons">
        <button class="modal-btn primary" data-type="자매">👩 자매 목장 → 분당 3지구</button>
        <button class="modal-btn secondary" data-type="형제">👨 형제 목장 → 분당 2지구</button>
        <button class="modal-btn secondary" data-type="부부">👫 부부 목장 → 분당 2지구</button>
        <button class="modal-btn secondary" data-type="직장자매">💼 직장자매 → 분당 2지구</button>
      </div>
    </div>
  </div>

  <script>
    // ==================== 데이터 (경량화) ====================
    const D = [{
        id: "분당 1지구",
        p: "남태욱 목사",
        k: "서울 위례신도시 위례 하남 남양주 인천 송도 경기북부",
        d: "서울, 위례신도시, 하남, 남양주, 인천, 송도, 경기북부",
        r: ""
      },
      {
        id: "분당 2지구",
        p: "김주호 목사",
        k: "성남 서현 시범단지 아름마을 이매촌 야탑",
        d: "성남(수정·중원), 시범단지, 아름, 이매촌(형제·직장자매·부부), 야탑",
        r: "분당",
        imae: ["형제", "부부", "직장자매"]
      },
      {
        id: "분당 3지구",
        p: "김재형 목사",
        k: "이매촌 판교 대장동 안양 산본 안산 과천",
        d: "이매촌(자매), 판교, 대장동, 안양, 산본, 안산, 과천",
        r: "분당",
        imae: ["자매"]
      },
      {
        id: "분당 4지구",
        p: "최지훈 목사",
        k: "효자촌 푸른마을 광주시 이천 여주",
        d: "효자촌, 푸른마을, 광주, 이천, 여주",
        r: "분당"
      },
      {
        id: "분당 5지구",
        p: "홍수민 목사",
        k: "샛별마을 느티마을 상록마을 한솔마을 파크타운",
        d: "샛별, 느티, 상록, 한솔, 파크타운",
        r: "분당"
      },
      {
        id: "분당 6지구",
        p: "이정호 목사",
        k: "양지마을 탄천 금호베스트빌1단지 분당파크뷰 분당현대아이파크 동양파라곤 아데나팰리스",
        d: "양지·탄천, 정자동 주요 아파트",
        r: "분당"
      },
      {
        id: "분당 7지구",
        p: "김승현 목사",
        k: "금곡동 트리폴리스 분당하우스토리 천사의도시 더헤리티지",
        d: "금곡(트리폴리스·분당하우스토리·천사의도시·더헤리티지)",
        r: "분당"
      },
      {
        id: "분당 8지구",
        p: "김영래 목사",
        k: "청솔마을 계룡 화인유천 한라 임광보성 궁내동",
        d: "청솔마을 1~4단지, 궁내동",
        r: "분당",
        cs: [1, 4]
      },
      {
        id: "분당 9지구",
        p: "김진성 목사",
        k: "청솔마을 공무원 주공 성원 대원 동아 미금 까치마을 정든마을",
        d: "청솔마을 5~10단지, 미금, 까치, 정든",
        r: "분당",
        cs: [5, 10]
      },
      {
        id: "분당 10지구",
        p: "정병철 목사",
        k: "구미동 무지개마을 하얀마을",
        d: "구미동(무지개·하얀마을)",
        r: "분당"
      },
      {
        id: "분당 11지구",
        p: "서일원 목사",
        k: "동천동 인현마을 고기동",
        d: "동천동, 인현마을, 고기동",
        r: "분당"
      },
      {
        id: "분당 12지구",
        p: "장성진 목사",
        k: "내대지 대지마을 도담마을 새터마을 성현마을 서모현",
        d: "내대지, 대지, 도담, 새터, 성현, 서모현",
        r: "분당"
      },
      {
        id: "분당 13지구",
        p: "김현철 목사",
        k: "꽃메마을 죽현마을 솔레시티 연원마을 보정동",
        d: "꽃메, 죽현, 솔레시티, 연원, 보정동",
        r: "분당"
      },
      {
        id: "분당 14지구",
        p: "라주영 목사",
        k: "마북동 구성동 구갈동 신갈동 상갈동 하갈동 민속촌 언남동 보라동 지곡동 공세동 고매동",
        d: "마북~고매동 일대",
        r: "분당"
      },
      {
        id: "분당 15지구",
        p: "황민구 목사",
        k: "물푸레마을 동백동 상하동 처인구 진흥더루벤스 수원동마을 강남마을 한라비발디",
        d: "물푸레, 동백, 상하동, 용인 처인구",
        r: "분당"
      },
      {
        id: "수지 1지구",
        p: "조명연 목사",
        k: "풍덕천동 삼성쉐르빌 건영캐스빌 우성그린빌 래미안수지이스트파크",
        d: "풍덕천1동, 삼성쉐르빌, 건영캐스빌",
        r: "수지"
      },
      {
        id: "수지 2지구",
        p: "고영수 목사",
        k: "현대프라임 극동임광 성지프라임 신봉우남퍼스트빌 신봉자이3차",
        d: "현대프라임, 극동임광, 신봉우남퍼스트빌",
        r: "수지"
      },
      {
        id: "수지 3지구",
        p: "박기선 목사",
        k: "진산마을 삼성래미안 성원상떼빌 수지구청역힐스테이트 솔뫼마을 성호샤인힐즈",
        d: "진산마을, 수지구청역, 솔뫼마을",
        r: "수지"
      },
      {
        id: "수지 4지구",
        p: "조요한 목사",
        k: "LG신봉자이1차 벽산 한일 한화",
        d: "LG신봉자이1차, 벽산, 한일, 한화",
        r: "수지"
      },
      {
        id: "수지 5지구",
        p: "최성욱 목사",
        k: "LG신봉자이2차 수지LG빌리지5차 광교산자이 동일하이빌 동부센트레빌 수지스카이뷰푸르지오 힐스테이트광교산",
        d: "LG신봉자이2차, 광교산자이, 동일하이빌",
        r: "수지"
      },
      {
        id: "수지 6지구",
        p: "고성현 목사",
        k: "신정마을 현대성우8단지 수지에듀파크 이스턴펠리스 태영데시앙 e편한세상수지 성동마을 서원마을 수지풍산",
        d: "신정마을, 수지에듀파크, 성동마을, 수지풍산",
        r: "수지"
      },
      {
        id: "수지 7지구",
        p: "임종득 목사",
        k: "수지자이 성복자이 성복힐스테이트 성복경남아너스빌 롯데캐슬파크나인 롯데캐슬클라시엘",
        d: "수지자이, 성복자이, 성복힐스테이트",
        r: "수지"
      },
      {
        id: "수지 8지구",
        p: "채정일 목사",
        k: "상현동 광교레이크포레 금호베스트빌 수지금호베스트빌 현대성우 동일스위트 수지센트럴아이파크 상현두산위브 상현LG자이 성복역리버파크",
        d: "상현동, 금호베스트빌, 현대성우, 두산위브",
        r: "수지"
      },
      {
        id: "수지 9지구",
        p: "정기성 목사",
        k: "광교 원천동 이의동 하동 광교호반베르디움 광교더샵 광교아이파크 상록자이 호수마을 휴먼시아",
        d: "광교신도시, 원천동, 이의동, 하동",
        r: "수지"
      },
      {
        id: "수지 10지구",
        p: "이정하 목사",
        k: "수원 오산 동탄 흥덕 영덕동 광교호반마을 경기남부",
        d: "수원, 오산, 동탄, 흥덕·영덕동",
        r: "수지"
      }
    ];
    const regionMap = {
      "수지": ["수지 1지구", "수지 2지구", "수지 3지구", "수지 4지구", "수지 5지구", "수지 6지구", "수지 7지구", "수지 8지구", "수지 9지구", "수지 10지구"],
      "분당": ["분당 1지구", "분당 2지구", "분당 3지구", "분당 4지구", "분당 5지구", "분당 6지구", "분당 7지구", "분당 8지구", "분당 9지구", "분당 10지구", "분당 11지구", "분당 12지구", "분당 13지구", "분당 14지구", "분당 15지구"],
      "광교": ["수지 9지구", "수지 10지구"],
      "판교": ["분당 3지구"],
      "성복": ["수지 7지구"],
      "상현": ["수지 8지구", "수지 6지구"],
      "신봉": ["수지 4지구", "수지 5지구", "수지 2지구"]
    };

    function norm(s) {
      return (s || '').replace(/[\(\[].*?[\)\]]/g, '').replace(/\s+/g, '').toLowerCase();
    }

    function extractRegion(name) {
      const regions = ["수지", "분당", "광교", "판교", "동탄", "성복", "상현", "신봉", "죽전", "동백", "구성"];
      const n = norm(name);
      for (const r of regions) {
        if (n.includes(r)) return r;
      }
      return null;
    }

    function matchBuilding(name) {
      if (!name) return null;
      const nn = norm(name);
      const region = extractRegion(name);
      // 청솔마을
      if (nn.includes('청솔마을')) {
        const m = nn.match(/(\d+)\s*단지/) || (name || '').match(/(\d+)\s*단지/);
        if (m) {
          const d = parseInt(m[1]);
          if (d >= 1 && d <= 4) return {
            dist: D.find(x => x.id === '분당 8지구'),
            reason: `청솔마을 ${d}단지 → 8지구`
          };
          if (d >= 5 && d <= 10) return {
            dist: D.find(x => x.id === '분당 9지구'),
            reason: `청솔마을 ${d}단지 → 9지구`
          };
        }
      }
      // 후보 필터링
      let candidates = D;
      if (region && regionMap[region]) {
        const ids = regionMap[region];
        candidates = D.filter(x => ids.includes(x.id));
      }
      let best = null,
        bestScore = 0;
      for (const dist of candidates) {
        for (const kw of dist.k.split(' ')) {
          const nk = norm(kw);
          if (nk.length < 2) continue;
          if (nn.includes(nk)) {
            const score = nk.length * 2;
            if (score > bestScore) {
              bestScore = score;
              best = dist;
            }
          }
        }
      }
      if (best) {
        return {
          dist: best,
          reason: `건물명 매칭: "${name}" → ${best.id}` + (region ? ` (지역:${region})` : '')
        };
      }
      return null;
    }
    // ==================== 이매촌 ====================
    let pending = null;

    function showModal() {
      document.getElementById('imaeModal').style.display = 'flex';
    }

    function hideModal() {
      document.getElementById('imaeModal').style.display = 'none';
    }
    // ==================== 메인 ====================
    function findDistrict(addr) {
      const bld = addr.buildingName;
      const full = (addr.roadAddress || addr.jibunAddress || '') + ' ' + (addr.bname || '');
      // 1. 건물명
      if (bld) {
        const r = matchBuilding(bld);
        if (r) return r;
      }
      // 2. 이매촌
      if (full.includes('이매촌') || (full.includes('이매동') && full.includes('촌'))) {
        pending = addr;
        showModal();
        return {
          needModal: true
        };
      }
      // 3. 키워드
      const low = full.toLowerCase();
      let best = null,
        bestScore = 0;
      for (const dist of D) {
        let s = 0;
        for (const kw of dist.k.split(' ')) {
          if (low.includes(kw.toLowerCase())) s += kw.length;
        }
        if (s > bestScore) {
          bestScore = s;
          best = dist;
        }
      }
      if (best) return {
        dist: best,
        reason: `지역 키워드 매칭 (점수:${bestScore})`
      };
      return null;
    }

    function showResult(result, addr) {
      const card = document.getElementById('resultCard');
      const title = document.getElementById('resultTitle');
      const pastor = document.getElementById('resultPastor');
      const detail = document.getElementById('resultDetail');
      card.className = 'result-card show';
      if (result && result.needModal) {
        title.textContent = '⚠️ 선택 필요';
        pastor.textContent = '';
        detail.innerHTML = '<span class="badge">이매촌</span> 목장 유형을 선택해주세요.';
        return;
      }
      if (result && result.dist) {
        title.textContent = result.dist.id;
        pastor.textContent = result.dist.p;
        let h = '';
        if (addr.roadAddress) h += `📍 <b>도로명:</b> ${addr.roadAddress}<br>`;
        if (addr.jibunAddress) h += `📋 <b>지번:</b> ${addr.jibunAddress}<br>`;
        if (addr.buildingName) h += `🏢 <b>건물명:</b> ${addr.buildingName}<br>`;
        h += `📋 <b>담당:</b> ${result.dist.d}<br>`;
        h += `🏠 <b>근거:</b> ${result.reason||''}`;
        detail.innerHTML = h;
      } else {
        title.textContent = '❌ 미발견';
        pastor.textContent = '';
        detail.innerHTML = `일치하는 교구를 찾을 수 없습니다.<br>📍 ${addr.roadAddress||addr.jibunAddress||''}`;
      }
      card.scrollIntoView({
        behavior: 'smooth',
        block: 'center'
      });
    }
    // ==================== 카카오 API ====================
    function openPostcode() {
      if (typeof daum === 'undefined' || typeof daum.Postcode === 'undefined') {
        alert('카카오 주소 API를 불러오는 중입니다. 잠시 후 다시 시도해주세요.');
        return;
      }
      new daum.Postcode({
        oncomplete: function(data) {
          const addr = {
            buildingName: data.buildingName || '',
            roadAddress: data.roadAddress || '',
            jibunAddress: data.jibunAddress || data.autoJibunAddress || '',
            bname: data.bname || '',
            bname1: data.bname1 || ''
          };
          const result = findDistrict(addr);
          showResult(result, addr);
        }
      }).open();
    }
    // ==================== 이벤트 연결 ====================
    document.getElementById('searchBtn').addEventListener('click', openPostcode);
    // 모달 버튼들
    document.querySelectorAll('#imaeModal .modal-btn').forEach(btn => {
      btn.addEventListener('click', function() {
        const type = this.getAttribute('data-type');
        hideModal();
        if (!pending) return;
        const dist = type === '자매' ?
          D.find(d => d.id === '분당 3지구') :
          D.find(d => d.id === '분당 2지구');
        showResult({
          dist,
          reason: `이매촌 ${type} 목장 → ${dist.id}`
        }, pending);
        pending = null;
      });
    });
    // 모달 배경 클릭 닫기
    document.getElementById('imaeModal').addEventListener('click', function(e) {
      if (e.target === this) {
        hideModal();
        pending = null;
      }
    });
    // API 로딩 확인
    window.addEventListener('load', function() {
      if (typeof daum === 'undefined') {
        console.warn('카카오 API 로딩 지연...');
      }
    });
  </script>
</body>

</html>
