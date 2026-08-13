Conversational Search for an eCommerce Application

A mobile eCommerce application where the primary way to shop is by typing what you want in plain English—for example:

"Show me running shoes under ₹5,000 suitable for beginners."

The application converts the natural-language query into a structured shopping intent, ranks products from the catalog using a deterministic scoring system, and returns matching products with a short "Why this matches" explanation.

The application is built using React Native (Expo) for the frontend and FastAPI + OpenAI for the backend.

Table of Contents
Demo Query
Features
Architecture
Request Flow
Project Structure
Tech Stack
Setup Instructions
API Reference
Design Notes
Known Limitations
Demo Query
Example

Show me running shoes under ₹5,000 suitable for beginners

The application parses the query into a structured intent:

{
  "category": "Footwear",
  "subcategory": "Running Shoes",
  "max_price": 5000,
  "audience": "beginner"
}

The catalog is then scored against this intent, and the highest-ranking products are returned.

Each product includes a "Why this matches" explanation based on the attributes that matched the user's request.

Features
🗣️ Conversational Search

Users can search for products using natural language instead of traditional filters.

Example:

"Show me wireless headphones under ₹3,000 with good battery life."

🧠 AI Intent Parsing

The OpenAI API extracts structured information from the user's query, including:

Category
Subcategory
Price range
Brand
Target audience
Desired features
Keywords
💡 "Why This Matches"

Each product includes a short explanation describing why it matches the user's requirements.

For example:

"Cushioned sole and lightweight construction make this suitable for beginners, while the ₹3,499 price is comfortably below your ₹5,000 budget."

🧮 Deterministic Ranking

Products are ranked using a transparent scoring function rather than relying on the LLM to choose the products.

The ranking considers:

Price fit
Category match
Subcategory match
Brand match
Audience match
Keyword overlap
Feature overlap

This makes the ranking fast, predictable, and reproducible.

🗂️ Browse View

When there is no active search query, users can browse products using category chips displayed on the home screen.

❤️ Wishlist

Users can add or remove products from their wishlist directly from:

Product cards
Product detail screen

A dedicated Wishlist tab displays saved products and includes a live badge.

🛒 Shopping Cart

The cart supports:

Adding products
Increasing/decreasing quantity
Removing products
Live subtotal calculation
Cart item count badge
📦 Product Details

Each product detail screen contains:

Product image
Product title
Price
Description
Features
Tags
Add-to-cart button
💳 Mock Checkout

The checkout flow allows users to enter an address and select a payment method:

Cash on Delivery
UPI
Card

The application then displays a mock order confirmation.

🔄 Graceful Degradation

If an OpenAI API key is not configured, the application does not crash.

Instead, the backend falls back to basic keyword and category matching.

Architecture
┌────────────────────────────┐
│     React Native (Expo)    │
│                            │
│   expo-router screens      │
└─────────────┬──────────────┘
              │
              │ HTTPS / JSON
              ▼
┌────────────────────────────┐
│       FastAPI Backend      │
│                            │
│  Search • Catalog • Cart   │
│  Wishlist • Checkout       │
│  Ranking + LLM Integration │
└─────────────┬──────────────┘
              │
              ▼
┌────────────────────────────┐
│       OpenAI API           │
│                            │
│ Intent Parsing + Reasons   │
└────────────────────────────┘
Frontend

The frontend is built using React Native with Expo SDK 54.

It uses:

Expo Router for file-based navigation
React Native components for the UI
AsyncStorage for local session persistence
REST APIs for communicating with the backend

The backend URL is configured using:

EXPO_PUBLIC_BACKEND_URL=http://localhost:8000
Backend

The backend is a single FastAPI application.

It handles:

Conversational search
Product browsing
Product details
Cart operations
Wishlist operations
Checkout
Product ranking
OpenAI API calls

All API endpoints use the /api prefix.

AI Layer

The OpenAI API is called during conversational search for two purposes:

Convert the user's natural-language query into structured intent.
Generate a search summary and a "Why this matches" explanation for the returned products.

The LLM does not determine the ranking.

Ranking

Product ranking is performed deterministically by the backend's _score_product function.

This keeps product selection independent of the LLM and makes the system more predictable.

Data Store

The application currently uses an in-memory data store implemented in:

backend/memory_db.py

The catalog contains 40 seeded products across categories such as:

Footwear
Electronics
Apparel
Home & Kitchen
Sports
Beauty
Books
Toys
Health
Accessories

All prices are represented in INR.

No external database server is required.

Note: Data is reset whenever the backend is restarted.

