<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>賽狗會 Doggg Club - 模擬馬會投注系統 (完美升級版)</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        hkjc: {
                            blue: '#003A8C',      
                            lightBlue: '#1890FF', 
                            gold: '#D4B106',      
                            lightGold: '#FADB14', 
                            dark: '#002140',      
                            gray: '#F0F2F5'       
                        }
                    }
                }
            }
        }
    </script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700;900&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Noto Sans TC', sans-serif; padding-bottom: 90px; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        /* 隱藏預設數字輸入框箭頭 */
        input[type=number]::-webkit-inner-spin-button, 
        input[type=number]::-webkit-outer-spin-button { -webkit-appearance: none; margin: 0; }
    </style>
</head>
<body class="bg-hkjc-gray min-h-screen">

    <!-- 頂部導航欄 -->
    <header class="bg-hkjc-blue text-white shadow-lg border-b-4 border-hkjc-gold sticky top-0 z-40">
        <div class="max-w-4xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <span class="text-3xl">🐕</span>
                <div>
                    <h1 class="text-xl font-black tracking-wider text-hkjc-lightGold">賽狗會</h1>
                    <p class="text-xs font-bold uppercase tracking-widest text-gray-300">Doggg Club</p>
                </div>
            </div>
            <!-- 當前選擇帳戶及餘額 -->
            <div class="flex flex-col items-end">
                <div class="flex items-center gap-1">
                    <span class="inline-block w-2 h-2 rounded-full bg-green-500 animate-pulse"></span>
                    <span class="text-[9px] text-gray-300">已連線</span>
                </div>
                <div class="flex items-center space-x-2 bg-hkjc-dark px-3 py-1 rounded-full border border-hkjc-gold/30">
                    <span id="header-user-name" class="font-bold text-sm text-white">讀取中...</span>
                    <span class="text-gray-400">|</span>
                    <span id="header-user-balance" class="font-black text-hkjc-lightGold text-sm">$0</span>
                </div>
            </div>
        </div>
    </header>

    <!-- 雲端同步加載遮罩 -->
    <div id="loading-overlay" class="fixed inset-0 bg-hkjc-dark/80 backdrop-blur-sm z-[60] flex flex-col items-center justify-center text-white">
        <span class="text-5xl animate-bounce mb-4">🐕</span>
        <p class="font-bold text-lg text-hkjc-lightGold">正在連接雲端賽狗大堂...</p>
        <p class="text-xs text-gray-400 mt-2">請稍候片刻</p>
    </div>

    <!-- 自定義 Toast 通知 -->
    <div id="toast" class="hidden fixed top-24 left-1/2 transform -translate-x-1/2 z-[70] bg-gray-900/95 text-white px-6 py-3 rounded-xl shadow-2xl flex items-center gap-3 transition-all duration-300 border border-hkjc-gold w-[90%] max-w-sm">
        <span id="toast-icon" class="text-xl">📢</span>
        <span id="toast-message" class="font-bold text-sm"></span>
    </div>

    <!-- 🌟 自訂通用確認對話框 (取代 confirm) -->
    <div id="confirm-modal" class="hidden fixed inset-0 bg-black/60 backdrop-blur-sm z-[80] flex items-center justify-center px-4 transition-opacity">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-sm overflow-hidden transform scale-100">
            <div class="p-5 text-center">
                <div id="confirm-icon" class="text-4xl mb-3">⚠️</div>
                <h3 id="confirm-title" class="font-black text-lg text-gray-800 mb-2"></h3>
                <p id="confirm-msg" class="text-sm text-gray-600 mb-6 leading-relaxed"></p>
                <div class="flex gap-3">
                    <button id="confirm-cancel" class="flex-1 py-2.5 rounded-xl font-bold text-gray-600 bg-gray-100 hover:bg-gray-200 transition">取消</button>
                    <button id="confirm-ok" class="flex-1 py-2.5 rounded-xl font-bold text-white transition">確定</button>
                </div>
            </div>
        </div>
    </div>

    <!-- 🌟 自訂玩家改名對話框 -->
    <div id="rename-modal" class="hidden fixed inset-0 bg-black/60 backdrop-blur-sm z-[80] flex items-center justify-center px-4">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-sm overflow-hidden">
            <div class="bg-hkjc-blue px-5 py-3 border-b-4 border-hkjc-gold flex justify-between items-center">
                <h3 class="font-black text-white">✏️ 更改玩家名稱</h3>
            </div>
            <div class="p-5">
                <label class="block text-sm font-bold text-gray-700 mb-2">請輸入新名稱：</label>
                <input id="rename-input" type="text" class="w-full border-2 border-gray-300 rounded-xl px-4 py-2.5 focus:border-hkjc-blue focus:outline-none mb-5 font-bold text-gray-800">
                <div class="flex gap-3">
                    <button onclick="closeRenameModal()" class="flex-1 py-2.5 rounded-xl font-bold text-gray-600 bg-gray-100 hover:bg-gray-200">取消</button>
                    <button onclick="executeRename()" class="flex-1 py-2.5 rounded-xl font-bold text-white bg-hkjc-blue hover:bg-hkjc-dark">儲存變更</button>
                </div>
            </div>
        </div>
    </div>

    <!-- 🌟 自訂編輯賽事對話框 -->
    <div id="edit-match-modal" class="hidden fixed inset-0 bg-black/60 backdrop-blur-sm z-[80] flex items-center justify-center px-4">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-lg max-h-[90vh] overflow-y-auto">
            <div class="bg-hkjc-dark px-5 py-3 border-b-4 border-hkjc-gold sticky top-0 z-10 flex justify-between items-center">
                <h3 class="font-black text-white">✏️ 編輯賽事</h3>
                <button onclick="closeEditMatchModal()" class="text-white hover:text-red-400">✖</button>
            </div>
            <div class="p-5 space-y-4">
                <div class="bg-red-50 text-red-600 text-[11px] p-2 rounded-lg mb-2">
                    💡 提示：若刪除已有玩家投注的選項，該選項的投注本金將自動全額退還給玩家。
                </div>
                <div>
                    <label class="block text-sm font-bold text-gray-700 mb-1">修改賽事名稱</label>
                    <input id="edit-match-title" type="text" class="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:border-hkjc-blue focus:outline-none font-bold">
                </div>
                <div>
                    <label class="block text-sm font-bold text-gray-700 mb-2">修改選項及賠率</label>
                    <div id="edit-options-container" class="space-y-2">
                        <!-- 動態載入 -->
                    </div>
                    <button onclick="addEditOptionRow()" class="mt-3 text-xs font-bold text-hkjc-lightBlue hover:text-hkjc-blue flex items-center gap-1">
                        ➕ 增加選項
                    </button>
                </div>
                <div class="pt-4 border-t mt-4 flex gap-3">
                    <button onclick="closeEditMatchModal()" class="flex-1 py-2.5 rounded-xl font-bold text-gray-600 bg-gray-100">取消</button>
                    <button onclick="saveEditedMatch()" class="flex-1 py-2.5 rounded-xl font-bold text-white bg-green-600 hover:bg-green-700">💾 儲存修改</button>
                </div>
            </div>
        </div>
    </div>

    <!-- 🌟 自訂修改資金對話框 -->
    <div id="edit-balance-modal" class="hidden fixed inset-0 bg-black/60 backdrop-blur-sm z-[80] flex items-center justify-center px-4">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-sm overflow-hidden">
            <div class="bg-hkjc-blue px-5 py-3 border-b-4 border-hkjc-gold flex justify-between items-center">
                <h3 class="font-black text-white">💰 修改玩家資金</h3>
            </div>
            <div class="p-5">
                <label id="edit-balance-label" class="block text-sm font-bold text-gray-700 mb-2">修改資金：</label>
                <input id="edit-balance-input" type="number" class="w-full border-2 border-gray-300 rounded-xl px-4 py-2.5 focus:border-hkjc-blue focus:outline-none mb-5 font-bold text-gray-800">
                <div class="flex gap-3">
                    <button onclick="closeEditBalanceModal()" class="flex-1 py-2.5 rounded-xl font-bold text-gray-600 bg-gray-100 hover:bg-gray-200 transition">取消</button>
                    <button onclick="executeEditBalance()" class="flex-1 py-2.5 rounded-xl font-bold text-white bg-hkjc-blue hover:bg-hkjc-dark transition">儲存變更</button>
                </div>
            </div>
        </div>
    </div>

    <!-- 主內容區 -->
    <main class="max-w-4xl mx-auto px-4 mt-5">
        
        <!-- 全域帳戶管理 -->
        <section class="bg-white rounded-xl shadow-md p-4 mb-6 border-t-4 border-hkjc-blue">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3">
                <div>
                    <h2 class="text-sm font-bold text-gray-700 flex items-center gap-1">
                        <span>👤</span> 點擊切換為你的帳戶：
                    </h2>
                    <p class="text-[10px] text-gray-400 mt-0.5">※ 點擊你的名字旁邊的「✏️」可以隨時改名</p>
                </div>
                <div class="flex gap-2">
                    <input id="new-username-input" type="text" placeholder="輸入新朋友名字" class="border border-gray-300 rounded-lg px-3 py-1.5 text-sm focus:outline-none focus:ring-2 focus:ring-hkjc-blue w-full sm:w-auto">
                    <button onclick="addNewUser()" class="bg-hkjc-blue hover:bg-hkjc-dark text-white text-sm font-bold px-4 py-1.5 rounded-lg transition shadow shrink-0">
                        新增
                    </button>
                </div>
            </div>
            <div id="users-list-container" class="flex gap-2 mt-3 overflow-x-auto pb-2 no-scrollbar items-center">
                <!-- 動態載入 -->
            </div>
        </section>

        <!-- ==================== 分頁 1: 投注面板 ==================== -->
        <section id="section-betting" class="space-y-6 block">
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-hkjc-blue">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-lg font-bold text-hkjc-blue flex items-center gap-2">
                        <span>🎟️</span> 進行中的賽事
                    </h2>
                    <span class="bg-red-100 text-red-700 text-xs font-bold px-2.5 py-1 rounded-full animate-pulse">即時受注中</span>
                </div>
                <div id="active-matches-container" class="space-y-4">
                    <!-- 動態載入 -->
                </div>
            </div>
        </section>

        <!-- ==================== 分頁 2: 記錄與排行榜 ==================== -->
        <section id="section-records" class="space-y-6 hidden">
            <!-- 排行榜 -->
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-hkjc-gold relative overflow-hidden">
                <div class="absolute -right-6 -top-6 text-7xl text-yellow-100 opacity-40 select-none">🏆</div>
                <h2 class="text-lg font-bold text-hkjc-blue flex items-center gap-2 mb-4 relative z-10">
                    <span>🏆</span> 財富排行榜
                </h2>
                <div id="leaderboard-container" class="space-y-3 relative z-10">
                    <!-- 動態載入 -->
                </div>
            </div>
            <!-- 投注紀錄 -->
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-gray-400 overflow-hidden">
                <h2 class="text-lg font-bold text-gray-800 flex items-center gap-2 mb-4">
                    <span>📋</span> 所有投注歷史紀錄
                </h2>
                <div class="overflow-x-auto -mx-5 px-5">
                    <table class="min-w-full text-left text-sm whitespace-nowrap">
                        <thead class="bg-gray-100 text-gray-700 font-bold">
                            <tr>
                                <th class="p-3">玩家</th>
                                <th class="p-3">賽事 / 選項</th>
                                <th class="p-3 text-right">金額</th>
                                <th class="p-3 text-center">狀態</th>
                            </tr>
                        </thead>
                        <tbody id="bets-history-tbody">
                            <!-- 動態載入 -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- ==================== 分頁 3: 莊家管理 ==================== -->
        <section id="section-admin" class="space-y-6 hidden">
            
            <!-- 賽事管理與結算中心 -->
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-green-500">
                <h2 class="text-lg font-bold text-green-700 flex items-center gap-2 mb-3">
                    <span>🏁</span> 賽事管理與結算中心
                </h2>
                <p class="text-xs text-gray-500 mb-4">莊家可於此處編輯賽事、刪除賽事，或在比賽結束後選擇獲勝選項並派彩。</p>
                <div id="admin-settle-container" class="space-y-4">
                    <!-- 動態載入 -->
                </div>
            </div>

            <!-- 開設新賽事 -->
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-hkjc-blue">
                <h2 class="text-lg font-bold text-hkjc-blue flex items-center gap-2 mb-4">
                    <span>🎯</span> 開設新賽事
                </h2>
                <div class="space-y-4">
                    <div>
                        <label class="block text-sm font-bold text-gray-700 mb-1">賽事名稱</label>
                        <input id="match-title-input" type="text" placeholder="例：聽日邊個係書童？" class="w-full border border-gray-300 rounded-lg px-4 py-2 text-sm focus:ring-2 focus:ring-hkjc-blue focus:outline-none">
                    </div>
                    <div>
                        <label class="block text-sm font-bold text-gray-700 mb-2">投注選項及賠率</label>
                        <div id="options-inputs-container" class="space-y-2">
                            <div class="flex items-center gap-2 create-option-row">
                                <span class="text-xs font-bold text-gray-400 w-4">A</span>
                                <input type="text" placeholder="會" class="flex-grow border border-gray-300 rounded-lg px-2 py-1.5 text-sm focus:ring-1 focus:ring-hkjc-blue focus:outline-none opt-name">
                                <input type="number" step="0.1" min="1.01" placeholder="1.8" class="w-20 border border-gray-300 rounded-lg px-2 py-1.5 text-sm focus:ring-1 focus:ring-hkjc-blue focus:outline-none opt-odds">
                                <button onclick="removeCreateOptionRow(this)" class="text-red-400 hover:text-red-600 p-1">❌</button>
                            </div>
                            <div class="flex items-center gap-2 create-option-row">
                                <span class="text-xs font-bold text-gray-400 w-4">B</span>
                                <input type="text" placeholder="不會" class="flex-grow border border-gray-300 rounded-lg px-2 py-1.5 text-sm focus:ring-1 focus:ring-hkjc-blue focus:outline-none opt-name">
                                <input type="number" step="0.1" min="1.01" placeholder="2.2" class="w-20 border border-gray-300 rounded-lg px-2 py-1.5 text-sm focus:ring-1 focus:ring-hkjc-blue focus:outline-none opt-odds">
                                <button onclick="removeCreateOptionRow(this)" class="text-red-400 hover:text-red-600 p-1">❌</button>
                            </div>
                        </div>
                        <button onclick="addCreateOptionRow()" class="mt-2 text-xs font-bold text-hkjc-lightBlue hover:text-hkjc-blue flex items-center gap-1">
                            ➕ 增加選項
                        </button>
                    </div>
                    <button onclick="createMatch()" class="w-full bg-hkjc-blue hover:bg-hkjc-dark text-white font-bold py-2.5 rounded-lg shadow-md transition">
                        📢 發布賽事、開放投注！
                    </button>
                </div>
            </div>

            <!-- 用戶管理 -->
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-purple-500">
                <h2 class="text-sm font-bold text-purple-700 flex items-center gap-2 mb-3">
                    <span>⚙️</span> 成員與資金管理
                </h2>
                <div id="admin-users-list" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-3">
                    <!-- 動態載入 -->
                </div>
                <button onclick="requestResetData()" class="mt-5 w-full bg-red-600 hover:bg-red-700 text-white font-bold py-2.5 rounded-lg text-xs transition shadow-md">
                    🚨 一鍵恢復全體預設數據（重新洗牌）
                </button>
            </div>
        </section>

    </main>

    <!-- 底部固定導航列 -->
    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t border-gray-200 shadow-[0_-5px_15px_rgba(0,0,0,0.05)] z-40 pb-safe">
        <div class="max-w-4xl mx-auto flex">
            <button onclick="switchTab('betting')" id="nav-betting" class="flex-1 py-3 flex flex-col items-center justify-center gap-1 border-t-4 border-hkjc-blue bg-blue-50 text-hkjc-blue transition-all">
                <span class="text-xl leading-none">🎟️</span>
                <span class="text-[11px] font-bold">投注面板</span>
            </button>
            <button onclick="switchTab('records')" id="nav-records" class="flex-1 py-3 flex flex-col items-center justify-center gap-1 border-t-4 border-transparent text-gray-400 hover:bg-gray-50 transition-all">
                <span class="text-xl leading-none">🏆</span>
                <span class="text-[11px] font-bold">記錄與排行榜</span>
            </button>
            <button onclick="switchTab('admin')" id="nav-admin" class="flex-1 py-3 flex flex-col items-center justify-center gap-1 border-t-4 border-transparent text-gray-400 hover:bg-gray-50 transition-all">
                <span class="text-xl leading-none">⚙️</span>
                <span class="text-[11px] font-bold">莊家管理</span>
            </button>
        </div>
    </nav>

    <!-- 載入 Firebase SDK (已注入您的 Firebase 配置) -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // 這是你專屬的 Firebase 配置
        const firebaseConfig = {
            apiKey: "AIzaSyBGOXw8BMq9DKe6Q_iBCCY0vHWfT-GoypA",
            authDomain: "doggyclub-8130c.firebaseapp.com",
            projectId: "doggyclub-8130c",
            storageBucket: "doggyclub-8130c.firebasestorage.app",
            messagingSenderId: "106666978506",
            appId: "1:106666978506:web:37ab6519b6320997141833",
            measurementId: "G-01KRRZLRVP"
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        const DEFAULT_USERS = ['泰臣', '辣椒', '秋葵', '薯片', '蘇媚', '水晶', '牛屎', '西瓜', '騾仔'];
        const DEFAULT_STATE = {
            users: DEFAULT_USERS.map(name => ({ name: name, balance: 1000 })),
            matches: [
                {
                    id: 'm1', title: '🐕 第一場：賽狗總決賽 (黃金組)',
                    options: [
                        { id: 'o1', name: '閃電哈士奇', odds: 1.8 },
                        { id: 'o2', name: '旋風柴犬', odds: 2.5 },
                        { id: 'o3', name: '火箭吉娃娃', odds: 4.2 }
                    ],
                    settled: false, winner: null
                }
            ],
            bets: []
        };

        // 定義資料庫文檔路徑 (獨立專屬的路徑)
        const roomDocRef = doc(db, 'doggyclub', 'roomState');

        let state = { users: [], currentUser: null, matches: [], bets: [] };
        let isInitialized = false;

        // 全局狀態管理物件 (暴露給非 module script 使用)
        window.GameSystem = {
            getState: () => state,
            updateCloud: async () => {
                if (!auth.currentUser) return;
                try {
                    await setDoc(roomDocRef, { users: state.users, matches: state.matches, bets: state.bets });
                } catch (err) {
                    console.error("雲端儲存失敗:", err);
                    window.showToast("雲端儲存失敗，請重試", "❌");
                }
            },
            resetData: async () => {
                await setDoc(roomDocRef, DEFAULT_STATE);
            }
        };

        const initCloudConnection = async () => {
            try {
                // 自動使用匿名登入
                await signInAnonymously(auth);
            } catch (err) {
                console.error("連接雲端失敗:", err);
                document.getElementById('loading-overlay').innerHTML = `
                    <span class="text-5xl mb-4">❌</span>
                    <p class="font-bold text-lg text-red-500">無法連接數據庫</p>
                    <p class="text-xs text-gray-400 mt-2">請確保你已在 Firebase 開啟「匿名登入」與 Firestore 資料庫</p>
                `;
            }
        };

        onAuthStateChanged(auth, (firebaseUser) => {
            if (firebaseUser) {
                onSnapshot(roomDocRef, async (snapshot) => {
                    if (snapshot.exists()) {
                        const cloudData = snapshot.data();
                        state.users = cloudData.users || [];
                        state.matches = cloudData.matches || [];
                        state.bets = cloudData.bets || [];
                        
                        if (!isInitialized && state.users.length > 0) {
                            const lastUser = localStorage.getItem('doggg_last_active_user');
                            if (lastUser && state.users.some(u => u.name === lastUser)) {
                                state.currentUser = lastUser;
                            } else {
                                state.currentUser = state.users[0].name;
                            }
                            isInitialized = true;
                            document.getElementById('loading-overlay').classList.add('hidden');
                        }

                        if (state.currentUser && !state.users.some(u => u.name === state.currentUser)) {
                            state.currentUser = state.users[0]?.name || null;
                        }

                        window.renderAll();
                    } else {
                        await setDoc(roomDocRef, DEFAULT_STATE);
                    }
                }, (error) => {
                    console.error("監聽資料失敗:", error);
                });
            }
        });

        initCloudConnection();
    </script>

    <!-- 交互邏輯 -->
    <script>
        // 🌟 通用組件：Toast 與 Confirm Modal
        window.showToast = function(message, icon = '📢') {
            const toast = document.getElementById('toast');
            document.getElementById('toast-message').textContent = message;
            document.getElementById('toast-icon').textContent = icon;
            toast.classList.remove('hidden');
            toast.classList.add('flex');
            setTimeout(() => { toast.classList.add('hidden'); toast.classList.remove('flex'); }, 3000);
        };

        let pendingConfirmAction = null;
        window.showConfirmModal = function(title, msg, onConfirm, okColorClass = 'bg-red-600 hover:bg-red-700', icon = '⚠️') {
            document.getElementById('confirm-title').textContent = title;
            document.getElementById('confirm-msg').textContent = msg;
            document.getElementById('confirm-icon').textContent = icon;
            
            const okBtn = document.getElementById('confirm-ok');
            okBtn.className = `flex-1 py-2.5 rounded-xl font-bold text-white transition ${okColorClass}`;
            
            pendingConfirmAction = onConfirm;
            document.getElementById('confirm-modal').classList.remove('hidden');
        };

        document.getElementById('confirm-cancel').onclick = () => { document.getElementById('confirm-modal').classList.add('hidden'); };
        document.getElementById('confirm-ok').onclick = () => {
            document.getElementById('confirm-modal').classList.add('hidden');
            if (pendingConfirmAction) pendingConfirmAction();
        };

        // 🌟 主渲染邏輯
        window.renderAll = function() {
            if (!window.GameSystem) return;
            renderHeader();
            renderUsersList();
            renderActiveMatches();
            renderBetsHistory();
            renderLeaderboard();
            renderAdminView();
        };

        function renderHeader() {
            const state = window.GameSystem.getState();
            const userObj = state.users.find(u => u.name === state.currentUser);
            if (userObj) {
                document.getElementById('header-user-name').textContent = userObj.name;
                document.getElementById('header-user-balance').textContent = `$${userObj.balance.toLocaleString()}`;
            } else {
                document.getElementById('header-user-name').textContent = '未選擇';
                document.getElementById('header-user-balance').textContent = '$0';
            }
        }

        function renderUsersList() {
            const state = window.GameSystem.getState();
            const container = document.getElementById('users-list-container');
            container.innerHTML = '';
            const sortedUsers = [...state.users].sort((a, b) => b.balance - a.balance);

            state.users.forEach(user => {
                const isActive = user.name === state.currentUser;
                const isLeader = sortedUsers.length > 0 && sortedUsers[0].name === user.name;
                
                const btnWrapper = document.createElement('div');
                btnWrapper.className = `flex-shrink-0 flex items-center px-3 py-1.5 rounded-xl transition shadow-sm border ${
                    isActive ? 'bg-hkjc-blue text-white border-hkjc-gold' : 'bg-white text-gray-700 border-gray-200 hover:bg-gray-50'
                }`;

                // 點擊切換用戶
                const nameBtn = document.createElement('button');
                nameBtn.className = "font-bold text-sm focus:outline-none flex items-center gap-1";
                nameBtn.innerHTML = `${isActive ? '🐕 ' : ''}${user.name}${isLeader ? ' 👑' : ''}`;
                nameBtn.onclick = () => selectUser(user.name);
                btnWrapper.appendChild(nameBtn);

                // 如果是當前用戶，顯示「改名」按鈕
                if (isActive) {
                    const editBtn = document.createElement('button');
                    editBtn.className = "ml-2 p-1 text-hkjc-lightGold hover:text-white rounded-md transition";
                    editBtn.innerHTML = "✏️";
                    editBtn.onclick = (e) => { e.stopPropagation(); openRenameModal(user.name); };
                    btnWrapper.appendChild(editBtn);
                }

                container.appendChild(btnWrapper);
            });
        }

        function selectUser(name) {
            const state = window.GameSystem.getState();
            state.currentUser = name;
            localStorage.setItem('doggg_last_active_user', name);
            window.renderAll();
        }

        window.addNewUser = async function() {
            const input = document.getElementById('new-username-input');
            const name = input.value.trim();
            const state = window.GameSystem.getState();

            if (!name) return showToast('請輸入名字！', '⚠️');
            if (state.users.some(u => u.name === name)) return showToast('此人已在大名單中！', '⚠️');

            state.users.push({ name: name, balance: 1000 });
            state.currentUser = name;
            localStorage.setItem('doggg_last_active_user', name);
            
            input.value = '';
            showToast(`成功建立玩家「${name}」！`, '🎉');
            await window.GameSystem.updateCloud();
        };

        // 🌟 玩家改名邏輯
        let renamingOldName = null;
        window.openRenameModal = function(oldName) {
            renamingOldName = oldName;
            document.getElementById('rename-input').value = oldName;
            document.getElementById('rename-modal').classList.remove('hidden');
        };

        window.closeRenameModal = function() {
            document.getElementById('rename-modal').classList.add('hidden');
            renamingOldName = null;
        };

        window.executeRename = async function() {
            const newName = document.getElementById('rename-input').value.trim();
            const state = window.GameSystem.getState();

            if (!newName) return showToast('名稱不能為空！', '⚠️');
            if (newName === renamingOldName) return closeRenameModal();
            if (state.users.some(u => u.name === newName)) return showToast('這個名字已經有人使用了！', '⚠️');

            // 1. 更新 Users 列表
            const userObj = state.users.find(u => u.name === renamingOldName);
            if (userObj) userObj.name = newName;

            // 2. 更新歷史投注紀錄中的名稱
            state.bets.forEach(b => {
                if (b.username === renamingOldName) b.username = newName;
            });

            // 3. 更新目前登入狀態
            if (state.currentUser === renamingOldName) {
                state.currentUser = newName;
                localStorage.setItem('doggg_last_active_user', newName);
            }

            closeRenameModal();
            showToast(`已成功改名為「${newName}」！`, '✨');
            await window.GameSystem.updateCloud();
        };

        // 🌟 賽事與投注邏輯
        function renderActiveMatches() {
            const state = window.GameSystem.getState();
            const container = document.getElementById('active-matches-container');
            container.innerHTML = '';
            const activeMatches = state.matches.filter(m => !m.settled);

            if (activeMatches.length === 0) {
                container.innerHTML = `<div class="text-center py-6 text-gray-400 font-bold">目前沒有賽事。請等候莊家開盤！</div>`;
                return;
            }

            activeMatches.forEach(match => {
                const card = document.createElement('div');
                card.className = 'border border-gray-200 rounded-xl bg-gray-50 p-4';
                
                let optionsHtml = match.options.map(opt => `
                    <label class="flex items-center justify-between bg-white border border-gray-200 rounded-lg p-2.5 hover:bg-blue-50 cursor-pointer transition">
                        <div class="flex items-center gap-2">
                            <input type="radio" name="match-${match.id}" value="${opt.id}" class="w-4 h-4 text-hkjc-blue">
                            <span class="font-bold text-sm text-gray-800">${opt.name}</span>
                        </div>
                        <span class="bg-hkjc-gold/20 text-hkjc-blue text-xs font-black px-2 py-1 rounded border border-hkjc-gold/40">
                            @${opt.odds.toFixed(2)}
                        </span>
                    </label>
                `).join('');

                card.innerHTML = `
                    <div class="font-black text-gray-800 text-sm mb-3 border-b pb-2">${match.title}</div>
                    <div class="space-y-2 mb-3">${optionsHtml}</div>
                    <div class="flex gap-2">
                        <input id="bet-amount-${match.id}" type="number" min="1" placeholder="投注金額" class="w-full border border-gray-300 rounded-lg px-3 text-sm focus:outline-none focus:ring-2 focus:ring-hkjc-blue font-bold">
                        <button onclick="placeBet('${match.id}')" class="bg-hkjc-blue hover:bg-hkjc-dark text-white font-bold px-4 py-2 rounded-lg text-sm shrink-0 shadow transition">
                            投注 🎟️
                        </button>
                    </div>
                `;
                container.appendChild(card);
            });
        }

        window.placeBet = async function(matchId) {
            const state = window.GameSystem.getState();
            const userObj = state.users.find(u => u.name === state.currentUser);
            if (!userObj) return showToast('請先選擇你的名字帳戶！', '⚠️');

            const match = state.matches.find(m => m.id === matchId);
            const selectedRadio = document.querySelector(`input[name="match-${matchId}"]:checked`);
            if (!selectedRadio) return showToast('請選好下注的選項！', '⚠️');
            
            const option = match.options.find(o => o.id === selectedRadio.value);
            const amountInput = document.getElementById(`bet-amount-${matchId}`);
            const amount = parseInt(amountInput.value);

            if (isNaN(amount) || amount <= 0) return showToast('請輸入合理的整數下注金額！', '⚠️');
            if (userObj.balance < amount) return showToast('你的模擬本金不足！', '❌');

            userObj.balance -= amount;
            state.bets.unshift({
                id: 'b_' + Date.now() + '_' + Math.random().toString(36).substr(2, 4),
                username: userObj.name,
                matchId: match.id,
                matchTitle: match.title,
                optionId: option.id,
                optionName: option.name,
                odds: option.odds,
                amount: amount,
                status: 'pending',
                payout: 0
            });

            amountInput.value = '';
            showToast(`下注成功！已扣除 $${amount.toLocaleString()}`, '✅');
            await window.GameSystem.updateCloud();
        };

        function renderLeaderboard() {
            const state = window.GameSystem.getState();
            const container = document.getElementById('leaderboard-container');
            container.innerHTML = '';
            const sortedUsers = [...state.users].sort((a, b) => b.balance - a.balance);

            sortedUsers.forEach((user, index) => {
                const isWinner = index === 0;
                let rankStyle = index === 0 ? "bg-hkjc-gold text-hkjc-dark" : (index === 1 ? "bg-gray-300" : (index === 2 ? "bg-amber-600 text-white" : "bg-gray-100 text-gray-500"));
                let icon = index === 0 ? "🥇 犬神" : `No.${index + 1}`;

                container.innerHTML += `
                    <div class="flex items-center justify-between p-3 rounded-lg border ${isWinner ? 'bg-amber-50 border-hkjc-gold' : 'bg-white border-gray-100'}">
                        <div class="flex items-center gap-3">
                            <span class="text-xs font-bold px-2 py-1 rounded ${rankStyle}">${icon}</span>
                            <span class="font-bold text-sm text-gray-800">${user.name}</span>
                        </div>
                        <span class="font-black text-hkjc-blue text-sm">$${user.balance.toLocaleString()}</span>
                    </div>
                `;
            });
        }

        function renderBetsHistory() {
            const state = window.GameSystem.getState();
            const tbody = document.getElementById('bets-history-tbody');
            tbody.innerHTML = '';
            if (state.bets.length === 0) return tbody.innerHTML = `<tr><td colspan="4" class="p-4 text-center text-gray-400">目前沒有任何投注紀錄</td></tr>`;

            state.bets.forEach(bet => {
                let statusHtml = bet.status === 'pending' ? `<span class="text-gray-500 text-xs font-bold bg-gray-100 px-2 py-1 rounded">待結算</span>` : 
                                 (bet.status === 'won' ? `<span class="text-green-600 text-xs font-bold bg-green-100 px-2 py-1 rounded">中獎 (+$${bet.payout.toLocaleString()})</span>` : 
                                 `<span class="text-red-500 text-xs font-bold bg-red-50 px-2 py-1 rounded">未中</span>`);
                tbody.innerHTML += `
                    <tr class="border-b border-gray-100">
                        <td class="p-3 font-bold text-gray-800">${bet.username}</td>
                        <td class="p-3 text-xs"><div class="text-gray-500 truncate w-32 md:w-auto">${bet.matchTitle}</div><div class="text-hkjc-blue font-bold">${bet.optionName} <span class="text-gray-400">@${bet.odds}</span></div></td>
                        <td class="p-3 font-bold text-right text-gray-700">$${bet.amount.toLocaleString()}</td>
                        <td class="p-3 text-center">${statusHtml}</td>
                    </tr>
                `;
            });
        }

        // 🌟 莊家管理視圖
        function renderAdminView() {
            const state = window.GameSystem.getState();
            const settleContainer = document.getElementById('admin-settle-container');
            settleContainer.innerHTML = '';
            const pendingMatches = state.matches.filter(m => !m.settled);

            if (pendingMatches.length === 0) {
                settleContainer.innerHTML = `<div class="text-center py-4 text-gray-400 text-sm">目前無待結算賽事。</div>`;
            } else {
                pendingMatches.forEach(match => {
                    let opts = match.options.map(opt => `
                        <label class="flex items-center gap-2 bg-white px-2 py-1.5 border rounded cursor-pointer">
                            <input type="radio" name="settle-${match.id}" value="${opt.id}">
                            <span class="text-sm font-bold text-gray-700">${opt.name} (${opt.odds.toFixed(2)})</span>
                        </label>
                    `).join('');

                    settleContainer.innerHTML += `
                        <div class="p-4 border rounded-xl bg-gray-50 border-gray-200">
                            <div class="font-bold text-sm mb-3 text-gray-800 flex justify-between items-start">
                                <span>${match.title}</span>
                                <div class="flex gap-1 shrink-0">
                                    <button onclick="openEditMatchModal('${match.id}')" class="text-xs bg-hkjc-lightBlue text-white px-2 py-1 rounded hover:bg-hkjc-blue">✏️ 編輯</button>
                                    <button onclick="requestDeleteMatch('${match.id}')" class="text-xs bg-red-500 text-white px-2 py-1 rounded hover:bg-red-600">🗑️ 刪除</button>
                                </div>
                            </div>
                            <div class="space-y-2 mb-3">${opts}</div>
                            <button onclick="settleMatch('${match.id}')" class="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-2 rounded-lg text-sm transition shadow">
                                🏆 結算派彩
                            </button>
                        </div>
                    `;
                });
            }

            const usersList = document.getElementById('admin-users-list');
            usersList.innerHTML = '';
            state.users.forEach(user => {
                usersList.innerHTML += `
                    <div class="flex flex-col bg-gray-50 border border-gray-200 p-3 rounded-lg gap-2 shadow-sm">
                        <div class="flex justify-between items-center">
                            <span class="text-sm font-bold text-gray-800">${user.name}</span>
                            <span class="text-sm font-black text-hkjc-blue">$${user.balance.toLocaleString()}</span>
                        </div>
                        <div class="flex gap-2 mt-1">
                            <button onclick="openEditBalanceModal('${user.name}', ${user.balance})" class="flex-1 text-hkjc-blue bg-blue-50 border border-blue-200 text-xs font-bold py-1.5 rounded hover:bg-hkjc-blue hover:text-white transition shadow-sm">修改資金</button>
                            <button onclick="requestDeleteUser('${user.name}')" class="flex-1 text-red-500 bg-red-50 border border-red-200 text-xs font-bold py-1.5 rounded hover:bg-red-500 hover:text-white transition shadow-sm">刪除</button>
                        </div>
                    </div>
                `;
            });
        }

        // --- 賽事編輯與刪除邏輯 ---
        window.requestDeleteMatch = function(matchId) {
            const state = window.GameSystem.getState();
            const match = state.matches.find(m => m.id === matchId);
            showConfirmModal(
                "刪除賽事", 
                `確定要刪除「${match.title}」嗎？\n\n注意：如果已有玩家下注這場比賽，他們的本金將會自動全額退還。`,
                async () => {
                    // 退還本金給已下注的玩家
                    const betsToRefund = state.bets.filter(b => b.matchId === matchId && b.status === 'pending');
                    betsToRefund.forEach(b => {
                        const u = state.users.find(user => user.name === b.username);
                        if (u) u.balance += b.amount;
                    });
                    // 移除投注紀錄與賽事
                    state.bets = state.bets.filter(b => !(b.matchId === matchId && b.status === 'pending'));
                    state.matches = state.matches.filter(m => m.id !== matchId);
                    
                    showToast("已刪除賽事並退還相關本金！", "🗑️");
                    await window.GameSystem.updateCloud();
                }
            );
        };

        let editingMatchId = null;
        window.openEditMatchModal = function(matchId) {
            const state = window.GameSystem.getState();
            editingMatchId = matchId;
            const match = state.matches.find(m => m.id === matchId);
            
            document.getElementById('edit-match-title').value = match.title.replace('🐕 ', '');
            
            const container = document.getElementById('edit-options-container');
            container.innerHTML = '';
            match.options.forEach((opt, index) => {
                const label = String.fromCharCode(65 + index);
                container.insertAdjacentHTML('beforeend', `
                    <div class="flex items-center gap-2 edit-option-row">
                        <input type="hidden" class="edit-opt-id" value="${opt.id}">
                        <span class="text-xs font-bold text-gray-400 w-4">${label}</span>
                        <input type="text" value="${opt.name}" class="flex-grow border border-gray-300 rounded-lg px-2 py-1.5 text-sm focus:border-hkjc-blue edit-opt-name">
                        <input type="number" step="0.1" min="1.01" value="${opt.odds}" class="w-20 border border-gray-300 rounded-lg px-2 py-1.5 text-sm focus:border-hkjc-blue edit-opt-odds">
                        <button onclick="removeEditOptionRow(this)" class="text-red-400 hover:text-red-600 p-1">❌</button>
                    </div>
                `);
            });
            document.getElementById('edit-match-modal').classList.remove('hidden');
        };

        window.closeEditMatchModal = function() {
            document.getElementById('edit-match-modal').classList.add('hidden');
            editingMatchId = null;
        };

        window.addEditOptionRow = function() {
            const container = document.getElementById('edit-options-container');
            const rowCount = container.children.length;
            const label = String.fromCharCode(65 + rowCount);
            container.insertAdjacentHTML('beforeend', `
                <div class="flex items-center gap-2 edit-option-row">
                    <input type="hidden" class="edit-opt-id" value="new_${Date.now()}">
                    <span class="text-xs font-bold text-gray-400 w-4">${label}</span>
                    <input type="text" placeholder="選項" class="flex-grow border border-gray-300 rounded-lg px-2 py-1.5 text-sm focus:border-hkjc-blue edit-opt-name">
                    <input type="number" step="0.1" min="1.01" placeholder="賠率" class="w-20 border border-gray-300 rounded-lg px-2 py-1.5 text-sm focus:border-hkjc-blue edit-opt-odds">
                    <button onclick="removeEditOptionRow(this)" class="text-red-400 hover:text-red-600 p-1">❌</button>
                </div>
            `);
        };

        window.removeEditOptionRow = function(btn) {
            const container = document.getElementById('edit-options-container');
            if (container.children.length <= 2) return showToast('最少必須保留兩個選項！', '⚠️');
            btn.parentElement.remove();
            Array.from(container.children).forEach((row, i) => {
                row.querySelector('span').textContent = String.fromCharCode(65 + i);
            });
        };

        window.saveEditedMatch = async function() {
            const state = window.GameSystem.getState();
            const title = document.getElementById('edit-match-title').value.trim();
            if (!title) return showToast('請填寫賽事名稱！', '⚠️');

            const rows = document.getElementsByClassName('edit-option-row');
            const newOptions = [];
            for (let row of rows) {
                const id = row.querySelector('.edit-opt-id').value;
                const name = row.querySelector('.edit-opt-name').value.trim();
                const odds = parseFloat(row.querySelector('.edit-opt-odds').value);
                if (!name || isNaN(odds) || odds <= 1) return showToast('選項與賠率不完整！', '⚠️');
                newOptions.push({ id: id.startsWith('new_') ? 'o_' + Date.now() + Math.random().toString(36).substr(2,4) : id, name, odds });
            }

            const match = state.matches.find(m => m.id === editingMatchId);
            match.title = '🐕 ' + title;
            
            // 處理被刪除的選項（退款）
            const newOptionIds = newOptions.map(o => o.id);
            const betsToRefund = state.bets.filter(b => b.matchId === editingMatchId && b.status === 'pending' && !newOptionIds.includes(b.optionId));
            betsToRefund.forEach(b => {
                const u = state.users.find(user => user.name === b.username);
                if (u) u.balance += b.amount;
            });
            // 移除已退款的投注，並更新保留投注的賽事標題和選項名稱(賠率不溯及既往，保持玩家下注時的賠率)
            state.bets = state.bets.filter(b => !(b.matchId === editingMatchId && b.status === 'pending' && !newOptionIds.includes(b.optionId)));
            state.bets.forEach(b => {
                if (b.matchId === editingMatchId && b.status === 'pending') {
                    b.matchTitle = match.title;
                    const opt = newOptions.find(o => o.id === b.optionId);
                    if (opt) b.optionName = opt.name;
                }
            });

            match.options = newOptions;

            closeEditMatchModal();
            showToast('賽事修改成功！', '💾');
            await window.GameSystem.updateCloud();
        };

        // --- 開設新賽事 ---
        window.addCreateOptionRow = function() {
            const container = document.getElementById('options-inputs-container');
            const rowCount = container.children.length;
            const label = String.fromCharCode(65 + rowCount);
            container.insertAdjacentHTML('beforeend', `
                <div class="flex items-center gap-2 create-option-row">
                    <span class="text-xs font-bold text-gray-400 w-4">${label}</span>
                    <input type="text" placeholder="選項名稱" class="flex-grow border border-gray-300 rounded-lg px-2 py-1.5 text-sm opt-name">
                    <input type="number" step="0.1" min="1.01" placeholder="賠率" class="w-20 border border-gray-300 rounded-lg px-2 py-1.5 text-sm opt-odds">
                    <button onclick="removeCreateOptionRow(this)" class="text-red-400 p-1">❌</button>
                </div>
            `);
        };

        window.removeCreateOptionRow = function(btn) {
            const container = document.getElementById('options-inputs-container');
            if (container.children.length <= 2) return showToast('最少保留兩個選項', '⚠️');
            btn.parentElement.remove();
            Array.from(container.children).forEach((row, i) => row.querySelector('span').textContent = String.fromCharCode(65 + i));
        };

        window.createMatch = async function() {
            const state = window.GameSystem.getState();
            const title = document.getElementById('match-title-input').value.trim();
            if (!title) return showToast('請填寫賽事名稱！', '⚠️');

            const rows = document.getElementsByClassName('create-option-row');
            const options = [];
            for (let row of rows) {
                const name = row.querySelector('.opt-name').value.trim();
                const odds = parseFloat(row.querySelector('.opt-odds').value);
                if (!name || isNaN(odds) || odds <= 1) return showToast('選項與賠率不完整！', '⚠️');
                options.push({ id: 'o_' + Date.now() + Math.random().toString(36).substr(2, 4), name, odds });
            }

            state.matches.push({ id: 'm_' + Date.now(), title: '🐕 ' + title, options, settled: false, winner: null });
            
            document.getElementById('match-title-input').value = '';
            showToast('新賽事發布成功！', '🎉');
            switchTab('betting');
            await window.GameSystem.updateCloud();
        };

        // --- 結算派彩 ---
        window.settleMatch = async function(matchId) {
            const state = window.GameSystem.getState();
            const selected = document.querySelector(`input[name="settle-${matchId}"]:checked`);
            if (!selected) return showToast('請選一個獲勝選項！', '⚠️');

            const winId = selected.value;
            const match = state.matches.find(m => m.id === matchId);
            match.settled = true;
            match.winner = winId;

            let total = 0;
            state.bets.filter(b => b.matchId === matchId && b.status === 'pending').forEach(bet => {
                if (bet.optionId === winId) {
                    bet.status = 'won';
                    bet.payout = Math.floor(bet.amount * bet.odds);
                    const user = state.users.find(u => u.name === bet.username);
                    if (user) user.balance += bet.payout;
                    total += bet.payout;
                } else {
                    bet.status = 'lost';
                }
            });

            showToast(`結算成功！共派彩 $${total.toLocaleString()}`, '💰');
            await window.GameSystem.updateCloud();
        };

        // --- 系統管理 ---
        window.requestDeleteUser = function(name) {
            const state = window.GameSystem.getState();
            if (state.users.length <= 1) return showToast('必須保留至少一名玩家！', '⚠️');
            showConfirmModal(
                "刪除帳戶", 
                `確定要刪除「${name}」嗎？他所有的未結算投注也會被移除。`,
                async () => {
                    state.users = state.users.filter(u => u.name !== name);
                    state.bets = state.bets.filter(b => b.username !== name);
                    showToast(`已刪除 ${name}。`, '🗑️');
                    await window.GameSystem.updateCloud();
                }
            );
        };

        window.requestResetData = function() {
            showConfirmModal(
                "重置全體數據", 
                "🚨 注意：此操作將會清除目前雲端上所有的賽事和投注記錄，重新回復成最初始設定！確定要清空並重新洗牌嗎？",
                async () => {
                    await window.GameSystem.resetData();
                    showToast("數據已成功回復初始設定！", "✨");
                }
            );
        };

        // --- 資金修改邏輯 ---
        let editingBalanceUser = null;

        window.openEditBalanceModal = function(name, currentBalance) {
            editingBalanceUser = name;
            document.getElementById('edit-balance-label').textContent = `修改「${name}」的資金：`;
            document.getElementById('edit-balance-input').value = currentBalance;
            document.getElementById('edit-balance-modal').classList.remove('hidden');
        };

        window.closeEditBalanceModal = function() {
            document.getElementById('edit-balance-modal').classList.add('hidden');
            editingBalanceUser = null;
        };

        window.executeEditBalance = async function() {
            const newBalanceStr = document.getElementById('edit-balance-input').value;
            const newBalance = parseInt(newBalanceStr);
            const state = window.GameSystem.getState();

            if (isNaN(newBalance) || newBalance < 0) return showToast('請輸入有效的資金數字（不能為負數）！', '⚠️');
            
            const userObj = state.users.find(u => u.name === editingBalanceUser);
            if (userObj) {
                userObj.balance = newBalance;
                closeEditBalanceModal();
                showToast(`已成功修改「${editingBalanceUser}」的資金為 $${newBalance.toLocaleString()}！`, '💰');
                await window.GameSystem.updateCloud();
            } else {
                showToast('找不到該玩家！', '⚠️');
                closeEditBalanceModal();
            }
        };

        // 🌟 分頁切換
        function switchTab(tabId) {
            ['betting', 'records', 'admin'].forEach(id => {
                document.getElementById(`section-${id}`).classList.add('hidden');
                document.getElementById(`section-${id}`).classList.remove('block');
                document.getElementById(`nav-${id}`).className = "flex-1 py-3 flex flex-col items-center justify-center gap-1 border-t-4 border-transparent text-gray-400 hover:bg-gray-50 transition-all";
            });

            document.getElementById(`section-${tabId}`).classList.remove('hidden');
            document.getElementById(`section-${tabId}`).classList.add('block');
            document.getElementById(`nav-${tabId}`).className = "flex-1 py-3 flex flex-col items-center justify-center gap-1 border-t-4 border-hkjc-blue bg-blue-50 text-hkjc-blue transition-all";
            window.scrollTo({ top: 0, behavior: 'smooth' });
        }
    </script>
</body>
</html>
