# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS builder

WORKDIR /src
COPY cartservice.csproj ./
RUN dotnet restore

COPY . .
RUN dotnet publish -c Release -o /out --no-restore

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0

WORKDIR /app
COPY --from=builder /out ./

ENV ASPNETCORE_HTTP_PORTS=7070

EXPOSE 7070

USER 1000
ENTRYPOINT ["dotnet", "cartservice.dll"]
