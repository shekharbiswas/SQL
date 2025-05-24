# 🧰 Install & Set Up Beekeeper Studio

To start querying data, we need a tool where we can **connect to a database**, **run SQL**, and **see our results clearly**.

That tool is **[Beekeeper Studio](https://www.beekeeperstudio.io/)** — a modern, beginner-friendly SQL editor that works beautifully with **SQLite** (our database format for Week 1).

<br>

## ✅ Step 1: Download Beekeeper Studio

Visit the official website:  
👉 [https://www.beekeeperstudio.io](https://www.beekeeperstudio.io)

Choose the version for your system:

- 💻 Windows: EXE installer  
- 🍎 macOS: DMG installer  
- 🐧 Linux: AppImage or Snap

> ⚠️ Make sure to install the **desktop version**, not the Cloud/Pro beta.

<br>

## 🚀 Step 2: Launch the App

Once installed, open **Beekeeper Studio**. You’ll see:

- A welcome screen with options to connect to databases  
- A left sidebar where your tables will appear once connected  
- A central editor where you can write and run SQL

<br>

## 🗃️ Step 3: Connect to Your SQLite Database

If you're starting fresh, you’ll first need a `.db` file created from your CSVs.  
(We’ll guide you through that in the next step.)

To open it:

1. Click **"New Connection"**
2. Select **SQLite**  
3. Click **“Choose a Database File”**
4. Browse and select your `.db` file (e.g., `trendhive_week1.db`)
5. Click **Connect**

You should now see your database listed in the left sidebar.

<br>

## ✅ Optional: Load Prebuilt `.db` File

If you already have a shared `.db` file with `orders` and `products` loaded, you can skip CSV import.

Use the same **SQLite → Choose File → Connect** process.

<br>

## 💡 What’s Next?

You're ready to write your first SQL queries!

