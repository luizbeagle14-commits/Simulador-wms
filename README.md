<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador WMS – Da Separação à Entrega</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Google Fonts: Poppins & Roboto -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@600;700&family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
    
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>

    <!-- Leaflet.js (for Maps) -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>
    <!-- Leaflet Animated Marker Plugin -->
    <script src="https://unpkg.com/leaflet.animatedmarker/src/AnimatedMarker.js"></script>


    <style>
        :root {
            --color-primary: #1A237E;
            --color-secondary: #9E9E9E;
            --color-background: #F5F5F5;
            --color-highlight: #00C853;
            --color-action: #FF6F00;
            --color-text-main: #212121;
            --color-text-light: #FFFFFF;
        }

        body {
            font-family: 'Roboto', sans-serif;
            background-color: var(--color-background);
            color: var(--color-text-main);
        }

        h1, h2, h3, h4, h5, h6 {
            font-family: 'Poppins', sans-serif;
            color: var(--color-primary);
        }

        .bg-primary { background-color: var(--color-primary); }
        .text-primary { color: var(--color-primary); }
        .bg-highlight { background-color: var(--color-highlight); }
        .text-highlight { color: var(--color-highlight); }
        .border-highlight { border-color: var(--color-highlight); }
        .bg-action { background-color: var(--color-action); }
        .text-action { color: var(--color-action); }
        .border-action { border-color: var(--color-action); }

        /* Animações */
        @keyframes flash-success {
            from { background-color: #e8f5e9; }
            to { background-color: white; }
        }
        .flash-success { animation: flash-success 1s ease; }

        @keyframes flash-error {
            from { background-color: #fff3e0; }
            to { background-color: white; }
        }
        .flash-error { animation: flash-error 1s ease; }

        /* Estilos para componentes interativos */
        #signature-canvas {
            border: 2px dashed var(--color-secondary);
            cursor: crosshair;
            touch-action: none;
        }
        
        /* Modal */
        #modal-overlay {
            transition: opacity 0.3s ease;
        }

        /* Leaflet Map */
        #map {
             height: 450px;
             border-radius: 0.5rem;
             border: 1px solid #ddd;
             z-index: 10;
        }

        /* Print Styles */
        .print-only {
            display: none;
        }

        @media print {
            body * {
                visibility: hidden;
            }
            #print-area, #print-area * {
                visibility: visible;
            }
            #print-area {
                position: absolute;
                left: 0;
                top: 0;
                width: 100%;
                padding: 20px;
                font-family: sans-serif;
                color: black;
            }
            #print-area h1 { font-size: 24px; color: black; text-align: center; }
            #print-area h2 { font-size: 18px; color: black; border-bottom: 1px solid #ccc; padding-bottom: 5px; margin-top: 20px; }
            #print-area table { width: 100%; border-collapse: collapse; margin-top: 15px; }
            #print-area th, #print-area td { border: 1px solid #ccc; padding: 8px; text-align: left; }
            #print-area th { background-color: #f2f2f2; }
            #print-area .order-block { page-break-inside: avoid; }
            #print-area .signature-block { margin-top: 50px; text-align: center; }
        }
        
        /* Item scanning animation */
        .item-to-scan {
            transition: all 0.5s ease-out, height 0.5s ease-out 0.2s, padding 0.5s ease-out 0.2s, margin 0.5s ease-out 0.2s;
        }
        .item-scanned {
            opacity: 0;
            transform: translateX(50px);
            height: 0 !important;
            padding-top: 0 !important;
            padding-bottom: 0 !important;
            margin-top: 0 !important;
            margin-bottom: 0 !important;
            border-width: 0 !important;
            overflow: hidden;
        }
    </style>
</head>
<body class="bg-background">

    <div id="wms-simulator-app" class="min-h-screen flex flex-col">
        <!-- O conteúdo da simulação será injetado aqui -->
    </div>

    <!-- Template do Cabeçalho -->
    <template id="header-template">
        <header class="bg-white shadow-md z-20 p-4">
            <div class="flex items-center justify-between">
                <div class="flex items-center space-x-4">
        <img src="https://www.imagemhost.com.br/images/2024/11/22/Logo-novo-SENAI_-sem-slogan_755X325.png" 
             alt="Logo SENAI" 
             class="h-10 object-contain mr-4">
                    <div>
                        <h1 class="text-xl font-bold">Simulador WMS</h1>
                        <p id="header-step-name" class="text-sm text-secondary">Etapa Atual</p>
                    </div>
                </div>
                <div class="text-right">
                    <p id="header-user-info" class="font-semibold">Operador: Nome (Turno: Manhã)</p>
                    <div class="w-full bg-gray-200 rounded-full h-2.5 mt-1">
                        <div id="header-progress-bar" class="bg-highlight h-2.5 rounded-full" style="width: 0%"></div>
                    </div>
                </div>
            </div>
        </header>
    </template>
    
    <!-- Modal Genérico -->
    <div id="modal-overlay" class="hidden fixed inset-0 bg-gray-900 bg-opacity-75 flex items-center justify-center p-4 z-50">
        <div id="modal-content" class="bg-white rounded-lg shadow-xl p-6 max-w-lg w-full text-center">
            <!-- Conteúdo do modal será injetado aqui -->
        </div>
    </div>
    
    <div id="print-area" class="print-only"></div>


