---
title: '[C#] OleDbTransaction'
date: 2009-03-11 23:58:06
updated: 2009-03-11 23:58:06
categories:
- Backend
- C#
tags:
- C#
---
當我要同時對資料庫做兩個刪除一個新增的動作...
用一般的方法...OleDbCommand.ExecuteNonQuery
當在第二個刪除失敗的時候...
要去救回第一個刪除資料...
會很麻煩...

<!--more-->

這時可以使用這個物件...OleDbTransaction
它可以在有失敗時將資料救回...
這還蠻方便的...
省掉許多自己去寫維護的時間...

以下是MSDN的範例...
``` csharp
public void ExecuteTransaction(string connectionString)
{
        using (OleDbConnection connection = new OleDbConnection(connectionString))
        {
            OleDbCommand command = new OleDbCommand();
            OleDbTransaction transaction = null;

            // Set the Connection to the new OleDbConnection.
            command.Connection = connection;

            // Open the connection and execute the transaction.
            try
            {
                connection.Open();

                // Start a local transaction
                transaction = connection.BeginTransaction();

                // Assign transaction object for a pending local transaction.
                command.Connection = connection;
                command.Transaction = transaction;

                // Execute the commands.
                command.CommandText = "Insert into Region (RegionID, RegionDescription) VALUES (100, 'Description')";
                command.ExecuteNonQuery();
                command.CommandText = "Insert into Region (RegionID, RegionDescription) VALUES (101, 'Description')";
                command.ExecuteNonQuery();

                // Commit the transaction.
                transaction.Commit();
                Console.WriteLine("Both records are written to database.");
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
                try
                {
                    // Attempt to roll back the transaction.
                    transaction.Rollback();
                }
                catch
                {
                    // Do nothing here; transaction is not active.
                }
            }
            // The connection is automatically closed when the
            // code exits the using block.
        }
}
```