# Developing an FTC Data Standard
#### Oliver Bates, Sarah Wise and Adrian Friday

## Introduction
In this document we layout a data standard that captures the detail and complexity of the day-to-day efforts of last mile logistics, grounded but the on-going empirical work of the EPSRC project FTC2050. The data standard looks to provide a standard way of formatting and describing data to enable data analytics and open data relating to last mile freight.

## Scenarios

In this section, we use a series of short scenarios relating to the work undertaken in FTC2050 to help describe why particular data is interesting, and what data is import in the scope of last mile logistics.

In these scenarios, important ***objects*** are bold and italic, with *parameters* italicised.

#### Jobs and rounds in the last mile.
The ***jobs*** (represented by consignments and manifests) are scheduled for delivery. A ***worker*** (e.g. driver) is allocated a number of ***jobs*** to complete in a day. The allocation for B2B and B2C consignments (primarily deliveries) typically happens over night, with a small amount of ad hoc collections being allocated during the daily operation.

  **Events** capture the status of **jobs** through the system as they are processed, loaded and completed (or fail).  ***Jobs*** are some times close in geographical location (e.g. latitude, longitude, address) and can be completed on a ***round***, other times jobs are littered across the city and the driver is left to decide on the most optimal round.

A ***job***, represents work where a contract has been agreed by a particular consigner (customer) to a consignee (recipient), consignments are completed by a courier (e.g. logistics/freight company/free lance). Each job will have a series of ***events***. ***Events*** are represented with a *timestamp* and an event *status*.

A round (or ***trip***) represents a series of jobs that a worker completes during the day. Rounds are allocated to a ***worker*** who perform a number of ***jobs*** on their rounds.
Work/job balancing happens dependant on the number of jobs that day, the ***vehicles*** available and other required ***assets***, such as trolleys or backpacks. Vehicles and assets have limited *capacity* (e.g. fully laden weight, internal dimensions), *range* (e.g. the maximum range the vehicle can travel).

#### Loading vehicles
Understanding the *order* in which items are loaded into the van helps understand the round order. The *weight*, *size* (can the parcel(s) fit on the vehicle?), *grouping* (multiple parcels delivered to the same address) and *priority* (e.g. time prioritised) of a parcel or consignment factor into the order in which parcels are loaded into vehicles.

These parameters potentially effect the ordering of a delivery round.

#### Navigating through the city
As **workers** navigate go about their **trips** through the city, completing their allocated  **jobs** using **vehicles** their location in space and time can help understand optimal routes through the city, how to navigate on foot, the relationship between driver effectiveness and congestion, optimal spotting and parking points, and the temporal and spatial overlap of jobs and trips. Driver and vehicle navigation can be represented as a series of geographical **waypoints**.


### Secondary Data

- Looking up location data of the

#### Navigating inside buildings.
Our work has demonstrated how the distance and navigation inside of a building has a hold over the effectiveness of the delivery driver.

To help better utilise internal navigation, *waypoint* data should also capture *elevation*, and additional information can be made available to help the worker get to the *delivery location* within a multi-story, multi-purpose building more effectively

## Challenges and Aims

- **Database** - Choosing the appropriate database technology - e.g. mysql vs. postgresql vs mongodb, and understanding where these databases should
- **Consistency** and documentation
- **Structuring** - Structured in such a way where the analysis and visualisation of the (large volumes of) data is intuitive, where relationships are clear.
    - Development of data schema and relational model (if necessary)
