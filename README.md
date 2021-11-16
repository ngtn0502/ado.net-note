# ADO.NET note

NhanNguyen

some little note by myself to remind what did i learn

---

## Things worth to review:

| No. | Content                                   |
| --- | ----------------------------------------- |
|     |                                           |
| 1   | [Basic](#basic) |
| 2   | [Connection string](#connection-string) |
| 3   | [SqlCommand Class](#sqlcommand-class) |

# Basic

## What ADO.NET is use for?

ADO.NET is a set of classes (framework) to connect our .NET application to our database, execute commands, and retrieve data

Data provider is a set of classes provide for us to connect our .NET application to our database

> Data provider of SQL Server is - System.Data.SqlClient

=> All the classes help us to connect to our sql server is contain on system.data.sqlclient namespace

- SqlConnection
- SqlCommand
- SqlDataReader
- SqlDataAdapter
- Dataset

# Connection String

Connection string cotain information about:

- Data source: what server you want to connect to

- Initial catalog / database: what databse you want to use

- User, Passwork: For Sql server authentication

- Integrated security: For window authentication

For create new connection string:

> SqlConnection con = new SqlConnection("data source=.; database=Sample; integrated security=SSPI");

For run a command in connection string

> SqlCommand cmd = new SqlCommand("Select * from tblProduct", con);\

It is important to close a connection string because database is valuable resource, if we dont close properly it will affect our performance

Sample ADO.NET code that 

    1. Creates a connection
    2. The created connection object is then passed to the command object, so that the command object knows on which sql server connection to execute this command.
    3. Execute the command, and set the command results, as the data source for the gridview control.
    4. Call the DataBind() method
    5. Close the connection in the finally block. Connections are limited and are very valuable. Connections must be closed properly, for better performance and scalability.

    protected void Page_Load(object sender, EventArgs e)
    {
        using (SqlConnection connection = new SqlConnection("data source=.; database=Sample_Test_DB; integrated security=SSPI"))
        {
            SqlCommand cmd = new SqlCommand("Select * from tblProductInventory", connection);
            connection.Open();
            GridView1.DataSource = cmd.ExecuteReader();
            GridView1.DataBind();
        }
    }

Common Interview Question: What are the 2 uses of an using statement in C#?

    1. To import a namespace. Example: using System;
    2. To close connections properly as shown in the example above

## Connection string in configuration file

There are 2 issues with hard coding the connection strings in application code

    1. For some reason, if we want to point our application to a different database server, we will have to change the application code. If you change application code, the application requires a re-build and a re-deployment which is a time waster.
    2. All the pages that has the connection string hard coded needs to change. This adds to the maintenance overhead and is also error prone.

In real time, we may point our applications from time to time, from Development database to testing database to UAT database.

    Because of these issues, the best practice is to store the connection in the configuration file, from which all the pages can read and use it. This way we have only one place to change, and we don't have to re-build and re-deploy our application. This saves a lot of time.

All configuaration and this related thing is stored in system.configuaration namespace 

=> We use ConfigurationManager in that namespace to retrive connection string

    protected void Page_Load(object sender, EventArgs e)
    {
        string ConnectionString = ConfigurationManager.co .ConnectionStrings["DatabaseConnectionString"].ConnectionString;
        using (SqlConnection connection = new SqlConnection( ConnectionString ))
        {
            SqlCommand cmd = new SqlCommand("Select * from tblProductInventory", connection);
            connection.Open();
            GridView1.DataSource = cmd.ExecuteReader();
            GridView1.DataBind();
        }
    }


# Sqlcommand class

SqlCommand class is used to prepare an SQL statement or StoredProcedure that we want to execute on a SQL Server database.

The following are the most commonly used methods of the SqlCommand class.

    ExecuteReader - Use when the T-SQL statement returns more than a single value. For example, if the query returns rows of data.
    ExecuteNonQuery - Use when you want to perform an Insert, Update or Delete operation
    ExecuteScalar - Use when the query returns a single(scalar) value. For example, queries that return the total number of rows in a table.


The following example performs an Insert, Update and Delete operations on a SQL server database using the ExecuteNonQuery() method of the SqlCommand object.

    protected void Page_Load(object sender, EventArgs e)
    {
        string ConnectionString = ConfigurationManager.ConnectionStrings["DatabaseConnectionString"].ConnectionString;
        using (SqlConnection connection = new SqlConnection("data source=.; database=Sample_Test_DB; integrated security=SSPI"))
        {
            //Create an instance of SqlCommand class, specifying the T-SQL command
            //that we want to execute, and the connection object.
            SqlCommand cmd = new SqlCommand("insert into tblProductInventory values (103, 'Apple Laptops', 100)", connection);
            connection.Open();
            //Since we are performing an insert operation, use ExecuteNonQuery()
            //method of the command object. ExecuteNonQuery() method returns an
            //integer, which specifies the number of rows inserted
            int rowsAffected = cmd.ExecuteNonQuery();
            Response.Write("Inserted Rows = " + rowsAffected.ToString() + "<br/>");

            //Set to CommandText to the update query. We are reusing the command object,
            //instead of creating a new command object
            cmd.CommandText = "update tblProductInventory set QuantityAvailable = 101 where Id = 101";
            //use ExecuteNonQuery() method to execute the update statement on the database
            rowsAffected = cmd.ExecuteNonQuery();
            Response.Write("Updated Rows = " + rowsAffected.ToString() + "<br/>");

            //Set to CommandText to the delete query. We are reusing the command object,
            //instead of creating a new command object
            cmd.CommandText = "Delete from tblProductInventory where Id = 102";
            //use ExecuteNonQuery() method to delete the row from the database
            rowsAffected = cmd.ExecuteNonQuery();
            Response.Write("Deleted Rows = " + rowsAffected.ToString() + "<br/>");
        }
    }
