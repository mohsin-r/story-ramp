<template>
    <div class="flex items-stretch">
        <chapter-menu
            class="side-menu"
            :active-chapter-index="activeChapterIndex"
            :slides="config.slides"
            :editor="!!configFileStructure"
            :lang="lang"
        />

        <VueScrollama class="relative story-scrollama w-full" @step-enter="stepEnter">
            <div
                v-for="(slide, idx) in config.slides"
                class="flex pt-24"
                :key="idx"
                :data-chapter-index="idx"
                :id="`${idx}-${slide.title.toLowerCase().replaceAll(' ', '-')}`"
                :name="`${idx}-${slide.title.toLowerCase().replaceAll(' ', '-')}`"
            >
                <slide :config="slide" :configFileStructure="configFileStructure" :slideIdx="idx" :lang="lang"></slide>
            </div>
        </VueScrollama>
    </div>
</template>

<script setup lang="ts">
import type { PropType } from 'vue';
import { onMounted, ref } from 'vue';
import { useRoute } from 'vue-router';
import 'intersection-observer';
import VueScrollama from 'vue3-scrollama';
import { ConfigFileStructure, StoryRampConfig } from '@storylines/definitions';

import ChapterMenu from './chapter-menu.vue';
import Slide from './slide.vue';

const route = useRoute();
const emit = defineEmits(['step']);

defineProps({
    config: {
        type: Object as PropType<StoryRampConfig>,
        required: true
    },
    configFileStructure: {
        type: Object as PropType<ConfigFileStructure>
    },
    lang: {
        type: String,
        required: true
    }
});

const activeChapterIndex = ref(-1);

onMounted(() => {
    const hash = route.hash.substring(1);
    if (hash) {
        const decodedHash = decodeURIComponent(hash);
        const el = document.getElementById(decodedHash);
        el?.scrollIntoView({ behavior: 'smooth' });
    }
});

const stepEnter = ({ element }: { element: HTMLElement }): void => {
    activeChapterIndex.value = parseInt(element.dataset.chapterIndex || '-1');
    emit('step', activeChapterIndex.value);
};
</script>

<style lang="scss" scoped>
.story-scrollama {
    background: var(--sr-content-background);
    // background: linear-gradient(to right, var(--sr-content-background) 33.3%, #fff 33.3%);
    // background: linear-gradient(to right, var(--sr-content-background) 40%, #fff 40%);

    border-style: solid none solid solid;
    border-width: 1px 0 1px 1px;

    // border-gray-200
    border-color: var(--sr-border-colour);

    // on the left
    &::before {
        content: '';
        position: absolute;
        height: 100%;
        width: 1px;
        left: 0;
        // modified tailwind shadow
        box-shadow: -3px 0px 6px 0px rgba(0, 0, 0, 0.1), -2px 0 4px 0px rgba(0, 0, 0, 0.06);
    }

    // above
    > *:first-child::before {
        content: '';
        position: absolute;
        width: 100%;
        height: 1px;
        top: 0;
        // modified tailwind shadow
        box-shadow: 0 -3px 6px 0px rgba(0, 0, 0, 0.1), 0 -2px 4px 0px rgba(0, 0, 0, 0.06);
    }

    // below
    > *:last-child::before {
        content: '';
        position: absolute;
        width: 100%;
        height: 1px;
        bottom: 0;
        box-shadow: 0 3px 6px 0px rgba(0, 0, 0, 0.1), 0 2px 4px 0px rgba(0, 0, 0, 0.06);
    }
}

@media screen and (max-width: 640px) {
    .side-menu {
        display: none;
    }
}
</style>
