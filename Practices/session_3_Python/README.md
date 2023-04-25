# Python Overview and Running Applications on Containers

In this practice we will develop and use a Python application running on a docker container

![Docker-Python](documentation_images/docker-python.png)

### Prerequisites
* [Install docker](https://docs.docker.com/engine/install/) 
* [pre-setup](pre-setup%20README.md)

### What You Will Learn
- How to use a docker container of a Python Application
- Docker Compose commands
- Operative System commands and Overview
- How to connect to a Docker Container
- How to connect to a Database by using a DB client
- Programming foundations
- Python Overview

# Practice

You are working on a Zoo, and the Zoo is asking you to create a software that classifies the animals of the Zoo.

This Zoo only passed you a list of the classification of the animals, and you will write a software that classifies them.

Once classification is done, store them into a database.

![img](documentation_images/zoo.png)


### Requirements
* Develop and setup a docker compose to use python on a container
* Develop classes for the following animal classification:
  * **Mammal:** lions, elephants, monkeys, bears, giraffes
  * **Bird:** parrots, eagles, flamingos, penguins, owls
  * **Fish:** sharks, rays, piranhas, clownfish, salmon
* Use Python's data to deposit on a PostgreSQL Database

# Let's do it!


## Step 1

Now let's create an initial python script to run a "hello world" program.

First let's connect to python_app service container:

```
docker-compose exec python_app bash
```

This command will start a bash session on the container of the python_app service.

Now let's create a hello world python application.

To do so, first, let's do following command to edit a python file:

```
vi main.py
```

Within **vi**, let's add following code:

```
print('HELLO WORLD!!')
```

Now let's exit from **vi** and run following command with:

```
python main.py
```

And there you go! You should see HELLO WORLD!! message on the command prompt


## Step 2

Now lets create our first python classes, in this case let's create the following classes:

![img](documentation_images/animal-diagram.png)

To do so, lets create a directory to have all the classes on it.

First create a directory called 'animals' and move to it:

```
$ mkdir animals
$ cd animals
```

Once on animals directory let's create and edit animal.py classes

```
vi animals.py
```

On it let's add the following classes definitions:

```
from typing import Type

class Animal:
    def __init__(self, name: str, most_liked_food: str) -> None:
        self.name = name
        self.most_liked_food = most_liked_food

    def __str__(self) -> str:
        return f'{self.name} likes {self.most_liked_food}'

    def make_sound(self) -> None:
        pass

class Mammal(Animal):
    def __init__(self, name: str, most_liked_food: str, number_of_paws: int) -> None:
        super().__init__(name, most_liked_food)
        self.number_of_paws = number_of_paws

    def walk(self) -> None:
        print(f'{self.name} walks with {self.number_of_paws} paws')

    def make_sound(self) -> None:
        print("Mammal's sound depends the animal")

class Fish(Animal):
    def __init__(self, name: str, most_liked_food: str, number_of_fins: int) -> None:
        super().__init__(name, most_liked_food)
        self.number_of_fins = number_of_fins

    def swim(self) -> None:
        print(f"{self.name} swims and has {self.number_of_fins} fins")
 
    def make_sound(self) -> None:
        print("Glu Glu")

class Bird(Animal):
    def __init__(self, name: str, most_liked_food: str, number_of_wings: int) -> None:
        super().__init__(name, most_liked_food)
        self.number_of_wings = number_of_wings

    def fly(self) -> None:
        print(f"{self.name} flies and has {self.number_of_wings} wings")
 
    def make_sound(self) -> None:
        print("Chirp chirp")
```

now, to be easily imported, let's do a __init__.py file to import the classes, to do, let's edit with following command:

```
vi __init__.py
```

and add the following imports:

```
from .animals import Animal
from .animals import Mammal
from .animals import Fish
from .animals import Bird
```

These imports will facilitate the access of the classes by referring only the parent directory name, in this case "animals"


## Step 3

Now let's use the created classes on main.py

Let's go to our /app directory with:

```
cd /app
```

Now let's edit main.py with following command:

```
vi main.py
```

And on it, let's use following code:

```
from animals import Animal, Mammal, Fish, Bird

print("let's create a Mammal")
mammal = Mammal('monkey','banana',2)
print(mammal)
mammal.walk()
mammal.make_sound()

print("let's create a Fish")
fish = Fish('shark','fish',1)
print(fish)
fish.swim()
fish.make_sound()

print("let's create a bird")
bird = Bird('parrot','seeds',2)
print(bird)
bird.fly()
bird.make_sound()
```

On this code, you are able to instantiate a mammal (monkey), a fish (dolphin), and a bird (parrot).

Now let's exit from editor and run following command to run the application:

```
python main.py
```

## Step 4

Now let's connect to PostgreSQL using DBeaver to create animal table

First let's open [DBeaver](https://dbeaver.io/download/) IDE and click on the New Database Connection Icon that is on the upper left of the IDE:

![img](documentation_images/dbeaver-1.png)

Then a pop up window will open and here selects **´PostgreSQL´** option and click on **Next**

![img](documentation_images/dbeaver-2.png)

Then on connection parameters use the following:
+ Server Host: **localhost**
+ Port: **5433**
+ Database: **animaldb**
+ Username: **myuser**
+ Password: **mypassword**

![img](documentation_images/dbeaver-3.png)

Now click on Test connection and should appear as **´Connected´**

![img](documentation_images/dbeaver-4.png)

Once done, now let's open a sql script by using this connection by clicking **´SQL´** button that is on the upper left part of the menu

![img](documentation_images/dbeaver-5.png)

Now, let's create the animal table by writing and executing the following create table on the SQL script editor:

```
create table animal(
	id serial primary key,
	name varchar(50),
	most_liked_food varchar(50),
	animal_classification varchar(50)
);
```

![img](documentation_images/dbeaver-6.png)

## Step 5

Now we are going to Re-edit the file to insert animal data into database.

To do so, let's re-edit main.py file with following content:

```
import psycopg2
from typing import Type
from animals import Animal, Mammal, Fish, Bird

# Define the connection parameters
conn_params = {
    "host": "postgres_db",
    "port": 5432,
    "database": "animaldb",
    "user": "myuser",
    "password": "mypassword"
}

# Connect to the database
conn = psycopg2.connect(**conn_params)
cur = conn.cursor()

# Define the SQL insert statement
sql = "INSERT INTO animal (name, most_liked_food, animal_classification) VALUES (%s, %s, %s)"

# Define a list of animals to insert
animals = [Mammal('monkey', 'banana', 4),
           Fish('shark', 'fish', 2),
           Bird('parrot', 'seeds', 2)]

# Loop through the animals and insert them into the database
for animal in animals:
    # Get the animal's classification based on its type
    classification = animal.__class__.__name__
    # Execute the SQL insert statement
    cur.execute(sql, (animal.name, animal.most_liked_food, classification))

# Commit the changes and close the connection
conn.commit()
cur.close()
conn.close()

print("Animals were inserted into Database")
```

## Step 6

Check Animal table on DB on dbeaver by running following query:

```
select * from animal;
```

![img](documentation_images/dbeaver-7.png)

## HOMEWORK TIME !!!

**On past main.py script we declared on conn_params variable the postgredb connection parameters on the code. But this is a BAD PRACTICE.**

For Homework, let's do the following:

**UPDATE main.py to grab the connection parameter values from ENV VARIABLES, let's say you are setting on the python_app container the following ENV VARIABLES:**

+ POSTGRE_HOST: **postgres_db**
+ POSTGRE_PORT: **5432**
+ POSTGRE_DB: **animaldb**
+ POSTGRE_USER: **myuser**
+ POSTGRE_PASSWORD: **mypassword**

**Assume you set above ENV VARIABLES, and UPDATE main.py to use them while connecting to DB rather than the hard coded values that are on conn_params**

# Conclusion

By following this tutorial, you should now have a development environment set up using Docker Compose with Python and PostgreSQL containers. Your Python application should be able to connect to the PostgreSQL container and perform operations on the data stored in it. You can continue to develop your application and use the docker-compose commands to manage your containers.
