# because of dotnet, we always build on amd64, and target platforms in cli
# dotnet doesn't support QEMU for building or running. 
# (errors common in arm/v7 32bit) https://github.com/dotnet/dotnet-docker/issues/1537
# https://hub.docker.com/_/microsoft-dotnet
# hadolint ignore=DL3029
# to build for a different platform than your host, use --platform=<platform>
# for example, if you were on Intel (amd64) and wanted to build for ARM, you would use:
# docker buildx build --platform "linux/arm64/v8" .


FROM mcr.microsoft.com/dotnet/sdk:7.0 AS builder
WORKDIR /app

# caches restore result by copying csproj file separately
COPY *.csproj .
RUN dotnet restore

COPY . .
RUN dotnet publish --output /app/ --configuration Release --no-restore
RUN sed -n 's:.*<AssemblyName>\(.*\)</AssemblyName>.*:\1:p' *.csproj > __assemblyname
RUN if [ ! -s __assemblyname ]; then filename=$(ls *.csproj); echo ${filename%.*} > __assemblyname; fi

# Stage 2
FROM mcr.microsoft.com/dotnet/aspnet:7.0
WORKDIR /app
COPY --from=builder /app .

ENV PORT 5000
EXPOSE 5000

ENTRYPOINT dotnet $(cat /app/__assemblyname).dll --urls "http://*:5000"