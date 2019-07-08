<style scoped>
    video {
        object-fit: cover;
    }
    .video-container {
        overflow: hidden;
        background: black;
    }
    #capturecanvas {
        display: none;
    }
</style>

<template>
    <div id="container"> <!--v-if="dataReady"-->
        <video
                id="video"
                ref="video"
                v-on:play="onPlay"
                muted
                playsinline
                width="100%"
                height="100%"
                style=" -moz-transform: scaleX(-1);
            -o-transform: scaleX(-1);
            -webkit-transform: scaleX(-1);
            transform: scaleX(-1);
            "
        ></video>
        <canvas
                ref="capturecanvas"
                id="capturecanvas"
                width="100%"
                height="100%"
        ></canvas>
    </div>

</template>

<script>
    import * as faceapi from "face-api.js/dist/face-api.js";

    export default {
            name: "webcam",
            components: {},
            data() { return {}},//dataReady: false}},
            computed: {},
            async mounted() {
                //wait for the loading to complete before displaying
                await faceapi.loadMtcnnModel('/'),
                await faceapi.loadFaceRecognitionModel('/')
                this.video = this.$refs.video;
                if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                    navigator.mediaDevices.getUserMedia({video: {}})
                        .then(stream => {
                            this.video.srcObject = stream;
                            this.video.play();
                        })
                        .catch((e)=> {
                            console.log(e);
                        })}
                
                window.addEventListener('resize', this.onResize)
                
                let cls = this;
                document.addEventListener(
                "deviceready",
                function() {
                    cls.loadFaceDetectModels();
                },
                false
                );

                // //specification of params for mtcnn model used:
                // const mtcnnForwardParams = {
                // // number of scaled versions of the input image passed through the CNN
                // // of the first stage, lower numbers will result in lower inference time,
                // // but will also be less accurate
                // maxNumScales: 10,
                // // scale factor used to calculate the scale steps of the image
                // // pyramid used in stage 1
                // scaleFactor: 0.709,
                // // the score threshold values used to filter the bounding
                // // boxes of stage 1, 2 and 3
                // scoreThresholds: [0.6, 0.7, 0.7],
                // // minimum face size (in px)to expect: the higher the no., the faster processing will be,
                // // but smaller faces won't be detected
                // minFaceSize: 200
                // }
                
                // const mtcnnResults = await faceapi.mtcnn(document.getElementById('inputVideo'), mtcnnForwardParams)
                // //draws the bounding boxes onto the canvas overlaying the videoElement
                // faceapi.drawDetection('overlay', mtcnnResults.map(res => res.faceDetection), { withScore: false })
                // faceapi.drawLandmarks('overlay', mtcnnResults.map(res => res.faceLandmarks), { lineWidth: 4, color: 'red' })  
                // //Create descriptions per face
                // const options = new faceapi.MtcnnOptions(mtcnnResults)
                // const input = document.getElementById('videoElement')
                // const fullFaceDescriptions = await faceapi.detectAllFaces(input, options).withFaceLandmarks().withFaceDescriptors()

                // //input reference data from example pictures: 
                // const labels = ['sheldon', 'raj', 'leonard', 'howard', 'Me']

                // const labeledFaceDescriptors = await Promise.all(
                //     labels.map(async label => {
                //         // fetch image data from urls and convert to HTMLImage element
                //         const imgUrl = `${label}.png`
                //         const img = await faceapi.fetchImage(imgUrl)
                        
                //         // detect the face with the highest score in the image and compute it's landmarks and face descriptor
                //         const fullFaceDescription = await faceapi.detectSingleFace(img).withFaceLandmarks().withFaceDescriptor()
                        
                //         if (!fullFaceDescription) {
                //         throw new Error(`no faces detected for ${label}`)
                //         }
                        
                //         const faceDescriptors = [fullFaceDescription.descriptor]
                //         return new faceapi.LabeledFaceDescriptors(label, faceDescriptors)
                //         })
                // )
                // // 0.6 is a good distance threshold value to judge if descriptors match
                // const maxDescriptorDistance = 0.6
                // const faceMatcher = new faceapi.FaceMatcher(labeledFaceDescriptors, maxDescriptorDistance)

                // const results = fullFaceDescriptions.map(fd => faceMatcher.findBestMatch(fd.descriptor))
                // //overlay bounding box for each matched face
                // results.forEach((bestMatch, i) => {
                //     const box = fullFaceDescriptions[i].detection.box
                //     const text = bestMatch.toString()
                //     const drawBox = new faceapi.draw.DrawBox(box, { label: text })
                //     var canvas = document.getElementById("overlay")
                //     drawBox.draw(canvas)}
                
                //switch to qualify the starting of UI element
                //this.dataReady = true
                
            },
                
            destroyed() {
                window.removeEventListener('resize', this.onResize);
            },
            methods: {
                onResize() {
                    this.updateVideoDimensions();
                },
                updateVideoDimensions() {
                    this.$store.commit('updateVideoDimensions', {width: this.video.clientWidth, height: this.video.clientHeight});
                },
                onPlay(event) {
                    if (this.video.paused || this.video.ended)
                        return setTimeout(() => this.onPlay());
                    this.capturecanvas = this.$refs.capturecanvas;
                    let capturecanvas = this.capturecanvas
                        .getContext("2d")
                        .drawImage(
                            this.video,
                            0,
                            0,
                            520,
                            440
                        );
                },
                async loadFaceDetectModels() {
                let MODEL_URL;
                    if (true) {
                        MODEL_URL =
                        window.cordova.file.applicationDirectory + "www/static/models/";
                        faceapi.env.monkeyPatch({
                        readFile: filePath =>
                            new Promise(resolve => {
                            window.resolveLocalFileSystemURL(
                                filePath,
                                function(fileEntry) {
                                fileEntry.file(
                                    function(file) {
                                    var reader = new FileReader();

                                    let fileExtension = filePath
                                        .split("?")[0]
                                        .split(".")
                                        .pop();
                                    if (fileExtension === "json") {
                                        reader.onloadend = function() {
                                        resolve(this.result);
                                        };
                                        reader.readAsText(file);
                                    } else {
                                        reader.onloadend = function() {
                                        resolve(new Uint8Array(this.result));
                                        };

                                        reader.readAsArrayBuffer(file);
                                    }
                                    },
                                    function() {
                                    resolve(false);
                                    }
                                );
                                },
                                function() {
                                resolve(false);
                                }
                            );
                            }),
                        Canvas: HTMLCanvasElement,
                        Image: HTMLImageElement,
                        ImageData: ImageData,
                        Video: HTMLVideoElement,
                        createCanvasElement: () => document.createElement("canvas"),
                        createImageElement: () => document.createElement("img")
                        });
                        await faceapi.nets.tinyFaceDetector.loadFromDisk(MODEL_URL);
                        await faceapi.nets.faceRecognitionNet.loadFromDisk(MODEL_URL);
                        } 
                    else {
                        MODEL_URL = "./static/models";
                        await faceapi.loadTinyFaceDetectorModel(MODEL_URL);
                        await faceapi.loadFaceRecognitionModel(MODEL_URL);
                    }
                },
               
  
            }
            
          };
        
</script>

