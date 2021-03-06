# *****************************************************
# Based on the official python docker image/file
# https://github.com/docker-library/python
# *****************************************************


# Base on windows servercore
FROM mcr.microsoft.com/windows/servercore:2004

# Set shell to be used as powershell
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ENV PYTHON_VERSION       3.9.0
ENV PYTHON_RELEASE       3.9.0
ENV PYTHON_CHECKSUM_MD5  b61a33dc28f13b561452f3089c87eb63


# **********************************************
# Download- and install Python
# **********************************************

RUN $url = ('https://www.python.org/ftp/python/{0}/python-{1}-amd64.exe' -f $env:PYTHON_RELEASE, $env:PYTHON_VERSION ); \
    Write-Host ('Downloading {0} ...' -f $url); \
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
    Invoke-WebRequest -Uri $url -OutFile 'python.exe'; \
 \
    Write-Host 'Verifying download ...'; \
    if((Get-FileHash 'python.exe' -Algorithm md5).Hash -ne $env:PYTHON_CHECKSUM_MD5) { \
        Write-Host 'FAILED!'; \
        exit 1; \
    }; \
 \
    Write-Host 'Installing ...'; \
# https://docs.python.org/3.5/using/windows.html#installing-without-ui
    Start-Process python.exe -Wait \
        -ArgumentList @( \
            '/quiet', \
            'InstallAllUsers=1', \
            'TargetDir=C:\\Python', \
            'PrependPath=1', \
            'Shortcuts=0', \
            'Include_doc=0', \
            'Include_pip=0', \
            'Include_test=0' \
        ); \
# the installer updated PATH, so we should refresh our local value
    $env:PATH = [Environment]::GetEnvironmentVariable('PATH', [EnvironmentVariableTarget]::Machine); \
 \
    Write-Host 'Verifying install ...'; \
 \
    Write-Host ' python --version'; python --version; \
 \
    Write-Host 'Cleaning up ...'; \
    Remove-Item python.exe -Force; \
 \
    Write-Host 'Complete.'

# **********************************************
# Download- and install PIP
# **********************************************

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 20.2.3
# https://github.com/pypa/get-pip
ENV PYTHON_GET_PIP_URL    https://github.com/pypa/get-pip/raw/fa7dc83944936bf09a0e4cb5d5ec852c0d256599/get-pip.py
ENV PYTHON_GET_PIP_SHA256 6e0bb0a2c2533361d7f297ed547237caf1b7507f197835974c0dd7eba998c53c

RUN Write-Host ('Downloading get-pip.py ({0}) ...' -f $env:PYTHON_GET_PIP_URL); \
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
    Invoke-WebRequest -Uri $env:PYTHON_GET_PIP_URL -OutFile 'get-pip.py'; \
    Write-Host ('Verifying sha256 ({0}) ...' -f $env:PYTHON_GET_PIP_SHA256); \
    if ((Get-FileHash 'get-pip.py' -Algorithm sha256).Hash -ne $env:PYTHON_GET_PIP_SHA256) { \
        Write-Host 'FAILED!'; \
        exit 1; \
    }; \
 \
    Write-Host ('Installing pip=={0} ...' -f $env:PYTHON_PIP_VERSION); \
    python get-pip.py \
        --disable-pip-version-check \
        --no-cache-dir \
        ('pip=={0}' -f $env:PYTHON_PIP_VERSION) \
    ; \
    Remove-Item get-pip.py -Force; \
 \
    Write-Host 'Verifying pip install ...'; \
    pip --version; \
 \
    Write-Host 'Complete.'

# Set /app to workingdir, copy webcontents and set entrypoint to
# start server in /app dir
WORKDIR /app

COPY ./webcontents .

ENTRYPOINT ["python", "-m", "http.server", "80"]