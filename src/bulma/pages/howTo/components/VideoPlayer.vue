<template>
    <div class="video-player" v-if="reseted">
        <video class="native-video"
            ref="video"
            :controls="controls"
            :muted="options.muted"
            :poster="options.poster"
            :preload="options.preload || 'metadata'"
            @loadeddata="emitState('loadeddata', true)"
            @canplay="emitState('canplay', true)"
            @canplaythrough="emitState('canplaythrough', true)"
            @play="emitState('play', true)"
            @pause="emitState('pause', true)"
            @waiting="emitState('waiting', true)"
            @playing="emitState('playing', true)"
            @ended="emitState('ended', true)"
            @error="emitState('error', true)"
            @timeupdate="emitState('timeupdate', currentTime())">
            <source v-for="source in sources"
                :key="source.src"
                :src="source.src"
                :type="source.type">
            <track v-for="crtTrack in trackList"
               :key="crtTrack.src"
               :kind="crtTrack.kind"
               :label="crtTrack.label"
               :src="crtTrack.src"
               :srcLang="crtTrack.srcLang"
               :default="crtTrack.default">
        </video>
    </div>
</template>

<script>
export default {
    name: 'VideoPlayer',

    props: {
        start: {
            type: Number,
            default: 0,
        },
        crossOrigin: {
            type: String,
            default: '',
        },
        playsinline: {
            type: Boolean,
            default: false,
        },
        customEventName: {
            type: String,
            default: 'statechanged',
        },
        options: {
            type: Object,
            required: true,
        },
        events: {
            type: Array,
            default: () => [],
        },
        globalOptions: {
            type: Object,
            default: () => ({
                // autoplay: false,
                controls: true,
                // preload: 'auto',
                // fluid: false,
                // muted: false,
                controlBar: {
                    remainingTimeDisplay: false,
                    playToggle: {},
                    progressControl: {},
                    fullscreenToggle: {},
                    volumeMenuButton: {
                        inline: false,
                        vertical: true,
                    },
                },
                techOrder: ['html5'],
                plugins: {},
            }),
        },
        globalEvents: {
            type: Array,
            default: () => [],
        },
        trackList: {
            type: Array,
            default: () => [],
        },
    },

    emits: ['ready'],

    data() {
        return {
            reseted: true,
        };
    },

    computed: {
        controls() {
            return this.options.controls ?? this.globalOptions.controls ?? true;
        },
        sources() {
            return this.options.sources ?? [];
        },
    },

    watch: {
        options: {
            deep: true,
            handler() {
                this.reload();
            },
        },
    },

    mounted() {
        this.initialize();
    },

    beforeUnmount() {
        this.dispose();
    },

    methods: {
        initialize() {
            if (this.playsinline) {
                this.$refs.video.setAttribute('playsinline', this.playsinline);
                this.$refs.video.setAttribute('webkit-playsinline', this.playsinline);
                this.$refs.video.setAttribute('x5-playsinline', this.playsinline);
                this.$refs.video.setAttribute('x5-video-player-type', 'h5');
                this.$refs.video.setAttribute('x5-video-player-fullscreen', false);
            }

            // cross origin
            if (this.crossOrigin !== '') {
                this.$refs.video.crossOrigin = this.crossOrigin;
                this.$refs.video.setAttribute('crossOrigin', this.crossOrigin);
            }

            this.$nextTick(() => {
                if (this.start) {
                    this.$refs.video.currentTime = this.start;
                }

                this.$emit('ready', this.$refs.video);
            });
        },
        currentTime() {
            return this.$refs.video?.currentTime ?? 0;
        },
        emitState(event, value) {
            this.$emit(event, this.$refs.video);
            this.$emit(this.customEventName, { [event]: value });
        },
        reload() {
            this.$nextTick(() => {
                this.$refs.video?.load();
            });
        },
        dispose(callback) {
            if (this.$refs.video) {
                this.$refs.video.pause();
                this.$refs.video.removeAttribute('src');
                this.$refs.video.load();
            }

            this.$nextTick(() => {
                this.reseted = false;
                this.$nextTick(() => {
                    this.reseted = true;
                    this.$nextTick(() => {
                        callback && callback();
                    });
                });
            });
        },
    },
};
</script>

<style>
.video-player,
.video-player .native-video {
    background-color: var(--bulma-scheme-main-bis);
}

.video-player .native-video {
    color: var(--bulma-text);
    display: block;
    width: 100%;
    aspect-ratio: 16 / 9;
}
</style>
