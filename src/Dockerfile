FROM microsoft/dotnet:2.2-sdk-alpine AS build

# Set working directory within container
WORKDIR /app

# Copy .csproj and restore as distinct layers
COPY *.csproj ./
RUN dotnet restore

# Copy everything else and build
COPY . ./
RUN dotnet publish -c Release -o dist

# Build runtime image
FROM microsoft/dotnet:2.2-aspnetcore-runtime-alpine AS runtime
COPY --from=build /app/dist ./
ENTRYPOINT ["dotnet", "app.dll"]

# docker build . -t tonysneed/aspnetapp-2.2
# docker run -it --rm -p 8080:80 --name myapp tonysneed/aspnetapp-2.2
