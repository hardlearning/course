# 7. Scanning C++ packages for vulnerabilities using Conan Audit

## Registering for the audit service

https://audit.conan.io/register

## Authenticating and Scanning out project

```
# Authenticate with Conan Audit
conan audit provider auth conancenter --token=<your_token_here>
# Or set the token using the CONAN_AUDIT_PROVIDER_TOKEN_CONANCENTER environment variable

# Scan dependency graph
conan audit scan .

# Scan a package that we know has vulnerabilities
conan audit list libpng/1.6.32
```

## Generating reports

```
# Scan libpng
conan audit list libpng/1.6.32 --format=html > report.html
```

## Using a private provider

```
# Configure a Private Provider
conan audit provider add myprovider --type=private --url=https://your.artifactory.url --token=<your_token>

# Scan dependency graph
conan audit scan . --provider=myprovider
```