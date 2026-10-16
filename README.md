# Yo-Chi-Bowl-Planner-
PostgreSQL database that calculates the calories, sugar and cost of Yo-Chi bowls
🍦 Yo-Chi Bowl Planner
A PostgreSQL database that works out the calories, sugar and cost of a self-serve frozen yogurt bowl, before it hits the scale.
At Yo-Chi you fill your own cup and pay by weight. Nutrition is published per 100g for each flavour and topping separately, so you never see the total for the bowl you actually build. This database stores real Yo-Chi nutrition data, saves the bowls customers make (grams of each item), and tells each customer Yes or No: does this bowl fit my calorie limit and budget?
Built for 31061 Database Principles at UTS (Spring 2026).
What it can answer
What are the total calories, sugar and cost of my bowl?
Does my bowl fit my calorie limit and budget?
What can I have for under 200 calories? (Only 7 combinations, all with fresh fruit.)
Which treat bowls fit 200–300 calories and under $12?
Which same-category topping swap saves 400+ calories?
Database design
Table	What it stores	Key
Customer	Name, calorie limit and budget per bowl	customer_id
Bowl	Each saved bowl and its date	bowl_id (FK: customer_id)
Flavour	Frozen yogurt flavours: calories, sugar, price per 100g	flavour_id
Topping	Toppings: calories, sugar, price per 100g	topping_id (FK: category_id)
Category	Topping groups (Fresh Fruit, Sweet Treats, ...)	category_id
Bowl_Flavour	Grams of each flavour in a bowl (bridge table)	bowl_id + flavour_id
Bowl_Topping	Grams of each topping on a bowl (bridge table)	bowl_id + topping_id
Relationships
One-to-many: a Customer saves many Bowls; a Category groups many Toppings.
Many-to-many: Bowl and Flavour (via Bowl_Flavour); Bowl and Topping (via Bowl_Topping).
Integrity
19 CHECK constraints, e.g. sugar must be 0–100g per 100g, and calories >= sugar * 4 (each gram of sugar is about 4 kcal).
ON DELETE RESTRICT: a flavour or topping used in a saved bowl can't be deleted.
ON DELETE CASCADE: deleting a customer also deletes their bowls.
View: Bowl_Summary adds up each bowl's flavours and toppings and compares the totals with the customer's limits.
Example
Which saved bowls fit both the calorie limit and the budget?
sql
SELECT bowl_id, customer, total_calories, total_cost
FROM Bowl_Summary
WHERE within_calories = 'Yes' AND within_budget = 'Yes'
ORDER BY bowl_id;
 bowl_id |  customer  | total_calories | total_cost
---------+------------+----------------+------------
     103 | Mia Chen   |          291.9 |      10.70
     105 | Liam Patel |          587.5 |      13.56
     106 | Zara Ahmed |          272.8 |       8.58
(3 rows)
What can I have for under 200 calories? (150g of yogurt + 30g of one topping)
sql
SELECT f.name AS flavour, t.name AS topping,
       ROUND((150*f.calories + 30*t.calories)/100.0, 1) AS calories,
       ROUND((150*f.price + 30*t.price)/100.0, 2) AS cost
FROM Flavour f, Topping t
WHERE (150*f.calories + 30*t.calories)/100.0 < 200
ORDER BY calories;
    flavour     |      topping      | calories | cost
----------------+-------------------+----------+------
 Ube            | Strawberries      |    186.9 | 7.65
 Ube            | Fresh Blueberries |    189.0 | 7.74
 Vanilla        | Strawberries      |    194.4 | 7.20
 Ube            | Fresh Mango       |    195.0 | 7.68
 Signature Tart | Strawberries      |    195.9 | 7.20
 Vanilla        | Fresh Blueberries |    196.5 | 7.29
 Signature Tart | Fresh Blueberries |    198.0 | 7.29
(7 rows)
How to run it
Install PostgreSQL (any recent version).
Download yochi_bowl_planner.sql from this repository.
Open a terminal in the folder where you saved it and start PostgreSQL:
   psql
Run the script:
   \i yochi_bowl_planner.sql
It creates all 7 tables and the view, inserts the data, and runs the example queries. The first time, you'll see a few NOTICE: table ... does not exist, skipping messages. These are expected, not errors. 5. Try it yourself:
sql
   SELECT * FROM Bowl_Summary;
The script starts with DROP ... IF EXISTS, so you can run it as many times as you like.
About the data
Calories and sugar come from Yo-Chi's published nutritional information. Energy was converted from kJ to kcal (kJ ÷ 4.184).
Yo-Chi prices the whole bowl by weight, so the per-item prices are illustrative.
Customers and their bowls are made up.
This is an independent student project and is not affiliated with Yo-Chi.
Files
File	Description
yochi_bowl_planner.sql	Full PostgreSQL script: tables, constraints, view, data and queries
erd.png	Entity Relationship Diagram
yochi_bowl_planner_slides.pdf	Presentation slides
Author: Fabiha Anbar, Bachelor of Economics and Business Analytics, UTS
