# TCPServer
IServer implementation for ASP.NET Core using bare TCP socket at the transport layer.

This has been create to have a minimal HTTP server which does not depends on Kestrel.

Kestrel is meant to be a complete HTTP server. This is an incomplete TCP Server using HTTP semantic to reuse HttpClient/ASP.NET Core.

Nuget:

The package is hosted on [nuget](https://www.nuget.org/packages/TCPServer).
```
dotnet add package TCPServer
```

Usage:

Create and run a server:
```
private IWebHost CreateHost()
{
	var host = new WebHostBuilder()
		.UseStartup<Startup>()
		.UseTCPServer(new ServerOptions(new IPEndPoint(IPAddress.Parse("0.0.0.0"), 29472)))
		.Build();
	host.Start();
	return host;
}
```

How to call it with HttpClient:

```
IWebHost host = CreateHost();
using(var client = new HttpClient(new TCPHttpMessageHandler()))
{
	var nico = client.GetAsync("http://127.0.0.1:29472/v1/hello/nico").Result.Content.ReadAsStringAsync().Result;
	Assert.Equal("\"nico\"", nico);
	nico = client.GetAsync("http://127.0.0.1:29472/v1/hello/nico?test=toto").Result.Content.ReadAsStringAsync().Result;
	Assert.Equal("\"nicototo\"", nico);


	var error = Assert.Throws<HttpRequestException>(() => client.GetAsync("http://127.0.0.1:29472/v1/badrequest/nico").Result.EnsureSuccessStatusCode());
	Assert.Contains("400", error.Message);

	nico = client.PostAsync("http://127.0.0.1:29472/v1/hellojson/", new StringContent("{ \"Name\" : \"Nicoo\" }", Encoding.UTF8, "application/json")).Result.Content.ReadAsStringAsync().Result;
	Assert.Equal("\"Nicoo\"", nico);
}
```

This is assuming this simple Controller:

```
public class MainController : Controller
{
	[HttpGet]
	[Route("v1/hello/{hello}")]
	public IActionResult Hello(string hello, string test)
	{
		return Json(hello + test);
	}

	[HttpPost]
	[Route("v1/hellojson")]
	public IActionResult HelloJson([FromBody]Request req)
	{
		return Json(req.Name);
	}

	[HttpPost]
	[Route("v1/nothing")]
	public IActionResult Nothing([FromBody]Request req)
	{
		return new NoResult();
	}

	[HttpGet]
	[Route("v1/badrequest/{hello}")]
	public IActionResult Bad(string hello)
	{
		return BadRequest("boom");
	}

	[HttpGet]
	[Route("v1/timeout")]
	public IActionResult Timeout()
	{
		Thread.Sleep(10000);
		return BadRequest("boom");
	}
}
```
