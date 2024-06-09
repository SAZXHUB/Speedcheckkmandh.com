
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Real-time Speed Tracker</title>
<style>
    body {
        font-family: 'Arial', sans-serif;
        text-align: center;
        margin-top: 50px;
        background-color: #f4f4f9;
        color: #333;
    }

    h1 {
        color: #4a90e2;
    }

    #speed {
        font-size: 2.5em;
        margin: 20px 0;
        color: #333;
        background: #fff;
        padding: 20px;
        border-radius: 10px;
        box-shadow: 0 0 10px rgba(0,0,0,0.1);
        display: inline-block;
    }

    #error {
        color: red;
        margin-top: 20px;
    }

    .settings {
        margin-top: 20px;
        display: inline-block;
        background: #fff;
        padding: 20px;
        border-radius: 10px;
        box-shadow: 0 0 10px rgba(0,0,0,0.1);
        text-align: left;
    }

    .settings label {
        font-weight: bold;
    }

    .settings input, .settings select, .settings button {
        display: block;
        margin-top: 10px;
        width: 100%;
        padding: 10px;
        border-radius: 5px;
        border: 1px solid #ccc;
    }

    .settings button {
        background-color: #4a90e2;
        color: white;
        border: none;
        cursor: pointer;
        transition: background-color 0.3s;
    }

    .settings button:hover {
        background-color: #357ABD;
    }

    .hidden {
        display: none;
    }
</style>
</head>
<body>
    <h1>Real-time Speed Tracker</h1>
    <div id="speed">Speed: 0 km/h</div>
    <p id="error"></p>
    
    <div class="settings" id="locationSettings">
        <button onclick="requestLocation()">Allow Location Access</button>
    </div>

    <div class="settings hidden" id="speedSettings">
        <label for="updateInterval">Update Interval (ms):</label>
        <input type="number" id="updateInterval" value="1000">
        <label for="minutes">Minutes:</label>
        <input type="number" id="minutes" value="0" min="0">
        <label for="seconds">Seconds:</label>
        <input type="number" id="seconds" value="10" min="10">
        <label for="milliseconds">Milliseconds:</label>
        <input type="number" id="milliseconds" value="100" min="100">
        <button onclick="applySettings()">Apply Settings</button>
    </div>

    <script>
        let watchID;
        let updateInterval = 1000; // Default update interval
        let lastSpeed = 0;
        const errorElement = document.getElementById('error');
        const locationSettings = document.getElementById('locationSettings');
        const speedSettings = document.getElementById('speedSettings');

        function startGeolocation() {
            if (navigator.geolocation) {
                watchID = navigator.geolocation.watchPosition(
                    function(position) {
                        const speed = position.coords.speed;
                        if (speed !== null) {
                            // Convert speed from m/s to km/h
                            const speedKmH = speed * 3.6;
                            document.getElementById('speed').textContent = `Speed: ${speedKmH.toFixed(2)} km/h`;
                            if (speedKmH >= 100 && speedKmH % 100 === 0 && speedKmH !== lastSpeed) {
                                const audio = new Audio('alert.wav');
                                audio.play();
                            }
                            lastSpeed = speedKmH;
                        } else {
                            document.getElementById('speed').textContent = `Speed: 0 km/h`;
                        }
                    },
                    function(error) {
                        switch(error.code) {
                            case error.PERMISSION_DENIED:
                                errorElement.textContent = "User denied the request for Geolocation. Please enable location services in your browser settings.";
                                break;
                            case error.POSITION_UNAVAILABLE:
                                errorElement.textContent = "Location information is unavailable.";
                                break;
                            case error.TIMEOUT:
                                errorElement.textContent = "The request to get user location timed out.";
                                break;
                            case error.UNKNOWN_ERROR:
                                errorElement.textContent = "An unknown error occurred.";
                                break;
                        }
                    },
                    {
                        enableHighAccuracy: true,
                        maximumAge: 0,
                        timeout: getTimeout()
                    }
                );
            } else {
                alert("Geolocation is not supported by this browser.");
            }
        }

        function requestLocation() {
            navigator.geolocation.getCurrentPosition(
                function(position) {
                    // Permission granted
                    startGeolocation();
                    locationSettings.classList.add('hidden');
                    speedSettings.classList.remove('hidden');
                },
                function(error) {
                    // Permission denied
                    errorElement.textContent = "User denied the request for Geolocation. Please enable location services in your browser settings.";
                }
            );
        }

        function applySettings() {
            updateInterval = parseInt(document.getElementById('updateInterval').value);
            const minutes = parseInt(document.getElementById('minutes').value);
            const seconds = parseInt(document.getElementById('seconds').value);
            const milliseconds = parseInt(document.getElementById('milliseconds').value);

            // Convert time settings to milliseconds
            const totalTime = (minutes * 60000) + (seconds * 1000) + milliseconds;

            // Set the new update interval
            if (totalTime >= 60000 && totalTime % 100 === 0) {
                // Ensure minutes are at least 1 and seconds are at least 10
                clearInterval(watchID);
                watchID = setInterval(startGeolocation, totalTime);
            } else {
                alert("Please enter valid time settings (minutes >= 1, seconds >= 10, milliseconds >= 100).");
            }
        }

        function getTimeout() {
            // Implementation remains the same
        }

        document.addEventListener('DOMContentLoaded', requestLocation);
    </script>
</body>
</html>
