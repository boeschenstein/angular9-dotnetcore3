# Angular 9 (Frontend) and .NET Core 3.1 (Backend)

## Goal

The goal is to create an Angular demo app with .NET Core backend in the following folder structure. In my eyes, this separates the projects better then the Angular template in Visual Studio. It is a recommended way to separate frontend and backend:

```cmd
rem Angular code in frontend folder
<myApp>\frontend\...

rem .NET Core project in backend folder
<myApp>\backend\...
```

## What you should bring

Some basic understanding of

- Windows
- .NET and C#
- npm, node
- Web technology

## New WebAPI project (backend)

### Backend Preparation

Install "Visual Studio 2019" from <https://visualstudio.microsoft.com/>. The free "Community" edition is just fine.

### Create new .NET WebAPI project

- Create a folder "c:\temp\myAngular9NetCore3App"
- Start cmd in this folder
- Now create a new webapi project in cmd:

```cmd
dotnet new webapi -n MyBackend
```

Rename the new folder from "MyBackend" to "backend".

In Visual Studio, load this project: "c:\temp\myAngular9NetCore3App\backend\MyBackend.csproj"

Configure the green arrow in the toolbar (use the small arrow on the right side of the button): choose "MyBackend" instead of "IIS Express". Run the project (press F5).

Your Browser will spin up and shows this URL: <https://localhost:5001/weatherforecast>

What you see here is some random json data from the demo controller. (see Controllers\WeatherForecastController.cs). Due to randomizing, every time you refresh the browser, you'll get different data.

## New Angular 9 project (frontend)

### Frontend Preparation

- Make sure you have the latest LTS of node/npm installed: <https://nodejs.org/en/download/>
- Install an editor for your Angular frontend. I use VS Code: <https://visualstudio.microsoft.com/>
- Install the Angular cli globally. Details see <https://angular.io/guide/setup-local>

Install the Angular CLI from cmd:

``` cmd
rem -g: globally in c:\Users\<username>\AppData\Roaming\npm\node_modules\

npm install -g @angular/cli
```

### Create new Angular project

Open a cmd in this folder "c:\temp\myAngular9NetCore3App"

Create new Angular app in cmd:

``` cmd
rem --routing true: add routing
rem --style scss: scss is a modern css preprocessor
rem -g: skip git

ng new MyFrontend --routing true --style scss -g
```

Rename the new folder "MyFrontend" to "frontend".

Run the new Angular app from cmd:

``` cmd
rem set \frontend as the start folder

cd c:\temp\myAngular9NetCore3App\frontend

rem s or serve: compile and start a small webserver
rem -o or --open: open browser

ng s -o
```

Your Browser will spin up and shows URL: <http://localhost:4200/>

What you see here is the Angular demo website.

## Put it all together

### Frontend: Load Data from Backend

Open cmd in "c:\temp\myAngular9NetCore3App\frontend". Enter "code ." to spin up VS Code in the right folder:

``` cmd
rem 'code .' starts VS Code and sets the current folder as work directory

cd c:\temp\myAngular9NetCore3App\frontend

code .
```

Press Control-C in cmd or close browser to stop the web app.
Edit \src\app\app.module.ts: Add HttpClientModule:

``` typescript
...
import { HttpClientModule } from '@angular/common/http';

  imports: [
    ...
    HttpClientModule
  ],

```

Call the backend: add some imports to  app.component.ts;

``` typescript
import { Component, Inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
```

And add a constructor to the class, which gets the HttpClient injected:

``` typescript
  constructor(http: HttpClient) {
    http.get<any[]>('https://localhost:5001/weatherforecast').subscribe(result => {
      console.warn("weatherforecast", result);
    }, error => console.error(error));
  }
```

<details>
  <summary>The code will look like this:</summary>

``` typescript

import { Component, Inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = 'MyFrontend';
  constructor(http: HttpClient) {
    http.get<any[]>('https://localhost:5001/weatherforecast').subscribe(result => {
      console.warn("weatherforecast", result);
    }, error => console.error(error));
  }
}
```

</details>

### Check new Angular app

If it is not already running, start .NET Core Backend by pressing F5 in Visual Studio.

Now start frontend project:

``` cmd
rem Go to the frontend folder
cd c:\temp\myAngular9NetCore3App\frontend

rem This will spin up a web server and a browser on http://localhost:4200
ng s --o
```

> Note: If you get an error "Port 4200 is already in use", you have it already running in another cmd.

Open Debugger tools in your browser by pressing Shift-Control-I. Refresh the console output and pressing F5.

Now you'll see a CORS error in the console output:

![CORS error](/images/cors_error.png)

### Fix CORS error

CORS is a web security feature: <https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS>

Backend and Frontend are served on different locations (ports), therefore we need to allow this access.

In the Backend, add CORS Policy to ConfigureServices (Startup.cs):

``` c#
readonly string MyAllowSpecificOrigins = "_myAllowSpecificOrigins";

public void ConfigureServices(IServiceCollection services)
{
    services.AddCors(options =>
    {
        options.AddPolicy(name: MyAllowSpecificOrigins,
            builder =>
            {
                builder.WithOrigins("http://localhost:4200"); // Frontend URL
            });
    });

    services.AddControllers();
}
```

Use CORS Policy in Configure (Startup.cs):

``` c#
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...
    app.UseRouting();
    app.UseCors(MyAllowSpecificOrigins); // AFTER app.UseRouting();
    ...
}
```

## Check the application

1. Start your new Backend
2. Start your new Frontend
3. Check Console Output: you should see an array of data:

![Success :)](/images/array.png)

If you can see the array, you just created your first business application in Angular 9 and .NET core 3.1. Congratulations! Please let me know on twitter ![@patrikbo](https://avatars3.githubusercontent.com/u/50278?s=14&v=4) [@patrikbo](https://twitter.com/patrikbo). Thank you!

### Whats next

Swagger/OpenApi are tools which can create your Angular code to access the backend: check this <https://github.com/boeschenstein/angular9-dotnetcore-openapi-swagger>

## Additional Information

### Links

- ASP.NET WebApi: <https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api>
- Angular: <https://angular.io/>
- CORS: <https://docs.microsoft.com/en-us/aspnet/core/security/cors>

### Current Versions

- Visual Studio 2019 16.5.3
- .NET core 3.1
- npm 6.14.4
- node 12.16.1
- Angular CLI 9.1
