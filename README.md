# Abby-backend
Backend for Abby cake and bites
import { useState, useRef, useEffect } from "react";

// ─────────────────────────────────────────────
// BUSINESS DATA
// ─────────────────────────────────────────────
const BUSINESS = {
  name: "Abby Cake and Bites",
  tagline: "Handcrafted with Love · Baked to Perfection",
  location: "Njiapanda, 25232 Njiapanda, Himo",
  phone: "0620 767 919",
  email: "lyndagift06@gmail.com",
  hours: {
    Monday: "7:00 AM – 7:00 PM",
    Tuesday: "7:00 AM – 7:00 PM",
    Wednesday: "7:00 AM – 7:00 PM",
    Thursday: "7:00 AM – 7:00 PM",
    Friday: "7:00 AM – 8:00 PM",
    Saturday: "7:00 AM – 8:00 PM",
    Sunday: "Closed",
  },
  status: "Closed · Opens 7 AM Monday",
};

const MENU = [
  {
    category: "Custom Cakes",
    emoji: "🎂",
    items: [
      { name: "Birthday Cake", sizes: ["6\"", "8\"", "10\"", "12\""], priceRange: "TZS 25,000 – 80,000", description: "Personalized, themed & decorated to your wishes" },
      { name: "Wedding Cake", sizes: ["2-tier", "3-tier", "4-tier"], priceRange: "TZS 80,000 – 300,000", description: "Elegant multi-tier masterpieces for your big day" },
      { name: "Anniversary Cake", sizes: ["6\"", "8\"", "10\""], priceRange: "TZS 30,000 – 75,000", description: "Romantic designs with personalized messages" },
      { name: "Baby Shower Cake", sizes: ["6\"", "8\""], priceRange: "TZS 28,000 – 60,000", description: "Adorable pastel creations for new arrivals" },
    ],
  },
  {
    category: "Flavors Available",
    emoji: "✨",
    items: [
      { name: "Vanilla", description: "Classic, light & fluffy", priceRange: "Included" },
      { name: "Chocolate", description: "Rich dark chocolate", priceRange: "Included" },
      { name: "Red Velvet", description: "Velvety smooth with cream cheese", priceRange: "Included" },
      { name: "Lemon", description: "Zesty & refreshing", priceRange: "Included" },
      { name: "Strawberry", description: "Sweet berry delight", priceRange: "+ TZS 3,000" },
      { name: "Caramel", description: "Buttery caramel indulgence", priceRange: "+ TZS 3,000" },
    ],
  },
  {
    category: "Bites & Pastries",
    emoji: "🍪",
    items: [
      { name: "Cupcakes", sizes: ["6 pcs", "12 pcs"], priceRange: "TZS 12,000 – 22,000", description: "Mini decorated cupcakes in any flavor" },
      { name: "Cookies", sizes: ["6 pcs", "12 pcs"], priceRange: "TZS 8,000 – 15,000", description: "Freshly baked butter & chocolate chip cookies" },
      { name: "Brownies", sizes: ["4 pcs", "9 pcs"], priceRange: "TZS 10,000 – 20,000", description: "Fudgy dark chocolate brownies" },
      { name: "Cake Pops", sizes: ["6 pcs", "12 pcs"], priceRange: "TZS 12,000 – 22,000", description: "Fun bite-sized cake balls on a stick" },
      { name: "Doughnuts", sizes: ["6 pcs", "12 pcs"], priceRange: "TZS 9,000 – 17,000", description: "Glazed, sprinkled or filled doughnuts" },
      { name: "Muffins", sizes: ["4 pcs", "8 pcs"], priceRange: "TZS 8,000 – 15,000", description: "Blueberry, banana or choco-chip muffins" },
    ],
  },
];