- **Errors** - The design of the standard/protocol should consider erroneous data, data handling, data cleansing, providing clear descriptions and standards of formatting and guidelines for developers and users
- **Extendability** - The standard must be able to be extended in future versions. *Hence JSON...*
- **Transforming** multiple data sources into this standard
- **Anonymity** - Raw, personally identifiable data, containing un anonymised data (e.g. real names, real clients, real customers, real
    - Developers should maintain lookup and transformation data so that mapping of data is possible
    - Hashing these strings is one option
- **Open Data** - Using the structure described in this document we intend on creating open data based on the datasets provided to us by our project partners.

## Data Structure and Description

Her we describe the structures for:

- [asset](#asset)
- [job](#job)
- [trip](#trip)
- [waypoint](#waypoint)
- [worker](#worker)

[A simple database relationship diagram (without primary, foreign or compound keys)](#relations)

### Anonymity

For where we want anonymity to be a concern we should use UUID (Universally unique identifier) to help mask the original data.

- Real names (e.g. consigner, consignee, company) must be redacted, kept private, and replaced with UUID to ensure privacy and anonymity. Anonymity is also important to help reduce biasing that may effect industry and workers.


### job

A job should contain all relevant information relating to the customer's requirements, start and end locations of the parcel, physical requirements

```javascript
job = {
    "uuid" :'0001', //unique id of the job
    "created": "", //the data/time that the job was created - helps contextualise priority, pick up, etc
    "courier_id":"", //the original courier the contract/job has been made with
    "depot_id":'', //the depot where the parcel starts or ends?
    "parcel_id": '', //barcode number of the item, what if there are multiple parcels?
    "consigner":'Adrian Friday Inc', //the client who has placed the order
    "consignee":'Julian Allen', //the customer
    "address":'31 Fleet Street, London',
    "postcode":'EC4Y 1AA',
    "geo_coded_address" : {
            "lat":'51.513853',
            "lon":'-0.110101'
        },
    "priority": '', //lists the priority of the item, e.g. next day
    "pick_up_by":'', //the time by which the item should be collected for delivery
    "pick_up_address":"",
    "pick_up_time":"",
    "drop_off_by":"",
    "drop_off_time":"",//feels like an event
    "events":[("10:34:33","loaded on van"),
        ("12:43:23", "deposited in secure bin")]
         //either can be tuples (like in e.g.),
        // or UUID of events in event table
}

```
mandatory fields:
- id, parcel_id, consigner, consignee, address, postcode

optional fields: ??

questions:
- what if there are multiple parcels on a job?
- should we be using terms and colloquialism specific to the industry, or should we use more plan and accessible English?

### worker

```javascript
worker = {
    "uuid" :'0001',
    "name":'Tom Cherrett',
    "roles": {'van_driver','loader'}, //the worker
    //may have multiple potential roles that
    //they can work in within last mile logistics
    vehicle_access: {'PERSONAL:001','FLEET:001'}, //
    vehicle_license:{'LGV'},//is preference a thing?
    constraints: {'Bad on Mondays', 'LGV license', 'prefers LGV', 'hates bicycles'},
    tenure: '4 Years',
    employment:{'FTC Logistics','self employed'}
}
```

We should consider look up tables to describe a discrete list of possible:
- *constraints*
- *roles*
- *vehicle_license*
- *vehicle_access* - list of known vehicles that are accessible to workers

### waypoint

- has to work for a worker and a vehicle, these must be distinguishable

```javascript
waypoint = {
    "uuid" :'0001',
    "worker_id":"", //should this be merged with
    //vehicle_id??, and then the UUID for worker
    //and vehicle containing a prefix
    "vehicle_id":"",
    "trip_id": '0001'
    "latitude": '0.43',
    "longitude": '52.12',
    "elevation": '324.3' // or altitude
    "current_job" = "" //uuid of job that is current
    "job_count" = "" //sequence number of current job
}
```
considerations:
- not every GPS device will capture altitude, so might need to be classed as optional
- standard units for elevation/altitude - looks like a [complex discussion](https://gis.stackexchange.com/questions/75572/how-is-elevation-and-altitude-measured)

### trip

Trips are the day long work loads made up of numerous **jobs** throughout a day. By linking corresponding location data (e.g. **waypoints**) to a specific vehicle or worker the **trip** captures the full journey of the **vehicle** (and *worker* if possible) throughout the day, helping track performance of drivers, effectiveness of parcel deliveries and the location of vehicular and other assets.

We are using the term trip as it can be used more ubiquitously than terms such as "round" or "journey" that suggest more rigid structures and are more tightly defined in last mile logistics.

```javascript
trip = {
    uuid :'0001',
    vehicle:'',
    assets:{'0001'}, // a list of the assests used on this trip
    worker:'',
    start:'',
    end:'',
    jobs:{'j001','j002'}
}
```

### asset

An asset describes a tool, or non road vehicle that may be available for last mile workers who wish to move more parcels from their vehicle on the pavement/kerb side (e.g. trolley, back pack, box on wheels).

```javascript
asset = {
    "uuid" :"0001",
    "barcode":"0119681320351", //for scanning
    // when taken out on a round
    "kind":"",
    "capacity":"",
    "home_depot":"",
    "current_location":"" //where the asset is
    // currently, which depot, or whether out
    // on a round
}
```

### depot

An asset describes a tool, or non road vehicle that may be available for last mile workers who wish to move more parcels from their vehicle on the pavement/kerb side (e.g. trolley, back pack, box on wheels).

```javascript
depot = {
    "uuid" :"0001",
    "address":"1 Depot Street", //for scanning when taken out on a round
    "owner":"FTC Logistics",
    "capacity":{"vans":12, "cargo_bike":50,
        "parcel_storage":"10000m2"}
}
```

### event

An event describes an action taken against a job, for example, loading onto vehicle, attempted delivery, proof of delivery, appeared on wrong van, ... . Events are an important component as they attempt to capture some of the detail of the actions taken by workers, as well as why things happen in last mile logistics. This object is useful in relational database, where as in NoSQL databases you may want to have the events as a tuple in the job.

```javascript
event = {
        "id":"0001",
        "timestamp":"2017-03-02 12:03:12",
        "event_name":"Proof of delivery"
         /*relational db may want to use table
         with keys for each event and another
          table with unique event names
          using e_id in event object, relating
           to an event description table*/
}

```
An event may include one of the following:
- arrived at depot
- ready for collection
- proof of delivery received
- delivered to safe location
- delivery window changed by client (??)

### Relations

- tracing the children up to join in the trip

trip (*1*) <-> (*many*) job

trip (*1*) <-> (*1*) worker

trip (*1*) <-> (*many*) waypoint

trip (*1*) <-> (*many*) asset


## Schema

Whilst I've used JSON to help draw the standard for each object in our data standard, the


JSON Schema:
- [job](/schema/json/ftc_job_schema.json)


see:
- [JSON Schema folder](/schema/json)
- [postgres schema](/schema/postgres)


# TODO
- key structure for database relationships
- identifying key fields and parameters that need to be anonymised and removed to enable **Open Data**
- identify mandatory and optional fields (e.g. NULL fields)
