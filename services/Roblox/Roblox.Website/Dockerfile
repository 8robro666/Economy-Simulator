FROM mcr.microsoft.com/dotnet/sdk:6.0 AS builder

WORKDIR /src

COPY ["services/Roblox/Roblox.Website/Roblox.Website.csproj", "services/Roblox/Roblox.Website/"]
COPY ["services/Roblox/Roblox.AbuseDetection/Roblox.AbuseDetection.csproj", "services/Roblox/Roblox.AbuseDetection/"]
COPY ["services/Roblox/Roblox.Configuration/Roblox.Configuration.csproj", "services/Roblox/Roblox.Configuration/"]
COPY ["services/Roblox/Roblox.Dto/Roblox.Dto.csproj", "services/Roblox/Roblox.Dto/"]
COPY ["services/Roblox/Roblox.EconomyChat/Roblox.EconomyChat.csproj", "services/Roblox/Roblox.EconomyChat/"]
COPY ["services/Roblox/Roblox.Libraries/Roblox.Libraries.csproj", "services/Roblox/Roblox.Libraries/"]
COPY ["services/Roblox/Roblox.Models/Roblox.Models.csproj", "services/Roblox/Roblox.Models/"]
COPY ["services/Roblox/Roblox.Services/Roblox.Services.csproj", "services/Roblox/Roblox.Services/"]
COPY ["services/Roblox/Roblox.Exceptions/Roblox.Exceptions.csproj", "services/Roblox/Roblox.Exceptions/"]

RUN dotnet restore "services/Roblox/Roblox.Website/Roblox.Website.csproj"

COPY . .

WORKDIR "/src/services/Roblox/Roblox.Website"
RUN dotnet publish "Roblox.Website.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS runner

WORKDIR /app

COPY --from=builder /app/publish .

EXPOSE 80
EXPOSE 443

ENV ASPNETCORE_URLS=http://+:80

ENTRYPOINT ["dotnet", "Roblox.Website.dll"]