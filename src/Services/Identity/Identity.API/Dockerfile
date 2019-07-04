FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/core/sdk:2.2 AS build
WORKDIR /src

COPY scripts scripts/

COPY src/ApiGateways/*/*.csproj /src/csproj-files/
COPY src/ApiGateways/*/*/*.csproj /src/csproj-files/
COPY src/BuildingBlocks/*/*/*.csproj /src/csproj-files/
COPY src/Services/*/*/*.csproj /src/csproj-files/
COPY src/Web/*/*.csproj /src/csproj-files/

COPY . .
WORKDIR /src/src/Services/Identity/Identity.API
RUN dotnet add Identity.API.csproj package Instana.Profiler.Linux.Bundled.Refs --version 1.0.53.21
RUN dotnet publish -c Release -o /app

FROM build AS publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
# The INSTANA Tracer needs libc++ and this is how we install it
RUN apt-get update && apt-get install libc++1

# Some env-vars, common to all profilers/tracers for .NET Core. These tell the runtime
# to load the Rewriter before starting the process, so that we can rewrite the IL code and call into our tracer.
ENV CORECLR_ENABLE_PROFILING=1
ENV CORECLR_PROFILER={cf0d821e-299b-5307-a3d8-b283c03916dd}
ENV CORECLR_PROFILER_PATH=/app/instana_tracing/CoreProfiler.so

# Env-vars which point to the INSTANA agent. This is the endpoint the traced
# application will call out to in order to get env-data and send traces.
ENV INSTANA_AGENT_HOST=docker.for.win.localhost
ENV INSTANA_AGENT_PORT=42699
ENTRYPOINT ["dotnet", "Identity.API.dll"]
