- 👋 Hi, I’m @09f5239
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
09f5239/09f5239 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
---><!DOCTYPE html>
<html>
<head>
   <meta charset="utf-8">
   <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
   <title>Arduino Gamepad Control</title>
   <meta name="description" content="">
   <meta name="viewport" content="width=device-width">
   <script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/3.7.1/jquery.min.js"></script>
</head>
<body>
 
<div id="gamepadPrompt"></div>
<div id="gamepadDisplay"></div>
   
<script>
var hasGP = false;
var repGP;
const arduinoIP = '192.168.254.200'; // Fixed IP address of the Arduino
var prevButtonState = [];
var prevAxisState = [];
 
function canGame() {
   return "getGamepads" in navigator;
}
 
function sendRequest(url) {
   fetch(url)
       .then(response => response.text())
       .then(data => console.log(data))
       .catch(error => console.error('Error:', error));
}
 
function reportOnGamepad() {
   var gp = navigator.getGamepads()[0];
   var html = "";
   var dataChanged = false;
 
   
 
   for (var i = 0; i < gp.buttons.length; i++) {
       var buttonPressed = gp.buttons[i].pressed;
       html += (buttonPressed ? "1" : "0") ;
 
       if (buttonPressed !== prevButtonState[i]) {
           prevButtonState[i] = buttonPressed;
           dataChanged = true;
       }
   }
 
   
           var axis1 = Math.round((gp.axes[0] + 1) * 127.5+100);
           var axis2 = Math.round((gp.axes[1] + 1) * 127.5+100);
           html += "Stick  1 :" + axis1 + "," + axis2 ;
           var axis3 = Math.round((gp.axes[2] + 1) * 127.5+100);
           var axis4 = Math.round((gp.axes[3] + 1) * 127.5+100);
           html += "Stick  2:" + axis3 + "," + axis4 ;
 
       if (axis1 !== prevAxisState[0] || axis2 !== prevAxisState[1] || axis3 !== prevAxisState[2] || axis4 !== prevAxisState[3]) {
           prevAxisState[0] = axis1;
           prevAxisState[1] = axis2;
           prevAxisState[2] = axis3;
           prevAxisState[3] = axis4;
           dataChanged = true;
       }
  // }
 
   $("#gamepadDisplay").html(html);
 
   // Send the html variable to the Arduino only if data changed
   if (dataChanged) {
       sendRequest(`http://${arduinoIP}/update?data=${encodeURIComponent(html)}`);
   }
}
 
$(document).ready(function() {
   if (canGame()) {
       var prompt = "To begin using your gamepad, connect it and press any button!";
       $("#gamepadPrompt").text(prompt);
 
       $(window).on("gamepadconnected", function() {
           hasGP = true;
           $("#gamepadPrompt").html("Gamepad connected!");
           console.log("connection event");
 
           // Initialize the previous states
           var gp = navigator.getGamepads()[0];
           prevButtonState = new Array(gp.buttons.length).fill(false);
           prevAxisState = new Array(gp.axes.length).fill(0);
 
           repGP = window.setInterval(reportOnGamepad, 100);
       });
 
       $(window).on("gamepaddisconnected", function() {
           console.log("disconnection event");
           $("#gamepadPrompt").text(prompt);
           window.clearInterval(repGP);
       });
 
       // Setup an interval for Chrome
       var checkGP = window.setInterval(function() {
           console.log('checkGP');
           if (navigator.getGamepads()[0]) {
               if (!hasGP) $(window).trigger("gamepadconnected");
               window.clearInterval(checkGP);
           }
       }, 300);
   }
});
</script>
</body>
</html>