<script>
document.addEventListener('DOMContentLoaded', () => {

    // --- Tabela ICMS Interestadual (Origem TO) ---
    const ICMS_RATES = {
        'AC': 0.12, 'AL': 0.12, 'AP': 0.12, 'AM': 0.12, 'BA': 0.12, 'CE': 0.12,
        'DF': 0.07, 'ES': 0.07, 'GO': 0.07, 'MA': 0.12, 'MT': 0.07, 'MS': 0.07,
        'MG': 0.07, 'PA': 0.12, 'PB': 0.12, 'PR': 0.07, 'PE': 0.12, 'PI': 0.12,
        'RJ': 0.07, 'RN': 0.12, 'RS': 0.07, 'RO': 0.12, 'RR': 0.12, 'SC': 0.07,
        'SP': 0.07, 'SE': 0.12, 'TO': 0.00 // Intra-estadual
    };

    // --- INITIAL DATA ---
    const initialData = {
        products: {
            'PROD-101': { name: 'Caixa de Leite Integral', img: 'https://lojacvscesta.com.br/cdn/shop/products/caixa-italac_grande.png', price: 5.50, weight: 1.0, barcode: 'PROD-101', stock: 150, length: 0.2, width: 0.1, height: 0.15, arrivalDate: '2025-06-15' },
            'PROD-102': { name: 'Pacote de Arroz 5kg', img: 'https://i.pinimg.com/736x/3e/02/e2/3e02e2cd1fe14e1f0f27629de3bf493b.jpg', price: 25.00, weight: 5.0, barcode: 'PROD-102', stock: 80, length: 0.4, width: 0.25, height: 0.1, arrivalDate: '2025-06-20' },
            'PROD-103': { name: 'Garrafa de Óleo de Soja', img: 'https://ibassets.com.br/ib.item.image.big/b-e3c332eab90a495d9bb6e540cd899d5c.jpeg', price: 8.75, weight: 0.9, barcode: 'PROD-103', stock: 200, length: 0.08, width: 0.08, height: 0.3, arrivalDate: '2025-06-18' },
        },
        orders: [],
    };

    // --- STATE MANAGEMENT ---
    let orderIntervalId = null; // To manage the automatic order generation
    const state = {
        currentStep: 'login',
        userName: '',
        userShift: '',
        products: {},
        orders: [],
        deliveredItems: [],
        qualityReport: {},
        currentShipment: {
            orders: [],
            totalWeight: 0,
            totalValue: 0,
            incidents: [],
            route: {
                totalDistance: 0,
                points: []
            }
        },
        scannedItems: {},
        nonConformities: [],
        totalSteps: 12,
        userLocation: null,
    };

    // Deep copy initial data to state
    state.products = JSON.parse(JSON.stringify(initialData.products));
    state.orders = JSON.parse(JSON.stringify(initialData.orders));


    const appContainer = document.getElementById('wms-simulator-app');
    
    // --- HELPER FUNCTIONS ---
    function calculateDistance(lat1, lon1, lat2, lon2) {
        const R = 6371; // Radius of the Earth in km
        const dLat = (lat2 - lat1) * Math.PI / 180;
        const dLon = (lon2 - lon1) * Math.PI / 180;
        const a =
            Math.sin(dLat / 2) * Math.sin(dLat / 2) +
            Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
            Math.sin(dLon / 2) * Math.sin(dLon / 2);
        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
        const distance = R * c; // Distance in km
        return distance;
    }

    // --- RENDER FUNCTIONS ---

    const renderHeader = () => {
        const headerTemplate = document.getElementById('header-template').content.cloneNode(true);
        appContainer.prepend(headerTemplate);
        lucide.createIcons();
        updateHeader();
    };

    const updateHeader = () => {
        const stepNameEl = document.getElementById('header-step-name');
        if (!stepNameEl) return;

        const stepNames = {
            login: "Login Operacional",
            dashboard: "Dashboard Gerencial",
            catalog: "Catálogo de Produtos",
            create_order: "Criar Pedido de Compra",
            order_list: "Painel de Pedidos",
            scanning: "Separação com Código de Barras",
            romaneio: "Romaneio Digital",
            routing: "Roteirização e Distância",
            freight_check: "Conferência e Frete",
            delivery_tracking: "Rastreamento da Entrega",
            delivery: "Baixa de Entrega",
            quiz: "Avaliação de Conhecimento",
            final_challenge_quiz: "Desafio Final de Gestão",
            final_report_dashboard: "Relatório Final da Operação"
        };
        const stepNumbers = { login: 0, dashboard: 1, catalog: 2, create_order: 3, order_list: 4, scanning: 5, romaneio: 6, routing: 7, freight_check: 8, delivery_tracking: 9, delivery: 10, quiz: 11, final_challenge_quiz: 12, final_report_dashboard: 13 };

        stepNameEl.textContent = stepNames[state.currentStep] || 'Bem-vindo';
        document.getElementById('header-user-info').textContent = `Operador: ${state.userName || 'N/A'} (Turno: ${state.userShift || 'N/A'})`;
        const progress = (stepNumbers[state.currentStep] / state.totalSteps) * 100;
        document.getElementById('header-progress-bar').style.width = `${progress}%`;
    };

    const render = () => {
        if (orderIntervalId) {
            clearInterval(orderIntervalId);
            orderIntervalId = null;
        }
        appContainer.innerHTML = ''; // Clear screen
        
        switch (state.currentStep) {
            case 'login': renderLogin(); break;
            case 'dashboard': renderHeader(); renderDashboard(); break;
            case 'catalog': renderHeader(); renderCatalog(); break;
            case 'create_order': renderHeader(); renderCreateOrder(); break;
            case 'order_list': renderHeader(); renderOrderList(); break;
            case 'scanning': renderHeader(); renderScanning(); break;
            case 'romaneio': renderHeader(); renderRomaneio(); break;
            case 'routing': renderHeader(); renderRouting(); break;
            case 'freight_check': renderHeader(); renderFreightCheck(); break;
            case 'delivery_tracking': renderHeader(); renderDeliveryTracking(); break;
            case 'delivery': renderHeader(); renderDelivery(); break;
            case 'quiz': renderHeader(); renderQuiz(); break;
            case 'final_challenge_quiz': renderHeader(); renderFinalChallengeQuiz(); break;
            case 'final_report_dashboard': renderHeader(); renderFinalReportDashboard(); break;
        }
    };
    
    // --- STEP RENDERERS ---

    function renderLogin() {
        appContainer.innerHTML = `
            <div class="flex-grow flex items-center justify-center p-4">
                <div class="w-full max-w-md bg-white p-8 rounded-xl shadow-2xl text-center">
                   <img src="https://www.imagemhost.com.br/images/2024/11/22/Logo-novo-SENAI_-sem-slogan_755X325.png" 
                 alt="Logo da Atividade" 
                 class="w-64 mx-auto mb-4 object-contain">
                    <h2 class="text-3xl font-bold mb-2">Simulador WMS</h2>
                    <p class="text-secondary mb-6">Login Operacional</p>
                    <div class="space-y-4 text-left">
                        <div>
                            <label for="user-name" class="block text-sm font-medium text-gray-700">Seu Nome:</label>
                            <input type="text" id="user-name" class="mt-1 block w-full px-3 py-2 bg-white border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-primary focus:border-primary">
                        </div>
                        <div>
                            <label for="user-shift" class="block text-sm font-medium text-gray-700">Selecione o Turno:</label>
                            <select id="user-shift" class="mt-1 block w-full px-3 py-2 bg-white border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-primary focus:border-primary">
                                <option>Manhã</option>
                                <option>Tarde</option>
                                <option>Noite</option>
                            </select>
                        </div>
                    </div>
                    <button id="login-btn" class="mt-8 w-full bg-primary text-white font-bold py-3 px-4 rounded-lg hover:opacity-90 transition-opacity flex items-center justify-center gap-2">
                        <i data-lucide="log-in"></i> Iniciar Turno
                    </button>
                    <p class="text-xs text-gray-400 mt-6">Professor: Luiz Eduardo Peixoto<br></p>
                </div>
            </div>
        `;
        lucide.createIcons();
        document.getElementById('login-btn').addEventListener('click', () => {
            state.userName = document.getElementById('user-name').value || 'Anônimo';
            state.userShift = document.getElementById('user-shift').value;
            state.currentStep = 'dashboard';
            render();
        });
    }

    function renderDashboard() {
        const deliveredCount = state.deliveredItems.length;
        const stockHtml = Object.entries(state.products).map(([id, prod]) => {
            const stockLevelColor = prod.stock > 50 ? 'text-green-600' : prod.stock > 10 ? 'text-yellow-600' : 'text-red-600';
            return `<li class="flex justify-between items-center text-sm"><span>${prod.name}</span> <span class="font-bold ${stockLevelColor}">${prod.stock} un.</span></li>`;
        }).join('');

        const soldItemsCount = state.deliveredItems.reduce((acc, item) => {
            acc[item] = (acc[item] || 0) + 1;
            return acc;
        }, {});
        const soldItemsHtml = Object.entries(soldItemsCount).sort((a,b) => b[1] - a[1]).map(([id, count]) => {
             return `<li class="flex justify-between items-center text-sm"><span>${state.products[id].name}</span> <span class="font-bold">${count} un.</span></li>`;
        }).join('');

        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Dashboard Gerencial</h2>
                    <div class="flex gap-4">
                        <button id="go-to-catalog-btn" class="bg-blue-600 text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2"><i data-lucide="book-open"></i> Catálogo</button>
                        <button id="go-to-orders-btn" class="bg-primary text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2">Ver Pedidos <i data-lucide="arrow-right"></i></button>
                    </div>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
                    <!-- Card: Itens Entregues -->
                    <div class="bg-white p-6 rounded-lg shadow-lg flex items-center gap-4">
                        <i data-lucide="package-check" class="w-12 h-12 text-highlight"></i>
                        <div>
                            <p class="text-secondary">Itens Entregues</p>
                            <p class="text-3xl font-bold">${deliveredCount}</p>
                        </div>
                    </div>
                    <!-- Card: Nível de Estoque -->
                    <div class="bg-white p-6 rounded-lg shadow-lg">
                        <h3 class="font-bold mb-3">Nível de Estoque</h3>
                        <ul class="space-y-2 max-h-48 overflow-y-auto pr-2">${stockHtml || '<p class="text-sm text-secondary">Nenhum produto no catálogo.</p>'}</ul>
                    </div>
                    <!-- Card: Itens Vendidos -->
                    <div class="bg-white p-6 rounded-lg shadow-lg">
                        <h3 class="font-bold mb-3">Produtos Mais Vendidos</h3>
                        <ul class="space-y-2 max-h-48 overflow-y-auto pr-2">${soldItemsHtml || '<p class="text-sm text-secondary">Nenhuma entrega realizada.</p>'}</ul>
                    </div>
                    <!-- Card: Ferramentas -->
                    <div class="bg-white p-6 rounded-lg shadow-lg space-y-4">
                         <div>
                            <h3 class="font-bold mb-2">Ponto de Ressuprimento</h3>
                            <div class="grid grid-cols-2 gap-2 text-sm">
                                <input id="demand" type="number" placeholder="Demanda/dia" class="p-1 border rounded">
                                <input id="leadtime" type="number" placeholder="Lead Time (dias)" class="p-1 border rounded">
                                <input id="safety-stock" type="number" placeholder="Estoque Seg." class="p-1 border rounded col-span-2">
                            </div>
                            <button id="calc-reorder-btn" class="w-full mt-2 text-sm bg-gray-200 py-1 rounded">Calcular</button>
                            <p class="text-center mt-1">Ponto de Pedido: <span id="reorder-result" class="font-bold">--</span></p>
                        </div>
                        <div class="border-t pt-4">
                            <h3 class="font-bold mb-2">Calculadora de Lead Time</h3>
                             <div class="grid grid-cols-2 gap-2 text-sm">
                                <div><label class="text-xs">Data Pedido</label><input id="order-date" type="date" class="w-full p-1 border rounded"></div>
                                <div><label class="text-xs">Data Receb.</label><input id="receipt-date" type="date" class="w-full p-1 border rounded"></div>
                            </div>
                            <button id="calc-leadtime-btn" class="w-full mt-2 text-sm bg-gray-200 py-1 rounded">Calcular</button>
                            <p class="text-center mt-1">Lead Time: <span id="leadtime-result" class="font-bold">--</span></p>
                        </div>
                    </div>
                </div>
                 <div class="mt-6 bg-white p-6 rounded-lg shadow-lg">
                    <h3 class="font-bold mb-3 text-xl">Calculadora de Paletização e Área</h3>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                        <div class="md:col-span-2">
                           <h4 class="font-semibold mb-2">Selecione os Produtos e Quantidades:</h4>
                           <div id="pallet-product-list" class="space-y-2 max-h-60 overflow-y-auto pr-2 border p-2 rounded-md">
                                <!-- Product list will be injected here -->
                           </div>
                        </div>
                        <div>
                            <h4 class="font-semibold mb-2">Resultado da Simulação</h4>
                            <button id="calc-pallet-btn" class="w-full bg-action text-white py-2 rounded-lg mb-4">Calcular</button>
                            <div class="bg-gray-100 p-4 rounded-lg space-y-2">
                                <p>Paletes PBR (1.0x1.2m) necessários:</p>
                                <p class="text-2xl font-bold text-primary" id="pallet-result">--</p>
                                <p>Área de Chão Ocupada:</p>
                                <p class="text-2xl font-bold text-primary" id="area-result">--</p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="mt-6 bg-white p-6 rounded-lg shadow-lg">
                    <h3 class="font-bold mb-3 text-xl">Registro de Controle de Qualidade</h3>
                    <div id="qc-form" class="space-y-4 text-left">
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div><label class="block text-sm font-medium">Operador de Qualidade:</label><input id="qc-operator" type="text" class="w-full p-2 border rounded bg-gray-100" value="${state.userName}" readonly></div>
                            <div><label class="block text-sm font-medium">ID do Pedido:</label><input id="qc-order-id" type="text" class="w-full p-2 border rounded" placeholder="Ex: PED-12345"></div>
                        </div>
                        <div><label class="block text-sm font-medium">Cliente:</label><input id="qc-customer" type="text" class="w-full p-2 border rounded" placeholder="Ex: Atacadão ABC"></div>
                        <div><label class="block text-sm font-medium">Produto:</label><input id="qc-product" type="text" class="w-full p-2 border rounded" placeholder="Ex: Caixa de Leite"></div>
                        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                            <div><label class="block text-sm font-medium">Peso Verificado (kg):</label><input id="qc-weight" type="number" step="0.1" class="w-full p-2 border rounded"></div>
                            <div><label class="block text-sm font-medium">Valor Declarado (R$):</label><input id="qc-value" type="number" step="0.01" class="w-full p-2 border rounded"></div>
                            <div><label class="block text-sm font-medium">Tempo Entrega (dias):</label><input id="qc-delivery-time" type="number" class="w-full p-2 border rounded"></div>
                        </div>
                        <div><label class="block text-sm font-medium">Observações:</label><textarea id="qc-notes" class="w-full p-2 border rounded" rows="3"></textarea></div>
                    </div>
                    <div class="flex gap-4 mt-4">
                        <button id="download-qc-btn" class="w-full bg-gray-500 text-white font-bold py-2 px-4 rounded-lg hover:opacity-90 flex items-center justify-center gap-2">
                            <i data-lucide="download"></i> Baixar Formulário
                        </button>
                        <button id="save-qc-btn" class="w-full bg-primary text-white font-bold py-2 px-4 rounded-lg hover:opacity-90 flex items-center justify-center gap-2">
                            <i data-lucide="save"></i> Salvar Registro
                        </button>
                    </div>
                </div>
            </main>
        `;
        lucide.createIcons();

        document.getElementById('go-to-orders-btn').onclick = () => { state.currentStep = 'order_list'; render(); };
        document.getElementById('go-to-catalog-btn').onclick = () => { state.currentStep = 'catalog'; render(); };
        
        // Calculators Logic
        document.getElementById('calc-reorder-btn').onclick = () => {
            const demand = parseFloat(document.getElementById('demand').value) || 0;
            const leadtime = parseFloat(document.getElementById('leadtime').value) || 0;
            const safetyStock = parseFloat(document.getElementById('safety-stock').value) || 0;
            const result = (demand * leadtime) + safetyStock;
            document.getElementById('reorder-result').textContent = `${result.toFixed(0)} un.`;
        };
        document.getElementById('calc-leadtime-btn').onclick = () => {
            const orderDate = new Date(document.getElementById('order-date').value);
            const receiptDate = new Date(document.getElementById('receipt-date').value);
            if(orderDate && receiptDate && receiptDate > orderDate) {
                const diffTime = Math.abs(receiptDate - orderDate);
                const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24)); 
                document.getElementById('leadtime-result').textContent = `${diffDays} dias`;
            } else {
                 document.getElementById('leadtime-result').textContent = 'Inválido';
            }
        };

        // Palletization Calculator Logic
        const palletProductList = document.getElementById('pallet-product-list');
        palletProductList.innerHTML = Object.entries(state.products).map(([id, prod]) => `
            <div class="flex items-center justify-between">
                <label for="pallet-prod-${id}" class="text-sm">${prod.name}</label>
                <input type="number" id="pallet-prod-${id}" min="0" placeholder="Qtd." class="w-20 p-1 border rounded text-sm">
            </div>
        `).join('');

        document.getElementById('calc-pallet-btn').onclick = () => {
            const PALLET_AREA = 1.0 * 1.2; // m²
            const MAX_HEIGHT = 1.5; // m
            const PALLET_VOLUME = PALLET_AREA * MAX_HEIGHT;
            let totalVolume = 0;

            Object.keys(state.products).forEach(id => {
                const qty = parseInt(document.getElementById(`pallet-prod-${id}`).value) || 0;
                if (qty > 0) {
                    const prod = state.products[id];
                    const prodVolume = prod.length * prod.width * prod.height;
                    totalVolume += prodVolume * qty;
                }
            });

            if(totalVolume > 0) {
                const palletsNeeded = Math.ceil(totalVolume / PALLET_VOLUME);
                const areaNeeded = palletsNeeded * PALLET_AREA;
                document.getElementById('pallet-result').textContent = `${palletsNeeded} paletes`;
                document.getElementById('area-result').textContent = `${areaNeeded.toFixed(2)} m²`;
            } else {
                document.getElementById('pallet-result').textContent = `0 paletes`;
                document.getElementById('area-result').textContent = `0.00 m²`;
            }
        };

        // Quality Control Logic
        document.getElementById('download-qc-btn').onclick = downloadQualityForm;
        document.getElementById('save-qc-btn').onclick = () => {
            state.qualityReport = {
                operator: document.getElementById('qc-operator').value,
                orderId: document.getElementById('qc-order-id').value,
                customer: document.getElementById('qc-customer').value,
                product: document.getElementById('qc-product').value,
                weight: document.getElementById('qc-weight').value,
                value: document.getElementById('qc-value').value,
                deliveryTime: document.getElementById('qc-delivery-time').value,
                notes: document.getElementById('qc-notes').value,
                timestamp: new Date().toISOString()
            };
            showModal(`<h3 class="text-xl font-bold text-highlight">Registro de Qualidade Salvo!</h3><p>As informações foram salvas com sucesso.</p>`);
        };
    }

    function renderCatalog() {
        let productsHtml = Object.entries(state.products).map(([id, prod]) => `
            <div class="bg-white rounded-lg shadow-md overflow-hidden flex flex-col">
                <img src="${prod.img}" alt="${prod.name}" class="w-full h-40 object-cover">
                <div class="p-4 flex-grow">
                    <h4 class="font-bold text-lg">${prod.name}</h4>
                    <p class="text-secondary">SKU: ${id}</p>
                    <p class="text-secondary">Peso: ${prod.weight.toFixed(2)} kg</p>
                    <p class="text-secondary">Volume: ${(prod.length * prod.width * prod.height).toFixed(3)} m³</p>
                    <p class="text-highlight font-semibold text-xl mt-2">R$ ${prod.price.toFixed(2)}</p>
                </div>
                <div class="p-4 border-t bg-gray-50">
                     <button data-prod-id="${id}" class="add-stock-btn w-full bg-blue-100 text-blue-800 font-bold py-2 px-4 rounded-lg flex items-center justify-center gap-2 text-sm hover:bg-blue-200"><i data-lucide="plus"></i> Adicionar ao Estoque</button>
                </div>
            </div>
        `).join('');

        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Catálogo de Produtos</h2>
                    <div class="flex gap-4">
                        <button id="back-to-dash-btn" class="bg-gray-200 py-2 px-4 rounded-lg flex items-center gap-2"><i data-lucide="arrow-left"></i> Voltar ao Dashboard</button>
                        <button id="add-product-btn" class="bg-action text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2"><i data-lucide="plus-circle"></i> Novo Produto</button>
                        <button id="go-to-create-order-btn" class="bg-primary text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2">Criar Pedido <i data-lucide="arrow-right"></i></button>
                    </div>
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
                    ${productsHtml}
                </div>
            </main>
        `;
        lucide.createIcons();
        document.getElementById('back-to-dash-btn').onclick = () => { state.currentStep = 'dashboard'; render(); };
        document.getElementById('add-product-btn').onclick = () => {
            showModal(`
                <h3 class="text-xl font-bold mb-4">Adicionar Novo Produto</h3>
                <div class="space-y-3 text-left">
                    <div><label class="block text-sm">Nome do Produto:</label><input id="new-prod-name" type="text" class="w-full p-2 border rounded"></div>
                    <div class="grid grid-cols-2 gap-2">
                        <div><label class="block text-sm">Preço (R$):</label><input id="new-prod-price" type="number" step="0.01" class="w-full p-2 border rounded"></div>
                        <div><label class="block text-sm">Quantidade Inicial:</label><input id="new-prod-stock" type="number" class="w-full p-2 border rounded"></div>
                    </div>
                     <div class="grid grid-cols-2 gap-2">
                        <div><label class="block text-sm">Peso (kg):</label><input id="new-prod-weight" type="number" step="0.1" class="w-full p-2 border rounded"></div>
                        <div><label class="block text-sm">Data Chegada:</label><input id="new-prod-arrival" type="date" class="w-full p-2 border rounded" value="${new Date().toISOString().substring(0, 10)}"></div>
                    </div>
                    <div>
                        <label class="block text-sm">Dimensões (metros):</label>
                        <div class="grid grid-cols-3 gap-2">
                            <input id="new-prod-l" type="number" step="0.01" placeholder="Comp." class="w-full p-2 border rounded">
                            <input id="new-prod-w" type="number" step="0.01" placeholder="Larg." class="w-full p-2 border rounded">
                            <input id="new-prod-h" type="number" step="0.01" placeholder="Alt." class="w-full p-2 border rounded">
                        </div>
                    </div>
                    <div><label class="block text-sm">Imagem do Produto:</label><input id="new-prod-img" type="file" accept="image/*" class="w-full text-sm"></div>
                </div>
                <button id="save-product-btn" class="mt-4 w-full bg-highlight text-white font-bold py-2 px-4 rounded-lg">Salvar Produto</button>
            `);
            document.getElementById('save-product-btn').onclick = () => {
                const name = document.getElementById('new-prod-name').value;
                const price = parseFloat(document.getElementById('new-prod-price').value);
                const stock = parseInt(document.getElementById('new-prod-stock').value);
                const weight = parseFloat(document.getElementById('new-prod-weight').value);
                const arrivalDate = document.getElementById('new-prod-arrival').value;
                const length = parseFloat(document.getElementById('new-prod-l').value);
                const width = parseFloat(document.getElementById('new-prod-w').value);
                const height = parseFloat(document.getElementById('new-prod-h').value);
                const imgFile = document.getElementById('new-prod-img').files[0];

                if(name && price > 0 && stock >= 0 && weight > 0 && arrivalDate && length > 0 && width > 0 && height > 0 && imgFile) {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        const newId = `PROD-${Date.now()}`;
                        state.products[newId] = { name, price, stock, weight, arrivalDate, length, width, height, img: e.target.result, barcode: newId };
                        closeModal();
                        render();
                    };
                    reader.readAsDataURL(imgFile);
                } else {
                    alert("Por favor, preencha todos os campos corretamente.");
                }
            };
        };
        
        document.querySelectorAll('.add-stock-btn').forEach(btn => {
            btn.onclick = (e) => {
                const prodId = e.currentTarget.dataset.prodId;
                const product = state.products[prodId];
                showModal(`
                    <h3 class="text-xl font-bold mb-4">Adicionar Estoque para ${product.name}</h3>
                    <p class="text-secondary mb-4">Estoque atual: ${product.stock} unidades.</p>
                    <div class="text-left">
                        <label class="block text-sm">Quantidade a adicionar:</label>
                        <input id="add-stock-qty" type="number" min="1" class="w-full p-2 border rounded">
                    </div>
                    <button id="save-stock-btn" class="mt-4 w-full bg-highlight text-white font-bold py-2 px-4 rounded-lg">Confirmar Entrada</button>
                `);
                document.getElementById('save-stock-btn').onclick = () => {
                    const qtyToAdd = parseInt(document.getElementById('add-stock-qty').value);
                    if (qtyToAdd > 0) {
                        state.products[prodId].stock += qtyToAdd;
                        closeModal();
                        render();
                    } else {
                        alert("Por favor, insira uma quantidade válida.");
                    }
                };
            };
        });

        document.getElementById('go-to-create-order-btn').onclick = () => {
            state.currentStep = 'create_order';
            render();
        };
    }

    function renderCreateOrder() {
        let orderItems = {};
        let productsHtml = Object.entries(state.products).map(([id, prod]) => `
            <div class="bg-white p-3 rounded-lg shadow-sm flex items-center justify-between">
                <div>
                    <p class="font-semibold">${prod.name}</p>
                    <p class="text-sm text-secondary">R$ ${prod.price.toFixed(2)}</p>
                </div>
                <button data-prod-id="${id}" class="add-to-cart-btn bg-highlight text-white rounded-full w-8 h-8 flex items-center justify-center hover:opacity-80"><i data-lucide="plus"></i></button>
            </div>
        `).join('');
        const statesBR = ["AC", "AL", "AP", "AM", "BA", "CE", "DF", "ES", "GO", "MA", "MT", "MS", "MG", "PA", "PB", "PR", "PE", "PI", "RJ", "RN", "RS", "RO", "RR", "SC", "SP", "SE", "TO"];
        const customerPrefixes = ["Distribuidora", "Atacadão", "Supermercado", "Comércio de Alimentos", "Empório"];
        const customerSuffixes = ["do Povo", "Central", "Norte Forte", "ABC", "Brasil"];
        const randomCustomer = `${customerPrefixes[Math.floor(Math.random() * customerPrefixes.length)]} ${customerSuffixes[Math.floor(Math.random() * customerSuffixes.length)]}`;
        
        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Criar Pedido de Compra</h2>
                    <button id="back-to-catalog-btn" class="bg-gray-200 py-2 px-4 rounded-lg flex items-center gap-2"><i data-lucide="arrow-left"></i> Voltar ao Catálogo</button>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
                    <div class="md:col-span-2 space-y-4">
                        <h3 class="text-xl font-bold">Selecione os Produtos</h3>
                        <div class="space-y-3 max-h-96 overflow-y-auto pr-2">${productsHtml}</div>
                    </div>
                    <div class="bg-white p-6 rounded-lg shadow-lg">
                        <h3 class="text-xl font-bold mb-4">Pedido Atual</h3>
                        <div class="mb-4 space-y-3">
                            <div>
                                <label for="customer-name" class="block text-sm font-medium">Nome do Cliente:</label>
                                <input id="customer-name" type="text" class="w-full p-2 border rounded" value="${randomCustomer}">
                            </div>
                            <div>
                                <label for="customer-address" class="block text-sm font-medium">Endereço:</label>
                                <input id="customer-address" type="text" class="w-full p-2 border rounded" placeholder="Ex: Rua das Flores, 123">
                            </div>
                             <div>
                                <label for="customer-state" class="block text-sm font-medium">Estado (UF):</label>
                                <select id="customer-state" class="w-full p-2 border rounded">${statesBR.map(uf => `<option value="${uf}" ${uf === 'TO' ? 'selected' : ''}>${uf}</option>`).join('')}</select>
                            </div>
                        </div>
                        <div id="cart-items" class="space-y-2 mb-4 border-t pt-4">
                            <p class="text-center text-secondary">O carrinho está vazio.</p>
                        </div>
                        <button id="generate-order-btn" class="w-full bg-primary text-white font-bold py-3 px-4 rounded-lg hover:opacity-90">Gerar Pedido</button>
                    </div>
                </div>
            </main>
        `;
        lucide.createIcons();

        document.getElementById('back-to-catalog-btn').onclick = () => {
            state.currentStep = 'catalog';
            render();
        };

        const cartContainer = document.getElementById('cart-items');
        const updateCart = () => {
            if (Object.keys(orderItems).length === 0) {
                cartContainer.innerHTML = `<p class="text-center text-secondary">O carrinho está vazio.</p>`;
                return;
            }
            cartContainer.innerHTML = '';
            Object.entries(orderItems).forEach(([id, qty]) => {
                const prod = state.products[id];
                const itemEl = document.createElement('div');
                itemEl.className = 'flex justify-between items-center text-sm';
                itemEl.innerHTML = `<span>${qty}x ${prod.name}</span> <span class="font-semibold">R$ ${(qty * prod.price).toFixed(2)}</span>`;
                cartContainer.appendChild(itemEl);
            });
        };

        document.querySelectorAll('.add-to-cart-btn').forEach(btn => {
            btn.onclick = (e) => {
                const prodId = e.currentTarget.dataset.prodId;
                showModal(`
                    <h3 class="text-xl font-bold mb-4">Adicionar ${state.products[prodId].name}</h3>
                    <div class="text-left">
                        <label class="block text-sm">Quantidade (fardos/caixas):</label>
                        <input id="add-qty" type="number" min="1" class="w-full p-2 border rounded" value="10">
                    </div>
                    <button id="confirm-add-btn" class="mt-4 w-full bg-highlight text-white font-bold py-2 px-4 rounded-lg">Confirmar</button>
                `);
                document.getElementById('confirm-add-btn').onclick = () => {
                    const qty = parseInt(document.getElementById('add-qty').value) || 0;
                    if (qty > 0) {
                        orderItems[prodId] = (orderItems[prodId] || 0) + qty;
                        updateCart();
                        closeModal();
                    } else {
                        alert("Insira uma quantidade válida.");
                    }
                };
            };
        });

        document.getElementById('generate-order-btn').onclick = () => {
            const customer = document.getElementById('customer-name').value;
            const address = document.getElementById('customer-address').value;
            const customerState = document.getElementById('customer-state').value;
            if (!customer || !address || Object.keys(orderItems).length === 0) {
                alert("Por favor, preencha todos os campos e adicione produtos ao pedido.");
                return;
            }
            const newOrder = {
                id: `PED-${Date.now()}`,
                customer: customer,
                address: { street: address, state: customerState },
                items: orderItems,
                status: 'pending'
            };
            state.orders.push(newOrder);
            state.currentStep = 'order_list';
            render();
        };
    }

    function renderOrderList() {
        const pendingOrders = state.orders.filter(o => o.status === 'pending');
        const dispatchedOrders = state.orders.filter(o => o.status === 'dispatched');
        const deliveredOrders = state.orders.filter(o => o.status === 'delivered');

        const createOrderCard = (order) => `
            <div class="bg-white p-4 rounded-lg border flex justify-between items-center">
                <div class="flex items-center gap-4">
                    ${order.status === 'pending' ? `<input type="checkbox" data-order-id="${order.id}" class="order-checkbox h-5 w-5 text-primary focus:ring-primary border-gray-300 rounded">` : ''}
                    <div>
                        <p class="font-bold">${order.id}</p>
                        <p class="text-sm text-secondary">${order.customer}</p>
                        <p class="text-sm text-secondary">${Object.keys(order.items).length} tipos de produto</p>
                    </div>
                </div>
            </div>
        `;

        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-2xl font-bold">Painel de Pedidos</h2>
                    <div class="flex gap-4">
                        <button id="go-to-create-order-btn-2" class="bg-action text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2"><i data-lucide="plus-circle"></i> Criar Novo Pedido</button>
                    </div>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <!-- A Separar -->
                    <div class="bg-gray-100 p-4 rounded-lg">
                        <h3 class="font-bold text-lg mb-4 text-center">A Separar (${pendingOrders.length})</h3>
                        <div class="space-y-3">${pendingOrders.length > 0 ? pendingOrders.map(createOrderCard).join('') : '<p class="text-center text-sm text-secondary">Nenhum pedido pendente.</p>'}</div>
                    </div>
                     <!-- Em Trânsito -->
                    <div class="bg-blue-50 p-4 rounded-lg">
                        <h3 class="font-bold text-lg mb-4 text-center">Em Trânsito (${dispatchedOrders.length})</h3>
                        <div class="space-y-3">${dispatchedOrders.length > 0 ? dispatchedOrders.map(createOrderCard).join('') : '<p class="text-center text-sm text-secondary">Nenhum pedido em trânsito.</p>'}</div>
                    </div>
                     <!-- Entregues -->
                    <div class="bg-green-50 p-4 rounded-lg">
                        <h3 class="font-bold text-lg mb-4 text-center">Entregues (${deliveredOrders.length})</h3>
                        <div class="space-y-3">${deliveredOrders.length > 0 ? deliveredOrders.map(createOrderCard).join('') : '<p class="text-center text-sm text-secondary">Nenhum pedido entregue.</p>'}</div>
                    </div>
                </div>
                <div class="mt-6 flex justify-between items-center">
                    <button id="select-all-btn" class="bg-gray-200 text-gray-700 font-bold py-3 px-6 rounded-lg flex items-center gap-2"><i data-lucide="check-square"></i> Marcar Todos</button>
                    <button id="start-shipment-btn" class="bg-highlight text-white font-bold py-3 px-6 rounded-lg flex items-center gap-2 disabled:opacity-50" disabled>
                        <i data-lucide="truck"></i> Iniciar Separação para Entrega
                    </button>
                </div>
            </main>
        `;
        lucide.createIcons();
        
        document.getElementById('go-to-create-order-btn-2').onclick = () => { state.currentStep = 'create_order'; render(); };
        
        orderIntervalId = setInterval(() => {
            const numNewOrders = Math.floor(Math.random() * 3) + 1;
            const productIds = Object.keys(state.products);
            const statesBR = ["AC", "AL", "AP", "AM", "BA", "CE", "DF", "ES", "GO", "MA", "MT", "MS", "MG", "PA", "PB", "PR", "PE", "PI", "RJ", "RN", "RS", "RO", "RR", "SC", "SP", "SE", "TO"];

            for(let i=0; i<numNewOrders; i++) {
                const numItems = Math.floor(Math.random() * 3) + 1;
                const items = {};
                const availableProducts = [...productIds];
                for(let j=0; j<numItems; j++) {
                    if(availableProducts.length === 0) break;
                    const productIndex = Math.floor(Math.random() * availableProducts.length);
                    const productId = availableProducts.splice(productIndex, 1)[0];
                    items[productId] = Math.floor(Math.random() * 50) + 10; // 10 to 60 units
                }

                const customerPrefixes = ["Distribuidora", "Atacadão", "Supermercado", "Comércio de Alimentos", "Empório"];
                const customerSuffixes = ["do Povo", "Central", "Norte Forte", "ABC", "Brasil"];
                const randomCustomer = `${customerPrefixes[Math.floor(Math.random() * customerPrefixes.length)]} ${customerSuffixes[Math.floor(Math.random() * customerSuffixes.length)]}`;

                const newOrder = {
                    id: `ECOMM-${Date.now() + i}`,
                    customer: randomCustomer,
                    address: { street: `Rua Virtual, ${Math.floor(Math.random() * 1000)}`, state: statesBR[Math.floor(Math.random() * statesBR.length)] },
                    items: items,
                    status: 'pending'
                };
                state.orders.push(newOrder);
            }
            if (state.currentStep === 'order_list') {
                render();
            }
        }, 10000);

        const startBtn = document.getElementById('start-shipment-btn');
        const checkboxes = document.querySelectorAll('.order-checkbox');
        
        document.getElementById('select-all-btn').onclick = () => {
            checkboxes.forEach(cb => cb.checked = true);
            startBtn.disabled = false;
        };

        checkboxes.forEach(cb => {
            cb.onchange = () => {
                const anyChecked = Array.from(checkboxes).some(c => c.checked);
                startBtn.disabled = !anyChecked;
            };
        });

        startBtn.onclick = () => {
            const selectedOrderIds = Array.from(checkboxes).filter(c => c.checked).map(c => c.dataset.orderId);
            state.currentShipment.orders = state.orders.filter(o => selectedOrderIds.includes(o.id));
            state.currentShipment.incidents = [];
            
            state.scannedItems = {};
            selectedOrderIds.forEach(id => { state.scannedItems[id] = []; });
            
            state.nonConformities = [];
            state.currentStep = 'scanning';
            render();
        };
    }

    function renderScanning() {
        let allItems = [];
        state.currentShipment.orders.forEach(order => {
            Object.entries(order.items).forEach(([itemCode, qty]) => {
                for (let i = 0; i < qty; i++) {
                    allItems.push({ orderId: order.id, itemCode: itemCode, instanceId: i });
                }
            });
        });

        let itemsHtml = allItems.map(({ orderId, itemCode, instanceId }) => {
            const product = state.products[itemCode];
            const order = state.currentShipment.orders.find(o => o.id === orderId);
            const totalQty = order.items[itemCode];
            return `
            <li id="item-${orderId}-${itemCode}-${instanceId}" class="item-to-scan p-3 bg-gray-100 rounded-lg border flex flex-col gap-2">
                <div class="flex items-center justify-between w-full">
                    <div>
                        <p class="font-semibold">${product.name} <span class="text-red-600 font-bold">(Unidade ${instanceId + 1}/${totalQty})</span></p>
                        <p class="text-xs text-blue-600 font-medium">Pedido: ${orderId}</p>
                    </div>
                    <i data-lucide="circle" class="text-secondary item-status-icon"></i>
                </div>
                <div class="bg-white p-2 rounded-md cursor-pointer hover:bg-blue-50 transition-colors barcode-container" data-barcode="${product.barcode}">
                    <img src="https://barcode.tec-it.com/barcode.ashx?data=${product.barcode}&code=Code128&dpi=200" alt="Código de Barras para ${product.name}" class="h-20 w-full object-contain pointer-events-none">
                    <p class="text-center text-xs text-secondary mt-1 pointer-events-none">${product.barcode}</p>
                </div>
            </li>
            `;
        }).join('');

        appContainer.innerHTML += `
            <main class="flex-grow p-8 grid grid-cols-1 md:grid-cols-2 gap-8">
                <div class="bg-white p-6 rounded-lg shadow-lg">
                    <h3 class="text-xl font-bold mb-4">Itens para separar - Entrega #${state.currentShipment.orders.map(o=>o.id).join(', ')}</h3>
                    <ul class="space-y-3 max-h-[60vh] overflow-y-auto pr-2">${itemsHtml}</ul>
                    <button id="finish-scan-btn" class="hidden mt-6 w-full bg-primary text-white font-bold py-3 px-4 rounded-lg hover:opacity-90">
                        Finalizar Separação e Gerar Romaneio
                    </button>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-lg text-center flex flex-col items-center justify-center">
                    <div id="product-display" class="w-48 h-48 bg-gray-200 rounded-lg flex items-center justify-center mb-4">
                        <i data-lucide="scan-line" class="w-24 h-24 text-secondary"></i>
                    </div>
                    <p id="product-name" class="font-bold text-lg h-8">Aguardando escaneamento...</p>
                    <div class="mt-6 w-full max-w-sm">
                        <label for="barcode-input" class="block text-sm font-medium text-gray-700 mb-1">Leitor de Código de Barras</label>
                        <div class="flex gap-2">
                            <input type="text" id="barcode-input" placeholder="Digite o código do produto..." class="flex-grow block w-full px-3 py-2 bg-white border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-primary focus:border-primary">
                            <button id="scan-btn" class="bg-highlight text-white font-bold py-2 px-4 rounded-lg hover:opacity-90 flex items-center gap-2">
                                <i data-lucide="scan-line"></i><span>Escanear</span>
                            </button>
                        </div>
                         <p class="text-xs text-center text-gray-500 mt-2">Dica: Clique em um código de barras para simular a leitura!</p>
                    </div>
                </div>
            </main>
        `;
        lucide.createIcons();
        
        const scanInput = document.getElementById('barcode-input');
        const scanBtn = document.getElementById('scan-btn');
        const productDisplay = document.getElementById('product-display');
        const productNameEl = document.getElementById('product-name');

        const updateScanUI = () => {
            const totalItemsToScan = allItems.length;
            const totalItemsScanned = Object.values(state.scannedItems).flat().length;
            if (totalItemsScanned >= totalItemsToScan) {
                document.getElementById('finish-scan-btn').classList.remove('hidden');
            }
        };

        const handleScan = () => {
            const scannedCode = scanInput.value.trim().toUpperCase();
            if (!scannedCode) return;

            let itemFound = false;
            for (const order of state.currentShipment.orders) {
                const requiredQty = order.items[scannedCode] || 0;
                const scannedQty = state.scannedItems[order.id].filter(i => i === scannedCode).length;

                if (requiredQty > scannedQty) {
                    const instanceId = scannedQty;
                    state.scannedItems[order.id].push(scannedCode);
                    const productData = state.products[scannedCode];
                    productDisplay.innerHTML = `<img src="${productData.img}" alt="${productData.name}" class="w-full h-full object-cover rounded-lg">`;
                    productNameEl.textContent = productData.name;
                    productDisplay.parentElement.classList.add('flash-success');
                    setTimeout(() => productDisplay.parentElement.classList.remove('flash-success'), 1000);

                    const itemLi = document.getElementById(`item-${order.id}-${scannedCode}-${instanceId}`);
                    if (itemLi) {
                        itemLi.classList.remove('bg-gray-100');
                        itemLi.classList.add('bg-green-100');
                        const iconElement = itemLi.querySelector('.item-status-icon');
                        if (iconElement) iconElement.outerHTML = `<i data-lucide="check-circle" class="text-highlight item-status-icon"></i>`;
                        lucide.createIcons();
                        setTimeout(() => itemLi.classList.add('item-scanned'), 200);
                    }
                    updateScanUI();
                    itemFound = true;
                    break; 
                }
            }

            if (!itemFound) {
                 productNameEl.textContent = "Item inválido ou já coletado!";
                showModal(`<h3 class="text-xl font-bold text-action">Erro de Separação</h3><p class="my-4">O item (${scannedCode}) não pertence a esta rota de separação ou todas as suas unidades já foram coletadas.</p>`);
            }
            scanInput.value = '';
            scanInput.focus();
        };

        scanBtn.addEventListener('click', handleScan);
        scanInput.addEventListener('keypress', (e) => e.key === 'Enter' && handleScan());
        document.querySelectorAll('.barcode-container').forEach(container => {
            container.addEventListener('click', () => {
                const barcode = container.dataset.barcode;
                scanInput.value = barcode;
                handleScan();
            });
        });
        document.getElementById('finish-scan-btn').onclick = () => { state.currentStep = 'romaneio'; render(); };
    }
    
    function renderRomaneio() {
        let totalWeight = 0;
        let totalValue = 0;
        let itemsHtml = '';
        state.currentShipment.orders.forEach(order => {
            itemsHtml += `<tr><td colspan="5" class="p-2 bg-blue-50 font-bold">Pedido: ${order.id} - Cliente: ${order.customer}</td></tr>`;
            Object.entries(order.items).forEach(([code, qty]) => {
                const product = state.products[code];
                totalWeight += product.weight * qty;
                totalValue += product.price * qty;
                itemsHtml += `
                <tr class="border-b">
                    <td class="p-2">${code}</td>
                    <td class="p-2">${product.name}</td>
                    <td class="p-2 text-center">${qty}</td>
                    <td class="p-2 text-center">${(product.weight * qty).toFixed(2)} kg</td>
                    <td class="p-2 text-center text-highlight"><i data-lucide="check"></i></td>
                </tr>
                `;
            });
        });
        
        state.currentShipment.totalWeight = totalWeight;
        state.currentShipment.totalValue = totalValue;

        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="bg-white p-6 rounded-lg shadow-lg max-w-4xl mx-auto">
                    <h2 class="text-2xl font-bold mb-2">Romaneio Digital - Rota de Entrega</h2>
                    <table id="romaneio-table" class="w-full text-left mt-4">
                        <thead><tr class="bg-gray-100"><th class="p-2">SKU</th><th class="p-2">Produto</th><th class="p-2 text-center">Qtd.</th><th class="p-2 text-center">Peso</th><th class="p-2 text-center">Status</th></tr></thead>
                        <tbody>${itemsHtml}</tbody>
                    </table>
                    <div class="mt-6 flex justify-between items-center">
                        <div>
                            <p><strong>Total de Pedidos:</strong> ${state.currentShipment.orders.length}</p>
                            <p><strong>Peso Total da Carga:</strong> ${totalWeight.toFixed(2)} kg</p>
                        </div>
                        <div class="flex gap-4">
                            <button id="print-orders-btn" class="bg-gray-500 text-white font-bold py-2 px-4 rounded-lg hover:opacity-90 flex items-center gap-2"><i data-lucide="file-text"></i> Imprimir Pedidos</button>
                            <button id="print-romaneio-btn" class="bg-gray-500 text-white font-bold py-2 px-4 rounded-lg hover:opacity-90 flex items-center gap-2"><i data-lucide="file-spreadsheet"></i> Imprimir Romaneio</button>
                            <button id="route-btn" class="bg-primary text-white font-bold py-2 px-4 rounded-lg hover:opacity-90 flex items-center gap-2">Planejar Rota <i data-lucide="arrow-right"></i></button>
                        </div>
                    </div>
                </div>
            </main>
        `;
        lucide.createIcons();
        document.getElementById('print-orders-btn').onclick = printOrders;
        document.getElementById('print-romaneio-btn').onclick = printRomaneio;
        document.getElementById('route-btn').onclick = () => { state.currentStep = 'routing'; render(); };
    }

    function renderRouting() {
        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="bg-white p-6 rounded-lg shadow-lg">
                    <h2 class="text-2xl font-bold mb-4">Roteirização e Cálculo de Distância</h2>
                    <div id="routing-content">
                        <!-- Content will be injected by initRoutingMap or the permission modal -->
                    </div>
                </div>
            </main>
        `;
        initRoutingMap();
    }

    function initRoutingMap() {
        const defaultLocation = [-10.1845, -48.3336]; // Palmas, TO
        const routingContent = document.getElementById('routing-content');
        if (!routingContent) return;

        routingContent.innerHTML = `
            <p class="text-secondary mb-4">O mapa exibe a rota do armazém até os clientes. A distância total é calculada para o frete.</p>
            <div class="flex flex-col md:flex-row gap-8">
                <div id="map" class="w-full md:w-2/3"></div>
                <div class="w-full md:w-1/3">
                    <h3 class="font-bold mb-2">Pontos da Rota:</h3>
                    <ol id="route-list" class="list-decimal list-inside space-y-2"></ol>
                    <div id="distance-summary" class="mt-4 font-semibold"></div>
                    <button id="confirm-route-btn" class="hidden mt-6 w-full bg-primary text-white font-bold py-3 px-4 rounded-lg">
                        Calcular Frete e Conferir
                    </button>
                </div>
            </div>
        `;

        const map = L.map('map');
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

        const routeListEl = document.getElementById('route-list');
        const distanceSummaryEl = document.getElementById('distance-summary');
        const confirmBtn = document.getElementById('confirm-route-btn');

        const warehouseIcon = L.icon({ iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-blue.png', shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png', iconSize: [25, 41], iconAnchor: [12, 41], popupAnchor: [1, -34], shadowSize: [41, 41] });
        const customerIcon = L.icon({ iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png', shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png', iconSize: [25, 41], iconAnchor: [12, 41], popupAnchor: [1, -34], shadowSize: [41, 41] });

        const setupMap = (lat, lon) => {
            state.userLocation = [lat, lon];
            map.setView([lat, lon], 5);
            
            const routePoints = [{ lat, lon, title: 'Nosso Armazém', icon: warehouseIcon }];
            
            state.currentShipment.orders.forEach((order, i) => {
                const angle = Math.random() * 2 * Math.PI;
                const distanceInDegrees = 5 + (Math.random() * 13); // Approx 555km to 2000km
                const customerLat = lat + distanceInDegrees * Math.sin(angle);
                const customerLon = lon + distanceInDegrees * Math.cos(angle) / Math.cos(lat * Math.PI / 180);
                routePoints.push({ lat: customerLat, lon: customerLon, title: order.customer, icon: customerIcon });
            });
            
            state.currentShipment.route.points = routePoints;

            let totalDistance = 0;
            let lastPoint = routePoints[0];
            
            routeListEl.innerHTML = `<li>${lastPoint.title} (Partida)</li>`;
            L.marker([lastPoint.lat, lastPoint.lon], { icon: lastPoint.icon }).addTo(map).bindPopup(lastPoint.title);

            for(let i = 1; i < routePoints.length; i++) {
                const currentPoint = routePoints[i];
                const legDistance = calculateDistance(lastPoint.lat, lastPoint.lon, currentPoint.lat, currentPoint.lon);
                totalDistance += legDistance;

                routeListEl.innerHTML += `<li>${currentPoint.title} (+${legDistance.toFixed(1)} km)</li>`;
                L.marker([currentPoint.lat, currentPoint.lon], { icon: currentPoint.icon }).addTo(map).bindPopup(currentPoint.title);
                L.polyline([[lastPoint.lat, lastPoint.lon], [currentPoint.lat, currentPoint.lon]], {color: 'var(--color-primary)', dashArray: '5, 10'}).addTo(map);

                lastPoint = currentPoint;
            }

            state.currentShipment.route.totalDistance = totalDistance;
            distanceSummaryEl.innerHTML = `Distância Total da Rota: <span class="text-primary text-xl">${totalDistance.toFixed(1)} km</span>`;
            
            const group = new L.featureGroup(routePoints.map(p => L.marker([p.lat, p.lon])));
            map.fitBounds(group.getBounds().pad(0.2));

            confirmBtn.classList.remove('hidden');
        };

        if (state.userLocation) {
             setupMap(state.userLocation[0], state.userLocation[1]);
        } else if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(
                (position) => setupMap(position.coords.latitude, position.coords.longitude),
                () => {
                    showModal("Não foi possível obter sua localização. Usando uma localização padrão em Palmas-TO.");
                    setupMap(defaultLocation[0], defaultLocation[1]);
                }
            );
        } else {
            setupMap(defaultLocation[0], defaultLocation[1]);
        }
        
        confirmBtn.onclick = () => { state.currentStep = 'freight_check'; render(); };
    }

    function renderFreightCheck() {
        const { totalWeight, totalValue, orders, route } = state.currentShipment;
        const distance = route.totalDistance;
        const freightCost = 50 + (totalWeight * 0.75) + (distance * 1.1); // Base + weight + distance
        const homeState = 'TO';
        let icms = 0;

        let ordersSummaryHtml = orders.map(order => {
            const orderValue = Object.entries(order.items).reduce((acc, [code, qty]) => acc + (state.products[code].price * qty), 0);
            const icmsRate = ICMS_RATES[order.address.state] || 0.12; // Default to 12% if not found
            let orderIcms = 0;
            if (order.address.state !== homeState) {
                orderIcms = orderValue * icmsRate;
                icms += orderIcms;
            }
            return `<p class="text-sm"> - Pedido ${order.id} (${order.address.state}): ${Object.keys(order.items).length} tipos de produto. ${orderIcms > 0 ? `ICMS (${(icmsRate * 100).toFixed(0)}%): R$ ${orderIcms.toFixed(2)}` : 'ICMS: Isento'}</p>`;
        }).join('');

        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="bg-white p-6 rounded-lg shadow-lg max-w-2xl mx-auto">
                    <h2 class="text-2xl font-bold mb-4">Conferência Final e Frete</h2>
                    <div class="space-y-4 divide-y">
                        <div class="pt-2">
                            <h3 class="font-semibold">Resumo da Rota</h3>
                            ${ordersSummaryHtml}
                        </div>
                        <div class="pt-4">
                            <h3 class="font-semibold">Cálculo de Frete Realista</h3>
                            <p>Valor dos Produtos: <span class="font-bold">R$ ${totalValue.toFixed(2)}</span></p>
                            <p>Peso Total: <span class="font-bold">${totalWeight.toFixed(2)} kg</span></p>
                            <p>Distância da Rota: <span class="font-bold">${distance.toFixed(1)} km</span></p>
                            <p>Custo do Frete: <span class="font-bold">R$ ${freightCost.toFixed(2)}</span></p>
                            <p>ICMS (Interestadual): <span class="font-bold text-red-600">R$ ${icms.toFixed(2)}</span></p>
                            <p class="text-2xl text-highlight font-bold mt-2">Custo Total da Operação: R$ ${(totalValue + freightCost + icms).toFixed(2)}</p>
                        </div>
                    </div>
                    <button id="dispatch-btn" class="mt-8 w-full bg-primary text-white font-bold py-3 px-4 rounded-lg hover:opacity-90 flex items-center justify-center gap-2">
                        <i data-lucide="truck"></i> Despachar Veículo para Entrega
                    </button>
                </div>
            </main>
        `;
        lucide.createIcons();
        document.getElementById('dispatch-btn').onclick = () => {
            state.currentShipment.orders.forEach(order => {
                const orderInState = state.orders.find(o => o.id === order.id);
                if(orderInState) orderInState.status = 'dispatched';
            });
            state.currentStep = 'delivery_tracking';
            render();
        };
    }

    function renderDeliveryTracking() {
        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="bg-white p-6 rounded-lg shadow-lg relative">
                    <h2 class="text-2xl font-bold mb-4">Rastreamento da Entrega</h2>
                    <p class="text-secondary mb-4">Acompanhe o percurso da rota.</p>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                        <div id="map" class="w-full md:col-span-2"></div>
                        <div id="timeline-container" class="max-h-[450px] overflow-y-auto">
                            <h3 class="font-bold mb-2">Histórico de Eventos</h3>
                            <ul id="timeline-list" class="space-y-3"></ul>
                        </div>
                    </div>
                    <div id="tracking-status" class="mt-4 text-center font-semibold text-lg text-primary"></div>
                    <button id="follow-truck-btn" class="absolute top-20 right-4 z-20 bg-white p-2 rounded-full shadow-lg text-primary hover:bg-gray-100">
                        <i data-lucide="locate-fixed"></i>
                    </button>
                </div>
            </main>
        `;
        initTrackingMap();
    }

    function initTrackingMap() {
        const mapContainer = document.getElementById('map');
        if (!mapContainer) return;

        const map = L.map('map');
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);
        
        const statusEl = document.getElementById('tracking-status');
        const followBtn = document.getElementById('follow-truck-btn');
        const timelineList = document.getElementById('timeline-list');
        let isFollowing = false;
        let followInterval;
        
        const truckIcon = L.icon({ iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png', shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png', iconSize: [25, 41], iconAnchor: [12, 41], popupAnchor: [1, -34], shadowSize: [41, 41] });
        const problemIcon = L.icon({ iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-orange.png', shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png', iconSize: [25, 41], iconAnchor: [12, 41], popupAnchor: [1, -34], shadowSize: [41, 41] });

        let simulatedClock = new Date();

        const addTimelineEvent = (iconName, text, colorClass = 'text-gray-500') => {
            const time = simulatedClock.toLocaleString('pt-BR', { day: '2-digit', month: '2-digit', hour: '2-digit', minute: '2-digit' });
            const li = document.createElement('li');
            li.className = 'flex items-start gap-3 text-sm';
            li.innerHTML = `
                <i data-lucide="${iconName}" class="w-5 h-5 mt-1 ${colorClass}"></i>
                <div>
                    <p class="font-semibold ${colorClass}">${text}</p>
                    <p class="text-xs text-gray-400">${time}</p>
                </div>
            `;
            timelineList.prepend(li);
            lucide.createIcons();
        };

        const routePoints = state.currentShipment.route.points;
        const routeLatLngs = routePoints.map(p => {
            const latLng = L.latLng(p.lat, p.lon);
            L.marker(latLng, { icon: p.icon }).addTo(map).bindPopup(p.title);
            return latLng;
        });

        const realisticPath = [];
        for (let i = 0; i < routeLatLngs.length - 1; i++) {
            const start = routeLatLngs[i];
            const end = routeLatLngs[i+1];
            realisticPath.push(start);
            // Add intermediate points to simulate turns
            const mid1 = L.latLng(start.lat, (start.lng + end.lng) / 2);
            const mid2 = L.latLng(end.lat, (start.lng + end.lng) / 2);
            realisticPath.push(mid1, mid2);
        }
        realisticPath.push(routeLatLngs[routeLatLngs.length - 1]);

        const routeLine = L.polyline(realisticPath, {color: 'var(--color-primary)', dashArray: '5, 10'}).addTo(map);
        map.fitBounds(routeLine.getBounds().pad(0.2));

        let animatedMarker;

        followBtn.onclick = () => {
            isFollowing = !isFollowing;
            followBtn.classList.toggle('bg-highlight', isFollowing);
            followBtn.classList.toggle('text-white', isFollowing);
            if (isFollowing) {
                if (animatedMarker) map.panTo(animatedMarker.getLatLng());
                followInterval = setInterval(() => {
                    if (animatedMarker && isFollowing) {
                       map.panTo(animatedMarker.getLatLng());
                    }
                }, 500);
            } else {
                clearInterval(followInterval);
            }
        };
        
        addTimelineEvent('package-search', 'Caminhão carregando no armazém');
        statusEl.innerHTML = `<img src="https://placehold.co/300x200/ECEFF1/37474F?text=Caminhão+Carregando" class="mx-auto rounded-lg shadow-md">`;

        setTimeout(() => {
            addTimelineEvent('truck', 'Veículo saiu para entrega', 'text-primary');
            animateLeg(0);
        }, 3000); // Loading simulation

        function animateLeg(legIndex) {
            if (legIndex >= routePoints.length - 1) {
                clearInterval(followInterval);
                statusEl.textContent = "Rota finalizada!";
                addTimelineEvent('flag', 'Rota finalizada com sucesso', 'text-highlight');
                const actionsContainer = document.getElementById('tracking-status');
                actionsContainer.innerHTML = `
                    <div class="text-center">
                        <p class="font-semibold text-lg text-highlight">Rota Finalizada!</p>
                        <button id="go-to-delivery-btn" class="mt-4 bg-primary text-white font-bold py-3 px-6 rounded-lg flex items-center gap-2 mx-auto">
                            <i data-lucide="signature"></i> Coletar Assinatura da Entrega
                        </button>
                    </div>
                `;
                lucide.createIcons();
                document.getElementById('go-to-delivery-btn').onclick = () => {
                    state.currentStep = 'delivery';
                    render();
                };
                return;
            }

            const startPoint = routePoints[legIndex];
            const endPoint = routePoints[legIndex + 1];
            const destination = state.currentShipment.orders[legIndex];
            
            const legDistance = calculateDistance(startPoint.lat, startPoint.lon, endPoint.lat, endPoint.lon);
            const animationInterval = 20000; // Fixed 20 seconds for animation
            const travelDays = Math.ceil(legDistance / 500); // 500km per day

            if (animatedMarker) map.removeLayer(animatedMarker);
            
            animatedMarker = L.animatedMarker([[startPoint.lat, startPoint.lon], [endPoint.lat, endPoint.lon]], {
                icon: truckIcon,
                interval: animationInterval,
                autostart: true,
                onEnd: () => {
                    simulatedClock.setDate(simulatedClock.getDate() + travelDays);
                    statusEl.textContent = `Chegou ao destino: ${destination.customer}.`;
                    addTimelineEvent('map-pin', `Chegou ao destino: ${destination.customer}`, 'text-highlight');
                    setTimeout(() => animateLeg(legIndex + 1), 2000);
                }
            });
            map.addLayer(animatedMarker);
            statusEl.innerHTML = `Veículo a caminho de: <span class="font-bold">${destination.customer}</span>`;

            setTimeout(() => {
                if (Math.random() < 0.4) {
                    animatedMarker.stop();
                    animatedMarker.setIcon(problemIcon);
                    const incidentTypes = ["Cliente ausente", "Endereço não localizado", "Entregador parado", "Entrega recusada", "Parada para almoço", "Parada para abastecer"];
                    const incidentType = incidentTypes[Math.floor(Math.random() * incidentTypes.length)];
                    statusEl.innerHTML = `<span class="text-action">Problema na Entrega: ${incidentType}</span>`;
                    addTimelineEvent('alert-triangle', `Incidente: ${incidentType}`, 'text-action');

                    showModal(`
                        <h3 class="text-xl font-bold text-action">Alerta de Incidente!</h3>
                        <p class="my-2">Ocorreu um problema na entrega para <span class="font-bold">${destination.customer}</span>.</p>
                        <p class="font-semibold">${incidentType}</p>
                        <div class="text-left mt-4">
                            <label class="block text-sm">Detalhes da ocorrência:</label>
                            <textarea id="incident-details" class="w-full p-2 border rounded" rows="3" placeholder="Descreva o que aconteceu..."></textarea>
                        </div>
                        <button id="register-incident-btn" class="mt-4 w-full bg-action text-white font-bold py-2 px-4 rounded-lg">Registrar e Continuar Rota</button>
                    `);
                    document.getElementById('register-incident-btn').onclick = () => {
                        const details = document.getElementById('incident-details').value;
                        const incident = { type: incidentType, details: details, customer: destination.customer, timestamp: Date.now() };
                        state.currentShipment.incidents.push(incident);
                        
                        animatedMarker.setIcon(truckIcon);
                        animatedMarker.start();
                        closeModal();
                    };
                }
            }, Math.random() * (animationInterval * 0.8) + (animationInterval * 0.1));
        }
        lucide.createIcons();
    }

    function renderDelivery() {
        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="bg-white p-6 rounded-lg shadow-lg max-w-2xl mx-auto">
                    <h2 class="text-2xl font-bold mb-4">Baixa de Entrega</h2>
                    <p class="text-secondary mb-4">Registre a assinatura do último cliente para finalizar a rota.</p>
                    <div id="delivery-proof" class="space-y-4">
                        <button id="photo-btn" class="w-full p-4 border-2 border-dashed rounded-lg flex items-center justify-center gap-2 text-secondary hover:bg-gray-50"><i data-lucide="camera"></i> Clicar para simular Foto da Entrega</button>
                        <div>
                            <p class="mb-2 font-medium">Assinatura Digital do Recebedor:</p>
                            <canvas id="signature-canvas" width="500" height="200" class="bg-white rounded-lg w-full"></canvas>
                            <button id="clear-signature-btn" class="text-sm text-secondary hover:underline mt-1">Limpar</button>
                        </div>
                    </div>
                    <div class="flex gap-4 mt-6">
                        <button id="print-delivery-proof-btn" class="w-full bg-gray-500 text-white font-bold py-3 px-4 rounded-lg hover:opacity-90 flex items-center justify-center gap-2"><i data-lucide="printer"></i> Imprimir Comprovante</button>
                        <button id="confirm-delivery-btn" class="w-full bg-highlight text-white font-bold py-3 px-4 rounded-lg hover:opacity-90 flex items-center justify-center gap-2"><i data-lucide="check-check"></i> Baixar Entrega no Sistema</button>
                    </div>
                </div>
            </main>
        `;
        lucide.createIcons();
        const canvas = document.getElementById('signature-canvas');
        const ctx = canvas.getContext('2d');
        let drawing = false;
        const startDrawing = (e) => { drawing = true; draw(e); };
        const stopDrawing = () => { drawing = false; ctx.beginPath(); };
        const draw = (e) => {
            if (!drawing) return;
            e.preventDefault();
            const rect = canvas.getBoundingClientRect();
            const x = (e.clientX || e.touches[0].clientX) - rect.left;
            const y = (e.clientY || e.touches[0].clientY) - rect.top;
            ctx.lineWidth = 3; ctx.lineCap = 'round'; ctx.strokeStyle = '#000';
            ctx.lineTo(x, y); ctx.stroke(); ctx.beginPath(); ctx.moveTo(x, y);
        };
        canvas.addEventListener('mousedown', startDrawing);
        canvas.addEventListener('mouseup', stopDrawing);
        canvas.addEventListener('mousemove', draw);
        canvas.addEventListener('touchstart', startDrawing);
        canvas.addEventListener('touchend', stopDrawing);
        canvas.addEventListener('touchmove', draw);
        document.getElementById('clear-signature-btn').onclick = () => ctx.clearRect(0, 0, canvas.width, canvas.height);
        document.getElementById('photo-btn').onclick = () => showModal(`<i data-lucide="camera" class="w-16 h-16 text-highlight mx-auto mb-4"></i><p>Simulação: Imagem da entrega capturada com sucesso!</p>`);
        document.getElementById('print-delivery-proof-btn').onclick = printDeliveryProof;
        document.getElementById('confirm-delivery-btn').onclick = () => { 
            // Update stock and delivered items
            const successfulCustomers = new Set(state.currentShipment.orders.map(o => o.customer));
            state.currentShipment.incidents.forEach(inc => successfulCustomers.delete(inc.customer));

            state.currentShipment.orders.forEach(order => {
                const orderInState = state.orders.find(o => o.id === order.id);
                if (successfulCustomers.has(order.customer)) {
                    Object.entries(order.items).forEach(([itemCode, qty]) => {
                        if(state.products[itemCode]) {
                           state.products[itemCode].stock -= qty;
                        }
                        for(let i=0; i<qty; i++) {
                            state.deliveredItems.push(itemCode);
                        }
                    });
                    if(orderInState) orderInState.status = 'delivered';
                }
            });

            state.currentStep = 'quiz';
            render(); 
        };
    }
    
    function renderQuiz() {
        appContainer.innerHTML += `<main class="flex-grow p-8"><div class="bg-white p-6 rounded-lg shadow-lg max-w-4xl mx-auto"><h2 class="text-2xl font-bold mb-4">Avaliação de Conhecimento</h2><p class="text-secondary mb-6">Parabéns! Teste seus conhecimentos.</p><div id="quiz-content"></div></div></main>`;
        const quizData = [
            { type: 'quiz', q: 'Qual o objetivo principal do romaneio?', a: 'Listar todos os itens de uma carga para conferência.', opts: ['Calcular o frete', 'Listar todos os itens de uma carga para conferência.', 'Ser a nota fiscal do produto'] },
            { type: 'quiz', q: 'O que representa a baixa da entrega?', a: 'A confirmação oficial de que o cliente recebeu o produto.', opts: ['O fim do turno do motorista', 'A confirmação oficial de que o cliente recebeu o produto.', 'O pagamento do pedido'] },
        ];
        let currentQuizIndex = 0;
        const quizContainer = document.getElementById('quiz-content');
        function showQuestion() {
            const item = quizData[currentQuizIndex];
            if (!item) {
                quizContainer.innerHTML = `<div class="text-center"><i data-lucide="graduation-cap" class="w-20 h-20 text-highlight mx-auto mb-4"></i><h3 class="text-2xl font-bold">Treinamento Básico Concluído!</h3><p class="my-4">Você completou o treinamento inicial. Agora, prepare-se para um desafio final de gestão.</p><button id="start-final-project-btn" class="bg-primary text-white font-bold py-3 px-6 rounded-lg">Iniciar Desafio Final</button></div>`;
                lucide.createIcons();
                document.getElementById('start-final-project-btn').onclick = () => {
                    state.currentStep = 'final_challenge_quiz';
                    render();
                };
                return;
            }
            if (item.type === 'quiz') {
                let optionsHtml = item.opts.map(opt => `<button class="quiz-option w-full text-left p-3 bg-gray-100 hover:bg-gray-200 rounded-md">${opt}</button>`).join('');
                quizContainer.innerHTML = `<h4 class="font-bold text-lg mb-3">${item.q}</h4><div class="space-y-2">${optionsHtml}</div>`;
                quizContainer.querySelectorAll('.quiz-option').forEach(btn => {
                    btn.onclick = () => {
                        if (btn.textContent === item.a) {
                            btn.classList.add('bg-highlight', 'text-white');
                            setTimeout(() => { currentQuizIndex++; showQuestion(); }, 1000);
                        } else {
                            btn.classList.add('bg-action', 'text-white');
                        }
                    };
                });
            }
        }
        showQuestion();
    }
    
    function renderFinalChallengeQuiz() {
        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="bg-white p-8 rounded-lg shadow-lg max-w-3xl mx-auto">
                    <i data-lucide="clipboard-edit" class="w-20 h-20 mx-auto text-primary mb-4"></i>
                    <h2 class="text-3xl font-bold mb-4">Desafio Final de Gestão</h2>
                    <p class="text-secondary mb-6">Como gestor de logística, você se deparou com as seguintes situações hoje. Descreva o plano de ação para cada uma delas.</p>
                    
                    <div class="space-y-6 text-left">
                        <div>
                            <label for="challenge1" class="font-bold block mb-2">Situação 1: Produto Danificado</label>
                            <p class="text-sm text-gray-600 mb-2">Um cliente ligou informando que uma caixa de "Produto A" chegou amassada e o conteúdo foi danificado. A entrega foi assinada como recebida sem ressalvas.</p>
                            <textarea id="challenge1" class="w-full p-2 border rounded-md" rows="4" placeholder="Seu plano de ação..."></textarea>
                        </div>
                        <div>
                            <label for="challenge2" class="font-bold block mb-2">Situação 2: Entregador Parado</label>
                            <p class="text-sm text-gray-600 mb-2">O sistema de rastreamento mostra que o veículo da Rota 02 está parado no mesmo local há mais de 1 hora, e o motorista não atende o rádio ou o celular.</p>
                            <textarea id="challenge2" class="w-full p-2 border rounded-md" rows="4" placeholder="Seu plano de ação..."></textarea>
                        </div>
                    </div>

                    <button id="finish-sim-btn" class="mt-8 w-full bg-highlight text-white font-bold py-3 px-6 rounded-lg text-lg">Finalizar Simulação</button>
                </div>
            </main>
        `;
        lucide.createIcons();
        document.getElementById('finish-sim-btn').onclick = () => {
             state.currentStep = 'final_report_dashboard';
             render();
        };
    }

    function renderFinalReportDashboard() {
        const dispatchedOrders = state.orders.filter(o => o.status === 'dispatched');
        const deliveredOrders = state.orders.filter(o => o.status === 'delivered');
        
        const createReportCard = (order) => `
            <div class="bg-white p-3 rounded-lg border">
                <p class="font-bold text-sm">${order.id}</p>
                <p class="text-xs text-secondary">${order.customer}</p>
            </div>
        `;

        const stockHtml = Object.entries(state.products).map(([id, prod]) => {
            const stockLevelColor = prod.stock > 50 ? 'text-green-600' : prod.stock > 10 ? 'text-yellow-600' : 'text-red-600';
            return `<li class="flex justify-between items-center text-sm p-2 odd:bg-gray-50"><span>${prod.name}</span> <span class="font-bold ${stockLevelColor}">${prod.stock} un.</span></li>`;
        }).join('');

        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="bg-white p-8 rounded-lg shadow-lg max-w-6xl mx-auto">
                    <h2 class="text-3xl font-bold mb-4">Relatório Final da Operação</h2>
                    <p class="text-secondary mb-6">Resumo do turno do operador ${state.userName}.</p>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                        <!-- Pedidos Enviados -->
                        <div class="bg-orange-50 p-4 rounded-lg">
                            <h3 class="font-bold text-lg mb-4 text-center">Pedidos com Incidentes</h3>
                            <div class="space-y-3 max-h-96 overflow-y-auto pr-2">${dispatchedOrders.length > 0 ? dispatchedOrders.map(createReportCard).join('') : '<p class="text-center text-sm text-secondary">Nenhum.</p>'}</div>
                        </div>
                        <!-- Pedidos Entregues -->
                        <div class="bg-green-50 p-4 rounded-lg">
                            <h3 class="font-bold text-lg mb-4 text-center">Pedidos Entregues com Sucesso</h3>
                            <div class="space-y-3 max-h-96 overflow-y-auto pr-2">${deliveredOrders.length > 0 ? deliveredOrders.map(createReportCard).join('') : '<p class="text-center text-sm text-secondary">Nenhum.</p>'}</div>
                        </div>
                        <!-- Estoque Final -->
                        <div class="bg-gray-100 p-4 rounded-lg">
                            <h3 class="font-bold text-lg mb-4 text-center">Estoque Final</h3>
                            <ul class="space-y-1 max-h-96 overflow-y-auto">${stockHtml}</ul>
                        </div>
                    </div>
                     <button id="restart-sim-btn" class="mt-8 w-full bg-primary text-white font-bold py-3 px-4 rounded-lg hover:opacity-90">Reiniciar Simulação</button>
                     <p class="text-xs text-gray-400 mt-6 text-center">Desenvolvido por Leanderson Teixeira Reis<br>Contato: leandersonteixeiraof@gmail.com</p>
                </div>
            </main>
        `;
        document.getElementById('restart-sim-btn').onclick = () => {
            // Reset state completely
            Object.assign(state, {
                currentStep: 'login', userName: '', userShift: '',
                products: JSON.parse(JSON.stringify(initialData.products)),
                orders: JSON.parse(JSON.stringify(initialData.orders)),
                deliveredItems: [],
                currentShipment: { orders: [], totalWeight: 0, totalValue: 0, incidents: [], route: { totalDistance: 0, points: [] } },
                scannedItems: {}, nonConformities: [],
                userLocation: null,
            });
            render();
        };
    }

    // --- PRINT FUNCTIONS ---
    function printContent(content) {
        const printArea = document.getElementById('print-area');
        printArea.innerHTML = content;
        window.print();
    }

    function printOrders() {
        let content = `<h1>Pedidos da Rota</h1>`;
        state.currentShipment.orders.forEach(order => {
            content += `
                <div class="order-block" style="margin-top: 20px; border: 1px solid #ccc; padding: 10px; page-break-inside: avoid;">
                    <h2>Pedido: ${order.id}</h2>
                    <p>Cliente: ${order.customer}</p>
                    <p>Endereço: ${order.address.street}, ${order.address.state}</p>
                    <h3>Itens:</h3>
                    <ul>
                        ${Object.entries(order.items).map(([code, qty]) => `<li>${qty}x ${state.products[code].name} (SKU: ${code})</li>`).join('')}
                    </ul>
                </div>
            `;
        });
        printContent(content);
    }

    function printRomaneio() {
        let itemsHtml = '';
        state.currentShipment.orders.forEach(order => {
            itemsHtml += `<tr><td colspan="5" style="background-color: #eef; font-weight: bold; padding: 8px;">Pedido: ${order.id} - Cliente: ${order.customer}</td></tr>`;
            Object.entries(order.items).forEach(([code, qty]) => {
                const product = state.products[code];
                itemsHtml += `
                <tr>
                    <td>${code}</td>
                    <td>${product.name}</td>
                    <td style="text-align: center;">${qty}</td>
                    <td style="text-align: center;">${(product.weight * qty).toFixed(2)} kg</td>
                    <td style="text-align: center;">OK</td>
                </tr>
                `;
            });
        });

        let content = `
            <h1>Romaneio da Carga</h1>
            <p><strong>Operador:</strong> ${state.userName}</p>
            <p><strong>Data:</strong> ${new Date().toLocaleDateString('pt-BR')}</p>
            <table border="1" style="width: 100%; border-collapse: collapse; margin-top: 10px;">
                <thead>
                    <tr style="background-color: #f2f2f2;">
                        <th style="padding: 8px;">SKU</th>
                        <th style="padding: 8px;">Produto</th>
                        <th style="padding: 8px;">Qtd.</th>
                        <th style="padding: 8px;">Peso</th>
                        <th style="padding: 8px;">Status</th>
                    </tr>
                </thead>
                <tbody>${itemsHtml}</tbody>
            </table>
        `;
        printContent(content);
    }

    function printDeliveryProof() {
        const canvas = document.getElementById('signature-canvas');
        const signatureImg = canvas.toDataURL('image/png');
        const lastOrder = state.currentShipment.orders[state.currentShipment.orders.length - 1];

        let content = `
            <h1>Comprovante de Entrega</h1>
            <p><strong>Data:</strong> ${new Date().toLocaleString('pt-BR')}</p>
            <p><strong>Cliente:</strong> ${lastOrder.customer}</p>
            <p><strong>Pedido(s):</strong> ${state.currentShipment.orders.map(o => o.id).join(', ')}</p>
            <div class="signature-block" style="margin-top: 20px; border: 1px solid #ccc; padding: 10px;">
                <p>Recebido por:</p>
                <img src="${signatureImg}" alt="Assinatura" style="width: 300px; height: auto; border: 1px solid #eee;">
                <p style="margin-top: 20px;">_________________________________________</p>
                <p>Assinatura do Recebedor</p>
            </div>
        `;
        printContent(content);
    }

    function downloadQualityForm() {
        const content = `
            <style>
                body { font-family: sans-serif; font-size: 12px; }
                .print-container { border: 1px solid #000; padding: 20px; margin: 20px; }
                h1 { font-size: 18px; text-align: center; margin-bottom: 20px; }
                table { width: 100%; border-collapse: collapse; margin-top: 15px; }
                th, td { border: 1px solid #ccc; padding: 8px; text-align: left; }
                th { background-color: #f2f2f2; }
                .field-label { font-weight: bold; }
                .empty-field { height: 25px; border-bottom: 1px solid #555; }
                .notes-field { height: 100px; border: 1px solid #555; padding: 5px; }
                .signature-field { margin-top: 50px; border-top: 1px solid #555; width: 250px; text-align: center; }
            </style>
            <div class="print-container">
                <h1>Formulário de Controle de Qualidade</h1>
                <table>
                    <tr>
                        <td class="field-label">Operador:</td>
                        <td>${state.userName}</td>
                        <td class="field-label">Data/Hora:</td>
                        <td>${new Date().toLocaleString('pt-BR')}</td>
                    </tr>
                    <tr>
                        <td class="field-label">ID do Pedido:</td>
                        <td colspan="3"><div class="empty-field"></div></td>
                    </tr>
                    <tr>
                        <td class="field-label">Cliente:</td>
                        <td colspan="3"><div class="empty-field"></div></td>
                    </tr>
                     <tr>
                        <td class="field-label">Produto(s):</td>
                        <td colspan="3"><div class="empty-field"></div></td>
                    </tr>
                    <tr>
                        <td class="field-label">Peso Verificado (kg):</td>
                        <td><div class="empty-field"></div></td>
                        <td class="field-label">Valor Declarado (R$):</td>
                        <td><div class="empty-field"></div></td>
                    </tr>
                     <tr>
                        <td class="field-label">Tempo de Entrega (dias):</td>
                        <td><div class="empty-field"></div></td>
                        <td class="field-label">Status:</td>
                        <td>( ) Aprovado ( ) Reprovado</td>
                    </tr>
                    <tr>
                        <td class="field-label" colspan="4">Observações:</td>
                    </tr>
                     <tr>
                        <td colspan="4"><div class="notes-field"></div></td>
                    </tr>
                </table>
                <div style="display: flex; justify-content: flex-end; margin-top: 60px;">
                     <div class="signature-field">
                        Assinatura do Responsável
                     </div>
                </div>
            </div>
        `;
        printContent(content);
    }


    // --- MODAL FUNCTIONS ---
    const showModal = (htmlContent) => {
        const overlay = document.getElementById('modal-overlay');
        const content = document.getElementById('modal-content');
        content.innerHTML = htmlContent;
        lucide.createIcons();
        overlay.classList.remove('hidden');
        if (!content.querySelector('.close-modal-btn')) {
            const closeBtn = document.createElement('button');
            closeBtn.textContent = 'Fechar';
            closeBtn.className = 'close-modal-btn mt-4 bg-primary text-white font-bold py-2 px-6 rounded-lg';
            closeBtn.onclick = closeModal;
            content.appendChild(closeBtn);
        }
    };
    const closeModal = () => document.getElementById('modal-overlay').classList.add('hidden');

    // --- INITIALIZATION ---
    render();
});
</script>

</body>
</html>
