# SQL



 
 
 
 

  string connetionString;
   SqlConnection cnn;
   connetionString = @"Data Source=WIN-50GP30FGO75;Initial Catalog=Demodb;User ID=sa;Password=demol23";
   cnn = new SqlConnection(connetionString);
   cnn.Open();
   MessageBox.Show("Connection Open  !");
   cnn.Close();

using System;
using System.Data.SqlClient;
class Program
{
    static void Main()
    {
        //
        // First access the connection string.
        // ... This may be autogenerated in Visual Studio.
        //
        string connectionString =
            ConsoleApplication1.Properties.Settings.Default.ConnectionString;
        //
        // In a using statement, acquire the SqlConnection as a resource.
        //
        using (SqlConnection con = new SqlConnection(connectionString))
        {
            //
            // Open the SqlConnection.
            //
            con.Open();
            //
            // This code uses an SqlCommand based on the SqlConnection.
            //
            using (SqlCommand command = new SqlCommand("SELECT TOP 2 * FROM Dogs1", con))
            using (SqlDataReader reader = command.ExecuteReader())
            {
                while (reader.Read())
                {
                    Console.WriteLine("{0} {1} {2}",
                        reader.GetInt32(0), reader.GetString(1), reader.GetString(2));
                }
            }
        }
    }
}
 
using System;
using System.Data.SqlClient;
class Program
{
    static void Main()
    {
        //
        // The name we are trying to match.
        //
        string dogName = "Fido";
        //
        // Use preset string for connection and open it.
        //
        string connectionString = ConsoleApplication1.Properties.Settings.Default.ConnectionString;
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            connection.Open();
            //
            // Description of SQL command:
            // 1. It selects all cells from rows matching the name.
            // 2. It uses LIKE operator because Name is a Text field.
            // 3. @Name must be added as a new SqlParameter.
            //
            using (SqlCommand command = new SqlCommand("SELECT * FROM Dogs1 WHERE Name LIKE @Name", connection))
            {
                //
                // Add new SqlParameter to the command.
                //
                command.Parameters.Add(new SqlParameter("Name", dogName));
                //
                // Read in the SELECT results.
                //
                SqlDataReader reader = command.ExecuteReader();
                while (reader.Read())
                {
                    int weight = reader.GetInt32(0);
                    string name = reader.GetString(1);
                    string breed = reader.GetString(2);
                    Console.WriteLine("Weight = {0}, Name = {1}, Breed = {2}", weight, name, breed);
                }
            }
        }
    }
}

using System;
using System.Data;
using System.Data.SqlClient;
class ParamDemo
{
 static void Main()
 {
  // conn and reader declared outside try
  // block for visibility in finally block
  SqlConnection conn   = null;
  SqlDataReader reader = null;
  string inputCity = "London";
  try
  {
   // instantiate and open connection
   conn =  new 
    SqlConnection("Server=(local);DataBase=Northwind;Integrated Security=SSPI");
   conn.Open();
   // don't ever do this
            // SqlCommand cmd = new SqlCommand(
            // "select * from Customers where city = '" + inputCity + "'";
   // 1. declare command object with parameter
   SqlCommand cmd = new SqlCommand(
    "select * from Customers where city = @City", conn);
   // 2. define parameters used in command object
   SqlParameter param  = new SqlParameter();
   param.ParameterName = "@City";
   param.Value         = inputCity;
   // 3. add new parameter to command object
   cmd.Parameters.Add(param);
   // get data stream
   reader = cmd.ExecuteReader();
   // write each record
   while(reader.Read())
   {
    Console.WriteLine("{0}, {1}", 
     reader["CompanyName"], 
     reader["ContactName"]);
   }
  }
  finally
  {
   // close reader
   if (reader != null)
   {
    reader.Close();
   }
   // close connection
   if (conn != null)
   {
    conn.Close();
   }
  }
 }
}
 
 
Building Asynchronous Task Functions
We start by building the initial wrapper which returns a task object. We shall be returning a dataset in this example.
Side note: Any data type can be returned with task objects, as such you could return a string, integer or a custom class.
The following is the basic wrapper for our function. This bit of code will return a task object based on the datatype, with a section to add our own code to build and return the dataset.

public Task<DataSet> GetDataSetAsync(string sConnectionString, string sSQL)
{
    return Task.Run(() =>
    {
        //YOUR CODE HERE
    });
}
We can now insert code which connects to the server to get data based on a query into this function.
Example 1: Function to Fill a dataset Based on the SQL Query
In this example, code has been added to the body of the function which uses a standard SQL connection string and objects from the System.Data.SQLClient .NET library to connect to the database and fill a dataset based on the query.
Once the dataset has been filled, the code will then return the dataset using a standard "return" call.

