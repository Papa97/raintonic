<template>
    <div class="json-tree">
        <div v-if="isArray(data)" class="array-container">
            <div class="node-header" @click="toggleNode('array')">
                <span class="toggle-icon">
                    <img
                        :src="chevronDownIcon"
                        :style="{
                            transform: isExpanded('array')
                                ? 'rotate(90deg)'
                                : 'rotate(270deg)',
                        }"
                        width="12"
                        height="12"
                        alt="chevron"
                    />
                </span>
                <span class="bracket">[</span>
                <span v-if="!isExpanded('array')" class="collapsed-content">
                    ...]
                </span>
            </div>
            <div v-if="isExpanded('array')" class="node-content">
                <div
                    v-for="(item, index) in data"
                    :key="index"
                    class="array-item"
                >
                    <div class="array-index">{{ index }}:</div>
                    <JsonTree :data="item" />
                </div>
                <div class="bracket">]</div>
            </div>
        </div>
        <div v-else-if="isObject(data)" class="object-container">
            <div class="node-header" @click="toggleNode('object')">
                <span class="toggle-icon">
                    <img
                        :src="chevronDownIcon"
                        :style="{
                            transform: isExpanded('object')
                                ? 'rotate(90deg)'
                                : 'rotate(270deg)',
                        }"
                        width="12"
                        height="12"
                        alt="chevron"
                    />
                </span>
                <span class="bracket">{</span>
                <span v-if="!isExpanded('object')" class="collapsed-content">
                    ...}
                </span>
            </div>
            <div v-if="isExpanded('object')" class="node-content">
                <div
                    v-for="(value, key) in data"
                    :key="key"
                    class="object-item"
                >
                    <div class="key">{{ key }}:</div>
                    <div class="value">
                        <JsonTree v-if="isComplex(value)" :data="value" />
                        <span v-else class="primitive-value">{{
                            formatValue(value)
                        }}</span>
                    </div>
                </div>
                <div class="bracket">}</div>
            </div>
        </div>
        <span v-else class="primitive-value">{{ formatValue(data) }}</span>
    </div>
</template>

<script setup lang="ts">
import { isArray, isObject } from 'lodash-es';
import { ref } from 'vue';
import chevronDownIcon from '@/assets/images/chevron-down-solid.svg';

const props = defineProps<{
    data: unknown;
}>();

const expandedNodes = ref<Set<string>>(new Set(['array', 'object']));

const isComplex = (value: any): boolean => {
    return isArray(value) || isObject(value);
};

const formatValue = (value: any): string => {
    if (value === null) return 'null';
    if (value === undefined) return 'undefined';
    if (typeof value === 'string') return `"${value}"`;
    return String(value);
};

const toggleNode = (nodeType: string) => {
    if (expandedNodes.value.has(nodeType)) {
        expandedNodes.value.delete(nodeType);
    } else {
        expandedNodes.value.add(nodeType);
    }
};

const isExpanded = (nodeType: string): boolean => {
    return expandedNodes.value.has(nodeType);
};
</script>

<style scoped>
.json-tree {
    font-family: monospace;
    font-size: 14px;
    line-height: 1.5;
}

.node-header {
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 4px;
}

.toggle-icon {
    color: #666;
    font-size: 12px;
    width: 16px;
    text-align: center;
}

.bracket {
    color: #666;
}

.collapsed-content {
    color: #666;
}

.node-content {
    margin-left: 20px;
}

.array-container,
.object-container {
    margin-left: 20px;
}

.array-item,
.object-item {
    display: flex;
    margin: 4px 0;
}

.array-index {
    color: #666;
    margin-right: 8px;
    min-width: 30px;
}

.key {
    color: #3c9eee;
    margin-right: 8px;
}

.primitive-value {
    color: #4caf50;
}

.object-item .value {
    flex: 1;
}
</style>
