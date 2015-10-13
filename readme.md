


<img width="40%" align="right" src="img/sample.png" />

# datamodelr
Data model diagrams with DiagrammeR

## Installation


```r
devtools::install_github("bergant/datamodelr")
```


## Usage

### Model Definition in YAML

Define a data model in YAML:

```yaml
# data model segments

- segment: &md Master data
- segment: &tran Transactions

# Tables and columns

- table: Person
  segment: *md
  columns:
    Person ID: {key: yes}
    Name:
    E-mail:
    Street:
    Street number:
    City:
    ZIP:

- table: Order
  segment: *tran
  columns:
    Order ID: {key: yes}
    Customer: {ref: Person}
    Sales person: {ref: Person}
    Order date:
    Requested ship date:
    Status:

- table: Order Line
  segment: *tran
  columns:
    Order ID: {key: yes, ref: Order}
    Line number: {key: yes}
    Order item: {ref: Item}
    Quantity:
    Price:

- table: Item
  segment: *md
  columns:
    Item ID: {key: yes}
    Item Name:
    Description:
    Category:
    Size:
    Color:
```

Create a data model object with `dm_read_yaml`:


```r
library(datamodelr)
file_path <- system.file("samples/example.yml", package = "datamodelr")

dm <- dm_read_yaml(file_path)
```

Create a [DiagrammeR](https://cran.r-project.org/package=DiagrammeR) graph object
to plot the model:


```r
graph <- dm_create_graph(dm, rankdir = "RL")

dm_render_graph(graph)
```

![](img/sample.png)

### Reverse-engineer an Existing Database

This example uses [Northwind](https://northwinddatabase.codeplex.com/) sample
database and [RODBC](http://CRAN.R-project.org/package=RODBC) 
package as an interface to SQL Server.


```r
library(RODBC)
con <- odbcConnect(dsn = "NW")
sQuery <- dm_re_query("sqlserver")
dm_northwind <- sqlQuery(con, sQuery, stringsAsFactors = FALSE, errors=TRUE)
odbcClose(con)

# convert to a data model
dm_northwind <- as.data_model(dm_northwind)
```

Plot the result:


```r
nw_graph <- dm_create_graph(dm_northwind, rankdir = "BT")

dm_render_graph(nw_graph)
```

![](img/northwind.png)

You can arrange tables in clusters with `dm_set_segment` function: 


```r
table_segments <- list(
  Orders = c("Orders", "Order Details"),
  Customer = c("Customers", "CustomerCustomerDemo", "CustomerDemographics"),
  Employees = c("Employees", "EmployeeTerritories", "Territories", "Region") )

dm_northwind <- dm_set_segment(dm_northwind, table_segments)
```


### PostgreSQL Example

This example uses [DVD Rental](http://www.postgresqltutorial.com/postgresql-sample-database/) 
sample database and [RPostgreSQL](https://cran.r-project.org/package=RPostgreSQL) 
package as an interface to PostgreSQL database. 


```r

library(RPostgreSQL)
#> Loading required package: DBI
con <- dbConnect(dbDriver("PostgreSQL"), dbname="dvdrental", user ="postgres")
sQuery <- dm_re_query("postgres")
dm_dvdrental <- dbGetQuery(con, sQuery) 
dbDisconnect(con)
#> [1] TRUE

dm_dvdrental <- as.data_model(dm_dvdrental)
```

Show model:

```r
dvd_graph <- dm_create_graph(dm_dvdrental, rankdir = "RL")
dm_render_graph(dvd_graph)
```

![](img/dvdrental.png)

To focus on a part of the model use `focus` attribute in `dm_create_graph` function:

```r
focus <- list(tables = c(
    "customer",
    "payment", 
    "rental",
    "inventory",
    "film"
))
    
dvd_graph_small <- dm_create_graph( dm_dvdrental, rankdir = "RL", focus = focus)
dm_render_graph(dvd_graph_small)
```

![](img/dvdrental_small.png)
