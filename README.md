# Cantilever
from flask import Flask, render_template, request
from sqlalchemy import create_engine, Column, String, Text
from sqlalchemy.orm import sessionmaker, declarative_base
import pandas as pd
import random
import os

# --- Database Setup ---
# Define the base for declarative models
Base = declarative_base()

# Define the Product model
class Product(Base):
    __tablename__ = 'products'
    id = Column(String, primary_key=True)
    title = Column(String, nullable=False)
    price = Column(String)
    rating = Column(String)
    description = Column(Text)

    def __repr__(self):
        return f"<Product(title='{self.title}', price='{self.price}')>"

# Database URL: Use an environment variable for production (e.g., for Heroku)
# For local development, it defaults to a local SQLite file.
DATABASE_URL = os.environ.get('DATABASE_URL', 'sqlite:///ecommerce.db')
engine = create_engine(DATABASE_URL)
Session = sessionmaker(bind=engine)

app = Flask(__name__)

# --- Data Generation/Scraping Simulation ---
def generate_sample_products(num_products=20):
    """Generates sample product data instead of live scraping."""
    sample_data = []
    adjectives = ["Stylish", "Comfortable", "Durable", "Modern", "Classic", "Trendy"]
    nouns = ["Sneakers", "T-Shirt", "Jeans", "Jacket", "Dress", "Socks", "Watch", "Bag"]

    for i in range(num_products):
        title = f"{random.choice(adjectives)} {random.choice(nouns)} {i+1}"
        price = f"${random.randint(20, 200)}.99"
        rating = f"{random.choice(['3', '3.5', '4', '4.5', '5'])}"
        description = f"This is a fantastic {title.lower()} with excellent quality and a great fit. Perfect for all occasions."
        product_id = title.replace(" ", "_").lower() + f"_{i}" # Simple unique ID

        sample_data.append({
            'id': product_id,
            'title': title,
            'price': price,
            'rating': rating,
            'description': description
        })
    return sample_data

def populate_database_with_sample_data():
    """Populates the database with sample products."""
    session = Session()
    try:
        # Check if the database is already populated
        if session.query(Product).count() == 0:
            print("Populating database with sample data...")
            sample_products = generate_sample_products()
            for item in sample_products:
                new_product = Product(**item)
                session.add(new_product)
            session.commit()
            print("Database populated successfully.")
        else:
            print("Database already contains data, skipping population.")
    except Exception as e:
        session.rollback()
        print(f"Error populating database: {e}")
    finally:
        session.close()

# --- Flask Routes ---
@app.route('/')
def index():
    session = Session()
    all_products = session.query(Product).limit(12).all() # Show first 12 products
    session.close()
    return render_template('index.html', products=all_products)

@app.route('/search')
def search():
    query = request.args.get('query', '').strip()
    results = []
    if query:
        session = Session()
        # Simple text search across title and description (case-insensitive)
        results = session.query(Product).filter(
            (Product.title.ilike(f'%{query}%')) |
            (Product.description.ilike(f'%{query}%'))
        ).all()
        session.close()
    return render_template('search_results.html', query=query, results=results)

# --- Main execution ---
if __name__ == '__main__':
    # Create tables if they don't exist
    Base.metadata.create_all(engine)
    # Populate with sample data on first run
    populate_database_with_sample_data()

    # Run the Flask app
    # Use debug=True for local development, set to False for production
    app.run(debug=True)

