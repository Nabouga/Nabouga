// WebSocket-Verbindung herstellen
const ws = new WebSocket('wss://gateway-ifs-antonhack');

// Nachrichtenwarteschlange
const messageQueue = [];
let waitingForAck = false;

// Nachricht senden (wartet auf ACK)
function sendMessage(message) {
    messageQueue.push(message);
    processQueue();
}

// Nachrichten in der Warteschlange verarbeiten
function processQueue() {
    if (!waitingForAck && messageQueue.length > 0) {
        const nextMessage = messageQueue.shift();
        ws.send(nextMessage);
        console.log('Gesendet:', nextMessage);
        waitingForAck = true;
    }
}

// Ereignis, wenn die Verbindung geöffnet wird
ws.addEventListener('open', () => {
    console.log('Verbindung geöffnet');

    // Nachrichten in die Warteschlange legen
    sendMessage(JSON.stringify({ event: '#handshake' }));
    sendMessage(JSON.stringify({ 
        event: '#login', 
        data: { "login-code": 'DEIN-ANMELDE CODE' } 
    }));
    sendMessage(JSON.stringify({ 
        event: '#hack', 
        data: { type: 'coins', value: 42 } 
    }));
});

// Ereignis, wenn eine Nachricht empfangen wird
ws.addEventListener('message', (message) => {
    console.log('Nachricht erhalten:', message.data);

    // Prüfen, ob ACK empfangen wurde
    try {
        const response = JSON.parse(message.data);
        if (response.event === '#ack' && response.data === null) {
            console.log('ACK erhalten');
            waitingForAck = false;
            processQueue(); // Nächste Nachricht senden
        }
    } catch (error) {
        console.error('Fehler beim Verarbeiten der Nachricht:', error);
    }
});

// Ereignis, wenn die Verbindung geschlossen wird
ws.addEventListener('close', () => {
    console.log('Verbindung geschlossen');
});

// Fehlerbehandlung
ws.addEventListener('error', (error) => {
    console.error('WebSocket-Fehler:', error);
});
