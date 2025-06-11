from flask import Flask, request, render_template_string
import html
from markupsafe import Markup

app = Flask(__name__)

# Basic HTML template with CSS for styling
HTML_TEMPLATE = r"""
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>Text Formatter</title>
    <style>
        body {
            font-family: sans-serif;
            line-height: 1.6;
            margin: 20px;
            background-color: #f4f4f4;
            color: #333;
        }
        .container {
            max-width: 800px;
            margin: auto;
            background: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #555;
            text-align: center;
        }
        textarea {
            width: 95%;
            min-height: 80px;
            padding: 10px;
            margin-bottom: 15px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-family: monospace; /* Use monospace for code-like text */
            font-size: 1rem;
        }
        .output {
            margin-top: 20px;
            padding: 15px;
            background-color: #f5f5f5;
            border: 1px solid #ddd;
            border-radius: 4px;
            word-wrap: break-word; /* Breaks long words */
            font-family: monospace; /* Use monospace for code-like text */
            white-space: pre-wrap;  /* Preserves line breaks and spacing */
            line-height: 1.5;       /* Improved line spacing */
        }
        /* Style for the output content */
        .output-content {
            margin: 0;
            padding: 0;
        }
        .copy-button {
            background-color: #007bff;
            color: white;
            border: none;
            padding: 4px 8px;
            border-radius: 50%;
            cursor: pointer;
            font-size: 0.7rem;
            position: absolute;
            width: 30px;
            height: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: background-color 0.3s;
            z-index: 10;
        }
        .copy-button.top {
            top: 10px;
            right: 10px;
        }
        .copy-button.bottom {
            bottom: 10px;
            right: 10px;
        }
        .copy-button:hover {
            background-color: #0056b3;
        }
        .copy-button:active {
            background-color: #004085;
        }
        .copy-button.success {
            background-color: #28a745;
        }
        .output-wrapper {
            position: relative;
        }
        h2 {
            margin-top: 30px;
            color: #555;
            border-bottom: 1px solid #eee;
            padding-bottom: 5px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Paste Your Text</h1>
        <form method="post" id="textForm">
            <textarea name="text_input" id="textInput" placeholder="Paste text here...">{{ text_input if text_input }}</textarea>
        </form>

        {% if formatted_text %}
        <h2>Formatted Output:</h2>
        <div class="output-wrapper">
            <button class="copy-button top" onclick="copyViaSelection(this)" title="Copy content with formatting">📋</button>
            <div class="output">
                <div class="output-content" id="formattedContent">{{ formatted_text }}</div>
            </div>
            <button class="copy-button bottom" onclick="copyViaSelection(this)" title="Copy content with formatting">📋</button>
        </div>
        <div style="margin-top: 10px; font-size: 0.9em; color: #666;">
            <span>Need plain text? Try: </span>
            <button onclick="copyFormattedText(this)" style="background:none; border:1px solid #ccc; padding:2px 8px; border-radius:3px; cursor:pointer; font-size:0.8em;">Plain Text Copy</button>
            <span> or manually select text above</span>
        </div>
        <div style="margin-top: 15px; padding: 10px; background-color: #f8f9fa; border: 1px solid #e9ecef; border-radius: 4px; font-size: 0.85em;">
            <strong>Debug Tools:</strong><br>
            <button onclick="debugCopyFunction()" style="margin: 2px; padding: 4px 8px; font-size: 0.8em; background: #17a2b8; color: white; border: none; border-radius: 3px; cursor: pointer;">Debug Info</button>
            <button onclick="testSimpleCopy()" style="margin: 2px; padding: 4px 8px; font-size: 0.8em; background: #6c757d; color: white; border: none; border-radius: 3px; cursor: pointer;">Test Simple Copy</button>
            <span style="color: #6c757d;">Check browser console (F12) for detailed logs</span>
        </div>
        {% endif %}
    </div>

    <script>
        // Function to copy formatted text content
        function copyFormattedText(button) {
            console.log('=== Copy button clicked ===');
            console.log('Button:', button);
            
            const formattedContent = document.getElementById('formattedContent');
            console.log('formattedContent element:', formattedContent);
            
            if (!formattedContent) {
                console.error('Formatted content element not found');
                return;
            }
            
            console.log('formattedContent.innerHTML:', formattedContent.innerHTML);
            console.log('formattedContent.textContent:', formattedContent.textContent);
            console.log('formattedContent.innerText:', formattedContent.innerText);
            
            // Get the text content more directly
            let textToCopy = formattedContent.textContent || formattedContent.innerText;
            console.log('Initial textToCopy (textContent/innerText):', textToCopy);
            
            // If textContent/innerText doesn't work, try extracting from innerHTML
            if (!textToCopy || textToCopy.trim() === '') {
                console.log('textContent/innerText failed, trying innerHTML extraction');
                textToCopy = formattedContent.innerHTML;
                console.log('innerHTML before processing:', textToCopy);
                
                // Convert <br> tags back to newlines for proper copying
                textToCopy = textToCopy.replace(/<br\s*\/?>/gi, '\n');
                console.log('After <br> replacement:', textToCopy);
                
                // Remove any other HTML tags
                textToCopy = textToCopy.replace(/<[^>]*>/g, '');
                console.log('After HTML tag removal:', textToCopy);
                
                // Decode HTML entities
                const tempDiv = document.createElement('div');
                tempDiv.innerHTML = textToCopy;
                textToCopy = tempDiv.textContent || tempDiv.innerText || '';
                console.log('After HTML entity decoding:', textToCopy);
            }
            
            console.log('Final text to copy:', JSON.stringify(textToCopy));
            console.log('Text length:', textToCopy.length);
            
            // Check clipboard API availability
            console.log('navigator.clipboard available:', !!navigator.clipboard);
            console.log('navigator.clipboard.writeText available:', !!(navigator.clipboard && navigator.clipboard.writeText));
            
            // Copy to clipboard
            if (navigator.clipboard && navigator.clipboard.writeText) {
                console.log('Using modern clipboard API');
                navigator.clipboard.writeText(textToCopy).then(function() {
                    console.log('✅ Clipboard API copy successful');
                    showCopySuccess(button);
                }).catch(function(err) {
                    console.error('❌ Clipboard API failed:', err);
                    console.log('Falling back to legacy method');
                    fallbackCopy(textToCopy, button);
                });
            } else {
                console.log('Clipboard API not available, using fallback method');
                fallbackCopy(textToCopy, button);
            }
        }
        
        // Fallback copy method for older browsers
        function fallbackCopy(text, button) {
            console.log('=== Fallback copy method ===');
            console.log('Text to copy via fallback:', JSON.stringify(text));
            
            const textArea = document.createElement('textarea');
            textArea.value = text;
            textArea.style.position = 'fixed';
            textArea.style.left = '-999999px';
            textArea.style.top = '-999999px';
            textArea.style.opacity = '0';
            
            console.log('Created textarea element:', textArea);
            console.log('Textarea value:', textArea.value);
            
            document.body.appendChild(textArea);
            console.log('Textarea appended to body');
            
            textArea.focus();
            console.log('Textarea focused');
            
            textArea.select();
            console.log('Textarea selected');
            
            try {
                console.log('Attempting document.execCommand("copy")');
                const successful = document.execCommand('copy');
                console.log('execCommand result:', successful);
                
                if (successful) {
                    console.log('✅ Fallback copy successful');
                    showCopySuccess(button);
                } else {
                    console.error('❌ Fallback copy failed - execCommand returned false');
                    alert('Copy failed. Please select the text and copy manually (Ctrl+C).');
                }
            } catch (err) {
                console.error('❌ Fallback copy exception:', err);
                alert('Copy failed. Please select the text and copy manually (Ctrl+C).');
            } finally {
                document.body.removeChild(textArea);
                console.log('Textarea removed from body');
            }
        }
        
        // Show visual feedback for successful copy
        function showCopySuccess(button) {
            console.log('=== Showing copy success feedback ===');
            const originalText = button.innerHTML;
            console.log('Original button content:', originalText);
            
            button.innerHTML = '✅';
            button.classList.add('success');
            console.log('Button updated to show success');
            
            setTimeout(function() {
                button.innerHTML = originalText;
                button.classList.remove('success');
                console.log('Button reverted to original state');
            }, 2000);
        }

        // Alternative copy function that creates a temporary selection
        function copyViaSelection(button) {
            console.log('=== Alternative copy via selection ===');
            const formattedContent = document.getElementById('formattedContent');
            console.log('formattedContent for selection:', formattedContent);
            
            if (!formattedContent) {
                console.error('formattedContent not found for selection');
                return;
            }

            try {
                console.log('Creating range and selection');
                // Create a range and select the content
                const range = document.createRange();
                range.selectNodeContents(formattedContent);
                console.log('Range created:', range);
                
                const selection = window.getSelection();
                selection.removeAllRanges();
                selection.addRange(range);
                console.log('Content selected');
                
                // Copy the selected content
                console.log('Attempting execCommand copy on selection');
                const successful = document.execCommand('copy');
                console.log('Selection copy result:', successful);
                
                // Clear the selection
                selection.removeAllRanges();
                console.log('Selection cleared');
                
                if (successful) {
                    console.log('✅ Selection copy successful');
                    showCopySuccess(button);
                } else {
                    console.error('❌ Selection copy failed');
                    // Try the text extraction method as fallback
                    console.log('Trying text extraction method as fallback');
                    copyFormattedText(button);
                }
            } catch (err) {
                console.error('❌ Selection copy exception:', err);
                // Try the text extraction method as fallback
                console.log('Trying text extraction method as fallback after exception');
                copyFormattedText(button);
            }
        }

        // Debug function to show copy-related information
        function debugCopyFunction() {
            console.log('=== DEBUG COPY FUNCTION ===');
            
            const formattedContent = document.getElementById('formattedContent');
            console.log('formattedContent element exists:', !!formattedContent);
            
            if (formattedContent) {
                console.log('Element details:');
                console.log('- tagName:', formattedContent.tagName);
                console.log('- className:', formattedContent.className);
                console.log('- id:', formattedContent.id);
                console.log('- innerHTML length:', formattedContent.innerHTML.length);
                console.log('- textContent length:', (formattedContent.textContent || '').length);
                console.log('- innerText length:', (formattedContent.innerText || '').length);
                console.log('- First 100 chars of innerHTML:', formattedContent.innerHTML.substring(0, 100));
                console.log('- First 100 chars of textContent:', (formattedContent.textContent || '').substring(0, 100));
            }
            
            console.log('Browser capabilities:');
            console.log('- navigator.clipboard exists:', !!navigator.clipboard);
            console.log('- navigator.clipboard.writeText exists:', !!(navigator.clipboard && navigator.clipboard.writeText));
            console.log('- document.execCommand exists:', !!document.execCommand);
            console.log('- window.getSelection exists:', !!window.getSelection);
            
            console.log('Browser info:');
            console.log('- User agent:', navigator.userAgent);
            console.log('- Protocol:', window.location.protocol);
            console.log('- Is HTTPS:', window.location.protocol === 'https:');
            
            // Try to detect clipboard permission
            if (navigator.permissions) {
                navigator.permissions.query({name: 'clipboard-write'}).then(function(result) {
                    console.log('Clipboard write permission:', result.state);
                }).catch(function(err) {
                    console.log('Could not check clipboard permission:', err);
                });
            }
            
            alert('Debug info logged to console. Open browser console (F12) to see details.');
        }
        
        // Simple test copy function
        function testSimpleCopy() {
            console.log('=== TEST SIMPLE COPY ===');
            const testText = 'This is a simple test text for copying functionality.';
            console.log('Test text:', testText);
            
            if (navigator.clipboard && navigator.clipboard.writeText) {
                console.log('Testing modern clipboard API...');
                navigator.clipboard.writeText(testText).then(function() {
                    console.log('✅ Simple clipboard test successful');
                    alert('✅ Simple clipboard test successful! Check if this text was copied: "' + testText + '"');
                }).catch(function(err) {
                    console.error('❌ Simple clipboard test failed:', err);
                    alert('❌ Simple clipboard test failed. Error: ' + err.message);
                });
            } else {
                console.log('Testing fallback method...');
                const textArea = document.createElement('textarea');
                textArea.value = testText;
                textArea.style.position = 'fixed';
                textArea.style.left = '-999999px';
                textArea.style.top = '-999999px';
                textArea.style.opacity = '0';
                
                document.body.appendChild(textArea);
                textArea.focus();
                textArea.select();
                
                try {
                    const successful = document.execCommand('copy');
                    document.body.removeChild(textArea);
                    
                    if (successful) {
                        console.log('✅ Simple fallback test successful');
                        alert('✅ Simple fallback test successful! Check if this text was copied: "' + testText + '"');
                    } else {
                        console.error('❌ Simple fallback test failed');
                        alert('❌ Simple fallback test failed - execCommand returned false');
                    }
                } catch (err) {
                    document.body.removeChild(textArea);
                    console.error('❌ Simple fallback test exception:', err);
                    alert('❌ Simple fallback test failed with exception: ' + err.message);
                }
            }
        }

        document.addEventListener('DOMContentLoaded', function() {
            console.log('=== DOM Content Loaded ===');
            
            const textInput = document.getElementById('textInput');
            const form = document.getElementById('textForm');
            const formattedContent = document.getElementById('formattedContent');
            
            console.log('textInput:', textInput);
            console.log('form:', form);
            console.log('formattedContent:', formattedContent);
            
            // Auto-submit form when text is pasted
            textInput.addEventListener('paste', function() {
                console.log('Paste event detected');
                // Use setTimeout to ensure the pasted content is available
                setTimeout(function() {
                    if (textInput.value.trim()) {
                        console.log('Submitting form after paste');
                        form.submit();
                    }
                }, 100);
            });

            // Clear content and paste when clicked
            textInput.addEventListener('click', async function() {
                console.log('TextInput clicked - clearing content');
                // Clear the current content
                textInput.value = '';
                
                try {
                    // Try to read from clipboard
                    if (navigator.clipboard && navigator.clipboard.readText) {
                        console.log('Attempting to read from clipboard');
                        const clipboardText = await navigator.clipboard.readText();
                        console.log('Clipboard text:', clipboardText);
                        if (clipboardText.trim()) {
                            textInput.value = clipboardText;
                            // Trigger form submission after pasting
                            setTimeout(function() {
                                console.log('Auto-submitting form with clipboard content');
                                form.submit();
                            }, 100);
                        }
                    } else {
                        console.log('Clipboard read API not available');
                        // Fallback: Focus the textarea and suggest manual paste
                        textInput.focus();
                        textInput.placeholder = "Content cleared. Please paste your text here (Ctrl+V)...";
                    }
                } catch (error) {
                    console.log('Clipboard access failed:', error);
                    // If clipboard access fails (common in some browsers/contexts)
                    textInput.focus();
                    textInput.placeholder = "Content cleared. Please paste your text here (Ctrl+V)...";
                }
            });

            // Reset placeholder when user starts typing
            textInput.addEventListener('input', function() {
                if (textInput.placeholder !== "Paste text here...") {
                    textInput.placeholder = "Paste text here...";
                }
            });
        });
    </script>
</body>
</html>
"""

@app.route('/', methods=['GET', 'POST'])
def index():
    text_input = ""
    formatted_text = ""
    if request.method == 'POST':
        text_input = request.form.get('text_input', '')
        # First, convert any literal "\n" sequences into real newlines so they are handled uniformly
        text_processed = text_input.replace('\\n', '\n')

        # 1. Escape HTML special characters for security
        escaped_text = html.escape(text_processed)

        # 2. Replace newline characters with <br> tags for display
        #    This now covers both real newlines and the ones that were originally escaped.
        formatted_text = escaped_text.replace('\n', '<br>')

    # Render the template, passing the original input and the formatted output
    # Using render_template_string for simplicity as the template is small
    # Mark the formatted_text as safe because we manually handled escaping and added <br>
    return render_template_string(HTML_TEMPLATE, text_input=text_input, formatted_text=Markup(formatted_text))

if __name__ == '__main__':
    # Making it accessible on the network uncomment the line below
    # app.run(debug=True, host='0.0.0.0')
    app.run(debug=True) # Runs on localhost by default 
