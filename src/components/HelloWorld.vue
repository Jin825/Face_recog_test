<template>
  <div id="container">
    <video autoplay="true" v-if="videoElement"></video>
    <canvas v-if="overlay" />
  </div>
</template>

<script>
import faceapi from "face-api.js/dist/face-api.js";

export default {
  mounted: async function run() {
    await faceapi.loadMtcnnModel('/')
    await faceapi.loadFaceRecognitionModel('/')

    // try to access users webcam and stream the images
    // to the video element
    const videoEl = document.getElementById("videoElement");

    navigator.getUserMedia(
    { video: {} },
    stream => videoEl.srcObject = stream,
    err => console.error(err)
    );
    /*if (navigator.mediaDevices.getUserMedia) {
      navigator.mediaDevices
        .getUserMedia({ video: true })
        .then(function(stream) {
          let fullFaceDescriptions = faceapi
            .detectAllFaces(stream)
            .withFaceLandmarks()
            .withFaceDescriptors();
          video.srcObject = fullFaceDescriptions;
        })
        .catch(function(error) {
          console.log("Something went wrong!");
        });
    }*/
    //specification of params for mtcnn model used:
    const mtcnnForwardParams = {
    // number of scaled versions of the input image passed through the CNN
    // of the first stage, lower numbers will result in lower inference time,
    // but will also be less accurate
      maxNumScales: 10,
    // scale factor used to calculate the scale steps of the image
    // pyramid used in stage 1
      scaleFactor: 0.709,
    // the score threshold values used to filter the bounding
    // boxes of stage 1, 2 and 3
      scoreThresholds: [0.6, 0.7, 0.7],
    // minimum face size (in px)to expect: the higher the no., the faster processing will be,
    // but smaller faces won't be detected
      minFaceSize: 200
    }
    const mtcnnResults = await faceapi.mtcnn(document.getElementById('videoElement'), mtcnnForwardParams)
    //draws the bounding boxes onto the invis canvas overlaying the videoElement
    faceapi.drawDetection('overlay', mtcnnResults.map(res => res.faceDetection), { withScore: false })
    faceapi.drawLandmarks('overlay', mtcnnResults.map(res => res.faceLandmarks), { lineWidth: 4, color: 'red' })  
    //Create descriptions per face
    const options = new faceapi.MtcnnOptions(mtcnnResults)
    const input = document.getElementById('videoElement')
    const fullFaceDescriptions = await faceapi.detectAllFaces(input, options).withFaceLandmarks().withFaceDescriptors()

    //input reference data from example pictures: 
    const labels = ['sheldon', 'raj', 'leonard', 'howard', 'Me']

    const labeledFaceDescriptors = await Promise.all(
      labels.map(async label => {
        // fetch image data from urls and convert blob to HTMLImage element
        const imgUrl = `${label}.png`
        const img = await faceapi.fetchImage(imgUrl)
        
        // detect the face with the highest score in the image and compute it's landmarks and face descriptor
        const fullFaceDescription = await faceapi.detectSingleFace(img).withFaceLandmarks().withFaceDescriptor()
        
        if (!fullFaceDescription) {
          throw new Error(`no faces detected for ${label}`)
        }
        
        const faceDescriptors = [fullFaceDescription.descriptor]
        return new faceapi.LabeledFaceDescriptors(label, faceDescriptors)
      })
    )
    // 0.6 is a good distance threshold value to judge
    // whether the descriptors match or not
    const maxDescriptorDistance = 0.6
    const faceMatcher = new faceapi.FaceMatcher(labeledFaceDescriptors, maxDescriptorDistance)

    const results = fullFaceDescriptions.map(fd => faceMatcher.findBestMatch(fd.descriptor))
    //overlay bounding box for each matched face
    results.forEach((bestMatch, i) => {
      const box = fullFaceDescriptions[i].detection.box
      const text = bestMatch.toString()
      const drawBox = new faceapi.draw.DrawBox(box, { label: text })
      var canvas = document.getElementById("overlay")
      drawBox.draw(canvas)
    })



    async function onPlay(videoEl) {
    // run face detection & recognition
    // ...
    // await faceapi.loadMtcnnModel('/')
    // await faceapi.loadFaceRecognitionModel('/')

    // // try to access users webcam and stream the images
    // // to the video element

    // navigator.getUserMedia(
    // { video: {} },
    // stream => videoEl.srcObject = stream,
    // err => console.error(err)
    // )
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
    // const mtcnnResults = await faceapi.mtcnn(document.getElementById('videoElement'), mtcnnForwardParams)
    // //draws the bounding boxes onto the invis canvas overlaying the videoElement
    // faceapi.drawDetection('overlay', mtcnnResults.map(res => res.faceDetection), { withScore: false })
    // faceapi.drawLandmarks('overlay', mtcnnResults.map(res => res.faceLandmarks), { lineWidth: 4, color: 'red' })  
    // const alignedFaceBoxes = results.map(
    // ({ faceLandmarks }) => faceLandmarks.align()
    // )

    // const alignedFaceTensors = await extractFaceTensors(input, alignedFaceBoxes)

    // const descriptors = await Promise.all(alignedFaceTensors.map(
    //   faceTensor => faceapi.computeFaceDescriptor(faceTensor)
    // ))

    // // free memory
    // alignedFaceTensors.forEach(t => t.dispose())
    // //input reference data from example pictures: 
    // //TODO: Add in own reference folder with pictures
    // const labels = ['sheldon', 'raj', 'leonard', 'howard', 'Me']

    // const labeledFaceDescriptors = await Promise.all(
    //   labels.map(async label => {
    //     // fetch image data from urls and convert blob to HTMLImage element
    //     const imgUrl = `${label}.png`
    //     const img = await faceapi.fetchImage(imgUrl)
        
    //     // detect the face with the highest score in the image and compute it's landmarks and face descriptor
    //     const fullFaceDescription = await faceapi.detectSingleFace(img).withFaceLandmarks().withFaceDescriptor()
        
    //     if (!fullFaceDescription) {
    //       throw new Error(`no faces detected for ${label}`)
    //     }
        
    //     const faceDescriptors = [fullFaceDescription.descriptor]
    //     return new faceapi.LabeledFaceDescriptors(label, faceDescriptors)
    //   })
    // )
    // // 0.6 is a good distance threshold value to judge
    // // whether the descriptors match or not
    // const maxDescriptorDistance = 0.6
    // const faceMatcher = new faceapi.FaceMatcher(labeledFaceDescriptors, maxDescriptorDistance)

    // const results = fullFaceDescriptions.map(fd => faceMatcher.findBestMatch(fd.descriptor))
    // //overlay bounding box for each matched face
    // results.forEach((bestMatch, i) => {
    // const box = fullFaceDescriptions[i].detection.box
    // const text = bestMatch.toString()
    // const drawBox = new faceapi.draw.DrawBox(box, { label: text })
    // drawBox.draw(canvas)
    // })
    run()
    setTimeout(() => onPlay(videoEl))
    }
  },
  data: () => ({
    ecosystem: [
      {
        text: "vuetify-loader",
        href: "https://github.com/vuetifyjs/vuetify-loader"
      },
      {
        text: "github",
        href: "https://github.com/vuetifyjs/vuetify"
      },
      {
        text: "awesome-vuetify",
        href: "https://github.com/vuetifyjs/awesome-vuetify"
      }
    ],
    importantLinks: [
      {
        text: "Documentation",
        href: "https://vuetifyjs.com"
      },
      {
        text: "Chat",
        href: "https://community.vuetifyjs.com"
      },
      {
        text: "Made with Vuetify",
        href: "https://madewithvuetifyjs.com"
      },
      {
        text: "Twitter",
        href: "https://twitter.com/vuetifyjs"
      },
      {
        text: "Articles",
        href: "https://medium.com/vuetify"
      }
    ],
    whatsNext: [
      {
        text: "Explore components",
        href: "https://vuetifyjs.com/components/api-explorer"
      },
      {
        text: "Select a layout",
        href: "https://vuetifyjs.com/layout/pre-defined"
      },
      {
        text: "Frequently Asked Questions",
        href: "https://vuetifyjs.com/getting-started/frequently-asked-questions"
      }
    ]
  })
};
</script>

<style scoped>
</style>
