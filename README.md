<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bon de commande CLUBS/ASSOCIATIONS</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.sheetjs.com/xlsx-0.20.2/package/dist/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.8.2/jspdf.plugin.autotable.min.js"></script>
    <style>
        body { font-family: 'Inter', sans-serif; }
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap');
        .modal { display: none; }
        .modal.is-open { display: flex; }
        input[type="text"], input[type="number"], input[type="date"], select, textarea, input[type="password"] {
            background-color: #f3f4f6;
            border-color: #9ca3af;
        }
        input[type="text"]:focus, input[type="number"]:focus, input[type="date"]:focus, select:focus, textarea:focus, input[type="password"]:focus {
            --tw-ring-color: #4f46e5;
            border-color: #4f46e5;
        }
        .toast {
            transition: opacity 0.5s, transform 0.5s;
            transform: translateX(100%);
            opacity: 0;
        }
        .toast.show {
            transform: translateX(0);
            opacity: 1;
        }
        .stock-info {
            font-size: 0.75rem;
            color: #4b5563;
        }
        .highlight-section {
            box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.4), 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            border-radius: 0.75rem;
            transition: box-shadow 0.3s ease-in-out;
        }
    </style>
</head>
<body class="bg-gray-50">

    <div id="main-app-view">
        <input type="file" id="load-order-input" class="hidden" accept=".json">
        <input type="file" id="import-licensees-input" class="hidden" accept=".xlsx, .xls">
        <input type="file" id="import-stock-input" class="hidden" accept=".json">
        <input type="file" id="import-club-range-input" class="hidden" accept=".json">
        <input type="file" id="load-all-data-input" class="hidden" accept=".json">
        <div id="toast-container" class="fixed top-5 right-5 z-[100] space-y-3 w-80"></div>
        <div id="main-modal" class="modal fixed inset-0 bg-black bg-opacity-50 z-50 justify-center items-center p-4">
            <div class="bg-white rounded-lg shadow-xl p-6 w-full max-w-md relative">
                <button id="main-modal-close-btn" class="absolute top-3 right-3 text-gray-400 hover:text-gray-600">
                    <svg class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                </button>
                <h3 id="main-modal-title" class="text-xl font-bold text-gray-800 mb-4">Titre du Modal</h3>
                <div id="main-modal-body" class="text-gray-600 mb-6 max-h-[60vh] overflow-y-auto">Contenu du modal.</div>
                <div id="main-modal-actions" class="flex justify-end space-x-3"></div>
            </div>
        </div>
        <div class="container mx-auto p-4 sm:p-6 lg:p-8">
            <header class="mb-8 flex justify-between items-start">
                <div>
                    <h1 id="main-title" class="text-4xl font-extrabold text-gray-800 tracking-tight">Bon de commande</h1>
                    <p class="mt-2 text-lg text-gray-500">Créez et validez votre document.</p>
                    <p id="autosave-status" class="mt-1 text-xs text-gray-400" style="min-height: 1em;"></p>
                </div>
                <div class="flex items-center gap-4">
                    <button id="init-stock-btn" class="px-4 py-2 bg-orange-500 text-white text-sm font-medium rounded-md hover:bg-orange-600">Initialiser le Stock</button>
                    <button id="manage-stock-btn" class="px-4 py-2 bg-green-600 text-white text-sm font-medium rounded-md hover:bg-green-700">Gérer le stock</button>
                    <button id="session-manager-btn" class="px-4 py-2 bg-purple-600 text-white text-sm font-medium rounded-md hover:bg-purple-700">Gérer les sessions</button>
                </div>
            </header>
            <section id="dashboard-section" class="mb-8 bg-white p-4 rounded-xl shadow-lg">
                <h2 class="text-xl font-bold text-gray-800 mb-3">Tableau de bord</h2>
                <div class="grid grid-cols-2 md:grid-cols-5 gap-4 text-center">
                    <div>
                        <p class="text-sm text-gray-500">Total Hauts</p>
                        <p id="summary-total-hauts" class="text-2xl font-bold text-indigo-600">0</p>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500">Total Bas</p>
                        <p id="summary-total-bas" class="text-2xl font-bold text-indigo-600">0</p>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500">Total Accessoires</p>
                        <p id="summary-total-accessoires" class="text-2xl font-bold text-indigo-600">0</p>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500">Nb. Licenciés</p>
                        <p id="summary-total-licensees" class="text-2xl font-bold text-indigo-600">0</p>
                    </div>
                    <div>
                        <p class="text-sm text-gray-500">Articles en Stock</p>
                        <p id="summary-total-stock" class="text-2xl font-bold text-green-600">0</p>
                    </div>
                </div>
            </section>
            <main class="bg-white p-6 rounded-xl shadow-lg mt-6 space-y-8">
                <section id="info-section">
    <h2 class="text-2xl font-bold text-gray-800 border-b pb-3 mb-6">Informations sur le document</h2>
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

        <div>
            <label for="clubName" class="block text-sm font-medium text-gray-700">Nom du Club <span class="text-red-500">*</span></label>
            <input type="text" id="clubName" list="club-list" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
            <datalist id="club-list"></datalist>
        </div>
        <div>
            <label for="departement" class="block text-sm font-medium text-gray-700">Département <span class="text-red-500">*</span></label>
            <input type="text" id="departement" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
        </div>
        <div>
            <label for="clientNumber" class="block text-sm font-medium text-gray-700">N° Client</label>
            <input type="text" id="clientNumber" list="client-list" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
            <datalist id="client-list"></datalist>
        </div>
        <div>
            <label for="orderDate" class="block text-sm font-medium text-gray-700">Date</label>
            <input type="date" id="orderDate" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
        </div>

        <div id="order-scope-container">
            <label class="block text-sm font-medium text-gray-700">Type de saisie <span class="text-red-500">*</span></label>
            <div class="mt-2 flex flex-col sm:flex-row items-start sm:items-center space-y-2 sm:space-y-0 sm:space-x-4">
                <div class="flex items-center">
                    <input id="scope-global" name="scope" type="radio" value="global">
                    <label for="scope-global" class="ml-2 block text-sm text-gray-900">Globale</label>
                </div>
                <div class="flex items-center">
                    <input id="scope-licensee" name="scope" type="radio" value="licensee">
                    <label for="scope-licensee" class="ml-2 block text-sm text-gray-900">Par licencié</label>
                </div>
                <div class="flex items-center">
                    <input id="scope-session" name="scope" type="radio" value="session">
                    <label for="scope-session" class="ml-2 block text-sm text-gray-900">Session Licenciés</label>
                </div>
            </div>
        </div>
        <div class="flex items-center">
            <input id="doc-type-reassort" type="checkbox" class="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500">
            <label for="doc-type-reassort" class="ml-2 block text-sm text-gray-900">Réassort 2 mois</label>
        </div>
        <div>
            <label class="block text-sm font-medium text-gray-700">Canal de Vente</label>
            <div class="mt-2 flex items-center">
                <input id="store-order-check" type="checkbox" class="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500">
                <label for="store-order-check" class="ml-2 block text-sm text-gray-900">Commande Magasin</label>
            </div>
        </div>
        <div>
            <label class="block text-sm font-medium text-gray-700">Type de Création</label>
            <div class="mt-2 flex items-center">
                <input id="custom-creation-check" type="checkbox" class="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500">
                <label for="custom-creation-check" class="ml-2 block text-sm text-gray-900">Commande Création Personnalisée</label>
            </div>
        </div>
        
        <div id="licencieName-container" class="hidden p-4 rounded-xl lg:col-span-1 md:col-span-2 col-span-1">
            <label for="licencieName" class="block text-sm font-medium text-gray-700">Nom du licencié</label>
            <div class="flex flex-col gap-2 mt-1">
                <div class="flex items-center gap-2">
                    <input type="text" id="licencieName" list="licensee-datalist" class="block w-full rounded-md border-gray-300 shadow-sm" placeholder="Taper ou sélectionner un nom...">
                    <datalist id="licensee-datalist"></datalist>
                    <button id="manage-licensees-btn" class="px-3 py-2 bg-gray-600 text-white rounded-md hover:bg-gray-700 text-sm">Gérer</button>
                </div>
                <button id="next-licensee-btn" class="w-full px-3 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 text-sm">Valider & Suivant</button>
            </div>
        </div>
        <div>
            <label class="block text-sm font-medium text-gray-700">Remise appliquée par le club</label>
            <div class="mt-2 flex items-center">
                <input id="apply-discount-check" type="checkbox" class="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500">
                <label for="apply-discount-check" class="ml-2 block text-sm text-gray-900">Activer la remise</label>
            </div>
        </div>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-2">
            <button id="manage-club-range-btn" class="w-full mt-1 px-4 py-2 bg-slate-600 text-white text-sm font-medium rounded-md hover:bg-slate-700"> Gérer la Gamme du Club </button>
            <button id="manage-visuals-btn" class="w-full mt-1 px-4 py-2 bg-cyan-600 text-white text-sm font-medium rounded-md hover:bg-cyan-700"> Gérer les Visuels </button>
        </div>

        <div class="md:col-span-2 lg:col-span-3 border-t pt-4 mt-4 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
            <div>
                <label for="preOrderNumber" class="block text-sm font-medium text-gray-700">N° Précommande</label>
                <input type="text" id="preOrderNumber" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
            </div>
            <div>
                <label for="factoryDepartureDate" class="block text-sm font-medium text-gray-700">Départ Usine Prévu</label>
                <input type="date" id="factoryDepartureDate" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
            </div>
            <div class="lg:col-span-2">
                <label for="deliveryContact" class="block text-sm font-medium text-gray-700">N° Portable (pour livraison)</label>
                <input type="text" id="deliveryContact" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
            </div>
            <div class="md:col-span-2 lg:col-span-4">
                <label for="deliveryAddress" class="block text-sm font-medium text-gray-700">Adresse de Livraison</label>
                <textarea id="deliveryAddress" rows="4" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm"></textarea>
            </div>
        </div>

        <div class="md:col-span-2 lg:col-span-3">
            <label for="orderSpecificity" class="block text-sm font-medium text-gray-700">Spécificité Commande</label>
            <textarea id="orderSpecificity" rows="1" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm" placeholder="Notes générales sur la commande..."></textarea>
        </div>

        <div id="portal-buttons-container" class="md:col-span-2 lg:col-span-3 border-t pt-4 mt-4 space-y-4">
            <h3 class="text-lg font-semibold text-gray-700 mb-2">🚀 Portail Licenciés</h3>
            <div>
                <label for="portalSessionName" class="block text-sm font-medium text-gray-700">Nom de la session (Optionnel)</label>
                <input type="text" id="portalSessionName" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm" placeholder="Ex: Commande Hiver 2024">
                <p class="text-xs text-gray-500 mt-1">Donnez un nom unique pour cette session de commande afin de la retrouver facilement.</p>
            </div>
            <div>
                <label for="portalDeadline" class="block text-sm font-medium text-gray-700">Date butoir de la commande</label>
                <input type="date" id="portalDeadline" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
            </div>
            <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                <button id="select-portal-products-btn" class="w-full px-4 py-3 bg-gray-700 text-white font-bold rounded-md hover:bg-gray-800 shadow-md disabled:bg-gray-300 disabled:cursor-not-allowed"> 1. Sélectionner les articles </button>
                <button id="generate-portal-link-btn" class="w-full px-4 py-3 bg-teal-600 text-white font-bold rounded-md hover:bg-teal-700 shadow-md disabled:bg-teal-300 disabled:cursor-not-allowed" title="Veuillez d'abord sélectionner des articles."> 2. Inviter les licenciés </button>
                <button id="import-portal-submissions-btn" class="w-full px-4 py-3 bg-blue-600 text-white font-bold rounded-md hover:bg-blue-700 shadow-md"> 3. Importer les commandes </button>
            </div>
        </div>
    </div>
</section>
                <section id="active-licensee-banner" class="hidden my-6 bg-blue-100 border-l-4 border-blue-500 text-blue-700 p-4 rounded-r-lg shadow" role="alert">
                    <div class="flex justify-between items-center">
                        <div>
                            <p class="font-bold">Vous ajoutez des articles pour le licencié :</p>
                            <p id="banner-licensee-name" class="text-lg"></p>
                        </div>
                        <button id="clear-active-licensee-btn" class="ml-4 text-sm font-medium text-blue-800 hover:text-blue-600">&times; Changer/Annuler</button>
                    </div>
                </section>
                <section id="add-article-section">
                    <div class="flex justify-between items-center border-b pb-3 mb-6">
                        <h2 class="text-2xl font-bold text-gray-800">Ajouter un Article</h2>
                        <div id="toggle-products-view-container" class="hidden">
                            <button id="toggle-products-btn" class="px-3 py-1 bg-gray-200 text-gray-700 text-xs font-medium rounded-md hover:bg-gray-300"> Afficher tous les articles </button>
                        </div>
                    </div>
                    <div id="product-tabs-container" class="flex border-b border-gray-200">
                        <button data-tab="CYCLISME/RUNNING" class="product-tab-btn py-2 px-4 -mb-px font-medium text-sm border-b-2 border-indigo-500 text-indigo-600">CYCLISME/RUNNING</button>
                        <button data-tab="Accessoires" class="product-tab-btn py-2 px-4 -mb-px font-medium text-sm text-gray-500 hover:text-gray-700">Accessoires</button>
                        <button data-tab="GAMME ENFANTS" class="product-tab-btn py-2 px-4 -mb-px font-medium text-sm text-gray-500 hover:text-gray-700">GAMME ENFANTS</button>
                    </div>
                    <div id="article-section-blocker" class="hidden text-center p-8 border-2 border-dashed rounded-lg mt-6 bg-yellow-100 border-yellow-400">
                        <svg class="mx-auto h-12 w-12 text-yellow-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" aria-hidden="true"><path vector-effect="non-scaling-stroke" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 010.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" /></svg>
                        <h3 class="mt-2 text-sm font-medium text-yellow-900">Commencez par les informations du document</h3>
                        <p class="mt-1 text-sm text-yellow-800">Veuillez renseigner le <strong>Nom du Club</strong>, le <strong>Département</strong> et le <strong>Type de saisie</strong> pour débloquer la suite.</p>
                    </div>
                    <div id="product-form-container" class="pt-6"></div>
                </section>
                <section>
                    <div class="flex justify-between items-center border-b pb-3 mb-6">
                        <h2 class="text-2xl font-bold text-gray-800">Détails de la commande</h2>
                        <span id="total-articles-display" class="text-lg font-semibold text-gray-600">Total des articles : 0</span>
                    </div>
                    <div id="quantity-dashboard-container" class="mb-6 bg-gray-50 p-4 rounded-lg shadow-inner"></div>
                    <div class="overflow-x-auto">
                        <table class="min-w-max w-full divide-y divide-gray-200">
                            <thead id="order-table-head" class="bg-gray-50"></thead>
                            <tbody id="order-table-body" class="bg-white divide-y divide-gray-200">
                                <tr><td colSpan="8" class="px-6 py-12 text-center text-gray-500">Aucun article dans la commande.</td></tr>
                            </tbody>
                        </table>
                    </div>
                </section>
                <section id="summary-and-actions-section">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-x-8 gap-y-6">
                        <div id="discount-controls-container" class="hidden">
                            <label class="block text-sm font-medium text-gray-700">Type de remise</label>
                            <div class="mt-2 flex items-center space-x-4">
                                <div class="flex items-center">
                                    <input id="discount-global" name="discount-type" type="radio" value="global" checked>
                                    <label for="discount-global" class="ml-2 block text-sm text-gray-900">Globale</label>
                                </div>
                                <div class="flex items-center">
                                    <input id="discount-item" name="discount-type" type="radio" value="item">
                                    <label for="discount-item" class="ml-2 block text-sm text-gray-900">Par article</label>
                                </div>
                            </div>
                            <label for="clubDiscount" class="block text-sm font-medium text-gray-700 mt-4">Remise appliquée par le club (%)</label>
                            <input type="number" id="clubDiscount" value="0" class="mt-1 block w-full md:w-1/2 rounded-md border-gray-300 shadow-sm" placeholder="0">
                        </div>
                        <div class="space-y-2 text-right md:col-start-2">
                            <div class="text-md"><span class="font-medium text-gray-600">Sous-total HT: </span><span id="subtotal-ht" class="font-semibold text-gray-800">0.00€</span></div>
                            <div class="text-md"><span class="font-medium text-gray-600">Sous-total TTC: </span><span id="subtotal-ttc" class="font-semibold text-gray-800">0.00€</span></div>
                            <div class="text-md text-red-600"><span class="font-medium">Remise Club HT (Information): </span><span id="discount-amount-ht" class="font-semibold">-0.00€</span></div>
                            <div class="text-md text-red-600"><span class="font-medium">Remise Club TTC (Information): </span><span id="discount-amount-ttc" class="font-semibold">-0.00€</span></div>
                            <div class="text-md border-t pt-2 mt-2"><span class="font-medium text-gray-600">Frais de port HT: </span><span id="shipping-ht" class="font-semibold text-gray-800">0.00€</span></div>
                            <div class="text-md"><span class="font-medium text-gray-600">Frais de port TTC: </span><span id="shipping-ttc" class="font-semibold text-gray-800">0.00€</span></div>
                            <div id="graphic-fee-container" class="hidden">
                                <div class="text-md"><span class="font-medium text-gray-600">Forfait Création Graphique HT: </span><span id="graphic-fee-ht" class="font-semibold text-gray-800">0.00€</span></div>
                                <div class="text-md"><span class="font-medium text-gray-600">Forfait Création Graphique TTC: </span><span id="graphic-fee-ttc" class="font-semibold text-gray-800">0.00€</span></div>
                            </div>
                            <div class="text-xl mt-4"><span class="font-bold text-gray-700">Total Général HT: </span><span id="grand-total-ht" class="font-extrabold text-indigo-600">0.00€</span></div>
                            <div class="text-2xl"><span class="font-bold text-gray-700">Total Général TTC: </span><span id="grand-total-ttc" class="font-extrabold text-indigo-600">0.00€</span></div>
                            <div id="down-payment-container" class="text-xl mt-2 font-bold text-green-600"><span class="font-bold text-gray-700">Acompte à verser (30%): </span><span id="down-payment">0.00€</span></div>
                        </div>
                    </div>
                    <div class="mt-8 pt-6 border-t flex flex-col sm:flex-row justify-between items-center gap-4">
                        <button id="new-order-btn" class="w-full sm:w-auto px-6 py-3 border border-red-500 text-base font-medium rounded-md text-red-500 bg-white hover:bg-red-50">Nouvelle Commande</button>
                        <div class="flex flex-col sm:flex-row gap-4 w-full sm:w-auto">
    <button id="export-distribution-btn" class="w-full sm:w-auto inline-flex justify-center items-center px-6 py-3 border border-transparent text-base font-medium rounded-md shadow-sm text-white bg-teal-600 hover:bg-teal-700">Distribution (PDF)</button>
    <button id="save-order-btn" class="w-full sm:w-auto inline-flex justify-center items-center px-6 py-3 border border-transparent text-base font-medium rounded-md shadow-sm text-white bg-blue-600 hover:bg-blue-700">Exporter Fichier (.json)</button>
    <label id="import-order-label" for="load-order-input" class="w-full sm:w-auto inline-flex justify-center items-center px-6 py-3 border border-transparent text-base font-medium rounded-md shadow-sm text-white bg-gray-600 hover:bg-gray-700 cursor-pointer">Importer Fichier</label>
    <button id="validate-order-btn" class="w-full sm:w-auto inline-flex justify-center items-center px-8 py-3 border border-transparent text-base font-bold rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 disabled:bg-indigo-300">Valider la Commande</button>
</div>
                    </div>
                </section>
            </main>
        </div>
    </div>
    <div id="portal-view" class="hidden">
        <div class="container mx-auto p-4 sm:p-6 lg:p-8 min-h-screen flex flex-col items-center justify-center">
            <div class="w-full max-w-2xl bg-white p-8 rounded-xl shadow-2xl">
                <h1 id="portal-club-name" class="text-3xl font-bold text-center text-gray-800 mb-2">Commande Club</h1>
                <p class="text-center text-gray-500 mb-8">Veuillez entrer votre nom et sélectionner vos tailles.</p>
                <div class="space-y-6">
                    <div>
                        <label for="portal-licensee-name" class="block text-sm font-medium text-gray-700">Votre Nom Complet <span class="text-red-500">*</span></label>
                        <input type="text" id="portal-licensee-name" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm text-lg p-3">
                    </div>
                    <div id="portal-product-list" class="space-y-5 pt-5 border-t"></div>
                    <button id="portal-submit-btn" class="w-full mt-6 px-8 py-4 bg-indigo-600 text-white font-bold text-lg rounded-md hover:bg-indigo-700 shadow-lg"> Valider ma sélection </button>
                </div>
            </div>
        </div>
    </div>
    <script type="module">
// =================================================================================
// --- DATA & CONFIG ---
// =================================================================================

// Grille de prix pour les vestes HIVER CONFORT (prix d'origine)
const confortWinterJacketTiers = [
    { quantity: 1, price: 155.40 }, { quantity: 2, price: 133.20 }, { quantity: 3, price: 111.00 },
    { quantity: 5, price: 88.80 }, { quantity: 15, price: 84.36 }, { quantity: 25, price: 81.70 },
    { quantity: 50, price: 79.92 }, { quantity: 80, price: 77.26 }, { quantity: 150, price: 75.48 }
];

