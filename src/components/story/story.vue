<template>
    <!-- If the configuration file is being fetched, display a spinner to indicate loading. -->
    <div v-if="loadStatus === 'loading'">
        <div class="block py-20 align-middle text-center h-full" style="margin: 0 auto">
            <VueSpinnerOval size="120px" color="#009cd1" style="margin: 0 auto"></VueSpinnerOval>
        </div>
    </div>

    <div v-else-if="loadStatus === 'error'">
        <div class="block py-20 align-middle text-center h-full" style="margin: 0 auto">
            <div style="font-size: 200px">!</div>
            <div>
                {{ $t('story.error') }}
            </div>
        </div>
    </div>

    <div v-else-if="loadStatus === 'loaded'">
        <div class="storyramp-app bg-white" v-if="config !== undefined">
            <header class="story-header sticky top-0 w-full h-16 leading-9 bg-white border-b border-gray-200">
                <div class="flex w-full sm:px-6 py-3 mx-auto">
                    <mobile-menu
                        class="mobile-menu"
                        :active-chapter-index="activeChapterIndex"
                        :slides="config.slides"
                        :lang="lang"
                    />
                    <div class="flex-none w-mobile-full truncate font-semibold">
                        <span class="text-lg">{{ config.title }}</span>
                    </div>
                    <div class="flex justify-end flex-auto space-x-6">
                        <!-- Any links we want in the header can go here -->
                    </div>
                </div>
            </header>

            <intro :config="config.introSlide"></intro>

            <div class="w-full mx-auto pb-10" id="story">
                <story-content :config="config" :lang="lang" @step="updateActiveIndex" />
            </div>

            <footer class="p-8 pt-2 text-right text-sm">
                Context:
                <a class="text-blue-700 font-semibold" :href="config.contextLink" target="_NEW">{{
                    config.contextLabel
                }}</a>
                |
                <a href="https://github.com/ramp4-pcar4/story-ramp" target="_NEW" class="font-semibold text-blue-700"
                    >ramp4-pcar4/story-ramp</a
                >
            </footer>

            <div class="storyramp-modified" v-if="config.dateModified">
                {{ $t('story.date') }}
                {{ config.dateModified }}
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { getCurrentInstance, onMounted, ref } from 'vue';
import { useRoute, RouteLocationNormalized } from 'vue-router';

import MobileMenu from './mobile-menu.vue';
import StoryContent from '@storylines/components/story/story-content.vue';
import Intro from '@storylines/components/story/introduction.vue';

import { StoryRampConfig } from '@storylines/definitions';
import { VueSpinnerOval } from 'vue3-spinners';

const route = useRoute();

const config = ref<StoryRampConfig | undefined>(undefined);
const loadStatus = ref('loading');
const activeChapterIndex = ref(-1);
const lang = ref('en');

onMounted(() => {
    const uid = route.params.uid as string;
    lang.value = (route.params.lang as string) ? (route.params.lang as string) : 'en';
    if (uid) {
        fetchConfig(uid, lang.value);
    } else {
        console.error(`Please supply the language and id URL params in the form of /[lang]/[uid].`);
        // if no URL params have been provided redirect to canada.ca 404 page
        window.location.href = 'https://www.canada.ca/errors/404.html';
    }

    // set page lang
    const html = document.documentElement; // returns the html tag
    html.setAttribute('lang', lang.value);
    const instance = getCurrentInstance();
    if (instance?.proxy?.$i18n) {
        instance.proxy.$i18n.locale = lang.value;
    }
});

// react to param changes in URL
// eslint-disable-next-line
const beforeRouteUpdate = (to: RouteLocationNormalized, from: RouteLocationNormalized, next: () => void): void => {
    const uid = to.params.uid as string;
    lang.value = to.params.lang as string;
    const instance = getCurrentInstance();
    if (instance?.proxy?.$i18n) {
        instance.proxy.$i18n.locale = lang.value;
    }
    fetchConfig(uid, lang.value);
    next();
};

const fetchConfig = (uid: string, lang: string): void => {
    fetch(`${uid}/${uid}_${lang}.json`)
        .then((res) => {
            res.json().then((configs: StoryRampConfig) => {
                config.value = configs;
                loadStatus.value = 'loaded';
                // set page title
                if (config.value) {
                    document.title = config.value.title + ' - Canada.ca';
                }
            });
        })
        .catch((err) => {
            if (err.code === 'MODULE_NOT_FOUND') {
                console.error(`There exists no config given by the URL params: ${err}`);
                // redirect to canada.ca 404 page on invalid URL params
                window.location.href = 'https://www.canada.ca/errors/404.html';
            } else {
                // Some unknown error, possibly a build error that could indicate an error in the
                // configuration file.
                loadStatus.value = 'error';

                // Print out the error stack.
                console.error(err.stack);
            }
        });
};

const updateActiveIndex = (idx: number): void => {
    activeChapterIndex.value = idx;
};
</script>

<style lang="scss">
$font-list: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;

.storyramp-app {
    h1,
    h2,
    h3,
    h4,
    h5,
    h6,
    .h1,
    .h2,
    .h3,
    .h4,
    .h5,
    .h6 {
        font-family: $font-list;
        line-height: 1.5;
        border-bottom: 0px;
    }

    .storyramp-modified {
        max-width: 1536px;
        margin: 0 auto;
        padding-left: 15px;
        padding-top: 1em;
        padding-bottom: 1em;
    }

    .prose a {
        font-weight: bold;
    }

    .prose a:not([panel]):not([target='_self'])::after {
        content: url('../../assets/popout.svg');
    }

    .w-mobile-full {
        width: 80%;
    }
}

.story-header {
    z-index: 60;
}

@media screen and (min-width: 640px) {
    .mobile-menu {
        display: none !important;
    }
    .w-mobile-full {
        width: 100% !important;
    }
}
</style>
