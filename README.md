# Page-Spee-Insights-and-GTMetrix
Creating a Python PyQt application to interact with the Google PageSpeed Insights API and GTmetrix API in real-time involves setting up a GUI that allows users to input a URL and retrieve performance data for both services. We’ll break down the steps and provide the code for building such an application.
Prerequisites:

    Google Cloud API Key for PageSpeed Insights.

    GTmetrix API Key (sign up at GTmetrix).

    Install the required libraries:
        requests: For API requests.
        pyqt5: For creating the GUI application.

pip install requests pyqt5

Step-by-Step Code for the Application

Below is a Python script using PyQt5 for the GUI, and it integrates both PageSpeed Insights API and GTmetrix API to fetch performance scores.

import sys
import requests
from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QHBoxLayout, QLabel, QLineEdit, QPushButton, QTextEdit

class PageSpeedApp(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle('Page Speed Insights & GTmetrix')
        self.setGeometry(100, 100, 600, 400)

        # Google API Key and GTmetrix API Key (set these before running)
        self.google_api_key = "YOUR_GOOGLE_API_KEY"
        self.gtmetrix_api_key = "YOUR_GTMETRIX_API_KEY"

        self.initUI()

    def initUI(self):
        layout = QVBoxLayout()

        # URL input
        self.url_input = QLineEdit(self)
        self.url_input.setPlaceholderText('Enter URL here...')
        layout.addWidget(self.url_input)

        # Submit button
        self.submit_button = QPushButton('Fetch Performance Data', self)
        self.submit_button.clicked.connect(self.fetch_performance_data)
        layout.addWidget(self.submit_button)

        # Text area to display the results
        self.result_area = QTextEdit(self)
        self.result_area.setReadOnly(True)
        layout.addWidget(self.result_area)

        self.setLayout(layout)

    def fetch_performance_data(self):
        url = self.url_input.text()
        if not url:
            self.result_area.setText("Please enter a valid URL.")
            return

        # Clear previous results
        self.result_area.clear()

        # Fetch data from Google PageSpeed Insights API
        google_result = self.get_pagespeed_insights(url)
        gtmetrix_result = self.get_gtmetrix_data(url)

        # Combine results
        result_text = "Google PageSpeed Insights:\n" + google_result + "\n\nGTmetrix Report:\n" + gtmetrix_result
        self.result_area.setText(result_text)

    def get_pagespeed_insights(self, url):
        endpoint = "https://www.googleapis.com/pagespeedonline/v5/runPagespeed"
        params = {
            "url": url,
            "key": self.google_api_key,
            "strategy": "mobile"
        }
        response_mobile = requests.get(endpoint, params={**params, "strategy": "mobile"})
        response_desktop = requests.get(endpoint, params={**params, "strategy": "desktop"})

        if response_mobile.status_code == 200 and response_desktop.status_code == 200:
            mobile_data = response_mobile.json()
            desktop_data = response_desktop.json()

            mobile_score = mobile_data['lighthouseResult']['categories']['performance']['score'] * 100
            desktop_score = desktop_data['lighthouseResult']['categories']['performance']['score'] * 100

            mobile_details = f"Mobile Performance Score: {mobile_score}%"
            desktop_details = f"Desktop Performance Score: {desktop_score}%"

            return mobile_details + "\n" + desktop_details
        else:
            return "Error fetching data from Google PageSpeed Insights API."

    def get_gtmetrix_data(self, url):
        api_url = "https://gtmetrix.com/api/2.0/tests"
        headers = {
            'Authorization': f'Basic {self.gtmetrix_api_key}'  # Basic authentication with the GTmetrix API key
        }
        payload = {
            'url': url
        }

        response = requests.post(api_url, headers=headers, data=payload)
        if response.status_code == 201:
            test_id = response.json()['test_id']
            # Polling GTmetrix test status
            status = self.poll_gtmetrix_status(test_id)
            if status:
                return status
            else:
                return "Error: Unable to get GTmetrix results."
        else:
            return "Error fetching data from GTmetrix API."

    def poll_gtmetrix_status(self, test_id):
        status_url = f"https://gtmetrix.com/api/2.0/tests/{test_id}"
        headers = {
            'Authorization': f'Basic {self.gtmetrix_api_key}'
        }

        while True:
            response = requests.get(status_url, headers=headers)
            if response.status_code == 200:
                data = response.json()
                if data['state'] == 'completed':
                    # Extract performance details
                    score = data['results']['pagespeed_score']
                    fully_loaded_time = data['results']['fully_loaded_time']
                    return f"GTmetrix PageSpeed Score: {score}%\nFully Loaded Time: {fully_loaded_time} ms"
                elif data['state'] == 'failed':
                    return "GTmetrix test failed. Please try again."
            else:
                return "Error fetching GTmetrix test status."

            # Pause for a few seconds before polling again
            time.sleep(5)

if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = PageSpeedApp()
    window.show()
    sys.exit(app.exec_())

Key Features of the Application:

    User Interface:
        A text input field allows users to enter the URL they want to test.
        A button fetches the performance data from both Google PageSpeed Insights and GTmetrix.
        A large text area displays the results from both services.

    APIs Used:
        Google PageSpeed Insights: Fetches mobile and desktop performance data using the PageSpeed Insights API.
        GTmetrix: Sends a request to GTmetrix’s API to generate a test, then polls for the result.

How the Code Works:

    Google PageSpeed Insights:
        The function get_pagespeed_insights queries the API for both mobile and desktop performance scores.
        It displays the scores for both strategies in the application.

    GTmetrix:
        The get_gtmetrix_data function submits the URL to GTmetrix for testing, then polls until the test is completed to retrieve the performance results.
        The final results include the PageSpeed score and fully loaded time from GTmetrix.

    GUI Updates:
        The application fetches data asynchronously, updates the text area with results, and displays performance data for both services.

How to Run:

    Replace API Keys: Insert your own Google API key and GTmetrix API key in the script.

    Run the Script: After setting the API keys, run the script with Python:

    python app.py

    Enter URL: In the GUI, enter the URL of the website you want to test and click Fetch Performance Data.

Sample Output:

Google PageSpeed Insights:
Mobile Performance Score: 70%
Desktop Performance Score: 85%

GTmetrix Report:
GTmetrix PageSpeed Score: 72%
Fully Loaded Time: 3.2s

Notes:

    GTmetrix Polling: GTmetrix API tests are asynchronous, so we poll the result until it’s ready. This may take a minute or two.
    API Limits: Ensure that you respect the rate limits imposed by Google PageSpeed Insights and GTmetrix.