// Grille de prix pour les vestes HIVER THERMIQUE (d'après votre image)
const thermiqueWinterJacketTiers = [
    { quantity: 1, price: 159.60 }, { quantity: 2, price: 136.80 }, { quantity: 3, price: 114.00 },
    { quantity: 5, price: 91.20 }, { quantity: 15, price: 86.64 }, { quantity: 25, price: 83.90 },
    { quantity: 50, price: 82.08 }, { quantity: 80, price: 79.34 }, { quantity: 150, price: 77.52 }
];
const allAvailableProducts = [
    // =========== CYCLISME ===========
    { name: 'MAILLOT CLASSIQUE HOMME CONFORT MC', category: 'CYCLISME', type: 'haut', subtype: 'Maillots Manches Courtes', pricingGroup: 'maillotClassiqueMC', pricingTiers: [ { quantity: 1, price: 86.10 }, { quantity: 2, price: 73.80 }, { quantity: 3, price: 61.50 }, { quantity: 5, price: 49.20 }, { quantity: 15, price: 46.74 }, { quantity: 25, price: 45.26 }, { quantity: 50, price: 44.28 }, { quantity: 80, price: 42.80 }, { quantity: 150, price: 41.82 } ] },
    { name: 'MAILLOT CLASSIQUE FEMME CONFORT MC', category: 'CYCLISME', type: 'haut', subtype: 'Maillots Manches Courtes', pricingGroup: 'maillotClassiqueMC', pricingTiers: [ { quantity: 1, price: 86.10 }, { quantity: 2, price: 73.80 }, { quantity: 3, price: 61.50 }, { quantity: 5, price: 49.20 }, { quantity: 15, price: 46.74 }, { quantity: 25, price: 45.26 }, { quantity: 50, price: 44.28 }, { quantity: 80, price: 42.80 }, { quantity: 150, price: 41.82 } ] },
    { name: 'MAILLOT MIXTE CONFORT SANS MANCHE', category: 'CYCLISME', type: 'haut', subtype: 'Maillots Manches Courtes', pricingTiers: [ { quantity: 1, price: 86.10 }, { quantity: 2, price: 73.80 }, { quantity: 3, price: 61.50 }, { quantity: 5, price: 49.20 }, { quantity: 15, price: 46.74 }, { quantity: 25, price: 45.26 }, { quantity: 50, price: 44.28 }, { quantity: 80, price: 42.80 }, { quantity: 150, price: 41.82 } ] },
    { name: 'MAILLOT MIXTE PERFORMANCE MC', category: 'CYCLISME', type: 'haut', subtype: 'Maillots Manches Courtes', pricingGroup: 'maillotPerformanceMC', pricingTiers: [ { quantity: 1, price: 89.25 },{ quantity: 2, price: 76.50 }, { quantity: 3, price: 63.75 }, { quantity: 5, price: 51.00 }, { quantity: 15, price: 48.45 }, { quantity: 25, price: 46.92 }, { quantity: 50, price: 45.90 }, { quantity: 80, price: 44.37 }, { quantity: 150, price: 43.35 } ] },
       { name: 'MAILLOT VTT/DESCENTE MIXTE CONFORT MC (Très ample)', category: 'CYCLISME', type: 'haut', subtype: 'Maillots Manches Courtes', sizeType: 'largeHaut', hasOptions: false, pricingTiers: [ { quantity: 1, price: 77.70 }, { quantity: 2, price: 66.60 }, { quantity: 3, price: 55.50 }, { quantity: 5, price: 44.40 }, { quantity: 15, price: 42.18 }, { quantity: 25, price: 40.85 }, { quantity: 50, price: 39.96 }, { quantity: 80, price: 38.63 }, { quantity: 150, price: 37.74 } ] },
    { name: 'MAILLOT MIXTE AERO MC', category: 'CYCLISME', type: 'haut', subtype: 'Maillots Manches Courtes', sizeType: 'aero', pricingTiers: [ { quantity: 1, price: 96.60 }, { quantity: 2, price: 82.80 }, { quantity: 3, price: 69.00 }, { quantity: 5, price: 55.20 }, { quantity: 15, price: 52.44 }, { quantity: 25, price: 45.00 }, { quantity: 50, price: 44.10 }, { quantity: 80, price: 42.75 }, { quantity: 150, price: 41.88 } ] },
   { name: 'MAILLOT MI-SAISON HOMME CONFORT ML', category: 'CYCLISME', type: 'haut', subtype: 'Maillots Manches Longues', pricingGroup: 'maillotMiSaisonML', pricingTiers: [ { quantity: 1, price: 94.50 }, { quantity: 2, price: 81.00 }, { quantity: 3, price: 67.50 }, { quantity: 5, price: 54.00 }, { quantity: 15, price: 51.30 }, { quantity: 25, price: 49.68 }, { quantity: 50, price: 48.60 }, { quantity: 80, price: 46.98 }, { quantity: 150, price: 45.90 } ] },
{ name: 'MAILLOT MI-SAISON FEMME CONFORT ML', category: 'CYCLISME', type: 'haut', subtype: 'Maillots Manches Longues', pricingGroup: 'maillotMiSaisonML', pricingTiers: [ { quantity: 1, price: 94.50 }, { quantity: 2, price: 81.00 }, { quantity: 3, price: 67.50 }, { quantity: 5, price: 54.00 }, { quantity: 15, price: 51.30 }, { quantity: 25, price: 49.68 }, { quantity: 50, price: 48.60 }, { quantity: 80, price: 46.98 }, { quantity: 150, price: 45.90 } ] },
    { name: 'MAILLOT BMX MIXTE CONFORT ML (Très ample)', category: 'CYCLISME', type: 'haut', subtype: 'Maillots Manches Longues', sizeType: 'largeHaut', hasOptions: false, pricingTiers: [ { quantity: 1, price: 88.20 }, { quantity: 2, price: 75.60 }, { quantity: 3, price: 63.00 }, { quantity: 5, price: 50.40 }, { quantity: 15, price: 47.88 }, { quantity: 25, price: 46.37 }, { quantity: 50, price: 45.36 }, { quantity: 80, price: 43.85 }, { quantity: 150, price: 42.84 } ] },
    { name: 'MAILLOT MI-SAISON MIXTE AERO ML', category: 'CYCLISME', type: 'haut', subtype: 'Maillots Manches Longues', sizeType: 'aero', pricingTiers: [ { quantity: 1, price: 107.10 }, { quantity: 2, price: 91.80 }, { quantity: 3, price: 76.50 }, { quantity: 5, price: 61.20 }, { quantity: 15, price: 58.14 }, { quantity: 25, price: 56.30 }, { quantity: 50, price: 55.08 }, { quantity: 80, price: 53.24 }, { quantity: 150, price: 52.02 } ] },
    { name: 'MAILLOT PLUIE MIXTE AERO MC (non sublimé, marquage DTF)', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', price: 85.20 },
{ name: 'MAILLOT PLUIE MIXTE AERO ML (non sublimé, marquage DTF)', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', price: 102.00 },
   { name: 'GILET COUPE-VENT LEGER MIXTE (vent et pluie fine, sans poche dos)', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'giletCoupeVent', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 72.45 }, { quantity: 2, price: 62.10 }, { quantity: 3, price: 51.75 }, { quantity: 5, price: 41.40 }, { quantity: 15, price: 39.33 }, { quantity: 25, price: 38.09 }, { quantity: 50, price: 37.26 }, { quantity: 80, price: 36.02 }, { quantity: 150, price: 35.19 } ] },
    { name: 'GILET COUPE-VENT MI-SAISON MIXTE (dos ajouré)', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'giletCoupeVent', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 91.35 }, { quantity: 2, price: 78.30 }, { quantity: 3, price: 65.25 }, { quantity: 5, price: 52.20 }, { quantity: 15, price: 49.59 }, { quantity: 25, price: 48.02 }, { quantity: 50, price: 46.98 }, { quantity: 80, price: 45.41 }, { quantity: 150, price: 44.37 } ] },
    { name: 'GILET COUPE-VENT HIVER MIXTE (tout membranné)', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'giletCoupeVent', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 97.65 }, { quantity: 2, price: 83.70 }, { quantity: 3, price: 69.75 }, { quantity: 5, price: 55.80 }, { quantity: 15, price: 53.01 }, { quantity: 25, price: 51.34 }, { quantity: 50, price: 50.22 }, { quantity: 80, price: 48.55 }, { quantity: 150, price: 47.43 } ] },
    { name: 'COUPE-VENT LEGER MIXTE CONFORT (vent et pluie fine)', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'coupeVent', pricingTiers: [ { quantity: 1, price: 96.60 }, { quantity: 2, price: 82.80 }, { quantity: 3, price: 69.00 }, { quantity: 5, price: 55.20 }, { quantity: 15, price: 52.44 }, { quantity: 25, price: 50.78 }, { quantity: 50, price: 49.68 }, { quantity: 80, price: 48.02 }, { quantity: 150, price: 46.92 } ] },
    { name: 'COUPE-VENT LEGER DEPERLANT MIXTE CONFORT (avec membranne)', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'coupeVent', pricingTiers: [ { quantity: 1, price: 128.10 }, { quantity: 2, price: 109.80 }, { quantity: 3, price: 91.50 }, { quantity: 5, price: 73.20 }, { quantity: 15, price: 69.54 }, { quantity: 25, price: 67.34 }, { quantity: 50, price: 65.88 }, { quantity: 80, price: 63.68 }, { quantity: 150, price: 62.22 } ] },
    { name: 'VESTE MI-SAISON MIXTE CONFORT (membranne coupe-vent + mi-saison)', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'vesteMiSaison', pricingTiers: [ { quantity: 1, price: 136.50 }, { quantity: 2, price: 117.00 }, { quantity: 3, price: 97.50 }, { quantity: 5, price: 78.00 }, { quantity: 15, price: 74.10 }, { quantity: 25, price: 71.76 }, { quantity: 50, price: 70.20 }, { quantity: 80, price: 67.86 }, { quantity: 150, price: 66.30 } ] },
    { name: 'VESTE MI-SAISON MIXTE CONFORT avec -6cm aux ML', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'vesteMiSaison', pricingTiers: [ { quantity: 1, price: 136.50 }, { quantity: 2, price: 117.00 }, { quantity: 3, price: 97.50 }, { quantity: 5, price: 78.00 }, { quantity: 15, price: 74.10 }, { quantity: 25, price: 71.76 }, { quantity: 50, price: 70.20 }, { quantity: 80, price: 67.86 }, { quantity: 150, price: 66.30 } ] },
    
    // ▼▼▼ BLOC CORRIGÉ ▼▼▼
    { name: 'VESTE HIVER HOMME CONFORT', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'vesteHiver', pricingTiers: confortWinterJacketTiers },
    { name: 'VESTE HIVER FEMME CONFORT', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'vesteHiver', pricingTiers: confortWinterJacketTiers },
    { name: 'VESTE HIVER THERMIQUE HOMME CONFORT', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'vesteHiver', pricingTiers: thermiqueWinterJacketTiers },
    { name: 'VESTE HIVER THERMIQUE FEMME CONFORT', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'vesteHiver', pricingTiers: thermiqueWinterJacketTiers },
    // ▲▲▲ FIN DU BLOC ▲▲▲
    { name: 'CUISSARD A BRETELLES HOMME CONFORT Peau LANDSCAPE', category: 'CYCLISME', type: 'haut', subtype: 'Cuissards Courts', isCuissardOrCollant: true, pricingGroup: 'cuissardConfortLandscape', pricingTiers: [ { quantity: 1, price: 121.80 }, { quantity: 2, price: 104.40 }, { quantity: 3, price: 87.00 }, { quantity: 5, price: 69.60 }, { quantity: 15, price: 66.12 }, { quantity: 25, price: 64.03 }, { quantity: 50, price: 62.64 }, { quantity: 80, price: 60.55 }, { quantity: 150, price: 59.16 }, ] },
    { name: 'CUISSARD A BRETELLES FEMME CONFORT Peau LANDSCAPE', category: 'CYCLISME', type: 'haut', subtype: 'Cuissards Courts', isCuissardOrCollant: true, pricingGroup: 'cuissardConfortLandscape', pricingTiers: [ { quantity: 1, price: 121.80 }, { quantity: 2, price: 104.40 }, { quantity: 3, price: 87.00 }, { quantity: 5, price: 69.60 }, { quantity: 15, price: 66.12 }, { quantity: 25, price: 64.03 }, { quantity: 50, price: 62.64 }, { quantity: 80, price: 60.55 }, { quantity: 150, price: 59.16 }, ] },
    { name: 'CUISSARD FEMME SANS BRETELLES CONFORT Peau LANDSCAPE', category: 'CYCLISME', type: 'haut', subtype: 'Cuissards Courts', isCuissardOrCollant: true, pricingGroup: 'cuissardConfortLandscape', pricingTiers: [ { quantity: 1, price: 117.60 }, { quantity: 2, price: 100.80 }, { quantity: 3, price: 84.00 }, { quantity: 5, price: 67.20 }, { quantity: 15, price: 63.84 }, { quantity: 25, price: 61.82 }, { quantity: 50, price: 60.48 }, { quantity: 80, price: 58.46 }, { quantity: 150, price: 57.12 }, ] },
    { name: 'CUISSARD HOMME AERO Peau CERVINO', category: 'CYCLISME', type: 'haut', subtype: 'Cuissards Courts', sizeType: 'aero', isCuissardOrCollant: true, pricingGroup: 'cuissardAeroCervino', pricingTiers: [ { quantity: 1, price: 142.80 }, { quantity: 2, price: 122.40 }, { quantity: 3, price: 102.00 }, { quantity: 5, price: 81.60 }, { quantity: 15, price: 77.52 }, { quantity: 25, price: 75.07 }, { quantity: 50, price: 73.44 }, { quantity: 80, price: 70.99 }, { quantity: 150, price: 69.36 }, ] },
    { name: 'CUISSARD FEMME AERO Peau CERVINO', category: 'CYCLISME', type: 'haut', subtype: 'Cuissards Courts', sizeType: 'aero', isCuissardOrCollant: true, pricingGroup: 'cuissardAeroCervino', pricingTiers: [ { quantity: 1, price: 142.80 }, { quantity: 2, price: 122.40 }, { quantity: 3, price: 102.00 }, { quantity: 5, price: 81.60 }, { quantity: 15, price: 77.52 }, { quantity: 25, price: 75.07 }, { quantity: 50, price: 73.44 }, { quantity: 80, price: 70.99 }, { quantity: 150, price: 69.36 }, ] },
    { name: 'SHORT VTT FOND Peau ENDURANCE 2.5', category: 'CYCLISME', type: 'haut', subtype: 'Cuissards Courts', isCuissardOrCollant: true, pricingTiers: [ { quantity: 1, price: 75.00 } ] },
    { name: 'CORSAIRE HOMME A BRETELLES CONFORT', category: 'CYCLISME', type: 'haut', subtype: 'Corsaires/Collants', sizeType: 'ample', isCuissardOrCollant: true, pricingGroup: 'corsaireConfortLandscape', pricingTiers: [ { quantity: 1, price: 79.20 }, { quantity: 2, price: 79.20 }, { quantity: 3, price: 79.20 }, { quantity: 5, price: 79.20 }, { quantity: 15, price: 75.24 }, { quantity: 25, price: 72.86 }, { quantity: 50, price: 71.28 }, { quantity: 80, price: 68.90 }, { quantity: 150, price: 67.32 } ] },
    { name: 'CORSAIRE FEMME SANS BRETELLES CONFORT', category: 'CYCLISME', type: 'haut', subtype: 'Corsaires/Collants', sizeType: 'ample', isCuissardOrCollant: true, pricingGroup: 'corsaireConfortLandscape', pricingTiers: [ { quantity: 1, price: 76.80 }, { quantity: 2, price: 76.80 }, { quantity: 3, price: 76.80 }, { quantity: 5, price: 73.00 }, { quantity: 15, price: 72.96 }, { quantity: 25, price: 70.66 }, { quantity: 50, price: 69.12 }, { quantity: 80, price: 66.82 }, { quantity: 150, price: 65.28 } ] },
    { name: 'COLLANT HIVER A BRETELLES HOMME CONFORT', category: 'CYCLISME', type: 'haut', subtype: 'Corsaires/Collants', sizeType: 'ample', isCuissardOrCollant: true, pricingGroup: 'collantHiverConfortLandscape', pricingTiers: [ { quantity: 1, price: 138.60 }, { quantity: 2, price: 118.80 }, { quantity: 3, price: 99.00 }, { quantity: 5, price: 79.20 }, { quantity: 15, price: 75.24 }, { quantity: 25, price: 72.86 }, { quantity: 50, price: 71.28 }, { quantity: 80, price: 68.90 }, { quantity: 150, price: 67.32 }, ] },
    { name: 'COLLANT HIVER A BRETELLES FEMME CONFORT', category: 'CYCLISME', type: 'haut', subtype: 'Corsaires/Collants', sizeType: 'ample', isCuissardOrCollant: true, pricingGroup: 'collantHiverConfortLandscape', pricingTiers: [ { quantity: 1, price: 138.60 }, { quantity: 2, price: 118.80 }, { quantity: 3, price: 99.00 }, { quantity: 5, price: 79.20 }, { quantity: 15, price: 75.24 }, { quantity: 25, price: 72.86 }, { quantity: 50, price: 71.28 }, { quantity: 80, price: 68.90 }, { quantity: 150, price: 67.32 }, ] },
    { name: 'COLLANT HIVER FEMME SANS BRETELLES CONFORT', category: 'CYCLISME', type: 'haut', subtype: 'Corsaires/Collants', sizeType: 'ample', isCuissardOrCollant: true, pricingGroup: 'collantHiverConfortLandscape', pricingTiers: [ { quantity: 1, price: 134.40 }, { quantity: 2, price: 115.20 }, { quantity: 3, price: 96.00 }, { quantity: 5, price: 76.80 }, { quantity: 15, price: 72.96 }, { quantity: 25, price: 70.66 }, { quantity: 50, price: 69.12 }, { quantity: 80, price: 66.82 }, { quantity: 150, price: 65.28 }, ] },
    { name: 'COLLANT HIVER HOMME AERO Peau CERVINO', category: 'CYCLISME', type: 'haut', subtype: 'Corsaires/Collants', pricingGroup: 'collantHiverAeroCervino', sizeType: 'aero', isCuissardOrCollant: true, pricingTiers: [ { quantity: 1, price: 165.90 }, { quantity: 2, price: 142.20 }, { quantity: 3, price: 118.50 }, { quantity: 5, price: 94.80 }, { quantity: 15, price: 90.06 }, { quantity: 25, price: 87.22 }, { quantity: 50, price: 85.32 }, { quantity: 80, price: 82.48 }, { quantity: 150, price: 80.58 }, ] },
    { name: 'COLLANT HIVER FEMME AERO Peau CERVINO', category: 'CYCLISME', type: 'haut', subtype: 'Corsaires/Collants', pricingGroup: 'collantHiverAeroCervino', sizeType: 'aero', isCuissardOrCollant: true, pricingTiers: [ { quantity: 1, price: 165.90 }, { quantity: 2, price: 142.20 }, { quantity: 3, price: 118.50 }, { quantity: 5, price: 94.80 }, { quantity: 15, price: 90.06 }, { quantity: 25, price: 87.22 }, { quantity: 50, price: 85.32 }, { quantity: 80, price: 82.48 }, { quantity: 150, price: 80.58 }, ] },
    { name: 'COLLANT MIXTE ECHAUFFEMENT', category: 'CYCLISME', type: 'haut', subtype: 'Corsaires/Collants', sizeType: 'ample', isCuissardOrCollant: true, pricingTiers: [ { quantity: 1, price: 98.70 }, { quantity: 2, price: 84.60 }, { quantity: 3, price: 70.50 }, { quantity: 5, price: 56.40 }, { quantity: 15, price: 53.58 }, { quantity: 25, price: 51.89 }, { quantity: 50, price: 50.76 }, { quantity: 80, price: 49.07 }, { quantity: 150, price: 47.94 }, ] },
    { name: 'COMBINAISON ROUTE MANCHES COURTES HOMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 115.20 }] },
    { name: 'COMBINAISON ROUTE MANCHES COURTES FEMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 115.20 }] },
    { name: 'COMBINAISON CHRONO ROUTE MANCHES COURTES HOMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 115.20 }] },
    { name: 'COMBINAISON CHRONO ROUTE MANCHES COURTES FEMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 115.20 }] },
    { name: 'COMBINAISON CHRONO ROUTE MANCHES LONGUES HOMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 120.00 }] },
    { name: 'COMBINAISON CHRONO ROUTE MANCHES LONGUES FEMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 120.00 }] },
    { name: 'COMBINAISON CHRONO PISTE MANCHES COURTES HOMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 111.60 }] },
    { name: 'COMBINAISON CHRONO PISTE MANCHES COURTES FEMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 111.60 }] },
    { name: 'COMBINAISON CHRONO PISTE MANCHES LONGUES HOMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 116.40 }] },
    { name: 'COMBINAISON CHRONO PISTE MANCHES LONGUES FEMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 111.60 }] },
    { name: 'COMBINAISON CYCLO-CROSS MANCHES LONGUES HOMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 120.00 }] },
    { name: 'COMBINAISON CYCLO-CROSSMANCHES LONGUES FEMME AERO', category: 'CYCLISME', type: 'haut', subtype: 'Combinaisons', sizeType: 'aero', pricingTiers: [{ quantity: 1, price: 120.00 }] },
    { name: 'MAILLOT RUNNING HOMME', category: 'RUNNING', type: 'haut', subtype: 'Hauts', pricingGroup: 'maillotRunning', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 63.00 }, { quantity: 2, price: 54.00 }, { quantity: 3, price: 45.00 }, { quantity: 5, price: 36.00 }, { quantity: 15, price: 34.20 }, { quantity: 25, price: 33.12 }, { quantity: 50, price: 32.40 }, { quantity: 80, price: 31.32 }, { quantity: 150, price: 30.60 } ] },
    { name: 'MAILLOT RUNNING FEMME', category: 'RUNNING', type: 'haut', subtype: 'Hauts', pricingGroup: 'maillotRunning', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 63.00 }, { quantity: 2, price: 54.00 }, { quantity: 3, price: 45.00 }, { quantity: 5, price: 36.00 }, { quantity: 15, price: 34.20 }, { quantity: 25, price: 33.12 }, { quantity: 50, price: 32.40 }, { quantity: 80, price: 31.32 }, { quantity: 150, price: 30.60 } ] },
    { name: 'DEBARDEUR ATHLETISME HOMME', category: 'RUNNING', type: 'haut', subtype: 'Hauts', pricingGroup: 'debardeurAthletisme', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 56.70 }, { quantity: 2, price: 48.60 }, { quantity: 3, price: 40.50 }, { quantity: 5, price: 32.40 }, { quantity: 15, price: 30.78 }, { quantity: 25, price: 29.81 }, { quantity: 50, price: 29.16 }, { quantity: 80, price: 28.19 }, { quantity: 150, price: 27.54 } ] },
    { name: 'DEBARDEUR ATHLETISME FEMME', category: 'RUNNING', type: 'haut', subtype: 'Hauts', pricingGroup: 'debardeurAthletisme', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 56.70 }, { quantity: 2, price: 48.60 }, { quantity: 3, price: 40.50 }, { quantity: 5, price: 32.40 }, { quantity: 15, price: 30.78 }, { quantity: 25, price: 29.81 }, { quantity: 50, price: 29.16 }, { quantity: 80, price: 28.19 }, { quantity: 150, price: 27.54 } ] },
    { name: 'BRASSIERE RUNNING FEMME', category: 'RUNNING', type: 'haut', subtype: 'Hauts', hasOptions: false, pricingTiers: [ { quantity: 1, price: 68.25 }, { quantity: 2, price: 58.50 }, { quantity: 3, price: 48.75 }, { quantity: 5, price: 39.00 }, { quantity: 15, price: 37.05 }, { quantity: 25, price: 35.88 }, { quantity: 50, price: 35.10 }, { quantity: 80, price: 33.93 }, { quantity: 150, price: 33.15 } ] },
    { name: 'MAILLOT RUNNING HIVER HOMME MANCHES LONGUES', category: 'RUNNING', type: 'haut', subtype: 'Hauts', pricingGroup: 'maillotRunningHiver', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 94.50 }, { quantity: 2, price: 81.00 }, { quantity: 3, price: 67.50 }, { quantity: 5, price: 54.00 }, { quantity: 15, price: 51.30 }, { quantity: 25, price: 49.68 }, { quantity: 50, price: 48.60 }, { quantity: 80, price: 46.98 }, { quantity: 150, price: 45.90 } ] },
    { name: 'MAILLOT RUNNING HIVER FEMME MANCHES LONGUES', category: 'RUNNING', type: 'haut', subtype: 'Hauts', pricingGroup: 'maillotRunningHiver', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 94.50 }, { quantity: 2, price: 81.00 }, { quantity: 3, price: 67.50 }, { quantity: 5, price: 54.00 }, { quantity: 15, price: 51.30 }, { quantity: 25, price: 49.68 }, { quantity: 50, price: 48.60 }, { quantity: 80, price: 46.98 }, { quantity: 150, price: 45.90 } ] },
    { name: 'GILET COUPE-VENT LEGER MIXTE (vent et pluie fine, sans poche dos)', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'giletCoupeVent', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 72.45 }, { quantity: 2, price: 62.10 }, { quantity: 3, price: 51.75 }, { quantity: 5, price: 41.40 }, { quantity: 15, price: 39.33 }, { quantity: 25, price: 38.09 }, { quantity: 50, price: 37.26 }, { quantity: 80, price: 36.02 }, { quantity: 150, price: 35.19 } ] },
    { name: 'GILET COUPE-VENT MI-SAISON MIXTE (dos ajouré)', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'giletCoupeVent', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 91.35 }, { quantity: 2, price: 78.30 }, { quantity: 3, price: 65.25 }, { quantity: 5, price: 52.20 }, { quantity: 15, price: 49.59 }, { quantity: 25, price: 48.02 }, { quantity: 50, price: 46.98 }, { quantity: 80, price: 45.41 }, { quantity: 150, price: 44.37 } ] },
    { name: 'GILET COUPE-VENT HIVER MIXTE (tout membranné)', category: 'CYCLISME', type: 'haut', subtype: 'Essentiels et Vestes', pricingGroup: 'giletCoupeVent', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 97.65 }, { quantity: 2, price: 83.70 }, { quantity: 3, price: 69.75 }, { quantity: 5, price: 55.80 }, { quantity: 15, price: 53.01 }, { quantity: 25, price: 51.34 }, { quantity: 50, price: 50.22 }, { quantity: 80, price: 48.55 }, { quantity: 150, price: 47.43 } ] },

    { name: 'COUPE-VENT LEGER MIXTE CONFORT', category: 'RUNNING', type: 'haut', subtype: 'Hauts', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 96.60 }, { quantity: 2, price: 82.80 }, { quantity: 3, price: 69.00 }, { quantity: 5, price: 55.20 }, { quantity: 15, price: 52.44 }, { quantity: 25, price: 50.78 }, { quantity: 50, price: 49.68 }, { quantity: 80, price: 48.02 }, { quantity: 150, price: 46.92 } ] },
    { name: 'VESTE MI-SAISON HOMME CONFORT', category: 'RUNNING', type: 'haut', subtype: 'Hauts', pricingTiers: [ { quantity: 1, price: 136.50 }, { quantity: 2, price: 117.00 }, { quantity: 3, price: 97.50 }, { quantity: 5, price: 78.00 }, { quantity: 15, price: 74.10 }, { quantity: 25, price: 71.76 }, { quantity: 50, price: 70.20 }, { quantity: 80, price: 67.86 }, { quantity: 150, price: 66.30 } ] },
    { name: 'VESTE MI-SAISON FEMME CONFORT', category: 'RUNNING', type: 'haut', subtype: 'Hauts', pricingTiers: [ { quantity: 1, price: 136.50 }, { quantity: 2, price: 117.00 }, { quantity: 3, price: 97.50 }, { quantity: 5, price: 78.00 }, { quantity: 15, price: 74.10 }, { quantity: 25, price: 71.76 }, { quantity: 50, price: 70.20 }, { quantity: 80, price: 67.86 }, { quantity: 150, price: 66.30 } ] },
    { name: 'SHORT RUNNING MIXTE', category: 'RUNNING', type: 'haut', subtype: 'Bas', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 79.80 }, { quantity: 2, price: 68.40 }, { quantity: 3, price: 57.00 }, { quantity: 5, price: 45.60 }, { quantity: 15, price: 43.32 }, { quantity: 25, price: 41.95 }, { quantity: 50, price: 41.04 }, { quantity: 80, price: 39.67 }, { quantity: 150, price: 38.76 } ] },
    { name: 'SHORTY FEMME RUNNING', category: 'RUNNING', type: 'haut', subtype: 'Bas', pricingGroup: 'cuissardShortyRunning', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 84.00 }, { quantity: 2, price: 72.00 }, { quantity: 3, price: 60.00 }, { quantity: 5, price: 48.00 }, { quantity: 15, price: 45.60 }, { quantity: 25, price: 44.16 }, { quantity: 50, price: 43.20 }, { quantity: 80, price: 41.76 }, { quantity: 150, price: 40.80 } ] },
    { name: 'CUISSARD RUNNING HOMME', category: 'RUNNING', type: 'haut', subtype: 'Bas', pricingGroup: 'cuissardShortyRunning', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 84.00 }, { quantity: 2, price: 72.00 }, { quantity: 3, price: 60.00 }, { quantity: 5, price: 48.00 }, { quantity: 15, price: 45.60 }, { quantity: 25, price: 44.16 }, { quantity: 50, price: 43.20 }, { quantity: 80, price: 41.76 }, { quantity: 150, price: 40.80 } ] },
    { name: 'COLLANT RUNNING MIXTE', category: 'RUNNING', type: 'haut', subtype: 'Bas', excludedOptions: ['POCHE DOS ZIPPEE'], pricingTiers: [ { quantity: 1, price: 105.00 }, { quantity: 2, price: 90.00 }, { quantity: 3, price: 75.00 }, { quantity: 5, price: 60.00 }, { quantity: 15, price: 57.00 }, { quantity: 25, price: 55.20 }, { quantity: 50, price: 54.00 }, { quantity: 80, price: 52.20 }, { quantity: 150, price: 51.00 } ] },
   { name: 'TRIFONCTION HOMME COURTE ET MOYENNE DISTANCE Peau TRI GEL', category: 'RUNNING', type: 'haut', subtype: 'Trifonctions', hasOptions: false, pricingGroup: 'trifonctionCourte', pricingTiers: [ { quantity: 1, price: 102.00 }, { quantity: 2, price: 102.00 }, { quantity: 3, price: 102.00 }, { quantity: 5, price: 102.00 }, { quantity: 15, price: 96.90 }, { quantity: 25, price: 93.84 }, { quantity: 50, price: 91.80 }, { quantity: 80, price: 88.74 }, { quantity: 150, price: 86.70 } ] },
{ name: 'TRIFONCTION FEMME COURTE ET MOYENNE DISTANCE Peau TRI GEL', category: 'RUNNING', type: 'haut', subtype: 'Trifonctions', hasOptions: false, pricingGroup: 'trifonctionCourte', pricingTiers: [ { quantity: 1, price: 102.00 }, { quantity: 2, price: 102.00 }, { quantity: 3, price: 102.00 }, { quantity: 5, price: 102.00 }, { quantity: 15, price: 96.90 }, { quantity: 25, price: 93.84 }, { quantity: 50, price: 91.80 }, { quantity: 80, price: 88.74 }, { quantity: 150, price: 86.70 } ] },
    { name: 'TRIFONCTION HOMME LONGUE DISTANCE Peau TRI GEL, ZIP DEVANT OU DOS', category: 'RUNNING', type: 'haut', subtype: 'Trifonctions', hasOptions: false, pricingGroup: 'trifonctionHalf', pricingTiers: [ { quantity: 1, price: 114.00 }, { quantity: 2, price: 114.00 }, { quantity: 3, price: 114.00 }, { quantity: 5, price: 114.00 }, { quantity: 15, price: 108.30 }, { quantity: 25, price: 104.88 }, { quantity: 50, price: 102.60 }, { quantity: 80, price: 99.18 }, { quantity: 150, price: 96.90 } ] },
{ name: 'TRIFONCTION FEMME LONGUE DISTANCE Peau TRI GEL, ZIP DEVANT OU DOS', category: 'RUNNING', type: 'haut', subtype: 'Trifonctions', hasOptions: false, pricingGroup: 'trifonctionHalf', pricingTiers: [ { quantity: 1, price: 114.00 }, { quantity: 2, price: 114.00 }, { quantity: 3, price: 114.00 }, { quantity: 5, price: 114.00 }, { quantity: 15, price: 108.30 }, { quantity: 25, price: 104.88 }, { quantity: 50, price: 102.60 }, { quantity: 80, price: 99.18 }, { quantity: 150, price: 96.90 } ] },
    { name: 'BANDANA ÉTÉ', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'unique', minQuantity: 10, pricingTiers: [ { quantity: 10, price: 12.00 }, { quantity: 20, price: 10.44 }, { quantity: 50, price: 10.20 } ] },
    { name: 'BANDEAU', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'unique', minQuantity: 10, pricingTiers: [ { quantity: 10, price: 9.00 },  { quantity: 20, price: 8.40 }, { quantity: 50, price: 7.20 } ] },
    { name: 'TOUR DE COU', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'unique', minQuantity: 10, pricingTiers: [ { quantity: 10, price: 10.20 }, { quantity: 20, price: 8.70 }, { quantity: 50, price: 8.40 } ] },
    { name: 'PASSE MONTAGNE', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'unique', minQuantity: 10, pricingTiers: [ { quantity: 10, price: 18.00 }, { quantity: 20, price: 15.60 }, { quantity: 50, price: 14.40 } ] },
    { name: 'MANCHETTES ETE VELO/RUNNING', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'manchettes', minQuantity: 10, pricingTiers: [ { quantity: 10, price: 15.00 }, { quantity: 20, price: 14.10 }, { quantity: 50, price: 12.90 } ] },
{ name: 'MANCHETTES HIVER VELO/RUNNING', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'manchettes', minQuantity: 10, pricingTiers: [ { quantity: 10, price: 20.40 }, { quantity: 20, price: 18.00 }, { quantity: 50, price: 16.80 } ] },
    { name: 'JAMBIERES', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'jambieres', minQuantity: 10, pricingTiers: [ { quantity: 10, price: 26.40 }, { quantity: 20, price: 24.00 }, { quantity: 50, price: 22.80 } ] },
    { name: 'GANTS ÉTÉ', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'gants', minQuantity: 10, pricingTiers: [ { quantity: 10, price: 21.00 }, { quantity: 20, price: 18.00 }, { quantity: 50, price: 16.80 } ] },
    { name: 'GANTS ÉTÉ SLIM', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'gants', minQuantity: 10, pricingTiers: [ { quantity: 10, price: 30.00 }, { quantity: 20, price: 28.20 }, { quantity: 50, price: 25.20 } ] },
    { name: 'TAPIS DE TRANSITION MULTISPORTS', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'unique', minQuantity: 10, pricingTiers: [ { quantity: 10, price: 10.80 }, { quantity: 20, price: 9.18 }, { quantity: 50, price: 8.40 } ] },
    { name: 'CHAUSSETTES AERO MIXTE 18cm', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'chaussettes', minQuantity: 10, pricingTiers: [ { quantity: 10, price: 21.00 }, { quantity: 20, price: 20.40 }, { quantity: 50, price: 19.20 } ] },
    { name: 'CHAUSSETTES VELO/COURSE A PIED Mixte Tige 13 ou 17cm', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'chaussettes', minQuantity: 50, pricingTiers: [ { quantity: 50, price: 13.08 }, { quantity: 100, price: 11.88 }, { quantity: 200, price: 11.88 } ] },
    { name: 'GAPETTE VELO', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'unique', minQuantity: 50, pricingTiers: [ { quantity: 50, price: 15.00 }, { quantity: 100, price: 13.20 }, { quantity: 200, price: 13.20 } ] },
    { name: 'DOSSARDS JEU DE 1 à 100', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'unique', pricingTiers: [{quantity: 1, price: 68.80}] },
    { name: 'DOSSARDS JEU DE 1 à 150', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'unique', pricingTiers: [{quantity: 1, price: 91.20}] },
    { name: 'DOSSARDS JEU DE 1 à 200', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'unique', pricingTiers: [{quantity: 1, price: 115.20}] },
    { name: 'DOSSARDS JEU DE 1 à 250', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'unique', pricingTiers: [{quantity: 1, price: 136.00}] },
    { name: 'DOSSARDS JEU DE 1 à 300', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERSONNALISÉS', sizeType:'unique', pricingTiers: [{quantity: 1, price: 158.40}] },
    { name: 'SOUS-MAILLOT SANS MANCHES', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'sousMaillot', price: 40, colors: ["blanc"]},
    { name: 'SOUS-MAILLOT MI-SAISON MANCHES COURTES', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'sousMaillot', price: 45, colors: ["blanc"]},
    { name: 'SOUS-MAILLOT HIVER MANCHES LONGUES', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'sousMaillotHiver', price: 55, colors: ["blanc"]},
    { name: 'SOUS CASQUE', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'unique', price: 18, colors: ["NOIR"]},
    { name: 'CAGOULE', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'unique', price: 20, colors: ["NOIR"]},
    { name: 'GANTS HIVER', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'gants', price: 55, colors: ["NOIR"]},
    { name: 'GANTS ETE CONFORT', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'gants', price: 30, colors: ["NOIR", "BLANC", "MARINE", "BRETON PUR BEURRE"]},
    { name: 'GANTS ETE SLIM', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'gants', price: 40, colors: ["NOIR"]},
    { name: 'GANTS MI-SAISON', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'gantsMiSaison', price: 30, colors: ["NOIR"]},
    { name: 'COUVRE-CHAUSSURES AÉRO', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'couvreChaussuresAero', price: 40, colors: ["NOIR"]},
    { name: 'COUVRE-CHAUSSURES HIVER/PLUIE', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'couvreChaussuresHiver', price: 65, colors: ["NOIR"]},
    { name: 'BANDEAU', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'bandeau', price: 12, colors: ["ARDENT", "FLUO", "HYPNOTIC", "BRETON PUR BEURRE"]},
    { name: 'TOUR DE COU', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'tourDeCou', price: 15, colors: ["ARDENT", "FLUO", "HYPNOTIC", "BRETON PUR BEURRE"]},
    { name: 'GAPETTE', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'unique', price: 20, colors: ["ARDENT", "MAGICREME", "HYPNOTIC", "NOIR"]},
    { name: 'MANCHETTES', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'manchettes', price: 33, colors: ["NOIR"]},
    { name: 'GENOUILLERES', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'manchettes', price: 33, colors: ["NOIR"]},
    { name: 'JAMBIERES', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'jambieres', price: 40, colors: ["NOIR"]},
    { name: 'CHAUSSETTES AÉRO', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'chaussettes', price: 30, colors: ["NOIR", "BLANC", "ARDENT", "HYPNOTIC"]},
    { name: 'CHAUSSETTES MI-HAUTES', category: 'ACCESSOIRES', type: 'accessoire', subtype: 'ACCESSOIRES PERMANENTS', sizeType: 'chaussettes', price: 17, colors: ["NOIR", "BLANC", "BEIGE", "BRETON PUR BEURRE"]},
    { name: 'MAILLOT ENFANT PERFORMANCE MC', category: 'ENFANTS', type: 'enfant', subtype: 'Cyclisme Enfant', hasOptions: false, pricingTiers: [{ quantity: 1, price: 42.00 }] },
       { name: 'MAILLOT BMX ENFANT CONFORT ML', category: 'ENFANTS', type: 'enfant', subtype: 'Cyclisme Enfant', hasOptions: false, pricingTiers: [{ quantity: 1, price: 45.60 }] },
    { name: 'MAILLOT MI-SAISON ENFANT CONFORT ML', category: 'ENFANTS', type: 'enfant', subtype: 'Cyclisme Enfant', hasOptions: false, pricingTiers: [{ quantity: 1, price: 45.60 }] },
    { name: 'GILET COUPE-VENT LEGER ENFANT', category: 'ENFANTS', type: 'enfant', subtype: 'Cyclisme Enfant', hasOptions: false, pricingTiers: [{ quantity: 1, price: 38.40 }] },
    { name: 'VESTE HIVER ENFANT CONFORT', category: 'ENFANTS', type: 'enfant', subtype: 'Cyclisme Enfant', hasOptions: false, pricingTiers: [{ quantity: 1, price: 84.00 }] },
    { name: 'CUISSARD ENFANT CONFORT', category: 'ENFANTS', type: 'enfant', subtype: 'Cyclisme Enfant', hasOptions: false, pricingTiers: [{ quantity: 1, price: 42.00 }] },
    { name: 'COLLANT HIVER ENFANT SUBLIME CONFORT', category: 'ENFANTS', type: 'enfant', subtype: 'Cyclisme Enfant', hasOptions: false, pricingTiers: [{ quantity: 1, price: 60.00 }] },
    { name: 'COLLANT ENFANT VELOURS ECHAUFFEMENT', category: 'ENFANTS', type: 'enfant', subtype: 'Cyclisme Enfant', hasOptions: false, pricingTiers: [{ quantity: 1, price: 49.80 }] },
    { name: 'MAILLOT RUNNING ENFANT MANCHES COURTES', category: 'ENFANTS', type: 'enfant', subtype: 'Running Enfants', hasOptions: false, pricingTiers: [{ quantity: 1, price: 30.00 }] },
    { name: 'DEBARDEUR ATHLETISME ENFANT', category: 'ENFANTS', type: 'enfant', subtype: 'Running Enfants', hasOptions: false, pricingTiers: [{ quantity: 1, price: 27.00 }] },
    { name: 'CUISSARD RUNNING ENFANT', category: 'ENFANTS', type: 'enfant', subtype: 'Running Enfants', hasOptions: false, pricingTiers: [{ quantity: 1, price: 36.00 }] },
   { name: 'TRIFONCTION ENFANT COURTE ET MOYENNE DISTANCE', category: 'ENFANTS', type: 'enfant', subtype: 'Running Enfants', hasOptions: false, pricingTiers: [ { quantity: 1, price: 90.00 }, { quantity: 2, price: 90.00 }, { quantity: 3, price: 90.00 }, { quantity: 5, price: 90.00 }, { quantity: 15, price: 85.50 }, { quantity: 25, price: 81.23 }, { quantity: 50, price: 77.16 }, { quantity: 80, price: 73.31 }, { quantity: 150, price: 69.64 } ] },
    { name: 'POCHE DOS ZIPPEE', category: 'option', type: 'option', pricingTiers: [ { quantity: 1, price: 11.72 }, { quantity: 2, price: 9.38 }, { quantity: 3, price: 7.50 }, { quantity: 5, price: 6.00 }, { quantity: 15, price: 5.70 }, { quantity: 25, price: 5.52 }, { quantity: 50, price: 5.40 }, { quantity: 80, price: 5.22 }, { quantity: 150, price: 5.10 } ] },
    { name: 'BANDE REFLECTIVE', category: 'option', type: 'option', pricingTiers: [ { quantity: 1, price: 7.08 }, { quantity: 2, price: 5.40 }, { quantity: 3, price: 4.50 }, { quantity: 5, price: 3.60 }, { quantity: 15, price: 3.42 }, { quantity: 25, price: 3.31 }, { quantity: 50, price: 3.24 }, { quantity: 80, price: 3.13 }, { quantity: 150, price: 3.06 }, ] },
    { name: 'Ajustement Longueur +3cm', category: 'option', type: 'option', optionGroup: 'length', fixedPriceTTC: 7.20 },
    { name: 'Ajustement Longueur +6cm', category: 'option', type: 'option', optionGroup: 'length', fixedPriceTTC: 7.20 },
    { name: 'Ajustement Longueur +9cm', category: 'option', type: 'option', optionGroup: 'length', fixedPriceTTC: 7.20 },
    { name: 'Ajustement Longueur -3cm', category: 'option', type: 'option', optionGroup: 'length', fixedPriceTTC: 7.20 },
    { name: 'Ajustement Longueur -6cm', category: 'option', type: 'option', optionGroup: 'length', fixedPriceTTC: 7.20 },
    { name: 'Ajustement Longueur -9cm', category: 'option', type: 'option', optionGroup: 'length', fixedPriceTTC: 7.20 },
];

