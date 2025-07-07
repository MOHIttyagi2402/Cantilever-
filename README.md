# Cantilever
// src/App.tsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";

function App() {
  const [products, setProducts] = useState([]);
  const [query, setQuery] = useState('');
  const [cart, setCart] = useState([]);

  useEffect(() => {
    axios.get('/api/products').then((res) => setProducts(res.data));
  }, []);

  const handleAddToCart = (product) => {
    setCart([...cart, product]);
  };

  const handleRemoveFromCart = (productId) => {
    setCart(cart.filter((p) => p._id !== productId));
  };

  const filteredProducts = products.filter((product) =>
    product.name.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <div className="p-4 grid grid-cols-1 md:grid-cols-3 gap-4">
      <div className="md:col-span-2">
        <Input
          placeholder="Search products..."
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          className="mb-4"
        />
        <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4">
          {filteredProducts.map((product) => (
            <Card key={product._id} className="p-2">
              <CardContent>
                <img src={product.image} alt={product.name} className="w-full h-32 object-cover rounded" />
                <h2 className="text-lg font-bold mt-2">{product.name}</h2>
                <p className="text-sm text-gray-500">${product.price}</p>
                <Button onClick={() => handleAddToCart(product)} className="mt-2 w-full">Add to Cart</Button>
              </CardContent>
            </Card>
          ))}
        </div>
      </div>
      <div className="bg-white rounded-xl p-4 shadow-md">
        <h2 className="text-xl font-semibold mb-4">Cart</h2>
        {cart.map((item) => (
          <div key={item._id} className="flex justify-between items-center mb-2">
            <span>{item.name}</span>
            <Button size="sm" variant="destructive" onClick={() => handleRemoveFromCart(item._id)}>Remove</Button>
          </div>
        ))}
        <p className="mt-4 font-bold">Total: ${cart.reduce((acc, item) => acc + item.price, 0)}</p>
      </div>
    </div>
  );
}

export default App;

        if matching_books:
            print(f"\nFound {len(matching_books)} matching books:\n")
            for book in matching_books:
                print(f"Title: {book['title']}\nPrice: {book['price']}\nAvailability: {book['availability']}\n")
        else:
            print("No books found with that keyword.")
    else:
        print("No books retrieved from the website.")
   
