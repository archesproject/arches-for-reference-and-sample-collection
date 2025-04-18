<script setup lang="ts">
import { onMounted, inject, ref } from "vue";
import type { Ref } from "vue";

import Button from "primevue/button";
import Skeleton from "primevue/skeleton";

import arches from "arches";
import { fetchImageData } from "@/afrc/Search/api.ts";

const resultsSelected = inject("resultsSelected") as Ref<string[]>;
const resultSelected = inject("resultSelected") as Ref<string>;
const zoomToFeature = inject("zoomToFeature") as Ref<string>;
const highlightResult = inject("highlightResult") as Ref<string>;
const showMap = inject("showMap") as Ref<string>;

const image: Ref<string> = ref("");

onMounted(async () => {
    if (props.searchResult?._source) {
        const res = await fetchImageData(
            [props.searchResult._source.resourceinstanceid],
            true,
        );
        if (res.length > 0) {
            image.value = res[0];
        }
    }
});

const props = defineProps({
    searchResult: {
        type: Object,
        required: true,
    },
    loading: {
        type: Boolean,
        required: true,
    },
});

function setHighlightResult(resourceid: string) {
    if (highlightResult.value !== resourceid) {
        highlightResult.value = resourceid;
    }
}

function clearHighlightResult() {
    highlightResult.value = "";
}

function selectResult(resourceid: string) {
    resultSelected.value = resourceid;
    resultsSelected.value = [resourceid];
}

function zoomToSearchResult(resourceid: string) {
    zoomToFeature.value = resourceid;
}
</script>

<template>
    <section v-if="loading">
        <div
            class="result"
            style="display: flex"
        >
            <div class="image-placeholder">
                <Skeleton
                    class="item-image"
                    height="100%"
                ></Skeleton>
            </div>
            <div
                class="result-content"
                width="100%"
                style="justify-content: unset; row-gap: 2rem; flex-grow: 1"
            >
                <Skeleton
                    width="100%"
                    height="4rem"
                ></Skeleton>
                <Skeleton
                    width="100%"
                    height="4rem"
                ></Skeleton>
            </div>
        </div>
    </section>
    <section v-else>
        <div
            class="result"
            @mouseover="setHighlightResult(props.searchResult._id)"
            @mouseleave="clearHighlightResult"
        >
            <div class="image-placeholder">
                <img class="item-image"
                    v-if="image"
                    :src="image"
                />
                <div
                    v-else
                    class="item-image no-image"
                >
                    <div>No image available</div>
                </div>
            </div>
            <div class="result-content">
                <div>
                    <div class="result-displayname">
                        {{ props.searchResult._source.displayname }}
                    </div>
                    <div class="item-current-location">
                        <span class="breadcrumb-title">Current location:</span>
                        <span class="breadcrumb"
                            >CGI Room 222, Aisle 3, Level B, Case 3</span
                        >
                    </div>
                    <div class="scope-note">
                        <span class="scope-note-title">Item description:</span>
                        <span
                            class="scope-note-content"
                            v-html="searchResult._source.displaydescription"
                        ></span>
                    </div>
                </div>
                <div>
                    <div class="actions">
                        <Button
                            class="action-button"
                            label="show more"
                            severity="secondary"
                            text
                            icon="pi pi-plus-circle"
                            size="large"
                            @click="
                                selectResult(
                                    searchResult._source.resourceinstanceid,
                                )
                            "
                        />
                        <Button
                            class="action-button"
                            label="edit"
                            severity="secondary"
                            text
                            as="a"
                            target="_blank"
                            size="large"
                            icon="pi pi-pen-to-square"
                            :href="
                                arches.urls.resource + '/' + searchResult._id
                            "
                        />
                        <div
                            v-if="
                                searchResult._source?.points?.length && showMap
                            "
                        >
                            <Button
                                class="action-button"
                                label="Map"
                                severity="secondary"
                                text
                                icon="pi pi-map-marker"
                                size="large"
                                @click="
                                    zoomToSearchResult(
                                        searchResult._source.resourceinstanceid,
                                    )
                                "
                            />
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<style scoped>
.result {
    background-color: #fff;
    border-bottom: 1px solid #ddd;
    display: flex;
    flex-direction: row;
}

.map-sidebar .result {
    border: none;
    border-bottom: 1px solid #ddd;
}

.result:hover {
    background: #f7f6fa;
}

.map-sidebar .result:hover {
    border: none;
    border-bottom: 1px solid #ddd;
    background: #f7f6fa;
}

.result .result-content {
    overflow: hidden;
    padding-inline-start: 10px;
    padding: 10px 10px 0px 15px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
}

.map-sidebar .result .result-content {
    justify-content: flex-start;
    padding-left: 15px;
}

.result h2 {
    margin: 0 0 10px;
    font-size: 1.2rem;
}
.item-current-location {
    line-height: 1;
}

.result .breadcrumb-title {
    color: #454545;
    font-size: 1.25rem;
    margin-bottom: 10px;
    padding-right: 5px;
}

.result .breadcrumb {
    color: #25476a;
    font-size: 1.25rem;
    margin-bottom: 10px;
    padding: unset;
}

.result .image-placeholder {
    width: 120px;
    height: 120px;
    min-width: 120px;
    background-color: #eee;
    margin: 5px;
    margin-left: 15px;
    display: flex;
    align-items: center;
}

.item-image {
    max-height: 118px;
    max-width: 118px;
    margin: 1px;
}

.no-image {
    display: flex;
    text-align: center;
    align-items: center;
    color: #555;
    justify-content: center;
    height: 100%;
    line-height: 1.15;
    font-weight: 500;
    background-color: rgb(236, 236, 236);
}

.map-sidebar .result .image-placeholder {
    display: none;
}

.result-displayname {
    font-size: 1.15em;
    font-weight: 500;
    color: #25476a;
    margin: 0px;
    line-height: 1.05;
}
.scope-note {
    margin-top: 5px;
    font-size: 1.125em;
}

.scope-note-title {
    color: #777;
    padding-right: 5px;
}

.map-sidebar .result .scope-note-title {
    display: none;
}

.scope-note-content {
    color: #25476a;
}

.map-sidebar .result .scope-note-content {
    font-size: 0.9em;
}

.actions {
    display: flex;
    gap: 10px;
    margin-top: 10px;
    padding-bottom: 5px;
}
.p-button-text.p-button-secondary.action-button {
    border-radius: 2px;
}

.p-button-text.p-button-secondary.action-button:hover {
    background: #dfdbeb;
    color: #25476a;
}
</style>