const productSizes = {
    haut: ['XXS', 'XS', 'S', 'M', 'L', 'XL', 'XXL', '3XL', '4XL', '5XL', '6XL'],
    enfant: ['6A', '8A', '10A', '12A', '14A', '16A'],
    aero: ['XXS', 'XS', 'S', 'M', 'L', 'XL', 'XXL', '3XL'],
    ample: ['XXS', 'XS', 'S', 'M', 'L', 'XL', 'XXL', '3XL', '4XL', '5XL', '6XL'],
    largeHaut: ['XXS', 'XS', 'S', 'M', 'L', 'XL', 'XXL', '3XL', '4XL', '5XL', '6XL'],
    manchettes: ["P (Biceps 27/31cm)", "G (Biceps 31/34cm)"],
    jambieres: ["P (Cuisses 39/44cm)", "G (Cuisses 44/50cm)"],
    unique: ["U"],
    gants: ["XXS", "XS", "S", "M", "L", "XL", "XXL"],
    chaussettes: ["S/M (35/40)", "L/XL (41/46)"],
    sousMaillot: ["2XS/XS", "S/M", "L/XL", "2XL/3XL"],
    sousMaillotHiver: ["S", "M", "L", "XL"],
    gantsMiSaison: ["S", "M", "L", "XL"],
    couvreChaussuresAero: ["36/38", "39/41", "42/44", "45/47"],
    couvreChaussuresHiver: ["38/39", "40/42", "43/44", "45/46", "47/48"],
    bandeau: ["XXS", "XS", "S", "M", "L", "XL", "XXL"],
    tourDeCou: ["XXS", "XS", "S", "M", "L", "XL", "XXL"],
};

const TVA_RATE = 0.20;
const DOWN_PAYMENT_RATE = 0.30;
const GRAPHIC_FEE_TTC = 150;
const ADMIN_PASSWORD = "582069Whim#";

// --- PERFORMANCE: Create a Map for quick product lookups ---
const productMap = new Map(allAvailableProducts.map(p => [p.name, p]));

// =================================================================================
// --- APPLICATION STATE ---
// =================================================================================
let state = {
    documentType: 'commande',
    isReassort: false,
    isCustomCreation: false,
    isStoreOrder: false,
    applyDiscount: false,
    clubName: '',
    departement: '',
    clientNumber: '',
    orderDate: new Date().toISOString().split('T')[0],
lastDeliveryDate: '',
    licencieName: '',
    activeLicensee: '',
    licenseeList: [],
    licenseeDeposits: {},
    clubDiscount: 0,
    currentOrderLineItems: [],
    discountType: 'global',
    orderScope: '',
    orderSpecificity: '',
    portalProductSelection: [],
    portalSessionName: '',
    portalDeadline: '',
    portalInfoShown: false,
    currentProduct: '',
    currentQuantities: {},
    currentCalculatedUnitPrice: 0,
    manualUnitPrice: '',
    currentSelectedOptions: [],
    currentSpecificity: '',
    currentAccessoryQuantity: '',
    currentColor: '',
    clubProductRange: [], 
    showAllProducts: false, 
    clubStock: {},
    isAddingForStock: false,
 clubVisuals: [],
    currentVisual: '',
 preOrderNumber: '',
    factoryDepartureDate: '',
    deliveryAddress: '',
    deliveryContact: '',
};

// =================================================================================
// --- DOM ELEMENT REFERENCES ---
// =================================================================================
const dom = {
    mainAppView: document.getElementById('main-app-view'),
    portalView: document.getElementById('portal-view'),
    mainModal: document.getElementById('main-modal'),
    mainModalTitle: document.getElementById('main-modal-title'),
    mainModalBody: document.getElementById('main-modal-body'),
    mainModalActions: document.getElementById('main-modal-actions'),
    mainModalCloseBtn: document.getElementById('main-modal-close-btn'),
    toastContainer: document.getElementById('toast-container'),
    mainTitle: document.getElementById('main-title'),
    historyBtn: document.getElementById('history-btn'),
    summaryTotalHauts: document.getElementById('summary-total-hauts'),
    summaryTotalBas: document.getElementById('summary-total-bas'),
    summaryTotalAccessoires: document.getElementById('summary-total-accessoires'),
    summaryTotalLicensees: document.getElementById('summary-total-licensees'),
    summaryTotalStock: document.getElementById('summary-total-stock'),
    docTypeReassortCheck: document.getElementById('doc-type-reassort'),
    customCreationCheck: document.getElementById('custom-creation-check'),
    orderScopeContainer: document.getElementById('order-scope-container'),
    scopeGlobalRadio: document.getElementById('scope-global'),
    scopeLicenseeRadio: document.getElementById('scope-licensee'),
 scopeSessionRadio: document.getElementById('scope-session'),
    storeOrderCheck: document.getElementById('store-order-check'),
    applyDiscountCheck: document.getElementById('apply-discount-check'),
    clubName: document.getElementById('clubName'),
    manageClientsBtn: document.getElementById('manage-clients-btn'),
    departement: document.getElementById('departement'),
    clientNumber: document.getElementById('clientNumber'),
    orderDate: document.getElementById('orderDate'),
    orderSpecificity: document.getElementById('orderSpecificity'),
    portalButtonsContainer: document.getElementById('portal-buttons-container'),
    portalSessionName: document.getElementById('portalSessionName'),
    portalDeadline: document.getElementById('portalDeadline'),
    selectPortalProductsBtn: document.getElementById('select-portal-products-btn'),
    generatePortalLinkBtn: document.getElementById('generate-portal-link-btn'),
    licencieNameContainer: document.getElementById('licencieName-container'),
    licencieNameInput: document.getElementById('licencieName'),
    licenseeDatalist: document.getElementById('licensee-datalist'),
    activeLicenseeBanner: document.getElementById('active-licensee-banner'),
    bannerLicenseeName: document.getElementById('banner-licensee-name'),
    clearActiveLicenseeBtn: document.getElementById('clear-active-licensee-btn'),
    manageLicenseesBtn: document.getElementById('manage-licensees-btn'),
    importPortalSubmissionsBtn: document.getElementById('import-portal-submissions-btn'),
    nextLicenseeBtn: document.getElementById('next-licensee-btn'),
    productTabsContainer: document.getElementById('product-tabs-container'),
    productFormContainer: document.getElementById('product-form-container'),
    orderTableBody: document.getElementById('order-table-body'),
    orderTableHead: document.getElementById('order-table-head'),
    totalArticlesDisplay: document.getElementById('total-articles-display'),
    discountControlsContainer: document.getElementById('discount-controls-container'),
    discountGlobalRadio: document.getElementById('discount-global'),
    discountItemRadio: document.getElementById('discount-item'),
    clubDiscount: document.getElementById('clubDiscount'),
    subtotalHT: document.getElementById('subtotal-ht'),
    subtotalTTC: document.getElementById('subtotal-ttc'),
    discountAmountHT: document.getElementById('discount-amount-ht'),
    discountAmountTTC: document.getElementById('discount-amount-ttc'),
    shippingHT: document.getElementById('shipping-ht'),
    shippingTTC: document.getElementById('shipping-ttc'),
    graphicFeeContainer: document.getElementById('graphic-fee-container'),
    graphicFeeHT: document.getElementById('graphic-fee-ht'),
    graphicFeeTTC: document.getElementById('graphic-fee-ttc'),
    grandTotalHT: document.getElementById('grand-total-ht'),
    grandTotalTTC: document.getElementById('grand-total-ttc'),
    downPaymentContainer: document.getElementById('down-payment-container'),
    downPayment: document.getElementById('down-payment'),
    newOrderBtn: document.getElementById('new-order-btn'),
    saveOrderBtn: document.getElementById('save-order-btn'),
    loadOrderInput: document.getElementById('load-order-input'),
    importLicenseesInput: document.getElementById('import-licensees-input'),
    importStockInput: document.getElementById('import-stock-input'),
    validateOrderBtn: document.getElementById('validate-order-btn'),
    exportDistributionBtn: document.getElementById('export-distribution-btn'),
    manageStockBtn: document.getElementById('manage-stock-btn'),
initStockBtn: document.getElementById('init-stock-btn'),
manageClubRangeBtn: document.getElementById('manage-club-range-btn'), // NOUVEAU
    toggleProductsViewContainer: document.getElementById('toggle-products-view-container'), // NOUVEAU
    toggleProductsBtn: document.getElementById('toggle-products-btn'), // NOUVEAU
    autosaveStatus: document.getElementById('autosave-status'),
};

// =================================================================================
// --- HELPER & UTILITY FUNCTIONS ---
// =================================================================================

const sanitizeForId = (text) => {
    if (!text) return '';
    return text.replace(/[^a-zA-Z0-9]/g, '_');
};

const scrollToLicensee = (licenseeName) => {
    if (!licenseeName) return;
    const sanitizedName = sanitizeForId(licenseeName);
    const targetElement = document.getElementById(`licencie-header-${sanitizedName}`);
    if (targetElement) {
        targetElement.scrollIntoView({ behavior: 'smooth', block: 'center' });
    }
};

let clientDatabase = [];

const saveClientInfo = () => {
    const clubName = dom.clubName.value.trim();
    const clientNumber = dom.clientNumber.value.trim();
    const departement = dom.departement.value.trim();

    if (clubName) {
        let clientFound = false;
        clientDatabase.forEach(client => {
            if (client.clubName === clubName) {
                client.clientNumber = clientNumber;
                client.departement = departement;
                clientFound = true;
            }
        });

        if (!clientFound) {
            clientDatabase.push({ clubName, clientNumber, departement });
        }

        localStorage.setItem('clientDatabase', JSON.stringify(clientDatabase));
        updateDatalists();
    }
};
const updateSectionHighlights = () => {
    const allSections = document.querySelectorAll('#info-section, #add-article-section, #summary-and-actions-section, #licencieName-container, #product-selection-step, #size-selection-step, #add-button-step, #order-scope-container');

    allSections.forEach(el => el.classList.remove('highlight-section'));

    // Logique de surbrillance mise à jour
    if (!state.clubName.trim() || !state.departement.trim()) {
        document.getElementById('info-section')?.classList.add('highlight-section');
    } else if (!state.orderScope) { // NOUVELLE CONDITION
        document.getElementById('order-scope-container')?.classList.add('highlight-section');
    } else if (state.orderScope === 'licensee' && !state.activeLicensee) {
        document.getElementById('licencieName-container')?.classList.add('highlight-section');
    } else if (!state.currentProduct) {
        document.getElementById('product-selection-step')?.classList.add('highlight-section');
    } else if (Object.values(state.currentQuantities).reduce((s, q) => s + (parseInt(q, 10) || 0), 0) === 0 && !state.currentAccessoryQuantity) {
        document.getElementById('size-selection-step')?.classList.add('highlight-section');
    } else if (state.currentProduct) {
        document.getElementById('add-button-step')?.classList.add('highlight-section');
    } else {
        document.getElementById('summary-and-actions-section')?.classList.add('highlight-section');
    }
};const updateDatalists = () => {
    const clubList = document.getElementById('club-list');
    const clientList = document.getElementById('client-list');
    if(clubList) clubList.innerHTML = clientDatabase.map(c => `<option value="${c.clubName}"></option>`).join('');
    if(clientList) clientList.innerHTML = clientDatabase.map(c => `<option value="${c.clientNumber}"></option>`).join('');
};

const showToast = (message, type = 'success') => {
    const toast = document.createElement('div');
    const bgColor = type === 'success' ? 'bg-green-500' : 'bg-red-500';
    toast.className = `toast ${bgColor} text-white p-4 rounded-lg shadow-lg mb-2`;
    toast.textContent = message;
    dom.toastContainer.appendChild(toast);
    
    setTimeout(() => toast.classList.add('show'), 10);

    setTimeout(() => {
        toast.classList.remove('show');
        toast.addEventListener('transitionend', () => toast.remove());
    }, 3000);
};

const showModal = (modalElement, title, content, actions = [], maxWidth = 'max-w-md', onOpen = null) => {
    const modalDialog = modalElement.querySelector('div');
    modalDialog.classList.remove('max-w-md', 'max-w-lg', 'max-w-xl', 'max-w-2xl', 'max-w-4xl');
    modalDialog.classList.add(maxWidth);
    dom.mainModalTitle.textContent = title;
    dom.mainModalBody.innerHTML = '';
    dom.mainModalBody.appendChild(content);
    dom.mainModalActions.innerHTML = '';
    if (actions.length > 0) {
        actions.forEach(action => {
            const button = document.createElement('button');
            button.textContent = action.label;
            button.className = `${action.className || 'bg-indigo-600 hover:bg-indigo-700 text-white'} font-bold py-2 px-4 rounded-lg`;
            button.onclick = action.onClick;
            dom.mainModalActions.appendChild(button);
        });
    } else {
        const closeButton = document.createElement('button');
        closeButton.textContent = 'Fermer';
        closeButton.className = 'bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded-lg';
        closeButton.onclick = () => hideModal(modalElement);
        dom.mainModalActions.appendChild(closeButton);
    }
    modalElement.classList.add('is-open');

    // On exécute une fonction si elle est fournie après l'ouverture
    if (onOpen && typeof onOpen === 'function') {
        onOpen();
    }
};

const hideModal = (modalElement) => {
    modalElement.classList.remove('is-open');
};

const getSortedSizesText = (item) => {
    if (item.quantitiesPerSize['Devis']) {
        return `Quantité globale`;
    }
    const product = productMap.get(item.productName);
    const defaultText = Object.entries(item.quantitiesPerSize).map(([s, q]) => `${s}: ${q}`).join(', ');
    if (!product) return defaultText;
    const sizeOrder = productSizes[product.sizeType || product.type] || [];
    if (sizeOrder.length === 0) return defaultText;
    const sortedQuantities = Object.entries(item.quantitiesPerSize)
        .filter(([, qty]) => (parseInt(qty, 10) || 0) > 0)
        .sort(([sizeA], [sizeB]) => {
            const indexA = sizeOrder.indexOf(sizeA);
            const indexB = sizeOrder.indexOf(sizeB);
            if (indexA === -1 && indexB === -1) return sizeA.localeCompare(sizeB);
            if (indexA === -1) return 1;
            if (indexB === -1) return -1;
            return indexA - indexB;
        });
    return sortedQuantities.map(([size, qty]) => `${size}: ${qty}`).join(', ');
};

// =================================================================================
// --- CALCULATION LOGIC ---
// =================================================================================

const getPriceForQuantity = (pricingTiers, totalQuantity) => {
    if (!pricingTiers || pricingTiers.length === 0) return 0;
    let applicableTier = pricingTiers[0];
    for (let i = pricingTiers.length - 1; i >= 0; i--) {
        if (totalQuantity >= pricingTiers[i].quantity) {
            applicableTier = pricingTiers[i];
            break;
        }
    }
    return applicableTier.price;
};

const getUnitPriceTTC = (productName, totalPricingQuantity, selectedOptions) => {
    const product = productMap.get(productName);
    if (!product) return 0;
    const basePrice = product.price ? product.price : getPriceForQuantity(product.pricingTiers, totalPricingQuantity);
    const optionsPrice = selectedOptions.reduce((total, optionName) => {
        const optionProduct = productMap.get(optionName);
        if (!optionProduct) return total;
        if (optionProduct.fixedPriceTTC) return total + optionProduct.fixedPriceTTC;
        if (optionProduct.pricingTiers) return total + getPriceForQuantity(optionProduct.pricingTiers, totalPricingQuantity);
        return total;
    }, 0);
    return basePrice + optionsPrice;
};

// --- CORRIGÉ ---
const calculateAllTotals = () => {
    state.currentOrderLineItems.sort((a, b) => {
        const indexA = allAvailableProducts.findIndex(p => p.name === a.productName);
        const indexB = allAvailableProducts.findIndex(p => p.name === b.productName);
        return indexA - indexB;
    });

    // ▼▼▼ NOUVELLE LOGIQUE DE CALCUL ▼▼▼

    // 1. On calcule le total spécifique et forcé pour toutes les vestes d'hiver.
    const winterJacketNames = [
        'VESTE HIVER HOMME CONFORT', 
        'VESTE HIVER FEMME CONFORT', 
        'VESTE HIVER THERMIQUE HOMME CONFORT', 
        'VESTE HIVER THERMIQUE FEMME CONFORT'
    ];
    const totalWinterJacketQty = state.currentOrderLineItems
        .filter(item => winterJacketNames.includes(item.productName))
        .reduce((sum, item) => sum + item.totalQuantity, 0);

    // 2. On calcule les totaux pour les AUTRES groupes de prix.
    const groupTotals = state.currentOrderLineItems.reduce((acc, item) => {
        const product = productMap.get(item.productName);
        if (product && product.pricingGroup) {
            acc[product.pricingGroup] = (acc[product.pricingGroup] || 0) + item.totalQuantity;
        }
        return acc;
    }, {});


    const updatedLineItems = state.currentOrderLineItems.map(item => {
        const product = productMap.get(item.productName);
        
        if (!product) {
            console.error(`Produit non trouvé : ${item.productName}. Cet article aura un prix de 0.`);
            return { ...item, unitPriceTTC: 0, unitPriceHT: 0, totalPriceTTC: 0, totalPriceHT: 0 };
        }

        let finalUnitPriceTTC;
        if (item.isManualPrice) {
            finalUnitPriceTTC = item.initialManualPrice;
        } else {
            let pricingQuantity;
            
            // 3. On applique la bonne quantité de groupe à chaque article.
            if (winterJacketNames.includes(product.name)) {
                // Si c'est une veste d'hiver, on utilise notre total forcé.
                pricingQuantity = totalWinterJacketQty;
            } else if (product.pricingGroup) {
                // Sinon, on utilise la logique de groupe normale.
                pricingQuantity = groupTotals[product.pricingGroup];
            } else {
                // Sinon, on utilise la quantité de l'article seul.
                pricingQuantity = item.totalQuantity;
            }
            finalUnitPriceTTC = getUnitPriceTTC(item.productName, pricingQuantity, item.options);
        }

        if (item.isFromStock && item.licencieName === 'Commande Globale') {
            finalUnitPriceTTC = 0;
        }

        const totalPriceTTC = finalUnitPriceTTC * item.totalQuantity;
        const finalUnitPriceHT = finalUnitPriceTTC / (1 + TVA_RATE);
        const totalPriceHT = totalPriceTTC / (1 + TVA_RATE);

        return { ...item, unitPriceTTC: finalUnitPriceTTC, unitPriceHT: finalUnitPriceHT, totalPriceTTC, totalPriceHT };
    });
    
    // ▲▲▲ FIN DE LA NOUVELLE LOGIQUE ▲▲▲

    state.currentOrderLineItems = updatedLineItems;

    const originalSubtotalHT = updatedLineItems.reduce((acc, item) => acc + item.totalPriceHT, 0);
    const originalSubtotalTTC = updatedLineItems.reduce((acc, item) => acc + item.totalPriceTTC, 0);

    let discountBaseHT = 0;
    if (state.applyDiscount && state.discountType === 'global') {
        discountBaseHT = originalSubtotalHT;
    } else if (state.applyDiscount && state.discountType === 'item') {
        discountBaseHT = updatedLineItems.filter(item => item.applyDiscount).reduce((acc, item) => acc + item.totalPriceHT, 0);
    }
    const discountAmountHT = discountBaseHT * (state.clubDiscount / 100);
    const discountAmountTTC = discountAmountHT * (1 + TVA_RATE);

    let shippingHT = 0;
    if (originalSubtotalHT > 2000) shippingHT = 0;
    else if (originalSubtotalHT > 1000) shippingHT = 14;
    else if (originalSubtotalHT > 500) shippingHT = 12;
    else if (originalSubtotalHT > 0) shippingHT = 9.50;
    
    const totalNonAccessoryQty = state.currentOrderLineItems.reduce((sum, item) => {
        const product = productMap.get(item.productName);
        if (product && product.type !== 'accessoire') {
            return sum + item.totalQuantity;
        }
        return sum;
    }, 0);

    const applyGraphicFee = state.isCustomCreation && totalNonAccessoryQty < 20;
    const graphicFeeTTC = applyGraphicFee ? GRAPHIC_FEE_TTC : 0;
    const graphicFeeHT = graphicFeeTTC / (1 + TVA_RATE);

    const shippingTTC = shippingHT * (1 + TVA_RATE);
    const grandTotalHT = originalSubtotalHT + shippingHT + graphicFeeHT;
    const grandTotalTTC = originalSubtotalTTC + shippingTTC + graphicFeeHT;
    const downPayment = grandTotalTTC * DOWN_PAYMENT_RATE;
    
    return { subtotalHT: originalSubtotalHT, subtotalTTC: originalSubtotalTTC, discountAmountHT, discountAmountTTC, shippingHT, shippingTTC, graphicFeeHT, graphicFeeTTC, grandTotalHT, grandTotalTTC, downPayment };
};
// =================================================================================
// --- UI RENDER FUNCTIONS ---
// =================================================================================

const renderAll = () => {
    renderUIState();
    renderTotals();
    renderDashboard();
    renderQuantityDashboard();
    renderFloatingCart(); // <-- Ajoutez cette ligne
    renderOrderTableHead();
    renderOrderTable();
    renderProductForm();
    renderLicenseeDatalist();
    updateButtonStates();
    updateSectionHighlights();
};
const renderUIState = () => {
    dom.clubName.value = state.clubName;
    dom.departement.value = state.departement;
    dom.clientNumber.value = state.clientNumber;
    dom.orderDate.value = state.orderDate;
    dom.orderSpecificity.value = state.orderSpecificity;
    document.getElementById('preOrderNumber').value = state.preOrderNumber;
    document.getElementById('factoryDepartureDate').value = state.factoryDepartureDate;
    document.getElementById('deliveryAddress').value = state.deliveryAddress;
    document.getElementById('deliveryContact').value = state.deliveryContact;
    
    dom.mainTitle.textContent = 'Bon de commande';
    dom.downPaymentContainer.style.display = 'block';
    dom.docTypeReassortCheck.checked = state.isReassort;
    dom.customCreationCheck.checked = state.isCustomCreation;
    dom.storeOrderCheck.checked = state.isStoreOrder;
    dom.applyDiscountCheck.checked = state.applyDiscount;
    
    // Logique de sélection des boutons radio
    dom.scopeGlobalRadio.checked = state.orderScope === 'global';
    dom.scopeLicenseeRadio.checked = state.orderScope === 'licensee';
    dom.scopeSessionRadio.checked = state.orderScope === 'session';
    
    dom.discountGlobalRadio.checked = state.discountType === 'global';
    dom.discountItemRadio.checked = state.discountType === 'item';
    
    // ▼▼▼ LOGIQUE D'AFFICHAGE CONDITIONNEL MISE À JOUR ▼▼▼
    dom.licencieNameContainer.classList.toggle('hidden', state.orderScope !== 'licensee');
    dom.portalButtonsContainer.classList.toggle('hidden', state.orderScope !== 'session');
    // ▲▲▲ FIN DE LA MISE À JOUR ▲▲▲

    dom.discountControlsContainer.classList.toggle('hidden', !state.applyDiscount);
    dom.exportDistributionBtn.style.display = state.orderScope === 'licensee' ? 'inline-flex' : 'none';

    if (state.activeLicensee) {
        dom.licencieNameInput.value = ''; // On vide le champ de saisie quand un licencié est actif
        dom.bannerLicenseeName.textContent = state.activeLicensee;
        dom.activeLicenseeBanner.classList.remove('hidden');
    } else {
        dom.activeLicenseeBanner.classList.add('hidden');
    }

    dom.validateOrderBtn.textContent = 'Valider la Commande';
    dom.validateOrderBtn.classList.remove('bg-blue-600', 'hover:bg-blue-700');
    dom.validateOrderBtn.classList.add('bg-indigo-600', 'hover:bg-indigo-700');

    const articleSectionBlocker = document.getElementById('article-section-blocker');
    if (!state.clubName.trim() || !state.departement.trim() || !state.orderScope) {
        dom.productTabsContainer.classList.add('hidden');
        dom.productFormContainer.classList.add('hidden');
        articleSectionBlocker.classList.remove('hidden');
    } else {
        dom.productTabsContainer.classList.remove('hidden');
        dom.productFormContainer.classList.remove('hidden');
        articleSectionBlocker.classList.add('hidden');
    }
};

