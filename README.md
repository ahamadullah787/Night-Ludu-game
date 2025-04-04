# Night-Ludu-game
<!DOCTYPE html><html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultimate Ludo Game with Voice Chat</title>
    <script src="https://cdn.socket.io/4.0.1/socket.io.min.js"></script>
    <style>
        body { text-align: center; font-family: Arial, sans-serif; background: #f4f4f4; }
        .board { display: grid; grid-template-columns: repeat(5, 80px); grid-gap: 5px; margin: 20px auto; }
        .cell { width: 80px; height: 80px; background: lightblue; display: flex; align-items: center; justify-content: center; font-size: 18px; border: 2px solid black; transition: background 0.5s ease-in-out; }
        .safe { background: lightgreen !important; }
        .dice { margin: 20px; padding: 15px; font-size: 24px; background: green; color: white; cursor: pointer; border: none; border-radius: 5px; }
        .player { width: 30px; height: 30px; border-radius: 50%; position: absolute; transition: transform 0.5s ease-in-out; }
        .p1 { background: red; }
        .p2 { background: blue; }
        .p3 { background: yellow; }
        .p4 { background: green; }
        .game-area { position: relative; width: 400px; height: 400px; margin: auto; }
        .chat-box { width: 300px; height: 200px; border: 2px solid black; overflow-y: scroll; margin: 20px auto; padding: 10px; background: white; }
        .chat-input { width: 250px; padding: 5px; }
        .chat-button { padding: 6px; background: blue; color: white; border: none; cursor: pointer; }
        .voice-button { padding: 6px; background: red; color: white; border: none; cursor: pointer; }
    </style>
</head>
<body>
    <h1>Ultimate Ludo Game with Voice Chat</h1>
    <div class="game-area">
        <div class="board">
            <div class="cell safe" id="c1"></div>
            <div class="cell" id="c2"></div>
            <div class="cell" id="c3"></div>
            <div class="cell safe" id="c4"></div>
            <div class="cell" id="c5"></div>
        </div>
        <div class="player p1" id="player1" style="top:10px; left:10px;"></div>
        <div class="player p2" id="player2" style="top:10px; left:50px;"></div>
        <div class="player p3" id="player3" style="top:10px; left:90px;"></div>
        <div class="player p4" id="player4" style="top:10px; left:130px;"></div>
    </div>
    <button class="dice" onclick="rollDice()">Roll Dice</button>
    <p id="result">Roll the dice to start!</p><div class="chat-box" id="chat-box"></div>
<input type="text" id="chat-input" class="chat-input" placeholder="Type a message...">
<button class="chat-button" onclick="sendMessage()">Send</button>
<button class="voice-button" onclick="toggleVoiceChat()">Start Voice Chat</button>

<script>
    let socket = io('https://yourserver.com'); // Replace with your server URL
    let positions = { player1: 0, player2: 0, player3: 0, player4: 0 };
    let currentPlayer = 'player1';
    let safeZones = [0, 3];
    let audioStream;

    function rollDice() {
        let roll = Math.floor(Math.random() * 6) + 1;
        document.getElementById("result").innerText = `${currentPlayer} rolled: ` + roll;
        movePlayer(currentPlayer, roll);
        checkWinner();
        switchTurn();
        playSound('dice-roll.mp3');
    }
    
    function movePlayer(player, steps) {
        positions[player] += steps;
        if (positions[player] > 4) positions[player] = 4;
        document.getElementById(player).style.transform = `translateX(${positions[player] * 85}px)`;
        playSound('move.mp3');
    }
    
    function switchTurn() {
        if (currentPlayer === 'player1') currentPlayer = 'player2';
        else if (currentPlayer === 'player2') currentPlayer = 'player3';
        else if (currentPlayer === 'player3') currentPlayer = 'player4';
        else currentPlayer = 'player1';
        
        if (currentPlayer === 'player4') {
            setTimeout(() => rollDice(), 1000); // Simulate computer player's move
        }
    }
    
    function checkWinner() {
        for (let player in positions) {
            if (positions[player] === 4) {
                document.getElementById("result").innerText = `${player} wins the game!`;
                document.querySelector(".dice").disabled = true;
            }
        }
    }
    
    function sendMessage() {
        let message = document.getElementById("chat-input").value;
        if (message.trim() !== "") {
            let chatBox = document.getElementById("chat-box");
            let newMessage = document.createElement("p");
            newMessage.innerText = currentPlayer + ": " + message;
            chatBox.appendChild(newMessage);
            document.getElementById("chat-input").value = "";
            chatBox.scrollTop = chatBox.scrollHeight;
            socket.emit('chat message', message);
        }
    }
    
    function toggleVoiceChat() {
        navigator.mediaDevices.getUserMedia({ audio: true }).then(stream => {
            audioStream = stream;
            let audioTrack = stream.getAudioTracks()[0];
            socket.emit('voice start', audioTrack);
        });
    }
    
    function playSound(soundFile) {
        let audio = new Audio(soundFile);
        audio.play();
    }
</script>

</body>
</html>