Request Flow

When a user enters:

"Show me running shoes under ₹5,000 suitable for beginners"

the following process occurs:

User enters query
       │
       ▼
POST /api/search
       │
       ▼
LLM Call #1
Query → Structured Intent
       │
       ▼
Score every catalog product
       │
       ▼
Select top N candidates
       │
       ▼
LLM Call #2
Candidates + Intent
       │
       ▼
Generate summary + reasons
       │
       ▼
Return search response
       │
       ▼
React Native renders results
       │
       ▼
"Why this matches" shown for each product
Project Structure
conv_ecommerce/
│
├── backend/
│   ├── server.py
│   │   └── FastAPI routes, intent extraction,
│   │       product scoring, and LLM integration
│   │
│   ├── catalog.py
│   │   └── Seeded 40-product catalog
│   │
│   ├── memory_db.py
│   │   └── In-memory data store
│   │
│   ├── requirements.txt
│   ├── .env
│   │   └── OPENAI_API_KEY
│   │       OPENAI_MODEL
│   │
│   └── tests/
│       └── backend_test.py
│
├── frontend/
│   ├── app/
│   │   ├── (tabs)/
│   │   │   ├── Search/Home
│   │   │   ├── Wishlist
│   │   │   └── Cart
│   │   │
│   │   ├── product/
│   │   │   └── [id].tsx
│   │   │
│   │   └── checkout.tsx
│   │
│   ├── src/
│   │   ├── components/
│   │   │   └── ProductCard
│   │   │
│   │   ├── context/
│   │   │   └── AppContext
│   │   │
│   │   ├── theme.ts
│   │   │
│   │   └── utils/
│   │       └── storage/
│   │
│   ├── .env
│   │   └── EXPO_PUBLIC_BACKEND_URL
│   │
│   └── package.json
│
├── memory/
│   └── PRD.md
│
├── LOCAL_SETUP.md
└── README.md
Tech Stack
AI Service
Technology	Purpose
OpenAI API	Natural-language intent parsing and explanation generation
OpenAI Python SDK	Backend integration with OpenAI
GPT-4o-mini	Default configurable model

The model can be configured using:

OPENAI_MODEL=gpt-4o-mini
Backend
Technology	Purpose
Python	Backend programming language
FastAPI	REST API framework
Uvicorn	ASGI server
Pydantic	Request/response validation
python-dotenv	Environment variable management
OpenAI SDK	LLM integration

See backend/requirements.txt for the exact package versions.

Frontend
Technology	Purpose
React Native	Mobile application framework
Expo SDK 54	Development and runtime environment
Expo Router	File-based navigation
expo-image	Image rendering
expo-linear-gradient	Gradient UI elements
expo-blur	Blur effects
expo-haptics	Haptic feedback
expo-symbols	Symbol support
@expo/vector-icons	Icons
AsyncStorage	Local session persistence
React Native Gesture Handler	Gesture support
React Native Reanimated	Animations
React Native Worklets	Animation/runtime support
React Native Screens	Navigation optimization
React Native Safe Area Context	Safe-area handling
WebView	Embedded web content
Expo Web Browser	Browser integration
Expo Linking	Deep linking
date-fns / dayjs	Date formatting

No paid or proprietary UI kit is used. The application uses custom styling defined in:

frontend/src/theme.ts
Setup Instructions
Prerequisites

Make sure the following are installed:

Python 3.10+
Node.js 18+
Yarn
Expo Go (optional, for physical-device testing)
OpenAI API key (optional)

Install Yarn if necessary:

npm install -g yarn
1. Backend Setup

Navigate to the backend directory:

cd backend

Create a virtual environment:

macOS/Linux
python3 -m venv venv
source venv/bin/activate
Windows
python -m venv venv
venv\Scripts\activate

Install dependencies:

pip install -r requirements.txt
2. Configure Environment Variables

Create:

backend/.env

Add:

OPENAI_API_KEY=sk-your-api-key
OPENAI_MODEL=gpt-4o-mini

If you don't have an OpenAI API key, you can leave OPENAI_API_KEY empty. The application will use its fallback keyword/category search.

3. Start the Backend

Run:

python server.py

Alternatively:

uvicorn server:app --reload

The backend will be available at:

http://localhost:8000

Health check:

http://localhost:8000/api/
Frontend Setup

Open a new terminal and navigate to the frontend:

cd frontend

Install dependencies:

yarn install

Start Expo:

yarn start

You can then:

Press w to run the web version.
Scan the QR code using Expo Go on your phone.
Physical Device Setup

