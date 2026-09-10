# 🏪 General Store Management System

A desktop **Retail / General Store Management System** built with **Python** and **Tkinter**, backed by a local **SQLite3** database. The application provides two access levels — **Admin** and **Employee (Staff)** — to handle inventory, employee records, billing, and invoices for a general store ("Mangalmurti Store").

---

## 📖 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
  - [Admin Module](#admin-module)
  - [Employee / Billing Module](#employee--billing-module)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Database Schema](#-database-schema)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Running the Application](#running-the-application)
- [How It Works](#-how-it-works)
- [Screenshots](#-screenshots)
- [Known Limitations](#-known-limitations)
- [Future Improvements](#-future-improvements)
- [Contributing](#-contributing)
- [License](#-license)
- [Author](#-author)

---

## 🧾 Overview

This project simulates a real-world **point-of-sale (POS) and inventory management system** for a general/retail store. On launch, the user chooses between two portals:

- **Employee Portal** — for billing/cashier staff to generate customer bills and search past invoices.
- **Admin Portal** — for the store administrator to manage inventory, employees, and view invoices.

All data (employees, products, and bills) is stored locally in a SQLite database (`Database/store.db`), and the GUI is built entirely with Python's built-in `tkinter` toolkit, styled using pre-rendered background images (`images/`) and custom fonts (`fonts/`).

---

## ✨ Features

### Admin Module
- 🔐 **Secure login** — validated against the `employee` table; only users with the `Admin` role can access the admin dashboard.
- 📦 **Inventory Management**
  - View all products in a sortable/scrollable table (Treeview)
  - Add new products (name, category, sub-category, stock, MRP, cost price, vendor number)
  - Update existing product details
  - Delete one or more selected products
  - Search a product by Product ID
- 👥 **Employee Management**
  - View all employees in a table
  - Add new employees (auto-generated Employee ID)
  - Update employee details
  - Delete employee records
  - Search employees
- 🧾 **Invoices** — view previously generated customer bills
- 🚪 Logout / Exit with confirmation dialogs
- 🕒 Live clock displayed on every admin screen

### Employee / Billing Module
- 🔐 **Staff login** using employee credentials
- 🛒 **Billing System (POS-style cart)**
  - Cascading dropdowns to pick **Category → Sub-Category → Product**
  - Real-time stock display for the selected product
  - Add items to cart with quantity validation (checked against available stock)
  - Remove last-added item from the cart
  - Calculate running **Total** amount
  - Capture customer **name** and **phone number** (validated)
  - **Generate Bill** — creates a uniquely-generated bill number, saves the invoice to the database, and automatically deducts sold quantities from inventory stock
  - **Search Bill** by bill number to retrieve a previously generated invoice
  - Clear cart / reset billing screen
- 🕒 Live clock and logged-in staff name displayed on the billing screen

---

## 🛠 Tech Stack

| Layer            | Technology                     |
|-------------------|--------------------------------|
| Language          | Python 3                       |
| GUI Framework     | Tkinter (`tkinter`, `ttk`, `scrolledtext`) |
| Database          | SQLite3 (`sqlite3` standard library) |
| UI Assets         | PNG background images (`images/`), custom fonts (`fonts/` — Poppins, Podkova) |

No external/third-party Python packages are required — the project relies solely on Python's standard library.

---

## 📁 Project Structure

```
general_store_management_system/
├── Database/          # SQLite database file(s) (e.g., store.db)
├── fonts/             # Custom fonts used in the UI (Poppins, Podkova, etc.)
├── images/            # Background images & icons for each screen
├── main.py            # Entry point — landing screen to choose Employee or Admin
├── admin.py           # Admin portal: login, inventory & employee management, invoices
└── employee.py        # Employee portal: staff login and billing/POS system
```

---

## 🗄 Database Schema

The app expects a SQLite database at `./Database/store.db` with (at minimum) the following tables:

**`employee`**
| Column      | Description                                    |
|-------------|-------------------------------------------------|
| emp_id      | Unique employee ID (e.g., auto-generated `EMPxxxxx`) |
| password    | Login password                                 |
| first_name  | Employee first name                            |
| last_name   | Employee last name                             |
| phone       | Employee phone number                          |
| aadhar      | 12-digit ID number                             |
| role        | `"Admin"` or staff role — determines dashboard access |

**`raw_inventory`**
| Column          | Description                          |
|-----------------|---------------------------------------|
| product_id      | Auto-incrementing product ID          |
| product_name    | Name of the product                   |
| product_cat     | Product category                      |
| product_subcat  | Product sub-category                  |
| stock           | Quantity currently in stock           |
| mrp             | Maximum retail price                  |
| cost_price      | Store's cost price                    |
| vendor_phn      | Vendor's phone number                 |

**`bill`**
| Column         | Description                             |
|----------------|-------------------------------------------|
| bill_no        | Auto-generated unique bill number (`BBxxxxxx`) |
| date           | Date the bill was generated              |
| customer_name  | Name of the customer                     |
| customer_no    | Customer's phone number                  |
| bill_details   | Full text of the itemized bill/cart      |

> ⚠️ Note: The schema above is inferred from the SQL queries used throughout `admin.py` and `employee.py`. If `Database/store.db` is not present in your local copy, you will need to create it (see [Getting Started](#-getting-started)) with matching tables before running the application.

---

## 🚀 Getting Started

### Prerequisites
- Python 3.x installed (Tkinter ships with most standard Python installations)
- No additional pip packages required

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/mayuresh2543/general_store_management_system.git
   cd general_store_management_system
   ```

2. **Verify/Set up the database**

   Ensure a `Database/store.db` SQLite file exists with the `employee`, `raw_inventory`, and `bill` tables described above. If it isn't already included, create it, for example:

   ```python
   import sqlite3

   conn = sqlite3.connect("Database/store.db")
   cur = conn.cursor()

   cur.execute("""
   CREATE TABLE IF NOT EXISTS employee (
       emp_id TEXT PRIMARY KEY,
       password TEXT,
       first_name TEXT,
       last_name TEXT,
       phone TEXT,
       aadhar TEXT,
       role TEXT
   )""")

   cur.execute("""
   CREATE TABLE IF NOT EXISTS raw_inventory (
       product_id INTEGER PRIMARY KEY AUTOINCREMENT,
       product_name TEXT,
       product_cat TEXT,
       product_subcat TEXT,
       stock INTEGER,
       mrp REAL,
       cost_price REAL,
       vendor_phn TEXT
   )""")

   cur.execute("""
   CREATE TABLE IF NOT EXISTS bill (
       bill_no TEXT PRIMARY KEY,
       date TEXT,
       customer_name TEXT,
       customer_no TEXT,
       bill_details TEXT
   )""")

   # Seed at least one Admin user so you can log in
   cur.execute(
       "INSERT OR IGNORE INTO employee VALUES (?,?,?,?,?,?,?)",
       ("EMP001", "admin123", "Store", "Admin", "9876543210", "123456789012", "Admin")
   )

   conn.commit()
   conn.close()
   ```

3. **Confirm assets are present**

   Ensure the `images/` folder contains all referenced PNGs (e.g., `main.png`, `admin_login.png`, `admin.png`, `inventory.png`, `add_product.png`, `update_product.png`, `employee.png`, `employee_login.png`, `bill_window.png`, etc.) since the UI loads these as backgrounds.

### Running the Application

Launch the main landing window:

```bash
python main.py
```

From the landing screen, click:
- **Employee** → opens `employee.py` for the staff/billing login
- **Admin** → opens `admin.py` for the admin dashboard login

> The app internally launches `admin.py` / `employee.py` as separate processes (`os.system("python ...")`), so both scripts must remain in the same directory as `main.py`.

---

## ⚙️ How It Works

1. **`main.py`** displays a splash/landing screen with two buttons — Employee and Admin — each launching the respective script as a subprocess while hiding the main window.
2. **`admin.py`** authenticates against the `employee` table and only proceeds if the user's `role` is `Admin`. Once logged in, the admin can navigate to Inventory, Employees, or Invoices via a dashboard.
3. **`employee.py`** authenticates any valid employee and opens a **billing window** where staff select a product via cascading category/sub-category/product dropdowns, add it to an in-memory cart, and generate an itemized bill. On bill generation, a unique bill number is created, the invoice is saved to the `bill` table, and stock levels in `raw_inventory` are decremented accordingly.
4. All screens include a live clock, confirmation dialogs for exit/logout, and basic input validation (phone numbers, Aadhar numbers, numeric fields).

---

## 🖼 Screenshots

*(Add screenshots of the Main Menu, Admin Dashboard, Inventory screen, and Billing screen here to help new users understand the UI at a glance.)*

```
images/main.png
images/admin.png
images/inventory.png
images/bill_window.png
```

---

## ⚠️ Known Limitations

- Passwords are stored and compared in plain text — **not suitable for production** without adding hashing (e.g., `bcrypt`/`hashlib`).
- Launching sub-screens via `os.system("python ...")` is OS/environment dependent and assumes `python` is on the system `PATH`.
- UI layout uses fixed pixel/relative placement (`place()`), so the window is not resizable and may not scale well on all screen resolutions.
- No automated tests are currently included.

---

## 🔮 Future Improvements

- Hash and salt employee passwords
- Replace `os.system()` subprocess calls with in-process window management (`Toplevel`)
- Add PDF invoice generation/export
- Add reporting/analytics dashboard (sales trends, low-stock alerts)
- Migrate UI to a more modern framework (e.g., PyQt, CustomTkinter) for responsive design
- Add unit tests for cart, billing, and inventory logic

---

## 🤝 Contributing

Contributions are welcome! To contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m "Add your feature"`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## 📄 License

No license file is currently included in this repository. Consider adding one (e.g., MIT License) to clarify how others may use, modify, or distribute this project.

---

## 👤 Author

**Mayuresh** ([@mayuresh2543](https://github.com/mayuresh2543))
