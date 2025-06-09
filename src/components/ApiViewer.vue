<template>
    <div class="api-viewer">
        <div class="input-section">
            <input
                v-model="apiUrl"
                type="text"
                placeholder="Inserisci l'URL dell'API (es. https://jsonplaceholder.typicode.com/users)"
                class="api-input"
            />
            <button
                @click="fetchData"
                :disabled="isLoading"
                class="fetch-button"
            >
                {{ isLoading ? 'Caricamento...' : 'Invia' }}
            </button>
        </div>

        <div class="history-section" v-if="apiHistory.length > 0">
            <div class="history-header" @click="toggleHistory">
                <h3>Cronologia API</h3>
                <span class="toggle-icon">
                    <img
                        :src="'/src/assets/images/chevron-down-solid.svg'"
                        :style="{
                            transform: isHistoryOpen
                                ? 'rotate(180deg)'
                                : 'none',
                        }"
                        width="12"
                        height="12"
                        alt="chevron"
                    />
                </span>
            </div>
            <div v-show="isHistoryOpen" class="history-content">
                <ul class="history-list">
                    <li
                        v-for="(url, index) in apiHistory"
                        :key="index"
                        class="history-item"
                    >
                        <span class="history-url" @click="useHistoryUrl(url)">{{
                            url
                        }}</span>
                        <button
                            @click="removeFromHistory(index)"
                            class="remove-button"
                        >
                            <img
                                :src="'/src/assets/images/x-solid.svg'"
                                width="12"
                                height="12"
                                alt="chevron"
                            />
                        </button>
                    </li>
                </ul>
            </div>
        </div>

        <div v-if="error" class="error-message">
            {{ error }}
        </div>

        <div v-if="isLoading" class="loading">Caricamento dati...</div>

        <div v-if="data" class="data-display">
            <div class="view-toggle">
                <button
                    @click="viewMode = 'formatted'"
                    :class="{ active: viewMode === 'formatted' }"
                >
                    Vista Formattata
                </button>
                <button
                    @click="viewMode = 'raw'"
                    :class="{ active: viewMode === 'raw' }"
                >
                    JSON Raw
                </button>
            </div>

            <div v-if="viewMode === 'formatted'" class="formatted-view">
                <JsonTree :data="data" />
            </div>
            <pre v-else class="raw-view">
                {{ JSON.stringify(data, null, 2) }}
            </pre>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import JsonTree from '@/components/JsonTree.vue';

const apiUrl = ref('');
const data = ref(null);
const error = ref('');
const isLoading = ref(false);
const viewMode = ref('formatted');
const apiHistory = ref<string[]>([]);
const isHistoryOpen = ref(false);

// Carica la cronologia all'avvio
const loadHistory = () => {
    apiHistory.value = JSON.parse(localStorage.getItem('apiHistory') || '[]');
};

// Toggle della visibilità della cronologia
const toggleHistory = () => {
    isHistoryOpen.value = !isHistoryOpen.value;
};

// Usa un URL dalla cronologia
const useHistoryUrl = (url: string) => {
    apiUrl.value = url;
    fetchData();
};

// Rimuovi un URL dalla cronologia
const removeFromHistory = (index: number) => {
    apiHistory.value.splice(index, 1);
    localStorage.setItem('apiHistory', JSON.stringify(apiHistory.value));
};

// Carica la cronologia al montaggio del componente
loadHistory();

const fetchData = async () => {
    if (!apiUrl.value) {
        error.value = 'Per favore, inserisci un URL valido';
        return;
    }

    try {
        isLoading.value = true;
        error.value = '';

        const response = await fetch(apiUrl.value);
        if (!response.ok) {
            throw new Error(`Errore HTTP: ${response.status}`);
        }

        const jsonData = await response.json();
        data.value = jsonData;

        // Salva l'URL nella cronologia
        const history = JSON.parse(localStorage.getItem('apiHistory') || '[]');
        if (!history.includes(apiUrl.value)) {
            history.unshift(apiUrl.value);
            localStorage.setItem(
                'apiHistory',
                JSON.stringify(history.slice(0, 10))
            );
        }
    } catch (err) {
        error.value =
            err instanceof Error ? err.message : 'Si è verificato un errore';
        data.value = null;
    } finally {
        isLoading.value = false;
    }
};
</script>

<style scoped>
.api-viewer {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

.input-section {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
}

.api-input {
    flex: 1;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 16px;
}

.fetch-button {
    padding: 10px 20px;
    background-color: #4caf50;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
}

.fetch-button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
}

.error-message {
    color: #ff0000;
    margin: 10px 0;
    padding: 10px;
    background-color: #ffebee;
    border-radius: 4px;
}

.loading {
    text-align: center;
    margin: 20px 0;
    color: #666;
}

.data-display {
    margin-top: 20px;
}

.view-toggle {
    margin-bottom: 10px;
}

.view-toggle button {
    padding: 8px 16px;
    margin-right: 10px;
    border: 1px solid #ccc;
    background-color: #f5f5f5;
    cursor: pointer;
}

.view-toggle button.active {
    background-color: #4caf50;
    color: white;
    border-color: #4caf50;
}

.raw-view {
    background-color: #f5f5f5;
    padding: 20px;
    border-radius: 4px;
    overflow-x: auto;
    white-space: pre-wrap;
}

.history-section {
    margin: 20px 0;
    padding: 15px;
    background-color: #f5f5f5;
    border-radius: 4px;
}

.history-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    cursor: pointer;
    user-select: none;
}

.history-header h3 {
    margin: 0;
    color: #333;
}

.toggle-icon {
    font-size: 12px;
    color: #666;
}

.history-content {
    margin-top: 10px;
}

.history-list {
    list-style: none;
    padding: 0;
    margin: 0;
}

.history-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 8px;
    border-bottom: 1px solid #ddd;
}

.history-item:last-child {
    border-bottom: none;
}

.history-url {
    cursor: pointer;
    flex: 1;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    color: #3c9eee;
}

.history-url:hover {
    text-decoration: underline;
}

.remove-button {
    background: none;
    border: none;
    cursor: pointer;
    padding: 0 8px;
}

.toggle-icon img {
    transition: transform 0.3s ease;
}
</style>
