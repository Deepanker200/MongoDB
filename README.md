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

    - Print all the cars model and price with hike of 55000(similarly we can use $subtract too)

    - db.cars.aggregate({
        $project:{
            model:1,_id:0,
            new_price:{$add:["$price",55000]}
        }
    })

    - AddFields/Set: To add new field in the document

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