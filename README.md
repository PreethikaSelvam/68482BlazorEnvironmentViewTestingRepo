# EnvironmentView validation

Sample used to validate `EnvironmentView` for [dotnet/aspnetcore#68482](https://github.com/dotnet/aspnetcore/issues/68482).

The test covered Static SSR, Interactive Server, Interactive WebAssembly, Interactive Auto, and standalone WebAssembly in Development, Production, Staging, and QA. All tested cases passed.

## Setup

The tested SDK was `11.0.100-preview.7.26381.103`.

```powershell
dotnet build .\BlazorWebApp\BlazorWebApp.sln
dotnet build .\StandaloneWebAssembly\StandaloneWebAssembly.csproj
```

Run the hosted app:

```powershell
dotnet run --project .\BlazorWebApp\BlazorWebApp\BlazorWebApp.csproj --launch-profile http
```

| Route | Mode |
| --- | --- |
| `/environment-ssr` | Static SSR |
| `/environment-server` | Interactive Server |
| `/environment-webassembly` | Interactive WebAssembly |
| `/environment-auto` | Interactive Auto |
| `/environmentview-discovery` | API discovery |

Run the standalone app at `http://localhost:5223/`:

```powershell
dotnet run --project .\StandaloneWebAssembly\StandaloneWebAssembly.csproj --launch-profile http
```

## Environments

The hosted app selects its environment when it starts:

```powershell
dotnet publish .\BlazorWebApp\BlazorWebApp\BlazorWebApp.csproj -c Release -o .\artifacts\hosted-publish
$env:ASPNETCORE_ENVIRONMENT = "Production" # Development, Production, Staging, or QA
dotnet .\artifacts\hosted-publish\BlazorWebApp.dll --urls http://localhost:5187
```

Use a separate client environment when required:

```powershell
dotnet publish .\BlazorWebApp\BlazorWebApp\BlazorWebApp.csproj -c Release -o .\artifacts\hosted-client-qa-retest
cd .\artifacts\hosted-client-qa-retest
$env:ASPNETCORE_ENVIRONMENT = "Production"
$env:CLIENT_ENVIRONMENT_OVERRIDE = "QA"
dotnet .\BlazorWebApp.dll --urls http://localhost:5192
```

The standalone environment is set at publish time:

```powershell
$environment = "QA" # Development, Production, Staging, or QA
dotnet publish .\StandaloneWebAssembly\StandaloneWebAssembly.csproj -c Release -o ".\artifacts\standalone-$environment-publish" -p:WasmApplicationEnvironmentName=$environment
python -m http.server 5225 --directory ".\artifacts\standalone-$environment-publish\wwwroot"
```

Open `http://localhost:5225/` for standalone WebAssembly.

## Evidence

- SDK and baseline build output: [`Evidence/Build`](Evidence/Build)
- Published-build output and environment records for every tested environment: [`Evidence/Build/PublishedEnvironments`](Evidence/Build/PublishedEnvironments)
- Exact test environment versions: [`Evidence/Build/test-environment.txt`](Evidence/Build/test-environment.txt)
- Screenshots and videos: [`Evidence/TestCasesAndOutput`](Evidence/TestCasesAndOutput)
- Full test report: [`Evidence/68482-EnvironmentViewTestReport.docx`](Evidence/68482-EnvironmentViewTestReport.docx)

## Recorded test environment

- Operating system: Windows 11 x64
- .NET SDK: `11.0.100-preview.7.26381.103`
- Google Chrome: `151.0.7922.174`
- Visual Studio Code: `1.134.0`