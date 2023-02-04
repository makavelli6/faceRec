<script>
  import * as faceapi from 'face-api.js';
  import { detectAllFaces } from 'face-api.js';
  import { onMount } from 'svelte';
  import Hls from 'hls.js';

  let webcam;
  let faceData;
  let video, canvas, snap, selfie, image;
  var webcamElement;

  // onMount(async ()=>{
  //   let script = document.createElement("script");
  //   script.src =  "https://cdn.jsdelivr.net/npm/hls.js@latest";
  //   document.head.append(script);

  //   script.onload = function () {
  //     if(Hls.isSupported()) {
  //       var hls = new Hls();
  //       hls.loadSource('rtsp://joshua:qwerty123@192.168.1.127:554/stream2');
  //       hls.attachMedia(video);
  //       hls.on(Hls.Events.MANIFEST_PARSED,function() {
  //         video.play();
  //         loadAI();
  //     });
  //   }}
   

  // })

  


  function init() {
     webcamElement = document.getElementById('webcam');
     var _canvas = window.document.getElementById('canvas');

     let All_mediaDevices=navigator.mediaDevices
      if (!All_mediaDevices || !All_mediaDevices.getUserMedia) {
        console.log("getUserMedia() not supported.");
        return;
      }

      navigator.mediaDevices.getUserMedia({
            video: true
         })
         .then(function(vidStream) {
            if ("srcObject" in video) {
              video.srcObject = vidStream;
            } else {
              video.src = window.URL.createObjectURL(vidStream);
            }
            video.onloadedmetadata = function(e) {
              video.play();
            };
            loadAI();
         })
         .catch(function(e) {
            console.log(e.name + ": " + e.message);
         });


    
		
  }

  const loadAI = () => {
		Promise.all([
			faceapi.nets.faceRecognitionNet.loadFromUri('/models'),
			faceapi.nets.faceLandmark68Net.loadFromUri('/models'),
			faceapi.nets.ssdMobilenetv1.loadFromUri('/models'),
			faceapi.nets.tinyFaceDetector.loadFromUri('/models'),

		]).then(async () => {
			console.log('models loaded');

       loadLabeled().then((lebalDescriptors)=>{
        const faceMatcher = new faceapi.FaceMatcher(lebalDescriptors, 0.6);
      checkVideo(faceMatcher);

      })
      
			//checkFace();
		});
	};

  const checkVideo  = (faceMatcher) =>{
    faceapi.matchDimensions(canvas, { width: 480,  height:480, });
    setInterval(async ()=>{
      const useTinyModel = true
      var cam = window.document.getElementById('webcam');

      //const detections = await faceapi.detectAllFaces(cam).withFaceLandmarks().withFaceDescriptors();

      const detections = await faceapi.detectAllFaces(video, new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks().withFaceDescriptors();

      const resizeDetections = faceapi.resizeResults(detections, { width: 480,  height:480, } );

      const results = resizeDetections.map(d => faceMatcher.findBestMatch(d.descriptor))
      canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height);

      results.forEach((result, i)=>{
        const box = resizeDetections[i].detection.box;
        const drawBox = new faceapi.draw.DrawBox(box, { label: result.toString()} )
        drawBox.draw(canvas);

      })
      
      // faceapi.draw.drawDetections(canvas, resizeDetections);

      // // faceapi.draw.drawFaceLandmarks(canvas, resizeDetections);
      console.log(detections.length)
    }, 200)

  }

  const setCanvas = () =>{


  }
  init();

  const loadLabeled  = ()=>{
    const labels = [ 'moha', 'musdafa', 'josh'];
    return Promise.all(
    labels.map(async label => {
      const descriptions = [];

      const img = await faceapi.fetchImage(`/images/${label}.jpeg`)
      const detections = await faceapi.detectSingleFace(img).withFaceLandmarks().withFaceDescriptor()
      descriptions.push(detections.descriptor);
      return new faceapi.LabeledFaceDescriptors(label, descriptions)
    })
  )
    
  }


</script>



<!-- svelte-ignore a11y-media-has-caption -->
<video  width="480"  height="480" bind:this={video}   id="webcam" playsinline />
<canvas width="480" height="480" id="canvas" bind:this={canvas}   />
<audio id="snapSound" bind:this={snap}  src="/audio/snap.mp3" preload="auto" />

<style>
 canvas{
  position: absolute;
  top:0; left:0;
 }
</style>