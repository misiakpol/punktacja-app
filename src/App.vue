<template>
  <!-- Główny kontener aplikacji (Ciemne tło, zajmuje cały ekran) -->
  <div class=" relative h-[100dvh] bg-[#141414] text-white flex flex-col font-sans overflow-hidden">

    <!-- 🟢 EKRAN 1: STRONA GŁÓWNA (Kafelki i podsumowanie) -->
    <main 
    v-show="currentView === 'home'"
    class="flex-grow p-6 flex flex-col gap-4 items-center overflow-y-auto pb-32"
    >
      <div class="flex flex-col uppercase gap-1 items-center w-full">
        <h1 class="text-3xl font-black font-lemon tracking-widest text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-blue-500 mb-2 text-center">
        Wybór
        </h1>
        <p class="text-gray-400 font-inter text-shadow-normal text-center">Wyświetl wyniki wygranego dla wybranego okresu</p>

      </div>

      <!-- Wybór okresu: -->
      <div class="mt-4 w-full z-10 max-w-sm flex gap-2">
        <select 
          v-model="selectedPeriodId" 
          class="flex-1 bg-[#2a2a2a] text-white font-inter border border-purple-500 rounded-lg p-3 outline-none focus:ring-2 focus:ring-purple-400 cursor-pointer appearance-none text-center"
        >
          <option v-for="period in periods" :key="period.id" :value="period.id">
            {{ period.period_name }}
          </option>
        </select>
        <button 
          @click="createNewPeriod"
          class="flex items-center justify-center bg-[#2a2a2a] text-white px-4 border border-purple-500 rounded-lg hover:bg-purple-500/20 transition-all shadow-[0_0_10px_rgba(168,85,247,0.2)] active:scale-95"
          title="Dodaj nową turę"
        >
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"/><path d="M12 5v14"/></svg>
        </button>
      </div>

      
      <!-- KAFELKI Z IMIONAMI  -->
      <div class="grid grid-cols-2 w-full max-w-sm gap-4 mt-4 z-10">

        <div class="flex flex-col gap-3">
          <div class="flex flex-col items-center bg-[#2a2a2a] py-6 rounded-2xl shadow-lg border border-blue-500 bg-gradient-to-b from-[#00CEC4]/10 to-blue-500/10 w-full">
            <span class="text-white font-inter font-medium text-xl uppercase mb-2">Adrian</span>
            <span class="text-4xl font-bold text-transparent bg-clip-text bg-gradient-to-b from-[#00CEC4] to-blue-500">
              {{ adrianWins }}
            </span>
          </div>
          <div :class="['w-full p-[1px] rounded-full', getIndicatorStyles(adrianWins, mateuszWins).border]">
             <div :class="['h-3 w-full rounded-full', getIndicatorStyles(adrianWins, mateuszWins).fill]"></div>
          </div>
        </div>

        <div class="flex flex-col gap-3">
          <div class="flex flex-col items-center bg-[#2a2a2a] py-6 rounded-2xl shadow-lg border border-purple-500 bg-gradient-to-b from-[#AA00E3]/10 to-purple-500/10 w-full">
            <span class="text-white font-inter font-medium text-xl uppercase mb-2">Mateusz</span>
            <span class="text-4xl font-bold text-transparent bg-clip-text bg-gradient-to-b from-purple-500 to-[#AA00E3]">
              {{ mateuszWins }}
            </span>
          </div>
          <div :class="['w-full p-[1px] rounded-full', getIndicatorStyles(mateuszWins, adrianWins).border]">
             <div :class="['h-3 w-full rounded-full', getIndicatorStyles(mateuszWins, adrianWins).fill]"></div>
          </div>
        </div>
      </div>
      
      <!-- KOMUNIKAT O KOŃCU GRY (DO 10 PUNKTÓW) -->
      <div v-if="adrianWins >= 10 || mateuszWins >= 10" class="w-full max-w-sm mt-4 bg-gradient-to-r from-red-500 to-orange-500 rounded-2xl p-6 text-center shadow-[0_0_25px_rgba(239,68,68,0.4)] z-10 border border-white/20">
        <h3 class="text-3xl font-black font-lemon mb-2 text-white drop-shadow-md">KONIEC GRY!</h3>
        <p class="text-white/90 font-inter font-medium mb-5 uppercase tracking-wider text-sm">
          Wymagane 10 zwycięstw zostało osiągnięte.
        </p>
        <button 
          @click="createNewPeriod"
          class="bg-white text-red-600 font-black px-6 py-3 w-full rounded-xl uppercase tracking-widest hover:bg-gray-100 transition-transform shadow-lg active:scale-95"
        >
          Zacznij nową turę
        </button>
      </div>
      
      <!-- KRÓL OKRESU -->
      <div class="relative w-full max-w-sm rounded-[22px] p-6 sm:p-8 flex flex-col items-center overflow-hidden shrink-0">
          
          <!-- 🟢 DEDYKOWANY CANVAS DO KONFETTI 🟢 -->
          <canvas ref="confettiCanvas" class="absolute inset-0 w-full h-full pointer-events-none z-0"></canvas>

          <!-- Efekt promienistego światła w tle -->
          <div class="absolute inset-0 bg-[radial-gradient(circle_at_center,_var(--tw-gradient-stops))] from-blue-500/20 via-transparent to-transparent pointer-events-none z-0"></div>

          <h2 class="text-3xl font-black font-lemon tracking-widest mb-6 z-10 text-transparent bg-clip-text bg-gradient-to-r from-blue-400 to-purple-400 text-center">
            KRÓL<br>TURY
          </h2>

          <!-- Skalowalny kontener zdjęcia (aspect-square robi magię) -->
          <div class="relative z-10 w-full max-w-[260px] aspect-square shrink-0">
            <div class="absolute inset-0 bg-gradient-to-r from-[#00CEC4] to-[#AA00E3] rounded-2xl blur-xl opacity-50 animate-pulse"></div>
            <img 
              :src="winner === 'Adrian' ? imgAdrian : (winner === 'Mateusz' ? imgMateusz : imgRemis)"
              class="relative w-full h-full object-cover rounded-2xl border-2 border-white/10 shadow-2xl z-10" 
              alt="Morda Zwycięzcy"
            >
          </div>

          <p class="mt-6 text-3xl font-inter font-bold z-10 tracking-wide uppercase">
            {{ winner }}
          </p>
        </div>
    </main>

    <!-- 🟢 EKRAN 2: HISTORIA (Tabela) -->
    <div 
      v-show="currentView === 'history'" 
      class="flex-grow flex flex-col p-4 pt-10 pb-32 overflow-hidden bg-[#141414]"
    >
      <!-- NAGŁÓWEK Z IKONĄ EDYCJI -->
      <div class="relative w-full max-w-lg mx-auto flex justify-center items-center mb-6">
        <h2 class="text-3xl font-black font-lemon tracking-widest text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-blue-500">
          HISTORIA
        </h2>
        
        <!-- Przycisk ołówka (Tryb usuwania) -->
        <button @click="toggleDeleteMode" class="absolute right-2 p-2 bg-white/5 rounded-full hover:bg-white/10 transition-colors">
          <svg v-if="!isDeleteMode" class="w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"></path></svg>
          <!-- Czerwony krzyżyk (Wyjście z trybu) -->
          <svg v-else class="w-5 h-5 text-red-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
        </button>
      </div>
      
      <!-- FILTRY (Rozwijane selecty ze spójnym, mocnym wypełnieniem po zaznaczeniu) -->
      <div class="w-full flex flex-wrap justify-center gap-3 mb-6 max-w-lg mx-auto">
        
        <!-- Filtr 1: Tury -->
        <select 
          v-model="filterPeriod" 
          :class="[
            'flex-1 min-w-[130px] text-xs font-inter p-3 rounded-xl outline-none appearance-none cursor-pointer text-center transition-all duration-300 border',
            filterPeriod !== '' 
              ? 'bg-gradient-to-br from-[#bd4bff] to-[#7b01a3] border-[#AA00E3] text-white font-bold shadow-[0_0_20px_rgba(170,0,227,0.6)] scale-[1.02]' 
              : 'bg-[#2a2a2a] text-gray-400 border-white/10 focus:border-[#AA00E3]'
          ]"
        >
          <option value="">Wszystkie Tury</option>
          <option v-for="period in periods" :key="period.id" :value="period.id">
            {{ period.period_name }}
          </option>
        </select>
        
        <!-- Filtr 2: Sporty -->
        <select 
          v-model="filterCategory" 
          :class="[
            'flex-1 min-w-[130px] text-xs font-inter p-3 rounded-xl outline-none appearance-none cursor-pointer text-center transition-all duration-300 border',
            filterCategory !== '' 
              ? 'bg-gradient-to-br from-[#bd4bff] to-[#7b01a3] border-[#AA00E3] text-white font-bold shadow-[0_0_20px_rgba(170,0,227,0.6)] scale-[1.02]'
              : 'bg-[#2a2a2a] text-gray-400 border-white/10 focus:border-[#AA00E3]'
          ]"
        >
          <option value="">Wszystkie Sporty</option>
          <option v-for="cat in availableCategories" :key="cat.id" :value="cat.id">
            {{ cat.name }}
          </option>
        </select>

        <!-- Filtr 3: Gracze -->
        <select 
          v-model="filterPlayer" 
          :class="[
            'flex-1 min-w-[130px] text-xs font-inter p-3 rounded-xl outline-none appearance-none cursor-pointer text-center transition-all duration-300 border',
            filterPlayer !== '' 
              ? 'bg-gradient-to-br from-[#bd4bff] to-[#7b01a3] border-[#AA00E3] text-white font-bold shadow-[0_0_20px_rgba(170,0,227,0.6)] scale-[1.02]'
              : 'bg-[#2a2a2a] text-gray-400 border-white/10 focus:border-[#AA00E3]'
          ]"
        >
          <option value="">Wszyscy</option>
          <option v-for="player in availablePlayers" :key="player.id" :value="player.id">
            {{ player.name }}
          </option>
        </select>

      </div>

      <!-- PANEL AKCJI - Pojawia się, gdy włączony jest tryb usuwania (ołówek) -->
      <div v-if="isDeleteMode" class="w-full max-w-lg mx-auto flex justify-between items-center mb-4 px-2">
        <span class="text-sm font-inter text-gray-400">
          Zaznaczono: <span class="text-white font-bold">{{ selectedMatches.length }}</span>
        </span>
        <button 
          @click="deleteSelected" 
          :disabled="selectedMatches.length === 0" 
          :class="['px-4 py-2 rounded-xl font-bold text-sm transition-all', selectedMatches.length > 0 ? 'bg-red-500/20 text-red-400 border border-red-500/50 shadow-[0_0_15px_rgba(239,68,68,0.2)]' : 'bg-gray-800 text-gray-600 cursor-not-allowed border border-transparent']"
        >
          Usuń zaznaczone
        </button>
      </div>
      <!-- KONTENER TABELI Z POZIOMYM SCROLLEM -->
      <div class="w-full max-w-lg mx-auto bg-[#1e1e1e] rounded-2xl border border-white/10 overflow-x-auto shadow-2xl pb-2">
        <table class="w-full text-left border-collapse whitespace-nowrap text-sm">
          
          <thead class="bg-[#2a2a2a] text-gray-400 font-inter text-xs uppercase tracking-wider">
            <tr>
              <!-- NOWA KOLUMNA CHECKBOXÓW W NAGŁÓWKU -->
              <th v-if="isDeleteMode" class="p-5 border-b border-white/5 text-center">
                🗑️
              </th>
              
              <th @click="handleSort('match_date')" class="sticky left-0 bg-[#2a2a2a] z-10 p-5 cursor-pointer hover:text-white transition-colors border-b border-white/5 shadow-[4px_0_10px_rgba(0,0,0,0.3)]">
                <div class="flex items-center gap-2">
                  Data
                  <span v-if="sortBy === 'match_date'" class="text-purple-400">{{ sortDesc ? '↓' : '↑' }}</span>
                </div>
              </th>
              <th @click="handleSort('category_id')" class="p-5 cursor-pointer hover:text-white transition-colors border-b border-white/5">
                <div class="flex items-center gap-2">
                  Dyscyplina
                  <span v-if="sortBy === 'category_id'" class="text-purple-400">{{ sortDesc ? '↓' : '↑' }}</span>
                </div>
              </th>
              <th @click="handleSort('winner_id')" class="p-5 cursor-pointer hover:text-white transition-colors border-b border-white/5">
                <div class="flex items-center gap-2">
                  Zwycięzca
                  <span v-if="sortBy === 'winner_id'" class="text-purple-400">{{ sortDesc ? '↓' : '↑' }}</span>
                </div>
              </th>
            </tr>
          </thead>
          
          <!-- DANE Z PROCESOWANEJ LISTY -->
          <tbody class="text-gray-200 font-inter font-medium">
            <tr v-for="match in processedResults" :key="match.id" class="hover:bg-white/5 transition-colors border-b border-white/5 last:border-0">
              
              <!-- KOLUMNA CHECKBOXÓW W WIERSZU -->
              <td v-if="isDeleteMode" class="p-5 border-b border-white/5 text-center bg-[#242424]">
                <input 
                  type="checkbox" 
                  :value="match.id" 
                  v-model="selectedMatches" 
                  class="w-5 h-5 accent-red-500 cursor-pointer rounded bg-[#141414] border-gray-600 transition-all"
                >
              </td>

              <!-- ZAMROŻONA KOLUMNA DATY -->
              <td class="sticky left-0 bg-[#1e1e1e] z-10 p-5 shadow-[4px_0_10px_rgba(0,0,0,0.15)] text-gray-300">
                {{ match.match_date }}
              </td>
              
              <!-- KATEGORIA (Dynamicznie mapowana) -->
              <td class="p-5">
                <div class="flex items-center gap-2">
                  <img :src="availableCategories.find(c => c.id === match.category_id)?.icon" class="w-5 h-5 opacity-80" />
                  {{ availableCategories.find(c => c.id === match.category_id)?.name || 'Nieznana' }}
                </div>
              </td>
              
              <!-- ZWYCIĘZCA (Dynamicznie mapowany) -->
              <td :class="[
                'p-5 uppercase font-bold', 
                availablePlayers.find(p => p.id === match.winner_id)?.name === 'Adrian' ? 'text-blue-400' : 'text-purple-400'
              ]">
                {{ availablePlayers.find(p => p.id === match.winner_id)?.name || 'Nieznany' }}
              </td>
              
            </tr>
            <!-- Jeśli brak danych po filtrach -->
            <tr v-if="processedResults.length === 0">
              <td colspan="3" class="p-10 text-center text-gray-500 font-inter italic">
                Brak wyników spełniających kryteria.
              </td>
            </tr>
          </tbody>

        </table>
      </div>
    </div>

    <!-- 🟢 EKRAN DODAWANIA WYNIKU (Płynny wjazd z dołu) -->
    <Transition
      enter-active-class="transition-all duration-500 ease-in-out"
      enter-from-class="opacity-0 translate-y-12"
      enter-to-class="opacity-100 translate-y-0"
      leave-active-class="transition-all duration-300 ease-in-out"
      leave-from-class="opacity-100 translate-y-0"
      leave-to-class="opacity-0 translate-y-12"
    >
      <div 
        v-if="isAddMode" 
        class="absolute inset-0 z-40 bg-[#1e1e1e]/80 backdrop-blur-xl flex flex-col items-center p-6 pt-16 pb-32 overflow-y-auto"
      >
        <h2 class="text-3xl font-black font-lemon tracking-widest text-transparent bg-clip-text bg-gradient-to-r from-[#F28100] to-[#AA00E3] mb-8">
          NOWY WYNIK
        </h2>
        
        <!-- ZBUDOWANY FORMULARZ DODAWANIA WYNIKU Z KAFELKAMI, PRZYCISKAMI I KALENDARZEM -->
        <div class="w-full flex flex-col gap-8 max-w-sm mt-4">

          <!-- 1. DATA MECZU -->
          <div class="flex flex-col gap-2 w-full">
            <label class="text-gray-400 font-inter text-sm uppercase tracking-wider pl-1">Data spotkania</label>
            <!-- style="color-scheme: dark" zmusza systemowy kalendarz do ciemnego motywu -->
            <input 
              type="date" 
              v-model="newMatchDate"
              style="color-scheme: dark;" 
              class="w-full bg-[#2a2a2a] text-white font-inter rounded-xl p-4 outline-none border border-white/10 focus:border-[#F28100] focus:ring-1 focus:ring-[#F28100] transition-colors appearance-none"
            />
          </div>

          <!-- 2. KATEGORIA (Z zawijaniem wierszy) -->
          <!-- 2. KATEGORIA (Flexbox wyśrodkowujący nieparzyste kafelki) -->
          <div class="flex flex-col gap-2 w-full">
            <label class="text-gray-400 font-inter text-sm uppercase tracking-wider pl-1">Wybierz sport</label>
            
            <!-- Flex wrap z centrowaniem -->
            <div class="flex flex-wrap justify-center gap-4 w-full">
              
              <button 
                v-for="cat in formCategories" 
                :key="cat.id"
                @click="selectedCategoryId = cat.id"
                :class="[
                  'flex flex-col items-center justify-center gap-3 rounded-2xl border-2 transition-all duration-300 aspect-square w-[calc(50%-0.5rem)]',
                  selectedCategoryId === cat.id 
                    ? 'border-[#00CEC4] bg-[#00CEC4]/10 shadow-[0_0_20px_rgba(0,206,196,0.25)] scale-105' 
                    : 'border-white/5 bg-[#2a2a2a] hover:bg-white/5 opacity-70 hover:opacity-100'
                ]"
              >
                <!-- IKONY PNG DO WYBORU -->
                <img 
                  :src="cat.icon" 
                  class="w-16 h-16 sm:w-20 sm:h-20 object-contain drop-shadow-lg transition-transform duration-300" 
                  :class="selectedCategoryId === cat.id ? 'scale-110' : ''" 
                  alt="Ikona sportu" 
                />
                
                <span class="text-sm font-inter font-bold tracking-wide uppercase">{{ cat.name }}</span>
              </button>

            </div>

            <!-- POLE TEKSTOWE DLA INNE -->
            <div v-if="selectedCategoryId === 'other'" class="mt-4 w-full">
              <input 
                type="text" 
                v-model="customCategoryName"
                placeholder="Wpisz nazwę (np. Kręgle, Poker)"
                class="w-full bg-[#2a2a2a] text-white font-inter rounded-xl p-4 outline-none border border-white/10 focus:border-[#F28100] focus:ring-1 focus:ring-[#F28100] transition-colors"
              />
            </div>
          </div>

          <!-- 3. ZWYCIĘZCA -->
          <div class="flex flex-col gap-2 w-full mt-2">
            <label class="text-gray-400 font-inter text-sm uppercase tracking-wider pl-1">Kto wygrał?</label>
            <div class="flex gap-4">
              
              <button 
                v-for="player in availablePlayers" 
                :key="player.id"
                @click="selectedWinnerId = player.id"
                :class="[
                  'flex-1 py-4 rounded-xl font-inter font-bold uppercase text-lg border-2 transition-all duration-300',
                  selectedWinnerId === player.id
                    ? (player.name === 'Adrian' ? 'border-blue-500 bg-blue-500/20 text-blue-400 shadow-[0_0_15px_rgba(59,130,246,0.4)] scale-105' : 'border-purple-500 bg-purple-500/20 text-purple-400 shadow-[0_0_15px_rgba(168,85,247,0.4)] scale-105')
                    : 'border-white/5 bg-[#2a2a2a] text-gray-500 hover:bg-white/5'
                ]"
              >
                {{ player.name }}
              </button>

            </div>
          </div>

          <!-- 4. GŁÓWNY PRZYCISK ZAPISU -->
          <button 
            @click="submitMatch"
            :disabled="!isFormValid"
            :class="[
              'mt-6 mb-6 w-full py-4 rounded-full font-bold uppercase tracking-widest transition-all duration-300',
              isFormValid 
                ? 'bg-[linear-gradient(90deg,_#F28100_0%,_#AA00E3_100%)] text-white shadow-[0_0_20px_rgba(242,129,0,0.4)] hover:shadow-[0_0_30px_rgba(242,129,0,0.6)] active:scale-95 cursor-pointer' 
                : 'bg-[linear-gradient(90deg,_#3f3f46_0%,_#18181b_100%)] text-gray-500 opacity-70 cursor-not-allowed shadow-none'
            ]"
          >
            Zapisz Wynik
          </button>

        </div>
        
      </div>
    </Transition>

    <nav class="absolute bottom-0 w-full h-[80px] translate-y-[2px] min-h-[80px] shrink-0 flex justify-center gap-40 items-center px-6 z-50">
          
          <!-- TŁO SVG: Rozciągnięte z tyłu, wyśrodkowane -->
        <img 
          src="./assets/hero_phone.svg" 
          class="absolute bottom-0 left-1/2 -translate-x-1/2 min-w-[1440px] max-w-none h-[80px] pointer-events-none" 
          alt="Tło nawigacji z dołkiem" 
        />

          <!-- DOMEK -->
          <button 
            @click="currentView = 'home'; isAddMode = false" 
            class="z-50 cursor-pointer transition-transform active:scale-90"
          >
            <img 
              :src="currentView === 'home' ? iconHomeFull : iconHomeLine" 
              class="w-10 h-10 transition-opacity duration-300" 
              alt="Home" 
            />
          </button>

          <!-- Główny przycisk PLUS (wypchnięty do góry, z gradientem) -->
          <!-- Kontener pozycjonujący z CIEPŁYM, PŁYNNIE ZMIENIAJĄCYM SIĘ CIENIEM -->
          <div class="absolute left-1/2 -top-11 -translate-x-1/2 w-[68px] h-[68px] z-50">
            <!-- Cień dla stanu domyślnego (Turkus/Fiolet) -->
            <div :class="['absolute inset-0 rounded-full shadow-[0_0_25px_rgba(168,85,247,0.5)] transition-opacity duration-500 ease-in-out', isAddMode ? 'opacity-0' : 'opacity-100']"></div>
            <!-- Cień dla stanu aktywnego (Pomarańcz/Czerwień) -->
            <div :class="['absolute inset-0 rounded-full shadow-[0_0_25px_rgba(255,0,55,0.6)] transition-opacity duration-500 ease-in-out', isAddMode ? 'opacity-100' : 'opacity-0']"></div>
            <!-- Właściwy, klikalny przycisk z gradientami -->
            <button 
              @click="isAddMode = !isAddMode"
              class="relative w-full h-full rounded-full flex items-center justify-center overflow-hidden active:scale-100 hover:scale-105 transition-transform duration-300 ease-in-out cursor-pointer"
            >
              <div :class="['absolute inset-0 bg-[radial-gradient(circle_at_30%_25%,_#00CEC4_0%,_#AA00E3_100%)] transition-opacity duration-500 ease-in-out', isAddMode ? 'opacity-0' : 'opacity-100']"></div>
              <div :class="['absolute inset-0 bg-[radial-gradient(circle_at_30%_25%,_#F28100_0%,_#AA00E3_100%)] transition-opacity duration-500 ease-in-out', isAddMode ? 'opacity-100' : 'opacity-0']"></div>

              <!-- Ikona (plus/krzyżyk) -->
              <img 
                src="./assets/plus.svg" 
                :class="['w-8 h-8 relative z-10 transition-transform duration-500 ease-in-out', isAddMode ? 'rotate-[135deg]' : 'rotate-0']" 
                alt="Dodaj" 
              />
            </button>
          </div>

          <!-- PRZYCISK PUCHARKA -->
          <button 
            @click="currentView = 'history'; isAddMode = false" 
            class="z-50 cursor-pointer transition-transform active:scale-90"
          >
            <img 
              :src="currentView === 'history' ? iconTrophyFull : iconTrophyLine" 
              class="w-10 h-10 transition-opacity duration-300" 
              alt="Historia" 
            />
          </button>

        </nav>
  </div>
