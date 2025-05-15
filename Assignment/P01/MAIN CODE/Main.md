from flask import Flask, jsonify, request
import json
import os

app = Flask(__name__)
DATA_FILE = 'products.json'

# Load data from JSON
def load_data():
    if not os.path.exists(DATA_FILE):
        return []
    with open(DATA_FILE, 'r') as file:
        return json.load(file)

# Save data to JSON
def save_data(data):
    with open(DATA_FILE, 'w') as file:
        json.dump(data, file, indent=4)

# GET all products
@app.route('/products', methods=['GET'])
def get_products():
    return jsonify(load_data())

# GET product by ID
@app.route('/products/<int:product_id>', methods=['GET'])
def get_product(product_id):
    products = load_data()
    for product in products:
        if product['id'] == product_id:
            return jsonify(product)
    return jsonify({'error': 'Product not found'}), 404

# CREATE new product
@app.route('/products', methods=['POST'])
def create_product():
    products = load_data()
    new_product = request.json
    new_product['id'] = max([p['id'] for p in products], default=0) + 1
    products.append(new_product)
    save_data(products)
    return jsonify(new_product), 201

# UPDATE product by ID
@app.route('/products/<int:product_id>', methods=['PUT'])
def update_product(product_id):
    products = load_data()
    for product in products:
        if product['id'] == product_id:
            updates = request.json
            product.update(updates)
            save_data(products)
            return jsonify(product)
    return jsonify({'error': 'Product not found'}), 404

# DELETE product by ID
@app.route('/products/<int:product_id>', methods=['DELETE'])
def delete_product(product_id):
    products = load_data()
    for i, product in enumerate(products):
        if product['id'] == product_id:
            deleted = products.pop(i)
            save_data(products)
            return jsonify(deleted)
    return jsonify({'error': 'Product not found'}), 404

if __name__ == '__main__':
    app.run(debug=True)

