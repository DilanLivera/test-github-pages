### How to setup SignalR with a .Net Core server and JavaScript client

#### Setup the hub in the server
1. Create a Hub
    ```C#
      using Microsoft.AspNetCore.SignalR;

      public class ViewHub : Hub
      {
          public static int ViewCounter { get; set; } = 0;

          public async Task NotifyWatchingAsync()
          {
              ViewCounter++;

              await Clients.All.SendAsync("updateViewCount", ViewCounter);
          }
      }
    ```

2. Add SingnalR services to `IServiceCollection` in `ConfigureServices` method of `Startup` class
    ```C#
      public void ConfigureServices(IServiceCollection services)
      {
          services.AddSignalR();
      }
    ```

3. Add an endpoint to the hub
    ```C#
      public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
      {
          app.UseEndpoints(routeBuilder =>
          {
              routeBuilder.MapHub<ViewHub>("/hubs/view");
          });
      }
    ```

#### Setup the client (.Net Core web application)
1. Add `@microsoft/signalr@latest` client-side library using `unkpg` provider. Set target location as `wwwroot/js/signalr/` in the Add Client-Side library window

2. Add `<script src="~/js/signalr/dist/browser/signalr.js"></script>` to `<body></body>` tag

3. Setup the SignalR connection in the `site.js` file
    ```JS
    "use strict";

    let connection = new signalR.HubConnectionBuilder()
      .withUrl("/hubs/view")
      .build();

    connection.on("updateViewCount", (number) => {
      var counter = document.getElementById("viewCounter");
      counter.innerText = number;
    });

    function notify() {
      connection.send("notifyWatchingAsync");
    }

    function startSuccess() {
      console.info("Connection started");
      notify();
    }

    function startFail() {
      console.error("Connection failed");
    }

    connection.start().then(startSuccess, startFail);
    ```


#### Links
- [Tutorial: Get started with ASP.NET Core SignalR](https://docs.microsoft.com/en-us/aspnet/core/tutorials/signalr?view=aspnetcore-5.0&tabs=visual-studio)
