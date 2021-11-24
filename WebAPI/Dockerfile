FROM mcr.microsoft.com/dotnet/aspnet:5.0
ENV ASPNETCORE_URLS=http://+:80
EXPOSE 80
COPY bin/Debug/net5.0/publish/ app/
WORKDIR /app
ENTRYPOINT dotnet FieldEngineerApi.dll
