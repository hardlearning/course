# 13. Working with Conan Repositories and Uploading Packages

## Setting up Artifactory CE

```
# to run docker with Artifactory:
docker run --name artifactory -d \
    -e JF_SHARED_DATABASE_TYPE=derby \
    -e JF_SHARED_DATABASE_ALLOWNONPOSTGRESQL=true \
    -p 8081:8081 -p 8082:8082 \
    releases-docker.jfrog.io/jfrog/artifactory-cpp-ce:latest
```

Access Artifactory at http://localhost:8082 (admin:password)

## Set up the remote in Conan

```
# list available conan remotes
conan remote list

# add our new remote to conan
conan remote add myartifactory http://localhost:8081/artifactory/api/conan/conan-local

conan remote login myartifactory admin -p <your_password>

# list again, myartifactory should be there
conan remote list
```

## Uploading the package

```
conan create . --build=missing

# search for all packages in our remote
conan search "*" -r=myartifactory

# upload our hello package
conan upload hello -r=myartifactory

# check everything is fine
conan list "hello/1.0:*" -r=myartifactory
```

## Verifying the Package from Remote

```
conan remove hello -c

conan install --requires=hello/1.0 -r=myartifactory
```