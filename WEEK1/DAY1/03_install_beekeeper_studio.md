# Install & Set Up Beekeeper Studio

To start querying data, we need a tool where we can connect to a database, run SQL, and see the results clearly.

That tool is [Beekeeper Studio](https://www.beekeeperstudio.io/) — a modern, beginner-friendly SQL editor that works well with SQLite and DuckDB, either of which we'll use to hold the Capital Bikeshare trip data for this week.

## Step 1: Download Beekeeper Studio

Visit the official site: [https://www.beekeeperstudio.io](https://www.beekeeperstudio.io)

Choose the version for your system:

- Windows: EXE installer
- macOS: DMG installer
- Linux: AppImage or Snap

Install the desktop version, not the Cloud/Pro beta — the desktop version is what we'll use throughout the course.

## Step 2: Launch the App

Once installed, open Beekeeper Studio. You'll see:

- A welcome screen with options to connect to a database
- A left sidebar where tables will appear once connected
- A central editor where you write and run SQL

## Step 3: Connect to Your Database

Here's the one place this week is different from a typical tutorial: there's no ready-made database file sitting in a repository waiting for you to download. Capital Bikeshare publishes its trip data as raw monthly CSV files, seven years of them, and part of this week's work is turning that raw data into an actual queryable database yourself.

The next lesson walks through exactly how to do that — downloading the CSVs and combining them into a single database file using DuckDB. Once that file exists, connecting to it in Beekeeper Studio is the same either way:

1. Click "New Connection"
2. Select SQLite or DuckDB, depending on which format you built
3. Click "Choose a Database File"
4. Browse to and select your database file (for example, `bikeshare_2019_2026.db`)
5. Click Connect

You should then see your database and its tables listed in the left sidebar.

## What's Next

Before building that database file, it's worth pausing on two basic ideas: what a database actually is, and what a table is. The next lesson covers that, and the one after it walks through building your own Capital Bikeshare database from scratch.