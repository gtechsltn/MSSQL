# MS SQL Server

```
+ System.Data.SqlClient version: xxx
+ .NET target: xxx
+ SQL Server version: N/A
```

```
+ Microsoft.Data.SqlClient version: xxx
+ .NET target: xxx
+ SQL Server version: N/A
```

## .NET Framework
+ .NET Data Provider Name: System.Data.SqlClient
+ .NET Framework 1.0
+ ...
+ .NET Framework 4.8

## .NET Core and .NET Standard
+ .NET Data Provider Name: Microsoft.Data.SqlClient
+ .NET Core 1.0
+ .NET Core 2.1
+ .NET Core 3.1 đã hết hạn hỗ trợ vào ngày 13 tháng 12 năm 2022.

# .NET
+ Microsoft.Data.SqlClient
+ .NET 5
+ .NET 6
+ .NET 7
+ .NET 8
+ .NET 9
+ .NET 10
+ ...

# Interfaces
+ IDisposable
+ IDbConnection
+ IDbTransaction
+ IDbCommand
+ IDbDataParameter
+ IDataParameterCollection
+ IDataReader
+ IEnumerator<T>

# ADO.NET
    + System.Data
        + DataSet
        + DataTable
            + DataColumn
            + DataRow
        + DataAdapter ~ SqlDataAdapter
    + System.Data.SqlClient
    + SqlConnection
        + Integrated Security=True;Encrypt=True;TrustServerCertificate=True;MultipleActiveResultSets=True;
        + MultipleActiveResultSets Or "Multiple Active Result Sets"
        + SqlConnectionBuilder
        + SqlConnectionstringBuilder
        + DatabaseConnectionFactory
    + SqlTransaction
    + SqlCommand
        + SqlCommand.CommandTimeout
        + SqlCommand.CommandText
        + SqlCommand.CommandType
            + Text
            + StoredProcedure
            + TableDirect
        + SqlCommand.ExecuteNonQuery();
        + SqlCommand.ExecuteScalar();
        + SqlCommand.ExecuteXmlReader();
        + SqlCommand.ExecuteReader(CommandBehavior.SequentialAccess);
        + SqlCommand.ExecuteReader(CommandBehavior.CloseConnection);
        + SqlCommand.ExecuteReader(CommandBehavior.SingleResult);
        + SqlCommand.ExecuteReader(CommandBehavior.SingleRow);        
    + SqlParameter
        + ParameterDirection.Input
        + ParameterDirection.Output
        + ParameterDirection.InputOutput
        + ParameterDirection.ReturnValue
    + SqlDbType        
        + SqlDbType.BigInt
        + SqlDbType.Binary
        + SqlDbType.Bit
        + SqlDbType.DateTime
        + SqlDbType.Decimal (số dấu phẩy động)
        + SqlDbType.Float (số dấu phẩy động)
        + SqlDbType.Image (ảnh)
        + SqlDbType.Int (số)
        + SqlDbType.Json (JSON)
        + SqlDbType.Money (tiền tệ)
        + SqlDbType.NVarChar (chữ)
        + SqlDbType.Udt: A SQL Server user-defined type (UDT)
        + SqlDbType.Real (số dấu phẩy động)
        + SqlDbType.SmallInt (số)
        + SqlDbType.Structured: A special data type for specifying structured data contained in table-valued parameters.
        + SqlDbType.TinyInt (số)
        + SqlDbType.VarBinary (tệp)
        + SqlDbType.VarChar (chữ)
        + SqlDbType.Xml (XML)
+ Use ExecuteNonQuery inside a SqlTransaction
+ SQL Server
+ IDENTITY IDENTITY(1,1)
+ SCOPE_IDENTITY()

## SQL Server: Create Table with IDENTITY Column
```
CREATE TABLE Users (
    UserId INT IDENTITY(1,1) PRIMARY KEY,
    UserName NVARCHAR(100) NOT NULL
);
```

## SQL Server: Inserting Rows (No need to specify UserId)
```
INSERT INTO Users (UserName)
VALUES ('Alice'), ('Bob'), ('Charlie');
```

## SQL Server: Query the last identity value inserted in the current scope
```
SELECT SCOPE_IDENTITY();
```

## SQL Server: Insert Explicit ID (Override IDENTITY)
```
SET IDENTITY_INSERT Users ON;

INSERT INTO Users (UserId, UserName)
VALUES (1001, 'Admin');

SET IDENTITY_INSERT Users OFF;
```

## App.config OR Web.config
```
<connectionStrings>  
    <clear />
    <add name="ConnStr" providerName="System.Data.SqlClient" connectionString="Data Source=localhost;Integrated security=SSPI;Initial Catalog=MyDatabase;" />  
    <!--By default, MARS is disabled when connecting to a MARS-enabled host. It must be enabled in the connection string. -->  
    <add name="ConnStrMARS" providerName="System.Data.SqlClient" connectionString="Data Source=localhost;Integrated security=SSPI;Initial Catalog=MyDatabase;MultipleActiveResultSets=True" />  
</connectionStrings> 
```

