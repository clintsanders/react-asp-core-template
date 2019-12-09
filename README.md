# react-asp-core-template
Template project for new React web applications with Web API backend

The project was created by executing the following command in the root project directory

  `dotnet new react --auth individual -uld`
  
## Identity Server SQL Database Setup
Apply entity framework migrations to create tables used by Identity Server
From the package manager console execute the following command
`PM> update-database`


A mssql database file is created and by default is placed in the user directory
`C:\Users\{Current User}`
This directory is not accessible in SSMS when trying to attach the database file. Move the .mdf and .ldf files to the following location for development purposes then attach the database to the local sql server to view the table data.
`C:\Program Files\Microsoft SQL Server\MSSQLXX.SQLEXPRESS\MSSQL\DATA`

Update the connection string in AppSettings.json
```json
"ConnectionStrings": {
  "IdentityConnection": "Server=INS15-CS\\SQLEXPRESS;Database=identity-users;Trusted_Connection=True;MultipleActiveResultSets=true"
}
```

In Startup.cs update the name of the connection string created in AppSettings.json in ConfigureServices()

```c#
services.AddDbContext<ApplicationDbContext>(options =>
  options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
```
  
## Configure IIS Express to use self-signed certificate
Usually on the dev machine you will get the error 

>`This site can't be reached localhost refused to connect` 

This is because the certifcate needs to be create in IIS Express for the port you will be debugging the projecct on.  To configure this for a new project first close Visual Studio and delete the .vs folder. This folder and config files will be re-created when you launch Visual Studio again.

Open cmd with admin privileges in the directory C:\Program Files (x86)\IIS Express and execute the following command

`iisexpressadmincmd.exe setupsslurl -url:https://localhost:12345/ -useselfsigned` 

The port number for HTTPS used by visual studio when launching IIS Express can be found in the launchSettings.json under the Properties folder.

Relaunch Visual Studio and you should be able view the index.html page in your browser.

## Launch React application on its own URL
One downside to running both the React and ASP.NET Core parts of the application together is that any changes to the C# code will trigger a recompile and restart of the application, rendering the React front-end unresponsive for a few seconds.  To work around this you can choose to launch the React part of the applicaition seperately on its own port.

Add a .env.development file to the ClientApp directory

`cmd> type nul > .env.development`

Add the following browser settings

`BROWSER=none`

`HTTPS=true`

`REACT_APP_API_URL=https://localhost:44392`

The port number used in the url setting can be found in properties/launchSettings.json under iisSettings

Now you will need to launch the React application manually.  From the ClientApp dir execute the following command

`npm start`

Next modify startup.cs so that React uses the separate dev server

```c#
if (env.IsDevelopment())
{
  //spa.UseReactDevelopmentServer(npmScript: "start");
  spa.UseProxyToSpaDevelopmentServer("https://localhost:3000");
}
```

Add the following to startup.cs to allow CORS for the app to utilize the API running the separate iis express dev server in the following function

```c#
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
  //make sure this is called before app.UseMVC otherwise it will have no effect on the incoming requests
  app.UseCors(builder => builder.WithOrigins("https://localhost:3000"));  
}
```

The last modification is to update FetchData.js to use api url setting that was added in .env.development
```javascript
async populateWeatherData() {
  const response = await fetch(`${process.env.REACT_APP_API_URL}/api/weatherforecast`);
  const data = await response.json();
  this.setState({ forecasts: data, loading: false });
 }
```

