
FROM mcr.microsoft.com/dotnet/sdk:3.1

WORKDIR /App
    
COPY . .

ARG BUILD_CONFIGURATION=Debug
ENV ASPNETCORE_ENVIRONMENT=Development
ENV DOTNET_USE_POLLING_FILE_WATCHER=true  
ENV ASPNETCORE_URLS=http://+:5000  

CMD ["dotnet", "run"]