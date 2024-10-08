# Transaction_Dashboard
This project is designed to implement a backend API with Node.js, Express, MongoDB (MERN stack), and a frontend using React, to interact with the provided APIs for managing product transactions and displaying various visualizations.

Here is a step-by-step README file for the MERN Stack coding challenge based on the task described:

---

# MERN Stack Coding Challenge

## Overview
This project is designed to implement a backend API with Node.js, Express, MongoDB (MERN stack), and a frontend using React, to interact with the provided APIs for managing product transactions and displaying various visualizations.

## Steps to Complete the Task:

### 1. **Setup Project**
   - **Backend Setup:**
     1. Initialize a Node.js project using `npm init`.
     2. Install required packages: 
        ```bash
        npm install express mongoose axios cors
        ```
     3. Create necessary folders for the backend structure:
        - `/models` - For Mongoose schemas.
        - `/controllers` - For handling API logic.
        - `/routes` - For defining routes.
     4. Set up MongoDB:
        - Use MongoDB locally or cloud (e.g., MongoDB Atlas).
        - Create a `.env` file for environment variables to store database connection strings and other secrets.
        - Example connection in `server.js`:
          ```javascript
          mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true });
          ```

   - **Frontend Setup:**
     1. Initialize a React project using `npx create-react-app`.
     2. Install necessary packages for the frontend:
        ```bash
        npm install axios chart.js react-chartjs-2
        ```

### 2. **Backend API Development**

#### a. **Initialize Database with Seed Data**
   - Create an API that fetches data from the third-party API using `axios` and seeds the MongoDB database.
     - **API Endpoint:** `GET /api/init`
     - **Task:** Fetch JSON from the URL and insert the data into MongoDB.
     ```javascript
     const axios = require('axios');
     const Product = require('./models/Product'); // Product Schema
     
     const initDB = async (req, res) => {
       const response = await axios.get('https://s3.amazonaws.com/roxiler.com/product_transaction.json');
       await Product.insertMany(response.data);
       res.json({ message: 'Database initialized successfully' });
     };
     ```

#### b. **List Transactions API**
   - Implement an API to list transactions based on search and pagination.
     - **API Endpoint:** `GET /api/transactions`
     - **Search Parameters:** product title/description/price.
     - **Pagination:** Default page = 1, per page = 10.
     ```javascript
     const listTransactions = async (req, res) => {
       const { page = 1, perPage = 10, search = '' } = req.query;
       const query = {
         $or: [
           { title: { $regex: search, $options: 'i' } },
           { description: { $regex: search, $options: 'i' } },
           { price: { $regex: search, $options: 'i' } }
         ]
       };
       const transactions = await Product.find(query)
         .skip((page - 1) * perPage)
         .limit(parseInt(perPage));
       res.json(transactions);
     };
     ```

#### c. **Statistics API**
   - Implement an API to calculate the total sale amount, sold items, and unsold items for a selected month.
     - **API Endpoint:** `GET /api/statistics`
     - **Input:** Month (January to December).
     ```javascript
     const getStatistics = async (req, res) => {
       const { month } = req.query;
       const transactions = await Product.find({ dateOfSale: { $regex: month, $options: 'i' } });
       const totalSale = transactions.reduce((sum, item) => sum + item.price, 0);
       const soldItems = transactions.filter(item => item.sold).length;
       const unsoldItems = transactions.filter(item => !item.sold).length;
       res.json({ totalSale, soldItems, unsoldItems });
     };
     ```

#### d. **Bar Chart API**
   - Create an API to return price ranges and the number of items in each range.
     - **API Endpoint:** `GET /api/bar-chart`
     - **Response:** Items in each price range.
     ```javascript
     const getPriceRangeData = async (req, res) => {
       const { month } = req.query;
       const transactions = await Product.find({ dateOfSale: { $regex: month, $options: 'i' } });
       const priceRanges = { '0-100': 0, '101-200': 0, ... };
       transactions.forEach(item => {
         if (item.price <= 100) priceRanges['0-100']++;
         // Continue for other ranges
       });
       res.json(priceRanges);
     };
     ```

#### e. **Pie Chart API**
   - Create an API to get categories and the number of items sold in each.
     - **API Endpoint:** `GET /api/pie-chart`
     ```javascript
     const getCategoryData = async (req, res) => {
       const { month } = req.query;
       const transactions = await Product.find({ dateOfSale: { $regex: month, $options: 'i' } });
       const categories = {};
       transactions.forEach(item => {
         categories[item.category] = (categories[item.category] || 0) + 1;
       });
       res.json(categories);
     };
     ```

### 3. **Frontend Development**

#### a. **Table Component**
   - Create a table to list transactions with search and pagination features.
   - Use Axios to call the `GET /api/transactions` API.
   - Implement search and pagination functionality using state hooks.

#### b. **Statistics Component**
   - Display total sale amount, total sold items, and unsold items for the selected month by fetching from `GET /api/statistics`.

#### c. **Bar Chart Component**
   - Implement a bar chart using `react-chartjs-2` by fetching data from `GET /api/bar-chart`.

#### d. **Pie Chart Component**
   - Implement a pie chart using `react-chartjs-2` to display categories and their respective item counts, using `GET /api/pie-chart`.

### 4. **Run the Application**
   - Run the backend using `node server.js` or `nodemon`.
   - Run the frontend using `npm start`.

### 5. **Testing**
   - Ensure that all APIs function correctly and that the frontend visualizations dynamically update based on the selected month.

### 6. **Deployment**
   - Use services like Heroku for backend and Netlify/Vercel for frontend to deploy the application.

---

This README should guide you through the task step by step. Let me know if you need further clarifications!
