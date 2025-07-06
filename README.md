# Cantilever-
import requests
from bs4 import BeautifulSoup

BASE_URL = "http://books.toscrape.com/catalogue/page-1.html"

def get_books(url):
    response = requests.get(url)
    if response.status_code != 200:
        print(f"Failed to retrieve page: {response.status_code}")
        return []

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

if __name__ == '__main__':
    books = get_books(BASE_URL)
    for idx, book in enumerate(books, 1):
        print(f"{idx}. {book['title']} - {book['price']} ({book['availability']})")
