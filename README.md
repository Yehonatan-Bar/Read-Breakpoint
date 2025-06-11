# Text Formatter Web App

A simple web application to format pasted text for better readability, especially for text with mixed newline characters.

## Features

*   Converts newline characters (including `\n` literals) into HTML line breaks (`<br>`).
*   Escapes HTML special characters to prevent rendering issues and for security.
*   Provides an easy-to-use interface to paste text and get the formatted version.
*   "Copy to clipboard" functionality for both formatted and plain text versions.
*   Auto-submits when you paste text.
*   Clicking the text area clears it and attempts to paste from the clipboard.

## How to Run

1.  **Clone the repository.**

2.  **Create a virtual environment (recommended):**
    ```bash
    # On macOS/Linux
    python3 -m venv venv
    source venv/bin/activate

    # On Windows
    python -m venv venv
    venv\Scripts\activate
    ```

3.  **Install the dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Run the application:**
    ```bash
    python app.py
    ```

5.  Open your browser and navigate to `http://127.0.0.1:5000`.

## How to Use

1.  Open the web application in your browser.
2.  Paste any text you want to format into the text area.
3.  The form will automatically submit, and the formatted text will appear below.
4.  Use the copy buttons to copy the formatted output to your clipboard. You can copy it with rich formatting or as plain text. 