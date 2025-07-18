# Cantilever
from flask import Flask, render_template, request, redirect, url_for, session, flash
# from flask_sqlalchemy import SQLAlchemy # Uncomment if using a database
# from werkzeug.security import generate_password_hash, check_password_hash # For password hashing

app = Flask(__name__)
app.secret_key = 'your_super_secret_key' # Replace with a strong, random key
# app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///bookstore.db' # SQLite example
# db = SQLAlchemy(app)

# --- Simplified Mock Data (Replace with Database Models) ---
books = [
    {'id': 1, 'title': 'The Lord of the Rings', 'author': 'J.R.R. Tolkien', 'price': 25.00, 'image': 'lotr.jpg'},
    {'id': 2, 'title': 'Pride and Prejudice', 'author': 'Jane Austen', 'price': 15.50, 'image': 'pride.jpg'},
    {'id': 3, 'title': '1984', 'author': 'George Orwell', 'price': 12.00, 'image': '1984.jpg'},
]

users = {} # Mock user storage: {username: {'password': 'hashed_password'}}

# --- Database Models (Conceptual - Uncomment and define if using SQLAlchemy) ---
# class User(db.Model):
#     id = db.Column(db.Integer, primary_key=True)
#     username = db.Column(db.String(80), unique=True, nullable=False)
#     password_hash = db.Column(db.String(120), nullable=False)

# class Book(db.Model):
#     id = db.Column(db.Integer, primary_key=True)
#     title = db.Column(db.String(100), nullable=False)
#     author = db.Column(db.String(100), nullable=False)
#     price = db.Column(db.Float, nullable=False)
#     image_url = db.Column(db.String(200), nullable=True)

# --- Routes ---

@app.route('/')
def index():
    return render_template('index.html', books=books)

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        if username in users and users[username]['password'] == password: # Simplified check
            # if username in users and check_password_hash(users[username]['password_hash'], password): # Actual check
            session['logged_in'] = True
            session['username'] = username
            flash('Logged in successfully!', 'success')
            return redirect(url_for('index'))
        else:
            flash('Invalid username or password', 'danger')
    return render_template('login.html')

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        if username in users:
            flash('Username already exists', 'danger')
        else:
            users[username] = {'password': password} # Simplified
            # users[username] = {'password_hash': generate_password_hash(password)} # Actual hashing
            flash('Registration successful! Please log in.', 'success')
            return redirect(url_for('login'))
    return render_template('register.html')

@app.route('/logout')
def logout():
    session.pop('logged_in', None)
    session.pop('username', None)
    flash('You have been logged out.', 'info')
    return redirect(url_for('index'))

@app.route('/add_to_cart/<int:book_id>')
def add_to_cart(book_id):
    if 'cart' not in session:
        session['cart'] = {}
    book_found = next((book for book in books if book['id'] == book_id), None)
    if book_found:
        book_id_str = str(book_id) # Convert to string for session keys
        session['cart'][book_id_str] = session['cart'].get(book_id_str, 0) + 1
        flash(f'{book_found["title"]} added to cart!', 'success')
    else:
        flash('Book not found!', 'danger')
    return redirect(url_for('index'))

@app.route('/cart')
def view_cart():
    cart_items = []
    total_price = 0
    if 'cart' in session:
        for book_id_str, quantity in session['cart'].items():
            book_id = int(book_id_str)
            book = next((b for b in books if b['id'] == book_id), None)
            if book:
                cart_items.append({'book': book, 'quantity': quantity})
                total_price += book['price'] * quantity
    return render_template('cart.html', cart_items=cart_items, total_price=total_price)

@app.route('/update_cart/<int:book_id>', methods=['POST'])
def update_cart(book_id):
    if 'cart' in session:
        book_id_str = str(book_id)
        action = request.form.get('action')
        if action == 'increase':
            session['cart'][book_id_str] += 1
        elif action == 'decrease':
            session['cart'][book_id_str] -= 1
            if session['cart'][book_id_str] <= 0:
                session['cart'].pop(book_id_str)
        session.modified = True # Important for modifying mutable session objects
    return redirect(url_for('view_cart'))

@app.route('/remove_from_cart/<int:book_id>')
def remove_from_cart(book_id):
    if 'cart' in session:
        book_id_str = str(book_id)
        if book_id_str in session['cart']:
            session['cart'].pop(book_id_str)
            session.modified = True
            flash('Item removed from cart.', 'info')
    return redirect(url_for('view_cart'))

@app.route('/checkout')
def checkout():
    # This would involve collecting shipping info, selecting payment, etc.
    # For a real site, you'd integrate with a payment gateway here.
    if 'logged_in' not in session:
        flash('Please log in to checkout.', 'warning')
        return redirect(url_for('login'))
    
    if 'cart' not in session or not session['cart']:
        flash('Your cart is empty!', 'warning')
        return redirect(url_for('index'))

    # Simulate clearing the cart after "checkout"
    session.pop('cart', None)
    flash('Checkout successful! (This is a placeholder)', 'success')
    return redirect(url_for('index'))


if __name__ == '__main__':
    # db.create_all() # Uncomment to create database tables
    app.run(debug=True)