## ADO.NET C# Code Snippet
```
using System;
using System.Data.SqlClient;

string connStr = "your_connection_string";
int userId = 1001;
string userName = "john_doe";

using (var connection = new SqlConnection(connStr))
{
    connection.Open();
    using (var transaction = connection.BeginTransaction())
    {
        try
        {
            var cmd = connection.CreateCommand();
            cmd.Transaction = transaction;

            // Turn ON IDENTITY_INSERT
            cmd.CommandText = "SET IDENTITY_INSERT Users ON";
            cmd.ExecuteNonQuery();

            // Insert with explicit ID
            cmd.CommandText = "INSERT INTO Users (UserId, UserName) VALUES (@UserId, @UserName)";
            cmd.Parameters.AddWithValue("@UserId", userId);
            cmd.Parameters.AddWithValue("@UserName", userName);
            cmd.ExecuteNonQuery();

            // Turn OFF IDENTITY_INSERT
            cmd.CommandText = "SET IDENTITY_INSERT Users OFF";
            cmd.Parameters.Clear();
            cmd.ExecuteNonQuery();

            transaction.Commit();
        }
        catch
        {
            transaction.Rollback();
            throw;
        }
    }
}
```

## ADO.NET Example: Insert & Read IDENTITY Value

### 1. Auto-insert (get generated IDENTITY value)

```
using System;
using System.Data.SqlClient;

string connStr = "your_connection_string_here";

using (var conn = new SqlConnection(connStr))
{
    conn.Open();

    var cmd = new SqlCommand(@"
        INSERT INTO Users (UserName)
        VALUES (@UserName);
        SELECT SCOPE_IDENTITY();", conn);

    cmd.Parameters.AddWithValue("@UserName", "David");

    var newId = Convert.ToInt32(cmd.ExecuteScalar());
    Console.WriteLine($"Inserted user with ID: {newId}");
}
```

### 2. Note about SCOPE_IDENTITY()
SCOPE_IDENTITY() returns the last identity value inserted in the current scope.

