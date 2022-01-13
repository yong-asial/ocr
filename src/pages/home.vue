<template>
  <f7-page name="home">
    <f7-navbar>
      <f7-nav-title>OCR</f7-nav-title>
    </f7-navbar>
    <f7-block>
      <f7-row class="justify-content-center text-align-center">
        <f-card>
          <div class="camera" v-show="!isMobile()">
            <video ref="videoRef" playsinline autoplay></video>
          </div>
          <div class="camera" v-show="isMobile()">
            <img ref="imageRef" src="static/icons/favicon.png">
          </div>
          <div class="camera">
            <canvas ref="canvasRef"></canvas>
          </div>
        </f-card>
      </f7-row>
      <f7-row class="justify-content-center">
        <f7-col>
          <f7-button fill raised @click="scan">Scan</f7-button>
        </f7-col>
      </f7-row>
      <f7-row class="justify-content-center">
        <div class="result">
          {{ result }}
        </div>
      </f7-row>
    </f7-block>
  </f7-page>
</template>

<script>
import { onMounted, ref } from 'vue';
import axios from 'axios';

export default {
  setup() {
    // variable and DOM
    const videoRef = ref(null);
    const canvasRef = ref(null);
    const imageRef = ref(null);
    const result = ref('Scan a photo!');
    const w = 224; // width for image, video, canvas elements
    const h = 224; // height for image, video, canvas elements
    const API_KEY = null; // Input your API Key for Cloud Version API
    const ENDPOINT = 'https://vision.googleapis.com/v1/images:annotate?key=' + API_KEY;
    const REQUEST_OPTION = {
      requests: [
        {
          image: {
            content: null,
          },
          features: [
            {
              type: 'TEXT_DETECTION'
            },
          ],
        },
      ],
    };
    let imageLoaded = false;

    onMounted(async () => {
      // check API key
      if (!API_KEY) {
        result.value = 'Please input the Cloud Vision API key';
        return;
      }
      // set canvas width/heigh
      canvasRef.value.width = w;
      canvasRef.value.height = h;
      if (!isMobile() && navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        // start UserMedia web api on supported browser
        startUserMedia();
      } else {
        // when image is loaded, call ocr function
        imageRef.value.addEventListener('load', e => {
          if (imageLoaded) {
            ocr(imageRef.value);
          }
        });
      }
    });

    const scan = () => {
      if (isMobile()) {
        // start camera plugin for ocr
        navigator.camera.getPicture(successCameraCallback, failureCameraCallback, {
            quality : 50,
            destinationType: Camera.DestinationType.DATA_URL,
            targetWidth: w,
            targetHeight: h
        });
      } else {
        // start ocr directly from UserMedia
        ocr(videoRef.value);
      }
    };

    const ocr = (image) => {
      // draw image to canvas
      canvasRef.value.getContext('2d').drawImage(image, 0, 0, canvasRef.value.width, canvasRef.value.height);
      // convert to base64
      const base64Data = canvasRef.value.toDataURL().replace('data:image/png;base64,', '');
      // recognize
      recongize(base64Data);
    };

    const recongize = (base64Data) => {
      REQUEST_OPTION.requests[0].image.content = base64Data;
      axios.post(ENDPOINT, REQUEST_OPTION).then(response => {
        const annotations = response.data.responses[0].textAnnotations
        if (!annotations) {
          result.value = 'Could not detect any text from the image';
        } else {
          // draw bounding boxes and show result
          drawBoundingBoxes(annotations);
        }
      }).catch(error => {
        result.value = error;
      })
    };

    /**
     * Web functions
     */

    const startUserMedia = () => {
      const constraint = {
        audio: false,
        video: {
          facingMode: 'environment',
          width: { min: w, ideal: w, max: w, exact: w },
          height: { min: h, ideal: h, max: h, exact: h },
        }
      };

      const handleSuccess = (stream) => {
        videoRef.value.srcObject = stream;
      }

      const handleError = (error) => {
        result.value = error.message;
      }

      navigator.mediaDevices.getUserMedia(constraint).then(handleSuccess).catch(handleError);
    };

    /** 
     * Mobile functions
     */

    const successCameraCallback = (data) => {
      imageRef.value.src = 'data:image/jpeg;base64,' + data;
      imageLoaded = true;
    };

    const failureCameraCallback = (errMessage) => {
      result.value = errMessage;
      imageLoaded = false;
    };

    /**
     * Helper functions
     */

    const drawBoundingBoxes = (boxes) => {
      if (!boxes) return
      // show result
      result.value = boxes[0].description;
      // draw bounding boxes
      for (let i = 1; i < boxes.length; i++) {
        drawBoundingBox(boxes[i].boundingPoly.vertices);
      }
    };

    const drawBoundingBox = (vertices) => {
      if (!vertices) return
      // get canvas context
      const ctx = canvasRef.value.getContext('2d');
      // drawing each vertice
      ctx.beginPath();
      ctx.lineWidth = '2';
      ctx.strokeStyle = 'green';
      ctx.moveTo(vertices[0].x, vertices[0].y);
      for (let i = 1; i < vertices.length; i++) {
        ctx.lineTo(vertices[i].x, vertices[i].y);
      }
      ctx.closePath();
      ctx.stroke();
    };

    const isMobile = () => {
      return window.cordova && (isAndroid() || isIos());
    };

    const isAndroid = () => {
      const userAgent = navigator.userAgent || navigator.vendor || window.opera;
      if (/android/i.test(userAgent)) return true;
      return false;
    };

    const isIos = () => {
      const userAgent = navigator.userAgent || navigator.vendor || window.opera;
      if (/iPad|iPhone|iPod/i.test(userAgent)) return true;
      return false;
    };

    return {
      videoRef,
      canvasRef,
      imageRef,
      scan,
      isMobile,
      result
    }
  }
}
</script>

<style scoped>

  .block {
    margin-top: 5px;
  }

  div.camera {
    height: 100%;
    width: 100%;
    overflow: hidden;
    margin-bottom: 5px;
  }

  div.result {
    margin-top: 5px;
    white-space: pre-line;
    border-style: solid;
  }

  canvas {
    border-style: solid;
  }

  img {
    height: 224px;
    width: 224px;
  }

</style>