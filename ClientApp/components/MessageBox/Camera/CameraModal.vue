<template>
<div class="camera-modal">
    <i @click="dialogVisible = true" class="fa fa-camera fa-2x"></i>

    <div class="modal" :class="{'is-active':dialogVisible}">
        <div class="modal-background"></div>
        <div class="modal-content">
                <component :online-user="onlineUser" @takePhoto="showPhotoComponent" @takeVideo="showVideoComponent" v-bind:is="currentTabComponent"></component>
        </div>
        <button class="modal-close is-large" aria-label="close"></button>
    </div>
   
</div>
</template>
<script>
import Photo from "./Photo"
import VideoComponent from "./Video"
export default {
    components: {
        Photo,
        VideoComponent
    },
    
    props: {
        onlineUser: {
            type: Object,
            required: true
        }
    },
    data () {
        return {
            dialogVisible: false,
            showRecordedVideo: false,
            currentTabComponent: Photo,
        }
    },
    methods: {
        showVideoComponent () {
            this.currentTabComponent = VideoComponent
        },
        showPhotoComponent (){
             this.currentTabComponent = Photo
        }
    },
}
</script>

<style lang="scss">
.camera-modal {
    .media-holder {

        position: relative;
        height: 100%;
        width: 100%;

        .counter {
            top: 50%;
            left: 50%;
            color: #ffffff;
            font-size: 90px;
            transform: translate(-50%, -50%);
            font-weight: bold;
            position: absolute;
        }
    }

    .modal-footer {
        .custom-modal-footer {
            display: flex;
            width: 100%;

            div {
                flex: 1
            }
        }
    }
}
</style>