const renderDashboard = () => {
    let totalHauts = 0;
    let totalBas = 0;
    let totalAccessoires = 0;
    const licenseeSet = new Set();

    state.currentOrderLineItems.forEach(item => {
        const product = productMap.get(item.productName);
        if (!product) return;

        if (product.type === 'accessoire') {
            totalAccessoires += item.totalQuantity;
        } else if (product.isCuissardOrCollant || product.subtype === 'Bas' || product.subtype.includes('Cuissard') || product.subtype.includes('Corsaire') || product.subtype.includes('Collant')) {
            totalBas += item.totalQuantity;
        } else {
            totalHauts += item.totalQuantity;
        }

        if (item.licencieName && item.licencieName !== 'Commande Globale') {
            licenseeSet.add(item.licencieName);
        }
    });

    const totalStock = Object.values(state.clubStock).reduce((sum, product) => 
        sum + Object.values(product).reduce((prodSum, qty) => prodSum + qty, 0), 0);

    dom.summaryTotalHauts.textContent = totalHauts;
    dom.summaryTotalBas.textContent = totalBas;
    dom.summaryTotalAccessoires.textContent = totalAccessoires;
    dom.summaryTotalLicensees.textContent = licenseeSet.size;
    dom.summaryTotalStock.textContent = totalStock;
};
const renderQuantityDashboard = () => {
    const container = document.getElementById('quantity-dashboard-container');
    if (state.currentOrderLineItems.length === 0) {
        container.innerHTML = '<p class="text-center text-gray-500 text-sm">Le tableau de bord des quantités apparaîtra ici.</p>';
        return;
    }

    const quantityBySubtype = state.currentOrderLineItems.reduce((acc, item) => {
        const product = productMap.get(item.productName);
        if (product && product.subtype) {
            if (product.subtype === 'ACCESSOIRES PERMANENTS') return acc;
            
            acc[product.subtype] = (acc[product.subtype] || 0) + item.totalQuantity;
        }
        return acc;
    }, {});

    const sortedSubtypes = Object.keys(quantityBySubtype).sort();

    if (sortedSubtypes.length === 0) {
        container.innerHTML = '<p class="text-center text-gray-500 text-sm">Le tableau de bord des quantités apparaîtra ici.</p>';
        return;
    }

    let contentHtml = '<h3 class="text-lg font-semibold text-gray-700 mb-3 text-center">Quantités par Type de Produit (Cliquable)</h3>';
    contentHtml += '<div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-4 text-center">';

    sortedSubtypes.forEach(subtype => {
        const quantity = quantityBySubtype[subtype];
        // On ajoute la classe "quantity-card" et l'attribut "data-subtype" pour le rendre cliquable
        contentHtml += `
            <div class="bg-white p-2 rounded-md shadow cursor-pointer hover:bg-indigo-50 hover:shadow-lg transition-all quantity-card" data-subtype="${subtype}">
                <p class="text-sm font-medium text-gray-600 pointer-events-none">${subtype}</p>
                <p class="text-2xl font-bold text-indigo-600 pointer-events-none">${quantity}</p>
            </div>
        `;
    });

    contentHtml += '</div>';
    container.innerHTML = contentHtml;
};

const renderLicenseeDatalist = () => {
    dom.licenseeDatalist.innerHTML = state.licenseeList
        .sort((a, b) => a.localeCompare(b))
        .map(name => `<option value="${name}"></option>`)
        .join('');
};
const renderFloatingCart = () => {
    const cart = document.getElementById('floating-cart');
    if (state.currentOrderLineItems.length === 0) {
        cart.classList.add('hidden');
        return;
    }

    cart.classList.remove('hidden');

    const totals = calculateAllTotals();
    const totalArticles = state.currentOrderLineItems.reduce((acc, item) => acc + item.totalQuantity, 0);

    document.getElementById('floating-cart-total-articles').textContent = totalArticles;
    document.getElementById('floating-cart-grand-total').textContent = `${totals.grandTotalTTC.toFixed(2)}€`;

    const summaryContainer = document.getElementById('floating-cart-summary');
    const quantityBySubtype = state.currentOrderLineItems.reduce((acc, item) => {
        const product = productMap.get(item.productName);
        if (product && product.subtype && product.type !== 'accessoire') {
            acc[product.subtype] = (acc[product.subtype] || 0) + item.totalQuantity;
        }
        return acc;
    }, {});

    const sortedSubtypes = Object.keys(quantityBySubtype).sort();
    
    if (sortedSubtypes.length > 0) {
        summaryContainer.innerHTML = sortedSubtypes.map(subtype => `
            <div class="flex justify-between">
                <span class="text-gray-500">${subtype}:</span>
                <span class="font-semibold text-gray-700">${quantityBySubtype[subtype]}</span>
            </div>
        `).join('');
    } else {
        summaryContainer.innerHTML = '<p class="text-center text-gray-400">-</p>';
    }
};
const renderOrderTableHead = () => {
    let headers = '';
    if (state.applyDiscount && state.discountType === 'item') {
        headers += `<th class="px-2 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Remise</th>`;
    }
    headers += `
        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Produit</th>
        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Qté</th>
        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Prix U. HT</th>
        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Prix U. TTC</th>
        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Total HT</th>
        <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Total TTC</th>
        <th class="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">Actions</th>
    `;
    dom.orderTableHead.innerHTML = `<tr>${headers}</tr>`;
};

const renderOrderTable = () => {
    if (state.currentOrderLineItems.length === 0) {
        dom.orderTableBody.innerHTML = `<tr><td colspan="9" class="px-6 py-12 text-center text-gray-500">Aucun article dans la commande.</td></tr>`;
        return;
    }

    let tableHtml = '';
    if (state.orderScope === 'licensee') {
        const groupedItems = state.currentOrderLineItems.reduce((acc, item) => {
            const key = item.licencieName || 'Commande Globale';
            if (!acc[key]) acc[key] = [];
            acc[key].push(item);
            return acc;
        }, {});

        const sortedLicensees = Object.keys(groupedItems).sort((a, b) => a.localeCompare(b));

        sortedLicensees.forEach(licensee => {
            let licenseeSubtotal = 0;
            const sanitizedName = sanitizeForId(licensee);
            
            tableHtml += `<tr id="licencie-header-${sanitizedName}" class="bg-indigo-50"><td colspan="8" class="px-6 py-2 text-left text-sm font-bold text-indigo-800 flex justify-between items-center">
                <span>${licensee}</span>
               <div class="flex items-center gap-2">
                    <button data-action="return-to-input" class="text-xs bg-gray-500 text-white px-2 py-1 rounded hover:bg-gray-600">↑ Retour Saisie</button>
                    <button data-action="manage-deposit" data-licensee-name="${licensee}" class="text-xs bg-yellow-500 text-white px-2 py-1 rounded hover:bg-yellow-600">Gérer Acompte</button>
                    <button data-action="add-for-licensee" data-licensee-name="${licensee}" class="text-xs bg-blue-500 text-white px-2 py-1 rounded hover:bg-blue-600">+ Ajouter article</button>
                </div>
            </td></tr>`;
            
            const itemRowsHTML = groupedItems[licensee].map(item => {
                let itemTotal = item.totalPriceTTC;
                if(state.applyDiscount && (state.discountType === 'global' || (state.discountType === 'item' && item.applyDiscount))){
                    itemTotal -= (item.totalPriceTTC * (state.clubDiscount / 100));
                }
                licenseeSubtotal += itemTotal;
                return createSingleItemRowHTML(item);
            }).join('');
            tableHtml += itemRowsHTML;

            const deposit = state.licenseeDeposits[licensee] || 0;
            const remaining = licenseeSubtotal - deposit;
            
            // ▼▼▼ LIGNE MODIFIÉE POUR UN AFFICHAGE FLEXIBLE ▼▼▼
            tableHtml += `<tr class="bg-indigo-100">
                            <td colspan="8" class="px-6 py-2 text-sm text-indigo-900 font-semibold">
                                <div class="flex flex-wrap justify-start items-center gap-x-4 gap-y-1">
                                    <span>Total: <strong class="font-bold">${licenseeSubtotal.toFixed(2)}€</strong></span>
                                    <span>Acompte Versé: <strong class="font-bold">${deposit.toFixed(2)}€</strong></span>
                                    <span class="text-red-600">Restant Dû: <strong class="font-bold">${remaining.toFixed(2)}€</strong></span>
                                </div>
                            </td>
                          </tr>`;
            // ▲▲▲ FIN DE LA MODIFICATION ▲▲▲
        });
    } else {
        tableHtml = state.currentOrderLineItems.map(createSingleItemRowHTML).join('');
    }
    dom.orderTableBody.innerHTML = tableHtml;
};
const createSingleItemRowHTML = (item) => {
    const sizesText = getSortedSizesText(item);
    const optionsText = item.options.length > 0 ? `<div class="text-xs text-blue-500">Options: ${item.options.join(', ')}</div>` : '';
    const specificityText = item.specificity ? `<div class="text-xs text-gray-600 italic mt-1">Note: ${item.specificity}</div>` : '';
    const visualText = item.visual ? `<div class="text-xs font-bold text-cyan-700 bg-cyan-100 px-2 py-0.5 rounded-full inline-block mt-1">Visuel: ${item.visual}</div>` : '';
    
    let rowClass = '';
    if (item.isStockOrder) {
        rowClass = 'bg-yellow-50';
    } else if (item.isFromStock) {
        rowClass = 'bg-teal-50';
    } else if (state.applyDiscount && state.discountType === 'item' && item.applyDiscount) {
        rowClass = 'bg-green-50';
    }

    let rowHtml = `<tr class="${rowClass} hover:bg-gray-50">`;

    if (state.applyDiscount && state.discountType === 'item') {
        rowHtml += `<td class="px-2 py-4 align-top"><input type="checkbox" class="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500" data-item-id="${item.id}" data-action="toggle-discount" ${item.applyDiscount ? 'checked' : ''}></td>`;
    }
    
    let totalTTCDisplay = `${item.totalPriceTTC.toFixed(2)}€`;
    if (state.applyDiscount && ((state.discountType === 'global') || (state.discountType === 'item' && item.applyDiscount))) {
        const discountedPrice = item.totalPriceTTC * (1 - (state.clubDiscount / 100));
        totalTTCDisplay = `<div class="flex flex-col items-end"><span class="line-through text-gray-500 text-xs">${item.totalPriceTTC.toFixed(2)}€</span><span class="text-red-600 font-bold">${discountedPrice.toFixed(2)}€</span></div>`;
    }
    
    const stockOrderLabel = item.isStockOrder ? `<div class="text-xs font-bold text-yellow-800 bg-yellow-200 px-2 py-0.5 rounded-full inline-block mb-1">📦 POUR LE STOCK</div>` : '';
    const fromStockLabel = item.isFromStock ? `<div class="text-xs font-bold text-teal-800 bg-teal-200 px-2 py-0.5 rounded-full inline-block mb-1">✔️ LIVRÉ DU STOCK</div>` : '';

    // ▼▼▼ MODIFICATIONS CI-DESSOUS ▼▼▼
    rowHtml += `
        <td class="px-3 py-4 align-top">
            <div class="text-sm font-medium text-gray-900">${stockOrderLabel}${fromStockLabel}${item.productName}</div>
            <div class="text-xs text-gray-500">${sizesText}</div>
            <div class="flex flex-wrap gap-2 items-center mt-1">${visualText}${optionsText}${specificityText}</div>
        </td>
        <td class="px-3 py-4 whitespace-nowrap text-sm text-gray-500 text-center align-top">${item.totalQuantity}</td>
        <td class="px-3 py-4 whitespace-nowrap text-sm text-gray-500 align-top">${item.unitPriceHT.toFixed(2)}€</td>
        <td class="px-3 py-4 whitespace-nowrap text-sm text-gray-500 align-top">${item.unitPriceTTC.toFixed(2)}€</td>
        <td class="px-3 py-4 whitespace-nowrap text-sm font-bold text-gray-900 align-top">${item.totalPriceHT.toFixed(2)}€</td>
        <td class="px-3 py-4 text-sm font-bold text-gray-900 align-top">${totalTTCDisplay}</td>
        <td class="px-3 py-4 text-right text-sm font-medium align-top">
            <div class="flex flex-col sm:flex-row justify-end gap-2">
                <button data-action="edit-item" data-item-id="${item.id}" class="text-blue-600 hover:text-blue-900">Modifier</button>
                <button data-action="remove-item" data-item-id="${item.id}" class="text-red-600 hover:text-red-900">Supprimer</button>
            </div>
        </td>
    </tr>`;
    // ▲▲▲ FIN DES MODIFICATIONS ▲▲▲
    return rowHtml;
};
const renderTotals = () => {
    const totals = calculateAllTotals();
    dom.subtotalHT.textContent = `${totals.subtotalHT.toFixed(2)}€`;
    dom.subtotalTTC.textContent = `${totals.subtotalTTC.toFixed(2)}€`;
    dom.discountAmountHT.textContent = `-${totals.discountAmountHT.toFixed(2)}€`;
    dom.discountAmountTTC.textContent = `-${totals.discountAmountTTC.toFixed(2)}€`;
    dom.shippingHT.textContent = `${totals.shippingHT.toFixed(2)}€`;
    dom.shippingTTC.textContent = `${totals.shippingTTC.toFixed(2)}€`;

    dom.graphicFeeContainer.classList.toggle('hidden', !state.isCustomCreation);
    dom.graphicFeeHT.textContent = `${totals.graphicFeeHT.toFixed(2)}€`;
    dom.graphicFeeTTC.textContent = `${totals.graphicFeeTTC.toFixed(2)}€`;

    dom.grandTotalHT.textContent = `${totals.grandTotalHT.toFixed(2)}€`;
    dom.grandTotalTTC.textContent = `${totals.grandTotalTTC.toFixed(2)}€`;
    dom.downPayment.textContent = `${totals.downPayment.toFixed(2)}€`;
    dom.totalArticlesDisplay.textContent = `Total des articles : ${state.currentOrderLineItems.reduce((acc, item) => acc + item.totalQuantity, 0)}`;
};

const renderProductForm = () => {
    if (state.clubProductRange.length > 0) {
        dom.toggleProductsViewContainer.classList.remove('hidden');
        dom.toggleProductsBtn.textContent = state.showAllProducts ? 'Afficher seulement la gamme' : 'Afficher tous les articles';
    } else {
        dom.toggleProductsViewContainer.classList.add('hidden');
    }

    let sourceProducts = allAvailableProducts;
    if (state.clubProductRange.length > 0 && !state.showAllProducts) {
        sourceProducts = allAvailableProducts.filter(p => state.clubProductRange.includes(p.name));
    }

    const activeProductTab = document.querySelector('.product-tab-btn.border-indigo-500')?.dataset.tab;
    const productsToShow = sourceProducts.filter(p => {
        if (p.category === 'option') return false;
        let categoryMatch = false;
        if (activeProductTab === 'CYCLISME/RUNNING') categoryMatch = p.category === 'CYCLISME' || p.category === 'RUNNING';
        else if (activeProductTab === 'Accessoires') categoryMatch = p.category === 'ACCESSOIRES';
        else if (activeProductTab === 'GAMME ENFANTS') categoryMatch = p.category === 'ENFANTS';
        return categoryMatch;
    });
    const grouped = productsToShow.reduce((acc, p) => {
        const groupKey = `${p.category} - ${p.subtype}`;
        acc[groupKey] = [...(acc[groupKey] || []), p];
        return acc;
    }, {});
    const productSelectorOptions = Object.entries(grouped).map(([groupLabel, productList]) => 
        `<optgroup label="${groupLabel}">${productList.map(p => `<option value="${p.name}" ${state.currentProduct === p.name ? 'selected' : ''}>${p.name}</option>`).join('')}</optgroup>`
    ).join('');
    
    let formHtml = `
        <div id="product-selection-step" class="p-4 rounded-xl grid grid-cols-1 md:grid-cols-2 gap-4">
            <div>
                <label class="block text-sm font-medium text-gray-700">Produit</label>
                <select id="current-product-select" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
                    <option value="">-- Sélectionner un produit --</option>
                    ${productSelectorOptions}
                </select>
            </div>
            <div>
                <label class="block text-sm font-medium text-gray-700">Visuel (Optionnel)</label>
                <select id="current-visual-select" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm ${state.clubVisuals.length === 0 ? 'hidden' : ''}">
                    <option value="">Standard / Non spécifié</option>
                    ${state.clubVisuals.map(v => `<option value="${v}" ${state.currentVisual === v ? 'selected' : ''}>${v}</option>`).join('')}
                </select>
                ${state.clubVisuals.length === 0 ? `<p class="text-xs text-gray-500 mt-1">Aucun visuel défini. Utilisez le bouton "Gérer les Visuels".</p>` : ''}
            </div>
        </div>`;
    
    const productData = productMap.get(state.currentProduct);
    if (productData) {
        formHtml += `<div id="product-details-form" class="space-y-4 mt-4">`; 
        
        formHtml += `<div id="size-selection-step" class="p-4 rounded-xl space-y-4">`;
        const showSizeGrid = productData.type === 'haut' || productData.type === 'enfant' || (productData.type === 'accessoire' && productData.sizeType && productData.sizeType !== 'unique');
        if (showSizeGrid) {
            const sizes = productSizes[productData.sizeType || productData.type] || [];
            const sizeInputs = sizes.map(size => {
                const stockQty = state.clubStock[productData.name]?.[size] ?? 0;
                const stockColor = stockQty > 0 ? 'text-green-600' : 'text-gray-500';
                return `<div>
                            <label for="size-${size}" class="block text-sm font-medium text-gray-700">${size}</label>
                            <input type="number" id="size-${size}" data-size="${size}" class="size-input mt-1 block w-full rounded-md border-gray-300 shadow-sm" placeholder="0" value="${state.currentQuantities[size] || ''}">
                            <span class="stock-info ${stockColor}">Stock: ${stockQty}</span>
                        </div>`
            }).join('');
            formHtml += `<div class="grid grid-cols-4 sm:grid-cols-6 md:grid-cols-8 lg:grid-cols-12 gap-2">${sizeInputs}</div>`;
        }
        if (productData.type === 'accessoire' && (!productData.sizeType || productData.sizeType === 'unique')) {
            const stockQty = state.clubStock[productData.name]?.['U'] ?? 0;
            const stockColor = stockQty > 0 ? 'text-green-600' : 'text-gray-500';
            formHtml += `<div>
                <label for="accessory-qty" class="block text-sm font-medium text-gray-700">Quantité</label>
                <input type="number" id="accessory-qty" value="${state.currentAccessoryQuantity}" min="1" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
                <span class="stock-info ${stockColor}">Stock: ${stockQty}</span>
                ${productData.minQuantity && state.orderScope === 'global' ? `<p class="text-xs text-gray-500 mt-1">Quantité minimale : ${productData.minQuantity}</p>` : ''}
            </div>`;
        }
        if (productData.colors) {
            const colorOptions = productData.colors.map(c => `<option value="${c}" ${state.currentColor === c ? 'selected' : ''}>${c}</option>`).join('');
            formHtml += `<div><label class="block text-sm font-medium text-gray-700">Coloris</label><select id="current-color-select" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm"><option value="">-- Sélectionner un coloris --</option>${colorOptions}</select></div>`;
        }
        formHtml += `</div>`; 

        formHtml += `<div class="p-4 rounded-xl space-y-4">`;
        let normalOptions = [];
        let lengthOptions = [];
        const isLongSleeved = productData.type === 'haut' && (productData.name.includes('ML') || productData.name.includes('MANCHES LONGUES'));
        if ((productData.isCuissardOrCollant || isLongSleeved)) {
            lengthOptions = allAvailableProducts.filter(p => p.optionGroup === 'length');
        }
        if (productData.hasOptions !== false && productData.type !== 'accessoire' && !productData.isCuissardOrCollant) {
            normalOptions = allAvailableProducts.filter(p => p.category === 'option' && !p.optionGroup && !productData.excludedOptions?.includes(p.name));
        }
        if (lengthOptions.length > 0) {
            const optionCheckboxes = lengthOptions.map(opt => `<div class="flex items-center"><input id="option-${opt.name}" type="checkbox" data-option-name="${opt.name}" data-option-group="length" class="option-checkbox h-4 w-4 rounded border-gray-300 text-indigo-600" ${state.currentSelectedOptions.includes(opt.name) ? 'checked' : ''}><label for="option-${opt.name}" class="ml-3 block text-sm text-gray-900">${opt.name.replace('Ajustement Longueur ', '')} (+${opt.fixedPriceTTC.toFixed(2)}€)</label></div>`).join('');
            formHtml += `<div><label class="block text-sm font-medium text-gray-700 mb-2">Ajustement Longueur</label><div class="grid grid-cols-2 md:grid-cols-3 gap-x-4 gap-y-2">${optionCheckboxes}</div></div>`;
        }
        if (normalOptions.length > 0) {
            const optionCheckboxes = normalOptions.map(opt => `<div class="flex items-center"><input id="option-${opt.name}" type="checkbox" data-option-name="${opt.name}" class="option-checkbox h-4 w-4 rounded border-gray-300 text-indigo-600" ${state.currentSelectedOptions.includes(opt.name) ? 'checked' : ''}><label for="option-${opt.name}" class="ml-3 block text-sm text-gray-900">${opt.name}</label></div>`).join('');
            formHtml += `<div><label class="block text-sm font-medium text-gray-700 mb-2">Options</label><div class="space-y-2">${optionCheckboxes}</div></div>`;
        }
        formHtml += `<div><label for="specificity" class="block text-sm font-medium text-gray-700">Spécificité / Notes</label><textarea id="specificity" rows="2" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">${state.currentSpecificity}</textarea></div>`;
        formHtml += `</div>`;
        
        formHtml += `<div id="add-button-step" class="p-4 rounded-xl">`;
        formHtml += `<div class="flex items-center justify-between gap-4 pt-4 border-t">`;
        formHtml += `<button id="reset-product-form-btn" class="inline-flex items-center px-4 py-2 border border-gray-300 text-sm font-medium rounded-md shadow-sm text-gray-700 bg-white hover:bg-gray-50">Réinitialiser</button>`;
        formHtml += `<div class="flex items-center">
            <input id="add-to-stock-check" type="checkbox" class="h-4 w-4 rounded border-gray-300 text-green-600 focus:ring-green-500" ${state.isAddingForStock ? 'checked' : ''}>
            <label for="add-to-stock-check" class="ml-2 block text-sm font-bold text-green-700">📦 Commander pour le stock</label>
        </div>`;
        if (state.isReassort && productData.type !== 'accessoire') {
            formHtml += `<div class="flex-grow"><label for="manual-price" class="block text-sm font-medium text-gray-700">Prix U. TTC Manuel</label><input type="number" id="manual-price" value="${state.manualUnitPrice}" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm" placeholder="0.00"></div>`;
        } else if (state.currentCalculatedUnitPrice > 0) {
            formHtml += `<div class="bg-indigo-50 p-3 rounded-lg flex-grow text-center"><p class="text-indigo-800 font-semibold">Prix unitaire TTC : <span class="text-xl">${state.currentCalculatedUnitPrice.toFixed(2)}€</span></p></div>`;
        } else {
            formHtml += '<div class="flex-grow"></div>';
        }
        formHtml += `<button id="add-product-btn" class="inline-flex items-center px-6 py-3 border border-transparent text-base font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 disabled:bg-indigo-300">Ajouter Article</button></div>`;
        formHtml += `</div>`; 

        formHtml += `</div>`;
    }
    
    dom.productFormContainer.innerHTML = formHtml;
    updateButtonStates();
};// =================================================================================
// --- STOCK MANAGEMENT ---
// =================================================================================

const saveStock = () => {
    if (!state.clubName) return;
    const stockKey = `stock_${state.clubName.replace(/[\s/\\?%*:|"<>]/g, '_')}`;
    localStorage.setItem(stockKey, JSON.stringify(state.clubStock));
    showToast('Stock sauvegardé !');
    renderDashboard();
};

const loadStock = () => {
    if (!state.clubName) {
        state.clubStock = {};
    } else {
        const stockKey = `stock_${state.clubName.replace(/[\s/\\?%*:|"<>]/g, '_')}`;
        const savedStock = localStorage.getItem(stockKey);
        state.clubStock = savedStock ? JSON.parse(savedStock) : {};
    }
    renderAll();
};

