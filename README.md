from flask import Flask, request, jsonify
import pandas as pd
import os

app = Flask(__name__)

# Directory to store uploaded files
UPLOAD_FOLDER = 'uploads'
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

@app.route('/upload', methods=['POST'])
def upload_receipt():
    if 'file' not in request.files:
        return jsonify({"error": "No file part"}), 400
    
    file = request.files['file']
    if file.filename == '':
        return jsonify({"error": "No selected file"}), 400
    
    # Save file temporarily
    file_path = os.path.join(UPLOAD_FOLDER, file.filename)
    file.save(file_path)
    
    # Extract receipt details (example: read from CSV/JSON/PDF)
    receipt_data = extract_receipt_details(file_path)
    
    # Save to accounting system (CSV example)
    save_to_accounting(receipt_data)
    
    return jsonify({"status": "success", "data": receipt_data}), 200

def extract_receipt_details(file_path):
    """Example: Read receipt data from a CSV file."""
    df = pd.read_csv(file_path)
    return {
        "vendor": df.iloc[0]["Vendor"],
        "amount": df.iloc[0]["Amount"],
        "date": df.iloc[0]["Date"]
    }

def save_to_accounting(data):
    """Save receipt data to CSV (example)."""
    df = pd.DataFrame([data])
    df.to_csv("accounting.csv", mode='a', header=not os.path.exists("accounting.csv"), index=False)

if __name__ == '__main__':
    app.run(debug=True)
