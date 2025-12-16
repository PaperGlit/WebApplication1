# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# This stage is used when running from VS in fast mode (Default for Debug configuration)
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS base
USER $APP_UID
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

# Please select the corresponding download for your Linux containers https://github.com/DataDog/dd-trace-dotnet/releases/latest

# Download and install the Tracer
USER root

RUN apt-get update \
    && apt-get install -y curl dpkg \
    && curl -fL https://github.com/DataDog/dd-trace-dotnet/releases/download/v3.33.0/datadog-dotnet-apm_3.33.0_amd64.deb \
       -o /tmp/datadog-dotnet-apm.deb \
    && dpkg -i /tmp/datadog-dotnet-apm.deb \
    && rm /tmp/datadog-dotnet-apm.deb \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

USER $APP_UID

# Enable the tracer
ENV CORECLR_ENABLE_PROFILING=1
ENV CORECLR_PROFILER={846F5F1C-F9AE-4B07-969E-05C26BC060D8}
ENV CORECLR_PROFILER_PATH=/opt/datadog/Datadog.Trace.ClrProfiler.Native.so
ENV DD_DOTNET_TRACER_HOME=/opt/datadog
ENV DD_TRACE_ENABLED=true
ENV DD_TRACE_DEBUG=true
ENV DD_TRACE_STARTUP_LOGS=true
ENV DD_TRACE_ASPNETCORE_ENABLED=true
ENV DD_TRACE_ROUTE_TEMPLATE_RESOURCE_NAMES_ENABLED=true

# This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["WebApplication1/WebApplication1.csproj", "WebApplication1/"]
RUN dotnet restore "./WebApplication1/WebApplication1.csproj"
COPY . .
WORKDIR "/src/WebApplication1"
RUN dotnet build "./WebApplication1.csproj" -c $BUILD_CONFIGURATION -o /app/build

# This stage is used to publish the service project to be copied to the final stage
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./WebApplication1.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# This stage is used in production or when running from VS in regular mode (Default when not using the Debug configuration)
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "WebApplication1.dll"]