const showStockManagerModal = () => {
    if (!state.clubName) {
        showToast("Veuillez d'abord renseigner un nom de club.", 'error');
        return;
    }

    const container = document.createElement('div');
    container.className = 'space-y-4';
    container.innerHTML = `
        <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4 pb-4 border-b border-gray-200">
            <button id="export-stock-btn" class="w-full px-4 py-2 bg-blue-600 text-white font-medium rounded-md hover:bg-blue-700">Exporter (JSON)</button>
            <label for="import-stock-input" class="w-full text-center cursor-pointer px-4 py-2 bg-gray-600 text-white font-medium rounded-md hover:bg-gray-700">Importer (JSON)</label>
            <button id="clear-stock-btn" class="w-full px-4 py-2 bg-red-600 text-white font-medium rounded-md hover:bg-red-700">Effacer le Stock</button>
        </div>
        <input type="text" id="stock-search-input" placeholder="Rechercher un produit..." class="w-full p-2 border rounded-md">
        <div id="stock-list-container" class="space-y-4 max-h-[50vh] overflow-y-auto pr-2"></div>
    `;

    const stockListContainer = container.querySelector('#stock-list-container');
    
    const renderStockList = (filter = '') => {
        stockListContainer.innerHTML = '';
        const productsWithSizes = allAvailableProducts.filter(p => p.category !== 'option' && (productSizes[p.sizeType || p.type] || p.sizeType === 'unique'));
        const filteredProducts = productsWithSizes.filter(p => p.name.toLowerCase().includes(filter.toLowerCase()));

        const grouped = filteredProducts.reduce((acc, p) => {
            const groupKey = `${p.category} - ${p.subtype}`;
            acc[groupKey] = [...(acc[groupKey] || []), p];
            return acc;
        }, {});

        for (const groupLabel in grouped) {
            const groupDiv = document.createElement('div');
            groupDiv.innerHTML = `<h4 class="font-bold text-gray-800 border-b pb-1 mb-2">${groupLabel}</h4>`;
            
            grouped[groupLabel].forEach(p => {
                const productDiv = document.createElement('div');
                productDiv.className = 'p-2 border-b';
                productDiv.innerHTML = `<p class="font-semibold">${p.name}</p>`;
                
                const sizes = productSizes[p.sizeType || p.type] || [];
                const sizeGrid = document.createElement('div');
                sizeGrid.className = 'grid grid-cols-3 sm:grid-cols-4 gap-x-4 gap-y-2 mt-2';
                
                sizes.forEach(size => {
                    const stockValue = state.clubStock[p.name]?.[size] ?? '';
                    sizeGrid.innerHTML += `
                        <div class="flex items-center gap-2">
                            <label class="text-sm w-16 text-right">${size}:</label>
                            <input type="number" class="stock-input w-20 rounded-md border-gray-300 shadow-sm text-center" 
                                   data-product-name="${p.name}" data-size="${size}" value="${stockValue}" placeholder="0">
                        </div>
                    `;
                });
                productDiv.appendChild(sizeGrid);
                groupDiv.appendChild(productDiv);
            });
            stockListContainer.appendChild(groupDiv);
        }
    };
    
    renderStockList();

    container.querySelectorAll('.stock-input').forEach(input => {
        input.addEventListener('input', (e) => {
            const { productName, size } = e.target.dataset;
            const qty = parseInt(e.target.value, 10);

            if (!state.clubStock[productName]) {
                state.clubStock[productName] = {};
            }

            if (!isNaN(qty) && qty > 0) {
                state.clubStock[productName][size] = qty;
            } else {
                delete state.clubStock[productName][size];
                if (Object.keys(state.clubStock[productName]).length === 0) {
                    delete state.clubStock[productName];
                }
            }
        });
    });

    container.querySelector('#export-stock-btn').addEventListener('click', handleExportStock);

    // ▼▼▼ DÉBUT DE LA MODIFICATION ▼▼▼
    container.querySelector('#clear-stock-btn').addEventListener('click', () => {
        const p = document.createElement('p');
        p.innerHTML = `Êtes-vous sûr de vouloir <strong>supprimer définitivement</strong> tout le stock pour le club <strong>${state.clubName}</strong> ?<br><br>Cette action est irréversible.`;
        showModal(dom.mainModal, "Confirmer la suppression du stock", p, [
            { label: 'Annuler', onClick: () => {
                hideModal(dom.mainModal);
                // On réaffiche la fenêtre de gestion du stock par dessus
                setTimeout(() => showStockManagerModal(), 50);
            }, className: 'bg-gray-400' },
            { label: 'Oui, tout effacer', onClick: () => {
                state.clubStock = {};
                saveStock(); // Sauvegarde le stock vide
                hideModal(dom.mainModal);
                showToast("Le stock a été effacé avec succès.", "success");
                // On réaffiche la fenêtre de gestion (maintenant vide)
                setTimeout(() => showStockManagerModal(), 50);
            }, className: 'bg-red-600'}
        ]);
    });
    // ▲▲▲ FIN DE LA MODIFICATION ▲▲▲

    container.querySelector('#stock-search-input').addEventListener('input', (e) => {
        renderStockList(e.target.value);
    });

    showModal(dom.mainModal, `Gestion du stock pour ${state.clubName}`, container, [
        { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
        { label: 'Enregistrer le stock', onClick: () => {
            document.querySelectorAll('.stock-input').forEach(input => {
                const { productName, size } = input.dataset;
                const qty = parseInt(input.value, 10);
                if (!state.clubStock[productName]) {
                    state.clubStock[productName] = {};
                }
                if (!isNaN(qty) && qty > 0) {
                    state.clubStock[productName][size] = qty;
                } else {
                    delete state.clubStock[productName][size];
                }
            });
            saveStock();
            hideModal(dom.mainModal);
        }, className: 'bg-green-600' }
    ], 'max-w-4xl');
};
const showVisualManagerModal = () => {
    if (!state.clubName) {
        showToast("Veuillez d'abord renseigner un nom de club.", 'error');
        return;
    }

    const renderManagerContent = () => {
        const container = document.createElement('div');
        container.className = 'space-y-4';
        
        container.innerHTML = `
            <p class="text-sm text-gray-600">Gérez ici les différents noms de visuels pour le club <strong>${state.clubName}</strong>. Ils seront ensuite disponibles lors de la saisie des articles.</p>
            <div class="flex gap-2 pt-4 border-t">
                <input type="text" id="new-visual-name" placeholder="Ajouter un nom de visuel..." class="block w-full rounded-md border-gray-300 shadow-sm">
                <button id="add-new-visual-btn" class="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700">Ajouter</button>
            </div>
            <div id="visual-list-container" class="space-y-2 max-h-40 overflow-y-auto"></div>
        `;

        const listContainer = container.querySelector('#visual-list-container');
        state.clubVisuals.sort((a,b) => a.localeCompare(b)).forEach(name => {
            const itemDiv = document.createElement('div');
            itemDiv.className = 'flex justify-between items-center p-2 border rounded-md bg-gray-50';
            itemDiv.innerHTML = `<span>${name}</span>
                                 <button data-visual-name="${name}" class="ml-4 text-xs bg-red-500 text-white px-2 py-1 rounded hover:bg-red-600">Supprimer</button>`;
            listContainer.appendChild(itemDiv);
        });

        container.querySelector('#add-new-visual-btn').onclick = () => {
            const input = container.querySelector('#new-visual-name');
            const newName = input.value.trim();
            if (newName && !state.clubVisuals.includes(newName)) {
                state.clubVisuals.push(newName);
                dom.mainModalBody.innerHTML = '';
                dom.mainModalBody.appendChild(renderManagerContent());
            }
            input.value = '';
            input.focus();
        };
        
        listContainer.querySelectorAll('button').forEach(btn => {
            btn.onclick = () => {
                state.clubVisuals = state.clubVisuals.filter(v => v !== btn.dataset.visualName);
                dom.mainModalBody.innerHTML = '';
                dom.mainModalBody.appendChild(renderManagerContent());
            };
        });

        return container;
    };

    showModal(dom.mainModal, 'Gérer les Visuels du Club', renderManagerContent(), [
        { label: 'Fermer', onClick: () => {
            hideModal(dom.mainModal);
            const visualsKey = `visuals_${state.clubName.replace(/[\s/\\?%*:|"<>]/g, '_')}`;
            localStorage.setItem(visualsKey, JSON.stringify(state.clubVisuals));
            renderProductForm();
        }, className: 'bg-gray-500' }
    ]);
};
const showClubRangeSelectorModal = () => {
    if (!state.clubName.trim() || !state.departement.trim()) {
        showToast("Veuillez d'abord renseigner le Nom du Club et le Département.", 'error');
        return;
    }

    const container = document.createElement('div');
    container.className = 'space-y-4';

    let showOnlyRange = state.clubProductRange.length > 0;

    const renderContent = () => {
        container.innerHTML = ''; 

        if (state.clubProductRange.length > 0) {
            container.innerHTML += `
                <div class="flex justify-center gap-4 mb-4 p-2 bg-gray-100 rounded-lg">
                    <button id="show-range-btn" class="px-4 py-2 text-sm rounded-md ${showOnlyRange ? 'bg-indigo-600 text-white' : 'bg-white text-gray-700'}">Afficher la Gamme (${state.clubProductRange.length})</button>
                    <button id="show-all-btn" class="px-4 py-2 text-sm rounded-md ${!showOnlyRange ? 'bg-indigo-600 text-white' : 'bg-white text-gray-700'}">Afficher Tout</button>
                </div>
            `;
        }
        
        container.innerHTML += `
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4 pb-4 border-b">
                <button id="export-range-btn" class="w-full px-4 py-2 bg-blue-600 text-white font-medium rounded-md hover:bg-blue-700">Exporter la Gamme</button>
                <label for="import-club-range-input" class="w-full text-center cursor-pointer px-4 py-2 bg-gray-600 text-white font-medium rounded-md hover:bg-gray-700">Importer une Gamme</label>
                <button id="clear-range-btn" class="w-full px-4 py-2 bg-red-600 text-white font-medium rounded-md hover:bg-red-700">Tout Effacer</button>
            </div>
        `;
        
        const sourceProducts = showOnlyRange ? allAvailableProducts.filter(p => state.clubProductRange.includes(p.name)) : allAvailableProducts;
        const productsToShow = sourceProducts.filter(p => p.category !== 'option');
        
        const grouped = productsToShow.reduce((acc, p) => {
            const groupKey = `${p.category} - ${p.subtype}`;
            acc[groupKey] = [...(acc[groupKey] || []), p];
            return acc;
        }, {});

        let contentHtml = '<div class="space-y-3 max-h-[50vh] overflow-y-auto pr-2">';
        for (const groupLabel in grouped) {
            contentHtml += `<div class="pt-2"><h4 class="font-bold text-gray-800 border-b pb-1 mb-2">${groupLabel}</h4>`;
            grouped[groupLabel].forEach(p => {
                // ▼▼▼ CORRECTION ICI : On vérifie dans state.clubProductRange ▼▼▼
                const isChecked = state.clubProductRange.includes(p.name);
                contentHtml += `<div class="flex items-center my-1">
                    <input id="range-prod-${p.name}" type="checkbox" data-product-name="${p.name}" class="range-product-checkbox h-4 w-4 rounded border-gray-300 text-indigo-600" ${isChecked ? 'checked' : ''}>
                    <label for="range-prod-${p.name}" class="ml-3 block text-sm text-gray-900">${p.name}</label>
                </div>`;
            });
            contentHtml += `</div>`;
        }
        contentHtml += `</div>`;
        container.innerHTML += contentHtml;

        const showRangeBtn = container.querySelector('#show-range-btn');
        if (showRangeBtn) {
            showRangeBtn.addEventListener('click', () => {
                showOnlyRange = true;
                renderContent();
            });
        }
        const showAllBtn = container.querySelector('#show-all-btn');
        if (showAllBtn) {
            showAllBtn.addEventListener('click', () => {
                showOnlyRange = false;
                renderContent();
            });
        }

        const clearRangeBtn = container.querySelector('#clear-range-btn');
        if (clearRangeBtn) {
            clearRangeBtn.addEventListener('click', () => {
                if (confirm("Êtes-vous sûr de vouloir effacer tous les articles de la gamme de ce club ?")) {
                    state.clubProductRange = [];
                    const rangeKey = `range_${state.clubName.replace(/[\s/\\?%*:|"<>]/g, '_')}`;
                    localStorage.removeItem(rangeKey);
                    showToast('Gamme du club effacée.', 'success');
                    renderContent();
                }
            });
        }
        
        const exportRangeBtn = container.querySelector('#export-range-btn');
        if (exportRangeBtn) {
            exportRangeBtn.addEventListener('click', handleExportClubRange);
        }
    };
    
    renderContent();

    showModal(dom.mainModal, `Gamme pour ${state.clubName}`, container, [
        { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
        // ▼▼▼ CORRECTION DE TOUTE LA LOGIQUE DE SAUVEGARDE ▼▼▼
        { 
            label: 'Enregistrer la Gamme', 
            onClick: () => {
                const checkedProducts = Array.from(container.querySelectorAll('.range-product-checkbox:checked')).map(cb => cb.dataset.productName);
                
                state.clubProductRange = checkedProducts;
                
                const rangeKey = `range_${state.clubName.replace(/[\s/\\?%*:|"<>]/g, '_')}`;
                localStorage.setItem(rangeKey, JSON.stringify(state.clubProductRange));
                
                hideModal(dom.mainModal);
                showToast(`Gamme du club mise à jour avec ${state.clubProductRange.length} article(s).`);
                
                // On s'assure que la vue principale se met à jour pour n'afficher que la gamme
                state.showAllProducts = false; 
                renderProductForm();
            }, 
            className: 'bg-green-600' 
        }
    ], 'max-w-2xl');
};
// =================================================================================
// --- EVENT HANDLERS & LOGIC ---
// =================================================================================
const handleExportStock = () => {
    if (!state.clubName) {
        showToast("Veuillez sélectionner un club pour exporter son stock.", "error");
        return;
    }
    if (Object.keys(state.clubStock).length === 0) {
        showToast("Le stock est vide, rien à exporter.", "error");
        return;
    }

    // Création d'un objet structuré pour l'export
    const exportData = {
        nomClub: state.clubName,
        dateEnregistrement: new Date().toISOString().split('T')[0],
        stock: state.clubStock
    };

    try {
        const dataStr = JSON.stringify(exportData, null, 2);
        const blob = new Blob([dataStr], { type: "application/json" });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `stock_${state.clubName.replace(/ /g, '_')}.json`;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
        showToast('Fichier de stock exporté avec succès !');
    } catch (error) {
        console.error("Erreur lors de l'export du stock", error);
        showToast("Une erreur est survenue lors de l'exportation.", 'error');
    }
};
const handleImportStock = (event) => {
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (e) => {
        try {
            const loadedData = JSON.parse(e.target.result);
            let stockToLoad;
            let fileInfo = '';

            // Vérifie si le fichier a la nouvelle structure (avec date) ou l'ancienne
            if (loadedData.stock && loadedData.dateEnregistrement) {
                stockToLoad = loadedData.stock;
                fileInfo = ` (Fichier de ${loadedData.nomClub} du ${loadedData.dateEnregistrement})`;
            } else {
                stockToLoad = loadedData; // Compatibilité avec l'ancien format
            }

            if (typeof stockToLoad !== 'object' || stockToLoad === null || Array.isArray(stockToLoad)) {
                throw new Error("Les données de stock dans le fichier ne sont pas valides.");
            }

            const p = document.createElement('p');
            p.innerHTML = `Êtes-vous sûr de vouloir remplacer le stock actuel par le contenu de ce fichier${fileInfo} ?<br><br>Cette action est irréversible.`;
            
            showModal(dom.mainModal, "Confirmer l'importation du stock", p, [
                { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
                { label: "Oui, importer", onClick: () => {
                    state.clubStock = stockToLoad;
                    saveStock();
                    hideModal(dom.mainModal);
                    showToast("Stock importé et sauvegardé avec succès !", "success");
                    showStockManagerModal();
                }, className: 'bg-green-600'}
            ]);

        } catch (error) {
            console.error("Erreur d'importation du stock", error);
            showToast("Fichier invalide ou corrompu.", 'error');
        } finally {
            event.target.value = '';
        }
    };
    reader.readAsText(file);
};
const handleExportClubRange = () => {
    if (!state.clubName || state.clubProductRange.length === 0) {
        showToast("Aucune gamme à exporter pour ce club.", "error");
        return;
    }

    const exportData = {
        clubName: state.clubName,
        exportDate: new Date().toISOString().split('T')[0],
        type: "clubProductRange",
        productRange: state.clubProductRange
    };

    try {
        const dataStr = JSON.stringify(exportData, null, 2);
        const blob = new Blob([dataStr], { type: "application/json" });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `gamme_${state.clubName.replace(/ /g, '_')}.json`;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
        showToast('Fichier de la gamme exporté avec succès !');
    } catch (error) {
        console.error("Erreur lors de l'export de la gamme", error);
        showToast("Une erreur est survenue lors de l'exportation.", 'error');
    }
};
const handleImportClubRange = (event) => {
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (e) => {
        try {
            const loadedData = JSON.parse(e.target.result);
            if (loadedData.type !== "clubProductRange" || !Array.isArray(loadedData.productRange)) {
                throw new Error("Le fichier n'est pas un fichier de gamme valide.");
            }

            const p = document.createElement('p');
            p.innerHTML = `Voulez-vous remplacer la gamme actuelle par celle du fichier pour le club <b>${loadedData.clubName}</b> ?<br>(${loadedData.productRange.length} articles seront importés).`;
            
            showModal(dom.mainModal, "Confirmer l'importation", p, [
                { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
                { label: "Oui, importer", onClick: () => {
                    state.clubProductRange = loadedData.productRange;
                    const rangeKey = `range_${state.clubName.replace(/[\s/\\?%*:|"<>]/g, '_')}`;
                    localStorage.setItem(rangeKey, JSON.stringify(state.clubProductRange));
                    
                    hideModal(dom.mainModal);
                    showToast("Gamme importée et sauvegardée avec succès !", "success");
                    // On rafraîchit la fenêtre de gestion pour voir les nouvelles coches
                    showClubRangeSelectorModal(); 
                }, className: 'bg-green-600'}
            ]);

        } catch (error) {
            console.error("Erreur d'importation de la gamme", error);
            showToast("Fichier invalide ou corrompu.", 'error');
        } finally {
            event.target.value = ''; // Réinitialise l'input
        }
    };
    reader.readAsText(file);
};
const calculateCurrentFormPrice = () => {
    const product = productMap.get(state.currentProduct);
    if (!product || (state.isReassort && product.type !== 'accessoire')) {
        state.currentCalculatedUnitPrice = 0;
        return;
    }

    let currentFormQuantity = 0;
    if (state.documentType === 'devis' || (product.type === 'accessoire' && (!product.sizeType || product.sizeType === 'unique'))) {
        currentFormQuantity = parseInt(state.currentAccessoryQuantity, 10) || 0;
    } else {
        currentFormQuantity = Object.values(state.currentQuantities).reduce((sum, q) => sum + (parseInt(q, 10) || 0), 0);
    }
    
    let groupQuantityInCart = 0;
    if (product.pricingGroup || (product.subtype === 'ACCESSOIRES PERSONNALISÉS' && product.pricingTiers)) {
        groupQuantityInCart = state.currentOrderLineItems.filter(li => {
            const liProduct = productMap.get(li.productName);
            const isSameGroup = product.pricingGroup ? (liProduct && liProduct.pricingGroup === product.pricingGroup) : (li.productName === product.name);
            return isSameGroup;
        }).reduce((sum, li) => sum + li.totalQuantity, 0);
    }
    
    const totalPricingQuantity = currentFormQuantity + groupQuantityInCart;
    state.currentCalculatedUnitPrice = getUnitPriceTTC(state.currentProduct, totalPricingQuantity, state.currentSelectedOptions);
};

const resetProductForm = () => {
    state.currentProduct = '';
    state.currentQuantities = {};
    state.currentCalculatedUnitPrice = 0;
    state.manualUnitPrice = '';
    state.currentSelectedOptions = [];
    state.currentSpecificity = '';
    state.currentAccessoryQuantity = '';
    state.currentColor = '';
    state.currentVisual = ''; // On réinitialise le visuel
    renderProductForm();
};
const handleNextLicensee = () => {
    const newName = state.licencieName.trim();
    if (!newName) {
        showToast("Veuillez saisir un nom de licencié.", 'error');
        return;
    }
    if (!state.licenseeList.includes(newName)) {
        state.licenseeList.push(newName);
        showToast(`'${newName}' ajouté à la liste.`);
    }
    state.activeLicensee = newName;
    state.licencieName = '';
    renderAll(); // C'est ici que updateSectionHighlights() est appelée
    dom.licencieNameInput.focus();

    document.getElementById('add-article-section').scrollIntoView({ behavior: 'smooth', block: 'start' });
};
const handleProductAdd = () => {
    const product = productMap.get(state.currentProduct);

    if (state.orderScope === 'licensee' && !state.activeLicensee && !state.isAddingForStock) {
        showToast("Veuillez sélectionner un licencié ou cocher 'Commander pour le stock'.", 'error');
        return;
    }

    if (!product) return;

    let totalQuantity = 0;
    let quantitiesPerSize = {};

    if (product.type === 'accessoire' && (!product.sizeType || product.sizeType === 'unique')) {
        totalQuantity = parseInt(state.currentAccessoryQuantity, 10) || 0;
        if (totalQuantity > 0) quantitiesPerSize = { 'U': totalQuantity };
    } else {
        for (const size in state.currentQuantities) {
            const qty = parseInt(state.currentQuantities[size], 10) || 0;
            if (qty > 0) {
                quantitiesPerSize[size] = qty;
                totalQuantity += qty;
            }
        }
    }

    if (totalQuantity === 0) {
        showToast("Veuillez entrer une quantité valide.", 'error');
        return;
    }
    
    const isManualPrice = state.isReassort && parseFloat(state.manualUnitPrice) > 0;
    const initialManualPrice = isManualPrice ? parseFloat(state.manualUnitPrice) : 0;
    const isForStockOrder = state.isAddingForStock;

    let stockQuantities = {};
    let productionQuantities = {};
    let totalFromStock = 0;
    let totalForProduction = 0;

    if (!isForStockOrder) {
        for (const size in quantitiesPerSize) {
            const requestedQty = parseInt(quantitiesPerSize[size], 10) || 0;
            if (requestedQty > 0) {
                const stockAvailable = state.clubStock[product.name]?.[size] ?? 0;
                const takenFromStock = Math.min(requestedQty, stockAvailable);
                if (takenFromStock > 0) {
                    stockQuantities[size] = takenFromStock;
                    totalFromStock += takenFromStock;
                    state.clubStock[product.name][size] -= takenFromStock;
                }
                const neededForProduction = requestedQty - takenFromStock;
                if (neededForProduction > 0) {
                    productionQuantities[size] = neededForProduction;
                    totalForProduction += neededForProduction;
                }
            }
        }
    } else {
        productionQuantities = quantitiesPerSize;
        totalForProduction = totalQuantity;
    }

    const mergeQuantities = (targetItem, newQuantities) => {
        for (const size in newQuantities) {
            const newQty = parseInt(newQuantities[size], 10) || 0;
            if (newQty > 0) {
                targetItem.quantitiesPerSize[size] = (parseInt(targetItem.quantitiesPerSize[size], 10) || 0) + newQty;
            }
        }
        targetItem.totalQuantity = Object.values(targetItem.quantitiesPerSize).reduce((sum, q) => sum + q, 0);
    };

    if (totalFromStock > 0) {
        const licenseeForStockItem = (state.orderScope === 'licensee' && state.activeLicensee) ? state.activeLicensee : 'Commande Globale';
        const existingStockItem = state.currentOrderLineItems.find(item => 
            item.productName === product.name &&
            item.isFromStock === true &&
            item.licencieName === licenseeForStockItem &&
            JSON.stringify(item.options) === JSON.stringify(state.currentSelectedOptions) &&
            item.specificity === state.currentSpecificity &&
            item.visual === state.currentVisual
        );

        if (existingStockItem) {
            mergeQuantities(existingStockItem, stockQuantities);
        } else {
            state.currentOrderLineItems.push({
                id: Date.now() + Math.random(), productName: product.name, quantitiesPerSize: stockQuantities,
                totalQuantity: totalFromStock, options: state.currentSelectedOptions, specificity: state.currentSpecificity,
                isFromStock: true, isStockOrder: false, licencieName: licenseeForStockItem,
                isManualPrice: isManualPrice, initialManualPrice: initialManualPrice,
                visual: state.currentVisual,
            });
        }
    }

    if (totalForProduction > 0) {
        let finalLicenseeName = (isForStockOrder) ? 'Stock Club' : (state.orderScope === 'licensee' && state.activeLicensee) ? state.activeLicensee : 'Commande Globale';
        const existingProductionItem = state.currentOrderLineItems.find(item =>
            item.productName === product.name &&
            item.isFromStock === false &&
            item.isStockOrder === isForStockOrder &&
            item.licencieName === finalLicenseeName &&
            JSON.stringify(item.options) === JSON.stringify(state.currentSelectedOptions) &&
            item.specificity === state.currentSpecificity &&
            item.isManualPrice === isManualPrice &&
            item.visual === state.currentVisual
        );

        if (existingProductionItem) {
            mergeQuantities(existingProductionItem, productionQuantities);
        } else {
            state.currentOrderLineItems.push({
                id: Date.now() + Math.random(), productName: product.name, quantitiesPerSize: productionQuantities,
                totalQuantity: totalForProduction, options: state.currentSelectedOptions, specificity: state.currentSpecificity,
                isFromStock: false, isStockOrder: isForStockOrder, licencieName: finalLicenseeName,
                isManualPrice: isManualPrice, initialManualPrice: initialManualPrice,
                visual: state.currentVisual,
            });
        }
    }
    
    if (state.showAllProducts && state.clubProductRange.length > 0 && !state.clubProductRange.includes(product.name)) {
        state.clubProductRange.push(product.name);
        const rangeKey = `range_${state.clubName.replace(/[\s/\\?%*:|"<>]/g, '_')}`;
        localStorage.setItem(rangeKey, JSON.stringify(state.clubProductRange));
        showToast(`"${product.name}" a été ajouté à la gamme du club.`, 'info');
    }

    const licenseeNameForModal = state.activeLicensee;
    resetProductForm();
    renderAll();
    showToast('Article(s) ajouté(s)/mis à jour.');

    if (state.orderScope === 'licensee' && licenseeNameForModal) {
        const content = document.createElement('p');
        content.textContent = `Voulez-vous ajouter un autre article pour ${licenseeNameForModal} ?`;
        
        showModal(dom.mainModal, 'Continuer la saisie ?', content, [
            {
                label: 'Non, changer de licencié',
                className: 'bg-gray-500 hover:bg-gray-600 text-white',
                onClick: () => {
                    hideModal(dom.mainModal);
                    state.activeLicensee = '';
                    renderAll();
                    dom.licencieNameInput.scrollIntoView({ behavior: 'smooth', block: 'center' });
                    dom.licencieNameInput.focus();
                }
            },
            {
                label: 'Oui, ajouter un autre',
                className: 'bg-indigo-600 hover:bg-indigo-700 text-white',
                onClick: () => {
                    hideModal(dom.mainModal);
                    document.getElementById('add-article-section').scrollIntoView({ behavior: 'smooth', block: 'start' });
                }
            }
        ]);
    } else if (state.orderScope === 'global') {
        const content = document.createElement('p');
        content.textContent = "L'article a bien été ajouté à la commande. Souhaitez-vous en saisir un autre ?";
        
        showModal(dom.mainModal, 'Continuer la saisie ?', content, [
            {
                label: 'Non, voir le récapitulatif',
                className: 'bg-gray-500 hover:bg-gray-600 text-white',
                onClick: () => {
                    hideModal(dom.mainModal);
                    document.getElementById('summary-and-actions-section').scrollIntoView({ behavior: 'smooth', block: 'start' });
                }
            },
            {
                label: 'Oui, saisir un autre',
                className: 'bg-indigo-600 hover:bg-indigo-700 text-white',
                onClick: () => {
                    hideModal(dom.mainModal);
                    document.getElementById('add-article-section').scrollIntoView({ behavior: 'smooth', block: 'start' });
                }
            }
        ]);
    }
};const promptForLastDeliveryDate = () => {
    const content = document.createElement('div');
    content.innerHTML = `
        <label for="last-delivery-date" class="block text-sm font-medium text-gray-700">Veuillez indiquer la date de livraison de la commande précédente :</label>
        <input type="date" id="last-delivery-date" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
    `;
    
    showModal(dom.mainModal, 'Date de dernière livraison', content, [
        { 
            label: 'Annuler', 
            className: 'bg-gray-400',
            onClick: () => {
                dom.docTypeReassortCheck.checked = false; // On décoche la case
                hideModal(dom.mainModal);
            } 
        },
        { 
            label: 'Valider la date', 
            className: 'bg-green-600',
            onClick: () => {
                const dateInput = document.getElementById('last-delivery-date');
                if (!dateInput.value) {
                    showToast("Veuillez entrer une date.", 'error');
                    return;
                }

                const lastDelivery = new Date(dateInput.value);
                const deadline = new Date(lastDelivery);
                deadline.setMonth(deadline.getMonth() + 2); // Ajoute 2 mois
                
                const today = new Date();
                today.setHours(0, 0, 0, 0); // On ignore l'heure pour la comparaison

                if (today > deadline) {
                    showToast(`Le délai de réassort de 2 mois est dépassé (date limite : ${deadline.toLocaleDateString('fr-FR')}).`, 'error');
                    dom.docTypeReassortCheck.checked = false;
                } else {
                    if (state.currentOrderLineItems.length > 0) {
                        if (!confirm("Activer le mode réassort videra le panier actuel. Continuer ?")) {
                            dom.docTypeReassortCheck.checked = false;
                            hideModal(dom.mainModal);
                            return;
                        }
                        state.currentOrderLineItems = [];
                    }
                    state.isReassort = true;
                    state.lastDeliveryDate = dateInput.value;
                    showToast('Mode Réassort activé.', 'success');
                    renderAll();
                }
                hideModal(dom.mainModal);
            }
        }
    ]);
};

const showReassortInfoModal = () => {
    const content = document.createElement('div');
    content.innerHTML = `
        <p class="text-sm">Vous activez le mode "Réassort 2 mois".</p>
        <ul class="list-disc list-inside mt-3 space-y-2 text-sm">
            <li>Ce mode permet de commander des articles manquants en conservant les tarifs de votre commande précédente (saisie manuelle des prix).</li>
            <li><strong>Minimum requis :</strong> 2 pièces par référence et 10 pièces au total (hors accessoires).</li>
        </ul>
    `;
    showModal(dom.mainModal, 'Information Réassort', content, [
        { 
            label: 'Annuler',
            className: 'bg-gray-400',
            onClick: () => {
                dom.docTypeReassortCheck.checked = false; // On décoche la case
                hideModal(dom.mainModal);
            }
        },
        { 
            label: 'Continuer',
            className: 'bg-indigo-600',
            onClick: () => {
                hideModal(dom.mainModal);
                setTimeout(promptForLastDeliveryDate, 100); // On lance la 2ème fenêtre
            }
        }
    ]);
};
const resetOrderDetails = () => {
    const preservedState = {
        clubName: state.clubName,
        departement: state.departement,
        clientNumber: state.clientNumber,
        licenseeList: state.licenseeList,
        licenseeDeposits: state.licenseeDeposits,
        clubStock: state.clubStock,
        clubVisuals: state.clubVisuals,
    };

    const resetState = {
        documentType: 'commande', isReassort: false, isCustomCreation: false, isStoreOrder: false,
        applyDiscount: false, orderDate: new Date().toISOString().split('T')[0], licencieName: '',
        activeLicensee: '', clubDiscount: 0, currentOrderLineItems: [], discountType: 'global',
        orderScope: '', orderSpecificity: '', portalProductSelection: [], portalSessionName: '',
        portalDeadline: '', currentProduct: '', currentQuantities: {}, currentCalculatedUnitPrice: 0,
        manualUnitPrice: '', currentSelectedOptions: [], currentSpecificity: '', currentAccessoryQuantity: '',
        currentColor: '', currentVisual: '',
        preOrderNumber: '', factoryDepartureDate: '', deliveryAddress: '', deliveryContact: '', // On réinitialise aussi les nouveaux champs
    };
    
    if (!preservedState.clubName) {
        preservedState.licenseeDeposits = {};
    }

    Object.assign(state, resetState, preservedState);
    renderAll();
};

const handleNewOrder = () => {
    const p = document.createElement('p');
    p.textContent = `Êtes-vous sûr de vouloir commencer une nouvelle commande ? Les articles du panier actuel seront supprimés. Les informations du club (nom, licenciés, acomptes, stock) seront conservées.`;
    showModal(dom.mainModal, 'Confirmation', p, [
        { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-300 hover:bg-gray-400 text-gray-800' },
        { label: 'Commencer une nouvelle commande', onClick: () => { 
            resetOrderDetails();
            localStorage.removeItem('autosavedOrder');
            hideModal(dom.mainModal); 
        }, className: 'bg-red-600 hover:bg-red-700 text-white' }
    ]);
};
    
const handleSaveOrderToFile = () => {
    if (!state.clubName.trim()) {
        showToast('Veuillez entrer un nom de club avant de sauvegarder.', 'error');
        return;
    }
    try {
        const dataStr = JSON.stringify(state, null, 2);
        const blob = new Blob([dataStr], { type: "application/json" });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        // ▼▼▼ NOM DU FICHIER MODIFIÉ ICI ▼▼▼
        a.download = `bon_de_commande_${state.orderDate}.json`;
        // ▲▲▲ FIN DE LA MODIFICATION ▲▲▲
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
        showToast('Fichier de sauvegarde (.json) exporté !');
    } catch (error) {
        console.error("Error saving order to file", error);
        showToast("Erreur lors de l'exportation du fichier .json.", 'error');
    }
};
const handleLoadOrderFromFile = (event) => {
    const file = event.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (e) => {
        try {
            const loadedState = JSON.parse(e.target.result);
            if (typeof loadedState.clubName !== 'string' || !Array.isArray(loadedState.currentOrderLineItems)) {
                throw new Error("Invalid file format.");
            }
            // On s'assure que toutes les propriétés existent pour éviter les erreurs avec d'anciens fichiers
            loadedState.licenseeList = loadedState.licenseeList || [];
            loadedState.licenseeDeposits = loadedState.licenseeDeposits || {};
            loadedState.portalProductSelection = loadedState.portalProductSelection || [];
            loadedState.portalSessionName = loadedState.portalSessionName || '';
            loadedState.portalDeadline = loadedState.portalDeadline || '';
            loadedState.clubStock = loadedState.clubStock || {};
            loadedState.clubVisuals = loadedState.clubVisuals || [];
            loadedState.preOrderNumber = loadedState.preOrderNumber || '';
            loadedState.factoryDepartureDate = loadedState.factoryDepartureDate || '';
            loadedState.deliveryAddress = loadedState.deliveryAddress || '';
            loadedState.deliveryContact = loadedState.deliveryContact || '';
            
            // ▼▼▼ CORRECTION POUR LA REMISE ▼▼▼
            loadedState.applyDiscount = loadedState.applyDiscount || false;
            loadedState.clubDiscount = loadedState.clubDiscount || 0;
            loadedState.discountType = loadedState.discountType || 'global';
            // ▲▲▲ FIN DE LA CORRECTION ▲▲▲

            Object.assign(state, loadedState);
            saveClientInfo(); 
            renderAll();
            showToast('Commande importée avec succès !');
        } catch (error) {
            console.error("Error loading order from file", error);
            showToast('Fichier invalide ou corrompu.', 'error');
        } finally {
            event.target.value = '';
        }
    };
    reader.readAsText(file);
};const handleImportLicensees = (event) => {
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (e) => {
        try {
            const data = new Uint8Array(e.target.result);
            const workbook = XLSX.read(data, { type: 'array' });
            const firstSheetName = workbook.SheetNames[0];
            const worksheet = workbook.Sheets[firstSheetName];
            const json = XLSX.utils.sheet_to_json(worksheet, { header: 1 });
            
            let importedCount = 0;
            json.forEach(row => {
                const name = row[0] ? String(row[0]).trim() : '';
                if (name && !state.licenseeList.includes(name)) {
                    state.licenseeList.push(name);
                    importedCount++;
                }
            });
            
            renderLicenseeDatalist();
            showLicenseeManagerModal();
            showToast(`${importedCount} licencié(s) importé(s) avec succès !`);
        } catch (error) {
            console.error("Error importing licensees", error);
            showToast("Erreur lors de l'importation du fichier Excel.", 'error');
        } finally {
            event.target.value = '';
        }
    };
    reader.readAsArrayBuffer(file);
};

const handleExportLicensees = () => {
    if (state.licenseeList.length === 0) {
        showToast("La liste des licenciés est vide.", 'error');
        return;
    }

    try {
        const dataForSheet = [
            ["Nom du Licencié"],
            ...state.licenseeList.map(name => [name])
        ];
        const wb = XLSX.utils.book_new();
        const ws = XLSX.utils.aoa_to_sheet(dataForSheet);
        ws['!cols'] = [{ wch: 30 }];
        XLSX.utils.book_append_sheet(wb, ws, 'Licenciés');
        const fileName = `liste_licencies_${state.clubName.replace(/ /g, '_') || 'club'}_${state.orderDate}.xlsx`;
        XLSX.writeFile(wb, fileName);
        showToast("Liste des licenciés exportée avec succès !");
    } catch (error) {
        console.error("Error exporting licensees to Excel", error);
        showToast("Erreur lors de l'exportation de la liste.", 'error');
    }
};

const showEditItemModal = (itemId) => {
    const itemIndex = state.currentOrderLineItems.findIndex(i => i.id == itemId);
    if (itemIndex === -1) return;
    
    const itemToEdit = { ...state.currentOrderLineItems[itemIndex] };
    const product = productMap.get(itemToEdit.productName);
    if (!product) return;

    const container = document.createElement('div');
    container.className = 'space-y-4';
    
    const sizes = productSizes[product.sizeType || product.type] || [];
    if (sizes.length > 0) {
        const sizeGrid = document.createElement('div');
        sizeGrid.className = 'grid grid-cols-3 gap-4';
        sizes.forEach(size => {
            const sizeDiv = document.createElement('div');
            const label = document.createElement('label');
            label.textContent = size;
            label.className = 'block text-sm font-medium text-gray-700';
            const input = document.createElement('input');
            input.type = 'number';
            input.dataset.size = size;
            input.className = 'mt-1 block w-full rounded-md border-gray-300 shadow-sm modal-size-input';
            input.value = itemToEdit.quantitiesPerSize[size] || '';
            sizeDiv.appendChild(label);
            sizeDiv.appendChild(input);
            sizeGrid.appendChild(sizeDiv);
        });
        container.appendChild(sizeGrid);
    }

    showModal(dom.mainModal, `Modifier: ${itemToEdit.productName}`, container, [
        { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-300' },
        { label: 'Enregistrer', onClick: () => {
            const newQuantities = {};
            let newTotalQuantity = 0;
            document.querySelectorAll('.modal-size-input').forEach(input => {
                const qty = parseInt(input.value, 10) || 0;
                if (qty > 0) {
                    newQuantities[input.dataset.size] = qty;
                    newTotalQuantity += qty;
                }
            });

            if (newTotalQuantity > 0) {
                state.currentOrderLineItems[itemIndex].quantitiesPerSize = newQuantities;
                state.currentOrderLineItems[itemIndex].totalQuantity = newTotalQuantity;
            } else {
                state.currentOrderLineItems.splice(itemIndex, 1);
            }
            
            hideModal(dom.mainModal);
            renderAll();
            showToast('Article modifié avec succès !');
        }, className: 'bg-green-600' }
    ]);
};

const showLicenseeManagerModal = () => {
    const renderManagerContent = () => {
        const container = document.createElement('div');
        container.className = 'space-y-4';
        
        container.innerHTML = `
            <div class="grid grid-cols-2 gap-2">
                <label for="import-licensees-input" class="w-full text-center block px-4 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 cursor-pointer">Importer Excel</label>
                <button id="export-licensees-btn" class="w-full text-center block px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700">Exporter Excel</button>
            </div>
            <p class="text-xs text-gray-500 text-center -mt-2">Le fichier doit contenir les noms dans la première colonne (A).</p>
            <div class="flex gap-2 pt-4 border-t">
                <input type="text" id="new-licensee-name" placeholder="Ou ajouter un nom manuellement" class="block w-full rounded-md border-gray-300 shadow-sm">
                <button id="add-new-licensee-btn" class="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700">Ajouter</button>
            </div>
            <div id="licensee-list-container" class="space-y-2 max-h-40 overflow-y-auto"></div>
        `;

        const listContainer = container.querySelector('#licensee-list-container');
        const licenseesWithOrders = new Set(state.currentOrderLineItems.map(item => item.licencieName));

        state.licenseeList.sort((a,b) => a.localeCompare(b)).forEach(name => {
            const itemDiv = document.createElement('div');
            itemDiv.className = 'flex justify-between items-center p-2 border rounded-md';
            const nameSpan = document.createElement('span');
            nameSpan.textContent = name;
            if (licenseesWithOrders.has(name)) nameSpan.className = 'text-red-500 font-bold';
            
            const deleteBtn = document.createElement('button');
            deleteBtn.textContent = 'Supprimer';
            deleteBtn.className = 'ml-4 text-xs bg-red-500 text-white px-2 py-1 rounded hover:bg-red-600';
            deleteBtn.onclick = () => {
                state.licenseeList = state.licenseeList.filter(n => n !== name);
                renderLicenseeDatalist();
                dom.mainModalBody.innerHTML = '';
                dom.mainModalBody.appendChild(renderManagerContent());
            };
            itemDiv.appendChild(nameSpan);
            itemDiv.appendChild(deleteBtn);
            listContainer.appendChild(itemDiv);
        });

        container.querySelector('#add-new-licensee-btn').onclick = () => {
            const input = container.querySelector('#new-licensee-name');
            const newName = input.value.trim();
            if (newName && !state.licenseeList.includes(newName)) {
                state.licenseeList.push(newName);
                renderLicenseeDatalist();
                dom.mainModalBody.innerHTML = '';
                dom.mainModalBody.appendChild(renderManagerContent());
            }
            input.value = '';
            input.focus();
        };

        return container;
    };

    showModal(dom.mainModal, 'Gérer les Licenciés', renderManagerContent(), [
        { label: 'Fermer', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-500' }
    ]);
};

const showDepositModal = (licenseeName) => {
    const container = document.createElement('div');
    container.className = 'space-y-3';
    container.innerHTML = `
        <p>Saisir le montant de l'acompte pour <b>${licenseeName}</b>.</p>
        <div>
            <label for="deposit-amount" class="block text-sm font-medium text-gray-700">Montant de l'acompte (€)</label>
            <input type="number" id="deposit-amount" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm" placeholder="0.00" value="${state.licenseeDeposits[licenseeName] || ''}">
        </div>
    `;
    
    showModal(dom.mainModal, `Acompte pour ${licenseeName}`, container, [
        { label: 'Supprimer Acompte', onClick: () => {
            delete state.licenseeDeposits[licenseeName];
            hideModal(dom.mainModal);
            renderAll();
            showToast(`Acompte pour ${licenseeName} supprimé.`);
        }, className: 'bg-red-600 hover:bg-red-700 text-white' },
        { label: 'Enregistrer', onClick: () => {
            const amount = parseFloat(document.getElementById('deposit-amount').value) || 0;
            if (amount > 0) {
                state.licenseeDeposits[licenseeName] = amount;
            } else {
                delete state.licenseeDeposits[licenseeName];
            }
            hideModal(dom.mainModal);
            renderAll();
            showToast(`Acompte pour ${licenseeName} mis à jour.`);
        }, className: 'bg-green-600 hover:bg-green-700 text-white' }
    ]);
};
    
const promptForAdminPassword = (callbackOnSuccess) => {
    const container = document.createElement('div');
    container.className = 'space-y-4';
    container.innerHTML = `
        <p>Veuillez saisir le mot de passe administrateur.</p>
        <input type="password" id="admin-password-input" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
        <p id="admin-password-error" class="text-red-500 text-sm hidden">Mot de passe incorrect.</p>
    `;

    const checkPassword = () => {
        const input = document.getElementById('admin-password-input');
        if (input.value === ADMIN_PASSWORD) {
            hideModal(dom.mainModal);
            callbackOnSuccess();
        } else {
            document.getElementById('admin-password-error').classList.remove('hidden');
            input.classList.add('border-red-500');
        }
    };

    showModal(dom.mainModal, 'Accès Administrateur', container, [
        { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-300' },
        { label: 'Confirmer', onClick: checkPassword, className: 'bg-green-600 hover:bg-green-700' }
    ]);
};

const showHistoryModal = () => {
    const displayHistoryList = () => {
        const history = JSON.parse(localStorage.getItem('documentHistory') || '[]');
        const historyContainer = document.createElement('div');
        historyContainer.className = 'space-y-2 max-h-64 overflow-y-auto';

        if (history.length === 0) {
            historyContainer.textContent = 'Aucun document dans l\'historique.';
        } else {
            history.sort((a, b) => new Date(b.orderDate) - new Date(a.orderDate)).forEach((docState, index) => {
                const itemDiv = document.createElement('div');
                itemDiv.className = 'flex justify-between items-center p-2 border rounded-md';
                itemDiv.innerHTML = `
                    <div>
                        <p class="font-semibold">${docState.documentType === 'devis' ? 'Devis' : 'Commande'} - ${docState.clubName}</p>
                        <p class="text-xs text-gray-500">Date: ${docState.orderDate}</p>
                    </div>
                    <div class="flex gap-2">
                        <button data-action="load-history" data-index="${index}" class="text-xs bg-blue-500 text-white px-2 py-1 rounded hover:bg-blue-600">Charger</button>
                        <button data-action="duplicate-history" data-index="${index}" class="text-xs bg-green-500 text-white px-2 py-1 rounded hover:bg-green-600">Dupliquer</button>
                        <button data-action="delete-history" data-index="${index}" class="text-xs bg-red-500 text-white px-2 py-1 rounded hover:bg-red-600">Suppr.</button>
                    </div>
                `;
                historyContainer.appendChild(itemDiv);
            });
        }
        
        const newModalBody = dom.mainModalBody.cloneNode(false);
        dom.mainModalBody.parentNode.replaceChild(newModalBody, dom.mainModalBody);
        dom.mainModalBody = newModalBody;

        dom.mainModalBody.addEventListener('click', (e) => {
            const target = e.target.closest('button');
            if (!target) return;
            const { action, index } = target.dataset;
            let history = JSON.parse(localStorage.getItem('documentHistory') || '[]');
            history.sort((a, b) => new Date(b.orderDate) - new Date(a.orderDate));
            const doc = history[index];

            if (action === 'load-history' && doc) {
                Object.assign(state, doc);
                renderAll();
                hideModal(dom.mainModal);
                showToast('Document chargé depuis l\'historique.');
            } else if (action === 'duplicate-history' && doc) {
                Object.assign(state, doc);
                state.orderDate = new Date().toISOString().split('T')[0];
                renderAll();
                hideModal(dom.mainModal);
                showToast('Document dupliqué. La date a été mise à jour.');
            } else if (action === 'delete-history' && doc) {
                history.splice(index, 1);
                localStorage.setItem('documentHistory', JSON.stringify(history));
                displayHistoryList();
            }
        });

        showModal(dom.mainModal, 'Historique des Documents', historyContainer, [
            { label: 'Fermer', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-500' }
        ]);
    };

    promptForAdminPassword(displayHistoryList);
};

const promptForForceCode = (callbackOnSuccess) => {
    hideModal(dom.mainModal); 
    const container = document.createElement('div');
    container.className = 'space-y-4';
    container.innerHTML = `
        <p>Pour forcer cette action, veuillez saisir le code d'autorisation.</p>
        <input type="password" id="force-code-input" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm">
        <p id="force-code-error" class="text-red-500 text-sm hidden">Code incorrect.</p>
    `;
    showModal(dom.mainModal, 'Code d\'Autorisation Requis', container, [
        { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-300' },
        { label: 'Confirmer', onClick: () => {
            if (document.getElementById('force-code-input').value === '582069') {
                hideModal(dom.mainModal);
                callbackOnSuccess();
            } else {
                document.getElementById('force-code-error').classList.remove('hidden');
                document.getElementById('force-code-input').classList.add('border-red-500');
            }
        }, className: 'bg-green-600 hover:bg-green-700' }
    ]);
};
    
const handleValidateOrder = () => {

    if (!state.clubName || !state.departement) {
        showToast('Le nom du club et le département sont obligatoires.', 'error');
        return;
    }
    if (state.currentOrderLineItems.length === 0) {
        showToast('Impossible de valider une commande vide.', 'error');
        return;
    }

    // On calcule le total des articles HORS accessoires pour les vérifications
    const totalNonAccessoryQuantity = state.currentOrderLineItems.reduce((sum, item) => {
        const product = productMap.get(item.productName);
        // On ajoute la quantité seulement si le produit existe et n'est pas un accessoire
        if (product && product.type !== 'accessoire') {
            return sum + item.totalQuantity;
        }
        return sum;
    }, 0);


    if (state.isReassort) {
        if (totalNonAccessoryQuantity < 10) {
            const p = document.createElement('p');
            p.innerHTML = `<b>Validation Réassort :</b> La commande doit contenir un minimum de 10 articles (hors accessoires).<br>Quantité actuelle : <b>${totalNonAccessoryQuantity}</b>.`;
            showModal(dom.mainModal, 'Validation Impossible (Réassort)', p, [
                { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-300' },
                { label: 'Forcer la validation', onClick: () => promptForForceCode(showFinalValidationModal), className: 'bg-orange-500 hover:bg-orange-600' }
            ]);
            return;
        }

        const itemQuantitiesByName = state.currentOrderLineItems
            .filter(item => productMap.get(item.productName)?.type !== 'accessoire')
            .reduce((acc, item) => {
                acc[item.productName] = (acc[item.productName] || 0) + item.totalQuantity;
                return acc;
            }, {});

        const failingItems = Object.entries(itemQuantitiesByName).filter(([, qty]) => qty < 2);
        if (failingItems.length > 0) {
            const p = document.createElement('p');
            p.innerHTML = '<b>Validation Réassort :</b> Les articles suivants ne respectent pas le minimum de 2 pièces par référence :<br><ul class="list-disc list-inside mt-2">' 
                + failingItems.map(([name, qty]) => `<li><b>${name}</b> (Qté: ${qty})</li>`).join('') + '</ul>';
            showModal(dom.mainModal, 'Validation Impossible (Réassort)', p, [
                { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-300' },
                { label: 'Forcer la validation', onClick: () => promptForForceCode(showFinalValidationModal), className: 'bg-orange-500 hover:bg-orange-600' }
            ]);
            return;
        }
    } else {
        // ▼▼▼ CONDITION MISE À JOUR ICI ▼▼▼
        if (totalNonAccessoryQuantity < 10) {
            const p = document.createElement('p');
            p.innerHTML = `<b>Validation Commande :</b> La commande doit contenir un minimum de 10 articles <strong>(hors accessoires)</strong>.<br>Quantité actuelle (hors accessoires) : <b>${totalNonAccessoryQuantity}</b>.`;
            showModal(dom.mainModal, 'Validation Impossible', p, [
                { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-300' },
                { label: 'Forcer la validation', onClick: () => promptForForceCode(showFinalValidationModal), className: 'bg-orange-500 hover:bg-orange-600' }
            ]);
            return;
        }
        // ▲▲▲ FIN DE LA MISE À JOUR ▲▲▲
    }

    const accessoryMinQuantityErrors = [];
    allAvailableProducts.filter(p => p.subtype === 'ACCESSOIRES PERSONNALISÉS' && p.minQuantity > 1).forEach(product => {
        const totalQuantityInCart = state.currentOrderLineItems
            .filter(item => item.productName === product.name)
            .reduce((sum, item) => sum + item.totalQuantity, 0);
        if (totalQuantityInCart > 0 && totalQuantityInCart < product.minQuantity) {
            accessoryMinQuantityErrors.push(`Le produit "${product.name}" requiert une quantité minimale de ${product.minQuantity} (actuellement: ${totalQuantityInCart}).`);
        }
    });

    if (accessoryMinQuantityErrors.length > 0) {
        const p = document.createElement('p');
        p.innerHTML = accessoryMinQuantityErrors.join('<br>');
        showModal(dom.mainModal, 'Erreur de Quantité Minimale', p);
        return;
    }
    
    showFinalValidationModal();
};
const showFinalValidationModal = () => {
    const p = document.createElement('p');
    p.innerHTML = `Êtes-vous sûr de vouloir finaliser cette commande ?<br><br>Après validation, le bon de commande PDF, le PDF détaillé et un fichier de sauvegarde seront générés.`;
    showModal(dom.mainModal, 'Confirmer la validation', p, [
        { label: 'Retourner aux modifications', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400 text-black' },
        { 
            label: 'Valider et Exporter', 
            onClick: () => {
                hideModal(dom.mainModal);
                saveClientInfo();
                
                const stockOrderItems = state.currentOrderLineItems.filter(item => item.isStockOrder);
                let stockWasModified = false;

                if (stockOrderItems.length > 0) {
                    stockWasModified = true;
                    stockOrderItems.forEach(item => {
                        for (const size in item.quantitiesPerSize) {
                            const qtyToAdd = parseInt(item.quantitiesPerSize[size], 10) || 0;
                            if (qtyToAdd > 0) {
                                if (!state.clubStock[item.productName]) {
                                    state.clubStock[item.productName] = {};
                                }
                                state.clubStock[item.productName][size] = (state.clubStock[item.productName][size] || 0) + qtyToAdd;
                            }
                        }
                    });
                    showToast(`${stockOrderItems.length} référence(s) ajoutée(s) au stock.`, "success");
                }
                
                saveStock(); 
                
                if (stockWasModified) {
                    handleExportStock();
                }
                
                const history = JSON.parse(localStorage.getItem('documentHistory') || '[]');
                history.push({ ...state, id: Date.now() });
                localStorage.setItem('documentHistory', JSON.stringify(history));

                handleExportPdf();
                handleExportDetailedPdf();
                // ▼▼▼ LIGNE AJOUTÉE POUR LA SAUVEGARDE AUTO ▼▼▼
                handleSaveOrderToFile();
                // ▲▲▲ FIN DE L'AJOUT ▲▲▲
                
                localStorage.removeItem('autosavedOrder');
            }, 
            className: 'bg-green-600 text-white'
        }
    ]);
};
const handleExportPdf = () => {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    const totals = calculateAllTotals();
    const pageHeight = doc.internal.pageSize.height;
    const rightMargin = doc.internal.pageSize.getWidth() - 14;

    const formatDateForDisplay = (dateString) => {
        if (!dateString) return 'N/A';
        const parts = dateString.split('-');
        if (parts.length !== 3) return dateString;
        return `${parts[2]}/${parts[1]}/${parts[0]}`;
    };

    const generatePdfContent = () => {
        let headerEndY = 0;
        
        doc.setFontSize(18);
        const docTitle = `Bon de commande ${state.isReassort ? '(Réassort 2 mois)' : ''}`;
        doc.text(docTitle, 14, 22);
        
        doc.setFontSize(11);
        doc.setTextColor(100);
        
        doc.text(`Nom du Club: ${state.clubName}`, 14, 32);
        doc.text(`Département: ${state.departement || 'N/A'}`, 14, 38);
        doc.text(`N° Client: ${state.clientNumber || 'N/A'}`, 14, 44);
        
        doc.text(`N° Précommande: ${state.preOrderNumber || 'N/A'}`, rightMargin, 32, { align: 'right' });
        doc.text(`Date Commande: ${formatDateForDisplay(state.orderDate)}`, rightMargin, 38, { align: 'right' });
        doc.text(`Départ Usine Prévu: ${formatDateForDisplay(state.factoryDepartureDate)}`, rightMargin, 44, { align: 'right' });
        
        headerEndY = 52;

        doc.setFont(undefined, 'bold');
        doc.text('Informations de Livraison', 14, headerEndY);
        doc.setFont(undefined, 'normal');
        const deliveryText = `Adresse: ${state.deliveryAddress || 'Non renseignée'}\nContact: ${state.deliveryContact || 'Non renseigné'}`;
        const splitText = doc.splitTextToSize(deliveryText, doc.internal.pageSize.width - 28);
        doc.text(splitText, 14, headerEndY + 6);
        headerEndY += (splitText.length * 6) + 4;

        const itemsForProduction = state.currentOrderLineItems.filter(item => !item.isFromStock);
        
        // ▼▼▼ NOUVELLE LOGIQUE DE REGROUPEMENT POUR LE PDF USINE ▼▼▼
        const consolidatedItems = itemsForProduction.reduce((acc, item) => {
            const key = `${item.productName}|${item.visual || ''}|${JSON.stringify(item.options)}|${item.specificity || ''}|${item.unitPriceTTC}`;
            
            if (!acc[key]) {
                acc[key] = {
                    ...item,
                    quantitiesPerSize: {},
                    totalQuantity: 0
                };
            }
            for (const size in item.quantitiesPerSize) {
                acc[key].quantitiesPerSize[size] = (acc[key].quantitiesPerSize[size] || 0) + item.quantitiesPerSize[size];
            }
            return acc;
        }, {});

        const finalBodyData = Object.values(consolidatedItems).map(item => {
            item.totalQuantity = Object.values(item.quantitiesPerSize).reduce((sum, q) => sum + q, 0);
            return item;
        });
        // ▲▲▲ FIN DE LA LOGIQUE DE REGROUPEMENT ▲▲▲

        const head = [['Produit', 'Tailles', 'Qté', 'Prix U. TTC', 'Total TTC']];

        const body = finalBodyData.map(item => {
            let productName = item.productName;
            if (item.visual) productName += `\nVisuel: ${item.visual}`;
            if (item.isStockOrder) productName = `[POUR STOCK] ${productName}`;
            if (item.options.length > 0) productName += `\nOptions: ${item.options.join(', ')}`;
            if (item.specificity) productName += `\nNote: ${item.specificity}`;
            
            const totalLinePrice = item.unitPriceTTC * item.totalQuantity;

            return [
                productName,
                getSortedSizesText(item),
                item.totalQuantity,
                `${item.unitPriceTTC.toFixed(2)} €`,
                `${totalLinePrice.toFixed(2)} €`
            ];
        });

        doc.autoTable({ 
            startY: headerEndY + 2, head, body, theme: 'striped', 
            headStyles: { fillColor: [41, 128, 185], textColor: 255 }, 
            styles: { cellPadding: 2, fontSize: 8, valign: 'middle' },
            columnStyles: { 
                0: { cellWidth: 80 }, 
                1: { cellWidth: 40 } 
            },
        });

        let finalY = doc.autoTable.previous.finalY + 10;
        const totalsX = 135;
        if (finalY > (pageHeight - 60)) { doc.addPage(); finalY = 20; }

        doc.setFontSize(10);
        doc.text(`Sous-total HT:`, totalsX, finalY); doc.text(`${totals.subtotalHT.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
        finalY += 6; doc.text(`Sous-total TTC:`, totalsX, finalY); doc.text(`${totals.subtotalTTC.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
        
        // La ligne de la remise club est maintenant supprimée de cet export
        
        finalY += 6; doc.text(`Frais de port TTC:`, totalsX, finalY); doc.text(`${totals.shippingTTC.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
        if (state.isCustomCreation) {
            finalY += 6; doc.text(`Forfait Création Graphique TTC:`, totalsX, finalY); doc.text(`${totals.graphicFeeTTC.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
        }
        finalY += 8; doc.setFontSize(12); doc.setFont(undefined, 'bold');
        doc.text(`Total Général TTC:`, totalsX, finalY); doc.text(`${totals.grandTotalTTC.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
        if(state.documentType === 'commande') {
            finalY += 8; doc.setTextColor(28, 175, 28);
            doc.text(`Acompte 30%:`, totalsX, finalY); doc.text(`${totals.downPayment.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
        }
        
        doc.save(`BON DE COMMANDE ${state.clubName} A TRANSMETTRE A NORET.pdf`);
    };
    
    generatePdfContent();
};const handleExportDetailedPdf = () => {
    if (state.orderScope !== 'licensee') {
        showToast("L'export détaillé par licencié n'est disponible qu'en mode de saisie 'Par licencié'.", 'error');
        return;
    }
    if (state.currentOrderLineItems.length === 0) {
        showToast("La commande est vide, rien à exporter.", "error");
        return;
    }
    
    const totals = calculateAllTotals();
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF({ orientation: 'landscape' });

    doc.setFontSize(16);
    doc.text(`Détail Financier par Licencié - ${state.clubName}`, 14, 15);
    doc.setFontSize(10);
    doc.text(`Date d'export: ${new Date().toLocaleDateString('fr-FR')}`, 14, 22);

    const groupedItems = state.currentOrderLineItems.reduce((acc, item) => {
        const key = item.licencieName || 'Commande Globale';
        if (!acc[key]) acc[key] = [];
        acc[key].push(item);
        return acc;
    }, {});

    const sortedLicensees = Object.keys(groupedItems).sort((a, b) => a.localeCompare(b));
    let startY = 30;

    sortedLicensees.forEach(licensee => {
        if (licensee === 'Commande Globale' || licensee === 'Stock Club') return; // On ignore les lignes non-nominatives

        const licenseeItems = groupedItems[licensee];
        let licenseeSubtotalTTC = 0;

        const head = [['Produit', 'Visuel', 'Spécificité', 'Options', 'Tailles & Qtés', 'Qté', 'P.U. TTC', 'Total TTC']];
        const body = licenseeItems.map(item => {
            let itemTotal = item.totalPriceTTC;
            if(state.applyDiscount && (state.discountType === 'global' || (state.discountType === 'item' && item.applyDiscount))){
                itemTotal *= (1 - (state.clubDiscount / 100));
            }
            licenseeSubtotalTTC += itemTotal;
            
            return [
                item.productName, item.visual || '', item.specificity || '',
                item.options.join(', '), getSortedSizesText(item), item.totalQuantity,
                `${item.unitPriceTTC.toFixed(2)} €`, `${item.totalPriceTTC.toFixed(2)} €`
            ];
        });

        if (startY + (body.length * 8) + 30 > doc.internal.pageSize.height) {
            doc.addPage();
            startY = 20;
        }

        doc.setFontSize(12);
        doc.setFont(undefined, 'bold');
        doc.text(licensee, 14, startY);

        doc.autoTable({
            startY: startY + 2, head: head, body: body, theme: 'grid',
            headStyles: { fillColor: [79, 70, 229], fontSize: 7 },
            styles: { fontSize: 7, cellPadding: 1.5, overflow: 'linebreak', valign: 'middle' },
            columnStyles: { 0: { cellWidth: 50 }, 4: { cellWidth: 30 }, 5: { cellWidth: 40 } }
        });

        startY = doc.autoTable.previous.finalY + 5;
        const deposit = state.licenseeDeposits[licensee] || 0;
        const remaining = licenseeSubtotalTTC - deposit;

        doc.setFontSize(9);
        doc.setFont(undefined, 'bold');
        doc.text(`Total: ${licenseeSubtotalTTC.toFixed(2)}€   |   Acompte Versé: ${deposit.toFixed(2)}€   |   Restant Dû: ${remaining.toFixed(2)}€`, 14, startY + 5);
        
        startY += 15;
    });

    let finalY = doc.autoTable.previous ? doc.autoTable.previous.finalY + 20 : startY;
    const pageHeight = doc.internal.pageSize.height;
    const rightMargin = doc.internal.pageSize.width - 14;
    const totalsX = 220;

    if (finalY > (pageHeight - 50)) {
        doc.addPage();
        finalY = 20;
    }

    doc.setFontSize(10);
    doc.text(`Sous-total HT:`, totalsX, finalY); doc.text(`${totals.subtotalHT.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
    finalY += 6; doc.text(`Sous-total TTC:`, totalsX, finalY); doc.text(`${totals.subtotalTTC.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
    if (state.applyDiscount) {
        finalY += 6; doc.setTextColor(255, 0, 0); doc.text(`Remise Club (${state.clubDiscount}%):`, totalsX, finalY); doc.text(`-${totals.discountAmountTTC.toFixed(2)} €`, rightMargin, finalY, { align: 'right' }); doc.setTextColor(0);
    }
    finalY += 6; doc.text(`Frais de port TTC:`, totalsX, finalY); doc.text(`${totals.shippingTTC.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
    if (state.isCustomCreation) {
        finalY += 6; doc.text(`Forfait Création Graphique TTC:`, totalsX, finalY); doc.text(`${totals.graphicFeeTTC.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
    }
    finalY += 8; doc.setFontSize(12); doc.setFont(undefined, 'bold');
    doc.text(`Total Général TTC:`, totalsX, finalY); doc.text(`${totals.grandTotalTTC.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
    if(state.documentType === 'commande') {
        finalY += 8; doc.setTextColor(28, 175, 28);
        doc.text(`Acompte 30%:`, totalsX, finalY); doc.text(`${totals.downPayment.toFixed(2)} €`, rightMargin, finalY, { align: 'right' });
    }

    doc.save(`detail-financier-licencies_${state.clubName.replace(/ /g, '_')}_${state.orderDate}.pdf`);
    showToast("Export détaillé par licencié (PDF) généré avec succès !", "success");
};const handleExportExcel = () => {
    if (state.currentOrderLineItems.length === 0 || typeof XLSX === 'undefined') return;

    const totals = calculateAllTotals();
    const totalArticles = state.currentOrderLineItems.reduce((acc, item) => acc + item.totalQuantity, 0);
    const docTitle = `${state.documentType === 'devis' ? 'Devis' : 'Bon de commande'} ${state.isReassort ? '(Réassort 2 mois)' : ''}`;
    
    const dataForSheet = [
        [docTitle], [], 
        ['Nom du Club:', state.clubName], ['Département:', state.departement], ['N° Client:', state.clientNumber], ['Date:', state.orderDate],
        ['Note Commande:', state.orderSpecificity], ['Total Articles:', totalArticles], []
    ];
    
    const excelHeader = ['Produit', 'Visuel', 'Licencié', 'Spécificité', 'Options', 'Tailles & Quantités', 'Qté Totale', 'Prix U. HT', 'Prix U. TTC', 'Total HT', 'Total TTC'];

    if (state.orderScope === 'licensee') {
        const groupedItems = state.currentOrderLineItems.reduce((acc, item) => {
            const key = item.licencieName || 'Commande Globale';
            if (!acc[key]) acc[key] = [];
            acc[key].push(item);
            return acc;
        }, {});
        const sortedLicensees = Object.keys(groupedItems).sort((a, b) => a.localeCompare(b));

        sortedLicensees.forEach(licensee => {
            dataForSheet.push([{ v: `Licencié: ${licensee}`, s: { font: { bold: true }, fill: { fgColor: { rgb: "E0E0E0" } } } }]);
            dataForSheet.push(excelHeader);
            let licenseeSubtotalTTC = 0;
            groupedItems[licensee].forEach(item => {
                let itemTotalTTC = item.totalPriceTTC;
                if (state.applyDiscount && (state.discountType === 'global' || (state.discountType === 'item' && item.applyDiscount))) {
                   itemTotalTTC *= (1 - (state.clubDiscount / 100));
                }
                licenseeSubtotalTTC += itemTotalTTC;
                let productName = item.productName;
                if (item.isFromStock) productName = `[STOCK] ${productName}`;
                if (item.isStockOrder) productName = `[POUR STOCK] ${productName}`;

                dataForSheet.push([
                    productName, item.visual || '', item.licencieName, item.specificity, item.options.join(', '),
                    getSortedSizesText(item).replace(/, /g, ' | '), item.totalQuantity,
                    { t: 'n', v: item.unitPriceHT, z: '#,##0.00 €' }, { t: 'n', v: item.unitPriceTTC, z: '#,##0.00 €' },
                    { t: 'n', v: item.totalPriceHT, z: '#,##0.00 €' }, { t: 'n', v: item.totalPriceTTC, z: '#,##0.00 €' }
                ]);
            });
            
            const deposit = state.licenseeDeposits[licensee] || 0;
            const remaining = licenseeSubtotalTTC - deposit;

            dataForSheet.push([]);
            dataForSheet.push(Array(excelHeader.length - 2).fill(null).concat([
                { v: "Total Articles Licencié:", s: { font: { bold: true } } }, 
                { t: 'n', v: licenseeSubtotalTTC, z: '#,##0.00 €', s: { font: { bold: true } } }
            ]));
            dataForSheet.push(Array(excelHeader.length - 2).fill(null).concat([
                { v: "Acompte Versé:", s: { font: { bold: true, color: { rgb: "008000" } } } }, 
                { t: 'n', v: deposit, z: '#,##0.00 €', s: { font: { bold: true, color: { rgb: "008000" } } } }
            ]));
            dataForSheet.push(Array(excelHeader.length - 2).fill(null).concat([
                { v: "Restant Dû:", s: { font: { bold: true, color: { rgb: "FF0000" } } } }, 
                { t: 'n', v: remaining, z: '#,##0.00 €', s: { font: { bold: true, color: { rgb: "FF0000" } } } }
            ]));
            dataForSheet.push([]);
        });
    } else {
        dataForSheet.push(excelHeader);
        state.currentOrderLineItems.forEach(item => {
            let productName = item.productName;
            if (item.isFromStock) productName = `[STOCK] ${productName}`;
            if (item.isStockOrder) productName = `[POUR STOCK] ${productName}`;
            dataForSheet.push([
                productName, item.visual || '', item.licencieName, item.specificity, item.options.join(', '),
                getSortedSizesText(item).replace(/, /g, ' | '), item.totalQuantity,
                { t: 'n', v: item.unitPriceHT, z: '#,##0.00 €' }, { t: 'n', v: item.unitPriceTTC, z: '#,##0.00 €' },
                { t: 'n', v: item.totalPriceHT, z: '#,##0.00 €' }, { t: 'n', v: item.totalPriceTTC, z: '#,##0.00 €' }
            ]);
        });
    }
    
    const totalsStartColumn = excelHeader.length - 2;
    dataForSheet.push([],
        Array(totalsStartColumn).fill(null).concat(['Sous-total HT:', { t: 'n', v: totals.subtotalHT, z: '#,##0.00 €' }]),
        Array(totalsStartColumn).fill(null).concat(['Sous-total TTC:', { t: 'n', v: totals.subtotalTTC, z: '#,##0.00 €' }]),
        Array(totalsStartColumn).fill(null).concat([`Remise Club (${state.clubDiscount}%) TTC (Info):`, { t: 'n', v: -totals.discountAmountTTC, z: '#,##0.00 €' }]),
        Array(totalsStartColumn).fill(null).concat(['Frais de port TTC:', { t: 'n', v: totals.shippingTTC, z: '#,##0.00 €' }]),
        ...(state.isCustomCreation ? [Array(totalsStartColumn).fill(null).concat(['Forfait Création Graphique TTC:', { t: 'n', v: totals.graphicFeeTTC, z: '#,##0.00 €' }])] : []),
        Array(totalsStartColumn).fill(null).concat([{v: "Total Général TTC:", s:{font:{bold:true}}}, { t: 'n', v: totals.grandTotalTTC, z: '#,##0.00 €', s:{font:{bold:true}}}])
    );
    if(state.documentType === 'commande') {
        dataForSheet.push(Array(totalsStartColumn).fill(null).concat([{v: "Acompte à verser (30%):", s:{font:{bold:true}}}, { t: 'n', v: totals.downPayment, z: '#,##0.00 €', s:{font:{bold:true}}}]));
    }
    
    const wb = XLSX.utils.book_new();
    const ws = XLSX.utils.aoa_to_sheet(dataForSheet);
    ws['!cols'] = excelHeader.map(h => ({wch: h.length > 20 ? 50 : 20}));
    XLSX.utils.book_append_sheet(wb, ws, 'Commande');
    XLSX.writeFile(wb, `detail-commande_${state.clubName.replace(/ /g, '_') || 'commande'}_${state.orderDate}.xlsx`);
};const handleExportDistributionList = () => {
    if (state.orderScope !== 'licensee') {
        showToast("La liste de distribution n'est disponible que pour les commandes par licencié.", 'error');
        return;
    }
    const licenseesWithItems = state.currentOrderLineItems.filter(item => item.licencieName && item.licencieName !== 'Commande Globale');
    if (licenseesWithItems.length === 0) {
        showToast("Aucun article n'a été assigné à un licencié.", 'error');
        return;
    }

    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();

    doc.setFontSize(20);
    doc.text(`Liste de Distribution - ${state.clubName}`, 14, 22);
    doc.setFontSize(12);
    doc.text(`Date d'export: ${new Date().toLocaleDateString('fr-FR')}`, 14, 30);

    const groupedItems = licenseesWithItems.reduce((acc, item) => {
        const key = item.licencieName;
        if (!acc[key]) acc[key] = [];
        acc[key].push(item);
        return acc;
    }, {});

    const sortedLicensees = Object.keys(groupedItems).sort((a, b) => a.localeCompare(b));
    let startY = 40;

    sortedLicensees.forEach(licensee => {
        const body = groupedItems[licensee].map(item => {
            let productName = item.productName;
            if (item.specificity) productName += `\n(${item.specificity})`;
            return [productName, item.totalQuantity, getSortedSizesText(item)];
        });
        
        const tableHeight = (body.length + 1) * 10 + 10;
        if (startY + tableHeight > doc.internal.pageSize.height - 20) {
            doc.addPage();
            startY = 20;
        }

        doc.setFontSize(16);
        doc.setFont(undefined, 'bold');
        doc.text(licensee, 14, startY);

        doc.autoTable({
            startY: startY + 5,
            head: [['Article', 'Qté Totale', 'Détail par Taille']],
            body: body,
            theme: 'grid',
            headStyles: { fillColor: [79, 70, 229] },
            styles: { fontSize: 9, cellPadding: 2 },
            columnStyles: { 0: { cellWidth: 80 } },
        });

        startY = doc.autoTable.previous.finalY + 15;
    });
    
    doc.save(`liste_distribution_${state.clubName.replace(/ /g, '_')}_${state.orderDate}.pdf`);
    showToast('PDF de distribution généré avec succès !');
};

const updateButtonStates = () => {
    const addProductBtn = document.getElementById('add-product-btn');
    if (addProductBtn) {
        const productSelected = !!state.currentProduct;
        let quantityEntered = false;
        const productData = productMap.get(state.currentProduct);
        if (productData) {
            if (state.documentType === 'devis' || (productData.type === 'accessoire' && (!productData.sizeType || productData.sizeType === 'unique'))) {
                quantityEntered = (parseInt(state.currentAccessoryQuantity, 10) || 0) > 0;
            } else {
                quantityEntered = Object.values(state.currentQuantities).some(q => (parseInt(q, 10) || 0) > 0);
            }
            if (state.isReassort && productData.type !== 'accessoire') {
                quantityEntered = quantityEntered && (parseFloat(state.manualUnitPrice) > 0);
            }
        }
        addProductBtn.disabled = !(productSelected && quantityEntered);
    }

    const isReadyForValidation = state.clubName.trim() && (state.documentType === 'devis' || state.departement.trim());
    dom.validateOrderBtn.disabled = !(isReadyForValidation && state.currentOrderLineItems.length > 0);

    if (dom.generatePortalLinkBtn) {
        const selectionMade = state.portalProductSelection.length > 0;
        const clubNameEntered = !!state.clubName.trim();
        dom.generatePortalLinkBtn.disabled = !(selectionMade && clubNameEntered);
        dom.generatePortalLinkBtn.title = !selectionMade ? "Veuillez d'abord sélectionner des articles." : !clubNameEntered ? "Veuillez d'abord renseigner le nom du club." : "";
    }
};
// =================================================================================
// --- PORTAL LOGIC ---
// =================================================================================

const getCurrentPortalSessionId = () => {
    if (!state.clubName || !state.orderDate) return null;
    const sanitizedClubName = state.clubName.replace(/[\s/\\?%*:|"<>]/g, '_');
    const sanitizedSessionName = state.portalSessionName.trim().replace(/[\s/\\?%*:|"<>]/g, '_');
    return sanitizedSessionName ? `portal_${sanitizedClubName}_${state.orderDate}_${sanitizedSessionName}` : `portal_${sanitizedClubName}_${state.orderDate}`;
};

const initializePortalView = (portalId) => {
    dom.mainAppView.classList.add('hidden');
    dom.portalView.classList.remove('hidden');

    const configStr = localStorage.getItem(`${portalId}-config`);
    if (!configStr) {
        dom.portalView.innerHTML = `<div class="text-center text-red-500">Erreur: Portail de commande non trouvé ou invalide.</div>`;
        return;
    }

    const config = JSON.parse(configStr);
    document.getElementById('portal-club-name').textContent = `Commande pour ${config.clubName || 'votre club'}`;
    const productListContainer = document.getElementById('portal-product-list');
    productListContainer.innerHTML = '';

    const today = new Date();
    today.setHours(0, 0, 0, 0); 
    const deadline = config.portalDeadline ? new Date(config.portalDeadline) : null;
    const isDeadlinePassed = deadline && today > deadline;

    const deadlineInfo = document.createElement('div');
    deadlineInfo.className = "mt-4 mb-4 text-center";
    if (isDeadlinePassed) {
        deadlineInfo.innerHTML = `<p class="font-bold text-red-600">La date butoir pour cette commande est passée (${new Date(deadline).toLocaleDateString('fr-FR')}).</p><p class="text-red-600">Il n'est plus possible de soumettre votre sélection.</p>`;
    } else if (deadline) {
        deadlineInfo.innerHTML = `<p class="font-bold text-blue-600">Date butoir pour passer votre commande : ${new Date(deadline).toLocaleDateString('fr-FR')}</p>`;
    }
    document.getElementById('portal-view').querySelector('.w-full.max-w-2xl').prepend(deadlineInfo);

    config.products.forEach(productName => {
        const product = productMap.get(productName);
        if (!product) return;
        const sizes = productSizes[product.sizeType || product.type] || [];
        if (sizes.length === 0) return;

        const productDiv = document.createElement('div');
        productDiv.className = "p-4 border rounded-lg";
        productDiv.innerHTML = `
            <h4 class="text-md font-semibold text-gray-800">${product.name}</h4>
            <div class="mt-2 space-y-2">${sizes.map(size => `
                <div class="flex items-center justify-between my-2">
                    <label for="portal-qty-${product.name}-${size}" class="text-sm text-gray-700">${size}</label>
                    <input type="number" id="portal-qty-${product.name}-${size}" min="0" placeholder="0" data-product-name="${product.name}" data-size="${size}" class="portal-quantity-input w-20 rounded-md border-gray-300 shadow-sm text-center">
                </div>`).join('')}
            </div>`;
        productListContainer.appendChild(productDiv);
    });

    document.getElementById('portal-submit-btn').onclick = () => handlePortalSubmit(portalId);
    if (isDeadlinePassed) {
        document.getElementById('portal-submit-btn').disabled = true;
    }
};

const handlePortalSubmit = (portalId) => {
    const licenseeName = document.getElementById('portal-licensee-name').value.trim();
    if (!licenseeName) { alert("Veuillez entrer votre nom complet."); return; }

    const choices = {};
    let hasSelection = false;
    document.querySelectorAll('.portal-quantity-input').forEach(input => {
        const qty = parseInt(input.value, 10) || 0;
        if (qty > 0) {
            const { productName, size } = input.dataset;
            if (!choices[productName]) choices[productName] = {};
            choices[productName][size] = qty;
            hasSelection = true;
        }
    });

    if (!hasSelection) { alert("Veuillez saisir une quantité pour au moins un article."); return; }

    const newSubmission = { licensee: licenseeName, choices };
    let submissions = JSON.parse(localStorage.getItem(`${portalId}-submissions`) || '[]');
    
    const existingIndex = submissions.findIndex(s => s.licensee.toLowerCase() === licenseeName.toLowerCase());
    if (existingIndex > -1) {
        if (confirm("Vous avez déjà soumis une sélection. Voulez-vous la remplacer ?")) {
            submissions[existingIndex] = newSubmission;
        } else { return; }
    } else {
        submissions.push(newSubmission);
    }

    localStorage.setItem(`${portalId}-submissions`, JSON.stringify(submissions));
    dom.portalView.innerHTML = `<div class="text-center"><h1 class="text-3xl font-bold text-green-600">Merci, ${licenseeName} !</h1><p class="mt-4 text-gray-600">Votre sélection a bien été enregistrée.</p></div>`;
};
const handleGeneratePortalLink = () => {
    if (state.portalProductSelection.length === 0) {
        showToast("Veuillez d'abord sélectionner des articles.", 'error');
        return;
    }
    if (!state.clubName) {
        showToast("Veuillez renseigner le nom du club.", 'error');
        return;
    }

    const portalId = getCurrentPortalSessionId();
    const portalConfig = { clubName: state.clubName, products: state.portalProductSelection, portalDeadline: state.portalDeadline };
    localStorage.setItem(`${portalId}-config`, JSON.stringify(portalConfig));
    const url = `${window.location.origin}${window.location.pathname}#${portalId}`;

    const content = document.createElement('div');
    content.className = 'space-y-4';
    content.innerHTML = `
        <p>Partagez ce lien avec les licenciés de votre club.</p>
        <input type="text" readonly value="${url}" id="portal-link-input" class="w-full p-2 border rounded bg-gray-100">
        <p class="text-xs text-gray-500">Ce lien est unique à cette session. Si vous modifiez les articles, générez un nouveau lien.</p>`;
    
    showModal(dom.mainModal, 'Lien du Portail de Commande', content, [
        {label: 'Fermer', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400'},
        {label: 'Copier le lien', onClick: () => {
            document.getElementById('portal-link-input').select();
            document.execCommand('copy');
            showToast('Lien copié !');
        }, className: 'bg-indigo-600'}
    ]);
};

const showPortalProductSelectorModal = () => {
    // Le contrôle que nous venons d'ajouter
    if (!state.clubName.trim() || !state.departement.trim()) {
        showToast("Veuillez d'abord renseigner le Nom du Club et le Département.", 'error');
        return;
    }

    const container = document.createElement('div');
    container.className = 'space-y-4';

    // Variable pour savoir si on affiche la gamme ou tout
    let showOnlyRange = state.clubProductRange.length > 0;

    const renderContent = () => {
        container.innerHTML = ''; // Vide le contenu pour le redessiner

        // Affiche les boutons de bascule seulement si une gamme existe
        if (state.clubProductRange.length > 0) {
            container.innerHTML += `
                <div class="flex justify-center gap-4 mb-4 p-2 bg-gray-100 rounded-lg">
                    <button id="show-range-btn" class="px-4 py-2 text-sm rounded-md ${showOnlyRange ? 'bg-indigo-600 text-white' : 'bg-white text-gray-700'}">Afficher la Gamme (${state.clubProductRange.length})</button>
                    <button id="show-all-btn" class="px-4 py-2 text-sm rounded-md ${!showOnlyRange ? 'bg-indigo-600 text-white' : 'bg-white text-gray-700'}">Afficher Tout</button>
                </div>
            `;
        }
        
        const sourceProducts = showOnlyRange ? allAvailableProducts.filter(p => state.clubProductRange.includes(p.name)) : allAvailableProducts;
        const productsToShow = sourceProducts.filter(p => p.category !== 'option');
        
        const grouped = productsToShow.reduce((acc, p) => {
            const groupKey = `${p.category} - ${p.subtype}`;
            acc[groupKey] = [...(acc[groupKey] || []), p];
            return acc;
        }, {});

        let contentHtml = '<div class="space-y-3 max-h-[50vh] overflow-y-auto pr-2">';
        for (const groupLabel in grouped) {
            contentHtml += `<div class="pt-2"><h4 class="font-bold text-gray-800 border-b pb-1 mb-2">${groupLabel}</h4>`;
            grouped[groupLabel].forEach(p => {
                const isChecked = state.portalProductSelection.includes(p.name);
                contentHtml += `<div class="flex items-center my-1">
                    <input id="portal-prod-${p.name}" type="checkbox" data-product-name="${p.name}" class="portal-product-checkbox h-4 w-4 rounded border-gray-300 text-indigo-600" ${isChecked ? 'checked' : ''}>
                    <label for="portal-prod-${p.name}" class="ml-3 block text-sm text-gray-900">${p.name}</label>
                </div>`;
            });
            contentHtml += `</div>`;
        }
        contentHtml += `</div>`;
        container.innerHTML += contentHtml;

        // Attache les écouteurs d'événements aux boutons s'ils existent
        const showRangeBtn = container.querySelector('#show-range-btn');
        if (showRangeBtn) {
            showRangeBtn.addEventListener('click', () => {
                showOnlyRange = true;
                renderContent();
            });
        }
        const showAllBtn = container.querySelector('#show-all-btn');
        if (showAllBtn) {
            showAllBtn.addEventListener('click', () => {
                showOnlyRange = false;
                renderContent();
            });
        }
    };
    
    renderContent(); // Premier affichage

    showModal(dom.mainModal, 'Sélectionner les Articles pour le Portail', container, [
        { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
        { label: 'Enregistrer la sélection', onClick: () => {
            // On s'assure de sauvegarder les coches même après avoir filtré
            const currentlyVisibleProducts = Array.from(container.querySelectorAll('.portal-product-checkbox')).map(cb => cb.dataset.productName);
            const checkedProducts = Array.from(container.querySelectorAll('.portal-product-checkbox:checked')).map(cb => cb.dataset.productName);
            
            // On met à jour la sélection en ne modifiant que les articles visibles
            let newSelection = state.portalProductSelection.filter(p => !currentlyVisibleProducts.includes(p));
            newSelection.push(...checkedProducts);
            state.portalProductSelection = [...new Set(newSelection)]; // Assure des valeurs uniques

            hideModal(dom.mainModal);
            showToast(`${state.portalProductSelection.length} article(s) sélectionné(s).`);
            updateButtonStates();
        }, className: 'bg-green-600' }
    ], 'max-w-2xl');
};
    
const handleImportFromPortal = (portalIdOverride = null) => {
    const portalId = portalIdOverride || getCurrentPortalSessionId();
    if (!portalId) {
        showToast("Veuillez renseigner le nom du club, la date et le nom de la session.", "error"); return;
    }
    const submissionsJSON = localStorage.getItem(`${portalId}-submissions`);
    if (!submissionsJSON || JSON.parse(submissionsJSON).length === 0) {
        showToast("Aucune commande de licencié trouvée pour cette session.", "error"); return;
    }

    const submissions = JSON.parse(submissionsJSON);
    const p = document.createElement('p');
    p.innerHTML = `Vous allez importer <b>${submissions.length}</b> soumission(s). Les commandes existantes pour ces licenciés seront remplacées. Continuer ?`;
    showModal(dom.mainModal, "Confirmer l'Importation", p, [
        { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
        { 
            label: 'Importer', 
            onClick: () => {
                let importedCount = 0;
                let unfoundProducts = new Set(); // Pour stocker les produits non trouvés

                submissions.forEach(sub => {
                    if (!state.licenseeList.includes(sub.licensee)) state.licenseeList.push(sub.licensee);
                    
                    Object.entries(sub.choices).forEach(([productName, quantitiesPerSize]) => {
                        const product = productMap.get(productName);
                        
                        // --- MODIFICATION PRINCIPALE ICI ---
                        if (!product) {
                            unfoundProducts.add(productName); // On ajoute le nom du produit non trouvé
                            return; // On passe à l'article suivant
                        }
                        // --- FIN DE LA MODIFICATION ---

                        const totalQuantity = Object.values(quantitiesPerSize).reduce((sum, qty) => sum + parseInt(qty, 10), 0);
                        if (totalQuantity === 0) return;

                        state.currentOrderLineItems = state.currentOrderLineItems.filter(item => !(item.licencieName === sub.licensee && item.productName === productName));
                        
                        state.currentOrderLineItems.push({
                            id: Date.now() + Math.random(),
                            productName,
                            type: product.type,
                            quantitiesPerSize,
                            totalQuantity,
                            options: [],
                            specificity: '',
                            applyDiscount: true,
                            licencieName: sub.licensee,
                            isManualPrice: false,
                            initialManualPrice: 0,
                            isFromStock: false,     // Propriété ajoutée pour la cohérence
                            isStockOrder: false,    // Propriété ajoutée pour la cohérence
                        });
                    });
                    importedCount++;
                });

                hideModal(dom.mainModal);
                renderAll();
                showToast(`${importedCount} sélections importées/mises à jour.`, 'success');

                // Affiche un avertissement si certains produits n'ont pas été trouvés
                if (unfoundProducts.size > 0) {
                    setTimeout(() => { // Petit délai pour ne pas superposer les toasts
                         showToast(`Avertissement : Les produits suivants n'ont pas été trouvés et n'ont pas été importés : ${[...unfoundProducts].join(', ')}`, 'error');
                    }, 500);
                }
            }, 
            className: 'bg-green-600' 
        }
    ]);
};
const showPortalSessionManagerModal = () => {
    const renderManagerContent = () => {
        const container = document.createElement('div');
        container.className = 'space-y-3';
        
        const sessions = [];
        for (let i = 0; i < localStorage.length; i++) {
            const key = localStorage.key(i);
            if (key.startsWith('portal_') && key.endsWith('-config')) {
                const keyPrefix = key.replace('-config', '');
                try {
                    const config = JSON.parse(localStorage.getItem(key));
                    const submissions = JSON.parse(localStorage.getItem(`${keyPrefix}-submissions`) || '[]');
                    const parts = keyPrefix.split('_');
                    sessions.push({
                        keyPrefix, clubName: config.clubName, orderDate: parts[2],
                        sessionName: parts.length > 3 ? parts.slice(3).join('_').replace(/-/g, ' ') : '(Sans nom)',
                        submissionCount: submissions.length
                    });
                } catch (e) { console.error(`Impossible de parser la session ${key}`, e); }
            }
        }

        if (sessions.length === 0) {
            container.innerHTML = `<p class="text-gray-500 text-center">Aucune session de portail n'est sauvegardée.</p>`;
            return container;
        }

        sessions.sort((a,b) => b.orderDate.localeCompare(a.orderDate)).forEach(session => {
            const itemDiv = document.createElement('div');
            itemDiv.className = 'flex justify-between items-center p-3 border rounded-lg';
            itemDiv.innerHTML = `
                <div>
                    <p class="font-bold text-gray-800">${session.clubName}</p>
                    <p class="text-sm text-gray-600">Session: ${session.sessionName}</p>
                    <p class="text-xs text-gray-500">Date: ${session.orderDate} | Commandes: ${session.submissionCount}</p>
                </div>
                <div class="flex items-center gap-2">
                    <button data-action="import-from-manager" data-key-prefix="${session.keyPrefix}" class="text-xs bg-blue-600 text-white px-3 py-1 rounded hover:bg-blue-700 ${session.submissionCount === 0 ? 'hidden' : ''}">Importer</button>
                    <button data-action="delete-session" data-key-prefix="${session.keyPrefix}" class="text-xs bg-red-500 text-white px-3 py-1 rounded hover:bg-red-600">Suppr.</button>
                </div>`;
            container.appendChild(itemDiv);
        });
        return container;
    };

    showModal(dom.mainModal, 'Gérer les Sessions de Portail', renderManagerContent(), [
        { label: 'Fermer', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-500' }
    ]);
    
    dom.mainModalBody.addEventListener('click', e => {
        const target = e.target.closest('button[data-action]');
        if (!target) return;
        
        const { action, keyPrefix } = target.dataset;

        if (action === 'delete-session') {
            if (confirm(`Supprimer définitivement cette session et toutes les commandes associées ?`)) {
                localStorage.removeItem(`${keyPrefix}-config`);
                localStorage.removeItem(`${keyPrefix}-submissions`);
                showToast('Session supprimée.', 'success');
                dom.mainModalBody.innerHTML = '';
                dom.mainModalBody.appendChild(renderManagerContent());
            }
        } else if (action === 'import-from-manager') {
            handleImportFromPortal(keyPrefix);
            hideModal(dom.mainModal);
        }
    });
};

// =================================================================================
// --- INITIALIZATION ---
// =================================================================================

const initializeAppView = () => {
    dom.orderDate.value = new Date().toISOString().split('T')[0];
    const savedClients = localStorage.getItem('clientDatabase');
    if (savedClients) clientDatabase = JSON.parse(savedClients);
    updateDatalists();
    
    const autosavedOrder = localStorage.getItem('autosavedOrder');
    if(autosavedOrder) {
        const p = document.createElement('p');
        p.textContent = 'Une commande non finalisée a été trouvée. Voulez-vous la restaurer ?';
        showModal(dom.mainModal, 'Restaurer la session ?', p, [
            {label: 'Non, commencer une nouvelle', onClick: () => { localStorage.removeItem('autosavedOrder'); hideModal(dom.mainModal); }, className: 'bg-red-600'},
            {label: 'Oui, restaurer', onClick: () => { Object.assign(state, JSON.parse(autosavedOrder)); renderAll(); hideModal(dom.mainModal); }, className: 'bg-green-600'}
        ]);
    }
    
    setInterval(() => {
        if(state.currentOrderLineItems.length > 0 || Object.keys(state.clubStock).length > 0) {
            dom.autosaveStatus.textContent = 'Sauvegarde en cours...';
            dom.autosaveStatus.classList.remove('text-gray-400');
            dom.autosaveStatus.classList.add('text-green-600');
            
            localStorage.setItem('autosavedOrder', JSON.stringify(state));

            setTimeout(() => {
                const now = new Date();
                const timeString = now.toLocaleTimeString('fr-FR');
                dom.autosaveStatus.textContent = `Dernière sauvegarde automatique : ${timeString}`;
                dom.autosaveStatus.classList.remove('text-green-600');
                dom.autosaveStatus.classList.add('text-gray-400');
            }, 500);
        }
    }, 30000);

    renderAll();
};

const init = () => {
    const portalId = window.location.hash.substring(1);
    if (portalId.startsWith('portal_')) {
        initializePortalView(portalId);
    } else {
        initializeAppView();
    }
    dom.portalSessionName.addEventListener('focus', () => {
        // On vérifie si l'info n'a PAS encore été montrée
        if (!state.portalInfoShown) {
            const content = document.createElement('div');
            content.innerHTML = `
                <h4 class="font-bold text-lg mb-2">Fonctionnement du Portail Licenciés</h4>
                <p class="text-sm">Le portail permet à chaque licencié de saisir sa propre commande via un lien unique que vous générez.</p>
                <ul class="list-disc list-inside mt-3 space-y-2 text-sm">
                    <li><strong>Nom de la session (Optionnel) :</strong> Donnez un nom (ex: "Commande Hiver 2024") pour retrouver facilement cette session plus tard.</li>
                    <li><strong>1. Sélectionner les articles :</strong> Choisissez les produits que vous souhaitez rendre disponibles pour les licenciés.</li>
                    <li><strong>2. Inviter les licenciés :</strong> Génère un lien web sécurisé. Partagez ce lien avec vos licenciés.</li>
                    <li><strong>3. Importer les commandes :</strong> Une fois la date butoir passée, cliquez ici pour tout ajouter à votre bon de commande principal.</li>
                </ul>
            `;
            showModal(dom.mainModal, "Info : Portail Licenciés", content, [
                { label: "J'ai compris", onClick: () => {
                    hideModal(dom.mainModal);
                    dom.portalSessionName.focus(); // Remet le focus sur le champ de saisie
                } }
            ], 'max-w-xl');
            
            // On mémorise que l'info a été montrée
            state.portalInfoShown = true; 
        }
    });
    dom.orderSpecificity.addEventListener('focus', () => {
        // On vérifie dans la mémoire du navigateur si l'info n'a pas déjà été montrée
        if (!localStorage.getItem('specificityInfoShown')) {
            const content = document.createElement('p');
            content.className = 'text-sm';
            content.textContent = "Dans ce champ, vous pouvez ajouter des informations complémentaires utiles pour le traitement de votre commande (par exemple : une demande de livraison spécifique, des détails pour la facturation, des contraintes particulières, etc.).";
            
            showModal(dom.mainModal, "Information : Spécificité Commande", content, [
                { 
                    label: "J'ai compris", 
                    onClick: () => {
                        hideModal(dom.mainModal);
                        // On remet le focus sur le champ pour que l'utilisateur puisse écrire
                        dom.orderSpecificity.focus();
                    } 
                }
            ]);
            
            // On enregistre de manière permanente que l'info a été montrée
            localStorage.setItem('specificityInfoShown', 'true'); 
        }
    });
    document.body.addEventListener('input', (e) => {
        const { id, value } = e.target;
        if (id === 'licencieName') { 
            state.licencieName = value; 
            if (state.activeLicensee && value.trim() !== state.activeLicensee) { 
                state.activeLicensee = ''; 
                renderUIState(); 
            }
            scrollToLicensee(value);
        }
        if (id === 'clubName') {
            state.clubName = value;
            if (value.trim() === '') {
                dom.departement.value = state.departement = '';
                dom.clientNumber.value = state.clientNumber = '';
                state.clubProductRange = [];
                renderAll();
            } else {
                const found = clientDatabase.find(c => c.clubName === value);
                if (found) {
                    dom.clubName.value = state.clubName = found.clubName;
                    dom.departement.value = state.departement = found.departement || '';
                    dom.clientNumber.value = state.clientNumber = found.clientNumber || '';
                    const rangeKey = `range_${state.clubName.replace(/[\s/\\?%*:|"<>]/g, '_')}`;
                    const savedRange = localStorage.getItem(rangeKey);
                    state.clubProductRange = savedRange ? JSON.parse(savedRange) : [];
                    state.showAllProducts = false;
                    renderProductForm();
                }
                renderAll();
            }
        }
        if (id === 'portalSessionName') {
            state.portalSessionName = value;
        }
        if (id === 'clientNumber') {
            state.clientNumber = value;
        }
    
        if (id === 'preOrderNumber') state.preOrderNumber = value;
        if (id === 'deliveryAddress') state.deliveryAddress = value;
        if (id === 'deliveryContact') state.deliveryContact = value;
        if (id === 'orderSpecificity') state.orderSpecificity = value;

        updateButtonStates();
    });

    document.body.addEventListener('focusout', (e) => {
        if (['clubName', 'clientNumber', 'departement'].includes(e.target.id)) saveClientInfo();
    });

    document.body.addEventListener('change', (e) => {
        const { id, name, value, checked, classList, dataset } = e.target;
        if (id === 'departement') { state.departement = value; renderAll(); }
        if (id === 'orderDate') { state.orderDate = value; renderAll(); }
        if (id === 'clubDiscount') { state.clubDiscount = parseFloat(value) || 0; renderAll(); }
        if (id === 'portalDeadline') state.portalDeadline = value;
        if (id === 'factoryDepartureDate') state.factoryDepartureDate = value;

        if (id === 'doc-type-reassort') {
            if (checked) {
                showReassortInfoModal();
            } else {
                state.isReassort = false;
                state.lastDeliveryDate = '';
                renderAll();
            }
        }
        if (id === 'custom-creation-check') {
            state.isCustomCreation = checked;
            renderAll();

            if (checked) {
                const content = document.createElement('div');
                content.innerHTML = `
                    <p class="text-sm">Vous avez sélectionné "Création Personnalisée". Les règles suivantes s'appliquent :</p>
                    <ul class="list-disc list-inside mt-3 space-y-2 text-sm">
                        <li>Applicable pour les commandes nécessitant une nouvelle création de maquette.</li>
                        <li>Un minimum de commande de <strong>10 pièces</strong> (hors accessoires) est requis.</li>
                        <li>Le <strong>forfait de création graphique</strong> s'applique automatiquement pour les commandes de <strong>moins de 20 pièces</strong> (hors accessoires).</li>
                    </ul>
                `;
                showModal(dom.mainModal, "Information : Commande Création Personnalisée", content, [
                    { label: "J'ai compris", onClick: () => hideModal(dom.mainModal) }
                ]);
            }
        }
        if (name === 'scope') {
            state.orderScope = value;
            renderAll();

            if (value === 'licensee') {
                const content = document.createElement('div');
                content.innerHTML = `
                    <p class="text-sm">Vous avez activé la <strong>saisie par licencié</strong>.</p>
                    <p class="text-sm mt-2">Utilisez le champ "Nom du licencié" pour ajouter des articles individuellement.</p>
                `;
                showModal(dom.mainModal, "Information : Saisie par Licencié", content, [
                    { label: "J'ai compris", onClick: () => hideModal(dom.mainModal) }
                ]);
            } else if (value === 'global') {
                const content = document.createElement('div');
                content.innerHTML = `<p class="text-sm">Vous avez sélectionné le mode de <strong>saisie globale</strong>.</p>
                                     <p class="text-sm mt-2">Tous les articles sont ajoutés à une seule et même commande pour l'ensemble du club.</p>`;
                showModal(dom.mainModal, "Information : Saisie Globale", content, [
                    { label: "J'ai compris", onClick: () => hideModal(dom.mainModal) }
                ]);
            } else if (value === 'session') {
                const content = document.createElement('div');
                content.innerHTML = `<p class="text-sm">Vous avez sélectionné le mode <strong>Session Licenciés</strong>.</p>
                                     <p class="text-sm mt-2">Ce mode utilise le portail en ligne pour permettre à chaque licencié de saisir sa propre commande. Utilisez les boutons de la section "Portail Licenciés" pour continuer.</p>`;
                showModal(dom.mainModal, "Information : Session Licenciés", content, [
                    { label: "J'ai compris", onClick: () => hideModal(dom.mainModal) }
                ]);
            }
        }
        if (id === 'store-order-check') state.isStoreOrder = checked;
        if (id === 'apply-discount-check') { state.applyDiscount = checked; if (!checked) state.clubDiscount = 0; renderAll(); }
        if (name === 'discount-type') { state.discountType = value; renderAll(); }
        if (id === 'current-product-select') {
            resetProductForm();
            state.currentProduct = value;
            calculateCurrentFormPrice();
            renderProductForm();
            setTimeout(() => {
                const productFormDetails = document.getElementById('product-details-form');
                if (productFormDetails) {
                    productFormDetails.scrollIntoView({ behavior: 'smooth', block: 'start' });
                }
            }, 50);
        }
        if (classList.contains('size-input') || id === 'accessory-qty') {
            if (classList.contains('size-input')) {
                state.currentQuantities[dataset.size] = value;
            } else {
                state.currentAccessoryQuantity = value;
            }
            calculateCurrentFormPrice();
            renderProductForm();
            setTimeout(() => {
                const addButton = document.getElementById('add-product-btn');
                if (addButton) {
                    addButton.scrollIntoView({ behavior: 'smooth', block: 'center' });
                }
            }, 50);
        }
        if (id === 'manual-price') { state.manualUnitPrice = value; updateButtonStates(); }
        if (id === 'current-color-select') state.currentColor = value;
        if (id === 'current-visual-select') {
            state.currentVisual = value;
        }
        if (id === 'specificity') state.currentSpecificity = value;
        if (id === 'add-to-stock-check') { state.isAddingForStock = checked; }
        if (classList.contains('option-checkbox')) {
            const { optionName, optionGroup } = dataset;
            if (optionGroup === 'length') {
                const allLengthOptions = allAvailableProducts.filter(p => p.optionGroup === 'length').map(p => p.name);
                const nonLengthOptions = state.currentSelectedOptions.filter(opt => !allLengthOptions.includes(opt));
                state.currentSelectedOptions = checked ? [...nonLengthOptions, optionName] : nonLengthOptions;
            } else {
                state.currentSelectedOptions = checked ? [...state.currentSelectedOptions, optionName] : state.currentSelectedOptions.filter(o => o !== optionName);
            }
            calculateCurrentFormPrice();
            renderProductForm();
        }
    });

document.body.addEventListener('click', (e) => {
    const target = e.target;
    const actionTarget = target.closest('[data-action]');
    
    // Logique pour le tableau de bord interactif
    const quantityCard = target.closest('.quantity-card');
    if (quantityCard) {
        const subtype = quantityCard.dataset.subtype;
        const relevantItems = state.currentOrderLineItems.filter(item => {
            const product = productMap.get(item.productName);
            return product && product.subtype === subtype;
        });

        const totalsByName = relevantItems.reduce((acc, item) => {
            acc[item.productName] = (acc[item.productName] || 0) + item.totalQuantity;
            return acc;
        }, {});

        const sortedNames = Object.keys(totalsByName).sort();
        const content = document.createElement('div');
        let tableHtml = `<table class="min-w-full text-left text-sm">
                            <thead class="border-b">
                                <tr>
                                    <th class="font-semibold p-2">Article (cliquable)</th>
                                    <th class="font-semibold p-2 text-right">Quantité</th>
                                </tr>
                            </thead>
                            <tbody>`;
        
        sortedNames.forEach(name => {
            tableHtml += `<tr class="border-b hover:bg-indigo-50 cursor-pointer clickable-article-row" data-product-name="${name}">
                            <td class="p-2">${name}</td>
                            <td class="p-2 text-right font-bold">${totalsByName[name]}</td>
                          </tr>`;
        });
        
        tableHtml += `</tbody></table>`;
        content.innerHTML = tableHtml;

        // Nouvelle façon d'appeler showModal avec la logique de clic en callback
        const onModalOpen = () => {
            dom.mainModalBody.querySelectorAll('.clickable-article-row').forEach(row => {
                row.addEventListener('click', () => {
                    const productName = row.dataset.productName;
                    const itemsForThisProduct = state.currentOrderLineItems.filter(item => item.productName === productName);

                    const licensees = itemsForThisProduct.reduce((acc, item) => {
                        const licensee = item.licencieName;
                        if (licensee && licensee !== 'Commande Globale' && licensee !== 'Stock Club') {
                            acc[licensee] = (acc[licensee] || 0) + item.totalQuantity;
                        }
                        return acc;
                    }, {});

                    const sortedLicensees = Object.keys(licensees).sort();
                    const licenseeContent = document.createElement('div');

                    if (sortedLicensees.length > 0) {
                        let licenseeTableHtml = `<table class="min-w-full text-left text-sm">...</table>`; // Le contenu du tableau est construit ici
                        licenseeTableHtml = `<table class="min-w-full text-left text-sm">
                                            <thead class="border-b">
                                                <tr>
                                                    <th class="font-semibold p-2">Licencié</th>
                                                    <th class="font-semibold p-2 text-right">Quantité</th>
                                                </tr>
                                            </thead>
                                            <tbody>`;
                        sortedLicensees.forEach(name => {
                            licenseeTableHtml += `<tr class="border-b">
                                            <td class="p-2">${name}</td>
                                            <td class="p-2 text-right font-bold">${licensees[name]}</td>
                                          </tr>`;
                        });
                        licenseeTableHtml += `</tbody></table>`;
                        licenseeContent.innerHTML = licenseeTableHtml;
                    } else {
                        licenseeContent.innerHTML = `<p class="text-center text-gray-500 p-4">Cet article a été commandé en mode global ou pour le stock (aucun licencié spécifique).</p>`;
                    }

                    showModal(dom.mainModal, `Licenciés pour : ${productName}`, licenseeContent, [
                         { label: "Fermer", onClick: () => hideModal(dom.mainModal) }
                    ]);
                });
            });
        };

        showModal(dom.mainModal, `Détail pour : ${subtype}`, content, [], 'max-w-md', onModalOpen);
        return;
    }

    // Le reste de la fonction de clic reste identique
    if (target.id === 'import-order-label') {
        e.preventDefault(); 
        const content = document.createElement('div');
        content.innerHTML = `<p class="text-sm">Cette fonction vous permet de <strong>charger un fichier <code>.json</code></strong> contenant toutes les informations d'un club (détails de la commande, stock, liste des licenciés, etc.).</p>
                             <p class="text-sm mt-2 text-red-600 font-bold">Attention, cela remplacera toutes les informations actuellement saisies à l'écran.</p>`;
        showModal(dom.mainModal, "Information : Importer un Fichier", content, [
            { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
            { label: 'Continuer', onClick: () => {
                hideModal(dom.mainModal);
                dom.loadOrderInput.click(); 
            }, className: 'bg-gray-600' }
        ]);
        return; 
    }

    if (target.id === 'add-product-btn' || target.parentElement?.id === 'add-product-btn') handleProductAdd();
    else if (target.id === 'clear-active-licensee-btn') { state.activeLicensee = ''; renderAll(); }
    else if (target.id === 'reset-product-form-btn') resetProductForm();
    else if (actionTarget) {
        const { action, itemId, licenseeName } = actionTarget.dataset;
        if (action === 'remove-item') {
            const itemIndex = state.currentOrderLineItems.findIndex(item => item.id == itemId);
            if (itemIndex > -1) {
                const itemToRemove = state.currentOrderLineItems[itemIndex];
                if (itemToRemove.isFromStock) {
                    for (const size in itemToRemove.quantitiesPerSize) {
                        const qtyToRestore = itemToRemove.quantitiesPerSize[size];
                        state.clubStock[itemToRemove.productName][size] = (state.clubStock[itemToRemove.productName][size] || 0) + qtyToRestore;
                    }
                }
                state.currentOrderLineItems.splice(itemIndex, 1);
                renderAll();
                showToast('Article supprimé.', 'error');
                updateSectionHighlights();
            }
        }
        if (action === 'toggle-discount') { 
            state.currentOrderLineItems = state.currentOrderLineItems.map(item => item.id == itemId ? { ...item, applyDiscount: !item.applyDiscount } : item); 
            renderAll(); 
        }
        if (action === 'edit-item') showEditItemModal(itemId);
        if (action === 'add-for-licensee') {
            state.activeLicensee = licenseeName;
            state.licencieName = '';
            renderAll();
            setTimeout(() => {
                document.getElementById('add-article-section').scrollIntoView({ behavior: 'smooth', block: 'start' });
            }, 50);
            scrollToLicensee(licenseeName);
        }
        if (action === 'manage-deposit') showDepositModal(licenseeName);
        if (action === 'return-to-input') {
            dom.licencieNameInput.scrollIntoView({ behavior: 'smooth', block: 'center' });
            dom.licencieNameInput.focus();
        }
    }
    else if (target.closest('.product-tab-btn')) {
        document.querySelectorAll('.product-tab-btn').forEach(btn => btn.classList.remove('border-indigo-500', 'text-indigo-600'));
        target.closest('.product-tab-btn').classList.add('border-indigo-500', 'text-indigo-600');
        resetProductForm();
    }
    else if (target.id === 'new-order-btn') handleNewOrder();
    else if (target.id === 'save-order-btn') {
        const content = document.createElement('div');
        content.innerHTML = `<p class="text-sm">Cette fonction vous permet de <strong>sauvegarder l'intégralité de votre travail</strong> actuel dans un fichier <code>.json</code>.</p>
                             <p class="text-sm mt-2">Ce fichier contient toutes les informations (détails du club, articles, stock, etc.) et peut être ré-importé plus tard pour continuer votre travail ou archiver la commande.</p>`;
        showModal(dom.mainModal, "Information : Exporter Fichier (.json)", content, [
            { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
            { label: 'Continuer', onClick: () => {
                hideModal(dom.mainModal);
                handleSaveOrderToFile();
            }, className: 'bg-blue-600' }
        ]);
    }
    else if (target.id === 'export-distribution-btn') {
        const content = document.createElement('div');
        content.innerHTML = `<p class="text-sm">Cette fonction génère un document <strong>PDF récapitulatif</strong>, idéal pour la distribution des articles à leur arrivée.</p>
                             <p class="text-sm mt-2">Le PDF liste <strong>chaque licencié</strong> avec les articles et les tailles qu'il a commandés, vous permettant de préparer facilement les paquets individuels.</p>
                             <p class="text-sm mt-3 pt-2 border-t border-gray-200"><strong>Note :</strong> Ce bouton ne fonctionne que si vous avez saisi la commande en mode <strong>'Par licencié'</strong>.</p>`;
        showModal(dom.mainModal, "Information : Liste de Distribution (PDF)", content, [
            { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
            { label: 'Continuer', onClick: () => {
                hideModal(dom.mainModal);
                handleExportDistributionList();
            }, className: 'bg-teal-600' }
        ]);
    }
    else if (target.id === 'manage-clients-btn') promptForAdminPassword(showClientManagerModal);
    else if (target.id === 'manage-licensees-btn') showLicenseeManagerModal();
    else if (target.id === 'next-licensee-btn') handleNextLicensee();
    else if (target.id === 'validate-order-btn') {
         const content = document.createElement('div');
        content.innerHTML = `<p class="text-sm">Ceci est la dernière étape. En cliquant sur 'Continuer', l'application vérifiera si votre commande respecte les conditions (quantités minimales, etc.).</p>
                             <p class="text-sm mt-2">Si tout est correct, <strong>les fichiers PDF et le fichier de sauvegarde seront générés</strong>.</p>
                             <p class="text-sm mt-2">Assurez-vous que toutes les informations sont correctes avant de continuer.</p>`;
        showModal(dom.mainModal, "Information : Valider la Commande", content, [
            { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
            { label: 'Continuer', onClick: () => {
                hideModal(dom.mainModal);
                handleValidateOrder();
            }, className: 'bg-indigo-600' }
        ]);
    }
    else if (target.closest('#main-modal-close-btn')) hideModal(dom.mainModal);
    else if (target.id === 'export-licensees-btn') handleExportLicensees();
    else if (target.id === 'manage-club-range-btn') {
        const content = document.createElement('div');
        content.innerHTML = `<p class="text-sm">Cette fonctionnalité vous permet de définir une <strong>sélection de produits prédéfinis</strong> pour le club actuel.</p>
                             <p class="text-sm mt-2">Une fois la gamme enregistrée, la liste des articles à ajouter sera <strong>filtrée par défaut</strong> pour ne montrer que ces produits, accélérant ainsi la saisie. Vous pourrez toujours choisir d'afficher tous les articles si nécessaire.</p>
                             <p class="text-sm mt-2">La gamme est sauvegardée automatiquement pour ce club.</p>`;
        showModal(dom.mainModal, "Information : Gérer la Gamme du Club", content, [
            { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
            { label: 'Continuer', onClick: () => {
                hideModal(dom.mainModal);
                setTimeout(() => showClubRangeSelectorModal(), 50);
            }, className: 'bg-slate-600' }
        ]);
    }
    else if (target.id === 'toggle-products-btn') {
        state.showAllProducts = !state.showAllProducts;
        renderProductForm();
    }
    else if (target.id === 'init-stock-btn') {
        if (!state.clubName) {
            showToast("Veuillez d'abord renseigner un nom de club.", 'error');
            return;
        }
        const content = document.createElement('div');
        content.innerHTML = `<p class="text-sm">Cette section vous permet de <strong>saisir pour la première fois</strong> les quantités en stock pour chaque article du club.</p>
                             <p class="text-sm mt-2">Utilisez cette fonction lorsque vous configurez un nouveau club ou après une réinitialisation. Pour des modifications ultérieures, utilisez le bouton "Gérer le stock".</p>`;
        showModal(dom.mainModal, "Information : Initialiser le Stock", content, [
            { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
            { label: 'Continuer', onClick: () => {
                hideModal(dom.mainModal);
                setTimeout(() => showStockManagerModal(), 50);
            }, className: 'bg-orange-500' }
        ]);
    }
    else if (target.id === 'manage-stock-btn') {
        if (!state.clubName) {
            showToast("Veuillez d'abord renseigner un nom de club.", 'error');
            return;
        }
        if (Object.keys(state.clubStock).length === 0) {
            showToast("Aucun stock initialisé pour ce club. Veuillez d'abord utiliser le bouton 'Initialiser le Stock'.", 'error');
            return;
        }
        const content = document.createElement('div');
        content.innerHTML = `<p class="text-sm">Cette section vous permet de <strong>modifier les quantités en stock</strong> actuelles, d'exporter votre inventaire ou d'importer un fichier de stock.</p>
                             <p class="text-sm mt-2">Les articles ajoutés à une commande seront automatiquement déduits du stock ici.</p>`;
        showModal(dom.mainModal, "Information : Gérer le Stock", content, [
            { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
            { label: 'Continuer', onClick: () => {
                hideModal(dom.mainModal);
                setTimeout(() => showStockManagerModal(), 50);
            }, className: 'bg-green-600' }
        ]);
    }
    else if (target.id === 'session-manager-btn') {
        const content = document.createElement('div');
        content.innerHTML = `<p class="text-sm">Cette section vous permet de <strong>visualiser toutes les sessions de commande par portail</strong> que vous avez créées.</p>
                             <p class="text-sm mt-2">Vous pouvez importer les commandes des licenciés depuis une session passée ou supprimer les anciennes sessions dont vous n'avez plus besoin.</p>`;
        showModal(dom.mainModal, "Information : Gérer les Sessions", content, [
            { label: 'Annuler', onClick: () => hideModal(dom.mainModal), className: 'bg-gray-400' },
            { label: 'Continuer', onClick: () => {
                hideModal(dom.mainModal);
                setTimeout(() => showPortalSessionManagerModal(), 50);
            }, className: 'bg-purple-600' }
        ]);
    }
    else if (target.id === 'select-portal-products-btn') {
        if (state.portalSessionName.trim() === '') {
            showToast("Veuillez d'abord saisir un nom pour la session.", 'error');
        } else {
            showPortalProductSelectorModal();
        }
    }
    else if (target.id === 'generate-portal-link-btn') handleGeneratePortalLink();
    else if (target.id === 'import-portal-submissions-btn') handleImportFromPortal();
});    

dom.loadOrderInput.addEventListener('change', handleLoadOrderFromFile);
    dom.importLicenseesInput.addEventListener('change', handleImportLicensees);
    dom.importStockInput.addEventListener('change', handleImportStock);
    document.getElementById('import-club-range-input').addEventListener('change', handleImportClubRange);
    
    window.addEventListener('beforeunload', (event) => {
        if (state.currentOrderLineItems.length > 0) {
            event.preventDefault();
            event.returnValue = '';
        }
    });

    // ▼▼▼ GESTION DES BOUTONS DE DÉFILEMENT (ÉTAPE 2) ▼▼▼
    const scrollToTopBtn = document.getElementById('scroll-to-top-btn');
    const scrollToBottomBtn = document.getElementById('scroll-to-bottom-btn');
    const summarySection = document.getElementById('summary-and-actions-section');

    scrollToBottomBtn.addEventListener('click', () => {
        summarySection.scrollIntoView({ behavior: 'smooth' });
    });

    scrollToTopBtn.addEventListener('click', () => {
        window.scrollTo({ top: 0, behavior: 'smooth' });
    });

    window.addEventListener('scroll', () => {
        if (window.scrollY > 300) {
            scrollToTopBtn.style.display = 'flex';
        } else {
            scrollToTopBtn.style.display = 'none';
        }
    });
    // ▲▲▲ FIN DE LA GESTION DES BOUTONS ▲▲▲
// ▼▼▼ GESTION DU CLIC SUR LE PANIER FLOTTANT ▼▼▼
    document.getElementById('floating-cart').addEventListener('click', () => {
        document.getElementById('summary-and-actions-section').scrollIntoView({ behavior: 'smooth' });
    });
    // ▲▲▲ FIN DE LA GESTION DU CLIC ▲▲▲
};    
document.addEventListener('DOMContentLoaded', init);
</script>
<div class="fixed bottom-5 right-5 z-50 flex flex-col gap-2">
    <button id="scroll-to-bottom-btn" title="Aller aux totaux" class="h-12 w-12 rounded-full bg-indigo-600 text-white shadow-lg hover:bg-indigo-700 flex items-center justify-center">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M19 13l-7 7-7-7m14-8l-7 7-7-7" />
        </svg>
    </button>
    <button id="scroll-to-top-btn" title="Remonter en haut" class="h-12 w-12 rounded-full bg-gray-600 text-white shadow-lg hover:bg-gray-700 flex items-center justify-center" style="display: none;">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M5 11l7-7 7 7M5 19l7-7 7 7" />
        </svg>
    </button>
</div>
<div id="floating-cart" class="hidden lg:block fixed top-1/2 -translate-y-1/2 right-0 bg-white p-4 rounded-l-lg shadow-2xl border-l border-t border-b border-gray-200 w-64 cursor-pointer hover:shadow-indigo-200 transition-shadow">
    <h4 class="font-bold text-center mb-3 text-indigo-700">Votre Commande</h4>
    <div class="flex justify-between text-sm mb-2">
        <span class="text-gray-600">Articles :</span>
        <span id="floating-cart-total-articles" class="font-bold text-gray-800">0</span>
    </div>
    <div class="flex justify-between text-lg mb-3">
        <span class="text-gray-600">Total TTC :</span>
        <span id="floating-cart-grand-total" class="font-bold text-indigo-600">0.00€</span>
    </div>
    <hr>
    <div id="floating-cart-summary" class="mt-3 text-xs space-y-1">
        </div>
</div>
</body>
</html>
