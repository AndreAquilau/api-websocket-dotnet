# API WebSocket .NET

### Settings Application

```cs
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

var webSocketOptions = new WebSocketOptions
{
    KeepAliveInterval = TimeSpan.FromMinutes(2)
};

webSocketOptions.AllowedOrigins.Add("*");

app.UseWebSockets(webSocketOptions);

//app.UseHttpsRedirection();

// app.UseAuthorization();

app.MapControllers();

app.Run();
```

### Template Base Controller

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using System.Net.WebSockets;
using System.Text;

namespace WebSocketServer.Controllers;

[ApiController]
public class SocketController : ControllerBase
{
    [Route("/ws")]
    public async Task Get()
    {
        if (HttpContext.WebSockets.IsWebSocketRequest)
        {
            using var webSocket = await HttpContext.WebSockets.AcceptWebSocketAsync();

            await Echo(webSocket);
        }
        else
        {
            HttpContext.Response.StatusCode = StatusCodes.Status400BadRequest;
        }
    }

    private async Task Echo(WebSocket webSocket)
    {
        byte[] buffer = new byte[1024 * 24];

        Console.WriteLine("Connection...");

        var received = await webSocket.ReceiveAsync(new ArraySegment<byte>(buffer), CancellationToken.None);

        while (!received.CloseStatus.HasValue)
        {
            var str = Encoding.UTF8.GetString(buffer);
            Console.WriteLine(str);

            await webSocket.SendAsync(
                new ArraySegment<byte>(buffer, 0, received.Count),
                received.MessageType,
                received.EndOfMessage,
                CancellationToken.None
            );

            received = await webSocket.ReceiveAsync(new ArraySegment<byte>(buffer), CancellationToken.None);
        }

        Console.WriteLine("Desconnect...");

        await webSocket.CloseAsync(
            received.CloseStatus.Value,
            received.CloseStatusDescription,
            CancellationToken.None
        );

    }
}
```
