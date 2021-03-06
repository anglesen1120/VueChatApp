<template>
    <div class="modal is-active">
        <div class="modal-background"></div>
        <div class="modal-card">
            <header class="modal-card-head">
                <p class="modal-card-title">Modal title</p>
            </header>
            <section class="modal-card-body">
                <div>
                    <div class="video-wrapper">
                        <figure>
                            <video ref="localVideo" autoplay muted></video>
                            <figcaption>
                                <p>Local Video</p>
                            </figcaption>
                        </figure>
                        <figure>
                            <video ref="remoteVideo" autoplay></video>
                            <figcaption>
                                <p>Remote Video</p>
                            </figcaption>
                        </figure>
                    </div>
                    <div class="call-controls">
                        <button :disabled="disableHangUpButton" @click="hangUpCall" class="btn btn-danger" ref="hangUpButton">Hang Up</button>
                    </div>
                </div>
            </section>
        </div>
    </div>


</template>

<script>
    //https://www.html5rocks.com/en/tutorials/webrtc/infrastructure/
    // Alice creates an RTCPeerConnection object.
    //     Alice creates an offer (an SDP session description) with the RTCPeerConnection createOffer() method.
    //     Alice calls setLocalDescription() with his offer.
    //     Alice stringifies the offer and uses a signaling mechanism to send it to Eve.
    //
    //     Eve calls setRemoteDescription() with Alice's offer, so that her RTCPeerConnection knows about Alice's setup.
    //     Eve calls createAnswer(), and the success callback for this is passed a local session description: Eve's answer.
    // Eve sets her answer as the local description by calling setLocalDescription().
    //     Eve then uses the signaling mechanism to send her stringified answer back to Alice.
    //     Alice sets Eve's answer as the remote session description using setRemoteDescription().
    
    import {EventBus} from "../../event-bus";

    export default {
        props: {
            onlineUser: {}
        },
        computed: {
            auth() {
                return this.$store.getters.auth
            },
        },

        data() {
            return {
                localVideo: null,
                remoteVideo: null,
                hangUpButton: null,
                localStream: null,
                myPeerConnection: null,
                disableHangUpButton:false,
               
                // remotePeerConnection: null,
                servers: null
            }
        },
        created() {
        },
        mounted() {
            let vm = this;
            this.localVideo = this.$refs.localVideo;
            this.remoteVideo = this.$refs.remoteVideo;
            this.hangUpButton = this.$refs.hangUpButton;
            EventBus.$on("webRtcSignalReceived", (msg)=>{
                if (msg.type=="answer") {
                    vm.handleAnswer(msg);
                }
                if(msg.type=="candidate"){
                    vm.handleIceCandidate(msg);
                }
            });
            this.createOffer();
            //https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling
            //https://github.com/mdn/samples-server/blob/master/s/webrtc-from-chat/chatclient.js
        },
        methods: {
             createOffer() {
                let vm = this;
              

                let mediaConstraints = {
                    audio: true, // We want an audio track
                    video: true, // ...and we want a video track,
                };
                navigator.mediaDevices.getUserMedia(mediaConstraints)
                    .then(function (localStream) {
                        vm.$refs.localVideo.srcObject = localStream;
                        vm.localStream=localStream;


                        vm.myPeerConnection = new RTCPeerConnection(
                            {
                                iceServers: [
                                    {
                                        urls: "stun:stun4.l.google.com:19302"
                                    }
                                ]
                            }
                        );
                        //  This is called when you call setRemoteDescription with the SDP info or your remote peer
                        vm.myPeerConnection.ontrack=vm.handleTrackEvent;
                        vm.myPeerConnection.onicecandidate =vm.gotIceCandidate;
                        localStream.getTracks().forEach(track => vm.myPeerConnection.addTrack(track, localStream));

                        vm.myPeerConnection.createOffer({
                            offerToReceiveAudio: true,
                            offerToReceiveVideo: true
                        }).then((desc) => {
                            vm.myPeerConnection.setLocalDescription(desc).then(() => {
                                return  vm.sendToServer({
                                    to: vm.onlineUser.id,
                                    from: vm.auth.userDetails.id,
                                    sdp: desc,
                                    type: "offer"
                                });

                            }).then(()=>{
                                vm.$store.commit("UPDATE_USER_ON_CALL",true);
                            });
                        });
                    });
              

            },
            async handleAnswer(msg){
                console.log("message answer handler");
                let vm = this;
               await vm.myPeerConnection.setRemoteDescription(new  RTCSessionDescription(msg.sdp)); 
              // vm.localStream.getTracks().forEach(track => vm.myPeerConnection.addTrack(track, vm.localStream));
            },
            
            handleTrackEvent(event) {
                console.log("handling remote streams");
                this.$refs.remoteVideo.srcObject = event.streams[0];
                //this.$refs.remoteVideo.play();
                //this.disableHangUpButton = false;
            },
            async  handleIceCandidate(msg){
                //RTCIceCandidate
               await this.myPeerConnection.addIceCandidate(new RTCIceCandidate(msg.candidate))
            },
            gotIceCandidate(event) {
                console.log("got ice candidate");
                let vm = this;
                if(event.candidate != null) {
                    vm.sendToServer({
                        'type': "candidate",
                        to: vm.onlineUser.id,
                        from: vm.auth.userDetails.id,
                        candidate:event.candidate
                    });
                }
            },
            
           async sendToServer(msg) {
              await  axios.post("/api/chats/web-rtc-signal", msg).then(resp => {
                  console.log("signaling done");
                })
            }
        }
    }
</script>

<style lang="scss" scoped>
    .video-wrapper {
        width: 100%;
        display: flex;
        //justify-content: space-between;
        figure {
            flex-basis: 50%;
            display: flex;
            flex-direction: column;

            video {
                flex: 1;
            }

            figcaption {
                align-self: center;
            }
        }

    }

    .call-controls {
        display: flex;
        justify-content: center;
    }
</style>