If you are testing on a physical phone, localhost will refer to the phone itself rather than your computer.

Therefore, update:

EXPO_PUBLIC_BACKEND_URL=http://YOUR_LAN_IP:8000

For example:

EXPO_PUBLIC_BACKEND_URL=http://192.168.1.5:8000

Find your computer's local IP address using:

Windows
ipconfig
macOS/Linux
ifconfig

Make sure your phone and computer are connected to the same Wi-Fi network.

Running Tests

Navigate to the backend:

cd backend

Run:

pytest

The backend test suite is located at:

backend/tests/backend_test.py
API Reference

All API endpoints are prefixed with:

/api
Method	Endpoint	Description
GET	/api/	Health check and product count
GET	/api/products	Browse products
GET	/api/products/{id}	Get product details
GET	/api/categories	List product categories
GET	/api/search/suggestions	Get example search suggestions
POST	/api/search	Conversational product search
GET	/api/cart/{session_id}	Get cart
POST	/api/cart/add	Add product to cart
POST	/api/cart/update	Update product quantity
POST	/api/cart/remove	Remove product from cart
GET	/api/wishlist/{session_id}	Get wishlist
POST	/api/wishlist/toggle	Add/remove product from wishlist
POST	/api/checkout	Create a mocked order
Search API Example
Request
curl -X POST http://localhost:8000/api/search \
  -H "Content-Type: application/json" \
  -d '{"query":"running shoes under 5000 for beginners"}'
Example Response
{
  "session_id": "example-session-id",
  "query": "running shoes under 5000 for beginners",
  "summary": "Found several beginner-friendly running shoes under ₹5,000.",
  "intent": {
    "category": "Footwear",
    "subcategory": "Running Shoes",
    "max_price": 5000,
    "audience": "beginner"
  },
  "results": [
    {
      "product": {
        "id": "product-id",
        "title": "Example Running Shoe",
        "price": 3499
      },
      "reason": "The cushioned sole and lightweight construction make it suitable for beginners, and the ₹3,499 price is below your ₹5,000 budget.",
      "score": 12.5
    }
  ]
}
Design Notes
Visual Design

The application follows a clean, iOS-inspired visual style.

Primary design characteristics:

Moss/sage green brand color: #768A44
Cream surface: #FDFDF9
Coral secondary accent: #C15B3D
System font
Rounded product cards
Bottom-tab navigation
Navigation

The application uses three primary tabs:

┌───────────────┬───────────────┬───────────────┐
│    Search     │   Wishlist    │     Cart      │
└───────────────┴───────────────┴───────────────┘

The Wishlist and Cart tabs display live item-count badges.

Session Management

A client-generated session ID is stored locally using AsyncStorage.

The session ID is used to associate:

Cart items
Wishlist items
Search-related state

with a particular client session without requiring user authentication.

Known Limitations
1. OpenAI API Key Is Optional

Without an OpenAI API key, the application falls back to basic keyword and category matching.

The application continues to function, but it loses:

Natural-language intent extraction
AI-generated search summaries
AI-generated "Why this matches" explanations

Therefore, the core conversational-search functionality is limited without the API key.

2. No Persistent Database

The current implementation uses an in-memory data store.

Therefore:

Catalog data is reseeded on backend restart.
Cart data is lost after restart.
Wishlist data is lost after restart.
Orders are lost after restart.

A production version should use a persistent database such as PostgreSQL or MongoDB.

3. Mock Checkout

The checkout system is for demonstration purposes only.

It does not process real payments and simply returns a simulated order confirmation.

4. Static Catalog

The application currently contains only 40 seeded products.

It does not connect to a real-time inventory or eCommerce product database.

5. No Authentication

The current application does not implement user registration, login, or account management.

Session-based state is used instead.

Future Improvements

The application can be extended with:

Persistent PostgreSQL/MongoDB database
User authentication
Real product inventory
Real payment gateway integration
Product recommendations
Search history
Voice-based shopping
Multi-turn conversational search
Personalized recommendations
Product reviews and ratings
Real-time stock availability
Order tracking
Admin dashboard
Semantic/vector search
Hybrid keyword + vector retrieval
Production-grade observability and error handling
Summary

This project demonstrates how LLM-based natural-language understanding can be combined with deterministic product ranking to create a conversational eCommerce search experience.

The key design principle is:

The LLM understands the user's request and explains the results, while deterministic backend logic decides which products actually rank highest.

This approach provides a balance between the flexibility of conversational AI and the reliability, transparency, and reproducibility of traditional search and ranking systems.
