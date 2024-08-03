# pywebview

# pywebview Communication between Javascript and Python

```bash
.
├── index.html
└── main.py
```

```py
# main.py
import webview
import time
import requests
import pickle
import os

class Api:
    def getMessage(self):
        return "Hello from the Python backend!"

    def process_input(self, input_text):
        time.sleep(3)
        return f"You entered: {input_text}"

    def display_file_content(self, file_content):
        return f"File content:\n\n{file_content}"

    def fetch_data(self):
        response = requests.get("https://httpbin.org/get")
        if response.status_code == 200:
            return response.json()
        else:
            return {"error": "Unable to fetch data"}

    def save_data(self, data):
        with open("data.pickle", "wb") as file:
            pickle.dump(data, file)
        return "Data saved successfully."

    def load_data(self):
        if os.path.exists("data.pickle"):
            with open("data.pickle", "rb") as file:
                data = pickle.load(file)
            return data
        else:
            return "No data found."


if __name__ == "__main__":
    api = Api()
    webview.create_window("Pywebview Example", "index.html", js_api=api, width=800, height=600)
    webview.start()
```

```html
<!-- index.html -->
 <!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pywebview Example</title>
</head>

<body>
    <h1>Hello, Pywebview!</h1>
    <p>This is a simple desktop application using Pywebview.</p>
</body>

<!-- ------------------------------------------------------------ -->

<button onclick="getMessageFromBackend()">Get Message</button>
<p id="message"></p>

<script>
    async function getMessageFromBackend() {
        const message = await window.pywebview.api.getMessage();
        document.getElementById('message').innerText = message;
    }
</script>

<!-- ------------------------------------------------------------ -->

<label for="user-input">Enter some text:</label>
<input type="text" id="user-input">
<button onclick="sendInputToBackend()">Send Input</button>
<p id="response"></p>

<script>
    async function sendInputToBackend() {
        const userInput = document.getElementById('user-input').value;
        const response = await window.pywebview.api.process_input(userInput);
        document.getElementById('response').innerText = response;
    }
</script>

<!-- ------------------------------------------------------------ -->

<label for="file-input">Choose a text file:</label>
<input type="file" id="file-input" accept=".txt">
<button onclick="sendFileToBackend()">Send File</button>
<p id="file-content"></p>

<script>
    async function sendFileToBackend() {
        const fileInput = document.getElementById('file-input');
        const file = fileInput.files[0];
        if (file) {
            const reader = new FileReader();
            reader.onload = async function (event) {
                const fileContent = event.target.result;
                // const response = await window.pywebview.api.display_file_content(fileContent);
                // document.getElementById('file-content').innerText = response;
                document.getElementById('file-content').innerText = fileContent;
            };
            reader.readAsText(file);
        } else {
            alert('Please select a file.');
        }
    }
</script>

<!-- ------------------------------------------------------------ -->

<button onclick="fetchDataFromBackend()">Fetch Data</button>
<pre id="fetched-data"></pre>

<script>
    async function fetchDataFromBackend() {
        const data = await window.pywebview.api.fetch_data();
        document.getElementById('fetched-data').innerText = JSON.stringify(data, null, 2);
    }
</script>

<!-- ------------------------------------------------------------ -->

<label for="data-input">Enter data to save:</label>
<input type="text" id="data-input">
<button onclick="saveDataToDisk()">Save Data</button>
<button onclick="loadDataFromDisk()">Load Data</button>
<p id="loaded-data"></p>

<script>
    async function saveDataToDisk() {
        const data = document.getElementById('data-input').value;
        const response = await window.pywebview.api.save_data(data);
        alert(response);
    }

    async function loadDataFromDisk() {
        const data = await window.pywebview.api.load_data();
        document.getElementById('loaded-data').innerText = data;
    }
</script>


</html>
```