# Task - Safari Park

We've now seen how to set up and query a relational database, including how to set up links between our tables and query many tables at once. Now we'll put it into practice by building a database to store data in a specific format.

Usually we'll have a back-end application handling user input and communicating with the database. It will receive that input in a format known as **JSON** (**J**ava**S**cript **O**bject **N**otation) and determine how to structure a query based on that. In this task we will provide you with sample data which you should use as a guide to set up the tables. 

## The Data

Our users will be able to enter details for the animals in the park, their enclosures and the staff working there. Each member of staff will be assinged to a different enclosure on a different day; each enclosure will have more than one person looking after it. The data entered by the user will look like this:

```json
// animal

{
	"id": 1,
	"name": "Tony",
	"type": "Tiger",
	"age": 59,
	"enclosure_id": 1
}

// enclosure

{
	"id": 1,
	"name": "big cat field",
	"capacity": 20,
	"closedForMaintenance": false
}

// staff

{
	"id": 1,
	"name": "Captain Rik",
	"employeeNumber": 12345,
}

// assignment

{
	"id": 1,
	"employeeId": 1,
	"enclosureId": 1,
	"day": "Tuesday"
}
```

## MVP

- Draw an entity relationship diagram to show the structure of the tables and the relationships between them. Each table should have enough columns to capture all the data shown in the JSON above.
- Set up the tables in a postgres database. You can set them up using the `psql` REPL, a GUI like Postico or PGAdmin or by writing an SQL file like the one in the previous task.
- Populate the tables with some of your own data (you don't need to use more cereal mascots, unless you want to). Don't worry about the capacity restriction on enclosures for now, checking the would be handled by the back-end before the data gets sent to the database.
*=====================================================*

Create table for Enclosure:
CREATE TABLE enclosure(
	id SERIAL PRIMARY KEY,
	name VARCHAR(255),
	capacity INT,
	closedForMaintenance BOOLEAN
	);

Create table for Animal:
CREATE TABLE animal(
	id SERIAL PRIMARY KEY,
	name VARCHAR(255),
	type VARCHAR(255),
	age INT,
	enclosure_id INT REFERENCES enclosure(id)
	);

Insert Enclosures:
INSERT INTO enclosure (name, capacity, closedForMaintenance) VALUES ('big cat field', 20, false);
INSERT INTO enclosure (name, capacity, closedForMaintenance) VALUES ('reptile shed', 25, false);
INSERT INTO enclosure (name, capacity, closedForMaintenance) VALUES ('bird house', 35, false);


Insert animals:
INSERT INTO animal (name, type, age, enclosure_id) VALUES ('Simba', 'Lion', 10, 1);
INSERT INTO animal (name, type, age, enclosure_id) VALUES ('Nagini', 'Snake', 71, 2);
INSERT INTO animal (name, type, age, enclosure_id) VALUES ('Tweety', 'Bird', 73, 3);

Create table for Staff:
CREATE TABLE staff(
	id SERIAL PRIMARY KEY,
	name VARCHAR(255),
	employeeNumber INT
	);

Insert staff:
INSERT INTO staff (name, employeeNumber) VALUES ('Monica', 10001);
INSERT INTO staff (name, employeeNumber) VALUES ('Rachel', 10002);
INSERT INTO staff (name, employeeNumber) VALUES ('Ross', 10003);
INSERT INTO staff (name, employeeNumber) VALUES ('Chandler', 10004);
INSERT INTO staff (name, employeeNumber) VALUES ('Phoebe', 10005);
INSERT INTO staff (name, employeeNumber) VALUES ('Joey', 10006);

Create table fro assignemnt:
CREATE TABLE assignment(
	id SERIAL PRIMARY KEY,
	employee_id INT REFERENCES staff(id),
	enclosure_id INT REFERENCES enclosure(id),
	day VARCHAR(255)
	);

Insert assignement:
Monday:
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10001, 1, 'Monday');
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10002, 2, 'Monday');
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10003, 3, 'Monday');
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10004, 1, 'Monday');
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10005, 2, 'Monday');
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10006, 3, 'Monday');

Tuesday:
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10001, 2, 'Tuesday');
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10002, 3, 'Tuesday');
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10003, 1, 'Tuesday');
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10004, 2, 'Tuesday');
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10005, 3, 'Tuesday');
INSERT INTO assignment (employee_id, enclosure_id, day) VALUES (10006, 1, 'Tuesday');

*=====================================================*
 
- Write queries to find:
	- The names of the animals in a given enclosure

All animal in all encosures:

SELECT animal.name, enclosure.name
FROM animal
INNER JOIN enclosure
ON animal.id = enclosure.id

Specific enclosure:

SELECT animal.name
FROM animal
INNER JOIN enclosure
ON animal.id = enclosure.id
WHERE enclosure.id = 1;


	- The names of the staff working in a given enclosure

All Staff in all encosures:

SELECT staff.name, enclosure.name
FROM staff
INNER JOIN assignment
ON staff.id = assignment.employee_id
INNER JOIN enclosure 
ON assignment.enclosure_id = enclosure.id

Specific enclosure:

SELECT staff.name
FROM staff
INNER JOIN assignment
ON staff.id = assignment.employee_id
INNER JOIN enclosure 
ON assignment.enclosure_id = enclosure.id
WHERE enclosure.id = 1;


## Extensions

Write queries to find:

- The names of staff working in enclosures which are closed for maintenance
SELECT staff.name
FROM staff
INNER JOIN assignment
ON staff.id = assignment.employee_id
INNER JOIN enclosure 
ON assignment.enclosure_id = enclosure.id
WHERE enclosure.closedformaintenance = true;

- The name of the enclosure where the oldest animal lives. If there are two animals who are the same age choose the first one alphabetically.
SELECT enclosure.name
FROM enclosure
INNER JOIN animal
ON enclosure.id = animal.enclosure_id
ORDER BY animal.age DESC
LIMIT 1;

- The number of different animal types a given keeper has been assigned to work with.
SELECT COUNT(DISTINCT animal.type) 
FROM animal
INNER JOIN enclosure 
ON animal.enclosure_id = enclosure.id
INNER JOIN assignment
ON enclosure.id = assignment.enclosure_id
WHERE assignment.employee_id = 1;


- The number of different keepers who have been assigned to work in a given enclosure
SELECT COUNT(DISTINCT assignment.employee_id) 
FROM assignment
INNER JOIN enclosure 
ON assignment.enclosure_id = enclosure.id
WHERE assignment.enclosure_id = 1;

From solutions:
SELECT COUNT(DISTINCT staff.name) FROM staff
INNER JOIN assignment
ON staff.id = assignment.employee_id
WHERE assignment.enclosure_id = 1;

- The names of the other animals sharing an enclosure with a given animal (eg. find the names of all the animals sharing the big cat field with Tony)
SELECT DISTINCT animal.name 
FROM animal 
INNER JOIN enclosure
ON animal.enclosure_id = enclosure.id
WHERE enclosure.id = 1;

From solution:
SELECT roommates.name FROM animal
INNER JOIN enclosure
ON animal.enclosure_id = enclosure.id
INNER JOIN animal AS roommates
ON enclosure.id = roommates.enclosure_id
WHERE animal.id = 1;

## Hints

- Don't be tempted to perform too many joins, think about the data stored in each table.
- Remember that the result of a join is just another table. It can be filtered, ordered, summarised and joined again as much as you need.