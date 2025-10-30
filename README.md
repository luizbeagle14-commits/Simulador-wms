<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador WMS – Da Separação à Entrega</title>
    
    <script src="https://cdn.tailwindcss.com"></script>
    
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@600;700&family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
    
    <script src="https://unpkg.com/lucide@latest"></script>

    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>
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
<body class="bg-gray-100 font-roboto">
      
    <div id="wms-simulator-app" class="min-h-screen flex flex-col">
        </div>

    <template id="header-template">
    <header class="bg-white border-b border-gray-200 p-4 flex justify-between items-center w-full">
        <img src="https://www.imagemhost.com.br/images/2024/11/22/Logo-novo-SENAI_-sem-slogan_755X325.png" 
             alt="Logo SENAI" 
             class="h-10 object-contain mr-4">
        <div class="flex items-center space-x-4">
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
    
    <div id="modal-overlay" class="hidden fixed inset-0 bg-gray-900 bg-opacity-75 flex items-center justify-center p-4 z-50">
        <div id="modal-content" class="bg-white rounded-lg shadow-xl p-6 max-w-lg w-full text-center">
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
            'PROD-101': { name: 'Caixa de Leite Integral', img: 'https://placehold.co/300x300/ECEFF1/37474F?text=Leite', price: 5.50, weight: 1.0, barcode: 'PROD-101', stock: 150, length: 0.2, width: 0.1, height: 0.15, arrivalDate: '2025-06-15' },
            'PROD-102': { name: 'Pacote de Arroz 5kg', img: 'https://placehold.co/300x300/ECEFF1/37474F?text=Arroz', price: 25.00, weight: 5.0, barcode: 'PROD-102', stock: 80, length: 0.4, width: 0.25, height: 0.1, arrivalDate: '2025-06-20' },
            'PROD-103': { name: 'Garrafa de Óleo de Soja', img: 'https://placehold.co/300x300/ECEFF1/37474F?text=Óleo', price: 8.75, weight: 0.9, barcode: 'PROD-103', stock: 200, length: 0.08, width: 0.08, height: 0.3, arrivalDate: '2025-06-18' },
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
                    <div class="mt-6 p-4 border border-blue-200 bg-blue-50 rounded-lg text-left space-y-2">
                        <h3 class="text-base font-bold text-primary flex items-center gap-2"><i data-lucide="info" class="w-5 h-5"></i>Instruções de Uso</h3>
                        <p class="text-sm">Este é um simulador de Sistema de Gerenciamento de Armazém (WMS) que o guiará pelas principais etapas da logística:</p>
                        <ul class="list-disc list-inside text-sm pl-4 space-y-1">
                            <li>**Login Operacional:** Inicie o turno e acesse o Dashboard Gerencial.</li>
                            <li>**Dashboard:** Visualize indicadores e use ferramentas de gestão de estoque (Ponto de Pedido, Lead Time, Paletização).</li>
                            <li>**Catálogo e Pedidos:** Adicione novos produtos, gerencie o estoque e crie pedidos de clientes (demanda de saída).</li>
                            <li>**Separação (Scanning):** Simule a coleta dos produtos do pedido usando o código de barras virtual, conferindo a acuracidade.</li>
                            <li>**Romaneio e Entrega:** Prepare o carregamento, faça a roteirização, calcule o frete (ICMS Interestadual) e simule a baixa de entrega com a assinatura digital.</li>
                        </ul>
                        <p class="text-sm font-semibold">Prossiga preenchendo seu nome e selecionando o turno para iniciar a simulação!</p>
                    </div>
                    <p class="text-xs text-gray-400 mt-6">Desenvolvido sob orientação do Professor Luiz Eduardo</p>
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
                    <div class="bg-white p-6 rounded-lg shadow-lg flex items-center gap-4">
                        <i data-lucide="package-check" class="w-12 h-12 text-highlight"></i>
                        <div>
                            <p class="text-secondary">Itens Entregues</p>
                            <p class="text-3xl font-bold">${deliveredCount}</p>
                        </div>
                    </div>
                    <div class="bg-white p-6 rounded-lg shadow-lg">
                        <h3 class="font-bold mb-3">Nível de Estoque</h3>
                        <ul class="space-y-2 max-h-48 overflow-y-auto pr-2">${stockHtml || '<p class="text-sm text-secondary">Nenhum produto no catálogo.</p>'}</ul>
                    </div>
                    <div class="bg-white p-6 rounded-lg shadow-lg">
                        <h3 class="font-bold mb-3">Produtos Mais Vendidos</h3>
                        <ul class="space-y-2 max-h-48 overflow-y-auto pr-2">${soldItemsHtml || '<p class="text-sm text-secondary">Nenhuma entrega realizada.</p>'}</ul>
                    </div>
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
                        <div class="space-y-3 max-h-96 overflow-y-auto pr-2">
                            ${productsHtml}
                        </div>
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
                                <select id="customer-state" class="w-full p-2 border rounded">
                                    ${statesBR.map(uf => `<option value="${uf}" ${uf === 'TO' ? 'selected' : ''}>${uf}</option>`).join('')}
                                </select>
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
        document.getElementById('back-to-catalog-btn').onclick = () => { state.currentStep = 'catalog'; render(); };


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

            // Create Order Logic
            const totalValue = Object.entries(orderItems).reduce((sum, [id, qty]) => {
                return sum + (qty * state.products[id].price);
            }, 0);
            
            const totalWeight = Object.entries(orderItems).reduce((sum, [id, qty]) => {
                return sum + (qty * state.products[id].weight);
            }, 0);

            const newOrder = {
                id: `PED-${Date.now()}`,
                customer: customer,
                address: { street: address, state: customerState },
                items: orderItems,
                totalValue: totalValue,
                totalWeight: totalWeight,
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

        const createOrderCard = (order) => {
            let statusColor = 'bg-yellow-100 text-yellow-800';
            if (order.status === 'dispatched') statusColor = 'bg-blue-100 text-blue-800';
            if (order.status === 'delivered') statusColor = 'bg-green-100 text-green-800';
            
            const orderTotalItems = Object.values(order.items).reduce((sum, qty) => sum + qty, 0);

            return `
                <div class="bg-white p-4 rounded-lg border flex justify-between items-center transition-shadow hover:shadow-md">
                    <div class="flex items-center gap-4">
                        ${order.status === 'pending' ? `<input type="checkbox" data-order-id="${order.id}" class="order-checkbox h-5 w-5 text-primary focus:ring-primary border-gray-300 rounded">` : ''}
                        <div>
                            <p class="font-bold text-lg">${order.id}</p>
                            <p class="text-sm text-secondary">${order.customer} - ${order.address.state}</p>
                            <p class="text-sm text-secondary">${orderTotalItems} itens (${Object.keys(order.items).length} tipos)</p>
                        </div>
                    </div>
                    <div class="text-right">
                        <span class="inline-block px-3 py-1 text-xs font-semibold rounded-full ${statusColor}">
                            ${order.status === 'pending' ? 'Pendente' : order.status === 'dispatched' ? 'Em Rota' : 'Entregue'}
                        </span>
                        <p class="font-bold text-primary mt-1">R$ ${order.totalValue.toFixed(2)}</p>
                        ${order.status === 'pending' ? `<button data-order-id="${order.id}" class="view-order-details-btn text-xs text-blue-600 hover:underline mt-1">Detalhes</button>` : ''}
                    </div>
                </div>
            `;
        };

        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-2xl font-bold">Painel de Pedidos</h2>
                    <div class="flex gap-4">
                        <button id="go-to-create-order-btn-2" class="bg-action text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2"><i data-lucide="plus-circle"></i> Novo Pedido</button>
                        <button id="back-to-dash-btn-2" class="bg-gray-200 py-2 px-4 rounded-lg flex items-center gap-2"><i data-lucide="arrow-left"></i> Dashboard</button>
                    </div>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    <div class="bg-gray-100 p-4 rounded-lg shadow-inner">
                        <h3 class="text-xl font-bold mb-4 text-primary flex justify-between items-center">
                            Aguardando Separação (${pendingOrders.length})
                            ${pendingOrders.length > 0 ? `<button id="process-selected-btn" class="bg-highlight text-white text-sm font-bold py-1 px-3 rounded-lg flex items-center gap-1 opacity-50 cursor-not-allowed" disabled><i data-lucide="scan" class="w-4 h-4"></i> Separar (0)</button>` : ''}
                        </h3>
                        <div id="pending-orders-list" class="space-y-3 max-h-[600px] overflow-y-auto pr-2">
                            ${pendingOrders.length > 0 ? pendingOrders.map(createOrderCard).join('') : '<p class="text-center text-secondary py-4">Nenhum pedido pendente.</p>'}
                        </div>
                    </div>

                    <div class="bg-gray-100 p-4 rounded-lg shadow-inner">
                        <h3 class="text-xl font-bold mb-4 text-blue-600">Em Rota/Despachados (${dispatchedOrders.length})</h3>
                        <div id="dispatched-orders-list" class="space-y-3 max-h-[600px] overflow-y-auto pr-2">
                            ${dispatchedOrders.length > 0 ? dispatchedOrders.map(createOrderCard).join('') : '<p class="text-center text-secondary py-4">Nenhum pedido em rota.</p>'}
                            ${dispatchedOrders.length > 0 ? `<div class="mt-4"><button id="start-delivery-btn" class="w-full bg-blue-600 text-white font-bold py-2 rounded-lg flex items-center justify-center gap-2"><i data-lucide="truck"></i> Iniciar Entregas</button></div>` : ''}
                        </div>
                    </div>

                    <div class="bg-gray-100 p-4 rounded-lg shadow-inner">
                        <h3 class="text-xl font-bold mb-4 text-highlight">Entregues (${deliveredOrders.length})</h3>
                        <div class="space-y-3 max-h-[600px] overflow-y-auto pr-2">
                            ${deliveredOrders.length > 0 ? deliveredOrders.map(createOrderCard).join('') : '<p class="text-center text-secondary py-4">Nenhum pedido entregue.</p>'}
                        </div>
                    </div>
                </div>
            </main>
        `;

        lucide.createIcons();

        document.getElementById('go-to-create-order-btn-2').onclick = () => { state.currentStep = 'create_order'; render(); };
        document.getElementById('back-to-dash-btn-2').onclick = () => { state.currentStep = 'dashboard'; render(); };

        const processBtn = document.getElementById('process-selected-btn');
        const checkboxes = document.querySelectorAll('.order-checkbox');
        
        const updateProcessButton = () => {
            const selectedCount = document.querySelectorAll('.order-checkbox:checked').length;
            processBtn.textContent = `Separar (${selectedCount})`;
            if (selectedCount > 0) {
                processBtn.disabled = false;
                processBtn.classList.remove('opacity-50', 'cursor-not-allowed');
            } else {
                processBtn.disabled = true;
                processBtn.classList.add('opacity-50', 'cursor-not-allowed');
            }
        };

        checkboxes.forEach(cb => cb.addEventListener('change', updateProcessButton));

        if (processBtn) {
            processBtn.onclick = () => {
                const selectedOrders = Array.from(document.querySelectorAll('.order-checkbox:checked')).map(cb => cb.dataset.orderId);
                
                if (selectedOrders.length > 0) {
                    // For the simulator, we only allow one order to be picked at a time for simplicity
                    // If multiple are selected, we take the first one.
                    state.currentShipment.orders = [state.orders.find(o => o.id === selectedOrders[0])];
                    state.scannedItems = {};
                    state.nonConformities = [];
                    state.currentStep = 'scanning';
                    render();
                }
            };
        }

        document.querySelectorAll('.view-order-details-btn').forEach(btn => {
            btn.onclick = (e) => {
                const orderId = e.target.dataset.orderId;
                const order = state.orders.find(o => o.id === orderId);
                if (order) {
                    let itemsHtml = Object.entries(order.items).map(([id, qty]) => {
                        const prod = state.products[id];
                        return `<li class="flex justify-between items-center text-sm border-b py-1"><span>${qty}x ${prod.name}</span> <span class="font-semibold">R$ ${(qty * prod.price).toFixed(2)}</span></li>`;
                    }).join('');

                    showModal(`
                        <h3 class="text-xl font-bold text-primary mb-3">Detalhes do Pedido ${order.id}</h3>
                        <div class="text-left space-y-2 mb-4 p-3 bg-gray-50 rounded-lg">
                            <p><strong>Cliente:</strong> ${order.customer}</p>
                            <p><strong>Endereço:</strong> ${order.address.street}, ${order.address.state}</p>
                            <p><strong>Valor Total:</strong> <span class="text-highlight font-bold">R$ ${order.totalValue.toFixed(2)}</span></p>
                            <p><strong>Peso Total:</strong> ${order.totalWeight.toFixed(2)} kg</p>
                        </div>
                        <h4 class="font-bold text-left mb-2">Itens do Pedido:</h4>
                        <ul class="text-left max-h-40 overflow-y-auto pr-2">
                            ${itemsHtml}
                        </ul>
                    `);
                }
            };
        });

        const startDeliveryBtn = document.getElementById('start-delivery-btn');
        if (startDeliveryBtn) {
            startDeliveryBtn.onclick = () => {
                // Collect all dispatched orders
                state.currentShipment.orders = dispatchedOrders;
                state.currentShipment.incidents = [];
                state.currentStep = 'routing';
                render();
            };
        }
    }

    function renderScanning() {
        const order = state.currentShipment.orders[0];
        if (!order) {
            state.currentStep = 'order_list';
            render();
            return;
        }

        const requiredItems = Object.entries(order.items).map(([id, qty]) => ({
            id: id,
            name: state.products[id].name,
            barcode: state.products[id].barcode,
            required: qty,
            scanned: state.scannedItems[id] || 0
        }));

        const isOrderComplete = requiredItems.every(item => item.scanned >= item.required);

        const itemsToScanHtml = requiredItems.map(item => {
            const isCompleted = item.scanned >= item.required;
            const statusClass = isCompleted ? 'bg-green-100 border-green-400 item-scanned' : 'bg-white border-gray-200 item-to-scan';
            const progress = (item.scanned / item.required) * 100;

            return `
                <li id="item-${item.id}" class="flex flex-col p-3 rounded-lg shadow-sm border ${statusClass}" data-required-qty="${item.required}">
                    <div class="flex justify-between items-center">
                        <div class="flex items-center space-x-3">
                            <i data-lucide="${isCompleted ? 'check-circle' : 'package'}" class="${isCompleted ? 'text-green-600' : 'text-primary'} w-6 h-6 flex-shrink-0"></i>
                            <div>
                                <p class="font-bold text-base">${item.name}</p>
                                <p class="text-xs text-secondary">Cód. Barra: ${item.barcode}</p>
                            </div>
                        </div>
                        <div class="text-right flex-shrink-0">
                            <p class="text-lg font-bold ${isCompleted ? 'text-green-600' : 'text-primary'}">${item.scanned} / ${item.required}</p>
                        </div>
                    </div>
                    <div class="w-full bg-gray-200 rounded-full h-1 mt-2">
                        <div class="bg-primary h-1 rounded-full transition-all duration-500" style="width: ${Math.min(100, progress)}%"></div>
                    </div>
                </li>
            `;
        }).join('');

        const totalRequired = requiredItems.reduce((sum, item) => sum + item.required, 0);
        const totalScanned = requiredItems.reduce((sum, item) => sum + item.scanned, 0);
        const overallProgress = (totalScanned / totalRequired) * 100 || 0;


        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Separação de Pedido: ${order.id}</h2>
                    <button id="cancel-picking-btn" class="bg-gray-200 py-2 px-4 rounded-lg flex items-center gap-2"><i data-lucide="x-circle"></i> Cancelar</button>
                </div>
                
                <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
                    <div class="md:col-span-1 space-y-4">
                         <div class="bg-white p-6 rounded-lg shadow-lg">
                            <h3 class="text-xl font-bold mb-3">Simulador de Scanner</h3>
                            <p class="text-sm text-secondary mb-4">Insira o código de barras para simular a leitura do produto.</p>
                            
                            <input type="text" id="barcode-input" placeholder="Insira o Código de Barras..." class="w-full p-3 border-2 border-primary rounded-lg focus:outline-none focus:ring-2 focus:ring-primary" autofocus>
                            <button id="scan-btn" class="w-full mt-3 bg-primary text-white font-bold py-3 rounded-lg flex items-center justify-center gap-2 hover:opacity-90">
                                <i data-lucide="qr-code"></i> Confirmar Leitura
                            </button>
                        </div>

                         <div class="bg-white p-6 rounded-lg shadow-lg">
                            <h3 class="text-xl font-bold mb-3">Status da Separação</h3>
                            <div class="w-full bg-gray-200 rounded-full h-4">
                                <div class="bg-highlight h-4 rounded-full transition-all duration-500" style="width: ${overallProgress}%"></div>
                            </div>
                            <p class="text-center font-bold mt-2">${totalScanned} de ${totalRequired} itens lidos (${overallProgress.toFixed(1)}%)</p>
                        </div>

                        <button id="finish-picking-btn" class="w-full bg-action text-white font-bold py-3 px-4 rounded-lg flex items-center justify-center gap-2 hover:opacity-90 transition-opacity ${isOrderComplete ? '' : 'opacity-50 cursor-not-allowed'}" ${isOrderComplete ? '' : 'disabled'}>
                            <i data-lucide="package-check"></i> Finalizar Separação
                        </button>
                    </div>
                    
                    <div class="md:col-span-2 space-y-4">
                         <h3 class="text-xl font-bold">Itens a Separar</h3>
                        <div id="items-to-scan-list" class="space-y-3 max-h-[700px] overflow-y-auto pr-2">
                            ${itemsToScanHtml || '<p class="text-center text-secondary py-4">Nenhum item no pedido.</p>'}
                        </div>
                    </div>
                </div>

                <div id="non-conformity-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-75 flex items-center justify-center p-4 z-50">
                    <div class="bg-white rounded-lg shadow-xl p-6 max-w-lg w-full text-center">
                        <i data-lucide="alert-triangle" class="mx-auto w-12 h-12 text-red-500 mb-4"></i>
                        <h3 class="text-xl font-bold mb-2 text-red-500">Não Conformidade Detectada!</h3>
                        <p class="text-secondary mb-4">O código de barras inserido não corresponde a um item esperado ou o estoque está esgotado.</p>
                        
                        <div class="text-left space-y-3 mb-4">
                            <label class="block text-sm font-medium">Motivo da Não Conformidade:</label>
                            <select id="nc-reason" class="w-full p-2 border rounded">
                                <option value="Produto Incorreto">Produto Incorreto (Erro de Leitura/Picking)</option>
                                <option value="Estoque Insuficiente">Estoque Insuficiente</option>
                                <option value="Produto Danificado">Produto Danificado</option>
                            </select>
                            <label class="block text-sm font-medium">Ação Tomada:</label>
                            <textarea id="nc-action" class="w-full p-2 border rounded" rows="2" placeholder="Ex: Informado ao Supervisor / Substituído por similar"></textarea>
                        </div>

                        <button id="record-nc-btn" class="w-full bg-red-500 text-white font-bold py-3 rounded-lg hover:opacity-90">
                            Registrar e Continuar
                        </button>
                        <button id="close-nc-modal-btn" class="w-full mt-2 bg-gray-200 text-gray-800 font-bold py-3 rounded-lg hover:bg-gray-300">
                            Cancelar e Revisar
                        </button>
                    </div>
                </div>
            </main>
        `;
        lucide.createIcons();
        const barcodeInput = document.getElementById('barcode-input');
        const itemsToScanList = document.getElementById('items-to-scan-list');
        const finishBtn = document.getElementById('finish-picking-btn');
        const ncModal = document.getElementById('non-conformity-modal');
        const recordNcBtn = document.getElementById('record-nc-btn');
        const closeNcModalBtn = document.getElementById('close-nc-modal-btn');
        let currentIncidentProduct = null;
        let ncModalVisible = false;

        const updateItemStatus = (id, newScannedQty, requiredQty) => {
            const itemEl = document.getElementById(`item-${id}`);
            if (itemEl) {
                const isCompleted = newScannedQty >= requiredQty;
                const statusClass = isCompleted ? 'bg-green-100 border-green-400' : 'bg-white border-gray-200';
                const icon = isCompleted ? 'check-circle' : 'package';
                const progress = (newScannedQty / requiredQty) * 100;
                
                itemEl.classList.remove('bg-white', 'bg-green-100', 'border-green-400', 'border-gray-200');
                itemEl.classList.add(statusClass);

                itemEl.querySelector('p:nth-child(2) .font-bold').textContent = `${newScannedQty} / ${requiredQty}`;
                itemEl.querySelector('.w-6.h-6').innerHTML = lucide.createIcons()[icon].toSvg({ class: `${isCompleted ? 'text-green-600' : 'text-primary'} w-6 h-6 flex-shrink-0` });
                itemEl.querySelector('.bg-primary').style.width = `${Math.min(100, progress)}%`;
                
                if (isCompleted && newScannedQty === requiredQty) {
                    itemEl.classList.add('flash-success');
                    setTimeout(() => {
                        itemEl.classList.remove('flash-success');
                        itemEl.classList.add('item-scanned'); // Hide completed item
                    }, 1000); 
                }
            }
        };

        const scanItem = (barcode) => {
            const productFound = Object.values(state.products).find(p => p.barcode === barcode);
            const productId = productFound ? productFound.barcode : null;
            const requiredItem = requiredItems.find(item => item.id === productId);

            if (!productId || !requiredItem) {
                // Barcode not found in any product or product not in this order
                currentIncidentProduct = productId ? productId : barcode; // Store actual barcode or product ID if found but not required
                showNonConformityModal();
                return;
            }

            const currentScanned = state.scannedItems[productId] || 0;
            if (currentScanned < requiredItem.required) {
                state.scannedItems[productId] = currentScanned + 1;
                updateItemStatus(productId, state.scannedItems[productId], requiredItem.required);
                
                // Update overall progress
                const totalScanned = requiredItems.reduce((sum, item) => sum + (state.scannedItems[item.id] || 0), 0);
                const overallProgress = (totalScanned / totalRequired) * 100 || 0;
                document.querySelector('#finish-picking-btn').disabled = totalScanned < totalRequired;
                if(totalScanned >= totalRequired) {
                    document.querySelector('#finish-picking-btn').classList.remove('opacity-50', 'cursor-not-allowed');
                }
                
                document.querySelector('.h-4 .bg-highlight').style.width = `${overallProgress}%`;
                document.querySelector('.h-4').nextElementSibling.textContent = `${totalScanned} de ${totalRequired} itens lidos (${overallProgress.toFixed(1)}%)`;

                // Flash input for success
                barcodeInput.classList.add('flash-success');
                setTimeout(() => barcodeInput.classList.remove('flash-success'), 1000);
            } else {
                 // Too many scanned
                currentIncidentProduct = productId;
                showNonConformityModal("Excesso de itens lidos para este produto. Leitura rejeitada.");
                barcodeInput.classList.add('flash-error');
                setTimeout(() => barcodeInput.classList.remove('flash-error'), 1000);
            }
            
            barcodeInput.value = ''; // Clear input
            barcodeInput.focus();
        };

        const showNonConformityModal = (message = "O código de barras inserido não corresponde a um item esperado ou o estoque está esgotado.") => {
            ncModal.classList.remove('hidden');
            ncModalVisible = true;
            ncModal.querySelector('p.text-secondary.mb-4').textContent = message;
        };
        const hideNonConformityModal = () => {
            ncModal.classList.add('hidden');
            ncModalVisible = false;
            currentIncidentProduct = null;
            barcodeInput.focus();
        };

        document.getElementById('scan-btn').onclick = () => scanItem(barcodeInput.value.trim().toUpperCase());
        barcodeInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter' && !ncModalVisible) {
                e.preventDefault();
                scanItem(barcodeInput.value.trim().toUpperCase());
            }
        });
        
        // Non-Conformity Modal Handlers
        recordNcBtn.onclick = () => {
             const reason = document.getElementById('nc-reason').value;
             const action = document.getElementById('nc-action').value;
             
             state.nonConformities.push({
                 orderId: order.id,
                 product: currentIncidentProduct,
                 reason: reason,
                 action: action,
                 timestamp: new Date().toISOString()
             });

             hideNonConformityModal();
             showModal(`<h3 class="text-xl font-bold text-red-500">Não Conformidade Registrada</h3><p>O incidente foi registrado e a separação pode continuar.</p>`);
        };
        closeNcModalBtn.onclick = hideNonConformityModal;
        
        document.getElementById('cancel-picking-btn').onclick = () => {
            if (confirm("Deseja cancelar a separação e voltar para a lista de pedidos?")) {
                state.currentStep = 'order_list';
                render();
            }
        };

        finishBtn.onclick = () => {
            if (isOrderComplete) {
                const completedOrder = state.orders.find(o => o.id === order.id);
                if (completedOrder) {
                    completedOrder.status = 'dispatched'; // Mark as dispatched after picking and before loading/routing
                    state.currentShipment.orders = [completedOrder];
                    state.currentShipment.totalWeight = completedOrder.totalWeight;
                    state.currentShipment.totalValue = completedOrder.totalValue;
                    state.currentShipment.incidents = state.nonConformities.filter(nc => nc.orderId === completedOrder.id);
                }
                state.currentStep = 'romaneio';
                render();
            }
        };
    }

    function renderRomaneio() {
         const order = state.currentShipment.orders[0];
         if (!order) {
            state.currentStep = 'order_list';
            render();
            return;
        }

        const orderTotalItems = Object.values(order.items).reduce((sum, qty) => sum + qty, 0);

        let itemsHtml = Object.entries(order.items).map(([id, qty]) => {
            const prod = state.products[id];
            return `
                <tr class="border-b">
                    <td class="p-2">${prod.barcode}</td>
                    <td class="p-2">${prod.name}</td>
                    <td class="p-2 text-right">${qty}</td>
                    <td class="p-2 text-right">R$ ${prod.price.toFixed(2)}</td>
                    <td class="p-2 text-right">R$ ${(qty * prod.price).toFixed(2)}</td>
                </tr>
            `;
        }).join('');

        let nonConformityHtml = state.currentShipment.incidents.map((nc, index) => `
            <tr class="${index % 2 === 0 ? 'bg-red-50' : ''}">
                <td class="p-2">${nc.product}</td>
                <td class="p-2">${nc.reason}</td>
                <td class="p-2">${nc.action}</td>
                <td class="p-2 text-sm">${new Date(nc.timestamp).toLocaleTimeString()}</td>
            </tr>
        `).join('');


         appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Romaneio de Carga e Conferência</h2>
                    <div class="flex gap-4">
                        <button id="print-romaneio-btn" class="bg-gray-500 text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2 hover:opacity-90"><i data-lucide="printer"></i> Imprimir Romaneio</button>
                        <button id="next-step-btn" class="bg-primary text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2 hover:opacity-90">Roteirização <i data-lucide="arrow-right"></i></button>
                    </div>
                </div>

                <div class="bg-white p-6 rounded-lg shadow-lg mb-6">
                    <h3 class="text-xl font-bold mb-4">Dados do Romaneio - PEDIDO ${order.id}</h3>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 border-b pb-4 mb-4 text-sm">
                        <p><strong>Cliente:</strong> ${order.customer}</p>
                        <p><strong>Destino:</strong> ${order.address.street}, ${order.address.state}</p>
                        <p><strong>Total de Itens:</strong> ${orderTotalItems}</p>
                        <p><strong>Peso Total (kg):</strong> ${order.totalWeight.toFixed(2)}</p>
                        <p><strong>Valor Total (R$):</strong> ${order.totalValue.toFixed(2)}</p>
                        <p><strong>Operador WMS:</strong> ${state.userName}</p>
                    </div>

                    <h4 class="font-bold mb-3">Lista de Produtos Conferidos</h4>
                    <div class="overflow-x-auto">
                        <table class="min-w-full bg-white border border-gray-200 rounded-lg">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th class="p-2 text-left text-sm font-semibold">Cód. Barra</th>
                                    <th class="p-2 text-left text-sm font-semibold">Produto</th>
                                    <th class="p-2 text-right text-sm font-semibold">Qtd</th>
                                    <th class="p-2 text-right text-sm font-semibold">Preço Un.</th>
                                    <th class="p-2 text-right text-sm font-semibold">Subtotal</th>
                                </tr>
                            </thead>
                            <tbody>
                                ${itemsHtml}
                            </tbody>
                            <tfoot>
                                <tr class="bg-gray-100 font-bold">
                                    <td colspan="4" class="p-2 text-right">TOTAL:</td>
                                    <td class="p-2 text-right">R$ ${order.totalValue.toFixed(2)}</td>
                                </tr>
                            </tfoot>
                        </table>
                    </div>

                    ${state.currentShipment.incidents.length > 0 ? `
                        <h4 class="font-bold mb-3 mt-6 text-red-600">Registro de Não Conformidades (${state.currentShipment.incidents.length})</h4>
                        <div class="overflow-x-auto">
                             <table class="min-w-full bg-white border border-red-300 rounded-lg">
                                <thead class="bg-red-100">
                                    <tr>
                                        <th class="p-2 text-left text-sm font-semibold">Produto/Cód.</th>
                                        <th class="p-2 text-left text-sm font-semibold">Motivo</th>
                                        <th class="p-2 text-left text-sm font-semibold">Ação Tomada</th>
                                        <th class="p-2 text-left text-sm font-semibold">Hora</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    ${nonConformityHtml}
                                </tbody>
                            </table>
                        </div>
                    ` : ''}
                </div>
            </main>
        `;

        lucide.createIcons();
        document.getElementById('next-step-btn').onclick = () => { state.currentStep = 'routing'; render(); };
        document.getElementById('print-romaneio-btn').onclick = () => printRomaneio(order);
    }

    function renderRouting() {
        // Simple routing simulation
        const order = state.currentShipment.orders[0];
        if (!order) {
            state.currentStep = 'order_list';
            render();
            return;
        }

        const warehouseLocation = { lat: -10.1834, lon: -48.3333, name: 'Armazém (TO)' }; // Palomas - TO
        const customerLocations = [
            { lat: -15.7801, lon: -47.9292, name: 'Brasília (DF)', state: 'DF' }, // Rota 1
            { lat: -16.6869, lon: -49.2644, name: 'Goiânia (GO)', state: 'GO' }, // Rota 2
            { lat: -23.5505, lon: -46.6333, name: 'São Paulo (SP)', state: 'SP' } // Rota 3
        ];
        
        // Find the best route based on the destination state
        const destinationState = order.address.state;
        let route = customerLocations.find(loc => loc.state === destinationState);
        
        if (!route) {
            // Default to a close-by location if state is not in the list (simple simulation)
            route = { lat: -10.25, lon: -48.33, name: `Destino ${destinationState}`, state: destinationState };
        }
        
        const distanceKm = calculateDistance(warehouseLocation.lat, warehouseLocation.lon, route.lat, route.lon);
        state.currentShipment.route.totalDistance = distanceKm;
        state.currentShipment.route.points = [warehouseLocation, route];


        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Roteirização e Distância</h2>
                    <button id="next-step-btn" class="bg-primary text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2 hover:opacity-90">Conferência de Frete <i data-lucide="arrow-right"></i></button>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                    <div class="bg-white p-6 rounded-lg shadow-lg">
                        <h3 class="text-xl font-bold mb-4">Rota Visualizada (TO para ${route.name})</h3>
                        <div id="map"></div>
                    </div>
                    
                    <div class="bg-white p-6 rounded-lg shadow-lg space-y-4">
                        <h3 class="text-xl font-bold mb-4">Detalhes da Entrega</h3>
                        
                        <div class="bg-gray-100 p-4 rounded-lg">
                            <p class="text-lg font-bold">Destino Final:</p>
                            <p class="text-2xl font-bold text-primary">${route.name} (${route.state})</p>
                            <p class="text-lg font-bold mt-3">Distância Calculada (Aérea):</p>
                            <p class="text-3xl font-bold text-highlight">${distanceKm.toFixed(2)} km</p>
                        </div>
                        
                        <div class="border-t pt-4">
                            <h4 class="font-bold mb-2">Estimativa de Tempo e Custo</h4>
                            <div class="grid grid-cols-2 gap-4">
                                <div>
                                    <p class="text-sm text-secondary">Tempo Estimado (caminhão):</p>
                                    <p class="font-bold text-lg">${(distanceKm / 80).toFixed(1)} horas</p>
                                </div>
                                <div>
                                    <p class="text-sm text-secondary">Custo Variável Estimado (R$ 5/km):</p>
                                    <p class="font-bold text-lg">R$ ${(distanceKm * 5).toFixed(2)}</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </main>
        `;
        lucide.createIcons();
        document.getElementById('next-step-btn').onclick = () => { state.currentStep = 'freight_check'; render(); };
        
        // Initialize Leaflet Map
        const map = L.map('map').setView([warehouseLocation.lat, warehouseLocation.lon], 6);
        
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(map);

        // Add Markers
        const startMarker = L.marker([warehouseLocation.lat, warehouseLocation.lon]).addTo(map)
            .bindPopup(`<b>${warehouseLocation.name}</b><br>Origem (Armazém)`).openPopup();
        const endMarker = L.marker([route.lat, route.lon], { icon: L.icon({ iconUrl: 'data:image/svg+xml;base64,' + btoa('<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#FF6F00" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-map-pin"><path d="M12 2v6"/><path d="M12 22v-6"/><circle cx="12" cy="12" r="4"/></svg>'), iconSize: [30, 30] }) }).addTo(map)
            .bindPopup(`<b>${route.name}</b><br>Destino (Cliente)`).openPopup();

        // Draw Polyline for the route
        const latlngs = [
            [warehouseLocation.lat, warehouseLocation.lon],
            [route.lat, route.lon]
        ];
        L.polyline(latlngs, { color: 'red', weight: 3, dashArray: '5, 10' }).addTo(map);

        // Fit map bounds to show both markers
        const bounds = L.latLngBounds([startMarker.getLatLng(), endMarker.getLatLng()]);
        map.fitBounds(bounds, { padding: [50, 50] });

    }

    function renderFreightCheck() {
        const order = state.currentShipment.orders[0];
        if (!order) {
            state.currentStep = 'order_list';
            render();
            return;
        }

        const destinationState = order.address.state;
        const originState = 'TO'; // Tocantins (Assumed Origin)
        const totalValue = order.totalValue;

        const icmsRate = ICMS_RATES[destinationState] || 0.12; // Default to 12% if state not mapped
        const icmsValue = totalValue * icmsRate;
        const baseFreightCost = state.currentShipment.route.totalDistance * 5; // R$5/km from routing step
        
        // Simulate a margin/markup on the base freight
        const finalFreightCost = baseFreightCost * 1.35; // 35% Markup for operational costs and profit
        const finalTotal = totalValue + finalFreightCost;

        state.currentShipment.freightDetails = {
            icmsRate: icmsRate,
            icmsValue: icmsValue,
            baseFreightCost: baseFreightCost,
            finalFreightCost: finalFreightCost,
            finalTotal: finalTotal
        };

        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Cálculo de Frete e Impostos</h2>
                    <button id="next-step-btn" class="bg-primary text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2 hover:opacity-90">Rastreamento de Entrega <i data-lucide="arrow-right"></i></button>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                    <div class="lg:col-span-1 bg-white p-6 rounded-lg shadow-lg space-y-2">
                        <h3 class="text-xl font-bold text-primary">Resumo do Pedido</h3>
                        <p><strong>Pedido ID:</strong> ${order.id}</p>
                        <p><strong>Cliente:</strong> ${order.customer}</p>
                        <p><strong>UF Origem:</strong> ${originState}</p>
                        <p><strong>UF Destino:</strong> ${destinationState}</p>
                        <div class="border-t pt-4 mt-4">
                            <p class="text-sm text-secondary">Valor Mercadoria (Base):</p>
                            <p class="text-2xl font-bold text-highlight">R$ ${totalValue.toFixed(2)}</p>
                            <p class="text-sm text-secondary mt-3">Distância Roteirizada:</p>
                            <p class="text-xl font-bold">${state.currentShipment.route.totalDistance.toFixed(2)} km</p>
                        </div>
                    </div>

                    <div class="lg:col-span-2 bg-white p-6 rounded-lg shadow-lg">
                        <h3 class="text-xl font-bold mb-4">Cálculo ICMS Interestadual</h3>
                        <div class="grid grid-cols-2 gap-4 mb-6">
                            <div>
                                <p class="text-sm text-secondary">Alíquota ICMS (${originState} -> ${destinationState}):</p>
                                <p class="text-3xl font-bold text-red-600">${(icmsRate * 100).toFixed(0)}%</p>
                            </div>
                            <div>
                                <p class="text-sm text-secondary">Valor do ICMS a ser recolhido:</p>
                                <p class="text-3xl font-bold text-red-600">R$ ${icmsValue.toFixed(2)}</p>
                            </div>
                        </div>
                        
                        <h3 class="text-xl font-bold mb-4 border-t pt-4">Composição do Frete</h3>
                        <div class="grid grid-cols-2 gap-4">
                             <div>
                                <p class="text-sm text-secondary">Custo Base de Transporte (R$ 5/km):</p>
                                <p class="text-xl font-bold">R$ ${baseFreightCost.toFixed(2)}</p>
                            </div>
                             <div>
                                <p class="text-sm text-secondary">Frete Final (Com Markup de 35%):</p>
                                <p class="text-2xl font-bold text-action">R$ ${finalFreightCost.toFixed(2)}</p>
                            </div>
                        </div>

                        <div class="border-t pt-4 mt-6">
                            <p class="text-lg font-bold text-secondary">Valor Total da Nota Fiscal (Mercadoria + Frete):</p>
                            <p class="text-5xl font-bold text-primary">R$ ${finalTotal.toFixed(2)}</p>
                        </div>
                    </div>
                </div>
            </main>
        `;
        lucide.createIcons();
        document.getElementById('next-step-btn').onclick = () => { state.currentStep = 'delivery_tracking'; render(); };
    }

    function renderDeliveryTracking() {
        const order = state.currentShipment.orders[0];
        if (!order) {
            state.currentStep = 'order_list';
            render();
            return;
        }

        const route = state.currentShipment.route.points;
        const totalDistance = state.currentShipment.route.totalDistance;
        let currentLocation = route[0]; // Start at origin

        const updateMap = (map, marker, distanceCovered) => {
            const fraction = Math.min(1, distanceCovered / totalDistance);
            
            if (fraction >= 1) {
                currentLocation = route[1]; // Final destination
            } else {
                // Interpolate location along the route
                currentLocation = {
                    lat: route[0].lat + (route[1].lat - route[0].lat) * fraction,
                    lon: route[0].lon + (route[1].lon - route[0].lon) * fraction
                };
            }
            
            marker.setLatLng([currentLocation.lat, currentLocation.lon]);
            document.getElementById('tracking-status').textContent = fraction < 1 ? 'Em Trânsito...' : 'Chegou ao Destino!';
            document.getElementById('distance-covered').textContent = `${(totalDistance * fraction).toFixed(2)} km`;
            document.getElementById('distance-remaining').textContent = `${(totalDistance * (1 - fraction)).toFixed(2)} km`;
        };

        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Rastreamento e Monitoramento</h2>
                    <button id="next-step-btn" class="bg-primary text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2 hover:opacity-90 opacity-50 cursor-not-allowed" disabled>Baixa de Entrega <i data-lucide="arrow-right"></i></button>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                    <div class="lg:col-span-1 bg-white p-6 rounded-lg shadow-lg space-y-4">
                        <h3 class="text-xl font-bold text-primary">Status da Entrega</h3>
                        <div class="text-center p-4 rounded-lg bg-gray-100">
                            <p class="text-lg font-bold text-secondary">Status Atual:</p>
                            <p id="tracking-status" class="text-3xl font-bold text-action">Preparando Saída...</p>
                        </div>
                         
                         <div class="border-t pt-4 space-y-2">
                             <p><strong>Pedido ID:</strong> ${order.id}</p>
                             <p><strong>Destino:</strong> ${order.address.state} (${route[1].name})</p>
                             <p><strong>Distância Total:</strong> ${totalDistance.toFixed(2)} km</p>
                         </div>
                         
                         <div class="border-t pt-4 space-y-2 text-sm">
                            <p class="flex justify-between">Distância Percorrida: <span id="distance-covered" class="font-bold">0.00 km</span></p>
                            <p class="flex justify-between">Distância Restante: <span id="distance-remaining" class="font-bold">${totalDistance.toFixed(2)} km</span></p>
                         </div>
                         
                         <button id="simulate-travel-btn" class="w-full bg-highlight text-white font-bold py-3 rounded-lg flex items-center justify-center gap-2 hover:opacity-90">
                            <i data-lucide="play"></i> Simular Viagem (10s)
                        </button>
                    </div>

                    <div class="lg:col-span-2 bg-white p-6 rounded-lg shadow-lg">
                        <h3 class="text-xl font-bold mb-4">Posição Atual do Veículo</h3>
                        <div id="map-tracking" style="height: 400px;"></div>
                    </div>
                </div>
            </main>
        `;

        lucide.createIcons();
        const map = L.map('map-tracking').setView([route[0].lat, route[0].lon], 6);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(map);

        const truckIcon = L.icon({
            iconUrl: 'data:image/svg+xml;base64,' + btoa('<svg xmlns="http://www.w3.org/2000/svg" width="30" height="30" viewBox="0 0 24 24" fill="#1A237E" stroke="#FFFFFF" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-truck"><path d="M14 18V6l-3-4H4a2 2 0 0 0-2 2v10a2 2 0 0 0 2 2h2"/><path d="M15 22l-1-10h7l1 10h-7z"/><path d="M10 22h13"/><circle cx="6" cy="18" r="2"/><circle cx="17" cy="18" r="2"/></svg>'),
            iconSize: [30, 30],
            iconAnchor: [15, 15]
        });
        
        // Animated Marker (using standard marker for simplicity if plugin is complex)
        const truckMarker = L.marker([route[0].lat, route[0].lon], { icon: truckIcon }).addTo(map)
            .bindPopup("Caminhão: Posição Atual").openPopup();

        const destinationMarker = L.marker([route[1].lat, route[1].lon], { icon: L.icon({ iconUrl: 'data:image/svg+xml;base64,' + btoa('<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#FF6F00" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-map-pin"><path d="M12 2v6"/><path d="M12 22v-6"/><circle cx="12" cy="12" r="4"/></svg>'), iconSize: [30, 30] }) }).addTo(map)
            .bindPopup(`Destino: ${route[1].name}`);

        const latlngs = [
            [route[0].lat, route[0].lon],
            [route[1].lat, route[1].lon]
        ];
        L.polyline(latlngs, { color: '#00C853', weight: 4 }).addTo(map);
        map.fitBounds(L.latLngBounds(latlngs), { padding: [50, 50] });

        let distanceCovered = 0;
        let animationInterval;

        const simulateTravel = () => {
            document.getElementById('simulate-travel-btn').disabled = true;
            document.getElementById('tracking-status').textContent = 'Em Trânsito...';
            
            const steps = 100; // 100 steps in 10 seconds
            const distancePerStep = totalDistance / steps;
            
            animationInterval = setInterval(() => {
                distanceCovered += distancePerStep;
                
                if (distanceCovered >= totalDistance) {
                    distanceCovered = totalDistance;
                    clearInterval(animationInterval);
                    document.getElementById('simulate-travel-btn').style.display = 'none';
                    document.getElementById('next-step-btn').disabled = false;
                    document.getElementById('next-step-btn').classList.remove('opacity-50', 'cursor-not-allowed');
                }

                updateMap(map, truckMarker, distanceCovered);

            }, 100); // 100ms per step * 100 steps = 10 seconds
        };
        
        document.getElementById('simulate-travel-btn').onclick = simulateTravel;
        document.getElementById('next-step-btn').onclick = () => { state.currentStep = 'delivery'; render(); };
    }

    function renderDelivery() {
        const order = state.currentShipment.orders[0];
         if (!order) {
            state.currentStep = 'order_list';
            render();
            return;
        }

        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Baixa de Entrega (Proof of Delivery)</h2>
                    <button id="next-step-btn" class="bg-action text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2 hover:opacity-90">Avaliação de Conhecimento <i data-lucide="arrow-right"></i></button>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                    <div class="bg-white p-6 rounded-lg shadow-lg space-y-4">
                        <h3 class="text-xl font-bold text-primary">Confirmação de Recebimento</h3>
                        <div class="p-4 bg-gray-100 rounded-lg">
                             <p><strong>Pedido ID:</strong> ${order.id}</p>
                             <p><strong>Cliente:</strong> ${order.customer}</p>
                             <p><strong>Valor Total (NF):</strong> R$ ${state.currentShipment.freightDetails.finalTotal.toFixed(2)}</p>
                        </div>
                        
                        <div class="space-y-3">
                            <div>
                                <label for="recipient-name" class="block text-sm font-medium">Nome do Recebedor:</label>
                                <input type="text" id="recipient-name" class="w-full p-2 border rounded" placeholder="Ex: João da Silva (Porteiro)">
                            </div>
                            <div>
                                <label for="delivery-notes" class="block text-sm font-medium">Observações da Entrega:</label>
                                <textarea id="delivery-notes" class="w-full p-2 border rounded" rows="2" placeholder="Ex: Sem avarias. Entregue na portaria."></textarea>
                            </div>
                        </div>

                        <button id="confirm-delivery-btn" class="w-full bg-highlight text-white font-bold py-3 rounded-lg flex items-center justify-center gap-2 hover:opacity-90" disabled>
                            <i data-lucide="save"></i> Confirmar Entrega
                        </button>
                    </div>

                    <div class="bg-white p-6 rounded-lg shadow-lg">
                         <h3 class="text-xl font-bold mb-4">Assinatura Digital (POD)</h3>
                         <p class="text-sm text-secondary mb-3">O recebedor deve assinar no campo abaixo:</p>
                         <canvas id="signature-canvas" width="400" height="200" class="w-full h-auto rounded-lg"></canvas>
                         <button id="clear-signature-btn" class="w-full mt-3 bg-gray-200 text-gray-800 font-bold py-2 rounded-lg hover:bg-gray-300">Limpar Assinatura</button>
                    </div>
                </div>
            </main>
        `;

        lucide.createIcons();
        const canvas = document.getElementById('signature-canvas');
        const confirmBtn = document.getElementById('confirm-delivery-btn');
        const ctx = canvas.getContext('2d');
        let isDrawing = false;
        let lastX = 0;
        let lastY = 0;

        const clearCanvas = () => {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            confirmBtn.disabled = true;
            confirmBtn.classList.add('opacity-50', 'cursor-not-allowed');
        };

        const draw = (e) => {
            if (!isDrawing) return;
            e.preventDefault(); 
            
            const rect = canvas.getBoundingClientRect();
            let currentX, currentY;

            if (e.touches) {
                currentX = e.touches[0].clientX - rect.left;
                currentY = e.touches[0].clientY - rect.top;
            } else {
                currentX = e.clientX - rect.left;
                currentY = e.clientY - rect.top;
            }
            
            ctx.beginPath();
            ctx.moveTo(lastX, lastY);
            ctx.lineTo(currentX, currentY);
            ctx.strokeStyle = '#000000';
            ctx.lineWidth = 2;
            ctx.stroke();
            [lastX, lastY] = [currentX, currentY];
            
            confirmBtn.disabled = false;
            confirmBtn.classList.remove('opacity-50', 'cursor-not-allowed');
        };

        const startDrawing = (e) => {
            isDrawing = true;
            const rect = canvas.getBoundingClientRect();
            if (e.touches) {
                [lastX, lastY] = [e.touches[0].clientX - rect.left, e.touches[0].clientY - rect.top];
            } else {
                [lastX, lastY] = [e.clientX - rect.left, e.clientY - rect.top];
            }
        };

        const stopDrawing = () => {
            isDrawing = false;
        };

        // Event Listeners for Drawing
        canvas.addEventListener('mousedown', startDrawing);
        canvas.addEventListener('mousemove', draw);
        canvas.addEventListener('mouseup', stopDrawing);
        canvas.addEventListener('mouseout', stopDrawing);
        
        // Touch events for mobile
        canvas.addEventListener('touchstart', startDrawing);
        canvas.addEventListener('touchmove', draw);
        canvas.addEventListener('touchend', stopDrawing);
        canvas.addEventListener('touchcancel', stopDrawing);

        document.getElementById('clear-signature-btn').onclick = clearCanvas;
        
        clearCanvas(); // Initialize clear

        confirmBtn.onclick = () => {
            const recipientName = document.getElementById('recipient-name').value;
            const deliveryNotes = document.getElementById('delivery-notes').value;
            const signatureImage = canvas.toDataURL();

            if (!recipientName) {
                alert("Por favor, insira o nome do recebedor.");
                return;
            }

            // Update order status
            const completedOrder = state.orders.find(o => o.id === order.id);
            if (completedOrder) {
                completedOrder.status = 'delivered';
                state.deliveredItems.push(...Object.keys(order.items)); // Simple delivery tracking
            }

            // Store delivery data
            state.currentShipment.deliveryProof = {
                recipient: recipientName,
                notes: deliveryNotes,
                signature: signatureImage,
                timestamp: new Date().toISOString()
            };

            // Proceed to next step
            state.currentStep = 'quiz';
            render();
        };

        document.getElementById('next-step-btn').onclick = () => { state.currentStep = 'quiz'; render(); };
    }

    // --- QUIZ & FINAL STEPS (Stubs) ---
    
    function renderQuiz() {
        const quizQuestions = [
            {
                question: "O que significa a sigla WMS no contexto logístico?",
                options: ["Warehouse Management System", "World Market Standard", "Web Monitoring Service", "Whole Management System"],
                answer: "Warehouse Management System"
            },
             {
                question: "Qual ferramenta é usada para planejar a melhor rota de entrega?",
                options: ["Just-In-Time", "Roteirizador", "Kanban", "Supply Chain"],
                answer: "Roteirizador"
            },
            {
                question: "O que o Ponto de Pedido (PP) calcula?",
                options: ["O custo de armazenagem de um produto.", "O momento ideal para realizar uma nova compra de estoque.", "A velocidade de separação de pedidos.", "O tempo de entrega do fornecedor."],
                answer: "O momento ideal para realizar uma nova compra de estoque."
            },
            {
                question: "Qual imposto é recolhido em operações de venda de mercadorias entre diferentes estados no Brasil?",
                options: ["ISS", "ICMS", "IPI", "IRPJ"],
                answer: "ICMS"
            },
        ];
        
        let score = 0;
        
        const questionsHtml = quizQuestions.map((q, index) => `
            <div class="bg-white p-6 rounded-lg shadow-md mb-6">
                <p class="font-bold text-lg mb-3">Questão ${index + 1}: ${q.question}</p>
                <div class="space-y-2">
                    ${q.options.map((opt) => `
                        <label class="block cursor-pointer">
                            <input type="radio" name="question-${index}" value="${opt}" class="quiz-option mr-2 text-primary focus:ring-primary">
                            ${opt}
                        </label>
                    `).join('')}
                </div>
                <p id="feedback-${index}" class="mt-2 font-semibold"></p>
            </div>
        `).join('');

        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Avaliação de Conhecimento</h2>
                </div>
                
                <p class="mb-6 text-lg text-secondary">Teste seu conhecimento sobre as funções do WMS e conceitos logísticos abordados na simulação.</p>

                <form id="quiz-form">
                    ${questionsHtml}
                </form>

                <div class="mt-8 flex gap-4">
                    <button id="submit-quiz-btn" class="bg-primary text-white font-bold py-3 px-6 rounded-lg hover:opacity-90 flex items-center justify-center gap-2">
                        <i data-lucide="award"></i> Verificar Respostas
                    </button>
                    <button id="next-step-btn" class="bg-action text-white font-bold py-3 px-6 rounded-lg hover:opacity-90 flex items-center justify-center gap-2 opacity-50 cursor-not-allowed" disabled>
                        Desafio Final <i data-lucide="arrow-right"></i>
                    </button>
                </div>
                <div id="quiz-result" class="mt-6 p-4 bg-yellow-100 rounded-lg hidden"></div>
            </main>
        `;

        lucide.createIcons();
        const quizForm = document.getElementById('quiz-form');
        const submitBtn = document.getElementById('submit-quiz-btn');
        const resultDiv = document.getElementById('quiz-result');
        const nextBtn = document.getElementById('next-step-btn');
        let quizCompleted = false;

        submitBtn.onclick = (e) => {
            e.preventDefault();
            if (quizCompleted) return;
            
            score = 0;
            quizQuestions.forEach((q, index) => {
                const selectedOption = quizForm.querySelector(`input[name="question-${index}"]:checked`);
                const feedbackEl = document.getElementById(`feedback-${index}`);
                
                if (selectedOption) {
                    if (selectedOption.value === q.answer) {
                        score++;
                        feedbackEl.textContent = "Correto!";
                        feedbackEl.classList.remove('text-red-600');
                        feedbackEl.classList.add('text-green-600');
                    } else {
                        feedbackEl.textContent = `Incorreto. A resposta correta é: ${q.answer}`;
                        feedbackEl.classList.remove('text-green-600');
                        feedbackEl.classList.add('text-red-600');
                    }
                } else {
                    feedbackEl.textContent = `Não respondido. A resposta correta é: ${q.answer}`;
                    feedbackEl.classList.remove('text-green-600');
                    feedbackEl.classList.add('text-red-600');
                }
            });

            const maxScore = quizQuestions.length;
            const percentage = (score / maxScore) * 100;
            resultDiv.innerHTML = `<h3 class="text-xl font-bold">Resultado: ${score}/${maxScore} (${percentage.toFixed(0)}%)</h3>`;
            resultDiv.classList.remove('hidden');
            resultDiv.classList.remove('bg-yellow-100', 'bg-red-100', 'bg-green-100');
            if (score === maxScore) {
                resultDiv.classList.add('bg-green-100');
            } else if (score >= maxScore / 2) {
                 resultDiv.classList.add('bg-yellow-100');
            } else {
                 resultDiv.classList.add('bg-red-100');
            }
            
            state.quizScore = score;
            submitBtn.textContent = 'Revisar Respostas';
            nextBtn.disabled = false;
            nextBtn.classList.remove('opacity-50', 'cursor-not-allowed');
            quizCompleted = true;
        };

        nextBtn.onclick = () => { state.currentStep = 'final_challenge_quiz'; render(); };
    }

    function renderFinalChallengeQuiz() {
        const challengeQuestion = {
            question: `Você é o novo Gerente de Logística e precisa otimizar o processo. Com base na simulação (Pedido ${state.currentShipment.orders[0].id}):`,
            tasks: [
                "Qual ação você tomaria para reduzir a incidência de 'Produtos Incorretos' (Não Conformidades) na separação?",
                "Qual seria o impacto de reduzir o Lead Time de recebimento de 10 dias para 5 dias no seu Ponto de Pedido (usando a fórmula Demanda*LT + Estoque de Segurança)?",
                "Quais duas métricas (KPIs) você monitoraria de perto para garantir a eficiência da operação de Armazém?"
            ]
        };
        
        appContainer.innerHTML += `
            <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Desafio Final de Gestão</h2>
                </div>
                
                <div class="bg-white p-6 rounded-lg shadow-lg">
                    <p class="mb-6 text-lg font-bold text-primary">${challengeQuestion.question}</p>
                    
                    <form id="challenge-form" class="space-y-6">
                        ${challengeQuestion.tasks.map((task, index) => `
                            <div>
                                <label for="answer-${index}" class="block text-lg font-semibold mb-2">${index + 1}. ${task}</label>
                                <textarea id="answer-${index}" class="w-full p-3 border rounded-lg focus:ring-primary focus:border-primary" rows="4" placeholder="Sua resposta..."></textarea>
                            </div>
                        `).join('')}
                    </form>
                    
                    <button id="submit-challenge-btn" class="mt-8 w-full bg-primary text-white font-bold py-3 px-6 rounded-lg hover:opacity-90 flex items-center justify-center gap-2">
                        <i data-lucide="send"></i> Enviar Respostas e Gerar Relatório
                    </button>
                </div>
            </main>
        `;
        
        lucide.createIcons();
        document.getElementById('submit-challenge-btn').onclick = () => {
            const answers = challengeQuestion.tasks.map((task, index) => ({
                question: task,
                answer: document.getElementById(`answer-${index}`).value
            }));
            
            state.challengeAnswers = answers;
            state.currentStep = 'final_report_dashboard';
            render();
        };
    }
    
    function renderFinalReportDashboard() {
        const order = state.currentShipment.orders[0];
        
        const finalWeight = order.totalWeight;
        const finalValue = state.currentShipment.freightDetails.finalTotal;
        const totalIncidents = state.currentShipment.incidents.length;
        const finalDistance = state.currentShipment.route.totalDistance;

        appContainer.innerHTML += `
             <main class="flex-grow p-8">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold">Relatório Final da Operação</h2>
                    <button onclick="window.print()" class="bg-gray-500 text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2 hover:opacity-90"><i data-lucide="printer"></i> Imprimir Relatório</button>
                </div>

                <div class="bg-white p-6 rounded-lg shadow-lg mb-6">
                    <h3 class="text-2xl font-bold text-primary mb-4">Resumo da Operação (Pedido ${order.id})</h3>
                    <div class="grid grid-cols-1 md:grid-cols-4 gap-4 text-center">
                        <div class="bg-blue-50 p-4 rounded-lg">
                            <p class="text-sm font-semibold">Peso Entregue</p>
                            <p class="text-3xl font-bold text-primary">${finalWeight.toFixed(2)} kg</p>
                        </div>
                        <div class="bg-green-50 p-4 rounded-lg">
                            <p class="text-sm font-semibold">Valor Total da NF</p>
                            <p class="text-3xl font-bold text-highlight">R$ ${finalValue.toFixed(2)}</p>
                        </div>
                        <div class="bg-red-50 p-4 rounded-lg">
                            <p class="text-sm font-semibold">Não Conformidades (Sep.)</p>
                            <p class="text-3xl font-bold text-red-600">${totalIncidents}</p>
                        </div>
                        <div class="bg-yellow-50 p-4 rounded-lg">
                            <p class="text-sm font-semibold">Distância Percorrida</p>
                            <p class="text-3xl font-bold text-action">${finalDistance.toFixed(2)} km</p>
                        </div>
                    </div>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                    <div class="bg-white p-6 rounded-lg shadow-lg">
                        <h3 class="text-xl font-bold mb-4 border-b pb-2">Resultado da Avaliação de Conhecimento</h3>
                        <p class="text-4xl font-bold text-highlight text-center">${state.quizScore}/${4} Pontos</p>
                        <p class="text-center text-sm text-secondary mt-2">${((state.quizScore / 4) * 100).toFixed(0)}% de Acerto.</p>
                    </div>

                    <div class="bg-white p-6 rounded-lg shadow-lg">
                        <h3 class="text-xl font-bold mb-4 border-b pb-2">Respostas do Desafio de Gestão</h3>
                        ${state.challengeAnswers.map((item, index) => `
                            <div class="mb-4">
                                <p class="font-semibold text-sm text-primary">Questão ${index + 1}: ${item.question}</p>
                                <p class="text-gray-700 p-2 bg-gray-50 rounded-md whitespace-pre-wrap">${item.answer}</p>
                            </div>
                        `).join('')}
                    </div>
                </div>
                
                 <div class="mt-8 text-center">
                    <h3 class="text-2xl font-bold text-blue-600">FIM DA SIMULAÇÃO</h3>
                    <p class="text-lg text-secondary">Agradecemos por completar a operação WMS. Você pode fechar o navegador ou reiniciar a simulação.</p>
                    <button onclick="window.location.reload()" class="mt-4 bg-red-600 text-white font-bold py-3 px-6 rounded-lg hover:opacity-90 flex items-center justify-center gap-2 mx-auto">
                        <i data-lucide="rotate-ccw"></i> Reiniciar Simulação
                    </button>
                </div>
            </main>
        `;

        lucide.createIcons();
    }


    // --- PRINT FUNCTIONS ---
    function printContent(content) {
        const printArea = document.getElementById('print-area');
        printArea.innerHTML = content;
        window.print();
    }
    
    function downloadQualityForm() {
        const qc = state.qualityReport;
        const content = `
            <div class="p-6 bg-white rounded-lg shadow-lg border border-gray-300">
                <h1 style="text-align: center; color: var(--color-primary);">REGISTRO DE CONTROLE DE QUALIDADE</h1>
                <p style="text-align: center; margin-bottom: 20px;">Data de Registro: ${new Date(qc.timestamp).toLocaleDateString()} - ${new Date(qc.timestamp).toLocaleTimeString()}</p>
                
                <h2 style="color: var(--color-highlight);">Informações Gerais</h2>
                <table style="width: 100%; border-collapse: collapse; margin-bottom: 20px;">
                    <tr>
                        <th style="padding: 8px; border: 1px solid #ccc; background-color: #f2f2f2;">Operador de Qualidade</th>
                        <th style="padding: 8px; border: 1px solid #ccc; background-color: #f2f2f2;">ID do Pedido</th>
                        <th style="padding: 8px; border: 1px solid #ccc; background-color: #f2f2f2;">Cliente</th>
                    </tr>
                    <tr>
                        <td style="padding: 8px; border: 1px solid #ccc;">${qc.operator}</td>
                        <td style="padding: 8px; border: 1px solid #ccc;">${qc.orderId}</td>
                        <td style="padding: 8px; border: 1px solid #ccc;">${qc.customer}</td>
                    </tr>
                </table>

                <h2 style="color: var(--color-highlight);">Parâmetros Verificados</h2>
                <table style="width: 100%; border-collapse: collapse; margin-bottom: 20px;">
                     <tr>
                        <th style="padding: 8px; border: 1px solid #ccc; background-color: #f2f2f2;">Produto (Foco)</th>
                        <th style="padding: 8px; border: 1px solid #ccc; background-color: #f2f2f2;">Peso Verificado (kg)</th>
                        <th style="padding: 8px; border: 1px solid #ccc; background-color: #f2f2f2;">Valor Declarado (R$)</th>
                        <th style="padding: 8px; border: 1px solid #ccc; background-color: #f2f2f2;">Tempo Entrega (dias)</th>
                    </tr>
                    <tr>
                        <td style="padding: 8px; border: 1px solid #ccc;">${qc.product}</td>
                        <td style="padding: 8px; border: 1px solid #ccc;">${qc.weight}</td>
                        <td style="padding: 8px; border: 1px solid #ccc;">${qc.value}</td>
                        <td style="padding: 8px; border: 1px solid #ccc;">${qc.deliveryTime}</td>
                    </tr>
                </table>
                
                <h2 style="color: var(--color-highlight);">Observações</h2>
                <div style="border: 1px solid #ccc; padding: 10px; min-height: 80px;">${qc.notes || 'Nenhuma observação registrada.'}</div>
                
                <div style="margin-top: 50px; text-align: center;">
                    <p style="border-top: 1px solid #000; display: inline-block; padding: 5px 20px;">Assinatura do Operador de Qualidade</p>
                </div>
            </div>
        `;
        printContent(content);
    }

    function printRomaneio(order) {
        const orderTotalItems = Object.values(order.items).reduce((sum, qty) => sum + qty, 0);

        let itemsHtml = Object.entries(order.items).map(([id, qty]) => {
            const prod = state.products[id];
            return `
                <tr>
                    <td>${prod.barcode}</td>
                    <td>${prod.name}</td>
                    <td style="text-align: right;">${qty}</td>
                    <td style="text-align: right;">R$ ${prod.price.toFixed(2)}</td>
                    <td style="text-align: right;">R$ ${(qty * prod.price).toFixed(2)}</td>
                </tr>
            `;
        }).join('');

        let nonConformityHtml = state.currentShipment.incidents.map((nc) => `
            <tr>
                <td colspan="5" style="background-color: #f9eaea;">
                    <span style="font-weight: bold; color: #cc0000;">NÃO CONFORMIDADE:</span> Produto: ${nc.product} | Motivo: ${nc.reason} | Ação: ${nc.action}
                </td>
            </tr>
        `).join('');

        const finalTotal = state.currentShipment.freightDetails ? state.currentShipment.freightDetails.finalTotal : order.totalValue;

        const content = `
            <div style="font-family: sans-serif; color: black; padding: 20px;">
                <h1 style="text-align: center; color: var(--color-primary); font-size: 24px;">ROMANEIO DE CARGA - PEDIDO ${order.id}</h1>
                <p style="text-align: center; margin-bottom: 20px;">Data de Emissão: ${new Date().toLocaleDateString()} - Operador: ${state.userName}</p>
                
                <div class="order-block">
                    <h2 style="color: var(--color-highlight); font-size: 18px;">Dados da Entrega</h2>
                    <table style="width: 100%; border-collapse: collapse; margin-top: 10px;">
                        <tr>
                            <th style="width: 30%;">Cliente</th>
                            <td colspan="4">${order.customer}</td>
                        </tr>
                        <tr>
                            <th>Endereço</th>
                            <td colspan="4">${order.address.street}, ${order.address.state}</td>
                        </tr>
                         <tr>
                            <th>Itens Totais</th>
                            <td>${orderTotalItems}</td>
                            <th>Peso (kg)</th>
                            <td>${order.totalWeight.toFixed(2)}</td>
                            <th>Valor Total (R$)</th>
                            <td>${finalTotal.toFixed(2)}</td>
                        </tr>
                    </table>
                </div>

                <div class="order-block" style="margin-top: 30px;">
                    <h2 style="color: var(--color-highlight); font-size: 18px;">Lista de Produtos</h2>
                    <table style="width: 100%; border-collapse: collapse; margin-top: 10px;">
                        <thead>
                            <tr>
                                <th style="width: 15%;">Cód. Barra</th>
                                <th style="width: 40%;">Produto</th>
                                <th style="width: 10%; text-align: right;">Qtd.</th>
                                <th style="width: 15%; text-align: right;">Preço Un.</th>
                                <th style="width: 20%; text-align: right;">Subtotal</th>
                            </tr>
                        </thead>
                        <tbody>
                            ${itemsHtml}
                            ${nonConformityHtml}
                        </tbody>
                        <tfoot>
                            <tr style="font-weight: bold; background-color: #f2f2f2;">
                                <td colspan="4" style="text-align: right; padding: 8px;">TOTAL GERAL:</td>
                                <td style="text-align: right; padding: 8px;">R$ ${finalTotal.toFixed(2)}</td>
                            </tr>
                        </tfoot>
                    </table>
                </div>

                 <div class="signature-block">
                    <div style="width: 45%; float: left; margin-right: 10%;">
                        <p style="border-top: 1px solid #000; padding-top: 5px;">Assinatura do Conferente (WMS)</p>
                    </div>
                    <div style="width: 45%; float: left;">
                        <p style="border-top: 1px solid #000; padding-top: 5px;">Assinatura do Motorista</p>
                    </div>
                    <div style="clear: both;"></div>
                </div>

                 <div class="signature-block" style="margin-top: 50px;">
                    <p style="font-style: italic; font-size: 12px; text-align: center;">Este documento não tem valor fiscal, serve apenas como comprovante interno de separação e expedição da carga.</p>
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

    // --- INITIALIZATION ---\
    render();
});
</script>

</body>
</html>
