# Ai2.DbCore


## What is Ai2.DbCore
  Ai2.DbCore is a C# DataBase query engine . It uses Dapper for db execute and queries , simplifies the operation of user transactions. 

## How to install
   Ai2.DbCore can't be used directly in a project. it should be referenced from Ai2.MySql, Ai2.SQLite, Ai2.SqlServer by nuget package.
   
## How to use
  If you want use Ai2.DbCore , you should supply your db provide , then create a db connection.
  
  ### Example:
  
    1.Install nuget package
    
       Install-Package Ai2.SqlServer 
       
    2.Generate a DbProvider
    
      namespace WpfApp1
      {
          class LocalSqlDbProvider : SqlDbProvider
          {
              public override IDbConnection GetConnection()
              {
                  var dbServer = new DbServerInfo
                  {
                      Ip = ".",
                      LoginName = "sa",
                      LoginPwd = "abc123!@#",
                      DataBase = "Test",
                      ConnectionTimeout = 3,
                      Extra = "min pool size = 2;max pool size=50000" 
                  };

                  return SqlConnectionFactory.GetConnection(dbServer);
              }
          }
      }
      
    3.Create a data entity
    
      [TableEntity("tb_User")]
      class User : IPropertyChanged
      {
          [Field(HasDefaultValue = true, IsReadOnly = true, Action = UserDataInputType.DataBaseAutoSetDefault)]
          public Guid Id { get; set; }

          [Field]
          public string LoginName { get; set; }

          [Field]
          public string LoginPwd { get; set; }


          [Field(IsReadOnly = true, Action = UserDataInputType.InternalUserInput)]
          public string LoginSalt { get; set; }

          [Field]
          public int IsBuildIn { get; set; }

          [Field]
          public int State { get; set; }

          [Field(HasDefaultValue = true,IsReadOnly = true, Action = UserDataInputType.InternalUserInput)]
          public DateTime CreateTime { get; set; }

          public void Reset()
          {

          }

          public List<PropertyChangedItem> GetChangedItems()
          {
              return null;
          }
      }
    
    
     4.OR entity likes 
   
        class User
        {
            public Guid Id { get; set; }

            public string LoginName { get; set; }

            public string LoginPwd { get; set; }

            public string LoginSalt { get; set; }

            public int IsBuildIn { get; set; }

            public int State { get; set; }

            public DateTime CreateTime { get; set; }
        }
        
    5.Do the query
    
        private void BtnClick(object sender, RoutedEventArgs e)
        {
            var unitOfWork = new UnitOfWork<LocalSqlDbProvider>();

            var user = new User
            {
                LoginName = "goldli",
                LoginPwd = "asdfasdfadsfads",
                LoginSalt = "adfadsfasdfa",
                State = 1
            };

            using (var iResult = unitOfWork.Insert<User,Guid>(user,out var id))
            {
                if (!iResult.IsFullSucceed)
                {
                    Debug.WriteLine(iResult.Error);
                    return;
                }


                Debug.WriteLine(id);

                var sql = "insert into tb_user(LoginName,LoginPwd) values (@name, @pwd)";
                var obj = new
                {
                    name = 123,
                    pwd = 123
                };


                using (var xResult = unitOfWork.DbContext.Insert(sql, obj, out id))
                {
                    if (!xResult.IsFullSucceed)
                    {
                        Debug.WriteLine(xResult.Error);
                    }
                    else
                    {
                        Debug.WriteLine(id);
                    }
                }
            }
        }
        
    6. the table sql is 
    
      CREATE TABLE tb_User
      (
          Id UNIQUEIDENTIFIER ROWGUIDCOL NOT NULL PRIMARY KEY DEFAULT(NEWSEQUENTIALID()),
        LoginName VARCHAR(50) NOT NULL,
        LoginPwd CHAR(28) NOT NULL,
        LoginSalt CHAR(36) NOT NULL,
        [IsBuildIn] INT NOT NULL DEFAULT(0),
        [State] INT NOT NULL DEFAULT(0)
        CreateTime DATETIME NOT NULL DEFAULT(GETDATE())
      )
      GO
    
    
