# This repo contains important MongoDB commands

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