public Task<DataSet> GetDataSetAsync
(string sConnectionString, string sSQL, params SqlParameter[] parameters)
{
    return Task.Run(() =>
    {
        using (var newConnection = new SqlConnection(sConnectionString))
        using (var mySQLAdapter = new SqlDataAdapter(sSQL, newConnection))
        {
            mySQLAdapter.SelectCommand.CommandType = CommandType.Text;
            if (parameters != null) mySQLAdapter.SelectCommand.Parameters.AddRange(parameters);
            DataSet myDataSet = new DataSet();
            mySQLAdapter.Fill(myDataSet);
            return myDataSet;
        }
    });
}
We can now call this function asynchronously to return a dataset of data based on our query.
Example 2: Function to Execute a Command, Passing Back the Rows Affected
In this example, we have changed the data type to integer, as we simply want to pass back the rows affected for the result of this function.
public Task<int> ExecuteAsync(string sConnectionString, string sSQL, params SqlParameter[] parameters)
{
    return Task.Run(() =>
    {
        using (var newConnection = new SqlConnection(sConnectionString))
        using (var newCommand = new SqlCommand(sSQL, newConnection))
        {
            newCommand.CommandType = CommandType.Text;
            if (parameters != null) newCommand.Parameters.AddRange(parameters);
            newConnection.Open();
            return newCommand.ExecuteNonQuery();
        }
    });
}
Further to this, there are existing async task functions on the SqlCommand object which can be used to simplify the function further. 

public async Task<int> ExecuteAsync(string sConnectionString, 
string sSQL, params SqlParameter[] parameters)
{
    using (var newConnection = new SqlConnection(sConnectionString))
    using (var newCommand = new SqlCommand(sSQL, newConnection))
    {
        newCommand.CommandType = CommandType.Text;
        if (parameters != null) newCommand.Parameters.AddRange(parameters);
        await newConnection.OpenAsync().ConfigureAwait(false);
        return await newCommand.ExecuteNonQueryAsync().ConfigureAwait(false);
    }
}
We can use this function to execute SQL and get the rows affected.
Calling Asynchronous Task Functions
Now that we have an async function to get data from SQL, we can now call this asynchronously using the async\await clause from any method.
To call the function to get data, we start by adding "async" into the method declaration.

private async <type> MyMethod()
Now that our method is an "async" method, we can use the "await" option on the tasks returned from our SQL functions.
Execute Task Function
To execute task functions, you need to add "await" before the function call.
This will execute the function, and return the "Result" to the variable should one be specified.

private async Task GetSomeData(string sSQL)
{
    //Use Async method to get data
    DataSet results = await GetDataSetAsync(sConnectionString, sSQL, sqlParams);
    //Populate once data received
    grdResults.DataSource = results.Tables[0];
}
Variables do not need to be specified. You can call the function using "await" which will run the task without retrieving a result.

private async Task ExecuteSomeData(string sSQL)
{
    //Use Async method to get data
    await ExecuteAsync(sConnectionString, sSQL, sqlParams);
}
The "await" option can be specified multiple times if you need to make other asyncronous calls.
You can make GUI updates before and after the "await" command, as it is only the await command which is run asynchronously.
The simplest use of "async/await" is to use the "await" option, then continue with your code afterwards.
However, the .NET task object provides other methods of continuation once the asynchronous task has completed.
Running Multiple Tasks/Chaining Tasks
If you need to run multiple SQL operations, there are a few ways of achieving this.
Specifying Tasks As Variables Then Running Together
You can specify a number of commands as variables and run them all at once asynchronously, then get the results as necessary.
This can be achieved by setting up your tasks as variables.
Once setup, you will need to add them into an array then use the "Task.WhenAll" to run all the tasks asynchronously.
You can then access the results on the task object after completion.

//Setup tasks
Task<int> ExecuteTask1 = database.ExecuteAsync(sConnectionString, sExecuteSQL1, sqlParams);
Task<int> ExecuteTask2 = database.ExecuteAsync(sConnectionString, sExecuteSQL2, sqlParams);
Task<int> ExecuteTask3 = database.ExecuteAsync(sConnectionString, sExecuteSQL3, sqlParams);
//Add to array
Task<int>[] Tasks = new Task<int>[] { ExecuteTask1, ExecuteTask2, ExecuteTask3 };
//Run all
await Task.WhenAll(Tasks);
//Get results
int iRowsAffected = Tasks[0].Result + Tasks[1].Result + Tasks[2].Result;
Calling Other Async Tasks With "ContinueWith"
Another method of chaining the tasks together is using the "ContinueWith" method on the task object returned by your SQL function.
In this example, we start by running an execute task function, then once complete, we run a select task function to get a dataset to a variable.

DataSet results = await database.ExecuteAsync(sConnectionString, sExecuteSQL, sqlParams).ContinueWith
(t => database.GetDataSetAsync(sConnectionString, sGetSQL, sqlParams)).Result;
Calling Standard Methods With "ContinueWith"
You can call a standard method on the "ContinueWith" method.
In this example, we use "ContinueWith" on the task object to pass the data through to a separate method.

private async Task RunSQLQuery(string sSQL)
{
    //Use await method to get data
    await GetDataSetAsync
         (sConnectionString, sSQL, sqlParams).ContinueWith(t => PopulateResults(t.Result));
}
private void PopulateResults(DataSet results)
{
    //Do something with results
}
WARNING: When using "ContinueWith" in this way, this will continue the async thread, meaning that if you intend to update GUI components such as a grid or textbox control, you may encounter the error:
Cross-thread operation not valid: Control MyControl accessed from a thread other than the thread it was created on.
In cases like this, you can invoke the method from within the "ContinueWith" which will allow for GUI updates.
await database.GetDataSetAsync(sConnectionString, sSQL, sqlParams).ContinueWith
(t => this.Invoke((Action)(() => { PopulateResultsToScreen(t.Result); })));
 