# References
+ [Introduction to ADO.NET](https://www.sitepoint.com/introduction-ado-net/)
+ [IDbTransaction](https://learn.microsoft.com/en-us/dotnet/api/system.data.idbtransaction)
+ [ADO.NET DataSet] (https://dotnettutorials.net/lesson/ado-net-dataset)
+ [ADO.NET](https://www.csharp-tutorial.hu/csharp/accessing-sql-from-c-code-sqlconnection-sqlcommand-executenonquery-executescalar-sqltransaction/)
+ [ADO.NET MultipleActiveResultSets=True and NextResult](https://learn.microsoft.com/en-us/dotnet/api/system.data.idatareader.nextresult)

# System.Data.IDataReader: NextResult
```
using System;
using System.Data;
using System.Data.SqlClient;
using System.Data.OleDb;
using System.Data.Odbc;

class Program
{
    static void Main(string[] args)
    {
        string connectionTypeName = "OleDb";
        string connectionString = @"Provider=SQLNCLI11;Data Source=(local);Initial Catalog=AdventureWorks;Integrated Security=SSPI";

        using (IDbConnection connection = DatabaseConnectionFactory.GetConnection(connectionTypeName, connectionString))
        {
            IDbCommand command = connection.CreateCommand();
            command.Connection = connection;

            // these 2 queries are executed as a single batch and return 2 separate results
            command.CommandText = "SELECT CountryRegionCode, Name FROM Person.CountryRegion;" + "SELECT CountryRegionCode, StateProvinceCode, Name, StateProvinceID FROM Person.StateProvince;";

            connection.Open();

            using (IDataReader reader = command.ExecuteReader())
            {
                // process the first result
                DisplayCountryRegions(reader);

                // use NextResult to move to the second result and verify it is returned
                if (!reader.NextResult())
                    throw new InvalidOperationException("Expected second result (StateProvinces) but only one was returned");

                // process the second result
                DisplayStateProvinces(reader);

                reader.Close();
            }

            connection.Close();
        }
    }

    static void DisplayCountryRegions(IDataReader reader)
    {
        if (reader.FieldCount != 2)
            throw new InvalidOperationException("First resultset (CountryRegions) must contain exactly 2 columns");

        while (reader.Read())
        {
            Console.WriteLine("CountryRegionCode={0}, Name={1}"
                , reader.GetString(reader.GetOrdinal("CountryRegionCode"))
                , reader.GetString(reader.GetOrdinal("Name")));
        }
    }

    static void DisplayStateProvinces(IDataReader reader)
    {
        if (reader.FieldCount != 4)
            throw new InvalidOperationException("Second resultset (StateProvinces) must contain exactly 4 columns");

        while (reader.Read())
        {
            Console.WriteLine("CountryRegionCode={0}, StateProvinceCode={1}, Name={2}, StateProvinceID={3}"
                , reader.GetString(reader.GetOrdinal("CountryRegionCode"))
                , reader.GetString(reader.GetOrdinal("StateProvinceCode"))
                , reader.GetString(reader.GetOrdinal("Name"))
                , reader.GetInt32(reader.GetOrdinal("StateProvinceID")));
        }
    }

    class DatabaseConnectionFactory
    {
        static SqlConnection GetSqlConnection(string connectionString)
        {
            return new SqlConnection(connectionString);
        }

        static OdbcConnection GetOdbcConnection(string connectionString)
        {
            return new OdbcConnection(connectionString);
        }

        static OleDbConnection GetOleDbConnection(string connectionString)
        {
            return new OleDbConnection(connectionString);
        }

        public static IDbConnection GetConnection(string connectionTypeName, string connectionString)
        {
            switch (connectionTypeName)
            {
                case "SqlClient":
                    return GetSqlConnection(connectionString);
                case "Odbc":
                    return GetOdbcConnection(connectionString);
                case "OleDb":
                    return GetOleDbConnection(connectionString);
                default:
                    throw new ArgumentException("Value must be SqlClient, Odbc, or OleDb", "connectionTypeName");
            }
        }
    }
}
```

# Persist Security Info

```
// Chuỗi kết nối KHUYẾN NGHỊ (Persist Security Info=False là mặc định)
string connectionString1 = "Data Source=YourServer;Initial Catalog=YourDatabase;User ID=YourUser;Password=YourPassword;";

// Chuỗi kết nối HIỂN THỊ RÕ Persist Security Info=False
string connectionString2 = "Data Source=YourServer;Initial Catalog=YourDatabase;User ID=YourUser;Password=YourPassword;Persist Security Info=False;";

// Chuỗi kết nối KHÔNG KHUYẾN NGHỊ (tiềm ẩn rủi ro bảo mật)
string connectionString3 = "Data Source=YourServer;Initial Catalog=YourDatabase;User ID=YourUser;Password=YourPassword;Persist Security Info=True;";
```

# Table-Valued Parameters (TVPs)
+ How to Send a Complex Type List of Objects to SQL Server
+ Using Table-Valued Parameters in SQL Server and .NET
+ Passing Table-Valued Parameters From .NET To SQL Server

```
CREATE TYPE dbo.TableValuedTypeExample AS TABLE
(
    CustomerId INT NOT NULL,
    CustomerName NVARCHAR(MAX),
    PRIMARY KEY (CustomerId)
)
CREATE PROC dbo.InsertValue
(@TempTable AS dbo.TableValuedTypeExample READONLY)
AS
BEGIN
      INSERT INTO CUSTOMER (CustomerId, CustomerName, Isdeleted)
      SELECT CustomerId, CustomerName, 0 AS Isdeleted FROM @TempTable
END
```

* https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/sql/table-valued-parameters
* https://stackoverflow.com/questions/10409576/pass-table-valued-parameter-using-ado-net
* https://www.dbdelta.com/sql-server-tvp-performance-gotchas/
* https://www.sqlservercentral.com/blogs/table-valued-parameter-example/
* https://www.c-sharpcorner.com/UploadFile/ff2f08/table-value-parameter-use-with-C-Sharp/
* https://dev.to/edwinoaragon/how-to-send-a-complex-type-list-of-objects-to-sql-server-266b

# Table-Valued User-Defined Functions
+ How to: Use Table-Valued User-Defined Functions

```
CREATE FUNCTION ProductsCostingMoreThan(@cost money)  
RETURNS TABLE  
AS  
RETURN  
    SELECT ProductID, UnitPrice  
    FROM Products  
    WHERE UnitPrice > @cost
```

# Others
```
    ("Application Intent", "ApplicationIntent"),
    ("Connect Retry Count", "ConnectRetryCount"),
    ("Connect Retry Interval", "ConnectRetryInterval"),
    ("Pool Blocking Period", "PoolBlockingPeriod"),
    ("Multiple Active Result Sets", "MultipleActiveResultSets"),
    ("Multi Subnet Failover", "MultiSubnetFailover"),
    ("Transparent Network IP Resolution", "TransparentNetworkIPResolution"),
    ("Trust Server Certificate", "TrustServerCertificate")
```

# SQL Server Data Type Mappings (SqlDbType)
* https://learn.microsoft.com/en-us/dotnet/api/system.data.sqldbtype
* https://stackoverflow.com/questions/425389/c-sharp-equivalent-of-sql-server-datatypes
* https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/sql-server-data-type-mappings

# ADO.NET, Dapper, and EF Core
* https://khalidabuhakmeh.com/passing-table-valued-parameters-from-dotnet-to-sql-server
* https://khalidabuhakmeh.com/multiple-result-sets-with-net-core-sql-server

# Multiple Result Sets With .Net Core and SQL Server
+ Multiple Result Sets Using SqlCommand
+ Multiple Result Sets Using SqlDataAdapter
+ Multiple Result Sets Using Dapper
+ Multiple Result Sets Using NPoco
+ Multiple Result Sets Using Entity Framework Core
+ https://github.com/gtechsltn/AllTheThings
+ https://github.com/khalidabuhakmeh/allthethings
