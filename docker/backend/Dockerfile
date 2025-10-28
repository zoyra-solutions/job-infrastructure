# Backend API Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy project files
COPY ["Recruitment.API/Recruitment.API.csproj", "Recruitment.API/"]
COPY ["Recruitment.Application/Recruitment.Application.csproj", "Recruitment.Application/"]
COPY ["Recruitment.Infrastructure/Recruitment.Infrastructure.csproj", "Recruitment.Infrastructure/"]

# Restore packages
RUN dotnet restore "Recruitment.API/Recruitment.API.csproj"

# Copy source code
COPY . .

# Build the application
WORKDIR "/src/Recruitment.API"
RUN dotnet build "Recruitment.API.csproj" -c Release -o /app/build

# Publish the application
FROM build AS publish
RUN dotnet publish "Recruitment.API.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime image
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app

# Install curl for health checks
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Copy published app
COPY --from=publish /app/publish .

# Change ownership
RUN chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:80/health || exit 1

# Start the application
ENTRYPOINT ["dotnet", "Recruitment.API.dll"]