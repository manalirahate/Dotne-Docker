# escape=`

# Installer image
FROM mcr.microsoft.com/windows/servercore:1809 AS installer

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Retrieve .NET Core SDK
ENV DOTNET_SDK_VERSION 3.0.100-rc1-014190

RUN Invoke-WebRequest -OutFile dotnet.zip https://dotnetcli.blob.core.windows.net/dotnet/Sdk/$Env:DOTNET_SDK_VERSION/dotnet-sdk-$Env:DOTNET_SDK_VERSION-win-x64.zip; `
    $dotnet_sha512 = 'ebea78935351142774f5a43f5b8604460c4a5aca5ae2073d4392c909992ab3ae09239f2e08080503cb6979cfbd12c42429fecbfd1f7d7f00aafc6e5849fbdd1a'; `
    if ((Get-FileHash dotnet.zip -Algorithm sha512).Hash -ne $dotnet_sha512) { `
        Write-Host 'CHECKSUM VERIFICATION FAILED!'; `
        exit 1; `
    }; `
    `
    Expand-Archive dotnet.zip -DestinationPath dotnet; `
    Remove-Item -Force dotnet.zip

# Install PowerShell global tool
ENV POWERSHELL_VERSION 7.0.0-preview.2

RUN Invoke-WebRequest -OutFile PowerShell.Windows.x64.$ENV:POWERSHELL_VERSION.nupkg https://pwshtool.blob.core.windows.net/tool/$ENV:POWERSHELL_VERSION/PowerShell.Windows.x64.$ENV:POWERSHELL_VERSION.nupkg; `
    $powershell_sha512 = '54eacb7db655c8de5f824a64376958514524ed1ad420f04faa54b90ace824b90e5709b362cbc229dec5d5f02b58a903edcf4424544786faa9d9cb0dfb52f69d4'; `
    if ((Get-FileHash PowerShell.Windows.x64.$ENV:POWERSHELL_VERSION.nupkg -Algorithm sha512).Hash -ne $powershell_sha512) { `
        Write-Host 'CHECKSUM VERIFICATION FAILED!'; `
        exit 1; `
    }; `
    `
    \dotnet\dotnet tool install --add-source . --tool-path \powershell --version $ENV:POWERSHELL_VERSION PowerShell.Windows.x64; `
    Remove-Item -Force PowerShell.Windows.x64.$ENV:POWERSHELL_VERSION.nupkg; `
    Remove-Item -Path \powershell\.store\powershell.windows.x64\$ENV:POWERSHELL_VERSION\powershell.windows.x64\$ENV:POWERSHELL_VERSION\powershell.windows.x64.$ENV:POWERSHELL_VERSION.nupkg -Force

# SDK image
FROM mcr.microsoft.com/windows/nanoserver:1809

COPY --from=installer ["/dotnet", "/Program Files/dotnet"]

COPY --from=installer ["/powershell", "/Program Files/powershell"]

# In order to set system PATH, ContainerAdministrator must be used
USER ContainerAdministrator
RUN setx /M PATH "%PATH%;C:\Program Files\dotnet;C:\Program Files\powershell"
USER ContainerUser

# Configure web servers to bind to port 80 when present
ENV ASPNETCORE_URLS=http://+:80 `
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true `
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true `
    # Skip extraction of XML docs - generally not useful within an image/container - helps performance
    NUGET_XMLDOC_MODE=skip

# Trigger first run experience by running arbitrary cmd
RUN dotnet help
