<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TeenPay - Secure UPI for Teenagers</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Plus Jakarta Sans', sans-serif; }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex items-center justify-center p-0 sm:p-4">

    <div id="app" class="w-full max-w-md bg-slate-900 sm:rounded-3xl shadow-2xl overflow-hidden min-h-screen sm:min-h-[750px] flex flex-col relative border border-slate-800">
        <!-- App content will be dynamically rendered here -->
    </div>

    <script>
        // --- State Management ---
        let state = {
            currentScreen: 'splash', // splash, login, signup, dashboard, scan, receive, history, profile, pinModal
            user: JSON.parse(localStorage.getItem('teenpay_user')) || null,
            usersDb: JSON.parse(localStorage.getItem('teenpay_users_db')) || [],
            transactions: JSON.parse(localStorage.getItem('teenpay_txns')) || [],
            showBalance: true,
            modalConfig: null // for PIN verification callback
        };

        function saveState() {
            if (state.user) {
                localStorage.setItem('teenpay_user', JSON.stringify(state.user));
                // Update in db
                const idx = state.usersDb.findIndex(u => u.phone === state.user.phone);
                if(idx >= 0) state.usersDb[idx] = state.user;
                else state.usersDb.push(state.user);
                localStorage.setItem('teenpay_users_db', JSON.stringify(state.usersDb));
            }
            localStorage.setItem('teenpay_txns', JSON.stringify(state.transactions));
            render();
        }

        // --- Render Engine ---
        function render() {
            const app = document.getElementById('app');
            app.innerHTML = '';

            switch(state.currentScreen) {
                case 'splash':
                    app.innerHTML = renderSplash();
                    break;
                case 'login':
                    app.innerHTML = renderLogin();
                    break;
                case 'signup':
                    app.innerHTML = renderSignup();
                    break;
                case 'dashboard':
                    app.innerHTML = renderDashboard();
                    break;
                case 'scan':
                    app.innerHTML = renderScanPay();
                    break;
                case 'receive':
                    app.innerHTML = renderReceive();
                    break;
                case 'history':
                    app.innerHTML = renderHistory();
                    break;
                case 'profile':
                    app.innerHTML = renderProfile();
                    break;
            }

            // Render active modal if any
            if (state.activeModal) {
                app.innerHTML += renderModal();
            }

            lucide.createIcons();
        }

        // --- Screens ---

        function renderSplash() {
            return `
                <div class="flex-1 flex flex-col items-center justify-between p-8 text-center bg-gradient-to-b from-indigo-950 via-slate-900 to-slate-950">
                    <div class="mt-12">
                        <div class="w-20 h-20 bg-indigo-600 rounded-3xl mx-auto flex items-center justify-center shadow-lg shadow-indigo-500/30 mb-6">
                            <i data-lucide="zap" class="w-10 h-10 text-white"></i>
                        </div>
                        <h1 class="text-3xl font-bold tracking-tight text-white mb-2">TeenPay</h1>
                        <p class="text-slate-400 text-sm">The smartest UPI & Wallet experience designed specially for teenagers.</p>
                    </div>
                    <div class="w-full space-y-4 mb-8">
                        <button onclick="navigate('${state.user ? 'dashboard' : 'login'}')" class="w-full py-4 bg-indigo-600 hover:bg-indigo-500 text-white font-semibold rounded-2xl shadow-lg shadow-indigo-600/30 transition">
                            ${state.user ? 'Open Dashboard' : 'Get Started'}
                        </button>
                        ${!state.user ? `
                        <button onclick="navigate('signup')" class="w-full py-4 bg-slate-800 hover:bg-slate-700 text-slate-300 font-semibold rounded-2xl transition">
                            Create New Account
                        </button>` : ''}
                    </div>
                </div>
            `;
        }

        function renderLogin() {
            return `
                <div class="flex-1 flex flex-col justify-between p-6">
                    <div class="flex items-center justify-between">
                        <button onclick="navigate('splash')" class="p-2 bg-slate-800 rounded-full text-slate-300"><i data-lucide="arrow-left" class="w-5 h-5"></i></button>
                        <span class="text-sm font-semibold text-slate-400">Step 1 of 1</span>
                    </div>
                    <div class="my-auto">
                        <h2 class="text-2xl font-bold mb-2">Welcome Back!</h2>
                        <p class="text-slate-400 text-sm mb-6">Enter your credentials to access your TeenPay account.</p>
                        <form onsubmit="handleLogin(event)" class="space-y-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-400 mb-1">REGISTERED PHONE / USERNAME</label>
                                <input type="text" id="loginId" required placeholder="Enter name or phone" class="w-full bg-slate-800 border border-slate-700 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-indigo-500">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-400 mb-1">PASSWORD</label>
                                <input type="password" id="loginPass" required placeholder="••••••••" class="w-full bg-slate-800 border border-slate-700 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-indigo-500">
                            </div>
                            <button type="submit" class="w-full py-4 bg-indigo-600 hover:bg-indigo-500 text-white font-semibold rounded-2xl shadow-lg shadow-indigo-600/30 transition mt-4">
                                Secure Login
                            </button>
                        </form>
                    </div>
                    <p class="text-center text-sm text-slate-400">Don't have an account? <button onclick="navigate('signup')" class="text-indigo-400 font-semibold">Sign Up</button></p>
                </div>
            `;
        }

        function renderSignup() {
            return `
                <div class="flex-1 flex flex-col justify-between p-6 overflow-y-auto">
                    <div class="flex items-center justify-between">
                        <button onclick="navigate('splash')" class="p-2 bg-slate-800 rounded-full text-slate-300"><i data-lucide="arrow-left" class="w-5 h-5"></i></button>
                        <span class="text-sm font-semibold text-slate-400">New Account</span>
                    </div>
                    <div class="my-6">
                        <h2 class="text-2xl font-bold mb-2">Create TeenPay</h2>
                        <p class="text-slate-400 text-sm mb-6">Zero bank account needed. Instant setup!</p>
                        <form onsubmit="handleSignup(event)" class="space-y-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-400 mb-1">REAL FULL NAME</label>
                                <input type="text" id="signupName" required placeholder="e.g. Pritosh Debnath" class="w-full bg-slate-800 border border-slate-700 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-indigo-500">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-400 mb-1">PHONE NUMBER</label>
                                <input type="tel" id="signupPhone" required placeholder="10-digit mobile number" class="w-full bg-slate-800 border border-slate-700 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-indigo-500">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-400 mb-1">SET PASSWORD</label>
                                <input type="password" id="signupPass" required placeholder="••••••••" class="w-full bg-slate-800 border border-slate-700 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-indigo-500">
                            </div>
                            <button type="submit" class="w-full py-4 bg-indigo-600 hover:bg-indigo-500 text-white font-semibold rounded-2xl shadow-lg shadow-indigo-600/30 transition mt-4">
                                Create Account (₹0 Balance)
                            </button>
                        </form>
                    </div>
                </div>
            `;
        }

        function renderDashboard() {
            const user = state.user;
            return `
                <div class="flex-1 flex flex-col justify-between pb-20">
                    <!-- Header -->
                    <div class="p-6 bg-slate-900/80 backdrop-blur sticky top-0 z-10 border-b border-slate-800 flex items-center justify-between">
                        <div class="flex items-center space-x-3">
                            <div class="w-10 h-10 bg-indigo-600 rounded-full flex items-center justify-center font-bold text-white">
                                ${user.name.charAt(0).toUpperCase()}
                            </div>
                            <div>
                                <h3 class="font-bold text-sm text-white">${user.name}</h3>
                                <p class="text-xs text-indigo-400">${user.upiId}</p>
                            </div>
                        </div>
                        <button onclick="navigate('profile')" class="p-2 bg-slate-800 hover:bg-slate-700 rounded-full text-slate-300">
                            <i data-lucide="settings" class="w-5 h-5"></i>
                        </button>
                    </div>

                    <!-- Main Body -->
                    <div class="p-6 space-y-6 overflow-y-auto flex-1">
                        <!-- Wallet Card -->
                        <div class="bg-gradient-to-br from-indigo-900 via-indigo-950 to-slate-900 border border-indigo-500/30 rounded-3xl p-6 shadow-xl relative overflow-hidden">
                            <div class="absolute -right-10 -bottom-10 w-32 h-32 bg-indigo-500/10 rounded-full blur-2xl"></div>
                            <div class="flex justify-between items-center mb-4">
                                <span class="text-xs font-semibold tracking-wider text-indigo-300 uppercase">TeenPay Wallet</span>
                                <button onclick="toggleBalance()" class="text-indigo-300 hover:text-white">
                                    <i data-lucide="${state.showBalance ? 'eye' : 'eye-off'}" class="w-5 h-5"></i>
                                </button>
                            </div>
                            <div class="text-3xl font-extrabold text-white mb-6">
                                ${state.showBalance ? '₹' + user.balance.toFixed(2) : '₹ ••••••'}
                            </div>
                            <div class="flex gap-3">
                                <button onclick="openAddMoneyModal()" class="flex-1 py-3 bg-indigo-600 hover:bg-indigo-500 text-white font-semibold rounded-xl text-sm flex items-center justify-center space-x-2 shadow-lg shadow-indigo-600/30 transition">
                                    <i data-lucide="plus-circle" class="w-4 h-4"></i>
                                    <span>Add Money</span>
                                </button>
                                <button onclick="navigate('receive')" class="flex-1 py-3 bg-slate-800 hover:bg-slate-700 text-slate-200 font-semibold rounded-xl text-sm flex items-center justify-center space-x-2 border border-slate-700 transition">
                                    <i data-lucide="qr-code" class="w-4 h-4"></i>
                                    <span>Receive QR</span>
                                </button>
                            </div>
                        </div>

                        <!-- Quick Actions -->
                        <div class="grid grid-cols-3 gap-4">
                            <button onclick="navigate('scan')" class="bg-slate-800 hover:bg-slate-700/80 p-4 rounded-2xl flex flex-col items-center justify-center space-y-2 border border-slate-700/50 transition">
                                <div class="w-12 h-12 bg-indigo-500/20 text-indigo-400 rounded-xl flex items-center justify-center">
                                    <i data-lucide="scan" class="w-6 h-6"></i>
                                </div>
                                <span class="text-xs font-semibold text-slate-300">Scan & Pay</span>
                            </button>
                            <button onclick="openSendMoneyModal()" class="bg-slate-800 hover:bg-slate-700/80 p-4 rounded-2xl flex flex-col items-center justify-center space-y-2 border border-slate-700/50 transition">
                                <div class="w-12 h-12 bg-emerald-500/20 text-emerald-400 rounded-xl flex items-center justify-center">
                                    <i data-lucide="send" class="w-6 h-6"></i>
                                </div>
                                <span class="text-xs font-semibold text-slate-300">To UPI ID</span>
                            </button>
                            <button onclick="checkBalanceWithPin()" class="bg-slate-800 hover:bg-slate-700/80 p-4 rounded-2xl flex flex-col items-center justify-center space-y-2 border border-slate-700/50 transition">
                                <div class="w-12 h-12 bg-amber-500/20 text-amber-400 rounded-xl flex items-center justify-center">
                                    <i data-lucide="shield-check" class="w-6 h-6"></i>
                                </div>
                                <span class="text-xs font-semibold text-slate-300">Check PIN</span>
                            </button>
                        </div>

                        <!-- Recent Activity -->
                        <div>
                            <div class="flex justify-between items-center mb-3">
                                <h4 class="text-sm font-bold text-slate-300">Recent Transactions</h4>
                                <button onclick="navigate('history')" class="text-xs text-indigo-400 font-semibold hover:underline">View All</button>
                            </div>
                            <div class="space-y-3">
                                ${renderRecentTransactionsList()}
                            </div>
                        </div>
                    </div>

                    <!-- Bottom Nav -->
                    ${renderBottomNav('dashboard')}
                </div>
            `;
        }

        function renderScanPay() {
            return `
                <div class="flex-1 flex flex-col justify-between p-6">
                    <div class="flex items-center justify-between">
                        <button onclick="navigate('dashboard')" class="p-2 bg-slate-800 rounded-full text-slate-300"><i data-lucide="arrow-left" class="w-5 h-5"></i></button>
                        <h3 class="font-bold">Scan & Pay</h3>
                        <div class="w-9"></div>
                    </div>
                    <div class="my-auto text-center space-y-4">
                        <div class="w-64 h-64 mx-auto border-2 border-dashed border-indigo-500 rounded-3xl flex flex-col items-center justify-center bg-slate-800/40 relative overflow-hidden">
                            <i data-lucide="camera" class="w-12 h-12 text-indigo-400 mb-2 animate-pulse"></i>
                            <p class="text-xs text-slate-400">Align QR code within frame</p>
                            <div class="absolute inset-x-0 top-1/2 h-0.5 bg-indigo-500 shadow-[0_0_15px_#6366f1]"></div>
                        </div>
                        <p class="text-xs text-slate-500">Or use manual transfer below</p>
                        <button onclick="openSendMoneyModal()" class="w-full py-3 bg-indigo-600 hover:bg-indigo-500 text-white rounded-xl font-semibold text-sm">
                            Enter UPI ID Manually
                        </button>
                    </div>
                    ${renderBottomNav('scan')}
                </div>
            `;
        }

        function renderReceive() {
            const user = state.user;
            const upiString = `upi://pay?pa=${user.upiId}&pn=${encodeURIComponent(user.name)}&cu=INR`;
            // Using public QR API for quick rendering
            const qrImageUrl = `https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=${encodeURIComponent(upiString)}&bgcolor=0f172a&color=ffffff`;

            return `
                <div class="flex-1 flex flex-col justify-between p-6">
                    <div class="flex items-center justify-between">
                        <button onclick="navigate('dashboard')" class="p-2 bg-slate-800 rounded-full text-slate-300"><i data-lucide="arrow-left" class="w-5 h-5"></i></button>
                        <h3 class="font-bold">Receive Money</h3>
                        <div class="w-9"></div>
                    </div>
                    <div class="my-auto text-center">
                        <div class="bg-slate-800 border border-slate-700 p-6 rounded-3xl inline-block shadow-2xl mb-4">
                            <img src="${qrImageUrl}" alt="UPI QR" class="w-48 h-48 mx-auto rounded-xl">
                            <h4 class="font-bold text-lg mt-4 text-white">${user.name}</h4>
                            <p class="text-xs text-indigo-400 mt-1">${user.upiId}</p>
                        </div>
                        <p class="text-xs text-slate-400">Scan this real interoperable UPI QR with Google Pay, PhonePe, or Paytm to send money.</p>
                    </div>
                    ${renderBottomNav('receive')}
                </div>
            `;
        }

        function renderHistory() {
            return `
                <div class="flex-1 flex flex-col justify-between pb-20">
                    <div class="p-6 bg-slate-900 border-b border-slate-800 flex items-center justify-between sticky top-0 z-10">
                        <button onclick="navigate('dashboard')" class="p-2 bg-slate-800 rounded-full text-slate-300"><i data-lucide="arrow-left" class="w-5 h-5"></i></button>
                        <h3 class="font-bold">Transaction History</h3>
                        <div class="w-9"></div>
                    </div>
                    <div class="p-6 space-y-3 overflow-y-auto flex-1">
                        ${state.transactions.length === 0 ? '<p class="text-center text-slate-500 text-sm mt-10">No transactions yet.</p>' : 
                          state.transactions.map(t => `
                            <div class="bg-slate-800/60 border border-slate-700/50 p-4 rounded-2xl flex items-center justify-between">
                        
