#Starting from Windows Server Core [1]
FROM microsoft/windowsservercore

# Install IIS Web Server [2]
RUN powershell -Command Add-WindowsFeature Web-Server

# hack for DNS cache bug
# https://github.com/docker/docker/issues/27499
RUN powershell -Command "set-itemproperty -path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name ServerPriorityTimeLimit -Value 0 -Type DWord"

# Install Chocolatey [3]
RUN @powershell -NoProfile -ExecutionPolicy unrestricted -Command "$env:chocolateyUseWindowsCompression = 'false'; (iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1')))"

# Remove existing websites (and port bindings as well)
RUN powershell -Command "Remove-Item c:/inetpub/wwwroot/iisstart.*"

# Install build tools [3]
RUN powershell add-windowsfeature web-asp-net45 \
    && choco install microsoft-build-tools -y --allow-empty-checksums -version 14.0.23107.10 \
    && choco install dotnet4.6-targetpack --allow-empty-checksums -y \
    && choco install nuget.commandline --allow-empty-checksums -y \
    && nuget install MSBuild.Microsoft.VisualStudio.Web.targets -Version 14.0.0.3 \
    && nuget install WebConfigTransformRunner -Version 1.0.0.1

# Copy files
RUN md c:\build
WORKDIR c:/build
COPY . c:/build

# Restore packages, build, copy
RUN nuget restore \
    && "c:\Program Files (x86)\MSBuild\14.0\Bin\MSBuild.exe" /p:Platform="Any CPU" /p:VisualStudioVersion=12.0 /p:VSToolsPath=c:\MSBuild.Microsoft.VisualStudio.Web.targets.14.0.0.3\tools\VSToolsPath SampleWebAppWithDocker.sln \
    && xcopy c:\build\SampleWebAppWithDocker\* c:\inetpub\wwwroot /s

EXPOSE 80

# Run blocking check
ENTRYPOINT ["powershell", ".\\wait_until_w3svc_done.ps1"] 