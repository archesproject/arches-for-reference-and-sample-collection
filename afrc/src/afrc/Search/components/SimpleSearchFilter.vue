<script setup lang="ts">
import { ref, watch } from "vue";

import AutoComplete from "primevue/autocomplete";
import type { AutoCompleteCompleteEvent } from "primevue/autocomplete";

import arches from "arches";

import type { GenericObject } from "@/afrc/Search/types";

const componentName = "term-filter";
const props = defineProps({
    updateFilter: {
        type: Function,
        required: true,
    },
});
// const emit = defineEmits(["update:filter"]);

const items = ref();
const filter = ref({ terms: [] });

watch(
    () => filter.value.terms,
    (newValue, oldValue) => {
        console.log("Something has changed from", oldValue, "to", newValue);
        updateQuery();
    },
    { deep: true },
);

watch(items, (newValue, oldValue) => {
    console.log("Item has changed from", oldValue, "to", newValue);
});

function search(event: AutoCompleteCompleteEvent) {
    // items.value = [...Array(10).keys()].map((item) => event.query + "-" + item);
    var queryString = new URLSearchParams();
    queryString.set("q", event.query);
    queryString.set("lang", "*");
    fetch(arches.urls.search_terms + "?" + queryString.toString())
        .then((response) => response.json())
        .then((data) => {
            Object.keys(data).forEach((key) => {
                data[key].forEach((item: GenericObject) => {
                    item.inverted = false;
                });
            });
            let ret = [
                { label: "Terms", items: data.terms },
                { label: "Concepts", items: data.concepts },
            ];
            ret[0].items.unshift({
                text: event.query,
                type: "term",
                value: event.query,
                inverted: false,
            });
            items.value = ret;
        });
}

const updateQuery = function () {
    var terms = filter.value.terms.filter(function (term: GenericObject) {
        return (
            term.type === "string" ||
            term.type === "concept" ||
            term.type === "term"
        );
    }, this);

    // const query = {};// JSON.parse(props.queryString);
    // // if (terms.length > 0){
    //     query[componentName] = terms;
    //     // queryObj["language"] = this.language();
    // // }
    // else {
    //     delete query[componentName];
    // }
    props.updateFilter(componentName, terms);
    // props.queryString.value = JSON.stringify(query);
};
</script>

<template>
    <div class="search-bar">
        <AutoComplete
            v-model="filter.terms"
            multiple
            fluid
            :suggestions="items"
            option-label="text"
            option-group-label="label"
            option-group-children="items"
            placeholder="find ..."
            input-class="autocomplete-input"
            :auto-option-focus="true"
            @complete="search"
        >
            <template #optiongroup="slotProps">
                <div class="option-group">
                    {{ slotProps.option.label }}
                </div>
            </template>
            <template #option="slotProps">
                <div class="search-option">
                    {{ slotProps.option.text }}
                </div>
            </template>
        </AutoComplete>
    </div>
</template>

<style scoped>
.search-bar {
    width: 100%;
    font-size: 1rem;
}
.option-group,
.search-option {
    font-size: 1.4rem;
    font-family: "Lucida Sans", "Lucida Sans Regular", "Lucida Grande",
        "Lucida Sans Unicode", Geneva, Verdana, sans-serif;
}
:deep(.p-autocomplete-input-multiple) {
    border-radius: 3px;
}
:deep(.autocomplete-input) {
    height: 3rem;
    font-size: 1.5rem;
    padding: 10px;
}
:deep(.p-autocomplete-chip) {
    font-size: 1.5rem;
}
</style>
