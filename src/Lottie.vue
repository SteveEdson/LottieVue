<template>
    <div ref="animation" :class="['lottie', { parentClass }]"></div>
</template>

<script>
import lottie from 'lottie-web';

export default {
    props: {
        completeOnStop: {
            type: Boolean,
            default: () => false,
            required: false,
        },
        parentClass: {
            type: String,
            default: () => '',
        },
        autoplay: {
            type: Boolean,
            default: () => false,
        },
        loop: {
            type: Boolean,
            default: () => false,
        },
        playing: {
            type: Boolean,
            default: () => false,
        },
        progress: {
            type: Number,
            default: () => null,
        },
        path: {
            type: String,
            default: () => '',
        },
        animationData: {
            type: Object,
            default: () => {},
        },
        speed: {
            type: Number,
            default: () => 1,
        },
        direction: {
            type: Number,
            default: () => 1,
        },
    },
    mounted() {
        const lottieData = {
            container: this.$refs.animation,
            renderer: 'svg',
            loop: this.loop,
            autoplay: this.autoplay || this.playing,
        };

        if (this.path) {
            lottieData.path = this.path;
        } else if (this.animationData) {
            lottieData.animationData = this.animationData;
        }

        this.animationInstance = lottie.loadAnimation(lottieData);

        this.animationInstance.setSpeed(this.speed);
        this.animationInstance.setDirection(this.direction);
        this.animationInstance.addEventListener('data_ready', () => {
            if (this.direction < 0) {
                if (this.autoplay || this.playing) {
                    this.animationInstance.goToAndPlay(this.animationInstance.totalFrames, true);
                } else {
                    this.animationInstance.goToAndStop(this.animationInstance.totalFrames, true);
                }
            }
        });

        this.animationInstance.addEventListener('complete', () => {
            this.$emit('onComplete');
        });
    },
    unmounted() {
        this.animationInstance.destroy();
    },
    watch: {
        playing(newVal) {
            if (!this.autoplay) {
                if (newVal) {
                    this.animationInstance.play();
                } else {
                    if(this.completeOnStop) {
                        let loopListener = this.animationInstance.addEventListener('loopComplete', () => {
                            this.animationInstance.stop();
                            loopListener();
                        });
                    } else {
                        this.animationInstance.stop();
                    }
                }
            }
        },
        direction() {
            this.animationInstance.setDirection(this.direction);

            if (this.autoplay || this.playing) {
                this.animationInstance.goToAndPlay(this.direction > 0 ? 0 : this.animationInstance.totalFrames, true);
            } else {
                this.animationInstance.goToAndStop(this.direction > 0 ? 0 : this.animationInstance.totalFrames, true);
            }
        },
        progress(newProgress) {
            let frame = (newProgress / 100) * this.animationInstance.totalFrames;
            if (frame >= this.animationInstance.totalFrames) frame -= 1;

            this.animationInstance.goToAndStop(frame, true);
        },
    },
};
</script>
