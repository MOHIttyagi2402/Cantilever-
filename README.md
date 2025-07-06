# Cantilever-
from bs4 import BeautifulSoup
import requests

def get_books(url):
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()  # Raise error for bad responses
    except requests.RequestException as e:
        print(f"Error fetching URL: {e}")
        return []

    soup = BeautifulSoup(response.text, 'html.parser')
    books = []

    for book in soup.select('article.product_pod'):
        # Safely extract title
        title_tag = book.h3.a
        title = title_tag['title'] if title_tag and 'title' in title_tag.attrs else 'No title'

        # Extract price and availability safely
        price_tag = book.select_one('.price_color')
        price = price_tag.text.strip() if price_tag else 'No price info'

        availability_tag = book.select_one('.availability')
        availability = availability_tag.text.strip() if availability_tag else 'No availability info'

        books.append({
            'title': title,
            'price': price,
            'availability': availability
        })

    return books

def search_books(books, query):
    query = query.lower()
    return [book for book in books if query in book['title'].lower()]

# Example usage
if __name__ == '__main__':
    url = 'http://books.toscrape.com/'
    all_books = get_books(url)

    if all_books:
        search_query = input("Enter keyword to search for a book title: ")
        matching_books = search_books(all_books, search_query)

        if matching_books:
            print(f"\nFound {len(matching_books)} matching books:\n")
            for book in matching_books:
                print(f"Title: {book['title']}\nPrice: {book['price']}\nAvailability: {book['availability']}\n")
        else:
            print("No books found with that keyword.")
    else:
        print("No books retrieved from the website.")
   
