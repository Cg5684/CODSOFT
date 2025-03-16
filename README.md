pip install flask requests
from flask import Flask, request, jsonify
pip install flask requests
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

# Define the weather API key and URL (Replace with your actual API key)
API_KEY = 'your_openweather_api_key'
WEATHER_URL = 'http://api.openweathermap.org/data/2.5/weather'

# Rule-based responses
def get_response(user_input):
    user_input = user_input.lower()
    
    if 'hello' in user_input or 'hi' in user_input:
        return "Hi! How can I assist you today?"
    
    elif 'weather' in user_input:
        # User asks for the weather
        return "Sure! Please tell me the city name for the weather update."
    
    elif 'bye' in user_input or 'goodbye' in user_input:
        return "Goodbye! Have a great day!"
    
    elif 'how are you' in user_input:
        return "I'm just a bot, but I'm doing great! How can I help you?"
    
    elif 'time' in user_input:
        return "The current time is: " + time.strftime('%Y-%m-%d %H:%M:%S')
    
    else:
        return "Sorry, I didn't understand that. Could you please clarify?"

# Fetch weather information based on city
def fetch_weather(city):
    params = {
        'q': city,
        'appid': API_KEY,
        'units': 'metric'
    }
    try:
        response = requests.get(WEATHER_URL, params=params)
        weather_data = response.json()
        
        if weather_data['cod'] == 200:
            # Extract relevant weather info
            city_name = weather_data['name']
            temp = weather_data['main']['temp']
            description = weather_data['weather'][0]['description']
            return f"The weather in {city_name} is {temp}°C with {description}."
        else:
            return "Sorry, I couldn't fetch the weather for that city."
    except requests.exceptions.RequestException as e:
        return "Sorry, there was an error fetching the weather data."

# Route for chatbot to handle user input
@app.route('/chat', methods=['POST'])
def chat():
    user_input = request.json.get('user_input')
    
    if not user_input:
        return jsonify({'response': 'Please provide a valid input.'}), 400

    if 'weather' in user_input.lower():
        # Check if the user specified a city
        if 'in' in user_input.lower():
            city = user_input.split('in')[-1].strip()
            weather_response = fetch_weather(city)
            return jsonify({'response': weather_response})
        else:
            return jsonify({'response': 'Please specify a city to get the weather.'})

    response = get_response(user_input)
    return jsonify({'response': response})

if __name__ == '__main__':
    app.run(debug=True)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rule-Based Chatbot</title>
    <style>
        #chat-container {
            width: 400px;
            height: 500px;
            border: 1px solid #ccc;
            padding: 20px;
            overflow-y: scroll;
            margin-top: 50px;
        }
        #input-container {
            margin-top: 10px;
        }
        input {
            width: 80%;
            padding: 10px;
        }
        button {
            width: 18%;
            padding: 10px;
        }
    </style>
</head>
<body>

    <div id="chat-container"></div>

    <div id="input-container">
        <input type="text" id="user-input" placeholder="Type your message..." />
        <button onclick="sendMessage()">Send</button>
    </div>

    <script>
        // Function to display the messages in the chat container
        function displayMessage(sender, message) {
            const chatContainer = document.getElementById("chat-container");
            const messageDiv = document.createElement("div");
            messageDiv.textContent = `${sender}: ${message}`;
            chatContainer.appendChild(messageDiv);
            chatContainer.scrollTop = chatContainer.scrollHeight; // Scroll to bottom
        }

        // Function to send user input to the server
        function sendMessage() {
            const userInput = document.getElementById("user-input").value;
            if (!userInput) return;

            // Display user message
            displayMessage("You", userInput);
            
            // Send request to Flask server
            fetch('http://127.0.0.1:5000/chat', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ user_input: userInput })
            })
            .then(response => response.json())
            .then(data => {
                const botResponse = data.response;
                // Display bot response
                displayMessage("Bot", botResponse);
            })
            .catch(error => {
                console.error('Error:', error);
            });

            // Clear the input field
            document.getElementById("user-input").value = '';
        }
    </script>

</body>
</html>
