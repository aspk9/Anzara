// Install dependencies before running: npm install next react react-dom tailwindcss axios recoil next-auth

// pages/index.js
import { useEffect, useState } from "react";
import axios from "axios";
import { useRecoilState } from "recoil";
import { cartState } from "../state/cart";
import { useSession, signIn, signOut } from "next-auth/react";

export default function Home() {
  const [products, setProducts] = useState([]);
  const [cart, setCart] = useRecoilState(cartState);
  const { data: session } = useSession();

  useEffect(() => {
    axios.get("https://fakestoreapi.com/products").then((res) => setProducts(res.data));
  }, []);

  const addToCart = (product) => {
    setCart([...cart, product]);
  };

  return (
    <div className="container mx-auto p-4">
      <h1 className="text-3xl font-bold text-center">E-Commerce Store</h1>
      <div className="flex justify-end">
        {session ? (
          <button onClick={() => signOut()} className="bg-red-500 text-white px-4 py-2 rounded">Sign Out</button>
        ) : (
          <button onClick={() => signIn()} className="bg-green-500 text-white px-4 py-2 rounded">Sign In</button>
        )}
      </div>
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mt-4">
        {products.map((product) => (
          <div key={product.id} className="border p-4 rounded shadow-md">
            <img src={product.image} alt={product.title} className="w-full h-48 object-cover" />
            <h2 className="text-lg font-semibold mt-2">{product.title}</h2>
            <p className="text-gray-700">${product.price}</p>
            <button
              className="bg-blue-500 text-white px-4 py-2 mt-2 rounded"
              onClick={() => addToCart(product)}
            >
              Add to Cart
            </button>
          </div>
        ))}
      </div>
    </div>
  );
}

// state/cart.js
import { atom } from "recoil";
export const cartState = atom({
  key: "cartState",
  default: [],
});

// pages/cart.js
import { useRecoilValue } from "recoil";
import { cartState } from "../state/cart";

export default function Cart() {
  const cart = useRecoilValue(cartState);
  return (
    <div className="container mx-auto p-4">
      <h1 className="text-3xl font-bold text-center">Shopping Cart</h1>
      {cart.length === 0 ? (
        <p className="text-center mt-4">Your cart is empty</p>
      ) : (
        <ul className="mt-4">
          {cart.map((item, index) => (
            <li key={index} className="border-b py-2">{item.title} - ${item.price}</li>
          ))}
        </ul>
      )}
    </div>
  );
}

// pages/api/auth/[...nextauth].js
import NextAuth from "next-auth";
import Providers from "next-auth/providers";

export default NextAuth({
  providers: [
    Providers.Credentials({
      name: "Credentials",
      credentials: {
        username: { label: "Username", type: "text" },
        password: { label: "Password", type: "password" }
      },
      async authorize(credentials) {
        if (credentials.username === "admin" && credentials.password === "password") {
          return { id: 1, name: "Admin" };
        }
        return null;
      }
    })
  ]
});
