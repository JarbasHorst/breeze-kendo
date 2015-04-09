# Note
This is a fork of [Teleriks Breeze Kendo DataSource](https://github.com/kendo-labs/breeze-kendo) containing additional features and bug fixes.

# Setup

Install mongodb, i.e. `apt-get install mongodb`, or however it's done
on your OS.  We don't need the system-wide service, so we can stop it:

    sudo service mongodb stop
    sudo update-rc.d -f mongodb remove

Then:

    git clone https://github.com/iozag/breeze-kendo.git
    cd breeze-kendo
    npm install
    mkdir db
    mongod --dbpath db

Open a new console.  To init the database:

    git clone https://github.com/mishoo/northwind-mongo.git
    cd northwind-mongo
    sh mongo-import.sh

To run the test page, `node bin/server.js` in the breeze-kendo2 dir,
and open http://localhost:3000/

"Save changes" is broken in this demo, seems due to a bug in
breeze-mongodb (watch the server console on "save").  The "save"
server-side handler appears to expect the request to provide a
metadata argument, but Breeze.js doesn't send it.  Any case, that's
not a bug of our wrappers, I suspect with a better server side (Breeze
seems to best support ASP.NET) it should work flawless.

## Features

- creates a Kendo DataSource object that is kept in sync with the
  Breeze entities.  For instance when an entity changes, the Kendo DS
  is updated automatically; also when the data changes on the Kendo
  side, the Breeze entities are updated automatically.

  This means that for an app that is properly configured to use Breeze
  (i.e. has metadata and breezeManager.saveChanges() works) adding in
  Kendo widgets that support a Kendo DataSource should be a snap.

- `kendo.data.breeze.SchedulerSource` object for using with the [Kendo UI Scheduler](http://docs.telerik.com/kendo-ui/web/scheduler/overview).

- auto-generates a Kendo-compatible data model (`schema.model`) based
  on metadata defined in the Breeze EntityManager.

- supports server-side pagination, sort, filters.

## Usage

### Loading

We assume your server is alredy configured for Breeze.js.

The code is defined in `docroot/breeze-kendo.js`.  Load it after Kendo
UI and Breeze:

```html
<script src=".../jquery.min.js"></script>
<script src=".../kendo.all.min.js"></script>
<script src=".../breeze.min.js"></script>
<script src="breeze-kendo.js"></script>
```

### Using with Kendo Widgets

It defines `kendo.data.breeze.Source`, an object which inherits from `kendo.data.DataSource` and can be used seamlessly with any widgets
that support the [DataSource API](http://docs.telerik.com/kendo-ui/api/framework/datasource).  
The Breeze-specific options are `manager` and `query`.  
Example:

```js
var manager = new breeze.EntityManager(...);
var query = breeze.EntityQuery.from("Products");
var dataSource = new kendo.data.breeze.Source({
  manager         : manager,
  query           : query,
  serverSorting   : true,
  serverPaging    : true,
  serverFiltering : true,
  pageSize        : 10
});
```

The query you specify should return a list of rows with your data.
You can craft it any way you want, for example if you always want to
discard some rows you can say:

```js
var query = breeze.EntityQuery.from("Products")
                              .where("UnitPrice", "<", 10);
```

Now you can pass the `dataSource` to, say, a Grid widget:

```js
$("#grid").kendoGrid({
  dataSource : dataSource,
  filterable : true,
  sortable   : true,
  pageable   : true,
  editable   : true,
  toolbar    : ["create", "save", "cancel"]
});
```

Now pagination, sorting, filtering and even saving is entirely handled by Breeze through our bindings.

### Using with Kendo Scheduler

Use `kendo.data.breeze.SchedulerSource`, an object which inherits from
[`kendo.data.SchedulerDataSource`](http://docs.telerik.com/kendo-ui/api/javascript/data/schedulerdatasource) 
and can be used seamlessly with any the [Scheduler Widget](http://docs.telerik.com/kendo-ui/web/scheduler/overview).  
The Breeze-specific options are `manager`, `query` and `scheduler`.  
Example:

```js
var manager = new breeze.EntityManager(...);
var query = breeze.EntityQuery.from("Products");
var dataSource = new kendo.data.breeze.SchedulerSource({
  manager         : manager,
  query           : query,
  scheduler       : $('#scheduler'),
  serverSorting   : true,
  serverPaging    : true,
  serverFiltering : true,
  pageSize        : 10
});
```

The scheduler jQuery element is only required if you want the data source to filter data to the current view of the scheduler.

### Options

Additionally to the default [DataSource configuration](http://docs.telerik.com/kendo-ui/api/javascript/data/datasource#configuration) 
the following options are available

#### query
The [breeze.EntityQuery](http://www.breezejs.com/sites/all/apidocs/classes/EntityQuery.html) which should be used for accessing the data.

#### manager
The [breeze.EntityManager](http://www.breezejs.com/sites/all/apidocs/classes/EntityManager.html) which should be used for accessing the data.

#### scheduler
A jQuery element. 

The scheduler jQuery element is only required if you want the data source to filter data to the current view of the scheduler.

#### useBreezeMapping
A boolean flag indicating if the Breeze entity should be available from the data item.

If set to `true`, the mapping between Breeze entities and Kendo data items is handled internally by 
the datasource, and the Breeze entities are not available from the data item.

If set to `false`, the Breeze entity will be added to the `__breezeEntity` property of the item in 
the data source.

The default value is `false` for `kendo.data.breeze.Source` and `true` for `kendo.data.breeze.SchedulerSource`
(because the Kendo scheduler cannot handle circular references.)