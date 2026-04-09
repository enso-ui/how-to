<template>
    <card class="how-to-video-card">
        <card-header>
            <template #title>
                <span class="icon is-small mr-1">
                    <fa :icon="faVideo"/>
                </span>
                {{ video.name }}
            </template>
            <template #controls>
                <card-control v-tooltip="video.description">
                    <span class="icon">
                        <fa :icon="faCircleInfo"/>
                    </span>
                </card-control>
                <card-control v-if="!video.poster && canAccess('howTo.posters.store')">
                    <enso-uploader class="pt-1"
                        :url="route('howTo.posters.store')"
                        :params="{ videoId: video.id }"
                        file-key="poster"
                        @upload-successful="video.poster = $event">
                        <template #control="{ controlEvents }">
                            <span class="icon"
                                v-on="controlEvents">
                                <fa :icon="faImage"/>
                            </span>
                        </template>
                    </enso-uploader>
                </card-control>
                <card-control v-if="canAccess('howTo.videos.update')">
                    <span class="icon"
                        @click="$emit('edit')">
                        <fa :icon="faPenToSquare"/>
                    </span>
                </card-control>
                <card-control v-if="canAccess('howTo.videos.update')">
                    <span class="icon"
                        @click="tagging = !tagging;
                            $emit(tagging ? 'start-tagging' : 'stop-tagging')">
                        <fa :icon="tagging ? faCheck : faTags"/>
                    </span>
                </card-control>
                <card-control v-if="canAccess('howTo.posters.destroy') && video.poster">
                    <confirmation @confirm="destroyPoster">
                        <span class="icon is-small">
                            <fa :icon="faTrashCan"/>
                        </span>
                    </confirmation>
                </card-control>
                <card-control v-else-if="canAccess('howTo.videos.destroy')">
                    <confirmation @confirm="destroyVideo">
                        <span class="icon is-small">
                            <fa :icon="faTrashCan"/>
                        </span>
                    </confirmation>
                </card-control>
                <card-collapse/>
            </template>
        </card-header>
        <card-content class="p-0">
            <video-player :options="options()"
                class="vjs-custom-skin"
                playsinline/>
        </card-content>
        <card-footer>
            <card-footer-item>
                <div class="field is-grouped is-grouped-multiline"
                    v-if="video.tagList.length">
                    <div class="control"
                        v-for="(tag, index) in tagList"
                        :key="index">
                        <div class="tags is-bold has-addons">
                            <span class="tag">
                                {{ tag.name }}
                            </span>
                            <a class="tag is-delete"
                                 @click="removeTag(tag)"
                                v-if="canAccess('howTo.videos.update') && tagging"/>
                        </div>
                    </div>
                </div>
                <span class="tag"
                    v-else>
                    {{ i18n('untagged') }}
                </span>
            </card-footer-item>
        </card-footer>
    </card>
</template>

<script>
import 'v-tooltip/dist/v-tooltip.css';
import { VTooltip } from 'v-tooltip';
import { FontAwesomeIcon as Fa } from '@fortawesome/vue-fontawesome';
import {
    faCheck,
    faCircleInfo,
    faTags,
    faVideo,
} from '@fortawesome/free-solid-svg-icons';
import { faTrashCan, faPenToSquare, faImage } from '@fortawesome/free-regular-svg-icons';
import {
    Card, CardHeader, CardCollapse, CardControl, CardContent,
    CardFooter, CardFooterItem,
} from '@enso-ui/card/bulma';
import Confirmation from '@enso-ui/confirmation/bulma';
import { EnsoUploader } from '@enso-ui/uploader';
import VideoPlayer from './VideoPlayer.vue';
import 'video.js/dist/video-js.css';

