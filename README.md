<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>분당·수지 교구 검색기</title>
<!-- 미리 로드해두면 버튼 클릭 시 즉시 열린다. 실패해도 클릭 시점에 재시도하므로 안전. -->
<script src="https://t1.kakaocdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js" async></script>
<style>
  :root{
    --primary:#4f46e5; --primary-light:#eef2ff; --primary-dark:#4338ca;
    --surface:#ffffff; --text:#1e293b; --text-secondary:#64748b;
    --border:#e2e8f0; --radius:16px;
    --ok:#059669; --ok-bg:#d1fae5; --warn:#b45309; --warn-bg:#fef3c7;
    --err:#dc2626; --err-bg:#fee2e2;
  }
  *{margin:0;padding:0;box-sizing:border-box;}
  body{font-family:'Pretendard','Noto Sans KR',-apple-system,BlinkMacSystemFont,sans-serif;
    background:#f1f5f9;min-height:100vh;padding:20px;color:var(--text);
    -webkit-font-smoothing:antialiased;}
  .container{background:var(--surface);border-radius:var(--radius);padding:32px 28px;
    max-width:640px;width:100%;margin:0 auto;
    box-shadow:0 20px 50px rgba(0,0,0,.08);border:1px solid var(--border);}
  h1{font-size:26px;font-weight:800;text-align:center;margin-bottom:6px;letter-spacing:-.5px;}
  .subtitle{text-align:center;color:var(--text-secondary);margin-bottom:22px;font-size:13.5px;}

  .tabs{display:flex;gap:6px;margin-bottom:20px;background:#f8fafc;padding:5px;
    border-radius:12px;border:1px solid var(--border);}
  .tab{flex:1;padding:10px;border-radius:9px;border:0;background:transparent;
    color:var(--text-secondary);font-size:13.5px;font-weight:700;cursor:pointer;
    transition:.15s;font-family:inherit;}
  .tab:hover{color:var(--text);}
  .tab.on{background:var(--surface);color:var(--primary);box-shadow:0 1px 4px rgba(0,0,0,.08);}

  .search-btn{width:100%;background:var(--primary);color:#fff;border:none;
    padding:17px 32px;border-radius:12px;font-size:17px;font-weight:700;cursor:pointer;
    transition:.2s;letter-spacing:-.3px;font-family:inherit;}
  .search-btn:hover{background:var(--primary-dark);}
  .search-btn:active{transform:scale(.98);}
  .search-btn.sec{background:#f1f5f9;color:var(--text);border:1px solid var(--border);
    font-size:14px;padding:13px;}
  .search-btn.sec:hover{background:#e2e8f0;}

  .manual{display:flex;gap:8px;margin-top:10px;}
  .manual input{flex:1;padding:14px;border-radius:11px;border:1px solid var(--border);
    font-size:14px;font-family:inherit;outline:none;transition:.15s;min-width:0;}
  .manual input:focus{border-color:var(--primary);box-shadow:0 0 0 3px rgba(79,70,229,.12);}
  .manual button{padding:14px 20px;border-radius:11px;border:0;background:var(--primary);
    color:#fff;font-weight:700;cursor:pointer;font-size:14px;font-family:inherit;white-space:nowrap;}
  .manual button:hover{background:var(--primary-dark);}
  .hint{font-size:12px;color:var(--text-secondary);margin-top:9px;line-height:1.7;}
  .ex{display:inline-block;background:#f8fafc;border:1px solid var(--border);border-radius:7px;
    padding:5px 10px;margin:3px 3px 0 0;font-size:12px;cursor:pointer;transition:.15s;}
  .ex:hover{border-color:var(--primary);color:var(--primary);}

  .result-card{background:#fafbff;border-radius:12px;padding:22px;border:1px solid #e0e7ff;
    display:none;margin-top:20px;}
  .result-card.show{display:block;}
  .result-title-row{display:flex;align-items:center;gap:11px;margin-bottom:14px;}
  .result-icon{width:42px;height:42px;background:var(--primary-light);border-radius:11px;
    display:flex;align-items:center;justify-content:center;font-size:21px;flex:none;}
  .result-district{font-size:21px;font-weight:800;letter-spacing:-.4px;}
  .result-pastor{font-size:15.5px;font-weight:600;color:var(--primary);}
  .result-detail{font-size:13px;color:var(--text-secondary);line-height:1.85;
    background:#fff;border-radius:10px;padding:15px;border:1px solid var(--border);}
  .result-detail b{color:var(--text);}

  .badge{display:inline-block;padding:3px 10px;border-radius:20px;font-size:11.5px;
    font-weight:700;margin:2px 2px 2px 0;background:var(--primary-light);color:var(--primary);}
  .badge-ok{background:var(--ok-bg);color:var(--ok);}
  .badge-warn{background:var(--warn-bg);color:var(--warn);}
  .badge-err{background:var(--err-bg);color:var(--err);}

  .conf-bar{height:6px;background:#e2e8f0;border-radius:4px;overflow:hidden;margin:9px 0 3px;}
  .conf-bar i{display:block;height:100%;border-radius:4px;transition:width .4s;}

  .alt{margin-top:12px;padding-top:12px;border-top:1px dashed var(--border);}
  .alt-title{font-size:12px;font-weight:700;color:var(--text-secondary);margin-bottom:7px;}
  .alt-item{display:flex;justify-content:space-between;align-items:center;gap:8px;
    padding:8px 11px;background:#fff;border:1px solid var(--border);border-radius:8px;
    margin-bottom:5px;font-size:12.5px;cursor:pointer;transition:.15s;}
  .alt-item:hover{border-color:var(--primary);background:var(--primary-light);}
  .alt-item .s{color:var(--text-secondary);font-size:11.5px;}

  .notice{padding:11px 14px;border-radius:10px;font-size:12.5px;margin-bottom:11px;line-height:1.65;}
  .n-warn{background:var(--warn-bg);color:#78350f;border:1px solid #fcd34d;}
  .n-err{background:var(--err-bg);color:#7f1d1d;border:1px solid #fca5a5;}
  .n-info{background:var(--primary-light);color:#3730a3;border:1px solid #c7d2fe;}

  .modal{display:none;position:fixed;inset:0;background:rgba(0,0,0,.5);z-index:9999;
    align-items:center;justify-content:center;padding:20px;}
  .modal.show{display:flex;}
  .modal-content{background:#fff;border-radius:20px;padding:28px;max-width:420px;width:100%;
    text-align:center;box-shadow:0 20px 60px rgba(0,0,0,.3);max-height:90vh;overflow-y:auto;}
  .modal h3{font-size:18px;margin-bottom:6px;}
  .modal p{font-size:13px;color:var(--text-secondary);margin-bottom:18px;}
  .modal-buttons{display:flex;flex-direction:column;gap:8px;}
  .modal-btn{padding:13px 20px;border-radius:10px;border:none;font-size:14px;
    font-weight:700;cursor:pointer;font-family:inherit;transition:.15s;}
  .modal-btn.primary{background:var(--primary);color:#fff;}
  .modal-btn.secondary{background:#f1f5f9;color:var(--text);}
  .modal-btn:hover{opacity:.88;}

  .sbox{position:relative;display:flex;gap:8px;}
  .sbox input{flex:1;min-width:0;padding:16px 15px;border-radius:12px;
    border:2px solid var(--border);font-size:15.5px;font-family:inherit;outline:none;transition:.15s;}
  .sbox input:focus{border-color:var(--primary);box-shadow:0 0 0 4px rgba(79,70,229,.12);}
  .sbox button{padding:16px 24px;border-radius:12px;border:0;background:var(--primary);
    color:#fff;font-weight:700;font-size:15px;cursor:pointer;font-family:inherit;white-space:nowrap;}
  .sbox button:hover{background:var(--primary-dark);}
  .sugg{position:absolute;top:calc(100% + 6px);left:0;right:0;background:#fff;
    border:1px solid var(--border);border-radius:12px;overflow:hidden;z-index:50;
    box-shadow:0 14px 40px rgba(0,0,0,.16);display:none;max-height:330px;overflow-y:auto;}
  .sugg.show{display:block;}
  .sugg-item{padding:12px 15px;cursor:pointer;border-bottom:1px solid #f1f5f9;
    display:flex;justify-content:space-between;align-items:center;gap:10px;}
  .sugg-item:last-child{border-bottom:0;}
  .sugg-item:hover,.sugg-item.on{background:var(--primary-light);}
  .sugg-term{font-size:14px;font-weight:700;}
  .sugg-tag{font-size:10.5px;color:var(--text-secondary);background:#f1f5f9;
    border-radius:5px;padding:2px 7px;margin-left:7px;font-weight:600;}
  .sugg-meta{font-size:12px;color:var(--primary);font-weight:700;text-align:right;flex:none;}
  .sugg-meta small{display:block;color:var(--text-secondary);font-weight:500;font-size:11px;}
  .sugg-empty{padding:16px 15px;font-size:13px;color:var(--text-secondary);text-align:center;}
  .kakao-more{margin-top:14px;border:1px solid var(--border);border-radius:11px;
    padding:12px 15px;background:#f8fafc;}
  .kakao-more summary{cursor:pointer;font-size:13px;font-weight:700;color:var(--text-secondary);}
  .kakao-more[open] summary{color:var(--text);margin-bottom:4px;}
  .pc-wrap{display:none;margin-top:14px;border:1px solid var(--border);
    border-radius:12px;overflow:hidden;background:#fff;}
  .pc-wrap.show{display:block;}
  .pc-head{display:flex;align-items:center;justify-content:space-between;
    padding:11px 14px;background:#f8fafc;border-bottom:1px solid var(--border);
    font-size:13.5px;font-weight:700;}
  .pc-close{border:1px solid var(--border);background:#fff;border-radius:8px;
    padding:6px 12px;font-size:12.5px;font-weight:700;cursor:pointer;
    color:var(--text-secondary);font-family:inherit;transition:.15s;}
  .pc-close:hover{background:#f1f5f9;color:var(--text);}
  .pc-layer{width:100%;height:460px;}
  .pc-layer iframe{width:100%!important;height:100%!important;border:0;display:block;}
  .pc-tip{padding:9px 14px;font-size:11.5px;color:var(--text-secondary);
    background:#f8fafc;border-top:1px solid var(--border);line-height:1.6;}
  .pc-loading{display:flex;align-items:center;justify-content:center;height:100%;
    font-size:13px;color:var(--text-secondary);}
  @media(max-width:520px){ .pc-layer{height:420px;} }

  .list-wrap{max-height:460px;overflow-y:auto;border:1px solid var(--border);border-radius:11px;}
  table{width:100%;border-collapse:collapse;font-size:12.5px;}
  th,td{padding:9px 11px;text-align:left;border-bottom:1px solid var(--border);vertical-align:top;}
  th{background:#f8fafc;font-size:11.5px;color:var(--text-secondary);font-weight:700;
    position:sticky;top:0;z-index:1;}
  tbody tr:hover{background:#f8fafc;}
  td b{color:var(--text);}
  .srch{width:100%;padding:11px 13px;border-radius:10px;border:1px solid var(--border);
    font-size:13.5px;margin-bottom:10px;font-family:inherit;outline:none;}
  .srch:focus{border-color:var(--primary);}

  textarea{width:100%;min-height:110px;padding:13px;border-radius:11px;
    border:1px solid var(--border);font-size:13px;font-family:inherit;
    line-height:1.65;resize:vertical;outline:none;}
  textarea:focus{border-color:var(--primary);}
  .row{display:flex;gap:8px;flex-wrap:wrap;align-items:center;margin-top:10px;}
  .stat{font-size:12.5px;color:var(--text-secondary);}
  .hide{display:none!important;}
  .foot{text-align:center;font-size:11.5px;color:var(--text-secondary);
    margin-top:20px;line-height:1.8;}
  @media(max-width:520px){
    body{padding:12px;} .container{padding:22px 17px;} h1{font-size:22px;}
    .manual{flex-direction:column;} .manual button{width:100%;}
  }
</style>
</head>
<body>
<div class="container">
  <h1>&#127968; 분당&middot;수지 교구 검색</h1>
  <p class="subtitle">주소 &rarr; 건물명 분석 &rarr; 담당 지구&middot;목사 확인</p>

  <div class="tabs">
    <button class="tab on" data-tab="search">&#128269; 검색</button>
    <button class="tab" data-tab="bulk">&#128203; 일괄</button>
    <button class="tab" data-tab="list">&#128100; 지구목록</button>
  </div>

  <!-- ============ 검색 ============ -->
  <div id="tab-search">
    <div class="sbox">
      <input type="text" id="q" autocomplete="off" spellcheck="false"
             placeholder="아파트명 · 동 이름 · 목사님 성함 (초성도 가능)">
      <button id="qBtn">조회</button>
      <div class="sugg" id="sugg"></div>
    </div>
    <div class="hint">
      &#9989; <b>인터넷 없이 바로 동작</b>합니다. 예) <b>ㄱㄱ</b> &rarr; 광교, <b>ㅊㅅㅁㅇ</b> &rarr; 청솔마을
      <div style="margin-top:6px">
        <span class="ex">광교레이크포레</span><span class="ex">청솔마을</span>
        <span class="ex">정자동</span><span class="ex">성복동</span><span class="ex">이매촌</span>
      </div>
    </div>

    <details class="kakao-more">
      <summary>&#127760; 도로명 주소로 찾기 (인터넷 필요)</summary>
      <div style="padding-top:10px">
        <button id="searchBtn" class="search-btn sec" style="width:100%">&#128269; 카카오 주소 검색 열기</button>
        <div id="pcWrap" class="pc-wrap">
          <div class="pc-head">
            <span>&#128269; 주소 검색</span>
            <button type="button" id="pcClose" class="pc-close">&#10005; 닫기</button>
          </div>
          <div id="pcLayer" class="pc-layer"></div>
        </div>
        <div class="hint" style="margin-top:8px">
          보안 정책이 엄격한 환경(앱 내 브라우저, 미리보기 등)에서는 카카오 검색 결과가
          이 페이지로 전달되지 않을 수 있습니다. 그럴 때는 위의 <b>기본 검색창</b>을 이용해 주세요.
        </div>
      </div>
    </details>

    <div id="resultCard" class="result-card">
      <div id="noticeBox"></div>
      <div class="result-title-row">
        <div class="result-icon" id="resultIcon">&#128205;</div>
        <div>
          <div class="result-district" id="resultTitle"></div>
          <div class="result-pastor" id="resultPastor"></div>
        </div>
      </div>
      <div class="result-detail" id="resultDetail"></div>
      <div id="altBox"></div>
    </div>
  </div>

  <!-- ============ 일괄 ============ -->
  <div id="tab-bulk" class="hide">
    <textarea id="bulkIn" placeholder="한 줄에 하나씩 입력하세요.&#10;광교레이크포레&#10;청솔마을 7단지&#10;경기 용인시 수지구 성복동"></textarea>
    <div class="row">
      <button class="search-btn sec" id="btnBulk" style="width:auto;padding:12px 20px">일괄 조회</button>
      <button class="search-btn sec" id="btnBulkCsv" style="width:auto;padding:12px 20px">CSV 저장</button>
      <button class="search-btn sec" id="btnBulkSample" style="width:auto;padding:12px 20px">샘플</button>
    </div>
    <div class="stat" id="bulkStat" style="margin-top:9px"></div>
    <div class="list-wrap hide" id="bulkWrap" style="margin-top:11px">
      <table id="bulkTbl"></table>
    </div>
  </div>

  <!-- ============ 지구목록 ============ -->
  <div id="tab-list" class="hide">
    <input type="text" class="srch" id="listSrch" placeholder="지구명 / 목사님 성함 / 담당 지역으로 찾기">
    <div class="list-wrap"><table id="listTbl"></table></div>
    <div class="hint">총 <b id="listCnt">0</b>개 지구 &middot; 행을 클릭하면 담당 구역이 검색창에 입력됩니다.</div>
  </div>

  <div class="foot">
    담당 구역은 교회 편성에 따라 변경될 수 있습니다.<br>
    결과가 <b>확인 필요</b>로 표시되면 교구 사무실에 문의해 주세요.
  </div>
</div>

<!-- 이매촌 모달 -->
<div id="imaeModal" class="modal">
  <div class="modal-content">
    <h3>&#127968; 이매촌 목장 유형 선택</h3>
    <p>이매촌은 목장 유형에 따라 지구가 다릅니다</p>
    <div class="modal-buttons">
      <button class="modal-btn primary"   data-type="자매">&#128105; 자매 목장 &rarr; 분당 3지구</button>
      <button class="modal-btn secondary" data-type="형제">&#128104; 형제 목장 &rarr; 분당 2지구</button>
      <button class="modal-btn secondary" data-type="부부">&#128107; 부부 목장 &rarr; 분당 2지구</button>
      <button class="modal-btn secondary" data-type="직장자매">&#128188; 직장자매 &rarr; 분당 2지구</button>
      <button class="modal-btn secondary" data-type="__cancel">닫기</button>
    </div>
  </div>
</div>

<script>
/* ===================== 지구 데이터 ===================== */
var D = [{"id": "분당 1지구", "p": "남태욱 목사", "d": "서울, 위례신도시, 하남, 남양주, 인천, 송도, 경기북부", "t1": [], "t2": ["위례신도시"], "t3": [], "t4": ["서울", "위례", "하남", "남양주", "인천", "송도", "경기북부"]}, {"id": "분당 2지구", "p": "김주호 목사", "d": "성남(수정·중원), 시범단지, 아름마을, 이매촌(형제·직장자매·부부), 야탑", "t1": [], "t2": ["시범단지", "아름마을", "이매촌"], "t3": ["서현동", "야탑동", "수정구", "중원구"], "t4": ["성남", "서현", "야탑"], "imae": ["형제", "부부", "직장자매"]}, {"id": "분당 3지구", "p": "김재형 목사", "d": "이매촌(자매), 판교, 대장동, 안양, 산본, 안산, 과천", "t1": [], "t2": ["이매촌", "판교신도시"], "t3": ["대장동", "백현동", "삼평동", "운중동", "판교동"], "t4": ["판교", "안양", "산본", "안산", "과천"], "imae": ["자매"]}, {"id": "분당 4지구", "p": "최지훈 목사", "d": "효자촌, 푸른마을, 광주, 이천, 여주", "t1": [], "t2": ["효자촌", "푸른마을"], "t3": [], "t4": ["광주시", "이천", "여주"]}, {"id": "분당 5지구", "p": "홍수민 목사", "d": "샛별마을, 느티마을, 상록마을, 한솔마을, 파크타운", "t1": [], "t2": ["샛별마을", "느티마을", "상록마을", "한솔마을", "파크타운"], "t3": [], "t4": []}, {"id": "분당 6지구", "p": "이정호 목사", "d": "양지마을·탄천마을, 정자동 주요 아파트", "t1": ["금호베스트빌1단지", "분당파크뷰", "분당현대아이파크", "동양파라곤", "아데나팰리스"], "t2": ["양지마을", "탄천마을"], "t3": ["정자동"], "t4": [], "roads": ["정자일로", "내정로"]}, {"id": "분당 7지구", "p": "김승현 목사", "d": "금곡동(트리폴리스·분당하우스토리·천사의도시·더헤리티지)", "t1": ["트리폴리스", "분당하우스토리", "천사의도시", "더헤리티지"], "t2": [], "t3": ["금곡동"], "t4": [], "roads": ["금곡로"]}, {"id": "분당 8지구", "p": "김영래 목사", "d": "청솔마을 1~4단지, 궁내동", "t1": ["청솔계룡", "청솔화인유천", "청솔한라", "청솔임광보성"], "t2": ["계룡아파트", "화인유천", "임광보성"], "t3": ["궁내동"], "t4": [], "cs": [1, 4]}, {"id": "분당 9지구", "p": "김진성 목사", "d": "청솔마을 5~10단지, 까치마을(미금일로 일대), 정든마을", "t1": ["청솔공무원", "청솔주공", "청솔성원", "청솔대원", "청솔동아", "까치마을주공2단지", "까치마을롯데선경", "까치마을건영빌라", "까치마을신원", "정든마을한진", "정든마을신화", "정든마을우성", "정든마을동아"], "t2": ["까치마을", "정든마을"], "t3": [], "t4": [], "cs": [5, 10], "roads": ["미금일로"]}, {"id": "분당 10지구", "p": "정병철 목사", "d": "구미동 무지개마을·하얀마을 (미금로·무지개로 일대)", "t1": ["무지개마을1단지", "무지개마을2단지", "무지개마을3단지", "무지개마을4단지", "무지개마을5단지", "하얀마을주공5단지", "하얀마을경남", "하얀마을현대"], "t2": ["무지개마을", "하얀마을"], "t3": ["구미동"], "t4": [], "roads": ["무지개로", "미금로"]}, {"id": "분당 11지구", "p": "서일원 목사", "d": "동천동, 인현마을, 고기동", "t1": [], "t2": ["인현마을"], "t3": ["동천동", "고기동"], "t4": []}, {"id": "분당 12지구", "p": "장성진 목사", "d": "내대지, 대지마을, 도담마을, 새터마을, 성현마을, 서모현", "t1": [], "t2": ["내대지마을", "대지마을", "도담마을", "새터마을", "성현마을", "서모현"], "t3": [], "t4": ["내대지"]}, {"id": "분당 13지구", "p": "김현철 목사", "d": "꽃메마을, 죽현마을, 솔레시티, 연원마을, 보정동", "t1": ["솔레시티"], "t2": ["꽃메마을", "죽현마을", "연원마을"], "t3": ["보정동"], "t4": []}, {"id": "분당 14지구", "p": "라주영 목사", "d": "마북동~고매동 일대 (구성·기흥)", "t1": [], "t2": ["민속촌"], "t3": ["마북동", "구성동", "구갈동", "신갈동", "상갈동", "하갈동", "언남동", "보라동", "지곡동", "공세동", "고매동"], "t4": []}, {"id": "분당 15지구", "p": "황민구 목사", "d": "물푸레마을, 동백, 상하동, 용인 처인구", "t1": ["진흥더루벤스", "한라비발디"], "t2": ["물푸레마을", "수원동마을", "강남마을"], "t3": ["동백동", "상하동", "처인구"], "t4": []}, {"id": "수지 1지구", "p": "조명연 목사", "d": "풍덕천1동, 삼성쉐르빌, 건영캐스빌, 우성그린빌", "t1": ["삼성쉐르빌", "건영캐스빌", "우성그린빌", "래미안수지이스트파크"], "t2": [], "t3": ["풍덕천동"], "t4": []}, {"id": "수지 2지구", "p": "고영수 목사", "d": "현대프라임, 극동임광, 성지프라임, 신봉우남퍼스트빌, 신봉자이3차", "t1": ["현대프라임", "극동임광", "성지프라임", "신봉우남퍼스트빌", "신봉자이3차"], "t2": [], "t3": [], "t4": []}, {"id": "수지 3지구", "p": "박기선 목사", "d": "진산마을, 수지구청역, 솔뫼마을", "t1": ["성원상떼빌", "수지구청역힐스테이트", "성호샤인힐즈", "삼성래미안"], "t2": ["진산마을", "솔뫼마을"], "t3": [], "t4": []}, {"id": "수지 4지구", "p": "조요한 목사", "d": "LG신봉자이1차, 벽산, 한일, 한화", "t1": ["LG신봉자이1차", "신봉자이1차"], "t2": ["신봉벽산", "신봉한일", "신봉한화"], "t3": [], "t4": []}, {"id": "수지 5지구", "p": "최성욱 목사", "d": "LG신봉자이2차, 광교산자이, 동일하이빌", "t1": ["LG신봉자이2차", "신봉자이2차", "수지LG빌리지5차", "LG빌리지5차", "광교산자이", "동일하이빌", "동부센트레빌", "수지스카이뷰푸르지오", "힐스테이트광교산"], "t2": [], "t3": [], "t4": []}, {"id": "수지 6지구", "p": "고성현 목사", "d": "신정마을, 수지에듀파크, 성동마을, 수지풍산", "t1": ["현대성우8단지", "수지에듀파크", "이스턴펠리스", "태영데시앙", "e편한세상수지", "수지풍산"], "t2": ["신정마을", "성동마을", "서원마을"], "t3": [], "t4": []}, {"id": "수지 7지구", "p": "임종득 목사", "d": "수지자이, 성복자이, 성복힐스테이트", "t1": ["수지자이", "성복자이", "성복힐스테이트", "성복경남아너스빌", "롯데캐슬파크나인", "롯데캐슬클라시엘"], "t2": [], "t3": ["성복동"], "t4": []}, {"id": "수지 8지구", "p": "채정일 목사", "d": "상현동, 금호베스트빌, 현대성우, 두산위브", "t1": ["광교레이크포레", "수지금호베스트빌", "수지센트럴아이파크", "상현두산위브", "상현LG자이", "성복역리버파크", "동일스위트"], "t2": ["금호베스트빌", "현대성우"], "t3": ["상현동"], "t4": []}, {"id": "수지 9지구", "p": "정기성 목사", "d": "광교신도시, 원천동, 이의동, 하동", "t1": ["광교호반베르디움", "광교더샵", "광교아이파크", "상록자이"], "t2": ["호수마을", "휴먼시아"], "t3": ["원천동", "이의동", "하동"], "t4": ["광교"], "roads": ["광교호수로", "이의동로"]}, {"id": "수지 10지구", "p": "이정하 목사", "d": "수원, 오산, 동탄, 흥덕·영덕동", "t1": ["광교호반마을"], "t2": [], "t3": ["영덕동", "흥덕동"], "t4": ["수원", "오산", "동탄", "경기남부"]}];

/* ===================== 매칭 엔진 v2 ===================== */
/* 교구 매칭 엔진 v2 — 계층형 특이도 스코어링 */
function norm(s){
  return (s||'').normalize('NFC')
    .replace(/[（(\[].*?[）)\]]/g,'')
    .replace(/[\s\-_,·・]/g,'')
    .toLowerCase();
}
// 공백을 유지한 정규화 (단어 경계 판정용)
function normSp(s){
  return (s||'').normalize('NFC')
    .replace(/[（(\[].*?[）)\]]/g,' ')
    .replace(/[\-_,·・]/g,'')
    .replace(/\s+/g,' ')
    .trim()
    .toLowerCase();
}
const TIER_W = { t1:100, t2:50, tR:40, t3:10, t4:3 };

// 법정동(t3)/광역(t4)처럼 짧은 지명은 "더 긴 지명의 꼬리"로 잘못 걸리기 쉽다.
// 예) "상하동"에 "하동"이 포함 → 앞 글자가 한글이면 다른 지명의 일부로 보고 기각.
function hasBoundary(haySp, nk, strict){
  if(!strict) return haySp.replace(/ /g,'').includes(nk);
  // 공백 유지 문자열에서, 앞 글자가 공백이거나 문장 시작일 때만 인정
  let i = haySp.indexOf(nk);
  while(i !== -1){
    const prev = i>0 ? haySp[i-1] : ' ';
    if(!/[가-힣]/.test(prev)) return true;
    i = haySp.indexOf(nk, i+1);
  }
  return false;
}

// 도로명 전용 매칭: 도로명 뒤에는 숫자·'번길'·공백·문자열 끝만 올 수 있다.
// 예) "미금일로154번길" ✅ / "미금로66"에서 "미금일로" ❌
function roadMatch(hay, nk){
  let i = hay.indexOf(nk);
  while(i !== -1){
    const after = hay.slice(i + nk.length);
    // 뒤가 비었거나 숫자로 시작하면 진짜 도로명
    if(after === '' || /^[0-9]/.test(after)) return true;
    // 뒤가 한글이면 더 긴 도로명의 일부일 수 있으므로 기각
    i = hay.indexOf(nk, i + 1);
  }
  return false;
}

function scoreDistrict(dist, hay, haySp){
  haySp = haySp==null ? hay : haySp;
  let total=0, hits=[];
  for(const t of ['t1','t2','tR','t3','t4']){
    const list = t==='tR' ? (dist.roads||[]) : dist[t];
    for(const kw of list){
      const nk=norm(kw);
      if(nk.length<2) continue;
      const strict = (t==='t3'||t==='t4') && nk.length<=3;
      let matched;
      if(t==='tR'){
        // 도로명: "미금일로"가 "미금로"에 걸리면 안 되므로 뒤에 숫자/번길/공백/끝만 허용
        matched = roadMatch(hay, nk);
      } else {
        matched = strict ? hasBoundary(haySp, nk, true) : hay.includes(nk);
      }
      if(matched){
        // 티어 가중치 + 키워드 길이(구체성) 보너스
        const s = TIER_W[t] + nk.length*2;
        total += s;
        hits.push({kw, tier:t, s});
      }
    }
  }
  return {total, hits};
}

// 청솔마을 단지번호 → 지구 (원문에서 숫자 추출)
function csRule(rawText, D){
  const n = norm(rawText);
  if(!n.includes('청솔')) return null;
  const m = n.match(/(\d{1,2})단지/);
  if(!m) return null;
  const num = parseInt(m[1],10);
  if(num>=1&&num<=4)  return {dist:D.find(x=>x.id==='분당 8지구'), reason:`청솔마을 ${num}단지 → 1~4단지 규칙`, conf:97};
  if(num>=5&&num<=10) return {dist:D.find(x=>x.id==='분당 9지구'), reason:`청솔마을 ${num}단지 → 5~10단지 규칙`, conf:97};
  // 범위 밖: 조용히 흘리지 않고 명시적으로 경고 반환 (원본 버그 #3 수정)
  return {dist:null, reason:`청솔마을 ${num}단지는 등록된 범위(1~10)를 벗어납니다`, conf:0, needCheck:true};
}

function isImae(rawText){
  const n = norm(rawText);
  return n.includes('이매촌') || n.includes('이매동') || n.includes('이매');
}

/**
 * @param {object} a {buildingName, roadAddress, jibunAddress, bname, sigungu, sido}
 * @param {Array} D districts
 */
function match(a, D){
  const bld  = a.buildingName||'';
  const road = a.roadAddress||'';
  const jibun= a.jibunAddress||'';
  const bname= a.bname||'';
  const sgg  = a.sigungu||'';

  // 1) 청솔마을 단지 규칙 (건물명 + 주소 모두 검사)
  const cs = csRule(bld+' '+jibun+' '+road, D);
  if(cs) return {...cs, stage:'청솔단지규칙'};

  // 2) 이매촌 → 목장 유형 선택 필요
  if(isImae(bld+' '+jibun+' '+bname)) return {needModal:true, stage:'이매촌'};

  // 3) 통합 스코어링: 건물명(가중 2배) + 주소 전체를 한 번에 평가
  //    ★ 원본처럼 지역으로 후보를 "잘라내지" 않는다 (버그 #1,2 수정)
  const hayB   = norm(bld);
  const hayBsp = normSp(bld);
  const hayA   = norm([road, jibun, bname, sgg].join(' '));
  const hayAsp = normSp([road, jibun, bname, sgg].join(' '));
  let best=null, ranked=[];
  for(const d of D){
    const sb = scoreDistrict(d, hayB, hayBsp);
    const sa = scoreDistrict(d, hayA, hayAsp);
    const total = sb.total*2 + sa.total;   // 건물명이 더 강한 근거
    if(total>0) ranked.push({d, total, hits:[...sb.hits.map(h=>({...h,from:'건물명'})), ...sa.hits.map(h=>({...h,from:'주소'}))]});
  }
  ranked.sort((x,y)=>y.total-x.total);
  if(!ranked.length) return {dist:null, reason:'일치하는 키워드가 없습니다', conf:0, stage:'미발견'};

  best = ranked[0];
  const second = ranked[1];
  const tierNum = t => t==='tR' ? 2.5 : +t[1];
  const topTier    = best.hits.reduce((m,h)=>Math.min(m, tierNum(h.tier)), 9);
  const secondTier = second ? second.hits.reduce((m,h)=>Math.min(m, tierNum(h.tier)), 9) : 9;

  // 모호 판정: 같은 티어에서 점수가 근접할 때만 (티어가 다르면 명확히 이긴 것)
  const ratio = second ? second.total / best.total : 0;
  const ambiguous = !!second && topTier === secondTier && ratio > 0.75;

  // 신뢰도: 최고 티어 기반
  let conf = topTier===1?95 : topTier===2?85 : topTier===2.5?88 : topTier===3?72 : 50;
  if(topTier < secondTier) conf += 6;   // 더 구체적인 근거로 이겼다면 가산
  if(ambiguous) conf -= 25;
  if(best.hits.some(h=>h.from==='건물명')) conf = Math.min(98, conf+4);
  conf = Math.max(15, Math.min(98, conf));

  return {
    dist: best.d,
    reason: best.hits.slice(0,3).map(h=>`${h.from}"${h.kw}"`).join(' + '),
    conf, ambiguous,
    candidates: ranked.slice(0,4).map(r=>({id:r.d.id, p:r.d.p, score:r.total, hits:r.hits.map(h=>h.kw)})),
    stage:'키워드매칭'
  };
}

/* ===================== 자동완성 엔진 (내장) ===================== */
/* 내장 자동완성 엔진 — 외부 API 없이 동작 */
var CHO=['ㄱ','ㄲ','ㄴ','ㄷ','ㄸ','ㄹ','ㅁ','ㅂ','ㅃ','ㅅ','ㅆ','ㅇ','ㅈ','ㅉ','ㅊ','ㅋ','ㅌ','ㅍ','ㅎ'];
function toCho(s){
  var o='';
  for(var i=0;i<s.length;i++){
    var c=s.charCodeAt(i);
    if(c>=0xAC00&&c<=0xD7A3) o+=CHO[Math.floor((c-0xAC00)/588)];
    else o+=s[i];
  }
  return o;
}
function nk(s){ return (s||'').replace(/[\s\-_,·・()（）]/g,'').toLowerCase(); }

// 지구 데이터 → 검색 인덱스
function buildIndex(D){
  var idx=[];
  var TIER_LABEL={1:'아파트·건물',2:'마을·단지',3:'법정동',4:'지역'};
  D.forEach(function(d){
    (d.roads||[]).forEach(function(rd){
      idx.push({term:rd, tier:2, label:'도로명', id:d.id, p:d.p, n:nk(rd), c:toCho(nk(rd))});
    });
    [['t1',1],['t2',2],['t3',3],['t4',4]].forEach(function(pair){
      d[pair[0]].forEach(function(term){
        var e={term:term, tier:pair[1], label:TIER_LABEL[pair[1]], id:d.id, p:d.p,
               n:nk(term), c:toCho(nk(term))};
        if(nk(term)==='이매촌'){ e.alias=term; e.label='목장 유형 선택'; }
        idx.push(e);
      });
    });
    idx.push({term:d.id, tier:0, label:'지구명', id:d.id, p:d.p, n:nk(d.id), c:toCho(nk(d.id))});
    // 청솔마을처럼 단지번호로 지구가 갈리는 경우 안내용 항목 추가
    if(d.cs){
      var t='청솔마을 '+d.cs[0]+'~'+d.cs[1]+'단지';
      idx.push({term:t, tier:1, label:'단지 범위', id:d.id, p:d.p, n:nk(t), c:toCho(nk('청솔마을'))});
    }
    if(d.imae){
      var ti='이매촌 '+d.imae.join('/')+' 목장';
      idx.push({term:ti, tier:1, label:'목장 유형', id:d.id, p:d.p, n:nk(ti), c:toCho(nk('이매촌'))});
    }
    var pname=d.p.replace(' 목사','');
    idx.push({term:pname+' 목사', tier:0, label:'담당 교역자', id:d.id, p:d.p, n:nk(pname), c:toCho(nk(pname))});
  });
  return idx;
}

// 검색: 접두 > 포함 > 초성
function suggest(q, idx, limit){
  limit=limit||8;
  var n=nk(q);
  if(n.length<1) return [];
  var isChoOnly=/^[ㄱ-ㅎ]+$/.test(n);
  var out=[];
  for(var i=0;i<idx.length;i++){
    var it=idx[i], score=-1;
    if(isChoOnly){
      if(it.c.indexOf(n)===0) score=70;
      else if(it.c.indexOf(n)>-1) score=40;
    }else{
      if(it.n===n) score=100;
      else if(it.n.indexOf(n)===0) score=85;
      else if(it.n.indexOf(n)>-1) score=60;
      else if(it.c.indexOf(n)===0) score=35;
    }
    if(score<0) continue;
    // 구체적인 항목(t1)일수록 가산, 짧을수록 가산
    score += (it.tier===0?6:(5-it.tier)*3);
    score -= Math.min(10, it.n.length*0.3);
    out.push({it:it, score:score});
  }
  out.sort(function(a,b){ return b.score-a.score || a.it.n.length-b.it.n.length; });
  // 같은 (검색어,지구) 중복 제거
  var seen={}, res=[];
  for(var j=0;j<out.length && res.length<limit;j++){
    var itj=out[j].it;
    var k = itj.alias ? ('alias|'+itj.alias) : (itj.term+'|'+itj.id);
    if(seen[k]) continue;
    seen[k]=1; res.push(itj);
  }
  return res;
}

/* ===================== UI ===================== */
(function(){
'use strict';
function $(s){return document.querySelector(s);}
function $$(s){return Array.prototype.slice.call(document.querySelectorAll(s));}
function esc(s){return String(s==null?'':s).replace(/[&<>"']/g,function(c){
  return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c];});}

var pending=null, lastAddr=null;

/* ---- 탭 ---- */
$$('.tab').forEach(function(t){
  t.onclick=function(){
    $$('.tab').forEach(function(x){x.classList.remove('on');});
    t.classList.add('on');
    ['search','bulk','list'].forEach(function(id){
      $('#tab-'+id).classList.toggle('hide', id!==t.dataset.tab);
    });
  };
});

/* ---- 결과 표시 ---- */
function confColor(c){ return c>=85?'#059669':(c>=65?'#d97706':'#dc2626'); }
function show(res, addr){
  lastAddr = addr;
  var card=$('#resultCard');
  card.className='result-card show';
  $('#noticeBox').innerHTML='';
  $('#altBox').innerHTML='';

  if(res && res.needModal){
    $('#resultIcon').innerHTML='&#10067;';
    $('#resultTitle').textContent='선택 필요';
    $('#resultPastor').textContent='';
    $('#resultDetail').innerHTML='<span class="badge">이매촌</span> 목장 유형을 선택해 주세요.';
    return;
  }

  // 청솔 범위 밖 등 확인 필요
  if(res && res.needCheck){
    $('#noticeBox').innerHTML='<div class="notice n-warn"><b>확인이 필요합니다</b><br>'+esc(res.reason)+'</div>';
  }

  if(res && res.dist){
    var d=res.dist;
    $('#resultIcon').innerHTML='&#128205;';
    $('#resultTitle').textContent=d.id;
    $('#resultPastor').textContent=d.p;

    var h='';
    if(addr.roadAddress)  h+='&#128205; <b>도로명:</b> '+esc(addr.roadAddress)+'<br>';
    if(addr.jibunAddress) h+='&#128203; <b>지번:</b> '+esc(addr.jibunAddress)+'<br>';
    if(addr.buildingName) h+='&#127970; <b>건물명:</b> '+esc(addr.buildingName)+'<br>';
    if(addr.query)        h+='&#128269; <b>입력:</b> '+esc(addr.query)+'<br>';
    h+='&#128101; <b>담당 구역:</b> '+esc(d.d)+'<br>';
    h+='&#127919; <b>매칭 근거:</b> '+esc(res.reason||'-');
    if(res.conf!=null){
      var c=res.conf, lab=c>=85?'정확':(c>=65?'보통':'낮음');
      h+='<div class="conf-bar"><i style="width:'+c+'%;background:'+confColor(c)+'"></i></div>';
      h+='<span style="font-size:11.5px;color:'+confColor(c)+';font-weight:700">신뢰도 '+c+'% ('+lab+')</span>';
    }
    $('#resultDetail').innerHTML=h;

    if(res.ambiguous){
      $('#noticeBox').innerHTML='<div class="notice n-warn"><b>비슷한 후보가 있습니다</b><br>'+
        '아래 다른 후보도 함께 확인해 주세요. 애매하면 교구 사무실에 문의 바랍니다.</div>';
    }
    if(res.candidates && res.candidates.length>1){
      var alt='<div class="alt"><div class="alt-title">다른 후보</div>';
      res.candidates.slice(1).forEach(function(c){
        alt+='<div class="alt-item" data-id="'+esc(c.id)+'"><span><b>'+esc(c.id)+'</b> &middot; '+
             esc(c.p)+'</span><span class="s">'+esc(c.hits.slice(0,2).join(', '))+'</span></div>';
      });
      $('#altBox').innerHTML=alt+'</div>';
      $$('#altBox .alt-item').forEach(function(el){
        el.onclick=function(){
          var pick=D.filter(function(x){return x.id===el.dataset.id;})[0];
          if(pick) show({dist:pick,reason:'사용자가 후보 목록에서 직접 선택',conf:70},addr);
        };
      });
    }
  } else {
    $('#resultIcon').innerHTML='&#10060;';
    $('#resultTitle').textContent='미발견';
    $('#resultPastor').textContent='';
    $('#resultDetail').innerHTML='일치하는 교구를 찾을 수 없습니다.<br>'+
      '&#128205; '+esc(addr.roadAddress||addr.jibunAddress||addr.query||'')+
      '<br><br><span style="font-size:12px">아파트 이름을 정확히 입력하거나, <b>지구목록</b> 탭에서 직접 찾아보세요.</span>';
  }
  if(card.scrollIntoView) try{card.scrollIntoView({behavior:'smooth',block:'nearest'});}catch(e){}
}

function run(addr){
  var r = match(addr, D);
  if(r && r.needModal){ pending=addr; $('#imaeModal').classList.add('show'); }
  show(r, addr);
}

/* ---- 카카오 주소 검색 (레이어 embed 방식) ----
   .open() 팝업은 file:// / iframe / 인앱브라우저에서 opener 통신이 막혀
   "창은 뜨는데 선택이 반영되지 않는" 문제가 생긴다.
   .embed()는 같은 문서 안에 iframe으로 삽입되므로 그 문제가 원천적으로 없다. */
var PC_SRC = 'https://t1.kakaocdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js';
var pcLoading = false;

function getPostcodeCtor(){
  if(window.daum && window.daum.Postcode)  return window.daum.Postcode;
  if(window.kakao && window.kakao.Postcode) return window.kakao.Postcode;
  return null;
}
function closePc(){
  $('#pcWrap').classList.remove('show');
  $('#pcLayer').innerHTML='';
}
function offlineNotice(){
  closePc();
  $('#resultCard').className='result-card show';
  $('#noticeBox').innerHTML='<div class="notice n-warn"><b>주소 검색 서비스에 연결할 수 없습니다</b><br>'+
    '인터넷이 차단된 환경이거나 보안 정책으로 막혀 있습니다. '+
    '위의 <b>기본 검색창</b>을 이용해 주세요. 인터넷 없이도 모든 지구를 조회할 수 있습니다.</div>';
  $('#resultIcon').innerHTML='&#128246;';
  $('#resultTitle').textContent='기본 검색창을 이용해 주세요';
  $('#resultPastor').textContent='';
  $('#resultDetail').innerHTML='예) <b>광교레이크포레</b>, <b>청솔마을 7단지</b>, <b>정자동</b>, <b>성복동</b>';
  $('#altBox').innerHTML='';
  $('#q').focus();
}

// 카카오가 돌려준 data → 우리 addr 객체로 변환
function toAddr(data){
  var road  = data.roadAddress  || data.autoRoadAddress  || '';
  var jibun = data.jibunAddress || data.autoJibunAddress || '';
  // 사용자가 지번을 골랐는데 도로명이 비어있는 경우 등 모든 조합을 방어
  return {
    buildingName: data.buildingName || '',
    roadAddress:  road,
    jibunAddress: jibun,
    bname:  data.bname  || '',
    bname1: data.bname1 || '',
    sigungu:data.sigungu|| '',
    sido:   data.sido   || '',
    zonecode: data.zonecode || ''
  };
}

function embedPostcode(){
  var Ctor = getPostcodeCtor();
  if(!Ctor){ offlineNotice(); return; }

  var wrap = $('#pcWrap'), layer = $('#pcLayer');
  layer.innerHTML='';
  wrap.classList.add('show');

  try{
    new Ctor({
      oncomplete: function(data){
        try{
          closePc();
          run(toAddr(data));
        }catch(err){
          // 콜백 내부 예외가 조용히 삼켜지지 않도록 사용자에게 알린다
          closePc();
          alert('주소 처리 중 오류가 발생했습니다: ' + (err && err.message ? err.message : err));
        }
      },
      onclose: function(state){
        // 사용자가 X로 닫았을 때 레이어 정리 (state: 'FORCE_CLOSE' | 'COMPLETE_CLOSE')
        if(state === 'FORCE_CLOSE') closePc();
      },
      onresize: function(size){
        if(size && size.height) layer.style.height = Math.max(360, size.height) + 'px';
      },
      width:'100%', height:'100%',
      maxSuggestItems: 5
    }).embed(layer, {autoClose:false, q: ($('#q').value||'').trim() });
  }catch(e){
    closePc();
    alert('주소 검색을 여는 중 오류가 발생했습니다: ' + (e && e.message ? e.message : e));
  }
}

function openAddressSearch(){
  // 이미 열려 있으면 닫기(토글)
  if($('#pcWrap').classList.contains('show')){ closePc(); return; }

  if(getPostcodeCtor()){ embedPostcode(); return; }
  if(pcLoading) return;
  pcLoading = true;

  // 로딩 표시
  $('#pcWrap').classList.add('show');
  $('#pcLayer').innerHTML='<div class="pc-loading">주소 검색을 불러오는 중…</div>';

  var s=document.createElement('script');
  s.src=PC_SRC; s.async=true;
  var done=false;
  var timer=setTimeout(function(){
    if(done) return; done=true; pcLoading=false;
    offlineNotice();
  }, 8000);   // 8초 내 미로딩 시 오프라인 안내

  s.onload=function(){
    if(done) return; done=true; clearTimeout(timer); pcLoading=false;
    if(getPostcodeCtor()) embedPostcode(); else offlineNotice();
  };
  s.onerror=function(){
    if(done) return; done=true; clearTimeout(timer); pcLoading=false;
    offlineNotice();
  };
  document.head.appendChild(s);
}

$('#searchBtn').onclick=openAddressSearch;
$('#pcClose').onclick=closePc;

/* ---- 기본 검색 (내장 인덱스 · 외부 API 불필요) ---- */
var IDX = buildIndex(D), sIdx = -1, sList = [];

function hideSugg(){ $('#sugg').classList.remove('show'); sIdx=-1; sList=[]; }

function paintSugg(){
  $$('#sugg .sugg-item').forEach(function(el,i){ el.classList.toggle('on', i===sIdx); });
}

function renderSugg(){
  var v=$('#q').value.trim();
  if(!v){ hideSugg(); return; }
  sList = suggest(v, IDX, 8);
  if(!sList.length){
    $('#sugg').innerHTML='<div class="sugg-empty">검색 결과가 없습니다.<br>'+
      '<span style="font-size:12px">아파트명 일부나 동 이름으로 다시 시도해 보세요.</span></div>';
    $('#sugg').classList.add('show'); sIdx=-1; return;
  }
  $('#sugg').innerHTML = sList.map(function(it,i){
    return '<div class="sugg-item" data-i="'+i+'">'+
      '<span><span class="sugg-term">'+esc(it.term)+'</span>'+
      '<span class="sugg-tag">'+esc(it.label)+'</span></span>'+
      '<span class="sugg-meta">'+(it.alias?'선택 필요<small>목장 유형</small>':esc(it.id)+'<small>'+esc(it.p)+'</small>')+'</span></div>';
  }).join('');
  $('#sugg').classList.add('show'); sIdx=-1;
  $$('#sugg .sugg-item').forEach(function(el){
    el.addEventListener('mousedown', function(e){
      e.preventDefault();
      pickSugg(sList[+el.dataset.i]);
    });
  });
}

// 자동완성 항목 확정 → 바로 결과 표시
function pickSugg(it){
  if(!it) return;
  hideSugg();
  $('#q').value = it.alias || it.term;
  // 이매촌/청솔 등 분기 항목은 매칭 엔진을 그대로 태워 모달·규칙을 살린다
  if(it.alias){ runQuery(it.alias); return; }
  var dist = D.filter(function(d){ return d.id===it.id; })[0];
  if(!dist){ runQuery(it.term); return; }
  show({dist:dist, reason:'"'+it.term+'" ('+it.label+') 선택', conf: it.tier===0?92:(it.tier===1?97:(it.tier===2?90:80))},
       {query:it.term});
}

function runQuery(v){
  v=(v==null?$('#q').value:v).trim();
  if(!v){ $('#q').focus(); return; }
  hideSugg();
  run({buildingName:v, roadAddress:'', jibunAddress:v, bname:'', query:v});
}

$('#qBtn').onclick=function(){
  // 자동완성이 열려있고 정확히 1건이면 그것을 채택
  if(sList.length===1){ pickSugg(sList[0]); return; }
  runQuery();
};
$('#q').addEventListener('input', renderSugg);
$('#q').addEventListener('focus', function(){ if($('#q').value.trim()) renderSugg(); });
$('#q').addEventListener('blur', function(){ setTimeout(hideSugg,150); });
$('#q').addEventListener('keydown', function(e){
  var n=sList.length;
  if(e.key==='ArrowDown' && n){ e.preventDefault(); sIdx=Math.min(sIdx+1,n-1); paintSugg(); }
  else if(e.key==='ArrowUp' && n){ e.preventDefault(); sIdx=Math.max(sIdx-1,-1); paintSugg(); }
  else if(e.key==='Enter'){
    e.preventDefault();
    if(sIdx>-1 && sList[sIdx]) pickSugg(sList[sIdx]);
    else if(n===1) pickSugg(sList[0]);
    else runQuery();
  }
  else if(e.key==='Escape') hideSugg();
});
$$('.ex').forEach(function(el){
  el.onclick=function(){ $('#q').value=el.textContent; runQuery(); };
});

/* ---- 이매촌 모달 ---- */
$$('#imaeModal .modal-btn').forEach(function(btn){
  btn.onclick=function(){
    var type=btn.getAttribute('data-type');
    $('#imaeModal').classList.remove('show');
    if(type==='__cancel'){ pending=null; return; }
    if(!pending) return;
    var id = type==='자매' ? '분당 3지구' : '분당 2지구';
    var dist = D.filter(function(d){return d.id===id;})[0];
    show({dist:dist, reason:'이매촌 '+type+' 목장 규칙', conf:96}, pending);
    pending=null;
  };
});
$('#imaeModal').onclick=function(e){
  if(e.target===this){ this.classList.remove('show'); pending=null; }
};
document.addEventListener('keydown',function(e){
  if(e.key!=='Escape') return;
  if($('#imaeModal').classList.contains('show')){
    $('#imaeModal').classList.remove('show'); pending=null; return;
  }
  if($('#pcWrap').classList.contains('show')) closePc();
});

/* ---- 일괄 ---- */
var bulkRows=[];
$('#btnBulkSample').onclick=function(){
  $('#bulkIn').value=['광교레이크포레','광교산자이','청솔마을 7단지','정자동','성복동',
    '경기 용인시 기흥구 보정동','한라비발디','수지자이'].join('\n');
};
$('#btnBulk').onclick=function(){
  var lines=$('#bulkIn').value.split('\n').map(function(s){return s.trim();}).filter(Boolean);
  if(!lines.length){alert('한 줄에 하나씩 입력해 주세요.');return;}
  bulkRows=lines.map(function(v){
    var r=match({buildingName:v,roadAddress:'',jibunAddress:v,bname:''},D);
    return {q:v,
      id:(r.needModal?'(이매촌: 선택 필요)':(r.dist?r.dist.id:'미발견')),
      p:(r.dist?r.dist.p:''), conf:(r.conf||0),
      why:(r.needModal?'목장 유형 선택 필요':(r.reason||''))};
  });
  var ok=bulkRows.filter(function(r){return r.conf>=85;}).length;
  var need=bulkRows.filter(function(r){return r.conf<65;}).length;
  $('#bulkStat').innerHTML='총 <b>'+bulkRows.length+'</b>건 &middot; 정확 <b style="color:#059669">'+ok+
    '</b> &middot; 확인필요 <b style="color:#dc2626">'+need+'</b>';
  $('#bulkTbl').innerHTML='<thead><tr><th>#</th><th>입력</th><th>지구</th><th>담당</th><th>신뢰도</th></tr></thead><tbody>'+
    bulkRows.map(function(r,i){
      return '<tr><td>'+(i+1)+'</td><td>'+esc(r.q)+'</td><td><b>'+esc(r.id)+'</b></td>'+
        '<td>'+esc(r.p||'-')+'</td><td style="color:'+confColor(r.conf)+';font-weight:700">'+r.conf+'%</td></tr>';
    }).join('')+'</tbody>';
  $('#bulkWrap').classList.remove('hide');
};
$('#btnBulkCsv').onclick=function(){
  if(!bulkRows.length){alert('먼저 일괄 조회를 실행해 주세요.');return;}
  var csv='입력,지구,담당목사,신뢰도,매칭근거\n'+bulkRows.map(function(r){
    return [r.q,r.id,r.p,r.conf+'%',r.why].map(function(v){
      return '"'+String(v==null?'':v).replace(/"/g,'""')+'"';}).join(',');
  }).join('\n');
  var blob=new Blob(['\ufeff'+csv],{type:'text/csv;charset=utf-8;'});
  var a=document.createElement('a');
  a.href=URL.createObjectURL(blob); a.download='교구조회결과.csv';
  document.body.appendChild(a); a.click();
  setTimeout(function(){URL.revokeObjectURL(a.href);a.remove();},400);
};

/* ---- 지구 목록 ---- */
function renderList(kw){
  var k=(kw||'').replace(/\s/g,'').toLowerCase();
  var rows=D.filter(function(d){
    if(!k) return true;
    return (d.id+d.p+d.d).replace(/\s/g,'').toLowerCase().indexOf(k)>-1;
  });
  $('#listTbl').innerHTML='<thead><tr><th style="width:88px">지구</th><th style="width:82px">담당</th><th>담당 구역</th></tr></thead><tbody>'+
    rows.map(function(d){
      return '<tr data-q="'+esc((d.t1[0]||d.t2[0]||d.t3[0]||d.id))+'">'+
        '<td><b>'+esc(d.id)+'</b></td><td>'+esc(d.p)+'</td>'+
        '<td style="color:#64748b">'+esc(d.d)+'</td></tr>';
    }).join('')+'</tbody>';
  $('#listCnt').textContent=rows.length;
  $$('#listTbl tbody tr').forEach(function(tr){
    tr.onclick=function(){
      $('#q').value=tr.dataset.q;
      $$('.tab').forEach(function(x){x.classList.remove('on');});
      document.querySelector('.tab[data-tab="search"]').classList.add('on');
      ['search','bulk','list'].forEach(function(id){
        $('#tab-'+id).classList.toggle('hide', id!=='search');
      });
      runQuery(tr.dataset.q);
    };
  });
}
$('#listSrch').addEventListener('input',function(){renderList(this.value);});
renderList('');
})();
</script>
</body>
</html>
