## Setup 

### Setup with docker
```
docker run --name mongodb -d -p 27017:27017 -v <local_directory_to_store_data_of_the_test> mongo
```

### Generate the database in local

```
./mgodatagen -f config.json
```

You can check out the config.json file to see how the data is generated and values of each field

## Testing the MongoDB indexing

### If the cardinality of values on the index field is a few, indexing doesn't work so well

#### How to test?

Run this query without index and with index on the workingStatus

##### Without indexing
```
db.person_large_data.find({"workingStatus": "No"})
   .explain("executionStats")

```   

The execution time is about 2100 - 3200 ms


##### With indexing
```
db.person_large_data.createIndex( { workingStatus: 1 });

db.person_large_data.find({"workingStatus": "No"})
   .explain("executionStats")

```   

The execution time is still around 2100 - 3200 ms

### The plan selector uses the cardinality of the values of the index field to decide the best one

#### Index these fields first

```
db.person_large_data.createIndex( { workingStatus: 1 });
db.person_large_data.createIndex( { job: 1 });
db.person_large_data.createIndex( { location: 1 });

```

##### Run below queries

```
db.person_large_data.find({"$and": [
    {"uuid": "0415a822-cfd0-462f-b93e-5e9780b7fdeb"},
    {"job": "Broker"}
    
    ]})
   .explain("executionStats")
```

With above query, as the uuid has a lot of values. It's always the index to be selected



```
db.person_large_data.find({"$and": [
    {"location": "Simonefort"},
    {"workingStatus": "Yes"}
    
    ]})
   .explain("executionStats")
```

With above query, as the location has more values than workingStatus. It's always the index to be selected

### In a query with $and condition, the compound index is more optimized than the field index

Create index for field name, income

```
db.person_large_data.createIndex( { name: 1 });
db.person_large_data.createIndex( { income: 1 });
```

Create compound index of name and income

```
db.person_large_data.createIndex( { name: 1, income: 1 });

```

Then run the below query

```
db.person_large_data.find({
    $and: [
         {'name':'Daphne Smith'},
         {'income': {$gte: 5000 }}
    ]
})
.explain("executionStats")
```

You will see the compound index is selected