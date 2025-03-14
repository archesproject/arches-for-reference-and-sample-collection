<script setup lang="ts">
import { computed, ref, shallowRef } from "vue";
import Divider from "primevue/divider";

import {
    ACTIVE_LANGUAGE_DIRECTION,
    LTR,
    LEFT,
    RIGHT,
} from "@/afrc/Search/components/InteractiveMap/constants.ts";

import type { Component, Ref } from "vue";
import type { Map } from "maplibre-gl";
import type { MapInteractionItem, Settings } from "@/afrc/Search/types.ts";

const props = defineProps<{
    map: Map;
    items: MapInteractionItem[];
    settings: Settings | null;
}>();

const headerContent: Ref<string | null> = ref(null);
const selectedComponent: Ref<Component | null> = shallowRef(null);
const isOverlayVisible = ref(false);

const drawerPosition = computed(() => {
    if (props.settings![ACTIVE_LANGUAGE_DIRECTION] === LTR) {
        return RIGHT;
    } else {
        return LEFT;
    }
});

const openDrawer = (item: MapInteractionItem) => {
    headerContent.value = item.header;
    selectedComponent.value = item.component;
    isOverlayVisible.value = true;
};
</script>

<template>
    <div class="sidebar-container">
        <aside class="sidebar">
            <div
                v-for="item in items"
                :key="item.name"
            >
                <div
                    class="sidebar-item"
                    @click="
                        () => {
                            if (selectedComponent === item.component) {
                                isOverlayVisible = !isOverlayVisible;
                            } else {
                                openDrawer(item);
                            }
                        }
                    "
                >
                    <i :class="item.icon"></i>
                    <span style="font-size: 1.2rem">{{ item.name }}</span>
                </div>
                <Divider :pt="{ root: 'sidebar-item-divider' }" />
            </div>
        </aside>

        <div
            class="sliding-panel"
            :class="[drawerPosition, { 'is-visible': isOverlayVisible }]"
        >
            <div class="sliding-panel-header">
                <span>{{ headerContent }}</span>
                <button
                    class="close-button"
                    aria-label="Close"
                    @click="isOverlayVisible = false"
                >
                    ×
                </button>
            </div>
            <Divider />

            <component
                :is="selectedComponent"
                :map="props.map"
                :settings="props.settings"
            />
        </div>
    </div>
</template>

<style scoped>
.sidebar-container {
    position: relative;
    display: flex;
    height: 100%;
}

.sidebar {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 6rem;
    border-right: 1px solid var(--p-menubar-border-color);
    z-index: 1;
}

.sliding-panel {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 36rem;
    padding: 15px;
    background-color: var(--p-content-background);
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    transition: transform 0.3s ease;
    z-index: 2;
    overflow: auto;
}

.sliding-panel.left {
    left: 3rem;
    transform: translateX(-100%);
}

.sliding-panel.left.is-visible {
    transform: translateX(0);
}

.sliding-panel.right {
    right: 0;
    transform: translateX(100%);
}

.sliding-panel.right.is-visible {
    transform: translateX(0);
}

.sliding-panel-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 1rem;
    font-weight: bold;
}

.close-button {
    background: none;
    border: none;
    font-size: 1.6rem;
    line-height: 1;
    cursor: pointer;
    color: var(--p-content-color);
    padding: 0;
    margin: 0;
}

.close-button:focus {
    outline: none;
}

.sidebar-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    font-size: 1.3rem;
    width: 6rem;
    height: 6rem;
}

.sidebar-item:hover {
    background: var(--p-button-secondary-hover-background);
    color: var(--p-button-secondary-hover-color);
}

.sidebar-item-divider {
    margin: 0;
}
</style>
