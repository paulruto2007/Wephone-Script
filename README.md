# Wephone
HTML (index.html)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Webphone</title>
    <style>
        #videoElement {
            width: 320px;
            height: 240px;
            border: 1px solid black;
        }
    </style>
</head>
<body>
    <h2>Webphone</h2>
    <button id="startCall">Start Call</button>
    <button id="endCall" disabled>End Call</button>
    <video id="videoElement" autoplay></video>

    <script src="app.js"></script>
</body>
</html>
JavaScript (app.js)
let localStream;
let peerConnection;
const startCallButton = document.getElementById('startCall');
const endCallButton = document.getElementById('endCall');
const videoElement = document.getElementById('videoElement');

// Get User Media (microphone + camera)
startCallButton.onclick = async () => {
    try {
        localStream = await navigator.mediaDevices.getUserMedia({ audio: true, video: true });
        videoElement.srcObject = localStream;

        // Set up WebRTC peer connection
        peerConnection = new RTCPeerConnection();
        localStream.getTracks().forEach(track => peerConnection.addTrack(track, localStream));

        // Set up WebSocket signaling (assumed signaling is set up)
        // signalingSocket.emit('startCall', {});

        // Add event listeners for the peer connection
        peerConnection.onicecandidate = event => {
            if (event.candidate) {
                // Send ICE candidate to the other peer via signaling server
            }
        };

        peerConnection.ontrack = event => {
            // Handle incoming media from remote peer
            videoElement.srcObject = event.streams[0];
        };

        // Create offer and send to other peer
        const offer = await peerConnection.createOffer();
        await peerConnection.setLocalDescription(offer);
        // signalingSocket.emit('offer', { offer });

        startCallButton.disabled = true;
        endCallButton.disabled = false;
    } catch (error) {
        console.error('Error starting call: ', error);
    }
};

// End call
endCallButton.onclick = () => {
    peerConnection.close();
    localStream.getTracks().forEach(track => track.stop());
    startCallButton.disabled = false;
    endCallButton.disabled = true;
};
