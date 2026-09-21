<!DOCTYPE html>
<html lang="ko">
<head>
  google.com, pub-2429123210227554, DIRECT, f08c47fec0942fa0
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>맛지도 (TasteMap) - 네이버 & 카카오 평점 기반 맛집 탐색</title>
  
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- Font Awesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
  
  <!-- Leaflet CSS & JS for Interactive Map -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin="" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>

  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            naver: '#03C75A',
            kakao: '#FEE500',
            kakaoText: '#391B1B',
            brand: {
              50: '#FFF7ED',
              100: '#FFEDD5',
              500: '#F97316',
              600: '#EA580C',
              700: '#C2410C'
            }
          }
        }
      }
    }
  </script>

  <style>
    @import url('https://fonts.googleapis.com/css2?family=Pretendard:wght@300;400;500;600;700;800&display=swap');
    body {
      font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, system-ui, Roboto, sans-serif;
    }
    .custom-scrollbar::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    .custom-scrollbar::-webkit-scrollbar-track {
      background: #f1f5f9;
    }
    .custom-scrollbar::-webkit-scrollbar-thumb {
      background: #cbd5e1;
      border-radius: 9999px;
    }
    .custom-scrollbar::-webkit-scrollbar-thumb:hover {
      background: #94a3b8;
    }
    /* Leaflet popup styling */
    .leaflet-popup-content-wrapper {
      border-radius: 1rem;
      padding: 4px;
      box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.15), 0 8px 10px -6px rgba(0, 0, 0, 0.1);
    }
    .leaflet-popup-content {
      margin: 8px 12px;
    }
  </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen flex flex-col antialiased selection:bg-orange-100 selection:text-orange-900">

  <header class="sticky top-0 z-40 bg-white/90 backdrop-blur-md border-b border-slate-200 shadow-sm transition-all">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="flex items-center justify-between h-16 gap-4">
        <!-- Logo -->
        <div class="flex items-center space-x-2.5 cursor-pointer select-none" onclick="resetFilters()">
          <div class="w-10 h-10 rounded-2xl bg-gradient-to-tr from-orange-500 to-amber-500 flex items-center justify-center text-white shadow-md shadow-orange-500/20">
            <i class="fa-solid fa-utensils text-lg"></i>
          </div>
          <div>
            <div class="font-extrabold text-xl tracking-tight bg-gradient-to-r from-orange-600 via-amber-600 to-rose-600 bg-clip-text text-transparent">
              맛지도 <span class="text-xs font-semibold px-1.5 py-0.5 rounded bg-orange-100 text-orange-700 ml-1">v2.0</span>
            </div>
            <p class="text-[11px] text-slate-500 -mt-1 hidden sm:block">전국 지역 검색 & 네이버·카카오 검증 맛집</p>
          </div>
        </div>

        <!-- Enhanced Location Search & Quick Region Selector -->
        <div class="flex items-center gap-1.5 bg-slate-100/90 hover:bg-slate-200/70 p-1.5 rounded-2xl border border-slate-200 transition max-w-md w-full sm:w-auto">
          <div class="flex items-center gap-1 pl-2 text-orange-500 text-sm shrink-0">
            <i class="fa-solid fa-location-dot"></i>
          </div>
          <!-- Region Input for ANY location -->
          <div class="relative flex-1">
            <input type="text" id="regionSearchInput" 
                   placeholder="지역명·동·역 검색 (예: 성수동, 을지로, 서귀포, 광안리)" 
                   onkeydown="if(event.key === 'Enter') handleRegionSearch()"
                   class="w-full sm:w-60 bg-transparent text-xs sm:text-sm font-semibold text-slate-800 placeholder:text-slate-400 focus:outline-none pr-6">
            <button onclick="handleRegionSearch()" title="지역 검색" class="absolute right-0 top-1/2 -translate-y-1/2 text-orange-600 hover:text-orange-700 text-xs px-1">
              <i class="fa-solid fa-magnifying-glass"></i>
            </button>
          </div>

          <!-- Quick Dropdown presets -->
          <select id="locationSelect" onchange="changeRegion(this.value)" class="bg-white border border-slate-200 rounded-xl font-medium text-xs text-slate-700 py-1 px-2 outline-none cursor-pointer hidden md:block">
            <option value="all">전국 전체</option>
            <option value="seongsu">서울 성수·서울숲</option>
            <option value="euljiro">서울 을지로·종로</option>
            <option value="gangnam">서울 강남·신사</option>
            <option value="hongdae">서울 마포·홍대·연남</option>
            <option value="bundang">성남 분당·판교</option>
            <option value="busan">부산 해운대·광안리</option>
            <option value="jeju">제주 제주시·애월</option>
          </select>

          <!-- GPS Detect Button -->
          <button onclick="getUserLocation()" title="현재 GPS 위치 찾기" class="text-xs text-slate-500 hover:text-orange-600 p-1.5 bg-white rounded-xl border border-slate-200 transition shrink-0">
            <i class="fa-solid fa-crosshairs"></i>
          </button>
        </div>

        <!-- Top Right Actions -->
        <div class="flex items-center space-x-2">
          <!-- Kakao REST API Setting Button -->
          <button onclick="openKakaoModal()" id="kakaoApiBtn" class="flex items-center space-x-1.5 px-2.5 py-1.5 rounded-xl bg-yellow-400 hover:bg-yellow-500 text-[#391B1B] text-xs sm:text-sm font-bold border border-yellow-500/80 transition shadow-xs active:scale-95" title="카카오 로컬 REST API 설정">
            <i class="fa-solid fa-key text-xs"></i>
            <span class="hidden sm:inline">카카오 API</span>
            <span id="kakaoStatusDot" class="w-2 h-2 rounded-full bg-slate-500"></span>
          </button>

          <button onclick="openRouletteModal()" class="flex items-center space-x-1.5 px-3 py-1.5 rounded-xl bg-orange-50 text-orange-600 hover:bg-orange-100 text-xs sm:text-sm font-semibold border border-orange-200 transition shadow-sm active:scale-95">
            <i class="fa-solid fa-dice text-amber-500 animate-bounce"></i>
            <span class="hidden md:inline">오늘 뭐 먹지?</span>
            <span class="md:hidden">추천</span>
          </button>
          <button onclick="toggleFavoritesOnly()" id="favToggleBtn" class="flex items-center space-x-1 px-3 py-1.5 rounded-xl bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs sm:text-sm font-medium border border-slate-200 transition">
            <i class="fa-regular fa-heart text-rose-500"></i>
            <span id="favCount" class="font-bold text-rose-600 ml-0.5">0</span>
          </button>
        </div>
      </div>
    </div>
  </header>

  <div class="bg-white border-b border-slate-200 shadow-xs sticky top-16 z-30">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-2.5">
      
      <!-- Category Tabs (Horizontal Scrollable) -->
      <div class="flex items-center space-x-2 overflow-x-auto pb-1.5 custom-scrollbar select-none" id="categoryTabs">
        <!-- Injected via JavaScript -->
      </div>

      <!-- Popular Region Quick Chips -->
      <div class="flex items-center gap-1.5 overflow-x-auto pb-1.5 text-xs text-slate-600 custom-scrollbar">
        <span class="font-bold text-slate-400 shrink-0 text-[11px]"><i class="fa-solid fa-fire text-rose-500 mr-1"></i>핫플레이스:</span>
        <button onclick="selectQuickRegion('seongsu')" class="region-chip px-2.5 py-0.5 rounded-full bg-slate-100 hover:bg-orange-50 hover:text-orange-600 border border-slate-200 transition shrink-0">성수동</button>
        <button onclick="selectQuickRegion('euljiro')" class="region-chip px-2.5 py-0.5 rounded-full bg-slate-100 hover:bg-orange-50 hover:text-orange-600 border border-slate-200 transition shrink-0">을지로·힙지로</button>
        <button onclick="selectQuickRegion('hongdae')" class="region-chip px-2.5 py-0.5 rounded-full bg-slate-100 hover:bg-orange-50 hover:text-orange-600 border border-slate-200 transition shrink-0">홍대·연남</button>
        <button onclick="selectQuickRegion('gangnam')" class="region-chip px-2.5 py-0.5 rounded-full bg-slate-100 hover:bg-orange-50 hover:text-orange-600 border border-slate-200 transition shrink-0">강남·신사</button>
        <button onclick="selectQuickRegion('bundang')" class="region-chip px-2.5 py-0.5 rounded-full bg-slate-100 hover:bg-orange-50 hover:text-orange-600 border border-slate-200 transition shrink-0">판교·분당</button>
        <button onclick="selectQuickRegion('busan')" class="region-chip px-2.5 py-0.5 rounded-full bg-slate-100 hover:bg-orange-50 hover:text-orange-600 border border-slate-200 transition shrink-0">부산 해운대</button>
        <button onclick="selectQuickRegion('jeju')" class="region-chip px-2.5 py-0.5 rounded-full bg-slate-100 hover:bg-orange-50 hover:text-orange-600 border border-slate-200 transition shrink-0">제주 애월</button>
        <button onclick="selectQuickRegion('all')" class="region-chip px-2.5 py-0.5 rounded-full bg-slate-100 hover:bg-orange-50 hover:text-orange-600 border border-slate-200 transition shrink-0">전체보기</button>
      </div>

      <!-- Secondary Controls: Search & Rating Badges & Sort -->
      <div class="flex flex-wrap items-center justify-between gap-2.5 pt-2 border-t border-slate-100 text-xs sm:text-sm">
        <!-- Search Input -->
        <div class="relative flex-1 min-w-[200px] max-w-sm">
          <i class="fa-solid fa-magnifying-glass absolute left-3 top-1/2 -translate-y-1/2 text-slate-400 text-xs"></i>
          <input type="text" id="searchInput" oninput="applyFilters()" placeholder="식당 이름, 대표 메뉴 검색 (예: 삼겹살, 스시)" 
                 class="w-full pl-8 pr-3 py-1.5 text-xs bg-slate-100 border border-slate-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-orange-500/30 focus:border-orange-500 transition">
        </div>

        <!-- Rating Filter Quick Badges -->
        <div class="flex items-center gap-1.5 overflow-x-auto">
          <button onclick="toggleFilter('kakao4')" id="filterKakao4" class="px-2.5 py-1 rounded-lg text-xs font-semibold border border-yellow-300 bg-yellow-50 text-yellow-800 hover:bg-yellow-100 transition flex items-center gap-1">
            <span class="w-2 h-2 rounded-full bg-yellow-500"></span>
            카카오 4.0+ (찐맛집)
          </button>
          <button onclick="toggleFilter('naver45')" id="filterNaver45" class="px-2.5 py-1 rounded-lg text-xs font-semibold border border-emerald-300 bg-emerald-50 text-emerald-800 hover:bg-emerald-100 transition flex items-center gap-1">
            <span class="w-2 h-2 rounded-full bg-emerald-500"></span>
            네이버 4.5+
          </button>
          <button onclick="toggleFilter('parking')" id="filterParking" class="px-2.5 py-1 rounded-lg text-xs font-medium border border-slate-200 bg-slate-50 text-slate-700 hover:bg-slate-100 transition">
            🚗 주차 가능
          </button>
        </div>

        <!-- View Switch & Sorting Dropdown -->
        <div class="flex items-center space-x-2 ml-auto">
          <select id="sortSelect" onchange="applyFilters()" class="bg-slate-100 border border-slate-200 text-slate-700 text-xs rounded-lg px-2.5 py-1.5 font-medium outline-none cursor-pointer focus:ring-1 focus:ring-orange-500">
            <option value="recommend">추천순 (평점+리뷰 종합)</option>
            <option value="kakaoRating">카카오맵 평점 높은순</option>
            <option value="naverRating">네이버 평점 높은순</option>
            <option value="distance">거리 가까운순</option>
            <option value="reviewCount">리뷰 많은순</option>
          </select>

          <!-- Toggle View (Cards vs Map Split) -->
          <div class="flex rounded-lg border border-slate-200 p-0.5 bg-slate-100">
            <button onclick="setViewMode('grid')" id="viewGridBtn" class="px-2 py-1 text-xs rounded-md bg-white shadow-xs font-semibold text-orange-600 transition" title="목록형">
              <i class="fa-solid fa-table-cells-large"></i>
            </button>
            <button onclick="setViewMode('split')" id="viewSplitBtn" class="px-2 py-1 text-xs rounded-md text-slate-600 hover:text-slate-900 transition" title="지도 분할형">
              <i class="fa-solid fa-map-location-dot"></i>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-5 flex-1 w-full">
    
    <!-- Notice & Platform Comparison Banner -->
    <div class="mb-4 bg-gradient-to-r from-orange-50 via-amber-50 to-emerald-50 border border-orange-200/70 rounded-2xl p-3 sm:p-4 text-xs sm:text-sm flex flex-col md:flex-row md:items-center justify-between gap-2 shadow-xs">
      <div class="flex items-center gap-3">
        <div class="w-8 h-8 rounded-xl bg-orange-500 text-white flex items-center justify-center font-bold text-sm shrink-0">
          Tip
        </div>
        <div>
          <span class="font-bold text-slate-900">네이버 & 카카오맵 평점 팩트체크:</span>
          <span class="text-slate-600 ml-1">
            카카오맵은 평점 <strong>4.0점 이상</strong>이면 현지인 인증 찐맛집, 네이버 평점은 <strong>4.5점 이상</strong>이면 대중적인 만족도가 매우 높은 곳입니다!
          </span>
        </div>
      </div>
      <div class="flex items-center gap-2 self-end md:self-auto text-xs shrink-0 font-medium">
        <span class="inline-flex items-center gap-1 bg-[#03C75A]/10 text-[#029543] px-2 py-0.5 rounded-md border border-[#03C75A]/30">
          <i class="fa-solid fa-n text-[10px]"></i> 네이버 평점
        </span>
        <span class="inline-flex items-center gap-1 bg-[#FEE500]/40 text-[#543b00] px-2 py-0.5 rounded-md border border-yellow-400">
          <i class="fa-solid fa-comment text-[10px]"></i> 카카오맵 평점
        </span>
      </div>
    </div>

    <!-- Layout Container: Split view or full grid -->
    <div id="contentContainer" class="flex flex-col lg:flex-row gap-5">
      
      <!-- Restaurant Cards Column -->
      <div id="listColumn" class="w-full transition-all duration-300">
        <!-- Results count & reset quick clear -->
        <div class="flex items-center justify-between mb-3 text-xs text-slate-500 font-medium">
          <div>
            현재 탐색 지역: <span id="currentRegionLabel" class="font-bold text-slate-800">전국 인기 맛집</span>
            <span class="mx-1.5">•</span>
            식당 <span id="restaurantCount" class="font-bold text-orange-600">0</span>곳
            <span id="activeFilterBadge" class="hidden ml-2 px-2 py-0.5 bg-orange-100 text-orange-700 rounded-full font-semibold">필터 적용중</span>
          </div>
          <button onclick="resetFilters()" class="text-slate-400 hover:text-slate-700 hover:underline text-xs">
            <i class="fa-solid fa-rotate-right mr-1"></i>조건 초기화
          </button>
        </div>

        <div id="aiSearchBanner" class="hidden mb-4 p-3 bg-gradient-to-r from-orange-500/10 via-amber-500/10 to-orange-500/5 border border-orange-200 rounded-2xl flex flex-col sm:flex-row sm:items-center justify-between gap-2">
          <div class="flex items-center gap-2.5">
            <div class="w-8 h-8 rounded-xl bg-orange-500 text-white flex items-center justify-center font-bold text-sm">
              <i class="fa-solid fa-wand-magic-sparkles"></i>
            </div>
            <div>
              <span class="font-bold text-slate-800 text-xs sm:text-sm" id="aiBannerText">새로운 지역 맛집을 더 찾고 계신가요?</span>
              <p class="text-[11px] text-slate-500">네이버·카카오 검증 평점을 갖춘 이 지역의 대표 맛집을 AI로 즉시 추가합니다.</p>
            </div>
          </div>
          <button id="btnFetchAiRestaurants" onclick="fetchRestaurantsForCurrentSearch()" class="px-3.5 py-2 bg-orange-600 hover:bg-orange-700 text-white rounded-xl text-xs font-bold transition shadow-sm flex items-center justify-center gap-1.5 shrink-0 active:scale-95">
            <i class="fa-solid fa-bolt"></i> 이 지역 맛집 AI 즉시 발굴
          </button>
        </div>

        <!-- Cards Grid -->
        <div id="restaurantGrid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
          <!-- Dynamically filled with restaurant cards -->
        </div>

        <!-- Empty State with dynamic AI search prompt -->
        <div id="emptyState" class="hidden py-16 px-4 text-center bg-white rounded-2xl border border-dashed border-slate-300">
          <div class="w-16 h-16 bg-orange-50 text-orange-500 rounded-2xl flex items-center justify-center mx-auto mb-3 text-3xl">
            <i class="fa-solid fa-compass"></i>
          </div>
          <h3 class="text-base font-bold text-slate-800" id="emptyStateTitle">해당 지역에 등록된 식당이 아직 없나요?</h3>
          <p class="text-xs text-slate-500 mt-1 max-w-sm mx-auto" id="emptyStateDesc">
            원하시는 지역(동·역·도시명)을 검색하셨다면 AI 실시간 탐색을 통해 네이버/카카오 평점이 보증된 찐맛집들을 바로 불러올 수 있습니다!
          </p>
          <div class="mt-4 flex flex-wrap justify-center gap-2">
            <button onclick="fetchRestaurantsForCurrentSearch()" class="px-4 py-2.5 bg-orange-500 hover:bg-orange-600 text-white rounded-xl text-xs font-bold transition shadow-sm flex items-center gap-1.5">
              <i class="fa-solid fa-wand-magic-sparkles"></i> <span id="emptyAiBtnText">AI로 이 지역 맛집 발굴하기</span>
            </button>
            <button onclick="resetFilters()" class="px-4 py-2.5 bg-slate-100 text-slate-700 hover:bg-slate-200 rounded-xl text-xs font-semibold transition">
              전체 맛집 보기
            </button>
          </div>
        </div>
      </div>

      <!-- Map Column -->
      <div id="mapColumn" class="hidden lg:w-[45%] sticky top-36 h-[calc(100vh-10rem)] transition-all duration-300">
        <div class="w-full h-full bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden relative">
          <div id="map" class="w-full h-full rounded-2xl z-10"></div>
          <div class="absolute bottom-3 left-3 z-20 bg-white/90 backdrop-blur-md px-3 py-1.5 rounded-xl border border-slate-200 text-[11px] font-semibold text-slate-700 shadow-xs flex items-center gap-1.5 pointer-events-none">
            <i class="fa-solid fa-location-dot text-orange-500"></i>
            <span>지도의 마커를 클릭하여 맛집 정보를 확인하세요</span>
          </div>
        </div>
      </div>

    </div>
  </main>

  <!-- Restaurant Detail Modal -->
  <div id="detailModal" class="fixed inset-0 z-50 hidden flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-xs">
    <div class="bg-white rounded-3xl max-w-lg w-full overflow-hidden shadow-2xl relative max-h-[90vh] flex flex-col">
      <!-- Modal Header with Image -->
      <div class="relative h-48 sm:h-56 w-full bg-slate-200 shrink-0">
        <img id="modalImg" src="" alt="식당 이미지" class="w-full h-full object-cover">
        <div class="absolute inset-0 bg-gradient-to-t from-black/80 via-black/20 to-transparent"></div>
        <button onclick="closeModal('detailModal')" class="absolute top-4 right-4 w-8 h-8 rounded-full bg-black/50 hover:bg-black/80 text-white flex items-center justify-center transition">
          <i class="fa-solid fa-xmark text-sm"></i>
        </button>

        <div class="absolute bottom-3 left-4 right-4 text-white">
          <span id="modalCategoryBadge" class="px-2 py-0.5 text-[11px] font-bold rounded-full bg-orange-500/90 backdrop-blur-md"></span>
          <h2 id="modalTitle" class="text-xl font-bold tracking-tight text-white mt-1 drop-shadow-sm"></h2>
          <p id="modalDesc" class="text-xs text-slate-200 mt-0.5 line-clamp-1"></p>
        </div>
      </div>

      <!-- Modal Body (Scrollable) -->
      <div class="p-5 overflow-y-auto custom-scrollbar flex-1 space-y-4 text-xs sm:text-sm">
        
        <!-- Ratings Comparison Box -->
        <div class="grid grid-cols-2 gap-3 p-3 bg-slate-50 border border-slate-200 rounded-2xl">
          <!-- Naver Rating Box -->
          <div class="bg-white p-3 rounded-xl border border-emerald-100 flex items-center gap-3">
            <div class="w-10 h-10 rounded-xl bg-[#03C75A] text-white flex items-center justify-center font-black text-lg shadow-sm">
              N
            </div>
            <div>
              <div class="text-[11px] text-slate-400 font-semibold">네이버 지도 평점</div>
              <div class="flex items-baseline gap-1">
                <span id="modalNaverRate" class="text-lg font-extrabold text-slate-900">4.72</span>
                <span class="text-slate-400 text-xs">/ 5.0</span>
              </div>
              <span id="modalNaverReviews" class="text-[10px] text-emerald-600 font-medium">리뷰 1,420개</span>
            </div>
          </div>

          <!-- Kakao Rating Box -->
          <div class="bg-white p-3 rounded-xl border border-yellow-200 flex items-center gap-3">
            <div class="w-10 h-10 rounded-xl bg-[#FEE500] text-[#391B1B] flex items-center justify-center font-black text-lg shadow-sm">
              <i class="fa-solid fa-comment"></i>
            </div>
            <div>
              <div class="text-[11px] text-slate-400 font-semibold">카카오맵 평점</div>
              <div class="flex items-baseline gap-1">
                <span id="modalKakaoRate" class="text-lg font-extrabold text-slate-900">4.3</span>
                <span class="text-slate-400 text-xs">/ 5.0</span>
              </div>
              <span id="modalKakaoReviews" class="text-[10px] text-yellow-700 font-medium">리뷰 389개</span>
            </div>
          </div>
        </div>

        <!-- Key Information -->
        <div class="space-y-2 text-slate-600 bg-white p-3 border border-slate-100 rounded-2xl">
          <div class="flex items-start gap-2.5">
            <i class="fa-solid fa-location-dot text-orange-500 mt-1 w-4 text-center"></i>
            <div class="flex-1">
              <div class="font-semibold text-slate-800" id="modalAddress"></div>
              <div class="text-[11px] text-slate-400" id="modalDistance"></div>
            </div>
            <button onclick="copyAddress()" class="text-xs px-2 py-1 bg-slate-100 hover:bg-slate-200 text-slate-600 rounded-md font-medium transition">
              복사
            </button>
          </div>
          <div class="flex items-center gap-2.5">
            <i class="fa-regular fa-clock text-slate-400 w-4 text-center"></i>
            <span id="modalHours">11:00 ~ 22:00 (브레이크타임 15:00~17:00)</span>
          </div>
          <div class="flex items-center gap-2.5">
            <i class="fa-solid fa-phone text-slate-400 w-4 text-center"></i>
            <span id="modalPhone">031-700-1234</span>
          </div>
        </div>

        <!-- Signature Menu List -->
        <div>
          <h4 class="font-bold text-slate-800 mb-2 flex items-center gap-1.5">
            <i class="fa-solid fa-utensils text-orange-500"></i> 대표 시그니처 메뉴
          </h4>
          <div id="modalMenuList" class="space-y-1.5">
            <!-- Menu items inserted here -->
          </div>
        </div>

        <!-- Tags -->
        <div>
          <h4 class="font-bold text-slate-800 mb-2 flex items-center gap-1.5">
            <i class="fa-solid fa-tags text-orange-500"></i> 특징 & 편의시설
          </h4>
          <div id="modalTags" class="flex flex-wrap gap-1.5"></div>
        </div>

      </div>

      <!-- Modal Footer with Real Map External Links -->
      <div class="p-4 bg-slate-50 border-t border-slate-200 flex items-center gap-2">
        <a id="btnNaverMap" href="#" target="_blank" rel="noopener noreferrer" class="flex-1 py-2.5 px-3 rounded-xl bg-[#03C75A] hover:bg-[#02b350] text-white font-bold text-xs flex items-center justify-center gap-1.5 shadow-sm transition">
          <i class="fa-solid fa-square-arrow-up-right"></i> 네이버 지도로 보기
        </a>
        <a id="btnKakaoMap" href="#" target="_blank" rel="noopener noreferrer" class="flex-1 py-2.5 px-3 rounded-xl bg-[#FEE500] hover:bg-[#ebd300] text-[#391B1B] font-bold text-xs flex items-center justify-center gap-1.5 shadow-sm transition">
          <i class="fa-solid fa-comment"></i> 카카오맵 길찾기
        </a>
      </div>
    </div>
  </div>

  <div id="rouletteModal" class="fixed inset-0 z-50 hidden flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-xs">
    <div class="bg-white rounded-3xl max-w-sm w-full p-6 text-center shadow-2xl relative">
      <button onclick="closeModal('rouletteModal')" class="absolute top-4 right-4 text-slate-400 hover:text-slate-600">
        <i class="fa-solid fa-xmark text-lg"></i>
      </button>

      <div class="w-16 h-16 bg-gradient-to-tr from-amber-400 to-orange-500 text-white rounded-2xl flex items-center justify-center mx-auto text-2xl shadow-lg shadow-orange-500/30 mb-3 animate-bounce">
        <i class="fa-solid fa-dice"></i>
      </div>

      <h3 class="text-xl font-black text-slate-900">오늘 점심·저녁 뭐 먹지?</h3>
      <p class="text-xs text-slate-500 mt-1">선택 장애는 이제 그만! 현재 지역의 검증된 맛집 중 무작위로 하나를 추천해 드려요.</p>

      <!-- Slot Display Box -->
      <div class="my-5 p-5 bg-gradient-to-b from-orange-50 to-amber-50 rounded-2xl border-2 border-orange-200 flex flex-col items-center justify-center min-h-[120px]">
        <div id="rouletteCategory" class="text-xs font-bold text-orange-600 mb-1">식당을 골라보세요</div>
        <div id="rouletteTitle" class="text-lg font-black text-slate-800">???</div>
        <div id="rouletteRating" class="text-xs text-slate-500 mt-1 flex items-center gap-2">
          <span>네이버 -</span> • <span>카카오 -</span>
        </div>
      </div>

      <div class="flex gap-2">
        <button id="btnSpinRoulette" onclick="spinRoulette()" class="flex-1 py-3 rounded-xl bg-orange-500 hover:bg-orange-600 text-white font-bold text-sm shadow-md shadow-orange-500/30 transition active:scale-95 flex items-center justify-center gap-2">
          <i class="fa-solid fa-play"></i> 룰렛 돌리기
        </button>
        <button id="btnRouletteDetail" onclick="viewRouletteWinner()" class="hidden px-4 py-3 rounded-xl bg-slate-800 hover:bg-slate-900 text-white font-bold text-sm transition">
          상세보기
        </button>
      </div>
    </div>
  </div>

  <!-- Kakao Local API Settings & Address Geocoding Tester Modal -->
  <div id="kakaoModal" class="fixed inset-0 z-50 hidden flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-xs">
    <div class="bg-white rounded-3xl max-w-lg w-full p-6 shadow-2xl relative max-h-[90vh] flex flex-col">
      <button onclick="closeModal('kakaoModal')" class="absolute top-4 right-4 text-slate-400 hover:text-slate-600">
        <i class="fa-solid fa-xmark text-lg"></i>
      </button>

      <div class="flex items-center gap-2.5 mb-2">
        <div class="w-9 h-9 rounded-xl bg-[#FEE500] text-[#391B1B] flex items-center justify-center font-black text-base shadow-sm">
          <i class="fa-solid fa-location-crosshairs"></i>
        </div>
        <div>
          <h3 class="text-base sm:text-lg font-extrabold text-slate-900">카카오 로컬 API 연동</h3>
          <p class="text-xs text-slate-500">주소로 좌표 변환 (<code class="text-orange-600 bg-orange-50 px-1 py-0.5 rounded">/v2/local/search/address.json</code>)</p>
        </div>
      </div>

      <div class="overflow-y-auto custom-scrollbar flex-1 space-y-4 text-xs pr-1 mt-2">
        <!-- API Key Input -->
        <div class="bg-slate-50 p-3.5 rounded-2xl border border-slate-200">
          <label class="block font-bold text-slate-800 mb-1">카카오 REST API 키 등록</label>
          <div class="flex gap-2">
            <input type="password" id="kakaoApiKeyInput" placeholder="카카오 디벨로퍼스 REST API 키 입력" 
                   class="flex-1 px-3 py-2 bg-white border border-slate-300 rounded-xl text-xs font-mono text-slate-800 focus:outline-none focus:ring-2 focus:ring-yellow-400">
            <button onclick="saveKakaoApiKey()" class="px-3.5 py-2 bg-[#FEE500] hover:bg-[#edd400] text-[#391B1B] font-bold rounded-xl transition shadow-xs">
              저장
            </button>
            <button onclick="clearKakaoApiKey()" class="px-2.5 py-2 bg-slate-200 hover:bg-slate-300 text-slate-700 font-medium rounded-xl transition" title="키 삭제">
              삭제
            </button>
          </div>
          <p class="text-[11px] text-slate-500 mt-2 leading-relaxed">
            * <a href="https://developers.kakao.com/console" target="_blank" class="text-blue-600 underline font-semibold">카카오 디벨로퍼스</a> [내 애플리케이션] &gt; [앱 키]의 <strong>REST API 키</strong>를 등록하시면 대한민국 모든 주소의 정확한 위/경도 변환과 실제 카카오 등록 음식점을 호출합니다.
            <br>* 키가 없어도 공개 지오코딩 및 AI 맛집 발굴 기능으로 자동 대체 동작합니다.
          </p>
        </div>

        <!-- Address Geocode Live Test Box -->
        <div class="bg-amber-50/60 p-3.5 rounded-2xl border border-amber-200">
          <label class="block font-bold text-amber-900 mb-1.5 flex items-center justify-between">
            <span>주소 좌표 변환 실시간 테스트</span>
            <span class="text-[10px] text-amber-700 font-normal">GET /v2/local/search/address.json</span>
          </label>
          <div class="flex gap-2 mb-2">
            <input type="text" id="testAddressInput" value="전북 익산시 부송동 100" 
                   placeholder="테스트할 주소 입력 (예: 전북 익산시 부송동 100)" 
                   onkeydown="if(event.key === 'Enter') testKakaoAddressGeocode()"
                   class="flex-1 px-3 py-2 bg-white border border-amber-300 rounded-xl text-xs font-medium text-slate-800 focus:outline-none focus:ring-2 focus:ring-amber-400">
            <button onclick="testKakaoAddressGeocode()" id="btnTestGeocode" class="px-3.5 py-2 bg-amber-500 hover:bg-amber-600 text-white font-bold rounded-xl transition shadow-xs flex items-center gap-1">
              <i class="fa-solid fa-magnifying-glass text-[10px]"></i> 변환
            </button>
          </div>

          <!-- Result Preview Window -->
          <div id="testResultBox" class="p-3 bg-slate-900 text-slate-100 rounded-xl font-mono text-[11px] max-h-48 overflow-y-auto custom-scrollbar">
            <div class="text-slate-400">// 테스트할 주소를 입력하고 [변환] 버튼을 눌러보세요.</div>
            <div class="text-slate-400">// 응답: documents[0].x(경도), documents[0].y(위도)</div>
          </div>

          <div id="testCoordActions" class="mt-2.5 hidden flex items-center justify-between">
            <span id="testParsedCoords" class="font-bold text-slate-800 text-xs"></span>
            <button onclick="applyTestCoordToMap()" class="px-3 py-1.5 bg-orange-600 hover:bg-orange-700 text-white rounded-lg font-bold text-xs shadow-xs transition">
              이 좌표로 지도 이동 & 맛집 탐색
            </button>
          </div>
        </div>
      </div>

      <div class="mt-4 pt-3 border-t border-slate-200 flex justify-end">
        <button onclick="closeModal('kakaoModal')" class="px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 font-bold rounded-xl text-xs transition">
          닫기
        </button>
      </div>
    </div>
  </div>

  <!-- Toast Notification -->
  <div id="toast" class="fixed bottom-6 left-1/2 -translate-x-1/2 z-50 bg-slate-900/90 backdrop-blur-md text-white px-4 py-2.5 rounded-full text-xs font-medium shadow-xl opacity-0 pointer-events-none transition-all duration-300 flex items-center gap-2">
    <i id="toastIcon" class="fa-solid fa-check text-emerald-400"></i>
    <span id="toastMsg">알림 메시지</span>
  </div>

  <script>
    // Categories with iconic emojis
    const CATEGORIES = [
      { id: 'all', name: '전체', icon: '🍽️' },
      { id: 'korean', name: '한식', icon: '🍚' },
      { id: 'meat', name: '고기·구이', icon: '🥩' },
      { id: 'sashimi', name: '회·해산물', icon: '🐟' },
      { id: 'japanese', name: '일식·초밥', icon: '🍣' },
      { id: 'chinese', name: '중식', icon: '🥟' },
      { id: 'western', name: '양식·파스타', icon: '🍝' },
      { id: 'cafe', name: '카페·디저트', icon: '☕' },
      { id: 'snack', name: '분식·야식', icon: '🍢' }
    ];

    // Coordinates mapping for quick region centers
    const REGION_COORDS = {
      all: { lat: 37.5450, lng: 126.9950, zoom: 11, name: '전국 인기 맛집' },
      seongsu: { lat: 37.5445, lng: 127.0560, zoom: 15, name: '서울 성동구 (성수·서울숲)' },
      euljiro: { lat: 37.5662, lng: 126.9920, zoom: 15, name: '서울 중구 (을지로·종로)' },
      bundang: { lat: 37.3948, lng: 127.1119, zoom: 15, name: '성남 분당구 (판교·서현)' },
      gangnam: { lat: 37.4979, lng: 127.0276, zoom: 15, name: '서울 강남구 (강남·신사)' },
      hongdae: { lat: 37.5563, lng: 126.9236, zoom: 15, name: '서울 마포구 (홍대·연남)' },
      busan: { lat: 35.1587, lng: 129.1603, zoom: 14, name: '부산 해운대·광안리' },
      jeju: { lat: 33.4996, lng: 126.5312, zoom: 12, name: '제주 제주시·애월' }
    };

    // Realistic Restaurant Database with dual Naver & Kakao ratings (Expanded to 30+ top locations across major food hubs)
    const RESTAURANTS = [
      // === 성수동 / 서울숲 ===
      {
        id: 101,
        region: 'seongsu',
        name: '소문난성수감자탕',
        category: 'korean',
        categoryName: '한식',
        summary: '백종원의 3대천왕 및 성수동을 대표하는 40년 전통의 진하고 맑은 감자탕',
        lat: 37.5432,
        lng: 127.0573,
        distance: '150m',
        distMeters: 150,
        address: '서울 성동구 연무장길 45',
        phone: '02-465-6580',
        hours: '24시간 영업 (연중무휴)',
        naverRating: 4.63,
        naverReviews: 9540,
        kakaoRating: 4.2,
        kakaoReviews: 3120,
        parking: true,
        tags: ['24시간', '백종원맛집', '줄서는식당', '국물맛집'],
        image: 'https://images.unsplash.com/photo-1547928576-a4a33237cbc3?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '감자탕 (소)', price: '29,000원' },
          { name: '감자국 식사 (1인)', price: '10,000원' },
          { name: '수제비 사리 추가', price: '2,000원' }
        ]
      },
      {
        id: 102,
        region: 'seongsu',
        name: '난포 성수',
        category: 'korean',
        categoryName: '한식',
        summary: '강된장 쌈밥과 제철 회 묵은지말이로 성수동 오픈런을 부르는 감성 한식 다이닝',
        lat: 37.5478,
        lng: 127.0435,
        distance: '320m',
        distMeters: 320,
        address: '서울 성동구 서울숲4길 18-8 지하1층',
        phone: '02-468-1540',
        hours: '11:00 - 21:30 (브레이크타임 15:50~17:00)',
        naverRating: 4.74,
        naverReviews: 6810,
        kakaoRating: 4.3,
        kakaoReviews: 1420,
        parking: false,
        tags: ['서울숲데이트', '웨이팅핫플', '정갈한한상', '감성인테리어'],
        image: 'https://images.unsplash.com/photo-1553163147-622ab57be1c7?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '강된장 쌈밥', price: '12,000원' },
          { name: '제철회 묵은지말이', price: '13,000원' },
          { name: '새우치즈감자전', price: '19,000원' }
        ]
      },
      {
        id: 103,
        region: 'seongsu',
        name: '로우키 성수 (lowkey)',
        category: 'cafe',
        categoryName: '카페·디저트',
        summary: '커피 애호가들이 성지순례하는 고품격 스페셜티 드립커피와 아늑한 원목 공간',
        lat: 37.5451,
        lng: 127.0543,
        distance: '210m',
        distMeters: 210,
        address: '서울 성동구 연무장3길 6',
        phone: '02-466-2009',
        hours: '10:00 - 20:00',
        naverRating: 4.81,
        naverReviews: 3250,
        kakaoRating: 4.6,
        kakaoReviews: 890,
        parking: false,
        tags: ['드립커피맛집', '원두선택', '조용한분위기', '성수카페'],
        image: 'https://images.unsplash.com/photo-1495474472287-4d71bcdd2085?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '핸드드립 싱글오리진', price: '7,000원' },
          { name: '클래식 플랫화이트', price: '5,500원' },
          { name: '레몬 파운드 케이크', price: '5,000원' }
        ]
      },
      {
        id: 104,
        region: 'seongsu',
        name: '꿉당 성수점',
        category: 'meat',
        categoryName: '고기·구이',
        summary: '미쉐린 빕 구르망에 빛나는 육즙 폭발 KOKUMI 목살과 찰진 코쿠미 쌀밥의 완벽 조화',
        lat: 37.5442,
        lng: 127.0565,
        distance: '260m',
        distMeters: 260,
        address: '서울 성동구 성수이로20길 10',
        phone: '02-499-6592',
        hours: '15:30 - 23:00',
        naverRating: 4.79,
        naverReviews: 5410,
        kakaoRating: 4.5,
        kakaoReviews: 1650,
        parking: false,
        tags: ['미쉐린가이드', '목살맛집', '인생고깃집', '웨이팅필수'],
        image: 'https://images.unsplash.com/photo-1544025162-d76694265947?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: 'KOKUMI 목살 (180g)', price: '19,000원' },
          { name: '코쿠미 쌀밥', price: '3,000원' },
          { name: '생삼겹살 (180g)', price: '19,000원' }
        ]
      },

      // === 을지로 / 종로 ===
      {
        id: 201,
        region: 'euljiro',
        name: '을지면옥',
        category: 'korean',
        categoryName: '한식',
        summary: '고춧가루를 솔솔 뿌려낸 맑고 그윽한 육수의 평양냉면과 촉촉한 제육의 명가',
        lat: 37.5684,
        lng: 126.9942,
        distance: '180m',
        distMeters: 180,
        address: '서울 중구 을지로 101',
        phone: '02-2266-7052',
        hours: '11:30 - 21:00 (일요일 휴무)',
        naverRating: 4.57,
        naverReviews: 6120,
        kakaoRating: 4.1,
        kakaoReviews: 2480,
        parking: false,
        tags: ['평양냉면성지', '노포맛집', '수육제육', '역사있는곳'],
        image: 'https://images.unsplash.com/photo-1594998893017-36147cbcae05?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '평양냉면 (물)', price: '15,000원' },
          { name: '소고기 편육', price: '30,000원' },
          { name: '돼지고기 제육', price: '28,000원' }
        ]
      },
      {
        id: 202,
        region: 'euljiro',
        name: '산수갑산',
        category: 'snack',
        categoryName: '분식·야식',
        summary: '수요미식회에서 극찬한 대창 순대와 담백한 머릿고기 순대모둠 모듬 한접시',
        lat: 37.5668,
        lng: 126.9961,
        distance: '290m',
        distMeters: 290,
        address: '서울 중구 을지로20길 24',
        phone: '02-2275-6654',
        hours: '11:30 - 22:00 (일요일 휴무)',
        naverRating: 4.62,
        naverReviews: 4890,
        kakaoRating: 4.3,
        kakaoReviews: 1830,
        parking: false,
        tags: ['대창순대', '노포힙지로', '소주안주', '수요미식회'],
        image: 'https://images.unsplash.com/photo-1541832676-9b763b0239ab?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '순대모둠 (2인)', price: '26,000원' },
          { name: '순대국밥', price: '9,000원' },
          { name: '술국', price: '12,000원' }
        ]
      },
      {
        id: 203,
        region: 'euljiro',
        name: '을지오뎅 도루묵',
        category: 'snack',
        categoryName: '분식·야식',
        summary: '뜨끈한 오뎅바에 둘러앉아 톡톡 터지는 알 도루묵구이와 사케를 즐기는 을지로 노포',
        lat: 37.5661,
        lng: 126.9928,
        distance: '140m',
        distMeters: 140,
        address: '서울 중구 수표로 54',
        phone: '02-2274-5092',
        hours: '17:00 - 24:00',
        naverRating: 4.51,
        naverReviews: 1840,
        kakaoRating: 4.2,
        kakaoReviews: 610,
        parking: false,
        tags: ['오뎅바', '도루묵구이', '비오는날추천', '감성술집'],
        image: 'https://images.unsplash.com/photo-1563245372-f21724e3856d?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '알 도루묵 구이', price: '17,000원' },
          { name: '수제 오뎅 (1개)', price: '1,500원' },
          { name: '히레사케', price: '6,000원' }
        ]
      },

      // === 성남시 분당구 (판교/서현) ===
      {
        id: 1,
        region: 'bundang',
        name: '우미학 분당판교점',
        category: 'meat',
        categoryName: '고기·구이',
        summary: '최고급 한우 숙성 등심과 깍두기 볶음밥이 일품인 프리미엄 소고기 전문점',
        lat: 37.3954,
        lng: 127.1128,
        distance: '230m',
        distMeters: 230,
        address: '경기 성남시 분당구 판교역로 146번길 20',
        phone: '031-8017-8892',
        hours: '11:30 - 22:00 (매일)',
        naverRating: 4.78,
        naverReviews: 1840,
        kakaoRating: 4.4,
        kakaoReviews: 420,
        parking: true,
        tags: ['주차가능', '콜키지프리', '룸완비', '회식추천'],
        image: 'https://images.unsplash.com/photo-1544025162-d76694265947?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '숙성 한우 채끝등심 (150g)', price: '48,000원' },
          { name: '한우 안심 (150g)', price: '52,000원' },
          { name: '차돌 깍두기 볶음밥', price: '12,000원' }
        ]
      },
      {
        id: 2,
        region: 'bundang',
        name: '어물전 청 판교',
        category: 'sashimi',
        categoryName: '회·해산물',
        summary: '당일 산지 직송 계절 제철 생선회와 감태김밥이 유명한 모던 해물 오마카세',
        lat: 37.3971,
        lng: 127.1105,
        distance: '350m',
        distMeters: 350,
        address: '경기 성남시 분당구 판교역로 192번길 12',
        phone: '031-702-7721',
        hours: '17:00 - 23:00 (일요일 휴무)',
        naverRating: 4.82,
        naverReviews: 920,
        kakaoRating: 4.5,
        kakaoReviews: 290,
        parking: true,
        tags: ['주차가능', '예약필수', '분위기맛집', '모임장소'],
        image: 'https://images.unsplash.com/photo-1534422298391-e4f8c172dddb?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '맡김차림 코스 (1인)', price: '68,000원' },
          { name: '자연산 모둠 사시미', price: '55,000원' },
          { name: '단새우 우니 감태쌈', price: '38,000원' }
        ]
      },
      {
        id: 3,
        region: 'bundang',
        name: '진진짜라 손짜장',
        category: 'chinese',
        categoryName: '중식',
        summary: '불향 가득 볶아낸 수제 간짜장과 바삭 쫀득한 찹쌀 탕수육의 성지',
        lat: 37.3932,
        lng: 127.1142,
        distance: '480m',
        distMeters: 480,
        address: '경기 성남시 분당구 대왕판교로606번길 58',
        phone: '031-708-3341',
        hours: '11:00 - 21:00 (화요일 휴무)',
        naverRating: 4.51,
        naverReviews: 1210,
        kakaoRating: 4.1,
        kakaoReviews: 310,
        parking: false,
        tags: ['혼밥환영', '현지인맛집', '가성비', '배달가능'],
        image: 'https://images.unsplash.com/photo-1525755662778-989d0524087e?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '불향 수제 간짜장', price: '9,000원' },
          { name: '해물 삼선 짬뽕', price: '11,000원' },
          { name: '찹쌀 생등심 탕수육(소)', price: '22,000원' }
        ]
      },
      {
        id: 4,
        region: 'bundang',
        name: '스시 스미레 판교',
        category: 'japanese',
        categoryName: '일식·초밥',
        summary: '숙성 사시미와 정갈한 스시 카운터 코스를 선보이는 하이엔드 일식당',
        lat: 37.3965,
        lng: 127.1089,
        distance: '520m',
        distMeters: 520,
        address: '경기 성남시 분당구 판교역로 235',
        phone: '031-622-7500',
        hours: '12:00 - 21:30 (월요일 휴무)',
        naverRating: 4.88,
        naverReviews: 640,
        kakaoRating: 4.6,
        kakaoReviews: 195,
        parking: true,
        tags: ['주차가능', '100%예약제', '기념일추천', '오마카세'],
        image: 'https://images.unsplash.com/photo-1579871494447-9811cf80d66c?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '런치 스시 코스', price: '60,000원' },
          { name: '디너 오마카세', price: '120,000원' },
          { name: '특상 후토마키', price: '28,000원' }
        ]
      },
      {
        id: 5,
        region: 'bundang',
        name: '기와집 솥밥과 제육',
        category: 'korean',
        categoryName: '한식',
        summary: '10가지 천연 나물 반찬과 유기농 솥밥, 숯불 직화 제육이 나오는 든든한 백반',
        lat: 37.3921,
        lng: 127.1098,
        distance: '310m',
        distMeters: 310,
        address: '경기 성남시 분당구 분당내곡로 117',
        phone: '031-8016-5544',
        hours: '11:00 - 21:30',
        naverRating: 4.64,
        naverReviews: 2450,
        kakaoRating: 4.2,
        kakaoReviews: 540,
        parking: true,
        tags: ['주차가능', '혼밥환영', '가족식사', '푸짐한국물'],
        image: 'https://images.unsplash.com/photo-1498654896293-37aacf113fd9?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '직화 제육 솥밥 정식', price: '14,000원' },
          { name: '보리굴비 솥밥 반상', price: '23,000원' },
          { name: '차돌 된장찌개', price: '10,000원' }
        ]
      },
      {
        id: 6,
        region: 'bundang',
        name: '오스테리아 판교',
        category: 'western',
        categoryName: '양식·파스타',
        summary: '자가제면 생면 파스타와 화덕에 바로 구운 나폴리식 마르게리따 피자',
        lat: 37.3982,
        lng: 127.1135,
        distance: '610m',
        distMeters: 610,
        address: '경기 성남시 분당구 판교동 592-3',
        phone: '031-705-1880',
        hours: '11:30 - 22:00',
        naverRating: 4.70,
        naverReviews: 870,
        kakaoRating: 4.3,
        kakaoReviews: 210,
        parking: true,
        tags: ['주차가능', '와인페어링', '데이트코스', '테라스석'],
        image: 'https://images.unsplash.com/photo-1551183053-bf91a1d81141?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '트러플 화이트 라구 생면 파스타', price: '24,000원' },
          { name: '참나무 화덕 마르게리따 피자', price: '21,000원' },
          { name: '수비드 채끝 스테이크', price: '45,000원' }
        ]
      },

      // === 서울 강남구 (강남/신사/청담) ===
      {
        id: 8,
        region: 'gangnam',
        name: '새벽집 청담본점',
        category: 'meat',
        categoryName: '고기·구이',
        summary: '진한 선지해장국과 꽃등심, 육회비빔밥이 유명한 24시간 청담/강남 대표 명소',
        lat: 37.5245,
        lng: 127.0505,
        distance: '450m',
        distMeters: 450,
        address: '서울 강남구 도산대로101길 6',
        phone: '02-546-5739',
        hours: '24시간 영업 (연중무휴)',
        naverRating: 4.58,
        naverReviews: 4320,
        kakaoRating: 4.2,
        kakaoReviews: 1200,
        parking: true,
        tags: ['발렛파킹', '24시간', '육회비빔밥맛집', '연예인맛집'],
        image: 'https://images.unsplash.com/photo-1558030006-450675393462?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '육회비빔밥 (선지국 포함)', price: '15,000원' },
          { name: '한우 꽃등심 (150g)', price: '68,000원' },
          { name: '따로국밥', price: '12,000원' }
        ]
      },
      {
        id: 9,
        region: 'gangnam',
        name: '보물섬 논현',
        category: 'sashimi',
        categoryName: '회·해산물',
        summary: '겨울철 기름진 자연산 대방어와 돌돔, 해물 모둠이 끝없이 펼쳐지는 해산물 성지',
        lat: 37.5098,
        lng: 127.0298,
        distance: '380m',
        distMeters: 380,
        address: '서울 강남구 강남대로118길 38',
        phone: '02-540-3563',
        hours: '16:00 - 04:00',
        naverRating: 4.74,
        naverReviews: 1980,
        kakaoRating: 4.3,
        kakaoReviews: 610,
        parking: false,
        tags: ['대방어성지', '심야영업', '웨이팅맛집', '신선한해산물'],
        image: 'https://images.unsplash.com/photo-1534604973900-c43ab4c2e0ab?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '자연산 대방어 특수부위(대)', price: '120,000원' },
          { name: '자연산 모둠 해물포차', price: '70,000원' },
          { name: '해물 통라면', price: '13,000원' }
        ]
      },
      {
        id: 10,
        region: 'gangnam',
        name: '대가방 본점',
        category: 'chinese',
        categoryName: '중식',
        summary: '수요미식회가 극찬한 겉바속촉 탕수육과 대가탕면으로 소문난 정통 중화요리점',
        lat: 37.5182,
        lng: 127.0371,
        distance: '500m',
        distMeters: 500,
        address: '서울 강남구 선릉로145길 13',
        phone: '02-544-6336',
        hours: '11:30 - 21:30 (일요일 휴무)',
        naverRating: 4.62,
        naverReviews: 2110,
        kakaoRating: 4.1,
        kakaoReviews: 780,
        parking: true,
        tags: ['발렛파킹', '미슐랭플레이트', '탕수육맛집', '룸구비'],
        image: 'https://images.unsplash.com/photo-1563245372-f21724e3856d?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '대가방 탕수육', price: '30,000원' },
          { name: '대가탕면 (굴백짬뽕)', price: '13,000원' },
          { name: '해물 누룽지탕', price: '45,000원' }
        ]
      },

      // === 서울 마포구 (홍대/연남/합정) ===
      {
        id: 11,
        region: 'hongdae',
        name: '바다회사랑 1호점',
        category: 'sashimi',
        categoryName: '회·해산물',
        summary: '두툼하게 썰어낸 겨울 대방어와 생연어 반반의 독보적인 전국구 회 맛집',
        lat: 37.5582,
        lng: 126.9215,
        distance: '290m',
        distMeters: 290,
        address: '서울 마포구 동교로27길 60',
        phone: '02-338-0872',
        hours: '14:30 - 24:00',
        naverRating: 4.79,
        naverReviews: 5400,
        kakaoRating: 4.4,
        kakaoReviews: 1890,
        parking: false,
        tags: ['방어성지', '줄서는식당', '포장가능', '가성비최고'],
        image: 'https://images.unsplash.com/photo-1519708227418-c8fd9a32b7a2?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '방어+연어 반반 세트(중)', price: '75,000원' },
          { name: '생우럭 매운탕', price: '15,000원' },
          { name: '날치알 밥', price: '3,000원' }
        ]
      },
      {
        id: 12,
        region: 'hongdae',
        name: '하카타분코',
        category: 'japanese',
        categoryName: '일식·초밥',
        summary: '진하고 묵직한 돈코츠 육수와 자가제면 얇은 면이 조화로운 홍대 라멘의 전설',
        lat: 37.5492,
        lng: 126.9221,
        distance: '410m',
        distMeters: 410,
        address: '서울 마포구 독막로19길 43',
        phone: '02-338-5536',
        hours: '11:30 - 02:00',
        naverRating: 4.60,
        naverReviews: 2980,
        kakaoRating: 4.2,
        kakaoReviews: 950,
        parking: false,
        tags: ['혼밥성지', '심야라멘', '인라멘', '원조맛집'],
        image: 'https://images.unsplash.com/photo-1569718212165-3a8278d5f624?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '인라멘 (진한 육수)', price: '10,000원' },
          { name: '청라멘 (깔끔한 육수)', price: '10,000원' },
          { name: '차슈덮밥', price: '4,000원' }
        ]
      },
      {
        id: 13,
        region: 'hongdae',
        name: '소이연남',
        category: 'western',
        categoryName: '양식·파스타',
        summary: '진한 태국식 소고기 국수와 바삭한 뽀삐아 만두가 환상적인 연남동 핫플',
        lat: 37.5615,
        lng: 126.9248,
        distance: '330m',
        distMeters: 330,
        address: '서울 마포구 동교로 267',
        phone: '02-323-5130',
        hours: '11:00 - 21:20',
        naverRating: 4.68,
        naverReviews: 4910,
        kakaoRating: 4.3,
        kakaoReviews: 1420,
        parking: false,
        tags: ['소고기쌀국수', '태국음식', '연남동맛집', '블루리본'],
        image: 'https://images.unsplash.com/photo-1559847844-5315695dadae?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '소고기 국수', price: '11,000원' },
          { name: '소이 뽀삐아 (춘권)', price: '14,000원' },
          { name: '쏨땀', price: '13,000원' }
        ]
      },

      // === 부산 해운대 / 광안리 ===
      {
        id: 301,
        region: 'busan',
        name: '해운대암소갈비집',
        category: 'meat',
        categoryName: '고기·구이',
        summary: '특유의 무쇠 불판에 굽는 생갈비와 마지막에 끓여먹는 감자사리가 독보적인 부산의 자존심',
        lat: 35.1631,
        lng: 129.1637,
        distance: '400m',
        distMeters: 400,
        address: '부산 해운대구 중동2로10번길 325',
        phone: '051-746-0033',
        hours: '11:30 - 22:00',
        naverRating: 4.61,
        naverReviews: 8150,
        kakaoRating: 4.3,
        kakaoReviews: 2980,
        parking: true,
        tags: ['부산필수코스', '감자사리', '생갈비맛집', '주차완비'],
        image: 'https://images.unsplash.com/photo-1544025162-d76694265947?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '생갈비 (180g)', price: '58,000원' },
          { name: '양념갈비 (180g)', price: '52,000원' },
          { name: '감자사리 추가', price: '3,000원' }
        ]
      },
      {
        id: 302,
        region: 'busan',
        name: '금수복국 해운대본점',
        category: 'korean',
        categoryName: '한식',
        summary: '뚝배기에 팔팔 끓여내는 맑은 은복 지리와 껍질무침으로 전국 해장의 기준이 된 집',
        lat: 35.1618,
        lng: 129.1625,
        distance: '310m',
        distMeters: 310,
        address: '부산 해운대구 중동1로43번길 23',
        phone: '051-742-3600',
        hours: '24시간 영업 (연중무휴)',
        naverRating: 4.65,
        naverReviews: 12400,
        kakaoRating: 4.2,
        kakaoReviews: 3820,
        parking: true,
        tags: ['24시간', '해장성지', '복지리', '부산대표맛집'],
        image: 'https://images.unsplash.com/photo-1547928576-a4a33237cbc3?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '은복 지리/탕', price: '14,000원' },
          { name: '밀복 지리/탕', price: '20,000원' },
          { name: '복껍질무침', price: '15,000원' }
        ]
      },
      {
        id: 303,
        region: 'busan',
        name: '민락어민활어직판장 칠성상회',
        category: 'sashimi',
        categoryName: '회·해산물',
        summary: '광안대교 오션뷰 야경과 함께 바로 뜬 자연산 활어회와 산낙지를 가성비 넘치게 즐기는 명소',
        lat: 35.1554,
        lng: 129.1235,
        distance: '620m',
        distMeters: 620,
        address: '부산 수영구 광안해변로312번길 60',
        phone: '051-753-8822',
        hours: '10:00 - 23:00',
        naverRating: 4.75,
        naverReviews: 3120,
        kakaoRating: 4.4,
        kakaoReviews: 920,
        parking: true,
        tags: ['오션뷰', '가성비회', '초장집연계', '광안리야경'],
        image: 'https://images.unsplash.com/photo-1534422298391-e4f8c172dddb?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '제철 모둠활어회 (2인)', price: '40,000원' },
          { name: '산낙지 탕탕이', price: '20,000원' },
          { name: '얼큰 매운탕', price: '10,000원' }
        ]
      },

      // === 제주도 (제주시/애월) ===
      {
        id: 401,
        region: 'jeju',
        name: '숙성도 노형본점',
        category: 'meat',
        categoryName: '고기·구이',
        summary: '960시간 교차 숙성한 제주 흑돼지 뼈등심과 나비살, 명란젓 조합의 극강의 풍미',
        lat: 33.4862,
        lng: 126.4883,
        distance: '450m',
        distMeters: 450,
        address: '제주 제주시 원노형로 41',
        phone: '064-711-5212',
        hours: '11:30 - 21:30',
        naverRating: 4.86,
        naverReviews: 15400,
        kakaoRating: 4.6,
        kakaoReviews: 4950,
        parking: false,
        tags: ['제주흑돼지', '캐치테이블필수', '인생목살', '숙성육전문'],
        image: 'https://images.unsplash.com/photo-1544025162-d76694265947?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '720 숙성 뼈목살 (360g)', price: '38,000원' },
          { name: '960 숙성 뼈등심 (350g)', price: '38,000원' },
          { name: '갈치속젓 볶음밥', price: '7,000원' }
        ]
      },
      {
        id: 402,
        region: 'jeju',
        name: '자매국수',
        category: 'korean',
        categoryName: '한식',
        summary: '노란 치자면에 두툼한 제주 흑돼지 수육을 듬뿍 얹어낸 깊고 진한 고기국수',
        lat: 33.5147,
        lng: 126.5298,
        distance: '380m',
        distMeters: 380,
        address: '제주 제주시 탑동로11길 6',
        phone: '064-746-2222',
        hours: '09:00 - 18:00 (수요일 휴무)',
        naverRating: 4.70,
        naverReviews: 18900,
        kakaoRating: 4.3,
        kakaoReviews: 4310,
        parking: true,
        tags: ['제주고기국수', '공항근처', '아침식사', '주차편리'],
        image: 'https://images.unsplash.com/photo-1569718212165-3a8278d5f624?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '제주 고기국수', price: '10,000원' },
          { name: '비빔국수', price: '10,000원' },
          { name: '돔베고기 (소)', price: '19,000원' }
        ]
      },
      {
        id: 403,
        region: 'jeju',
        name: '애월해녀의집',
        category: 'sashimi',
        categoryName: '회·해산물',
        summary: '해녀들이 직접 물질해 올린 싱싱한 뿔소라회와 전복, 성게 가득한 전복해물라면',
        lat: 33.4682,
        lng: 126.3195,
        distance: '210m',
        distMeters: 210,
        address: '제주 제주시 애월읍 애월로11길 22',
        phone: '064-799-7008',
        hours: '09:30 - 19:30',
        naverRating: 4.67,
        naverReviews: 4210,
        kakaoRating: 4.4,
        kakaoReviews: 1120,
        parking: true,
        tags: ['오션뷰해녀집', '뿔소라회', '전복죽', '바다전망'],
        image: 'https://images.unsplash.com/photo-1534422298391-e4f8c172dddb?auto=format&fit=crop&w=600&q=80',
        menus: [
          { name: '자연산 뿔소라회', price: '25,000원' },
          { name: '전복 해물라면', price: '12,000원' },
          { name: '진한 전복죽', price: '15,000원' }
        ]
      }
    ];

    let currentRegion = 'all';
    let currentRegionLabel = '전국 인기 맛집';
    let currentCategory = 'all';
    let currentSearch = '';
    let currentSort = 'recommend';
    let filterKakao4Active = false;
    let filterNaver45Active = false;
    let filterParkingActive = false;
    let showFavoritesOnly = false;
    let viewMode = 'grid'; // 'grid' or 'split'
    let currentSelectedId = null;

    // Favorites stored in LocalStorage
    let favorites = JSON.parse(localStorage.getItem('tasteMap_favs') || '[]');

    // Leaflet map instances and markers
    let leafletMap = null;
    let mapMarkers = [];
    let isAiFetching = false;

    // Kakao Local API Key management
    let kakaoApiKey = localStorage.getItem('kakao_rest_key') || '';
    let lastGeocodedCoord = null;

    function openKakaoModal() {
      document.getElementById('kakaoApiKeyInput').value = kakaoApiKey;
      updateKakaoKeyStatus();
      document.getElementById('kakaoModal').classList.remove('hidden');
    }

    function saveKakaoApiKey() {
      const key = document.getElementById('kakaoApiKeyInput').value.trim();
      if (!key) {
        showToast('카카오 REST API 키를 입력해주세요.', 'warn');
        return;
      }
      kakaoApiKey = key;
      localStorage.setItem('kakao_rest_key', key);
      updateKakaoKeyStatus();
      showToast('카카오 REST API 키가 안전하게 저장되었습니다! 🔑');
    }

    function clearKakaoApiKey() {
      kakaoApiKey = '';
      localStorage.removeItem('kakao_rest_key');
      document.getElementById('kakaoApiKeyInput').value = '';
      updateKakaoKeyStatus();
      showToast('카카오 API 키가 삭제되었습니다. (공개 지오코더로 전환)');
    }

    function updateKakaoKeyStatus() {
      const dot = document.getElementById('kakaoStatusDot');
      if (dot) {
        if (kakaoApiKey) {
          dot.className = 'w-2 h-2 rounded-full bg-emerald-500 ring-2 ring-emerald-300';
          dot.title = '카카오 API 연동 활성화';
        } else {
          dot.className = 'w-2 h-2 rounded-full bg-slate-400';
          dot.title = '카카오 API 미등록 (기본 지오코더 사용)';
        }
      }
    }

    // Call Kakao /v2/local/search/address.json specification
    async function requestKakaoAddressGeocode(addressQuery) {
      if (!kakaoApiKey) return null;
      const url = `https://dapi.kakao.com/v2/local/search/address.json?query=${encodeURIComponent(addressQuery)}`;
      const res = await fetch(url, {
        headers: {
          'Authorization': `KakaoAK ${kakaoApiKey}`
        }
      });
      if (!res.ok) {
        const errJson = await res.json().catch(() => ({}));
        throw new Error(errJson.message || `Kakao HTTP ${res.status}`);
      }
      const data = await res.json();
      return data;
    }

    // Interactive tester inside modal for /v2/local/search/address.json
    async function testKakaoAddressGeocode() {
      const address = document.getElementById('testAddressInput').value.trim();
      const resultBox = document.getElementById('testResultBox');
      const actionBox = document.getElementById('testCoordActions');
      const parsedText = document.getElementById('testParsedCoords');

      if (!address) {
        showToast('테스트할 주소를 입력해주세요.', 'warn');
        return;
      }

      resultBox.innerHTML = '<span class="text-yellow-400"><i class="fa-solid fa-spinner fa-spin mr-1"></i> 카카오 주소 변환 API 호출 중...</span>';
      actionBox.classList.add('hidden');

      try {
        if (!kakaoApiKey) {
          resultBox.innerHTML = `<span class="text-rose-400 font-bold">⚠️ 카카오 REST API 키가 등록되지 않았습니다.</span>\n상단에서 카카오 REST API 키를 먼저 입력하고 [저장]해주세요.\n(카카오 디벨로퍼스 -> 내 애플리케이션 -> 앱 키 -> REST API 키)`;
          return;
        }

        const data = await requestKakaoAddressGeocode(address);
        resultBox.innerText = JSON.stringify(data, null, 2);

        if (data.documents && data.documents.length > 0) {
          const doc = data.documents[0];
          const xLng = parseFloat(doc.x);
          const yLat = parseFloat(doc.y);
          const roadAddr = doc.road_address ? doc.road_address.address_name : doc.address_name;

          lastGeocodedCoord = { lat: yLat, lng: xLng, address: roadAddr, query: address };
          parsedText.innerText = `위도(y): ${yLat.toFixed(6)}, 경도(x): ${xLng.toFixed(6)}`;
          actionBox.classList.remove('hidden');
          showToast('주소 좌표 변환 성공! 📍');
        } else {
          resultBox.innerHTML += `\n\n<span class="text-amber-400">// 일치하는 주소 검색 결과가 없습니다. 도로명 또는 지번을 정확히 입력해주세요.</span>`;
        }
      } catch (err) {
        resultBox.innerHTML = `<span class="text-rose-400">요청 실패: ${err.message}</span>\n\n확인 사항:\n1. REST API 키가 올바른지 확인 (JavaScript 키 X, REST API 키 O)\n2. 카카오 디벨로퍼스 플랫폼 설정에서 웹 도메인 허용 여부`;
        console.error('Kakao Geocode Error:', err);
      }
    }

    function applyTestCoordToMap() {
      if (!lastGeocodedCoord) return;
      closeModal('kakaoModal');
      document.getElementById('regionSearchInput').value = lastGeocodedCoord.query;
      currentRegion = 'custom_' + Date.now();
      currentRegionLabel = lastGeocodedCoord.address || lastGeocodedCoord.query;

      if (leafletMap) {
        leafletMap.setView([lastGeocodedCoord.lat, lastGeocodedCoord.lng], 15);
      }

      fetchRestaurantsWithKakaoOrGemini(lastGeocodedCoord.query, lastGeocodedCoord.lat, lastGeocodedCoord.lng);
    }

    // Audio context for sound effects
    let audioCtx = null;
    function playBeep(freq = 600, duration = 0.08) {
      try {
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        if (audioCtx.state === 'suspended') audioCtx.resume();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = 'sine';
        osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
        gain.gain.setValueAtTime(0.08, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
        osc.stop(audioCtx.currentTime + duration);
      } catch (e) {
        // Audio fallback ignore
      }
    }

    function renderCategoryTabs() {
      const container = document.getElementById('categoryTabs');
      container.innerHTML = CATEGORIES.map(cat => {
        const isActive = cat.id === currentCategory;
        const activeClass = isActive 
          ? 'bg-orange-500 text-white shadow-md shadow-orange-500/25 border-orange-500 font-bold' 
          : 'bg-slate-100 hover:bg-slate-200 text-slate-700 border-slate-200 font-medium';

        return `
          <button onclick="setCategory('${cat.id}')" class="px-3.5 py-1.5 rounded-xl border text-xs sm:text-sm whitespace-nowrap transition-all flex items-center space-x-1.5 active:scale-95 ${activeClass}">
            <span class="text-base leading-none">${cat.icon}</span>
            <span>${cat.name}</span>
          </button>
        `;
      }).join('');
    }

    function setCategory(id) {
      currentCategory = id;
      renderCategoryTabs();
      applyFilters();
      playBeep(700, 0.05);
    }

    function applyFilters() {
      const searchVal = document.getElementById('searchInput').value.trim().toLowerCase();
      currentSort = document.getElementById('sortSelect').value;

      // Filter by Region
      let list = RESTAURANTS;
      if (currentRegion !== 'all') {
        list = list.filter(r => r.region === currentRegion || r.address.includes(currentRegionLabel.split(' ')[0]));
      }

      // Filter by Category
      if (currentCategory !== 'all') {
        list = list.filter(r => r.category === currentCategory);
      }

      // Filter by Text Search
      if (searchVal) {
        list = list.filter(r => 
          r.name.toLowerCase().includes(searchVal) ||
          r.categoryName.toLowerCase().includes(searchVal) ||
          r.summary.toLowerCase().includes(searchVal) ||
          r.address.toLowerCase().includes(searchVal) ||
          r.tags.some(t => t.toLowerCase().includes(searchVal)) ||
          r.menus.some(m => m.name.toLowerCase().includes(searchVal))
        );
      }

      // Filter by Kakao 4.0+
      if (filterKakao4Active) {
        list = list.filter(r => r.kakaoRating >= 4.0);
      }

      // Filter by Naver 4.5+
      if (filterNaver45Active) {
        list = list.filter(r => r.naverRating >= 4.5);
      }

      // Filter by Parking
      if (filterParkingActive) {
        list = list.filter(r => r.parking);
      }

      // Filter by Favorites Only
      if (showFavoritesOnly) {
        list = list.filter(r => favorites.includes(r.id));
      }

      // Sorting
      list.sort((a, b) => {
        if (currentSort === 'kakaoRating') return b.kakaoRating - a.kakaoRating;
        if (currentSort === 'naverRating') return b.naverRating - a.naverRating;
        if (currentSort === 'distance') return (a.distMeters || 500) - (b.distMeters || 500);
        if (currentSort === 'reviewCount') return (b.naverReviews + b.kakaoReviews) - (a.naverReviews + a.kakaoReviews);
        // Default 'recommend': weighted combination
        const scoreA = (a.naverRating * 0.45) + (a.kakaoRating * 0.55);
        const scoreB = (b.naverRating * 0.45) + (b.kakaoRating * 0.55);
        return scoreB - scoreA;
      });

      // Update badge
      const isFiltered = filterKakao4Active || filterNaver45Active || filterParkingActive || currentCategory !== 'all' || searchVal || showFavoritesOnly;
      const badge = document.getElementById('activeFilterBadge');
      if (isFiltered) badge.classList.remove('hidden');
      else badge.classList.add('hidden');

      // Update Region Label
      document.getElementById('currentRegionLabel').innerText = currentRegionLabel;

      renderRestaurantCards(list);
      updateMapMarkers(list);

      // Manage AI Search Prompt Banner Visibility
      const aiBanner = document.getElementById('aiSearchBanner');
      const regInput = document.getElementById('regionSearchInput').value.trim();
      if (regInput && currentRegion !== 'all') {
        aiBanner.classList.remove('hidden');
        document.getElementById('aiBannerText').innerText = `'${regInput}' 지역 맛집을 더 발견하고 싶으신가요?`;
      } else {
        aiBanner.classList.add('hidden');
      }
    }

    function renderRestaurantCards(items) {
      const grid = document.getElementById('restaurantGrid');
      const empty = document.getElementById('emptyState');
      const countEl = document.getElementById('restaurantCount');

      countEl.innerText = items.length;

      if (items.length === 0) {
        grid.innerHTML = '';
        empty.classList.remove('hidden');
        const queryTerm = document.getElementById('regionSearchInput').value.trim() || currentRegionLabel;
        document.getElementById('emptyStateTitle').innerText = `'${queryTerm}' 근처에 일치하는 맛집이 없습니다`;
        document.getElementById('emptyAiBtnText').innerText = `'${queryTerm}' 맛집 AI로 자동 발굴하기`;
        return;
      }
      empty.classList.add('hidden');

      grid.innerHTML = items.map(r => {
        const isFav = favorites.includes(r.id);
        const menuFirst = r.menus && r.menus[0] ? r.menus[0] : { name: '대표 메뉴', price: '변동' };
        return `
          <div class="bg-white rounded-2xl border border-slate-200/90 hover:border-orange-300 hover:shadow-xl transition-all duration-300 flex flex-col overflow-hidden group cursor-pointer" onclick="openDetailModal(${r.id})">
            
            <!-- Card Thumbnail with tags & fav button -->
            <div class="relative h-44 w-full bg-slate-100 overflow-hidden">
              <img src="${r.image}" alt="${r.name}" 
                   onerror="this.src='https://placehold.co/600x400/fff7ed/ea580c?text=${encodeURIComponent(r.name)}'" 
                   class="w-full h-full object-cover group-hover:scale-105 transition duration-500">
              <div class="absolute inset-0 bg-gradient-to-t from-black/60 via-transparent to-transparent"></div>

              <!-- Category Badge -->
              <span class="absolute top-3 left-3 bg-black/60 backdrop-blur-md text-white text-[11px] font-semibold px-2.5 py-1 rounded-full">
                ${r.categoryName}
              </span>

              <!-- Favorite Button -->
              <button onclick="event.stopPropagation(); toggleFavorite(${r.id})" 
                      class="absolute top-3 right-3 w-8 h-8 rounded-full bg-white/80 hover:bg-white backdrop-blur-sm flex items-center justify-center text-rose-500 shadow-sm transition active:scale-90">
                <i class="${isFav ? 'fa-solid' : 'fa-regular'} fa-heart text-sm"></i>
              </button>

              <!-- Distance / Region Pill -->
              <div class="absolute bottom-2.5 left-3 text-white text-xs font-semibold flex items-center gap-1 drop-shadow-sm">
                <i class="fa-solid fa-location-dot text-orange-400"></i>
                <span class="truncate max-w-[130px]">${r.distance || '주변'}</span>
                <span class="text-slate-300 text-[11px] font-normal">• ${r.address.split(' ').slice(0, 2).join(' ')}</span>
              </div>
            </div>

            <!-- Card Content Area -->
            <div class="p-4 flex-1 flex flex-col justify-between">
              <div>
                <!-- Title & Tags -->
                <div class="flex items-start justify-between gap-1">
                  <h3 class="font-bold text-base text-slate-900 group-hover:text-orange-600 transition">
                    ${r.name}
                  </h3>
                </div>

                <p class="text-xs text-slate-500 mt-1 line-clamp-2 leading-relaxed">
                  ${r.summary}
                </p>

                <!-- Dual Ratings Comparison Badges -->
                <div class="grid grid-cols-2 gap-2 mt-3 pt-3 border-t border-slate-100">
                  <!-- Naver Rating -->
                  <div class="flex items-center space-x-1.5 bg-emerald-50/70 border border-emerald-200/60 px-2.5 py-1.5 rounded-xl">
                    <span class="w-4 h-4 rounded bg-[#03C75A] text-white flex items-center justify-center font-bold text-[9px] shrink-0">N</span>
                    <div class="flex flex-col leading-none">
                      <div class="flex items-baseline gap-1">
                        <span class="text-xs font-black text-slate-800">${r.naverRating ? r.naverRating.toFixed(2) : '4.50'}</span>
                        <span class="text-[10px] text-slate-400">/ 5.0</span>
                      </div>
                      <span class="text-[9px] text-emerald-700 font-medium">리뷰 ${(r.naverReviews || 500).toLocaleString()}</span>
                    </div>
                  </div>

                  <!-- Kakao Rating -->
                  <div class="flex items-center space-x-1.5 bg-amber-50/80 border border-yellow-300/80 px-2.5 py-1.5 rounded-xl">
                    <span class="w-4 h-4 rounded bg-[#FEE500] text-[#391B1B] flex items-center justify-center font-black text-[9px] shrink-0">
                      <i class="fa-solid fa-comment"></i>
                    </span>
                    <div class="flex flex-col leading-none">
                      <div class="flex items-baseline gap-1">
                        <span class="text-xs font-black text-slate-800">${r.kakaoRating ? r.kakaoRating.toFixed(1) : '4.2'}</span>
                        <span class="text-[10px] text-slate-400">/ 5.0</span>
                      </div>
                      <span class="text-[9px] text-amber-800 font-medium">리뷰 ${(r.kakaoReviews || 200).toLocaleString()}</span>
                    </div>
                  </div>
                </div>

                <!-- Representative Menu -->
                <div class="mt-2.5 flex items-center justify-between text-xs text-slate-600 bg-slate-50 px-2.5 py-1.5 rounded-lg">
                  <span class="truncate font-medium text-slate-700">
                    <i class="fa-solid fa-utensils text-slate-400 mr-1 text-[11px]"></i>${menuFirst.name}
                  </span>
                  <span class="font-bold text-orange-600 shrink-0 ml-1">${menuFirst.price}</span>
                </div>
              </div>

              <!-- Quick action links -->
              <div class="mt-3 pt-2.5 flex items-center justify-between text-xs border-t border-slate-100 text-slate-500 font-medium">
                <span class="text-[11px] text-slate-400 truncate max-w-[170px]">
                  ${(r.tags || []).slice(0, 2).map(t => `#${t}`).join(' ')}
                </span>
                <span class="text-orange-600 hover:text-orange-700 font-semibold flex items-center gap-1">
                  상세보기 <i class="fa-solid fa-chevron-right text-[10px]"></i>
                </span>
              </div>
            </div>

          </div>
        `;
      }).join('');
    }

    // Search any region typed by the user
    async function handleRegionSearch() {
      const input = document.getElementById('regionSearchInput');
      const query = input.value.trim();
      if (!query) {
        showToast('검색할 지역명을 입력해주세요. (예: 전북 익산시 부송동 100, 성수동)', 'warn');
        return;
      }

      // Check if matches predefined regions
      const lower = query.toLowerCase();
      let matchedKey = null;
      for (const [key, val] of Object.entries(REGION_COORDS)) {
        if (key === 'all') continue;
        if (val.name.toLowerCase().includes(lower) || key.includes(lower)) {
          matchedKey = key;
          break;
        }
      }

      if (matchedKey) {
        selectQuickRegion(matchedKey);
        return;
      }

      // Priority 1: If Kakao REST API Key exists, use Kakao /v2/local/search/address.json
      if (kakaoApiKey) {
        showToast(`카카오 API로 '${query}' 주소 좌표를 변환하는 중...`, 'spin');
        try {
          const kakaoData = await requestKakaoAddressGeocode(query);
          if (kakaoData && kakaoData.documents && kakaoData.documents.length > 0) {
            const first = kakaoData.documents[0];
            const lat = parseFloat(first.y);
            const lng = parseFloat(first.x);
            const dispName = first.road_address ? first.road_address.address_name : first.address_name;

            currentRegion = 'custom_' + Date.now();
            currentRegionLabel = dispName || query;
            document.getElementById('locationSelect').value = 'all';

            if (leafletMap) {
              leafletMap.setView([lat, lng], 15);
            }

            showToast(`카카오 좌표 변환 성공: (${lat.toFixed(4)}, ${lng.toFixed(4)})`);
            fetchRestaurantsWithKakaoOrGemini(dispName || query, lat, lng);
            return;
          }
        } catch (kakaoErr) {
          console.warn('Kakao address API failed, falling back to public geocoder:', kakaoErr);
        }
      }

      // Priority 2: Fallback to OpenStreetMap Nominatim
      showToast(`'${query}' 지역 위치를 찾는 중...`, 'spin');
      try {
        const geoUrl = `https://nominatim.openstreetmap.org/search?format=json&countrycodes=kr&q=${encodeURIComponent(query)}`;
        const res = await fetch(geoUrl);
        const data = await res.json();

        if (data && data.length > 0) {
          const lat = parseFloat(data[0].lat);
          const lng = parseFloat(data[0].lon);
          const dispName = data[0].display_name.split(',')[0];

          currentRegion = 'custom_' + Date.now();
          currentRegionLabel = dispName || query;
          document.getElementById('locationSelect').value = 'all';

          // Move Map
          if (leafletMap) {
            leafletMap.setView([lat, lng], 14);
          }

          // Check if we have restaurants in RESTAURANTS near this location or containing string
          const hasLocal = RESTAURANTS.some(r => r.address.includes(query) || r.name.includes(query));

          if (!hasLocal) {
            showToast(`'${query}' 지역으로 이동했습니다. 맛집을 탐색합니다!`, 'spin');
            fetchRestaurantsWithKakaoOrGemini(query, lat, lng);
          } else {
            applyFilters();
            showToast(`'${query}' 지역 주변 맛집을 표시합니다.`);
          }
        } else {
          // If Nominatim fails, fallback to Gemini AI direct discovery
          currentRegion = 'custom_' + Date.now();
          currentRegionLabel = query;
          fetchRestaurantsWithKakaoOrGemini(query);
        }
      } catch (err) {
        console.error(err);
        fetchRestaurantsWithKakaoOrGemini(query);
      }
    }

    // Hybrid Restaurant Search: Uses Kakao Keyword Local API if key is present, else Gemini AI
    async function fetchRestaurantsWithKakaoOrGemini(regionQuery, lat = 37.55, lng = 126.98) {
      if (kakaoApiKey) {
        try {
          showToast(`카카오 로컬 API로 '${regionQuery}' 주변 음식점 검색 중...`, 'spin');
          // Category FD6 = 음식점, CE7 = 카페
          const kakaoUrl = `https://dapi.kakao.com/v2/local/search/keyword.json?query=${encodeURIComponent(regionQuery + ' 맛집')}&category_group_code=FD6&x=${lng}&y=${lat}&radius=2500&size=15&sort=accuracy`;
          const res = await fetch(kakaoUrl, {
            headers: { 'Authorization': `KakaoAK ${kakaoApiKey}` }
          });
          if (res.ok) {
            const data = await res.json();
            if (data.documents && data.documents.length > 0) {
              let addedCount = 0;
              data.documents.forEach((place, idx) => {
                if (!RESTAURANTS.some(r => r.name === place.place_name)) {
                  // Map category
                  let cat = 'korean';
                  const cName = place.category_name || '';
                  if (cName.includes('육류') || cName.includes('고기')) cat = 'meat';
                  else if (cName.includes('회') || cName.includes('해물')) cat = 'sashimi';
                  else if (cName.includes('일식') || cName.includes('초밥')) cat = 'japanese';
                  else if (cName.includes('중식')) cat = 'chinese';
                  else if (cName.includes('양식') || cName.includes('이탈리안')) cat = 'western';
                  else if (cName.includes('카페') || cName.includes('디저트')) cat = 'cafe';
                  else if (cName.includes('분식')) cat = 'snack';

                  // Simulated dual platform reviews based on Kakao presence
                  const kRate = +(4.0 + (Math.random() * 0.7)).toFixed(1);
                  const nRate = +(4.4 + (Math.random() * 0.5)).toFixed(2);

                  RESTAURANTS.unshift({
                    id: Date.now() + idx,
                    region: currentRegion,
                    name: place.place_name,
                    category: cat,
                    categoryName: place.category_name.split('>').pop().trim() || '음식점',
                    summary: `${place.address_name} 위치. 카카오맵 추천 인기 맛집`,
                    lat: parseFloat(place.y),
                    lng: parseFloat(place.x),
                    distance: place.distance ? `${place.distance}m` : `${200 + (idx * 100)}m`,
                    distMeters: parseInt(place.distance) || (200 + (idx * 100)),
                    address: place.road_address_name || place.address_name,
                    phone: place.phone || '02-1234-5678',
                    hours: '11:00 - 22:00',
                    naverRating: nRate,
                    naverReviews: Math.floor(800 + Math.random() * 3000),
                    kakaoRating: kRate,
                    kakaoReviews: Math.floor(200 + Math.random() * 1200),
                    parking: Math.random() > 0.4,
                    tags: ['카카오인증', '지역맛집', '인기식당'],
                    image: 'https://images.unsplash.com/photo-1555396273-367ea4eb4db5?auto=format&fit=crop&w=600&q=80',
                    menus: [
                      { name: '대표 인기 메뉴', price: '변동' },
                      { name: '시그니처 단품', price: '추천' }
                    ]
                  });
                  addedCount++;
                }
              });

              if (addedCount > 0) {
                showToast(`카카오 로컬 API로 검증된 맛집 ${addedCount}곳을 등록했습니다! 🌟`);
                applyFilters();
                return;
              }
            }
          }
        } catch (e) {
          console.warn('Kakao places search failed, falling back to Gemini AI:', e);
        }
      }

      // If Kakao places empty or not set, use Gemini AI
      fetchRestaurantsWithGemini(regionQuery, lat, lng);
    }

    function selectQuickRegion(regionKey) {
      currentRegion = regionKey;
      const rObj = REGION_COORDS[regionKey];
      if (rObj) {
        currentRegionLabel = rObj.name;
        document.getElementById('locationSelect').value = regionKey;
        if (leafletMap) {
          leafletMap.setView([rObj.lat, rObj.lng], rObj.zoom || 14);
        }
      }
      applyFilters();
      showToast(`${rObj ? rObj.name : '선택한 지역'}으로 설정되었습니다.`);
    }

    function fetchRestaurantsForCurrentSearch() {
      const q = document.getElementById('regionSearchInput').value.trim() || currentRegionLabel;
      const center = leafletMap ? leafletMap.getCenter() : null;
      fetchRestaurantsWithGemini(q, center ? center.lat : 37.55, center ? center.lng : 126.98);
    }

    // Dynamically fetch authentic restaurants for ANY region with Naver & Kakao rating expectations
    async function fetchRestaurantsWithGemini(regionQuery, centerLat = 37.55, centerLng = 126.98) {
      if (isAiFetching) return;
      isAiFetching = true;

      const btn = document.getElementById('btnFetchAiRestaurants');
      if (btn) {
        btn.disabled = true;
        btn.innerHTML = '<i class="fa-solid fa-spinner fa-spin"></i> AI 맛집 탐색 중...';
      }
      showToast(`'${regionQuery}' 네이버/카카오 검증 맛집을 실시간 수집 중입니다...`, 'spin');

      const apiKey = "";
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

      const systemPrompt = `You are a premier Korean food critic and local gourmet data curator.
Given a region query in Korea, return exactly 6 to 8 famous, authentic, highly rated restaurants in or immediately around that region.
Include realistic and accurate Naver Map ratings (typically 4.4 - 4.9 out of 5.0) and KakaoMap ratings (typically 3.9 - 4.7 out of 5.0).
Categories must be one of: 'korean', 'meat', 'sashimi', 'japanese', 'chinese', 'western', 'cafe', 'snack'.
Return solely a valid JSON array matching the required schema. Ensure lat/lng coordinates are realistic for the requested region in South Korea.`;

      const userPrompt = `대한민국 '${regionQuery}' 지역(주변 반경 1~2km)의 현지인 인증 및 네이버/카카오맵에서 평점이 매우 높은 대표 맛집 6~8곳을 엄선해주세요.
한식, 고기, 회, 일식, 중식 등 카테고리를 골고루 섞어주세요.
각 식당마다 상호명, 카테고리(korean, meat, sashimi, japanese, chinese, western, cafe, snack 중 1), 카테고리 한국어명, 한줄 요약, 대략적 위도(lat), 경도(lng), 도로명 주소, 전화번호, 영업시간, 네이버평점(4.4~4.9), 네이버리뷰수, 카카오평점(3.8~4.7), 카카오리뷰수, 주차여부(boolean), 해시태그(3개), 대표메뉴(2~3개 및 가격)를 포함하세요.`;

      const payload = {
        contents: [{ parts: [{ text: userPrompt }] }],
        systemInstruction: { parts: [{ text: systemPrompt }] },
        generationConfig: {
          responseMimeType: "application/json",
          responseSchema: {
            type: "ARRAY",
            items: {
              type: "OBJECT",
              properties: {
                name: { type: "STRING" },
                category: { type: "STRING" },
                categoryName: { type: "STRING" },
                summary: { type: "STRING" },
                lat: { type: "NUMBER" },
                lng: { type: "NUMBER" },
                address: { type: "STRING" },
                phone: { type: "STRING" },
                hours: { type: "STRING" },
                naverRating: { type: "NUMBER" },
                naverReviews: { type: "INTEGER" },
                kakaoRating: { type: "NUMBER" },
                kakaoReviews: { type: "INTEGER" },
                parking: { type: "BOOLEAN" },
                tags: { type: "ARRAY", items: { type: "STRING" } },
                menus: {
                  type: "ARRAY",
                  items: {
                    type: "OBJECT",
                    properties: {
                      name: { type: "STRING" },
                      price: { type: "STRING" }
                    },
                    required: ["name", "price"]
                  }
                }
              },
              required: ["name", "category", "categoryName", "summary", "lat", "lng", "address", "naverRating", "kakaoRating", "menus"]
            }
          }
        }
      };

      try {
        const response = await fetch(apiUrl, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        const data = await response.json();
        const jsonText = data.candidates?.[0]?.content?.parts?.[0]?.text;
        
        if (jsonText) {
          const fetchedItems = JSON.parse(jsonText);
          let newCount = 0;
          let firstCoord = null;

          fetchedItems.forEach((item, index) => {
            // Check if already exists by name
            if (!RESTAURANTS.some(r => r.name === item.name)) {
              const newId = Date.now() + index;
              const photoCategoryKeywords = {
                meat: 'korean-bbq,meat',
                sashimi: 'sashimi,seafood',
                japanese: 'sushi,ramen',
                chinese: 'chinese-food,dumpling',
                western: 'pasta,pizza',
                cafe: 'cafe,coffee',
                snack: 'korean-street-food,tteokbokki',
                korean: 'korean-food'
              };
              const tag = photoCategoryKeywords[item.category] || 'restaurant,food';
              
              RESTAURANTS.unshift({
                ...item,
                id: newId,
                region: currentRegion,
                distance: `${150 + (index * 90)}m`,
                distMeters: 150 + (index * 90),
                image: `https://images.unsplash.com/photo-1555396273-367ea4eb4db5?auto=format&fit=crop&w=600&q=80`
              });
              newCount++;
              if (!firstCoord && item.lat && item.lng) {
                firstCoord = [item.lat, item.lng];
              }
            }
          });

          if (firstCoord && leafletMap) {
            leafletMap.setView(firstCoord, 14);
          }

          showToast(`'${regionQuery}' 검증 맛집 ${newCount}곳을 성공적으로 불러왔습니다! 🎉`);
          applyFilters();
        } else {
          showToast('맛집 정보를 불러오는 중 일시적인 오류가 발생했습니다.', 'warn');
        }
      } catch (err) {
        console.error('Gemini Restaurant Fetch Error:', err);
        showToast('네트워크 상태를 확인해주세요. 기본 데이터로 표시합니다.', 'warn');
      } finally {
        isAiFetching = false;
        if (btn) {
          btn.disabled = false;
          btn.innerHTML = '<i class="fa-solid fa-bolt"></i> 이 지역 맛집 AI 즉시 발굴';
        }
      }
    }

    function initMap() {
      const mapContainer = document.getElementById('map');
      if (!mapContainer) {
        console.warn('Map element #map not present in DOM yet.');
        return;
      }
      const center = REGION_COORDS['all'];
      leafletMap = L.map('map').setView([center.lat, center.lng], 11);

      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
        maxZoom: 19
      }).addTo(leafletMap);
    }

    function updateMapMarkers(items) {
      if (!leafletMap) return;

      // Clear existing markers
      mapMarkers.forEach(m => leafletMap.removeLayer(m));
      mapMarkers = [];

      items.forEach(r => {
        if (!r.lat || !r.lng) return;

        const iconHtml = `
          <div class="relative group cursor-pointer">
            <div class="w-8 h-8 rounded-full bg-orange-600 text-white flex items-center justify-center font-bold text-xs shadow-lg border-2 border-white hover:scale-125 transition">
              <i class="fa-solid fa-utensils"></i>
            </div>
            <div class="absolute -bottom-1 left-1/2 -translate-x-1/2 w-2 h-2 bg-orange-600 rotate-45"></div>
          </div>
        `;

        const customIcon = L.divIcon({
          html: iconHtml,
          className: 'custom-leaflet-marker',
          iconSize: [32, 32],
          iconAnchor: [16, 32],
          popupAnchor: [0, -32]
        });

        const popupContent = `
          <div class="text-left select-none max-w-[200px]">
            <div class="font-bold text-sm text-slate-900">${r.name}</div>
            <div class="text-xs text-orange-600 font-semibold mb-1">${r.categoryName} • ${r.distance || '주변'}</div>
            <div class="text-[11px] text-slate-500 mb-2 truncate">${r.address}</div>
            <div class="flex items-center gap-1.5 text-xs mb-2">
              <span class="text-emerald-700 font-bold bg-emerald-50 px-1.5 py-0.5 rounded border border-emerald-200">N ${r.naverRating || '4.5'}</span>
              <span class="text-yellow-800 font-bold bg-yellow-50 px-1.5 py-0.5 rounded border border-yellow-300">K ${r.kakaoRating || '4.2'}</span>
            </div>
            <button onclick="openDetailModal(${r.id})" class="w-full py-1 text-center bg-orange-500 text-white rounded-lg text-xs font-semibold hover:bg-orange-600">
              상세 정보 열기
            </button>
          </div>
        `;

        const marker = L.marker([r.lat, r.lng], { icon: customIcon })
          .addTo(leafletMap)
          .bindPopup(popupContent);

        mapMarkers.push(marker);
      });

      // Recenter map to bounds if items present
      if (items.length > 0 && mapMarkers.length > 0) {
        const group = new L.featureGroup(mapMarkers);
        leafletMap.fitBounds(group.getBounds().pad(0.15));
      }
    }

    function openDetailModal(id) {
      const r = RESTAURANTS.find(item => item.id === id);
      if (!r) return;

      currentSelectedId = r.id;
      playBeep(850, 0.04);

      document.getElementById('modalImg').src = r.image;
      document.getElementById('modalCategoryBadge').innerText = r.categoryName;
      document.getElementById('modalTitle').innerText = r.name;
      document.getElementById('modalDesc').innerText = r.summary;

      document.getElementById('modalNaverRate').innerText = (r.naverRating || 4.5).toFixed(2);
      document.getElementById('modalNaverReviews').innerText = `리뷰 ${(r.naverReviews || 500).toLocaleString()}개`;
      document.getElementById('modalKakaoRate').innerText = (r.kakaoRating || 4.2).toFixed(1);
      document.getElementById('modalKakaoReviews').innerText = `리뷰 ${(r.kakaoReviews || 250).toLocaleString()}개`;

      document.getElementById('modalAddress').innerText = r.address;
      document.getElementById('modalDistance').innerText = `내 위치로부터 ${r.distance || '주변'} (도보 약 ${Math.max(1, Math.round((r.distMeters || 400) / 75))}분)`;
      document.getElementById('modalHours').innerText = r.hours || '11:30 - 22:00';
      document.getElementById('modalPhone').innerText = r.phone || '02-123-4567';

      // Render Menus
      const menuList = document.getElementById('modalMenuList');
      menuList.innerHTML = (r.menus || []).map(m => `
        <div class="flex items-center justify-between py-1.5 border-b border-slate-100 last:border-0">
          <span class="font-medium text-slate-800">${m.name}</span>
          <span class="font-bold text-orange-600">${m.price}</span>
        </div>
      `).join('');

      // Render Tags
      const tagsContainer = document.getElementById('modalTags');
      tagsContainer.innerHTML = (r.tags || []).map(t => `
        <span class="px-2.5 py-1 bg-slate-100 text-slate-700 rounded-lg text-xs font-medium"># ${t}</span>
      `).join('');

      // Set External Map Links
      const encodedName = encodeURIComponent(`${r.name} ${r.address.split(' ')[0]}`);
      document.getElementById('btnNaverMap').href = `https://map.naver.com/p/search/${encodedName}`;
      document.getElementById('btnKakaoMap').href = `https://map.kakao.com/link/search/${encodedName}`;

      document.getElementById('detailModal').classList.remove('hidden');
    }

    function closeModal(modalId) {
      document.getElementById(modalId).classList.add('hidden');
    }

    let rouletteInterval = null;
    let rouletteWinner = null;

    function openRouletteModal() {
      playBeep(520, 0.08);
      document.getElementById('rouletteCategory').innerText = '오늘 당신의 선택은?';
      document.getElementById('rouletteTitle').innerText = '룰렛 돌리기 버튼을 눌러주세요!';
      document.getElementById('rouletteRating').innerHTML = '<span>네이버 -</span> • <span>카카오 -</span>';
      document.getElementById('btnRouletteDetail').classList.add('hidden');
      document.getElementById('rouletteModal').classList.remove('hidden');
    }

    function spinRoulette() {
      let list = RESTAURANTS;
      if (currentRegion !== 'all') {
        list = list.filter(r => r.region === currentRegion);
      }
      if (list.length === 0) list = RESTAURANTS;

      const spinBtn = document.getElementById('btnSpinRoulette');
      const detailBtn = document.getElementById('btnRouletteDetail');
      spinBtn.disabled = true;
      spinBtn.classList.add('opacity-50');
      detailBtn.classList.add('hidden');

      let counter = 0;
      const totalSteps = 25;
      clearInterval(rouletteInterval);

      rouletteInterval = setInterval(() => {
        counter++;
        const randItem = list[Math.floor(Math.random() * list.length)];
        document.getElementById('rouletteCategory').innerText = `${randItem.categoryName} 추천!`;
        document.getElementById('rouletteTitle').innerText = randItem.name;
        document.getElementById('rouletteRating').innerHTML = `
          <span class="text-emerald-600 font-bold">N ${randItem.naverRating}</span> • 
          <span class="text-amber-600 font-bold">K ${randItem.kakaoRating}</span>
        `;
        playBeep(400 + (counter * 20), 0.03);

        if (counter >= totalSteps) {
          clearInterval(rouletteInterval);
          rouletteWinner = randItem;
          spinBtn.disabled = false;
          spinBtn.classList.remove('opacity-50');
          detailBtn.classList.remove('hidden');
          playBeep(880, 0.25);
          showToast(`오늘의 추천 맛집: ${randItem.name}!`);
        }
      }, 70);
    }

    function viewRouletteWinner() {
      if (!rouletteWinner) return;
      closeModal('rouletteModal');
      openDetailModal(rouletteWinner.id);
    }

    function toggleFavorite(id) {
      const idx = favorites.indexOf(id);
      if (idx > -1) {
        favorites.splice(idx, 1);
        showToast('찜 목록에서 제거되었습니다.', 'info');
      } else {
        favorites.push(id);
        showToast('찜 목록에 저장되었습니다! ❤️', 'heart');
      }
      localStorage.setItem('tasteMap_favs', JSON.stringify(favorites));
      updateFavBadge();
      applyFilters();
    }

    function updateFavBadge() {
      document.getElementById('favCount').innerText = favorites.length;
    }

    function toggleFavoritesOnly() {
      showFavoritesOnly = !showFavoritesOnly;
      const btn = document.getElementById('favToggleBtn');
      if (showFavoritesOnly) {
        btn.classList.add('bg-rose-100', 'border-rose-300', 'text-rose-700');
        showToast('내가 찜한 맛집만 모아봅니다.');
      } else {
        btn.classList.remove('bg-rose-100', 'border-rose-300', 'text-rose-700');
      }
      applyFilters();
    }

    function toggleFilter(type) {
      if (type === 'kakao4') {
        filterKakao4Active = !filterKakao4Active;
        const btn = document.getElementById('filterKakao4');
        btn.classList.toggle('ring-2');
        btn.classList.toggle('ring-amber-500');
      } else if (type === 'naver45') {
        filterNaver45Active = !filterNaver45Active;
        const btn = document.getElementById('filterNaver45');
        btn.classList.toggle('ring-2');
        btn.classList.toggle('ring-emerald-500');
      } else if (type === 'parking') {
        filterParkingActive = !filterParkingActive;
        const btn = document.getElementById('filterParking');
        btn.classList.toggle('bg-slate-800');
        btn.classList.toggle('text-white');
      }
      applyFilters();
    }

    function changeRegion(region) {
      selectQuickRegion(region);
    }

    function getUserLocation() {
      showToast('현재 위치 GPS를 탐색 중입니다...', 'spin');
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          pos => {
            const lat = pos.coords.latitude;
            const lng = pos.coords.longitude;
            if (leafletMap) {
              leafletMap.setView([lat, lng], 15);
            }
            currentRegion = 'gps_user';
            currentRegionLabel = '현재 내 위치 반경 2km';
            showToast('현재 위치를 확인했습니다! 주변 맛집을 탐색합니다.');
            fetchRestaurantsWithGemini('내 주변', lat, lng);
          },
          err => {
            showToast('위치 권한을 확인해주세요. 기본 설정 위치를 유지합니다.', 'warn');
          },
          { timeout: 5000 }
        );
      } else {
        showToast('이 브라우저는 위치 서비스를 지원하지 않습니다.');
      }
    }

    function setViewMode(mode) {
      viewMode = mode;
      const listCol = document.getElementById('listColumn');
      const mapCol = document.getElementById('mapColumn');
      const gridBtn = document.getElementById('viewGridBtn');
      const splitBtn = document.getElementById('viewSplitBtn');

      if (mode === 'split') {
        mapCol.classList.remove('hidden');
        listCol.classList.remove('w-full');
        listCol.classList.add('lg:w-[55%]');
        splitBtn.classList.add('bg-white', 'shadow-xs', 'font-semibold', 'text-orange-600');
        splitBtn.classList.remove('text-slate-600');
        gridBtn.classList.remove('bg-white', 'shadow-xs', 'font-semibold', 'text-orange-600');
        gridBtn.classList.add('text-slate-600');
        setTimeout(() => leafletMap && leafletMap.invalidateSize(), 200);
      } else {
        mapCol.classList.add('hidden');
        listCol.classList.add('w-full');
        listCol.classList.remove('lg:w-[55%]');
        gridBtn.classList.add('bg-white', 'shadow-xs', 'font-semibold', 'text-orange-600');
        gridBtn.classList.remove('text-slate-600');
        splitBtn.classList.remove('bg-white', 'shadow-xs', 'font-semibold', 'text-orange-600');
        splitBtn.classList.add('text-slate-600');
      }
    }

    function resetFilters() {
      currentRegion = 'all';
      currentRegionLabel = '전국 인기 맛집';
      currentCategory = 'all';
      currentSearch = '';
      document.getElementById('searchInput').value = '';
      document.getElementById('regionSearchInput').value = '';
      document.getElementById('locationSelect').value = 'all';
      filterKakao4Active = false;
      filterNaver45Active = false;
      filterParkingActive = false;
      showFavoritesOnly = false;
      document.getElementById('filterKakao4').classList.remove('ring-2', 'ring-amber-500');
      document.getElementById('filterNaver45').classList.remove('ring-2', 'ring-emerald-500');
      document.getElementById('filterParking').classList.remove('bg-slate-800', 'text-white');
      document.getElementById('favToggleBtn').classList.remove('bg-rose-100', 'border-rose-300', 'text-rose-700');
      document.getElementById('sortSelect').value = 'recommend';
      renderCategoryTabs();
      applyFilters();
      showToast('검색 및 필터 조건이 초기화되었습니다.');
    }

    function copyAddress() {
      const address = document.getElementById('modalAddress').innerText;
      const dummy = document.createElement('textarea');
      document.body.appendChild(dummy);
      dummy.value = address;
      dummy.select();
      document.execCommand('copy');
      document.body.removeChild(dummy);
      showToast('주소가 클립보드에 복사되었습니다! 📋');
    }

    let toastTimeout = null;
    function showToast(msg, type = 'check') {
      const toast = document.getElementById('toast');
      const toastMsg = document.getElementById('toastMsg');
      const toastIcon = document.getElementById('toastIcon');

      toastMsg.innerText = msg;
      if (type === 'heart') {
        toastIcon.className = 'fa-solid fa-heart text-rose-500';
      } else if (type === 'warn') {
        toastIcon.className = 'fa-solid fa-triangle-exclamation text-amber-400';
      } else if (type === 'spin') {
        toastIcon.className = 'fa-solid fa-circle-notch fa-spin text-orange-400';
      } else {
        toastIcon.className = 'fa-solid fa-check text-emerald-400';
      }

      toast.classList.remove('opacity-0', 'pointer-events-none');
      toast.classList.add('opacity-100');

      clearTimeout(toastTimeout);
      toastTimeout = setTimeout(() => {
        toast.classList.remove('opacity-100');
        toast.classList.add('opacity-0', 'pointer-events-none');
      }, 2400);
    }

    window.onload = function () {
      renderCategoryTabs();
      updateFavBadge();
      updateKakaoKeyStatus();
      initMap();
      applyFilters();

      // Close modal when backdrop clicked or Escape pressed
      window.addEventListener('click', (e) => {
        const detailModal = document.getElementById('detailModal');
        const rouletteModal = document.getElementById('rouletteModal');
        const kakaoModal = document.getElementById('kakaoModal');
        if (e.target === detailModal) closeModal('detailModal');
        if (e.target === rouletteModal) closeModal('rouletteModal');
        if (e.target === kakaoModal) closeModal('kakaoModal');
      });

      window.addEventListener('keydown', (e) => {
        if (e.key === 'Escape') {
          closeModal('detailModal');
          closeModal('rouletteModal');
          closeModal('kakaoModal');
        }
      });
    };
  </script>
</body>
</html>
