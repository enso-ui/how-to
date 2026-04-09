<template>
    <div class="columns is-reverse-mobile">
        <div class="column is-three-quarters-desktop is-full-touch">
            <fade enter="down"
                leave="up">
                <div class="columns how-to-editor"
                    v-if="addingVideo || editingVideo">
                    <div class="column is-narrow">
                        <div class="control">
                            <input class="input"
                                v-focus
                                type="text"
                                :placeholder="i18n('Video name')"
                                v-model="video.name">
                        </div>
                    </div>
                    <div class="column">
                        <div class="control is-expanded">
                            <textarea class="textarea"
                                rows="2"
                                type="text"
                                :placeholder="i18n('Video description')"
                                v-model="video.description"/>
                        </div>
                    </div>
                    <div class="column is-narrow is-flex">
                        <fade>
                            <div class="control"
                                 v-if="video.name">
                                <enso-uploader :url="uploadLink"
                                    :params="video"
                                    :file-size-limit="25*1024*1024"
                                    file-key="video"
                                    @upload-successful="reset(); getVideos()"
                                    v-if="addingVideo">
                                    <template #control="{ controlEvents }">
                                        <a v-on="controlEvents">
                                            <span class="file-cta how-to-editor__upload-cta">
                                                  <span class="file-icon">
                                                    <fa :icon="faUpload"/>
                                                  </span>
                                                  <span class="file-label">
                                                    {{ i18n('Video') }}…
                                                  </span>
                                            </span>
                                        </a>
                                    </template>
                                </enso-uploader>
                                <a class="button is-outlined is-success"
                                    @click="video = video; update()"
                                    v-if="editingVideo">
                                    <span class="icon">
                                        <fa :icon="faCheck"/>
                                    </span>
                                </a>
                            </div>
                        </fade>
                        <fade>
                            <div class="control"
                                v-if="addingVideo || editingVideo">
                                <a class="button is-danger is-outlined"
                                    @click="reset()">
                                    <span class="icon">
                                        <fa :icon="faBan"/>
                                    </span>
                                </a>
                            </div>
                        </fade>
                    </div>
                </div>
            </fade>
            <div class="columns is-multiline">
                <div class="column is-half"
                    v-for="(vid, index) in filteredVideos"
                    :key="vid.id">
                <how-to-video class="is-rounded"
                    :video="vid"
                    :tags="tags"
                    @start-tagging="video = vid; taggingId = video.id"
                    @stop-tagging="video = vid; taggingId = null; update()"
                    @delete="videos.splice(index, 1)"
                    @update="video = vid; update()"
                    @edit="video = vid; editingVideo = true;"/>
                </div>
            </div>
        </div>
        <div class="column is-one-quarter">
            <a class="button is-info is-fullwidth mb-2"
                :disabled="addDisabled"
                @click="addingVideo = true"
                v-if="canAccess('howTo.videos.store')">
                <span>
                    {{ i18n('Add video') }}
                </span>
                <span class="icon is-small">
                    <fa :icon="faPlus"/>
                </span>
            </a>
            <div class="box how-to-tags-panel">
                <div class="level">
                    <div class="level-left">
                        <div class="level-item">
                            <label class="label">
                                <span class="icon is-small">
                                    <fa :icon="faTags"
                                        size="xs"/>
                                </span>
                                {{ i18n('Tags') }}
                            </label>
                        </div>
                    </div>
                    <div class="level-right">
                        <div class="level-item">
                            <a class="button is-small is-outlined is-success"
                                @click="addTag"
                                v-if="canAccess('howTo.tags.store') && query && tagIsNew">
                                <span class="icon is-small">
                                    <fa :icon="faCheck"/>
                                </span>
                            </a>
                            <a class="button is-small is-outlined is-danger"
                                v-if="canAccess('howTo.tags.update') && !query && selectedTag"
                                @click="editingTag = true">
                                <span class="icon is-small">
                                    <fa :icon="faPen"/>
                                </span>
                            </a>
                            <a class="button is-small is-outlined is-success ml-1"
                                v-if="editingTag"
                                @click="editingTag = false; updateTag()">
                                <span class="icon is-small">
                                    <fa :icon="faCheck"/>
                                </span>
                            </a>
                            <a class="button is-small is-outlined ml-1"
                                v-if="editingTag"
                                @click="editingTag = false">
                                <span class="icon is-small">
                                    <fa :icon="faBan"/>
                                </span>
                            </a>
                        </div>
                    </div>
                </div>
                <input class="input"
                    type="text"
                    v-model="selectedTag.name"
                    v-if="editingTag">
                <input class="input"
                    type="text"
                    v-model="query"
                    @keypress.enter="addTag"
                    v-else>
                <div class="field is-grouped is-grouped-multiline mt-2">
                    <div class="control"
                         v-for="tag in filteredTags"
                         :key="tag.id">
                        <div class="tags has-addons">
                            <span :class="[
                                'tag is-white is-clickable',
                                { 'is-bold' : tag.selected }
                                ]"
                                @click="taggingId
                                ? video.tagList.push(tag.id)
                                : tag.selected = !tag.selected">
                                {{ tag.name }}
                            </span>
                            <a class="tag is-delete is-white"
                                @click="deleteTag(tag.id)"
                                v-if="canAccess('howTo.tags.destroy') && !taggingId"/>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { focus } from '@enso-ui/directives';
