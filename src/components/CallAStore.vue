<template>
  <div class="wrapper">
    <h3>Store 123</h3>

    <div class="my-box">
      <h4>My feeds</h4>
      <div id="me"></div>
    </div>
    <h4>Remote Feeds :</h4>
    <div id="remote-container">
    </div>
  </div>
</template>

<script>
import AgoraRTC from 'agora-rtc-sdk';


    // Defines a client for RTC
const client = AgoraRTC.createClient({
    mode: 'live',
    codec: "h264"
});

let handleFail = function(err){
    console.log("Error : ",err);
}

// let remoteContainer= document.getElementById("remote-container");

function addVideoStream(streamId){
    let streamDiv=document.createElement("div"); // Create a new div for every stream
    streamDiv.id='div_' + streamId;                       // Assigning id to div
    streamDiv.style.transform="rotateY(180deg)";
    if (!document.getElementById(streamDiv.id)) {
      window.remoteContainer.appendChild(streamDiv);  
    }
    // Takes care of lateral inversion (mirror image)
    // Add new div to container
}
/**
 * @name removeVideoStream
 * @param evt - Remove event
 * @description Helper function to remove the video stream from "remote-container"
 */
function removeVideoStream (evt) {
    let stream = evt.stream;
    stream.stop();
    let remDiv=document.getElementById(stream.getId());
    if (remDiv) {
      remDiv.parentNode.removeChild(remDiv);
      console.log("Remote stream is removed " + stream.getId());
    }
}

client.init("e9bad95a5d674f86902ec2ece9b6f654",() => console.log("AgoraRTC client initialized") ,handleFail);


// Title, userid
client.join(null, 'Call a store' , 'store1234' , (uid)=>{
    
  // Stream object associated with your web cam is initialized
  let localStream = AgoraRTC.createStream({
      streamID: uid,
      audio: false,
      video: true,
      screen: false
  });
  // this.setState({localStream});

  // Associates the stream to the client
  localStream.init(function() {
      
      //Plays the localVideo
      localStream.play('me');

      //Publishes the stream to the channegel
      client.publish(localStream, handleFail);
      //When a stream is added to a channel
      client.on('stream-added', function (evt) {
          client.subscribe(evt.stream, handleFail);
      });
      //When you subscribe to a stream
      client.on('stream-subscribed', function (evt) {
        let stream = evt.stream;
        addVideoStream(stream.getId());
        stream.play('div_' + stream.getId());
        // addCanvas(stream.getId());
      });
      //When a person is removed from the stream
      client.on('stream-removed',removeVideoStream);

      client.on('peer-leave',removeVideoStream);
  },handleFail);

},handleFail);
export default {
  name: 'SocialThreads',
  props: {
    msg: String
  },
  computed: {

  },
  data() {
    return {
      store_number: '',
      canShowFeeds: false,
      remoteContainer: window.document.getElementById('remote-container')
    }
  },
  methods: {

  },
  mounted() {
    //fetch
    window.remoteContainer = window.document.getElementById('remote-container');
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

.my-box {
  margin: 0px auto;
}
#me {
  width: 200px;
  height: 200px;
  margin: auto;
}
#me video{
    height: auto;
    position: relative !important;
}
#remote-container{
    display: flex;
}
#remote-container video{
    height: auto;
    position: relative !important;
}
</style>
