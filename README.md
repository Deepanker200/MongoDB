# This repo contains important MongoDB commands

- About 
MongoDB is mainly based on a document-oriented (JSON-like) model with B-tree indexes.

- Showing DBs
    - show dbs

- Showing Collections
    - show collections

- Creating a new or using an existing a DB
    - use car_dealership

--------------------------------    
- For creating a collection
    - db.createCollection("cars")
--------------------------------
# Create
- For inserting document in collection
    - db.cars.insertOne({name:"Maruti"})
    - db.cars.insertMany([{"name":"Tata"},{"name":"Toyota"}])

# Read
- For reading data in collection
    - db.cars.find()                     //It will retrieve and read all the data of the collection
    - db.cars.find({"model":"XUV"})      //Gives all details of XUV model
    - db.cars.find({},{model:1})         //Only gives model details along with _id
    - db.cars.find({fuel_type:"Petrol"},{model:1})          //Gives cars model whose fuel_type is Petrol along with _id
    - db.cars.find({fuel_type:"Diesel"},"{name:0})          //Except name it gives all details of the car whose fuel_type is Diesel    

    - db.cars.find({features:"Sunroof"})        //features is an array inside document
    - db.cars.find({"engine.type":"Turbocharged"})      //engine is an object inside document(Objec/Nested Document)

# Update
- db.cars.updateOne({model:"XUV"},{$set:{color:"Pink"}})        //sets new field or add if not existed in the existing document
- db.cars.updateOne({model:"Nexon"},{$push:{features:"Heated Seats"}})      //Only for array to adding new value

- db.cars.updateOne({model:"Nexon"},{$pull:{features:"Heated Seats"}})      //Only for array to remove a value

- UpdateMany
- db.cars.updateMany({fuel_type:"Petrol"},{$set:{alloys:"yes"}})

- db.cars.updateOne({model:"Nexon"},{$push:{features:{$each:["Wireless Charging","Voice Control"]}}})   //For adding new values to the array
- db.cars.updateOne({model:"Nexon"},{$unset:{color:""}})        //For removing or unset the field
- db.cars.updateMany({},{$set:{color:"Blue"}})
- db.cars.updateOne({model:"Venue"},{$set:{color:"Black"}},{upsert:true})   //Creates new document if not existed

# Delete

-db.cars.deleteOne({model:"Venue"})
- db.cars.deleteMany({color:"Black"})

# Data Types
- ObjectId
- String
- Integer
- Double
- Boolean
- Array
- Object/Embedded Document
- Date date:new Date()
- Timestamps ts:new Timestamp()
- Null
- Decimal128

# Operators
- Comparison Operators
    - $eq =
    - $lt <
    - $gt >  db.cars.find({"engine.cc":{$gt:1400}})
    - $lte <=
    - $gte >=
    - $ne !=
    - $in   db.cars.find({"engine.cc":{$in:[1400,1598]}})       //Checks one field against multiple possible values.
    - $nin

- Logical Operators
    - AND
        db.cars.find({
            $and:[
                {condition1},
                {condition2}
            ]
        })

    - OR        //Checks different fields or different conditions across fields.
        db.cars.find({
            $or:[
                {condition1},
                {condition2}
            ]
        })
    - NOR
    - NOT

- Element Operators
    - exists operator
    db.cars.find({fuel_type:{$exists:true}})        //It will fetch data that has fuel_type field in the document

    - $type
    db.cars.find({name:{$type:"string"}})       //If int then no data will be fetched

    // Here we can filter the content based on BASON type like string, bool etc
    // This can be useful to find field with null values
    // {name:{$type:"string"}}

- Array Operators
    - $size
    //Return all documents that match specified array size
    db.cars.find({features:{$size:3}})

    - $all
    // Return all documents that match the pattern 
    db.cars.find({features:{$all:['Bluetooth','ABS']}})


# Cursor Methods
- Count
db.cars.find().count()
db.cars.find({fuel_type:"Petrol"}).count()

- Sort
db.cars.find({},{model:1}).sort({model:1})  //-1 for descending order

- Limit
db.cars.find().limit(2)     //Returns first 2 docs

- Skip
db.cars.find().skip(2)      //Skips first 2 documents

# Aggregate Framework

    - The aggregation pipeline only transforms data in the output.
    - It does not update documents in the database.

    - Filtering
    - Grouping
    - Sorting
    - Reshaping
    - Summarizing Data using a pipeline

- Group
    db.cars.aggregate([
        {$group:{_id:"$maker"}}
    ])

    - Find no. of cars in each maker

        db.cars.aggregate([
        {$group:{
            _id:"$maker",
            TotalCars:{$sum:1}      //1 means number of documents to be printed
            }
        }
        ])

        - Accumulator Operators

            - $sum
            - $avg
                - db.cars.aggregate([ { $group: { _id: "$maker", AvgPrice: { $avg: "$price" } } }] )
            - $min
            - $max
            - $push
            - $addToSet

        Hyundai cars having engine of more than 1200cc
        db.cars.aggregate(
            [{$match:
            {
                maker:"Hyundai",
                "engine.cc":{$gt:1200}
            }
            }]
        )

        //When we are using find we use multiple conditions and not stages

        or 

        db.cars.find({ maker: "Hyundai", "engine.cc": { $gt: 1200 } })

        or
        
        db.cars.find({  $and: [{ maker: "Hyundai" },{ "engine.cc": {$gt:1200 }}]})

    - Print Total no. of Hyundai cars
        db.cars.aggregate([ {$match:{ maker:"Hyundai"}}, {$count:"Total_cars"}])    //stage1 and stage2

    - Count no. of diesel and petrol cars of Hyundai Brand
        db.cars.aggregate([
            {
                $match:{
                    maker:"Hyundai"
                }
            },
            {$group:{
                _id:"$fuel_type",
                TotalCars:{$sum:1}
            }}
        ])

        //Answer

        db.cars.aggregate([
        {
            $match: {
            maker: "Hyundai",
            fuel_type: { $in: ["Petrol", "Diesel"] }
            }
        },
        {
            $group: {
            _id: "$fuel_type",
            TotalCars: { $sum: 1 }
            }
        }
        ])


    - Find all the Hyundai cars and only show Maker, Model and fuel_type details
        db.cars.aggregate([
            {$match:{
                maker:"Hyundai"
            }},
            {$project:{
                _id:0,
                maker:1,
                model:1,
                fuel_type:1
            }}
        ])

    - For the previous output, sort the data based on Model
        db.cars.aggregate([
            {$match:{
                maker:"Hyundai"
            }},
            {$project:{
                _id:0,
                maker:1,
                model:1,
                fuel_type:1
            }},
            {$sort:{model:1}}
        ])

    - Group the cars by Maker and then sort based on count(no. of cars)~ For sorting and counting both

        - db.cars.aggregate(
            {$sortByCount:"$maker"}
        )
    
    - unwind
        - db.cars.unwind([{$unwind:"$owners"}]) //to make seperate documents having 2 or 3 owners
    
# String Operators
    - db.cars.aggregate([
    {$match:{maker:"Hyundai"}},
    {$project:{ 
        _id:0, Car_Name:{$concat:[ "$maker"," ","$model"]
        }}
    }])
    
    - Upper Case + Concat[uses array]

    - db.cars.aggregate([
        {$match:{maker:"Hyundai"}},
        {$project:{
            _id:0,
            Car_Name:{$toUpper:{$concat:[
                "$maker"," ","$model"
            ]}}
        }}
    ])


    - $regexMatch

    Performs a regular expression(regex) pattern matching and returns true or false

    Add a flag is_diesel=true/false for each car    
    - db.cars.aggregate([
        {$project:{
            _id:0,
            model:1,
            is_diesel:{
                $regexMatch:{
                    input:"$fuel_type",
                    regex:"Dies"
                }
            }
        }}
    ])

    - $out

    After aggregating, store the result in an another collection 'hyundai_cars'

    - db.cars.aggregate([
        {$match:{maker:"Hyundai"}},
        {$project:{
            _id:0,
            Car_Name:{$toUpper:{$concat:[
                "$maker"," ","$model"
            ]}}
        }},
        {$out:"hyundai_cars"}
    ])

# Arithmetic Operators
    - $add
    - $subtract
    - $divide
    - $multiply
    - $round
    - $abs
    - $ceil

    - Sum
        - db.cars.aggregate([
            {
                $project:{
                    sum:{$add:[2,3]}
                }
            }
        ])

    Print all the cars model and price with hike of 55000(similarly we can use $subtract too)

    - db.cars.aggregate({
        $project:{
            model:1,_id:0,
            new_price:{$add:["$price",55000]}
        }
    })

    AddFields/Set: To add new field in the document

    - db.cars.aggregate({
        $project:{
            model:1,
            _id:0,
            price:1
        }},{
            $addFields:{
                price_in_lakhs:{
                    <!-- $divide:["$price",100000] -->
                    $round:[{$divide:["$price",100000]},1]
                }
            }
        }
    )

    Calculate Total service cost of each Hyundai Car

    - db.cars.aggregate([
        {$match:{maker:"Hyundai"}},
        {$set:{
            Total_service_cost:{
                $sum:"$service_history.cost"
            }
        }},
        {
            $project:{
                _id:0,
                model:1,
                Total_service_cost:1
            }
        }
    ])

# Conditional Operators

    - $cond
    - $ifNull
    - $switch

    Suppose we want to check if a car's fuel_type is "Petrol" and categorize the cars into "Petrol Car"  & "Non-Petrol"

    - $cond
        - db.cars.aggregate([
        { $project:{
                _id:0,
                maker:1,
                model:1,
                fuel_cat:{
                    $cond:{
                        if:{$eq:["$fuel_type","Petrol"]},
                        then:"Petrol Car",
                        else:"Non_Petrol Car"
                    }
                }
            }}
        ])

    - $switch
        - Suppose we want to categorize the price of the car into three categories: "Budget", "Midrange" and "Premium"
        
        - db.cars.aggregate([
            {$project:{
                _id:0,
                maker:1,
                model:1,
                budget_cat:{
                    $switch:{
                        branches:[
                            {
                                case:{$lt:["$price",500000]},
                                then:"Budget"
                            },
                            {
                                case:{
                                    $and:[{$gte:["$price",500000]},{$lte:["$price",1000000]}]
                                },
                                then:"Midrange"
                            },
                            {
                                case:{$gte:["$price",1000000]},
                                then:"Premium"
                            }
                        ],
                        default:"Unknown"
                    }
                }
            }}
        ]) 

# Date Operators

    - $dateAdd
    - $dateDiff
    - $month
    - $year
    - $hour
    - $dateOfMonth
    - $dayOfYear

    - Example:
        - db.collection.aggregate([
            {
                $project:{
                    newDate:{
                        $dateAdd:{
                            startDate: new Date("2024-08-29"),  //Starting Date
                            unit:"day",     //Unit to be added
                            amount:7        //Amount to add
                        }
                    }
                }
            }
        ])

# Variables in MongoDB

    - System Generated Variables

    - Print current date and time
        -  db.cars.aggregate({$project:{_id:0,model:1,date:"$$NOW"}})

    - User Defined Variables: These variables allow us to store values and reuse them within the same pipeline, making the pipeline more readable and efficient in certain scenarios

    $let is used to create temporary variable 
    -  my_price=1500000 // to directly assign variable to the document
    - Object.keys(this) //For checking variables assigned and used by MongoDB

# DATA MODELING

    - The two main types of relationships are:
    1. Embedded Documents(Denormalization)
    2. Referenced Documents(Normalization)

# Types of Relationship
    - One to One
    - One To Many
    - Many To Many

    # Embedded Documents: In Same Document
    userA
        orderA
        orderB

    //Document size is of 16mb
    // Only 100 levels deep

    # Referenced Documents: 
    users
    - {
        "_id": "user1",
        "name": "Amit Sharma",
        "email": "amit.sharma@example.com",
        "phone": "+91-987654210",
        "address": "MG Road, Mumbai, Maharashtra"
    }
    orders
    -  {
        "_id": "order1",
        "user_id": "user1",
        "product": "Laptop",
        "amount": 50000,
        "order_date": "2024-08-01"
    }

    - db.users.aggregate([
        {
            $lookup:{
                "from":"orders",        //The target collection to join with
                "localField":"_id",     //The field from the 'users' collection
                "foreignField":"user_id",   //The field from the 'orders' collection
                "as":"orders"               //The name of the new array field to add to the 'user'
            }
        }
    ])
    
    -> Note: We can use $out to store this relationship into a new collection

    - $lookup: Performs a left join with another collection

# Schema Validation
    - MongoDB uses a JSON Schema format to define the validation rules.

    - db.createCollection("users1",{
        validator:{
            $jsonSchema:{
                bsonType:"object",
                required:["name","phone"],
                properties:{
                    name:{
                        bsonType:"string",
                        description:"Name should be string"
                    }
                }
            }
        }
    })

    Validation Levels:
    - strict
    - moderate

    Validation Actions:
    - error
    - warn

# Indexes in MongoDB

    An Index is a data structure that improves the speed of query operations by allowing the database to quickly locate anad access the required data without scanning every document in a collection.

    Stores the indexed field in a sorted order, along with pointers to the actual documents in the collection

    - db.users.createIndex({name:1})
    - db.users.getIndexes()
    - db.users.dropIndex("name_1")
    - db.movies.find({title:"The Ace"}).explain("executionStats")

    -> Types of Indexs
    1. Single Field Index
    2. Compound Index
    3. Unique Index
    4. TTL Index(Time to Live)  Automatically delete documents from a collection after a specified period. Such as session logs, caching etc

    -> Performance Consideration when using Indexing in MongoDB
    1. Impact on Write Operations: Increases read but slow down insertions, updations and deletions
    2. Indexing Large Collections: Slow down the performance

# Transactions 
    - A Sequence of operations that are executed as a single unit
    - Maintaining ACID properties across documents and collections

# Sharding
    - Sharding is a method of distributing data across multiple servers (shards) to enable horizontal scaling, allowing the database to handle large datasets and high-throughput operation efficiently

# Replication
    - Replication is a group of MongoDB servers that maintain identical copies of data to ensure high availability, redundancy and data durability with one primary node handling writes and multiple secondary nodes replicating the data

# Questions
Q. Explain how to perform a backup and restore operation in a MongoDB database.
    - Backup: Use mongodump to create a binary backup of the DB.
    - Restore: Use mongorestore to restore the data from a backup.
    Additionally, for cloud deployments, MongoDB Atlas provides automated backups

Q. Explain the role of journaling in MongoDB. How does it help in ensuring durability?
    - Journaling in MongoDB is a mechanism that logs write operations to a journal file before applying them to the database. In case of a crash, MongoDB can use the journal to recover to a consistent state, ensuring durability of write operations.