import { Fade } from '@enso-ui/transitions';
import { EnsoUploader } from '@enso-ui/uploader/bulma';
import {
    faBan,
    faCheck,
    faPen,
    faPlus,
    faTags,
    faUpload,
} from '@fortawesome/free-solid-svg-icons';
import { FontAwesomeIcon as Fa } from '@fortawesome/vue-fontawesome';
import HowToVideo from './components/HowToVideo.vue';

export default {
    name: 'Index',

    directives: { focus },

    components: {
        Fa, Fade, HowToVideo, EnsoUploader,
    },

    inject: ['canAccess', 'errorHandler', 'http', 'i18n', 'route', 'toastr'],

    data: () => ({
        faBan,
        faCheck,
        faPen,
        faPlus,
        faTags,
        faUpload,
        videos: [],
        query: '',
        tags: [],
        video: {
            name: null,
            description: null,
            tagList: [],
        },
        addingVideo: false,
        editingVideo: false,
        taggingId: null,
        editingTag: false,
    }),

    computed: {
        addDisabled() {
            return this.addingVideo || this.editingVideo
                ? true
                : null;
        },
        uploadLink() {
            return this.route('howTo.videos.store');
        },
        filteredVideos() {
            if (this.taggingId) {
                return this.videos.filter(({ id }) => id === this.taggingId);
            }

            return this.selectedTags.length === 0
                ? this.videos
                : this.videos.filter(({ tagList }) => tagList
                    .filter(
                        tagId => this.selectedTags.findIndex(({ id }) => tagId === id) !== -1,
                    ).length === this.selectedTags.length);
        },
        filteredTags() {
            return !this.query
                ? this.tags.filter(({ id }) => !this.video.tagList.includes(id))
                : this.tags.filter(({ name, id }) => !this.video.tagList.includes(id)
                    && name.toLowerCase().indexOf(this.query.toLowerCase()) > -1);
        },
        tagIsNew() {
            return !!this.query && this.tags
                .findIndex(({ name }) => name
                    .toLowerCase() === this.query.toLowerCase()) === -1;
        },
        selectedTags() {
            return this.tags.filter(({ selected }) => selected);
        },
        selectedTag() {
            return this.selectedTags.length === 1 && this.selectedTags[0];
        },
    },

    created() {
        this.getVideos();
        this.getTags();
    },

    methods: {
        getVideos() {
            this.http.get(this.route('howTo.videos.index'))
                .then(({ data }) => (this.videos = data))
                .catch(this.errorHandler);
        },
        getTags() {
            this.http.get(this.route('howTo.tags.index'))
                .then(({ data }) => (this.tags = data))
                .catch(this.errorHandler);
        },
        reset() {
            this.video = {
                name: null,
                description: null,
                tagList: [],
            };

            this.addingVideo = false;
            this.editingVideo = false;
            this.taggingId = null;
            this.editingTag = false;
        },
        tagVideo(tagMode) {
            if (!tagMode) {
                this.video.tagList.push(...this.selectedTags);
                this.update();
            }

            this.deselectTags();
        },
        deselectTags() {
            this.tags.map(tag => {
                tag.selected = false;
                return tag;
            });
        },
        addTag() {
            if (!this.tagIsNew || !this.canAccess('howTo.tags.store')) {
                return;
            }

            this.http.post(this.route('howTo.tags.store'), { name: this.query })
                .then(({ data }) => {
                    this.tags.push(data);
                    this.query = '';
                }).catch(this.errorHandler);
        },
        updateTag() {
            this.http.patch(this.route('howTo.tags.update', this.selectedTag.id), {
                name: this.selectedTag.name,
            }).catch(this.errorHandler);
        },
        deleteTag(tagId) {
            this.http.delete(this.route('howTo.tags.destroy', tagId))
                .then(() => {
                    const index = this.tags.findIndex(({ id }) => id === tagId);
                    this.tags.splice(index, 1);
                }).catch(this.errorHandler);
        },
        update() {
            this.http.patch(this.route('howTo.videos.update', this.video.id), this.video)
                .then(({ data }) => {
                    this.toastr.success(data.message);
                    this.reset();
                }).catch(this.errorHandler);
        },
    },
};
</script>

<style lang="scss">
.how-to-editor__upload-cta {
    background-color: var(--enso-filter-control-surface);
    color: var(--bulma-text-strong);

    &:hover,
    &:focus {
        background-color: var(--enso-filter-surface);
        color: var(--bulma-text-strong);
    }
}

.how-to-editor {
    .file-cta {
        color: var(--bulma-text-strong);
    }
}

.how-to-tags-panel.box {
    color: var(--bulma-text);

    .label {
        color: var(--bulma-text-strong);
    }

    .tag.is-white,
    .tag.is-delete.is-white {
        background-color: var(--bulma-scheme-main-ter);
        color: var(--bulma-text-strong);
    }

    .tag.is-delete.is-white:hover {
        background-color: color-mix(
            in srgb,
            var(--bulma-danger) 16%,
            var(--bulma-scheme-main-ter)
        );
    }
}
</style>
