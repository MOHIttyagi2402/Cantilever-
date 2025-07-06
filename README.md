# Cantilever-
from bs4 import BeautifulSoup
import requests

def get_books(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    books = []

    # Each book is in an article tag with class 'product_pod'
    for book in soup.select('article.product_pod'):
        title = book.h3.a['title']
        price = book.select_one('.price_color').text
        availability = book.select_one('.availability').text.strip()

        books.append({
            'title': title,
            'price': price,
            'availability': availability
        })

    return books

def search_books(books, query):
    query = query.lower()
    results = [book for book in books if query in book['title'].lower()]
    return results

# Example usage
url = 'http://books.toscrape.com/'
all_books = get_books(url)

# Prompt user for search keyword
search_query = input("Enter keyword to search for a book title: ")
matching_books = search_books(all_books, search_query)

# Display results
if matching_books:
    print(f"\nFound {len(matching_books)} matching books:\n")
    for book in matching_books:
        print(f"Title: {book['title']}\nPrice: {book['price']}\nAvailability: {book['availability']}\n")
else:
    print("No books found with that keyword.")

           