// ─────────────────────────────────────────────
// SYSTEM PROMPT FOR AI
// ─────────────────────────────────────────────
const SYSTEM_PROMPT = `You are the friendly AI assistant for Abby Cake and Bites, a professional bakery located in Njiapanda, 25232 Njiapanda, Himo. Your name is "Abby" — the bakery's virtual assistant.

BUSINESS INFO:
- Name: Abby Cake and Bites
- Location: Njiapanda, 25232 Njiapanda, Himo
- Phone/WhatsApp: 0620 767 919
- Email: lyndagift06@gmail.com
- Hours: Mon–Fri 7 AM–7 PM, Fri–Sat 7 AM–8 PM, Sunday: Closed
- Current Status: Closed · Opens 7 AM Monday

MENU & PRICES:
Custom Cakes: Birthday (TZS 25,000–80,000), Wedding (TZS 80,000–300,000), Anniversary (TZS 30,000–75,000), Baby Shower (TZS 28,000–60,000)
Flavors: Vanilla, Chocolate, Red Velvet, Lemon (included), Strawberry & Caramel (+TZS 3,000)
Bites: Cupcakes, Cookies, Brownies, Cake Pops, Doughnuts, Muffins (TZS 8,000–22,000)

DELIVERY: Available within Himo town and nearby areas. Delivery fee: TZS 3,000–8,000 depending on distance.
CUSTOM ORDERS: Yes, we do fully custom cakes. Please order at least 3 days in advance.
PAYMENT: Cash on pickup or M-Pesa.

ORDER FLOW: When a customer wants to order, collect information step by step:
1. Ask for their Name
2. Phone number
3. What they want to order (cake type / bites)
4. Size preference
5. Flavor
6. Quantity
7. Pickup or delivery?
8. If delivery: their address
9. Date/time they need it

When you have collected ALL order details (name, phone, item, size, flavor, quantity, pickup/delivery, date), respond with a JSON block exactly like this so the app can process it:

ORDER_COMPLETE:{"name":"...","phone":"...","item":"...","size":"...","flavor":"...","quantity":"...","deliveryType":"...","address":"...","dateNeeded":"...","notes":"..."}

IMPORTANT RULES:
- Only answer questions about Abby Cake and Bites
- Be warm, cheerful and professional
- If asked unrelated questions, politely redirect to bakery services
- Use simple language — many customers may not be tech-savvy
- Keep responses concise and friendly
- Use emojis occasionally to keep it warm 🎂`;

// ─────────────────────────────────────────────
// SIMPLE LOCAL STORAGE DATABASE
// ─────────────────────────────────────────────
const DB = {
  getOrders: () => {
    try { return JSON.parse(localStorage.getItem("abby_orders") || "[]"); } catch { return []; }
  },
  saveOrder: (order) => {
    const orders = DB.getOrders();
    const newOrder = { ...order, id: Date.now(), timestamp: new Date().toISOString(), status: "Pending" };
    orders.unshift(newOrder);
    localStorage.setItem("abby_orders", JSON.stringify(orders));
    return newOrder;
  },
};