</template>

<script setup>
  import { ref, computed, watch, nextTick, onMounted } from 'vue'
  import { createClient } from '@supabase/supabase-js'
  import confetti from 'canvas-confetti'
  import imgAdrian from './assets/adi.png'
  import imgMateusz from './assets/mati.png'
  import imgRemis from './assets/remis.png'
  import iconTenis from './assets/tennis.png'
  import iconStrzelnica from './assets/target.png'
  import iconKosci from './assets/dice.png'
  import iconInne from './assets/poker-cards.png'

  import iconHomeFull from './assets/house-chimney-full.svg'
  import iconHomeLine from './assets/house-chimney-line.svg'
  import iconTrophyFull from './assets/trophy-full.svg'
  import iconTrophyLine from './assets/trophy-line.svg'
  
  // --- WALIDACJA FORMULARZA ---
  const customCategoryName = ref('')
  
  const isFormValid = computed(() => {
    if (!newMatchDate.value || !selectedCategoryId.value || !selectedWinnerId.value) return false
    if (selectedCategoryId.value === 'other' && !customCategoryName.value.trim()) return false
    return true
  })

  // 'home' = strona główna z kafelkami, 'history' = tabela
  const allMatches = ref([])
  const currentView = ref('home')

  const supabaseUrl = 'https://tirgkibilgrguvlcbpdw.supabase.co'
  const supabaseKey = 'sb_publishable_tI9JshKjRc9LOFBOutsA5Q_eimty9qY'
  const supabase = createClient(supabaseUrl, supabaseKey)
    // 1. Zmienne, które będą trzymać nasze dane
  const periods = ref([])
  const selectedPeriodId = ref(null)
  const adrianWins = ref(0)
  const mateuszWins = ref(0)

  // Zmienna sterująca okienkiem dodawania
  const isAddMode = ref(false)

  // 2. Funkcja pobierająca okresy (Tura I, Tura II itd.)
  const fetchPeriods = async () => {
    const { data, error } = await supabase.from('periods').select('*')
    
    if (data) {
      periods.value = data
      // Ustawiamy domyślnie ten okres, który w bazie ma is_active = true
      const activePeriod = data.find(p => p.is_active)
      if (activePeriod) {
        selectedPeriodId.value = activePeriod.id
      } else if (data.length > 0) {
        // Fallback do ostatniego jeśli żaden nie jest aktywny
        selectedPeriodId.value = data[data.length - 1].id
      }
    }
  }

  // Funkcja dodawania nowej tury
  const createNewPeriod = async () => {
    const confirmCreate = confirm("Czy na pewno chcesz rozpocząć nową turę?")
    if (!confirmCreate) return
    
    const periodName = `Tura ${periods.value.length + 1}`

    // Najpierw dezaktywujemy stare tury, żeby nowa była jedyną aktywną
    if (periods.value.length > 0) {
      await supabase.from('periods').update({ is_active: false }).neq('id', 0)
    }

    // Dodajemy nową turę
    const { data, error } = await supabase
      .from('periods')
      .insert({ period_name: periodName.trim(), is_active: true })
      .select()
      .single()

    if (!error && data) {
      periods.value.push(data)
      selectedPeriodId.value = data.id
      // Odświeżamy punkty (będą 0:0 w nowej turze)
      fetchResults()
    } else {
      console.error("Błąd tworzenia tury:", error)
      alert("Wystąpił błąd podczas dodawania nowej tury.")
    }
  }

  // 3. Funkcja pobierająca mecze i licząca punkty
  const fetchResults = async () => {
    if (!selectedPeriodId.value) return

    const { data, error } = await supabase
      .from('match_results')
      .select(`
        id,
        players!winner_id (name)
      `)
      .eq('period_id', selectedPeriodId.value)

    if (data) {
      // Zerujemy liczniki
      adrianWins.value = 0
      mateuszWins.value = 0

      // Dodajemy punkty do odpowiedniego gracza
      data.forEach(match => {
        if (match.players.name === 'Adrian') adrianWins.value++
        if (match.players.name === 'Mateusz') mateuszWins.value++
      })
    }
  }

  // Funkcja pobierająca całą historię dla zakładki "Pucharek"
  const fetchHistory = async () => {
    const { data, error } = await supabase
      .from('match_results')
      .select('id, match_date, period_id, category_id, winner_id')
      .order('match_date', { ascending: false }) // Domyślne sortowanie od najnowszych w bazie

    if (data) {
      allMatches.value = data
    } else {
      console.error("Błąd pobierania historii:", error)
    }
  }

  // 4. Odpal pobieranie okresów od razu przy starcie strony
  onMounted(() => {
    fetchPeriods()
    fetchCategories()
    fetchHistory()
  })

  // 5. Jeśli zmienisz okres w rozwijanej liście, od razu przelicz punkty
  watch(selectedPeriodId, () => {
    fetchResults()
  })

  // Funkcja ustalająca kolory gradientów dla ramki i wypełnienia
  const getIndicatorStyles = (myWins, opponentWins) => {
    if (myWins > opponentWins) {
      // Wygrywa - Zielony
      return {
        border: 'bg-gradient-to-br from-green-700 to-green-950 shadow-[0_0_12px_rgba(34,197,94,0.25)]',
        fill: 'bg-gradient-to-br from-lime-400 to-green-500'
      }
    } else if (myWins < opponentWins) {
      // Przegrywa - Czerwony
      return {
        border: 'bg-gradient-to-br from-red-800 to-rose-950 shadow-[0_0_12px_rgba(244,63,94,0.25)]',
        fill: 'bg-gradient-to-br from-red-500 to-rose-500'
      }
    } else {
      // Remis - Żółty
      return {
        border: 'bg-gradient-to-br from-yellow-700 to-amber-950 shadow-[0_0_12px_rgba(234,179,8,0.25)]',
        fill: 'bg-gradient-to-br from-yellow-400 to-amber-500'
      }
    }
  }

  const winner = computed(() => {
    if (adrianWins.value > mateuszWins.value) return 'Adrian'
    if (mateuszWins.value > adrianWins.value) return 'Mateusz'
    return 'Remis'
  })


  // --- KONFETTI ---
  // --- KONFETTI ---
  // --- KONFETTI ---
  // --- KONFETTI ---
  // --- KONFETTI ---

  // REFERENCJA DO PŁÓTNA W HTML
  const confettiCanvas = ref(null) 
  let confettiInterval = null
  let myConfetti = null 

  // Wydzielamy sam strzał do małej funkcji
  const fireConfetti = () => {
    if (!myConfetti) return
    // Strzał z lewej
    myConfetti({
      particleCount: 25, // Zwiększyłem lekko ilość dla lepszego efektu przy rzadszym strzelaniu
      angle: 60,
      spread: 45,
      origin: { x: -0.1, y: 1 },
      colors: ['#00CEC4', '#AA00E3']
    })
    // Strzał z prawej
    myConfetti({
      particleCount: 25,
      angle: 120,
      spread: 45,
      origin: { x: 1.1, y: 1 },
      colors: ['#00CEC4', '#AA00E3']
    })
  }

  const startConfetti = async () => {
    if (confettiInterval) return
    
    await nextTick() 
    if (!confettiCanvas.value) return 

    myConfetti = confetti.create(confettiCanvas.value, {
      resize: true,
      useWorker: true
    })

    fireConfetti() // 🟢 ODPALAMY PIERWSZY STRZAŁ NATYCHMIAST
    confettiInterval = setInterval(fireConfetti, 2000) // 🟢 KOLEJNE STRZAŁY CO 2 SEKUNDY
  }

  const stopConfetti = () => {
    clearInterval(confettiInterval)
    confettiInterval = null
    if (myConfetti) {
      myConfetti.reset() 
    }
  }

  watch(winner, (newWinner) => {
    if (newWinner === 'Adrian' || newWinner === 'Mateusz') {
      startConfetti()
    } else {
      stopConfetti()
    }
  }, { immediate: true })

  // --- FORMULARZ DODAWANIA WYNIKU ---
  // --- FORMULARZ DODAWANIA WYNIKU ---
  // --- FORMULARZ DODAWANIA WYNIKU ---
  // --- FORMULARZ DODAWANIA WYNIKU ---
  // --- FORMULARZ DODAWANIA WYNIKU ---
  
  // Domyślna data to "dzisiaj" (format YYYY-MM-DD wymagany przez input date)
  const newMatchDate = ref(new Date().toISOString().split('T')[0])
  
  const selectedCategoryId = ref(null)
  const selectedWinnerId = ref(null)

  const availableCategories = ref([])

  const formCategories = computed(() => {
    const allowed = ['Tenis', 'Strzelnica', 'Kości', 'Inne']
    return availableCategories.value.filter(c => allowed.includes(c.name))
  })

  const fetchCategories = async () => {
    const { data, error } = await supabase.from('categories').select('*').order('id')
    if (data) {
      const iconMap = {
        'Tenis': iconTenis,
        'Strzelnica': iconStrzelnica,
        'Kości': iconKosci
      }
      
      const mapped = data.map(c => ({
        id: c.id,
        name: c.name,
        icon: iconMap[c.name] || iconInne
      }))
      
      mapped.push({ id: 'other', name: 'Inne', icon: iconInne })
      availableCategories.value = mapped
    }
  }

  const availablePlayers = ref([
    { id: 1, name: 'Adrian' },
    { id: 2, name: 'Mateusz' }
  ])

  // Funkcja wysyłająca dane do bazy
  const submitMatch = async () => {
    if (!isFormValid.value) {
      alert("Musisz wypełnić wszystkie pola!")
      return
    }

    let finalCategoryId = selectedCategoryId.value

    if (finalCategoryId === 'other') {
      const categoryName = customCategoryName.value.trim()
      const formattedName = categoryName.charAt(0).toUpperCase() + categoryName.slice(1)
      
      let { data: existingCat } = await supabase
        .from('categories')
        .select('id')
        .ilike('name', formattedName)
        .single()
        
      if (existingCat) {
        finalCategoryId = existingCat.id
      } else {
        const { data: newCat, error: insertErr } = await supabase
          .from('categories')
          .insert({ name: formattedName })
          .select('id')
          .single()
          
        if (insertErr) {
          console.error("Błąd tworzenia kategorii:", insertErr)
          alert("Wystąpił błąd podczas dodawania nowej kategorii.")
          return
        }
        finalCategoryId = newCat.id
      }
    }

    const { error } = await supabase
      .from('match_results')
      .insert({
        match_date: newMatchDate.value,
        period_id: selectedPeriodId.value, // Pobieramy automatycznie wybrany okres z głównej strony!
        category_id: finalCategoryId,
        winner_id: selectedWinnerId.value
      })

    if (!error) {
      // 1. Zamykamy okienko z płynną animacją
      isAddMode.value = false
      // 2. Resetujemy formularz na wypadek kolejnego dodawania
      selectedCategoryId.value = null
      selectedWinnerId.value = null
      customCategoryName.value = ''
      // 3. Odświeżamy punkty na głównej stronie, żeby od razu było widać nowy wynik!
      fetchResults()
      fetchCategories()
      fetchHistory() // Odświeżamy też historię, żeby w tabeli było widać nowy wynik
    } else {
      console.error("Błąd zapisu:", error)
    }
  }

  
  // --- TABELA WYNIKÓW ---
  // --- TABELA WYNIKÓW ---
  // --- TABELA WYNIKÓW ---
  // --- TABELA WYNIKÓW ---
  // --- TABELA WYNIKÓW ---


  // Zmienna sterująca widokiem tabeli
  const isTrophyView = ref(false)

  // --- FILTRY ---
  // Ustawienie na pusty string oznacza brak filtra ("Wszystkie")
  const filterPeriod = ref('')
  const filterCategory = ref('')
  const filterPlayer = ref('')

  // --- SORTOWANIE ---
  // Domyślnie sortujemy po dacie, Z->A (najnowsze na górze)
  const sortBy = ref('match_date')
  const sortDesc = ref(true) 

  // Funkcja odpalana po kliknięciu w nagłówek kolumny
  const handleSort = (column) => {
    if (sortBy.value === column) {
      // Jeśli klikamy w tę samą kolumnę, odwracamy kierunek
      sortDesc.value = !sortDesc.value
    } else {
      // Jeśli w nową, ustawiamy ją i domyślnie dajemy Z->A
      sortBy.value = column
      sortDesc.value = true 
    }
  }

  // --- SILNIK TABELI (Filtrowanie + Sortowanie) ---
  const processedResults = computed(() => {
    // UWAGA: Zmień `allMatches.value` na zmienną, w której trzymasz całą historię pobraną z Supabase!
    let data = [...allMatches.value] 

    // 1. Aplikowanie filtrów
    if (filterPeriod.value) data = data.filter(m => String(m.period_id) === String(filterPeriod.value))
    if (filterCategory.value) data = data.filter(m => String(m.category_id) === String(filterCategory.value))
    if (filterPlayer.value) data = data.filter(m => String(m.winner_id) === String(filterPlayer.value))

    // 2. Sortowanie bąbelkowe dostosowane do kierunku
    data.sort((a, b) => {
      let valA = a[sortBy.value]
      let valB = b[sortBy.value]

      // Zabezpieczenie dla stringów (np. imion), by sortować alfabetycznie bez względu na wielkość liter
      if (typeof valA === 'string') valA = valA.toLowerCase()
      if (typeof valB === 'string') valB = valB.toLowerCase()

      if (valA < valB) return sortDesc.value ? 1 : -1
      if (valA > valB) return sortDesc.value ? -1 : 1
      return 0
    })

    return data
  })