export default {
    name: 'HowToVideo',

    directives: { tooltip: VTooltip },

    components: {
        Card,
        CardControl,
        Confirmation,
        CardHeader,
        CardCollapse,
        CardFooter,
        CardFooterItem,
        CardContent,
        Fa,
        EnsoUploader,
        VideoPlayer,
    },

    inject: ['canAccess', 'errorHandler', 'http', 'i18n', 'route', 'toastr'],

    props: {
        video: {
            type: Object,
            required: true,
        },
        tags: {
            type: Array,
            required: true,
        },
    },

    emits: ['delete', 'edit', 'start-tagging', 'stop-tagging'],

    data: () => ({
        faCheck,
        faCircleInfo,
        faImage,
        faPenToSquare,
        faTags,
        faTrashCan,
        faVideo,
        tagging: false,
    }),

    computed: {
        tagList() {
            return this.tags.filter(({ id }) => this.video.tagList.includes(id));
        },
    },

    methods: {
        options() {
            return {
                muted: false,
                language: 'en',
                playbackRates: [0.7, 1.0, 1.5, 2.0],
                aspectRatio: '16:9',
                sources: [{
                    type: 'video/mp4',
                    src: this.route('howTo.videos.show', this.video.id),
                }],
                poster: this.video.poster
                    ? this.route('howTo.posters.show', this.video.poster.id)
                    : '',
            };
        },
        destroyPoster() {
            this.http.delete(this.route('howTo.posters.destroy', this.video.poster.id))
                .then(({ data }) => {
                    this.toastr.success(data.message);
                    this.video.poster = null;
                }).catch(this.errorHandler);
        },
        destroyVideo() {
            this.http.delete(this.route('howTo.videos.destroy', this.video.id))
                .then(({ data }) => {
                    this.toastr.success(data.message);
                    this.$emit('delete');
                }).catch(this.errorHandler);
        },
        removeTag(tag) {
            const index = this.video.tagList.findIndex(id => id === tag.id);
            this.video.tagList.splice(index, 1);
        },
    },
};
</script>

<style lang="scss" scoped>
:deep(.how-to-video-card.card) {
    overflow: hidden;
}

:deep(.how-to-video-card .card-header-title),
:deep(.how-to-video-card .card-header-icon),
:deep(.how-to-video-card .card-header .icon) {
    color: var(--bulma-text-strong);
}

:deep(.how-to-video-card > .card-content) {
    color: var(--bulma-text);
}

:deep(.how-to-video-card > .card-footer),
:deep(.how-to-video-card .card-footer-item) {
    background-color: var(--bulma-card-footer-background-color) !important;
    border-color: var(--bulma-border);
    color: var(--bulma-text);
}

:deep(.how-to-video-card .card-content .video-js),
:deep(.how-to-video-card .card-content .vjs-poster),
:deep(.how-to-video-card .card-content .vjs-tech) {
    background-color: var(--bulma-scheme-main-bis) !important;
}

:deep(.how-to-video-card .card-content .video-js) {
    color: var(--bulma-text);
}

:deep(.how-to-video-card .card-content .vjs-control-bar) {
    background: color-mix(
        in srgb,
        var(--bulma-scheme-main-ter) 88%,
        black
    );
}

:deep(.how-to-video-card .card-content .vjs-big-play-button) {
    border-color: color-mix(in srgb, var(--bulma-primary) 35%, var(--bulma-border));
    background-color: color-mix(
        in srgb,
        var(--bulma-scheme-main) 80%,
        black
    );
    color: var(--bulma-text-strong);
}

:deep(.how-to-video-card .card-control) {
    color: var(--bulma-text-light);
}

:deep(.how-to-video-card .card-control:hover),
:deep(.how-to-video-card .card-control:focus) {
    color: var(--bulma-text-strong);
}

:deep(.how-to-video-card .tag) {
    background-color: var(--bulma-scheme-main-ter);
    color: var(--bulma-text-strong);
}

.card-footer {
    white-space: nowrap;
    overflow: auto;
    text-overflow: ellipsis;
}
</style>
