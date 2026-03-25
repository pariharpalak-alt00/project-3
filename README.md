from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import serialization, hashes
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

def generate_rsa_keys():
    private_key = rsa.generate_private_key(
        public_exponent=65537,
        key_size=2048
    )

    public_key = private_key.public_key()

    return private_key, public_key


def encrypt_rsa(public_key, message):
    return public_key.encrypt(
        message,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )


def decrypt_rsa(private_key, ciphertext):
    return private_key.decrypt(
        ciphertext,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )


def generate_aes_key():
    return os.urandom(32)


def encrypt_aes(key, plaintext):
    iv = os.urandom(16)

    cipher = Cipher(algorithms.AES(key), modes.CFB(iv))
    encryptor = cipher.encryptor()

    ciphertext = encryptor.update(plaintext) + encryptor.finalize()

    return iv + ciphertext


def decrypt_aes(key, ciphertext):
    iv = ciphertext[:16]
    data = ciphertext[16:]

    cipher = Cipher(algorithms.AES(key), modes.CFB(iv))
    decryptor = cipher.decryptor()

    return decryptor.update(data) + decryptor.finalize()



from flask import Flask, render_template
from flask_socketio import SocketIO, send


app = Flask(__name__)
socketio = SocketIO(app)

clients = {}

@app.route("/")
def index():
    return render_template("chat.html")

@socketio.on("connect")
def handle_connect():
    print("User Connected")


@socketio.on("public_keys")
def receive_key(data):
    clients[data["user"]] = data["key"]    

@socketio.on("message")
def handle_mesage(data):
    send(data, broadcast=True) 


if __name__ == "__main__":
    #socketio.run(app, host="0.0.0.0", port=8000)


const socket = io();

function sendMessage(){
    let msg = document.getElementById("message").value;
    socket.send({
        message: msg
    });
}

socket.on("message", function(data){
    let li = document.createElement("li");
    li.innerText = data.message;

    document.getElementById("chat").appendChild(li);
});


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
    <title>Secure Chat</title>
</head>
<body>
    <h2>Secure E2EE Chat</h2>
    <input id="message" placeholder="Enter message">
    <button onclick="sendMessage()">Send</button>

    <ul id="chat"></ul>

    <script src="https://cdn.socket.io/4.0.0/socket.io.min.js"></script>
    <script src="/static/client.js"></script>
    
</body>
</html>


    
