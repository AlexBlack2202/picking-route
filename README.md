# Improve Warehouse Productivity using Order Batching with Python 📦

In a **Distribution Center (DC)**, walking time from one location to another during picking route can account for 60% to 70% of the operator’s working time. Reducing this walking time is the most effective way to increase your DC overall productivity.

<p align="center">
  <img align="center" src="static/img/intro_1.gif" width=75%>
</p>

I have published a series of articles that propose an approach to  design a model to simulate the impact of several picking processes and routing methods to find optimal order picking by using the **Single Picker Routing Problem (SPRP)** for a two-dimensional warehouse model (axis-x, axis-y).

SPRP is a specific application of the general **Traveling Salesman Problem (TSP)** answering the question:
>  “Given a list of storage locations and the distances between each pair of locations, what is the shortest possible route that visits each storage location and returns to the depot ?”

This repo is containing a ready-to-use **Streamlit App** designed for **Logistics Engineers** to test these different strategies by only uplooading their own dataset of order lines records.

### Understand the theory behind 📜
- Improve Warehouse Productivity using Order Batching with Python - [Medium Article](https://towardsdatascience.com/optimizing-warehouse-operations-with-python-part-1-83d02d001845)
- Improve Warehouse Productivity using Spatial Clustering with Python Scipy - [Medium Article](https://towardsdatascience.com/optimizing-warehouse-operations-with-python-part-2-clustering-with-scipy-for-waves-creation-9b7c7dd49a84)
- Design Pathfinding Algorithm using Google AI to Improve Warehouse Productivity - [Medium Article](https://towardsdatascience.com/optimizing-warehouse-operations-with-python-part-3-google-ai-for-sprp-308c258cb66f)


# Picking Route Optimization 🚶‍♂️ 

## 💾**Initial: prepare order lines datasets with picking locations**

Based on your **actual warehouse layout**, storage locations are mapped with **2-D (x, y) coordinates** that will be used to measure walking distance.

<p align="center">
  <img align="center" src="static/img/warehouse_layout.png" width=75%>
</p>

Every storage location must be linked to a Reference using Master Data. (For instance, reference #123129 is located in coordinate (xi, yi)). You can then associate every order line to a geographical location for picking.

<p align="center">
  <img align="center" src="static/img/processing_layout.png" width=75%>
</p>

Order lines can be extracted from your WMS Database, this table should be joined with the Master Data table to link every order line to a storage location and its (x, y) coordinate in your warehouse. Extra tables can be added to include more parameters in your model like (Destination, Delivery lead time, Special Packing, ..).

## 🧪**Experiment 1: Inmpact of wave picking in the pickers walking distance?**
_For more information and details about calculation: [Medium Article](https://towardsdatascience.com/optimizing-warehouse-operations-with-python-part-1-83d02d001845)_

### ✔️ Problem Statement

For this study, we will use the example of E-Commerce type DC where items are stored in 4 level shelves. These shelves are organized in multiple rows (Row#: 1 … n) and aisles (Aisle#: A1 … A_n).

<p align="center">
  <img align="center" src="static/img/trolley.jpeg" width=35%>
</p>

1. Items Dimensions: Small and light dimensions items
2. Picking Cart: lightweight picking cart with a capacity of 10 orders
3. Picking Route: Picking Route starts and ends at the same location

<p align="center">
  <img align="center" src="static/img/wave_picking.gif" width=75%>
</p>

Scenario 1, the worst in terms of productivity, can be easily optimized because of
- Locations: Orders #1 and #2 have common picking locations
- Zones: orders have picking locations in a common zone
- Single-line Orders: items_picked/walking_distance efficiency is very low

### 📊 Simulation 

In the article I have built a set of functions needed to run different scenarios and simulate the pickers walking distance.

**Function:** Calculate distance between two picking locations
<p align="center">
  <img align="center" src="static/img/batch_function_1.png" width=75%>
</p>
This function will be used to calculate the walking distance from a point i (xi, yi) and j (xj, yj).

Objective: return the shortest walking distance between the two potential routes from point i to point j.
> Parameters
- y_low : lowest point of your alley (y-axis)
- y_high : highest point of your alley (y-axis)

**Function:** the Next Closest Location
<p align="center">
  <img align="center" src="static/img/batch_function_2.png" width=75%>
</p>
This function will be used to choose the next location among several candidates to continue your picking route.

Objective: return the closest location as the best candidate

**Function:** Create your picking route and calculate the total walking distance
<p align="center">
  <img align="center" src="static/img/batch_function_2.png" width=75%>
</p>

This function will be used to create your picking route from a set of orders to prepare.
- Input: a list of (x, y) locations based on items to be picked for this route
- Output: an ordered sequence of locations covered and total walking distance


**Function:** Create batches of n orders to be picked at the same time
- Input: order lines data frame (df_orderlines), number of orders per wave (orders_number)
- Output: data frame mapped with wave number (Column: WaveID), the total number of waves (waves_number)


**Function:** listing picking locations of wave_ID picking route
- Input: order lines data frame (df_orderlines) and wave number (waveID)
- Output: list of locations i(xi, yi) included in your picking route

### ☑️**Results and Next Steps**

After setting up all necessary functions to measure picking distance, we can now test our picking route strategy with picking order lines.

Here, we first decided to start with a very simple approach
- Orders Waves: orders are grouped by chronological order of receiving time from OMS ( TimeStamp)
- Picking Route: picking route strategy is following the Next Closest Location logic

To estimate the impact of wave picking strategy on your productivity, we will run several simulations with a gradual number of orders per wave:
1. Measure Total Walking Distance: how much walking distance is reduced when the number of orders per route is increased?
2. Record Picking Route per Wave: recording the sequence of locations per route for further analysis

<p align="center">
  <img align="center" src="static/img/batch_final.png" width=100%>
</p>


## 🧮**Experiment 2 - What is the impact of grouping orders by spatial clusters of picking locations?**
_For more information and details about calculation: [Medium Article](https://towardsdatascience.com/optimizing-warehouse-operations-with-python-part-2-clustering-with-scipy-for-waves-creation-9b7c7dd49a84)_


<p align="center">
  <img align="center" src="static/img/cluster_process.png" width=100%>
</p>

### 💡**Idea: Picking Locations Clusters** ###

Group picking locations by clusters to reduce the walking distance for each picking route. _(Example: the maximum walking distance between two locations is <15 m)_

Spatial clustering is the task of grouping together a set of points in a way that objects in the same cluster are more similar to each other than to objects in other clusters.

For this part we will split the orders in two categories:
- Mono-line orders: they can be associated to a unique picking locations 
- Multi-line orders: that are associated with several picking locations

#### **Mono-line orders** 
<p align="center">
  <img align="center" src="static/img/cluster_walking_distance.png" width=100%>
</p>

_Grouping orders in cluster within n meters of walking distance_

#### **Multi-line orders** 
<p align="center">
  <img align="center" src="static/img/cluster_centroids.png" width=75%>
</p>

_Grouping multi-line orders in cluster (using centroids of picking locations) within n meters of walking distance_


### 🐁 **Model Simulation** ###

#### **Methodology** 

To sum up, our model construction, see the chart below, we have several steps before Picking Routes Creation using Wave Processing.

At each step, we have a collection of parameters that can be tuned to improve performance:
<p align="center">
  <img align="center" src="static/img/cluster_analysis.png" width=100%>
</p>


#### **Comparing three methods of wave creation**
<p align="center">
  <img align="center" src="static/img/wave_creation.png" width=75%>
</p>

We’ll start first by assessing the impact of Order Wave processing by clusters of picking locations on total walking distance.

We’ll be testing three different methods:
- Method 1: we do not apply clustering (i.e Initial Scenario)
- Method 2: we apply clustering on single-line orders only
- Method 3: we apply clustering to single-line orders and centroids of multiline orders

#### **Parameters of Simulation**
- Order lines: 20,000 Lines
- Distance Threshold: Maximum distance between two picking locations _(distance_threshold = 35 m)_
- Orders per Wave: orders_number in [1, 9]

#### **Final Results**
<p align="center">
  <img align="center" src="static/img/cluster_final_results.png" width=100%>
</p>
- Best Performance: Method 3 for 9 orders/Wave with 83% reduction of walking distance
- Method 2 vs. Method 1: Clustering for mono-line orders reduce the walking distance by 34%
- Method 3 vs. Method 2: Clustering for mono-line orders reduce the walking distance by 10%

# Build the application locally 🏗️ 

Because the ressources provided by Streamlit cloud or Heroku are limited, I would suggest to run this application locally.

## **Build a python local environment (recommanded)** 

### Then install **virtualenv** using pip3

    sudo pip3 install virtualenv 

### Now create a virtual environment 

    virtualenv venv 
  
### Active your virtual environment    
    
    source venv/bin/activate
  
## Launch Streamlit 🚀

### Install all dependencies needed using requirements.txt

     pip install -r requirements.txt 

### Run the application  

    streamlit run app.py --server.address 0.0.0.0 

### Click on the Network URL in the shell   
  <p align="center">
    <img align="center" src="static/img/launch_streamlit.png" width=50%>
  </p>
  
> -> Enjoy!

# Use the application 🖥️ 
> This app has not been deployed, you need to use it locally/.

## **Why should you use it?**
This Streamlit Web Application has been designed for **Supply Chain Engineers** to help them simulating the impact on picking route optimization in the total distance of their picking operators.

## **Load the data**

- You can use the dataset located in the folder 
 In/df_lines.csv
- You can build your own dataset following the step of ('Initial Step') above

## 🔬 Experiment 1
<p align="center">
  <img align="center" src="static/img/params_1.PNG" width=75%>
</p>

### **Step 1:** Scope

As the computation time can increase exponentially with the size of the dataset _(optimization can be done)_ you can ask the model to take only the n thousands first lines for analysis.

### **Step 2:** Fix the range of orders/wave to simulate

In the picture below we ask the model to run a loop testing scenarios with the number of orders per wave varying between 1 to 10

### **Step 3:** START CALCULATION

Click the button to start the calculations

### **Final Results**
<p align="center">
  <img align="center" src="static/img/batch_experiment_final.png" width=75%>
</p>

💡 This is the same graph with the one presented in the article 

## 🧪 Experiment 2
<p align="center">
  <img align="center" src="static/img/params_2.PNG" width=75%>
</p>

### **Step 1:** Scope

As the computation time can increase exponentially with the size of the dataset _(optimization can be done)_ you can ask the model to take only the n thousands first lines for analysis.

### **Step 2:** START CALCULATION

Click the button to start the calculations

### **Final Results**
<p align="center">
  <img align="center" src="static/img/cluster_final_results.png" width=75%>
</p>

💡 This is the same graph with the one presented in the article 

# About me 🤓
Senior Supply Chain Engineer with an international experience working on Logistics and Transportation operations. \
Have a look at my portfolio: [Data Science for Supply Chain Portfolio](https://samirsaci.com) \
Data Science for Warehousing📦, Transportation 🚚 and Demand Forecasting 📈 