// ─────────────────────────────────────────────
// STYLES
// ─────────────────────────────────────────────
const styles = `
  @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;1,400&family=DM+Sans:wght@300;400;500;600&display=swap');

  :root {
    --cream: #fdf6ee;
    --blush: #f5d6c8;
    --rose: #e8a598;
    --dusty-rose: #c97d6e;
    --brown: #6b3f2a;
    --dark: #2c1810;
    --gold: #c9a96e;
    --sage: #a8b89a;
    --white: #ffffff;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    font-family: 'DM Sans', sans-serif;
    background: var(--cream);
    color: var(--dark);
    overflow-x: hidden;
  }

  /* NAV */
  .nav {
    background: var(--white);
    border-bottom: 1px solid var(--blush);
    padding: 0 24px;
    height: 64px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: sticky;
    top: 0;
    z-index: 100;
    box-shadow: 0 2px 20px rgba(107,63,42,0.06);
  }
  .nav-brand {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  .nav-logo {
    width: 38px; height: 38px;
    background: linear-gradient(135deg, var(--rose), var(--dusty-rose));
    border-radius: 50%;
    display: flex; align-items: center; justify-content: center;
    font-size: 18px;
  }
  .nav-title {
    font-family: 'Playfair Display', serif;
    font-size: 18px;
    color: var(--brown);
    font-weight: 700;
  }
  .nav-links {
    display: flex;
    gap: 8px;
  }
  .nav-btn {
    padding: 8px 16px;
    border-radius: 20px;
    border: none;
    cursor: pointer;
    font-family: 'DM Sans', sans-serif;
    font-size: 14px;
    font-weight: 500;
    transition: all 0.2s;
    background: transparent;
    color: var(--brown);
  }
  .nav-btn:hover { background: var(--blush); }
  .nav-btn.active {
    background: var(--dusty-rose);
    color: white;
  }
  .status-dot {
    display: inline-flex; align-items: center; gap: 5px;
    font-size: 12px; color: #888;
    padding: 4px 10px;
    background: #f9f9f9;
    border-radius: 20px;
    border: 1px solid #eee;
  }
  .dot { width: 7px; height: 7px; border-radius: 50%; background: #e0a0a0; animation: pulse 2s infinite; }
  @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.4} }

  /* HERO */
  .hero {
    background: linear-gradient(160deg, #fdf0e6 0%, #f9e2d2 50%, #f5d6c8 100%);
    padding: 80px 24px 60px;
    text-align: center;
    position: relative;
    overflow: hidden;
  }
  .hero::before {
    content: '';
    position: absolute; inset: 0;
    background-image: radial-gradient(circle at 20% 50%, rgba(201,165,110,0.12) 0%, transparent 50%),
                      radial-gradient(circle at 80% 20%, rgba(232,165,152,0.15) 0%, transparent 40%);
  }
  .hero-badge {
    display: inline-block;
    background: rgba(201,125,110,0.12);
    color: var(--dusty-rose);
    padding: 6px 16px;
    border-radius: 20px;
    font-size: 13px;
    font-weight: 500;
    letter-spacing: 0.05em;
    margin-bottom: 20px;
    border: 1px solid rgba(201,125,110,0.2);
  }
  .hero h1 {
    font-family: 'Playfair Display', serif;
    font-size: clamp(36px, 8vw, 68px);
    color: var(--brown);
    line-height: 1.1;
    margin-bottom: 16px;
  }
  .hero h1 em {
    font-style: italic;
    color: var(--dusty-rose);
  }
  .hero p {
    font-size: 17px;
    color: #8a6050;
    max-width: 480px;
    margin: 0 auto 32px;
    line-height: 1.6;
  }
  .hero-ctas {
    display: flex; gap: 12px; justify-content: center; flex-wrap: wrap;
  }
  .btn-primary {
    background: linear-gradient(135deg, var(--dusty-rose), #b56e60);
    color: white;
    padding: 14px 28px;
    border-radius: 30px;
    border: none;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    font-family: 'DM Sans', sans-serif;
    transition: all 0.2s;
    box-shadow: 0 4px 20px rgba(201,125,110,0.35);
  }
  .btn-primary:hover { transform: translateY(-2px); box-shadow: 0 6px 24px rgba(201,125,110,0.45); }
  .btn-secondary {
    background: white;
    color: var(--dusty-rose);
    padding: 14px 28px;
    border-radius: 30px;
    border: 2px solid var(--rose);
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    font-family: 'DM Sans', sans-serif;
    transition: all 0.2s;
  }
  .btn-secondary:hover { background: var(--blush); }

  /* CONTENT */
  .container {
    max-width: 900px;
    margin: 0 auto;
    padding: 40px 24px;
  }

  /* INFO CARDS */
  .info-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
    gap: 16px;
    margin-bottom: 48px;
  }
  .info-card {
    background: white;
    border-radius: 16px;
    padding: 20px;
    border: 1px solid rgba(232,165,152,0.2);
    box-shadow: 0 2px 16px rgba(107,63,42,0.05);
  }
  .info-card-icon { font-size: 24px; margin-bottom: 10px; }
  .info-card-label { font-size: 11px; font-weight: 600; color: var(--rose); text-transform: uppercase; letter-spacing: 0.08em; margin-bottom: 4px; }
  .info-card-value { font-size: 15px; color: var(--dark); font-weight: 500; line-height: 1.5; }

  /* SECTION HEADER */
  .section-header {
    text-align: center;
    margin-bottom: 32px;
  }
  .section-header h2 {
    font-family: 'Playfair Display', serif;
    font-size: clamp(26px, 5vw, 38px);
    color: var(--brown);
    margin-bottom: 8px;
  }
  .section-header p { color: #8a6050; font-size: 15px; }

  /* MENU */
  .menu-category {
    margin-bottom: 40px;
  }
  .menu-category-title {
    font-family: 'Playfair Display', serif;
    font-size: 22px;
    color: var(--brown);
    margin-bottom: 16px;
    display: flex; align-items: center; gap: 10px;
    padding-bottom: 12px;
    border-bottom: 2px solid var(--blush);
  }
  .menu-items {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 14px;
  }
  .menu-item {
    background: white;
    border-radius: 14px;
    padding: 18px;
    border: 1px solid rgba(232,165,152,0.2);
    transition: all 0.2s;
    cursor: pointer;
  }
  .menu-item:hover {
    border-color: var(--rose);
    box-shadow: 0 4px 16px rgba(201,125,110,0.15);
    transform: translateY(-2px);
  }
  .menu-item-name { font-weight: 600; font-size: 15px; color: var(--dark); margin-bottom: 4px; }
  .menu-item-desc { font-size: 12px; color: #9a7060; margin-bottom: 8px; line-height: 1.4; }
  .menu-item-price { font-size: 13px; font-weight: 600; color: var(--dusty-rose); }
  .menu-item-sizes { font-size: 11px; color: #aaa; margin-top: 4px; }

  /* HOURS */
  .hours-grid {
    background: white;
    border-radius: 16px;
    padding: 24px;
    border: 1px solid rgba(232,165,152,0.2);
    box-shadow: 0 2px 16px rgba(107,63,42,0.05);
  }
  .hours-row {
    display: flex; justify-content: space-between; align-items: center;
    padding: 10px 0;
    border-bottom: 1px solid #f5ede8;
    font-size: 15px;
  }
  .hours-row:last-child { border-bottom: none; }
  .hours-day { font-weight: 500; color: var(--dark); }
  .hours-time { color: #8a6050; }
  .hours-closed { color: #d4a0a0; font-style: italic; }

  /* MAP */
  .map-container {
    border-radius: 16px;
    overflow: hidden;
    border: 1px solid rgba(232,165,152,0.2);
    box-shadow: 0 2px 16px rgba(107,63,42,0.05);
    margin-top: 32px;
  }
  .map-placeholder {
    background: linear-gradient(135deg, #f5ede8, #fdf0e6);
    height: 280px;
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    gap: 12px;
    color: var(--brown);
  }
  .map-placeholder-icon { font-size: 40px; }
  .map-placeholder h3 { font-family: 'Playfair Display', serif; font-size: 20px; }
  .map-placeholder p { font-size: 14px; color: #8a6050; }
  .map-open-btn {
    background: var(--dusty-rose);
    color: white;
    padding: 10px 20px;
    border-radius: 20px;
    border: none;
    cursor: pointer;
    font-family: 'DM Sans', sans-serif;
    font-size: 14px;
    font-weight: 500;
  }

  /* CHAT */
  .chat-wrapper {
    max-width: 700px;
    margin: 0 auto;
  }
  .chat-box {
    background: white;
    border-radius: 24px;
    border: 1px solid rgba(232,165,152,0.3);
    box-shadow: 0 4px 30px rgba(107,63,42,0.08);
    overflow: hidden;
    display: flex;
    flex-direction: column;
    height: 580px;
  }
  .chat-header {
    background: linear-gradient(135deg, var(--dusty-rose), #b56e60);
    padding: 18px 20px;
    display: flex; align-items: center; gap: 12px;
    color: white;
  }
  .chat-avatar {
    width: 44px; height: 44px;
    background: rgba(255,255,255,0.2);
    border-radius: 50%;
    display: flex; align-items: center; justify-content: center;
    font-size: 22px;
  }
  .chat-header-info h3 { font-size: 16px; font-weight: 600; }
  .chat-header-info p { font-size: 12px; opacity: 0.85; }
  .online-dot { width: 8px; height: 8px; background: #7fff7f; border-radius: 50%; display: inline-block; margin-right: 5px; }

  .chat-messages {
    flex: 1;
    overflow-y: auto;
    padding: 20px;
    display: flex;
    flex-direction: column;
    gap: 12px;
    background: #fdf9f7;
  }
  .chat-messages::-webkit-scrollbar { width: 4px; }
  .chat-messages::-webkit-scrollbar-thumb { background: var(--blush); border-radius: 2px; }

  .msg {
    max-width: 80%;
    animation: fadeSlide 0.3s ease;
  }
  @keyframes fadeSlide { from { opacity:0; transform: translateY(8px); } to { opacity:1; transform: translateY(0); } }

  .msg.user { align-self: flex-end; }
  .msg.assistant { align-self: flex-start; }

  .msg-bubble {
    padding: 12px 16px;
    border-radius: 18px;
    font-size: 14px;
    line-height: 1.6;
    white-space: pre-wrap;
  }
  .msg.user .msg-bubble {
    background: linear-gradient(135deg, var(--dusty-rose), #b56e60);
    color: white;
    border-bottom-right-radius: 4px;
  }
  .msg.assistant .msg-bubble {
    background: white;
    color: var(--dark);
    border: 1px solid rgba(232,165,152,0.25);
    border-bottom-left-radius: 4px;
    box-shadow: 0 2px 8px rgba(107,63,42,0.06);
  }
  .msg-time { font-size: 11px; color: #bbb; margin-top: 4px; padding: 0 4px; }
  .msg.user .msg-time { text-align: right; }

  .typing-indicator {
    display: flex; gap: 5px; align-items: center; padding: 14px 16px;
  }
  .typing-dot {
    width: 7px; height: 7px;
    background: var(--rose);
    border-radius: 50%;
    animation: typingBounce 1.2s infinite;
  }
  .typing-dot:nth-child(2) { animation-delay: 0.2s; }
  .typing-dot:nth-child(3) { animation-delay: 0.4s; }
  @keyframes typingBounce { 0%,80%,100%{transform:translateY(0)} 40%{transform:translateY(-6px)} }

  .chat-input-area {
    padding: 14px 16px;
    border-top: 1px solid rgba(232,165,152,0.2);
    background: white;
    display: flex; gap: 10px; align-items: flex-end;
  }
  .chat-input {
    flex: 1;
    border: 1.5px solid var(--blush);
    border-radius: 20px;
    padding: 10px 16px;
    font-family: 'DM Sans', sans-serif;
    font-size: 14px;
    color: var(--dark);
    outline: none;
    resize: none;
    max-height: 100px;
    transition: border-color 0.2s;
    background: #fdf9f7;
  }
  .chat-input:focus { border-color: var(--dusty-rose); }
  .chat-send-btn {
    width: 42px; height: 42px;
    background: linear-gradient(135deg, var(--dusty-rose), #b56e60);
    border: none;
    border-radius: 50%;
    color: white;
    font-size: 18px;
    cursor: pointer;
    display: flex; align-items: center; justify-content: center;
    transition: all 0.2s;
    flex-shrink: 0;
  }
  .chat-send-btn:hover { transform: scale(1.08); }
  .chat-send-btn:disabled { opacity: 0.5; cursor: not-allowed; transform: none; }

  .quick-replies {
    display: flex; flex-wrap: wrap; gap: 8px; padding: 0 20px 14px;
    background: #fdf9f7;
  }
  .quick-reply-btn {
    padding: 7px 14px;
    background: white;
    border: 1.5px solid var(--blush);
    border-radius: 16px;
    font-size: 13px;
    color: var(--dusty-rose);
    cursor: pointer;
    font-family: 'DM Sans', sans-serif;
    font-weight: 500;
    transition: all 0.15s;
  }
  .quick-reply-btn:hover { background: var(--blush); border-color: var(--rose); }

  /* ORDER SUCCESS */
  .order-success {
    background: white;
    border-radius: 20px;
    padding: 32px;
    text-align: center;
    border: 2px solid rgba(168,184,154,0.4);
    box-shadow: 0 4px 24px rgba(107,63,42,0.08);
  }
  .order-success-icon { font-size: 52px; margin-bottom: 16px; }
  .order-success h3 { font-family: 'Playfair Display', serif; font-size: 24px; color: var(--brown); margin-bottom: 8px; }
  .order-success p { color: #8a6050; font-size: 15px; line-height: 1.6; }
  .order-details {
    background: #fdf9f7;
    border-radius: 12px;
    padding: 16px;
    margin: 20px 0;
    text-align: left;
    font-size: 14px;
    line-height: 2;
    color: var(--dark);
  }
  .order-detail-row { display: flex; justify-content: space-between; border-bottom: 1px solid #f0e8e4; }
  .order-detail-row:last-child { border-bottom: none; }
  .order-detail-key { color: #9a7060; font-weight: 500; }
  .order-detail-val { font-weight: 600; color: var(--brown); }

  /* ADMIN */
  .admin-login {
    max-width: 360px;
    margin: 60px auto;
    background: white;
    border-radius: 20px;
    padding: 40px 32px;
    box-shadow: 0 4px 30px rgba(107,63,42,0.1);
    text-align: center;
  }
  .admin-login h2 { font-family: 'Playfair Display', serif; font-size: 26px; color: var(--brown); margin-bottom: 6px; }
  .admin-login p { color: #9a7060; font-size: 14px; margin-bottom: 28px; }
  .form-field { margin-bottom: 16px; text-align: left; }
  .form-field label { display: block; font-size: 13px; font-weight: 600; color: var(--brown); margin-bottom: 6px; }
  .form-input {
    width: 100%;
    padding: 11px 14px;
    border: 1.5px solid var(--blush);
    border-radius: 10px;
    font-family: 'DM Sans', sans-serif;
    font-size: 14px;
    color: var(--dark);
    outline: none;
    transition: border-color 0.2s;
  }
  .form-input:focus { border-color: var(--dusty-rose); }

  .admin-panel { max-width: 900px; margin: 0 auto; padding: 32px 24px; }
  .admin-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 28px; flex-wrap: wrap; gap: 12px; }
  .admin-header h2 { font-family: 'Playfair Display', serif; font-size: 28px; color: var(--brown); }
  .admin-stats { display: grid; grid-template-columns: repeat(auto-fit, minmax(160