// --- TRYB EDYCJI I USUWANIA ---
// --- TRYB EDYCJI I USUWANIA ---
// --- TRYB EDYCJI I USUWANIA ---
// --- TRYB EDYCJI I USUWANIA ---
// --- TRYB EDYCJI I USUWANIA ---

  const isDeleteMode = ref(false)
  const selectedMatches = ref([]) // Tablica przechowująca ID wyników do usunięcia

  const toggleDeleteMode = () => {
    isDeleteMode.value = !isDeleteMode.value
    if (!isDeleteMode.value) {
      selectedMatches.value = [] // Resetujemy zaznaczenie, gdy wychodzimy z trybu
    }
  }
  
  // Funkcja usuwająca zaznaczone wyniki z bazy Supabase
  const deleteSelected = async () => {
    if (selectedMatches.value.length === 0) return
    
    // Proste potwierdzenie, żeby nie usunąć przez przypadek
    const confirmDelete = confirm(`Czy na pewno chcesz usunąć ${selectedMatches.value.length} wyników?`)
    if (!confirmDelete) return

    // Usuwamy wiele rekordów naraz, wyszukując po ich ID
    const { error } = await supabase
      .from('match_results')
      .delete()
      .in('id', selectedMatches.value)

    if (!error) {
      selectedMatches.value = [] // Czyścimy wybór
      isDeleteMode.value = false // Wyłączamy tryb
      
      // Odświeżamy punkty na głównej stronie i samą tabelę!
      fetchResults()
      fetchHistory()
    } else {
      console.error("Błąd usuwania:", error)
      alert("Wystąpił błąd podczas usuwania. Sprawdź logi.")
    }
  }

</script>

<style>
/* Ciemny, elegancki scrollbar dla całej aplikacji (pionowy i poziomy) */
::-webkit-scrollbar {
  width: 6px;  /* Szerokość dla pionowego */
  height: 6px; /* Wysokość dla poziomego (tabela) */
}

/* Tło pod scrollbarem (ścieżka) */
::-webkit-scrollbar-track {
  background: transparent; 
  border-radius: 10px;
  margin: 10px;
}

/* Sam suwak (uchwyt) */
::-webkit-scrollbar-thumb {
  background: #333333; 
  border-radius: 10px;
}

/* Kolor suwaka po najechaniu / dotknięciu */
::-webkit-scrollbar-thumb:hover {
  background: #555555; 
}

/* Krawędź rogu scrollbara (jeśli jest widoczny) */
::-webkit-scrollbar-corner {
  background: transparent;
}
</style>