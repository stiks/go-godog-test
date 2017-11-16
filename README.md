### Why?

This docker images is used for Gitlab CI. I'm using Godog scenarious to test final build stage. It's taking ages to get build dependancies. In this case I've decide to create custom Docker image with main components I used in all my projects

As result I've managed to reduce Gitlab CI cycle from 5 min 30 sec (unit + behaviour + build + deploy) to 2 min 15 sec. Build itself is taking around 1 min.

#### How to use it

This is my old Gitlab CI testing job
```
test project:
  image: golang:1.9
  services:
    - mysql:5.7
  before_script:
    - go version
    - go get -t
  stage: test
  variables:
    APP_PORT: 8080
    APP_JWT_SECRET: RANDOM_STRING_GEN
  script:
    - go test -v
```

New config is looking like this:
```
test project:
  image: golang:1.9
  services:
    - mysql:5.7
  before_script:
    - go version
    - FIRSTGOPATH="$(go env GOPATH | cut -d':' -f1)"
    - mkdir -p "${FIRSTGOPATH}/src/${REPO_DIR}"
    - ln -s "${CI_PROJECT_DIR}" "${FIRSTGOPATH}/src/${REPO_DIR}/api"
    - git config --global url."https://gitlab-ci-token:$CI_REGISTRY_TOKEN@git.iterium.co.uk/".insteadOf "https://git.iterium.co.uk/"
    - cd "${FIRSTGOPATH}/src/${REPO_DIR}/api"
    - go get -t
  stage: test
  variables:
    APP_PORT: 8080
    APP_JWT_SECRET: RANDOM_STRING_GEN
  script:
    - go test -v

```

I'm using this docker configuration for my app:
```
FROM scratch

ADD ./app /

EXPOSE 8080
ENTRYPOINT ["/app"]
```
